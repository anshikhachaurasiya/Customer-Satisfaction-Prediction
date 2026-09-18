# 📊 Customer Satisfaction Prediction: Production ML Pipeline

> An end-to-end MLOps pipeline for predicting customer satisfaction from e-commerce data, with automated data processing, model training, experiment tracking, evaluation, conditional model deployment, and a Streamlit prediction application.

---

## 🚀 Overview

Customer satisfaction is influenced by multiple factors including:

* Order status
* Product characteristics
* Price
* Payment method
* Freight performance
* Delivery information
* Customer and seller attributes

This project builds a **production-oriented machine learning pipeline** to predict customer review scores from historical e-commerce data.

Instead of training a model once and manually deploying it, the project uses **ZenML and MLflow** to create a reproducible workflow covering:

```text
Raw Data
   ↓
Data Ingestion
   ↓
Data Cleaning
   ↓
Feature Preparation
   ↓
Model Training
   ↓
Model Evaluation
   ↓
Deployment Trigger
   ↓
Conditional Model Deployment
   ↓
Streamlit Prediction App
```

The underlying dataset is the **Brazilian E-Commerce Public Dataset by Olist**, containing approximately 100,000 orders from multiple marketplaces in Brazil.

---

# 🎯 Problem Statement

Given historical information about an e-commerce order, predict the **customer satisfaction/review score** associated with the purchase.

The model uses information available around the order, including factors such as:

```text
Order Information
      +
Product Information
      +
Payment Information
      +
Price / Freight
      +
Delivery Information
      ↓
Customer Satisfaction Prediction
```

The objective is to create a reusable ML workflow that can continuously train, evaluate, and deploy improved models.

---

# 🏗️ Architecture

```text
                         ┌──────────────────┐
                         │   Olist Dataset  │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │  Data Ingestion  │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │   Data Cleaning  │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │  Model Training  │
                         │                  │
                         │  MLflow Tracking │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │ Model Evaluation │
                         │                  │
                         │ MSE / Metrics    │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │ Deployment       │
                         │ Trigger          │
                         └────────┬─────────┘
                                  │
                         Meets Threshold?
                           │            │
                          YES           NO
                           │            │
                           ▼            ▼
                  ┌──────────────┐   Stop
                  │ MLflow Model │
                  │  Deployment  │
                  └──────┬───────┘
                         │
                         ▼
                  ┌──────────────┐
                  │   Streamlit  │
                  │     App      │
                  └──────────────┘
```

---

# 🔄 ML Pipeline

The training workflow is implemented as a sequence of modular ZenML steps.

## 1. Data Ingestion

The ingestion step loads the source dataset into a Pandas DataFrame.

```text
Dataset
   ↓
ingest_data
   ↓
DataFrame
```

---

## 2. Data Cleaning

The cleaning stage removes unwanted columns and prepares the dataset for model training.

```text
Raw DataFrame
      ↓
Cleaning
      ↓
Processed Features
```

This separates data preparation from the model-training logic and allows the pipeline to be reproduced consistently.

---

## 3. Model Training

The training step trains the machine learning model and uses **MLflow autologging** to capture model-related information.

MLflow tracks:

* Model parameters
* Training information
* Evaluation metrics
* Model artifacts

The repository explicitly uses MLflow tracking as part of the ZenML pipeline.

---

## 4. Model Evaluation

The trained model is evaluated before it can be considered for deployment.

The project uses **Mean Squared Error (MSE)** as the configurable deployment criterion.

```text
Trained Model
     ↓
Evaluation
     ↓
MSE
     ↓
Deployment Decision
```

---

# 🚦 Conditional Model Deployment

One of the key MLOps components is the **deployment trigger**.

A newly trained model is not automatically deployed.

Instead:

```text
New Model
    ↓
Evaluate Model
    ↓
Compare Against Threshold
    │
    ├── Meets Criteria → Deploy
    │
    └── Fails Criteria → Do Not Deploy
```

The deployment pipeline adds a `deployment_trigger` step that checks whether the newly trained model meets the configured evaluation criterion before deploying it.

This prevents an inferior model from automatically replacing the currently deployed model.

---

# 📦 MLflow Model Deployment

When the model satisfies the deployment criteria, the pipeline uses the **MLflow model deployer** to serve the new model.

