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
# **Audit Logs in AWS Databricks**
### **1. Using AWS CloudTrail for Workspace-Level Events**
1. **Access CloudTrail**:
   - Go to **AWS Console → CloudTrail → Event history**
   - Filter for `Event source = databricks.amazonaws.com`

2. **Key Event Types**:
   ```bash
   # Cluster management
   CreateCluster
   DeleteCluster
   EditCluster

   # Notebook operations
   CreateNotebook
   DeleteNotebook
   ```

3. **Sample CloudTrail Entry**:
   ```json
   {
     "eventTime": "2024-05-20T10:15:30Z",
     "eventSource": "databricks.amazonaws.com",
     "eventName": "CreateCluster",
     "userIdentity": {
       "arn": "arn:aws:iam::123456789012:user/john.doe"
     },
     "requestParameters": {
       "clusterName": "production-job-cluster"
     }
   }
   ```

