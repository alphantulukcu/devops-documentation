# MLOps

![MLOps](../screenshots/mlops-chart.png)

In the last day of the CodeCamp'24 Week 1, we focused on MLOps, specifically utilizing MLflow for managing machine learning workflows. The session, led by Gufran Yeşilyurt, provided an introduction to MLOps, covering key concepts, tools, and practical applications. Here's a detailed summary:

## Introduction to MLOps
The session began by discussing the importance of MLOps (Machine Learning Operations) and how it differs from traditional DevOps:

## Why MLOps? 
- People working on ML are different.

- Data scientists are not software developers. 

- Product is data, not code

- ML works differently

Unlike DevOps, which focuses on software engineering and infrastructure, MLOps incorporates aspects unique to ML, such as data versioning, model training, and hyperparameter tuning.

## MLflow Overview
MLflow is an open-source platform designed to manage the machine learning lifecycle. It provides a suite of tools to track experiments, package models, and deploy them:

> **Tracking:** Allows logging and querying of experiments, tracking various parameters, metrics, and artifacts.

> **Models:** Offers a standard format for packaging and sharing models.

> **Model Registry:** Provides versioning, annotations, and stage transitions for models.

> **Projects:** Ensures reproducibility and reusability of ML code.

## MLflow UI and Components
We can see experiment and models thanks to the MLflow UI

## Tracking and Logging Experiments
The session demonstrated how to use MLflow's tracking capabilities to log parameters, metrics, and artifacts during the model training process. Code snippets showed how to:

Start an MLflow run:
 ```py
 mlflow.start_run()
 ```
Log parameters:
```py
mlflow.log_params()
```
Log metrics:
```py
mlflow.log_metric()
```
Save models:
```py
mlflow.pyfunc.save_model()
```

## Model Registry
The MLflow Model Registry allows for central management of models:

> **Registered Models:** Unique models with metadata and versioning.

> **Model Versioning:** Tracks different versions of the same model.

> **Annotations:** Provides a space for descriptions, methodology, and other relevant information.

> **Stage Transitions:** Models can be moved through different stages, such as "Staging" and "Production," ensuring controlled deployment.

## GitLab Integration
GitLab was integrated into the MLOps pipeline, demonstrating how to use GitLab for:

> **Experiment Tracking:** Utilizing GitLab CI/CD pipelines for automating the ML workflow.

> **Model Registry Management:** Leveraging GitLab’s APIs and tools to manage the lifecycle of models.

> **Pipelines:** Implementing automated pipelines for ML tasks such as data processing, model training, and deployment.

## Case MLOps for Political Prediction for Turkey

In this case, I implemented GitLab pipeline and MLflow operations for the previous case ([Political Prediction for Turkey](python-fundamentals.md)). 

I created pipelines with using `.gitlab-ci.yml` as follows:

```bash
stages:
  - train
  - notify_register

variables:
  MLFLOW_TRACKING_URI: "https://gitlab-codecamp24.obss.io/api/v4/projects/120/ml/mlflow/"
  MLFLOW_TRACKING_TOKEN: "glpat-fRs-kyabsrcpnPSZKPD1"

train-candidate:
  stage: train
  image: python:3.10.14-slim
  before_script:
    - apt-get update && apt-get install -y git
    - python -m venv venv
    - source venv/bin/activate
    - pip install --upgrade pip
    - pip install -r requirements.txt
  script:
    - python turkey_election.py
  rules:
    - if: $CI_MERGE_REQUEST_ID
  artifacts:
    paths:
      - metrics_report.md
      - plots/questions.png

notify-and-register:
  stage: notify_register
  image: python:3.10.14-slim
  variables:
    RUN_ID: $RUN_ID 
    MODEL_NAME: $MODEL_NAME 
  before_script:
    - apt-get update && apt-get install -y git
    - python -m venv venv
    - source venv/bin/activate
    - pip install --upgrade pip
    - pip install -r requirements.txt
  script:
    - echo "selam"
    - echo "ben alphan"
    - python register_model.py $RUN_ID $MODEL_NAME
  rules:
    - when: manual  
  needs:
    - train-candidate
  artifacts:
    paths:
      - metrics_report.md
      - plots/questions.png

```

The pipelines was successfully runned:

![alt text](../screenshots/mlops-pipeline.png)

I also create model experiments for each merge requests as follows:

![alt text](../screenshots/mlops-experiments.png)

