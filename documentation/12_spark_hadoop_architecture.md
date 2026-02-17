# Spark & Hadoop Architecture Documentation

## 1. Executive Summary

PropIntel utilizes **Apache Spark** and **Hadoop** to enable scalable, distributed data processing and machine learning capabilities. While the application currently runs in a "local" mode within Docker containers for simplicity, the underlying architecture is designed to scale horizontally across a cluster. This document details how PySpark interacts with the Hadoop ecosystem to execute complex ML pipelines and manage data processing tasks.

---

## 2. Component Integration

### 2.1 Spark-Hadoop Ecosystem

```mermaid
graph TD
    App[Streamlit App] -->|Initialize| SparkSession[Spark Session]
    
    subgraph "Spark Execution Engine"
        SparkSession -->|Driver| Driver[Driver Program]
        Driver -->|Task| Executor[Executor 1 (Local)]
        
        Executor -->|Read| HadoopFS[Hadoop FileSystem API]
        HadoopFS -->|I/O| LocalFS[Local File System / HDFS]
    end
    
    subgraph "Libraries"
        MLlib[Spark MLlib] -->|Algorithm| LinearReg[LinearRegression]
        SQL[Spark SQL] -->|Schema| DataFrame[DataFrame API]
    end
    
    Executor -.-> MLlib
    Executor -.-> SQL
```

### 2.2 Roles & Responsibilities

#### Apache Spark (Compute)
- **Driver**: The Python process (PySpark) managed by `st.cache_resource`.
- **Executor**: Local threads processing tasks (Feature Engineering, Prediction).
- **Catalyst Optimizer**: Optimizes logical plans for DataFrame transformations.
- **Tungsten**: Optimizes physical execution (memory management).

#### Apache Hadoop (Storage & Resource)
- **Hadoop Common**: Provides libraries (like `winutils.exe` on Windows) required for file system operations.
- **HDFS**: Distributed File System (abstracted as local file system in current deployment).
- **YARN**: (Optional) Resource negotiator for cluster deployment (not active in local mode).

---

## 3. Implementation Details

### 3.1 Session Configuration
The Spark Session is configured for local execution to maximize resource utilization on single machines/containers.

```python
spark = SparkSession.builder \
    .appName("RealEstateApp") \
    .config("spark.driver.host", "localhost") \
    .config("spark.ui.showConsoleProgress", "false") \
    .master("local[*]") \ # Uses all available cores
    .getOrCreate()
```

### 3.2 ML Pipeline Architecture
PySpark ML pipelines define a sequence of stages that transform data and apply the model.

1. **StringIndexer (Hadoop/Spark)**: Maps string columns to indices.
2. **OneHotEncoder**: Converts indices to binary vectors.
3. **VectorAssembler**: Combines all features into a single dense/sparse vector.
4. **LinearRegression**: The predictive model.

This pipeline is **saved** and **loaded** as a serialized object, ensuring consistency between training (Notebook) and inference (App).

---

## 4. Performance Optimization

### 4.1 Caching
Spark's lazy evaluation is powerful, but repeated actions can be costly. We use Streamlit caching (`@st.cache_resource`) to keep the Spark Context alive, avoiding the 10-15s startup overhead.

### 4.2 Vectorization
Data processing in Spark (even local) benefits from JVM-level optimizations for vectorized operations on columnar data, significantly faster than row-by-row processing in pure Python.

### 4.3 Memory Management
- **Driver Memory**: Configured implicitly. In Docker, capped by container limit (2GB).
- **Executor Memory**: Shared with driver in local mode.

---

## 5. Deployment Considerations

### 5.1 Docker Environment
The Docker image includes `openjdk-17-jre-headless` to support the Spark JVM execution environment.

```dockerfile
ENV JAVA_HOME="/usr/lib/jvm/java-17-openjdk-amd64"
ENV PYSPARK_PYTHON=python3
```

### 5.2 Windows compatibility
On Windows, Spark requires `winutils.exe` (Hadoop binaries) to perform file system operations. The application checks for this availability:

```python
if not SPARK_AVAILABLE:
    st.error("Spark functionality disabled... Missing winutils.exe")
```
*Solution*: Use Docker or Kubernetes on Windows to bypass this requirement by running in a Linux environment.

---

## 6. Scalability Logic

To scale from "Local" to "Cluster" mode:
1. **Update Master**: Change `.master("local[*]")` to `.master("spark://master-node:7077")`.
2. **Data Access**: Move CSVs to HDFS or S3 (`s3a://...`) so all nodes can access them.
3. **Resource Mgmt**: Enable YARN or K8s as the resource manager.

---

## 7. Future Enhancements

- **Spark Serving**: switch to Spark Structured Streaming for real-time inference if throughput increases.
- **Delta Lake**: Adopt Delta Lake format for ACID transactions on the dataset.
- **Model Registry**: Use MLflow (integrated with Spark) to manage model versions.
