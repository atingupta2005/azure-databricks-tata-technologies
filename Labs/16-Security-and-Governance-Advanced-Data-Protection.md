# **Databricks Security & Governance**

## **1. Apply Table- and Column-Level Permissions**

### **Step 1: Create a Table**
```sql
-- Create a sample table
CREATE TABLE hr.employees (
  id INT,
  name STRING,
  email STRING,
  salary DOUBLE,
  ssn STRING
);

-- Insert test data
INSERT INTO hr.employees VALUES
  (1, 'John Doe', 'john@company.com', 75000, '123-45-6789'),
  (2, 'Jane Smith', 'jane@company.com', 85000, '987-65-4321');
```

### **Step 2: Grant Table Permissions**
```sql
-- Allow analysts to read the table
GRANT SELECT ON TABLE hr.employees TO ROLE analyst;

-- Restrict HR from seeing SSN
DENY SELECT (ssn) ON TABLE hr.employees TO ROLE hr_assistant;
```

### **Step 3: Verify Access**
```sql
-- Check permissions
SHOW GRANT ON TABLE hr.employees;
```

---

## **2. Perform Audit Queries**

### **Step 1: Check Recent Queries**
```sql
-- See who accessed the table
SELECT 
  user_identity.email,
  action_name,
  timestamp
FROM system.access.audit
WHERE table_name = 'hr.employees'
ORDER BY timestamp DESC
LIMIT 10;
```

### **Step 2: View Data Lineage**
```sql
-- See where data comes from
SELECT * FROM system.lineage.table_lineage
WHERE downstream_table = 'hr.employees';
```

---

## **3. Secure Data for Compliance**

### **Step 1: Mask Sensitive Data**
```sql
-- Create a masked view
CREATE VIEW hr.employees_masked AS
SELECT 
  id,
  name,
  CONCAT('***', SUBSTRING(email, 4)) AS email,
  '***-**-****' AS ssn
FROM hr.employees;
```

### **Step 2: Test the Masked View**
```sql
-- Regular users see masked data
SELECT * FROM hr.employees_masked;
```

---

## **Verification**
1. Try selecting from `hr.employees` as different roles to test permissions  
2. Run the audit query to see your own accesses  
3. Compare `hr.employees` and `hr.employees_masked` results  