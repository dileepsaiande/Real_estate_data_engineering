# PropIntel Documentation Index

## Overview

This documentation provides comprehensive technical analysis of the **PropIntel Real Estate Prediction Application**, suitable for academic research and technical reference. The system combines machine learning, big data processing (Apache Spark & Hadoop), and AI-powered insights to deliver accurate property price predictions and intelligent recommendations.

---

## Documentation Structure

### 📋 Feature-Level Documentation

1. [**Authentication System**](01_authentication_system.md)
   - Login/Signup mechanisms
   - Role-based access control (RBAC)
   - Session management
   - Password hashing and security

2. [**Customer Dashboard**](02_customer_dashboard.md)
   - Property search and filtering
   - Weighted scoring algorithm
   - AI-powered recommendations
   - Data normalization techniques

3. [**Admin Property Management**](03_admin_property_management.md)
   - Property data entry system
   - UUID-based ID generation
   - Data validation and deduplication
   - CSV persistence layer

4. [**ML Price Prediction**](04_ml_price_prediction.md)
   - Buy price prediction (per sqft + total)
   - Rent price prediction (monthly)
   - Feature engineering pipeline
   - Model loading and caching strategies

5. [**AI Insights Engine**](05_ai_insights_engine.md)
   - Ollama/DeepSeek integration
   - Prompt engineering for real estate
   - Market trend analysis
   - Negotiation advice generation

---

### 🤖 Model-Level Documentation

6. [**Model Training Pipeline**](06_model_training_pipeline.md)
   - CatBoost regression training
   - Data preprocessing and feature engineering
   - Model evaluation (RMSE, R²)
   - Model export and versioning

7. [**Buy Price Model Architecture**](07_buy_price_model_architecture.md)
   - PySpark Linear Regression model
   - 57-column feature set
   - Pipeline transformations
   - Performance metrics (R² = 0.8178)

8. [**Rent Price Model Architecture**](08_rent_price_model_architecture.md)
   - PySpark Linear Regression model
   - Rental-specific features
   - Price normalization (10k-40k range)
   - Performance metrics (R² = 0.5816)

---

### 🏗️ Architecture Documentation

9. [**Application Architecture**](09_application_architecture.md)
   - Multi-page Streamlit application
   - Component interaction diagrams
   - Session state management
   - File structure and organization

10. [**System Architecture**](10_system_architecture.md)
    - Deployment architectures (Local, Docker, Kubernetes)
    - Network topology
    - Container orchestration
    - Resource allocation and scalability

11. [**Data Architecture**](11_data_architecture.md)
    - Data sources and storage (CSV, S3)
    - 57-column schema definition
    - ETL processes and data flow
    - Normalization strategies

12. [**Spark & Hadoop Architecture**](12_spark_hadoop_architecture.md)
    - Spark cluster architecture
    - Hadoop's role in distributed processing
    - PySpark execution model
    - Resource management and optimization

---

### 📐 Design Documentation

13. [**High-Level Design**](13_high_level_design.md)
    - System overview and components
    - Technology stack
    - Deployment strategies
    - Security considerations

14. [**Low-Level Design**](14_low_level_design.md)
    - Class and sequence diagrams
    - Function-level documentation
    - Algorithm implementations
    - Code flow analysis

---

### 🐳 Deployment Documentation

15. [**Docker Architecture**](15_docker_architecture.md)
    - Dockerfile analysis
    - Multi-stage builds
    - Volume mounting strategies
    - Environment configuration

16. [**Kubernetes Architecture**](16_kubernetes_architecture.md)
    - Deployment manifests
    - Service configuration
    - Resource limits and health checks
    - Scaling strategies

---

## Technology Stack

| Layer | Technologies |
|-------|-------------|
| **Frontend** | Streamlit 1.43.2, Lottie Animations |
| **Backend** | Python 3.11, PySpark 3.5.5 |
| **ML Framework** | PySpark MLlib, CatBoost 1.2.8 |
| **Big Data** | Apache Spark 3.5.5, Hadoop (HDFS/YARN) |
| **AI/LLM** | Ollama, DeepSeek-v3.1:671b-cloud |
| **Data Processing** | Pandas 1.5.3, NumPy <2.0.0 |
| **Deployment** | Docker, Kubernetes, Docker Compose |
| **Storage** | CSV (local), AWS S3 (optional) |

---

## Key Features

### 🎯 Core Capabilities
- **Dual Prediction Models**: Separate models for buy and rent price predictions
- **Big Data Processing**: Apache Spark for distributed ML pipeline execution
- **AI-Powered Insights**: Ollama integration for intelligent property recommendations
- **Role-Based Access**: Customer and Admin/Owner dashboards
- **Scalable Deployment**: Docker and Kubernetes support

### 📊 Performance Metrics
- **Buy Model**: RMSE = 3,188.88, R² = 0.8178
- **Rent Model**: RMSE = 11,004.18, R² = 0.5816
- **Dataset**: 100,000+ properties with 57 features
- **Prediction Speed**: ~1-2 seconds (cached Spark session)

---

## Research Applications

This documentation is designed to support:
- **Academic Research**: Detailed technical analysis suitable for research papers
- **System Replication**: Complete implementation details for reproduction
- **Performance Analysis**: Benchmarks and optimization strategies
- **Architecture Studies**: Multi-tier system design patterns
- **ML Pipeline Research**: Big data ML pipeline implementation

---

## Quick Navigation

### For Researchers
1. Start with [High-Level Design](13_high_level_design.md) for system overview
2. Review [Spark & Hadoop Architecture](12_spark_hadoop_architecture.md) for big data insights
3. Examine [Model Training Pipeline](06_model_training_pipeline.md) for ML methodology

### For Developers
1. Begin with [Application Architecture](09_application_architecture.md)
2. Study [Feature Documentation](#-feature-level-documentation) for implementation details
3. Review [Deployment Documentation](#-deployment-documentation) for setup

### For System Architects
1. Read [System Architecture](10_system_architecture.md)
2. Analyze [Data Architecture](11_data_architecture.md)
3. Study [Low-Level Design](14_low_level_design.md)

---

## Document Conventions

- **Code Blocks**: Annotated with line-by-line explanations
- **Diagrams**: Mermaid diagrams for visual representation
- **File Links**: Clickable links to source files
- **Performance Notes**: Benchmarks and optimization tips
- **Use Cases**: Real-world scenarios and examples

---

## Version Information

- **Application Version**: 2.1
- **Documentation Date**: February 2026
- **Python Version**: 3.11
- **PySpark Version**: 3.5.5
- **Spark Version**: 3.5.5

---

## Additional Resources

- [Implementation Plan](implementation_plan.md)
- [Task Breakdown](task.md)
- [README](../README.md)
- [Quick Start Guide](../QUICKSTART.md)

---

**Note**: All documentation includes detailed code explanations, architecture diagrams, and performance analysis suitable for academic research and technical implementation.