```text
ZenML Pipeline
      ↓
MLflow Tracking
      ↓
Evaluation
      ↓
Deployment Trigger
      ↓
MLflow Model Deployer
      ↓
Prediction Service
```

When a newer model passes the configured threshold, the running MLflow deployment is updated to serve the new model.

> **Note:** The repository uses local MLflow deployment for this implementation. Cloud/Kubernetes deployment is not part of the current implementation.

---

# 🖥️ Streamlit Prediction Application

A Streamlit application provides a simple interface for making predictions using the latest deployed model.

```text
User Input
    ↓
Streamlit
    ↓
MLflow Prediction Service
    ↓
Latest Deployed Model
    ↓
Customer Satisfaction Prediction
```

The application consumes the prediction service asynchronously from the deployment pipeline.

---

# 🧠 MLOps Workflow

The project demonstrates the following MLOps lifecycle:

```text
                 ┌────────────────────┐
                 │     Data Source    │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │   Data Ingestion   │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │   Data Cleaning    │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │  Model Training    │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │ Model Evaluation   │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │ Deployment Trigger │
                 └─────────┬──────────┘
                           │
                    ┌──────┴──────┐
                    │             │
                  PASS           FAIL
                    │             │
                    ▼             ▼
             Deploy Model       Reject
                    │
                    ▼
             Prediction API
                    │
                    ▼
               Streamlit
```

---

# 🛠️ Technology Stack

| Category               | Technology                                   |
| ---------------------- | -------------------------------------------- |
| Programming            | Python                                       |
| Data Processing        | Pandas                                       |
| Machine Learning       | Scikit-learn                                 |
| Pipeline Orchestration | ZenML                                        |
| Experiment Tracking    | MLflow                                       |
| Model Deployment       | MLflow Model Deployer                        |
| Frontend               | Streamlit                                    |
| Dataset                | Brazilian E-Commerce Public Dataset by Olist |

---

# 📁 Project Structure

```text
customer-satisfaction-mlops/
│
├── data/
│   └── Dataset files
│
├── materializer/
│   └── Custom materialization components
│
├── model/
│   └── Model-related components
│
├── pipelines/
│   └── ZenML pipeline definitions
│
├── steps/
│   ├── ingest_Data.py
│   ├── clean_Data.py
│   ├── model_train.py
│   └── evalution.py
│
├── tests/
│   └── Pipeline tests
│
├── saved_model/
│   └── Saved model artifacts
│
├── config.yaml
├── run_pipeline.py
├── run_deployment.py
├── streamlit_app.py
├── requirements.txt
└── README.md
```

The repository separates pipeline definitions, individual pipeline steps, model artifacts, tests, data, and the Streamlit application.

---

# ⚙️ Installation

## 1. Clone the Repository

```bash
git clone https://github.com/ayush714/customer-satisfaction-mlops.git

cd customer-satisfaction-mlops
```

---

## 2. Create a Virtual Environment

### Windows

```bash
python -m venv venv

venv\Scripts\activate
```

### Linux / macOS

```bash
python3 -m venv venv

source venv/bin/activate
```

---

## 3. Install Dependencies

```bash
pip install -r requirements.txt
```

---

# 🔧 Configure ZenML

The project requires a ZenML stack containing:

* MLflow experiment tracker
* MLflow model deployer

Install the MLflow integration:

```bash
zenml integration install mlflow -y
```

Register the experiment tracker:

```bash
zenml experiment-tracker register mlflow_tracker --flavor=mlflow
```

Register the model deployer:

```bash
zenml model-deployer register mlflow --flavor=mlflow
```

Register the ZenML stack:

```bash
zenml stack register mlflow_stack \
  -a default \
  -o default \
  -d mlflow \
  -e mlflow_tracker \
  --set
```

The repository requires a ZenML stack configured with MLflow tracking and model deployment components.

---

# ▶️ Running the Training Pipeline

Run:

```bash
python run_pipeline.py
```

This executes the standard training pipeline:

```text
Ingestion
   ↓
Cleaning
   ↓
Training
   ↓
Evaluation
```

---

# 🚀 Running the Continuous Deployment Pipeline

Run:

```bash
python run_deployment.py
```

The deployment pipeline extends the training workflow:

