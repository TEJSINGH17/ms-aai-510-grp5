# 📈 Term Deposit Subscription Prediction Using Machine Learning

This project uses machine learning to predict whether a bank customer will subscribe to a term deposit. The objective is to improve the efficiency of marketing campaigns by identifying high-potential customers ahead of time, reducing cost, and improving conversion rates.

---

## Project Overview

- **Dataset**: Bank Marketing Dataset (UCI)
- **Goal**: Predict customer response (`yes`/`no`) to a term deposit offer
- **Techniques**:
  - KMeans Clustering for behavioral segmentation
  - XGBoost Classification for final prediction
  - SMOTE for class imbalance correction
  - SHAP for model explainability
- **Performance**: ~95.3% Accuracy, Balanced Precision & Recall

---

## ML Pipeline Summary

1. **Data Preprocessing**
   - One-hot encoding
   - Feature scaling (StandardScaler)
   - Missing value imputation (mean)
2. **Unsupervised Learning**
   - KMeans Clustering + PCA (for cluster labeling)
3. **Balancing**
   - SMOTE to handle class imbalance
4. **Modeling**
   - XGBoost Classifier (final model)
   - Evaluation using accuracy, precision, recall, F1-score
5. **Explainability**
   - SHAP values and feature importance
6. **Models**
   - Cluster model (`.joblib`)
   - XGBoost model (`.joblib`)
   - Scaler object (`.joblib`)

---

## Deployment Strategy

This project is deployed using containerized and GitOps principles:

All dependencies are listed in requirements.txt. Example
flask
pandas
numpy
scikit-learn
xgboost
joblib
gunicorn

### 1. **Containerization**
- All model files (`XGBoost`, `KMeans`, `Scaler`) are saved and bundled in a **Docker container**.
- A lightweight **Flask API** is created to:
  - Accept input data
  - Scale and assign clusters
  - Predict using XGBoost
  - Return the result in real-time

```bash
# Example Dockerfile snippet
FROM python:3.10-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]

Build and Push Docker Image
docker build -t your-docker-username/ml-term-deposit-app .
docker push your-docker-username/ml-term-deposit-app

### 2. **Kubernetes Deployment**
Sample deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ml-deposit-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: ml-deposit
  template:
    metadata:
      labels:
        app: ml-deposit
    spec:
      containers:
      - name: ml-container
        image: your-docker-username/ml-term-deposit-app
        ports:
        - containerPort: 5000
Sample service.yaml
apiVersion: v1
kind: Service
metadata:
  name: ml-deposit-service
spec:
  type: LoadBalancer
  selector:
    app: ml-deposit
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
### 3. ** Argo CD Integration**
We use Argo CD for GitOps-style Continuous Deployment:
	•	Argo CD monitors the GitHub repo for changes to Kubernetes YAML files.
	•	On every commit to the main branch (e.g., image tag update), it syncs and redeploys the app automatically.
	•	This ensures CI/CD without manual intervention.
