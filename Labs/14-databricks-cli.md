# 📘 Databricks CLI (`databricks-cli` v0.x) Setup Guide for AWS CloudShell

---

## 🧱 Prerequisites

* AWS CloudShell (Amazon-provided Linux shell)
* Python 3 installed
* A Databricks **workspace URL**
* A Databricks **personal access token (PAT)**

---

## ✅ Step-by-Step Installation

### 1. 🛠️ Create and Activate Python Virtual Environment

```bash
python3 -m venv databricks-env
source databricks-env/bin/activate
```

---

### 2. 📦 Install the Legacy Databricks CLI (`databricks-cli`)

```bash
pip install databricks-cli
```

Check version:

```bash
databricks --version
```

---

### 3. 🔐 Configure CLI with Your Token

Generate a token from your Databricks workspace:

1. Go to your Databricks workspace in the browser
2. Click your username → **User Settings**
3. Go to **Access Tokens** → click **Generate New Token**

Then run in CloudShell:

```bash
databricks configure --token
```

It will prompt:

```
Databricks Host (should begin with https://): https://<your-workspace>.cloud.databricks.com
Token: dapiXXXXXXXXXXXXXXXXXXXXXXXX
```

Your credentials will be saved to `~/.databrickscfg`.

---

## ✅ Test the CLI

```bash
databricks fs ls dbfs:/
```

You should see DBFS contents if authentication was successful.

---

## 🔧 Common CLI Commands

### 📁 DBFS

| Action        | Command                                             |
| ------------- | --------------------------------------------------- |
| List files    | `databricks fs ls dbfs:/path/`                      |
| Upload file   | `databricks fs cp local.txt dbfs:/data/local.txt`   |
| Download file | `databricks fs cp dbfs:/data/local.txt ./local.txt` |
| Delete file   | `databricks fs rm dbfs:/data/local.txt`             |

---

### 📂 Workspace

| Action          | Command                                                                      |
| --------------- | ---------------------------------------------------------------------------- |
| List items      | `databricks workspace ls /Users/<user>`                                      |
| Import notebook | `databricks workspace import ./notebook.py /Users/<user>/notebook -f SOURCE` |
| Export notebook | `databricks workspace export /Users/<user>/notebook ./notebook.py`           |
| Delete notebook | `databricks workspace delete /Users/<user>/notebook`                         |

---

### ⚙️ Jobs

| Action    | Command                                     |
| --------- | ------------------------------------------- |
| List jobs | `databricks jobs list`                      |
| Run job   | `databricks jobs run-now --job-id <job-id>` |

---

### 🔒 Secrets

| Action       | Command                                                |
| ------------ | ------------------------------------------------------ |
| List scopes  | `databricks secrets list-scopes`                       |
| Create scope | `databricks secrets create-scope --scope my-scope`     |
| Add secret   | `databricks secrets put --scope my-scope --key my-key` |

---

## 🧼 Clean Up

When finished:

```bash
deactivate
```

---

## 📁 Optional: `requirements.txt`

If you’re managing a project, include:

```txt
databricks-cli
```

Install with:

```bash
pip install -r requirements.txt
```

---

## ✅ Summary

| Step               | Command                              |
| ------------------ | ------------------------------------ |
| Create virtual env | `python3 -m venv databricks-env`     |
| Activate env       | `source databricks-env/bin/activate` |
| Install CLI        | `pip install databricks-cli`         |
| Configure          | `databricks configure --token`       |
| Test               | `databricks fs ls dbfs:/`            |