```text
Ingestion
   ↓
Cleaning
   ↓
Training
   ↓
Evaluation
   ↓
Deployment Trigger
   ↓
MLflow Model Deployment
```

A new model is deployed only when it meets the configured evaluation threshold.

---

# 🖥️ Run the Streamlit Application

After starting the model deployment:

```bash
streamlit run streamlit_app.py
```

The application allows users to provide order/product-related inputs and obtain a customer satisfaction prediction from the latest deployed model.

---

# 📊 Experiment Tracking

MLflow is integrated with ZenML to track:

* Hyperparameters
* Model artifacts
* Training information
* Evaluation metrics
* Model versions

This makes experiments reproducible and provides visibility into model performance across pipeline runs.

---

# 🧪 Model Evaluation

The deployment process uses an evaluation threshold to decide whether a trained model should be deployed.

```text
              Model Training
                    │
                    ▼
              Model Evaluation
                    │
                    ▼
                  MSE
                    │
                    ▼
          ┌─────────────────────┐
          │ Deployment Threshold│
          └──────────┬──────────┘
                     │
              ┌──────┴──────┐
              │             │
            Better        Worse
              │             │
              ▼             ▼
           Deploy        Reject
```

This creates a basic **quality gate** between model training and deployment.

---

# 📌 Key MLOps Concepts Demonstrated

This project demonstrates:

* Modular ML pipelines
* Data ingestion
* Data preprocessing
* Model training
* Model evaluation
* Experiment tracking
* Model artifact management
* Model deployment
* Conditional deployment
* Pipeline orchestration
* Reproducible workflows
* ML model serving
* Streamlit-based inference

---

# 🔮 Future Improvements

Potential extensions include:

* [ ] Automated data validation
* [ ] Feature engineering pipeline
* [ ] Data drift detection
* [ ] Model drift monitoring
* [ ] Automated retraining
* [ ] CI/CD integration
* [ ] Cloud model deployment
* [ ] Kubernetes deployment
* [ ] Model registry
* [ ] Automated model comparison
* [ ] Production monitoring
* [ ] API-based model serving

---

# 📈 Production Workflow

The complete intended workflow is:

```text
                  New Data
                     │
                     ▼
              Data Ingestion
                     │
                     ▼
              Data Processing
                     │
                     ▼
               Model Training
                     │
                     ▼
               MLflow Tracking
                     │
                     ▼
              Model Evaluation
                     │
                     ▼
             Quality Gate / MSE
                     │
             ┌───────┴───────┐
             │               │
           PASS             FAIL
             │               │
             ▼               ▼
      Model Deployment     Reject
             │
             ▼
       Prediction Service
             │
             ▼
        Streamlit App
```

---

# 🎯 Project Highlights

### End-to-End ML Lifecycle

The project covers the complete workflow from raw data ingestion to model serving.

### Reproducible Pipelines

ZenML provides a structured pipeline abstraction for separating individual ML steps and connecting them into repeatable workflows.

### Experiment Tracking

MLflow tracks model parameters, metrics, and artifacts during pipeline execution.

### Conditional Deployment

Models are evaluated before deployment, preventing models that fail the configured quality criterion from replacing the currently deployed model.

### User-Facing Inference

A Streamlit application provides a practical interface for consuming the deployed model.

---

# 👨‍💻 Project

**Customer Satisfaction Prediction: Production ML Pipeline**

Built with:

```text
Python • Pandas • Scikit-learn
ZenML • MLflow • Streamlit
```

---

# 📚 Dataset

The project uses the **Brazilian E-Commerce Public Dataset by Olist**, containing approximately 100,000 orders from 2016–2018 across multiple marketplaces in Brazil.

The dataset contains information related to:

* Orders
* Products
* Payments
* Freight
* Customers
* Sellers
* Reviews
* Delivery

---

# ⭐ Summary

**Customer Satisfaction Prediction: Production ML Pipeline** demonstrates how a machine learning model can be transformed from an isolated training script into a structured MLOps workflow:

```text
Data
 ↓
Pipeline
 ↓
Training
 ↓
Evaluation
 ↓
Quality Gate
 ↓
Deployment
 ↓
Prediction
```

The project focuses on **reproducibility, experiment tracking, model evaluation, and automated deployment**, providing a foundation for extending the system toward a fully productionized ML platform.
