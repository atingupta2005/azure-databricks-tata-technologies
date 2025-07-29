# **AWS Databricks MLflow Hands-On**

## **1. Setup Environment & Data Preparation**

```python
# Initialize MLflow with Databricks tracking
import mlflow
from databricks import dbutils

user_email = dbutils.notebook.entry_point.getDbutils().notebook().getContext().userName().get()

username = user_email.split('@')[0] if '@' in user_email else user_email

experiment_path = f"/Users/{username}/iris_experiment"

try:
    mlflow.set_tracking_uri("databricks")
    mlflow.set_experiment("/Users/{your_email}/iris_experiment")
except mlflow.exceptions.MlflowException:
    pass  # Experiment already exists


# Load Iris dataset directly in Databricks
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
import pandas as pd

# Load data
iris = load_iris()
df = pd.DataFrame(iris.data, columns=iris.feature_names)
df['target'] = iris.target

# Split data
X_train, X_test, y_train, y_test = train_test_split(
    iris.data, 
    iris.target, 
    test_size=0.2, 
    random_state=42
)
```

## **2. Train Model with MLflow Tracking**

```python
from sklearn.ensemble import RandomForestClassifier

with mlflow.start_run(run_name="baseline_model"):
    
    # Enable autologging (captures parameters, metrics, and model)
    mlflow.sklearn.autolog()
    
    # Train model
    model = RandomForestClassifier(
        n_estimators=100,
        max_depth=4,
        random_state=42
    )
    model.fit(X_train, y_train)
    
    # Manually log additional metrics
    test_accuracy = model.score(X_test, y_test)
    mlflow.log_metric("test_accuracy", test_accuracy)
    
    # Add custom tags
    mlflow.set_tag("model_type", "RandomForest")
    mlflow.set_tag("data_source", "sklearn_iris_dataset")
    
    print(f"Model training complete. Test accuracy: {test_accuracy:.2f}")
```

## **3. Register Model in Unity Catalog**

```python
# Register the trained model
model_uri = f"runs:/{mlflow.active_run().info.run_id}/model"
registered_model = mlflow.register_model(
    model_uri,
    "catalog.schema.iris_classifier"  # Update with your catalog/schema
)

print(f"Model registered as: {registered_model.name}")
print(f"Version: {registered_model.version}")
```

## **4. Batch Inference with Registered Model**

```python
# Load the registered model
loaded_model = mlflow.pyfunc.load_model(
    f"models:/{registered_model.name}/{registered_model.version}"
)

# Create test DataFrame
test_df = pd.DataFrame(X_test, columns=iris.feature_names)

# Run batch predictions
predictions = loaded_model.predict(test_df)
test_df['predicted_class'] = predictions
test_df['true_class'] = y_test

# Display results
display(test_df)
```

## **5. View Run History & Performance**

### **View in MLflow UI**
1. Go to left sidebar → **Experiments**
2. Select your experiment: `/Users/{your_email}/iris_experiment`
3. Click on specific runs to see:
   - Parameters and metrics
   - Model artifacts
   - Performance charts

### **Programmatic Access**
```python
# Get all runs for the experiment
runs = mlflow.search_runs()
display(runs)

# Get best run
best_run = runs.sort_values("metrics.test_accuracy", ascending=False).iloc[0]
print(f"Best run ID: {best_run.run_id}")
print(f"Best accuracy: {best_run['metrics.test_accuracy']:.2f}")
```

## **6. Cleanup (Optional)**

```python
# Archive old runs if needed
# client = mlflow.tracking.MlflowClient()
# client.delete_run(run_id)
```

## **Key Features Used**
1. **MLflow Tracking**: Automatic logging of parameters and metrics
2. **Model Registry**: Versioned model storage in Unity Catalog
3. **Batch Inference**: Loading models for scoring new data
4. **Experiment UI**: Visual comparison of model performance
