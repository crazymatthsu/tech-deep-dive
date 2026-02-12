This setup involves three main layers: **Microsoft Entra ID** (formerly Azure AD) as the Identity Provider, **Kubernetes (EKS)** for infrastructure, and the **GridGain Control Center** configuration itself.  
Since you are running in a Pod, you cannot simply "edit a file." You must inject the configuration using a Kubernetes **Secret** or **ConfigMap**.  
Here is the step-by-step guide to setting up OIDC with Microsoft AD Groups.

### **Phase 1: Configure Microsoft Entra ID (Azure AD)**

Since "Microsoft AD" usually implies **Entra ID** for OIDC, you need to configure an Enterprise Application.

1. **Register the App:**  
   * Go to **Entra ID \> App registrations \> New registration**.  
   * **Name:** GridGain Control Center.  
   * **Redirect URI (Web):** https://{YOUR\_EKS\_LOADBALANCER\_DNS}/api/v1/oauth2/login/azure  
   * *Note: You must use the external DNS of your EKS LoadBalancer/Ingress, not an internal IP.*  
2. **Create a Client Secret:**  
   * Go to **Certificates & secrets \> Client secrets**. Create a new one and copy the **Value** (not the ID).  
3. **Configure Group Claims (Crucial Step):**  
   * GridGain needs to know if a user is an Admin.  
   * **The Problem:** Azure AD sends Group *Object IDs* (GUIDs) (e.g., a1b2-c3d4...) in the token, not the group names (e.g., GridGainAdmins), by default.  
   * **The Fix:** Go to **Token configuration \> Add groups claim**. Select "Security groups" and choose to emit them in the **ID** token.  
   * **Note the Object ID:** Find your target AD Group (e.g., "GG\_Admins") in Azure and copy its **Object ID**. You will need this for the config.

### ---

**Phase 2: Create the Kubernetes Configuration**

In EKS, the best practice is to store sensitive configs (like Client Secrets) in a Kubernetes **Secret** and mount it as the application.yml.

#### **1\. Prepare your application.yml content**

Save this locally as application.yml. Note the specific Azure settings.

YAML

server:  
  port: 8080 \# Standard container port

account:  
  internal:  
    enabled: false \# Force OIDC login  
  oidc:  
    rbac:  
      enabled: true  
      \# Azure sends group memberships in the 'groups' claim  
      attributeName: "groups"  
      \# If the 'groups' list contains this Object ID, the user becomes a generic User  
      defaultRole: "User"   
      \# MAPPING: Replace this GUID with your actual Azure AD Group Object ID  
      adminRole: "11111111-2222-3333-4444-555555555555" 

spring:  
  security:  
    oauth2:  
      client:  
        registration:  
          \# Registration ID 'azure' matches the Redirect URI suffix  
          azure:  
            provider: azure-ad  
            client-id: "{YOUR\_AZURE\_CLIENT\_ID}"  
            client-secret: "{YOUR\_AZURE\_CLIENT\_SECRET}"  
            authorization-grant-type: authorization\_code  
            scope:  
              \- openid  
              \- profile  
              \- email  
            \# MUST match the Redirect URI set in Azure Portal  
            redirect-uri: "https://{YOUR\_EKS\_DNS}/api/v1/oauth2/login/azure"

        provider:  
          azure-ad:  
            \# Replace {TENANT\_ID} with your Azure Directory (Tenant) ID  
            issuer-uri: "https://login.microsoftonline.com/{TENANT\_ID}/v2.0"  
            user-name-attribute: preferred\_username

#### **2\. Create the Kubernetes Secret**

Run this command to upload your config to the cluster:

Bash

kubectl create secret generic gridgain-cc-config \\  
  \--from-file=application.yml=./application.yml \\  
  \-n {YOUR\_NAMESPACE}

### ---

**Phase 3: Deploy to EKS**

Update your Kubernetes Deployment manifest to mount this secret. The Control Center image expects the config at /opt/gridgain-cc/application.yml (path may vary slightly by version, check your specific Dockerfile, but usually, it's the working directory).

YAML

apiVersion: apps/v1  
kind: Deployment  
metadata:  
  name: gridgain-control-center  
spec:  
  replicas: 1  
  selector:  
    matchLabels:  
      app: gridgain-cc  
  template:  
    metadata:  
      labels:  
        app: gridgain-cc  
    spec:  
      containers:  
      \- name: gridgain-cc  
        image: gridgain/control-center:latest  
        ports:  
        \- containerPort: 8080  
        volumeMounts:  
        \# Mount the Secret as a file  
        \- name: config-volume  
          mountPath: /opt/gridgain-cc/application.yml  
          subPath: application.yml  
          readOnly: true  
      volumes:  
      \- name: config-volume  
        secret:  
          secretName: gridgain-cc-config

### **Summary of "Gotchas" for AWS EKS & Azure AD**

1. **The Redirect URI Loop:** If you use a LoadBalancer in front of EKS, ensure it handles SSL termination correctly. Azure AD requires https. If your pod sees http traffic (because the LB terminated SSL), the redirect might break. You may need to set server.forward-headers-strategy: framework in the YAML or configure your Ingress Controller (e.g., NGINX) to pass X-Forwarded-Proto: https.  
2. **Group Object IDs:** Do not put the text name "MyGroup" in the adminRole field unless you have configured Azure AD to emit "sAMAccountName" for groups (which is rare/legacy for cloud apps). Always use the **Object ID (GUID)**.  
3. **The Issuer URI:** Ensure the issuer-uri ends in /v2.0 if you are using modern Entra ID defaults, otherwise token validation might fail.