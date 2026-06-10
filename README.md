# 🎛️ Secure Kubernetes Dashboard Deployment Guide

## 📖 Project Overview

This project demonstrates how to deploy and securely access the official Kubernetes Dashboard on a Kubernetes cluster.

Instead of relying on insecure default configurations, this setup implements proper **Role-Based Access Control (RBAC)** by creating a dedicated administrator ServiceAccount and binding it to the `cluster-admin` role.

The dashboard provides a graphical interface for managing and monitoring Kubernetes resources directly from a web browser.

---

# 🏗️ Architecture

```text
Web Browser
      │
      ▼
 NodePort (32022)
      │
      ▼
Kubernetes Dashboard
      │
      ▼
 Kubernetes API Server
      │
      ▼
 Cluster Resources
```

---

# 🚀 Step 1: Deploy Kubernetes Dashboard

Deploy the official Kubernetes Dashboard manifests.

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

Verify Deployment:

```bash
kubectl get pods -n kubernetes-dashboard
```

Expected Output:

```text
dashboard-metrics-scraper-xxxxx   Running
kubernetes-dashboard-xxxxx        Running
```

---

# 🔐 Step 2: Configure Admin User & RBAC

By default, the dashboard has no permissions to access cluster resources.

Create a file named:

```text
dashboard-adminuser.yaml
```

Paste the following content:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user

roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin

subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

Apply the configuration:

```bash
kubectl apply -f dashboard-adminuser.yaml
```

Verify:

```bash
kubectl get sa -n kubernetes-dashboard

kubectl get clusterrolebinding admin-user
```

---

# 🌐 Step 3: Expose Dashboard Using NodePort

By default, the dashboard is accessible only within the cluster.

Expose the Dashboard using NodePort:

```bash
kubectl patch svc kubernetes-dashboard \
-n kubernetes-dashboard \
-p '{"spec":{"type":"NodePort","ports":[{"port":443,"nodePort":32022}]}}'
```

Verify:

```bash
kubectl get svc -n kubernetes-dashboard
```

Expected Output:

```text
kubernetes-dashboard
NodePort
443:32022/TCP
```

> **Note:** Ensure port **32022** is allowed in your AWS Security Group.

---

# 🔓 Step 4: Generate Access Token

Generate a login token for the administrator account:

```bash
kubectl -n kubernetes-dashboard create token admin-user
```

Example Output:

```text
eyJhbGciOiJSUzI1NiIsImtpZCI6...
```

Copy the entire token.

---

# 🖥️ Accessing the Dashboard

Open your browser and navigate to:

```text
https://<EC2-PUBLIC-IP>:32022
```

Login Steps:

1. Select **Token**
2. Paste the generated token
3. Click **Sign In**

You should now have full administrative access to the Kubernetes Dashboard.

---

# 📸 Screenshots

## Dashboard Deployment

<img width="1919" height="1079" alt="1" src="https://github.com/user-attachments/assets/cb95163a-1408-40cb-8337-17126f1274d8" />

---

## Dashboard Pods Running

<img width="1919" height="132" alt="k8swebdash_pods" src="https://github.com/user-attachments/assets/0012788d-a3da-4ea6-9434-29eeac816d03" />


---

## Admin User & RBAC Configuration

<img width="647" height="450" alt="dashboard_admin_user" src="https://github.com/user-attachments/assets/36495355-6f40-41df-8467-28bfed901880" />

---

## NodePort Exposure & Token Generation
<img width="1919" height="237" alt="2" src="https://github.com/user-attachments/assets/5609d05a-d3dc-4392-b253-2ce2d1db4b96" />

---

## Dashboard Login Page

<img width="1919" height="1079" alt="3_accessing_web_dashboard" src="https://github.com/user-attachments/assets/79debffd-0a84-4b3d-ad1f-cb210e187ecd" />


---

## Kubernetes Dashboard Home

<img width="1919" height="1079" alt="4_dashboard" src="https://github.com/user-attachments/assets/e8bc94df-762a-48ff-814a-ed44f4073842" />


---

## Cluster Overview

<img width="1919" height="1079" alt="5" src="https://github.com/user-attachments/assets/84acdbb7-1747-4177-b3ed-e981cca2660a" />

---

# 🚨 Troubleshooting & Common Issues

## Issue 1: Dashboard Not Accessible

### Error

```text
This site can't be reached
```

### Cause

NodePort is not exposed or blocked by the firewall/security group.

### Resolution

Verify Service:

```bash
kubectl get svc -n kubernetes-dashboard
```

Verify Security Group:

```text
Port 32022 must be allowed.
```

---

## Issue 2: Unauthorized Access

### Error

```text
Unauthorized
```

### Cause

Invalid or expired token.

### Resolution

Generate a new token:

```bash
kubectl -n kubernetes-dashboard create token admin-user
```

Login again using the newly generated token.

---

## Issue 3: Dashboard Shows No Resources

### Cause

RBAC permissions are missing.

### Resolution

Verify:

```bash
kubectl get clusterrolebinding admin-user
```

Reapply:

```bash
kubectl apply -f dashboard-adminuser.yaml
```

---

## Issue 4: Your Connection is Not Private

### Error

```text
Your connection is not private
```

### Cause

The Kubernetes Dashboard uses a self-signed SSL certificate.

### Resolution

### Google Chrome

Click anywhere on the page and type:

```text
thisisunsafe
```

The browser will bypass the warning.

### Firefox / Edge

Click:

```text
Advanced
```

Then:

```text
Accept Risk and Continue
```

---

## Issue 5: Token Expired

### Error

```text
Token has expired
```

### Resolution

Generate a new token:

```bash
kubectl -n kubernetes-dashboard create token admin-user
```

---

# 🎯 What I Learned

Through this project, I gained hands-on experience with:

- Kubernetes Dashboard Deployment
- Kubernetes RBAC Configuration
- Service Accounts
- ClusterRole & ClusterRoleBinding
- Secure Dashboard Authentication
- Kubernetes API Access Management
- NodePort Service Exposure
- TLS & Browser Certificate Handling
- Troubleshooting Authentication Issues
- Kubernetes Administration Using Web UI

---
