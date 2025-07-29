# **Applying Cluster Access Policies in AWS Databricks**  

## **Scenario**  
Your organization wants to:  
- Restrict **non-admin users** from launching large clusters.  
- Enforce **auto-termination** to avoid unnecessary costs.  
- Ensure only **approved instance types** are used.  

---

## **Step-by-Step Implementation**  

### **1. Access Admin Console**  
- Log in to **AWS Databricks workspace** as an **Admin**.  
- Navigate to:  
  ```
  Workspace URL → Admin Console (⚙️) → Compute → Cluster Policies
  ```  

### **2. Create a Custom Cluster Policy**  
**Example 1: Restrict Cluster Size for Data Analysts**  
- **Policy Name**: `analyst-restricted-policy`  
- **Definition (JSON)**:  
  ```json
  {
    "autotermination_minutes": {
      "type": "fixed",
      "value": 30,
      "hidden": false
    },
    "cluster_type": {
      "type": "fixed",
      "value": "all-purpose",
      "hidden": true
    },
    "num_workers": {
      "type": "range",
      "max": 5,
      "min": 1,
      "default": 2,
      "hidden": false
    },
    "node_type_id": {
      "type": "allowed",
      "values": ["m5.large", "m5.xlarge"],
      "default": "m5.large",
      "hidden": false
    }
  }
  ```  
- **Explanation**:  
  - Limits workers to **1-5 nodes**.  
  - Restricts instance types to **m5.large/m5.xlarge** (AWS-specific).  

**Example 2: High-Cost Policy for Engineers**  
- **Policy Name**: `engineer-high-cost-policy`  
- **Definition**:  
  ```json
  {
    "autotermination_minutes": {
      "type": "fixed",
      "value": 60,
      "hidden": false
    },
    "num_workers": {
      "type": "fixed",
      "value": 0,
      "hidden": true
    },
    "spark_conf.spark.databricks.cluster.profile": {
      "type": "fixed",
      "value": "singleNode",
      "hidden": true
    }
  }
  ```  

### **3. Assign Policies to Groups**  
- Go to **Admin Console → Groups**.  
- Select a group (e.g., `data-analysts`).  
- Under **Cluster Policies**, assign `analyst-restricted-policy`.  

### **4. Test Policy Enforcement**  
1. Log in as a **non-admin user** (e.g., a data analyst).  
2. Try creating a cluster:  
   - Verify **only allowed instance types** appear.  
   - Attempt to exceed **5 workers** → Should fail.  

---

# **Reviewing Audit Logs in AWS Databricks**  

## **Scenario**  
You need to:  
- Track **who created/deleted clusters**.  
- Monitor **notebook changes**.  
- Detect **failed login attempts** (security threat detection).  

---

## **Step-by-Step Implementation**  

### **1. Access Audit Logs**  
- Navigate to:  
  ```
  Admin Console → Audit Logs
  ```  
- AWS Databricks sends logs to **AWS CloudTrail** (for workspace-level events) and **Databricks audit logs** (for detailed actions).  

### **2. Filter Logs for Key Events**  
**Example 1: Detect Unauthorized Cluster Creation**  
- **Filter**: `actionName = "createCluster"`  
- **Sample Log Entry**:  
  ```json
  {
    "serviceName": "clusters",
    "actionName": "createCluster",
    "timestamp": "2024-05-15T14:30:00Z",
    "requestParams": {
      "cluster_name": "unauthorized-large-cluster",
      "num_workers": 20,
      "node_type_id": "m5.4xlarge"
    },
    "userIdentity": {
      "email": "user@example.com"
    }
  }
  ```  
- **Action**: If a user violates policies, **revoke permissions**.  

**Example 2: Monitor Notebook Changes**  
- **Filter**: `actionName = "notebookUpdate"`  
- **Sample Log Entry**:  
  ```json
  {
    "serviceName": "notebooks",
    "actionName": "notebookUpdate",
    "notebookId": "123456",
    "userIdentity": {
      "email": "developer@example.com"
    },
    "timestamp": "2024-05-15T15:45:00Z"
  }
  ```  
