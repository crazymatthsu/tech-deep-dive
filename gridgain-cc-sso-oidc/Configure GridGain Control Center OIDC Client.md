You can configure GridGain Control Center as an OpenID Connect (OIDC) client by modifying the application.yml (or application.properties) file located in the Control Center root directory.  
Here is the configuration guide and a complete example.

### **1\. The Configuration File**

Locate or create the application.yml file in the root directory where you installed GridGain Control Center.

### **2\. Configuration Example (YAML)**

Below is a complete example configuration for a generic OIDC provider (like Keycloak, Okta, or Azure AD).  
Copy this into your application.yml. Replace the {placeholders} with your actual OIDC provider details.

YAML

server:  
  port: 3000

\# GridGain Account Settings  
account:  
  \# Enable internal (username/password) auth alongside OIDC? (Set false to force OIDC)  
  internal:  
    enabled: true  
    
  \# OIDC Configuration  
  oidc:  
    \# Enable Role-Based Access Control via OIDC  
    rbac:  
      enabled: true  
      \# The attribute in your OIDC token that contains the role (e.g., 'groups', 'roles')  
      attributeName: "cc-role"   
      
    \# Optional: Skip the "Create Account" screen if the user exists in IDP  
    skipSignUp: true

\# Spring Security OIDC Configuration  
spring:  
  security:  
    oauth2:  
      client:  
        registration:  
          \# 'myoidc' is the registrationId. You can name this whatever you want (e.g., okta, keycloak, google)  
          myoidc:  
            client-id: "{YOUR\_CLIENT\_ID}"  
            client-secret: "{YOUR\_CLIENT\_SECRET}"  
            \# The authorization grant type is almost always authorization\_code  
            authorization-grant-type: authorization\_code  
            \# The scope usually requires 'openid', 'profile', and 'email'  
            scope:  
              \- openid  
              \- profile  
              \- email  
            \# The redirect URI template must match this pattern exactly  
            redirect-uri: "http://{YOUR\_CC\_HOST}:3000/api/v1/oauth2/login/myoidc"  
              
        provider:  
          \# This must match the registrationId above  
          myoidc:  
            \# The issuer URL (e.g. https://idp.example.com/auth/realms/myrealm)  
            issuer-uri: "{YOUR\_ISSUER\_URI}"  
            \# Required if rbac.enabled is true. Endpoint to fetch user details/roles.  
            user-info-uri: "{YOUR\_USER\_INFO\_URI}"  
            \# Attribute name in the UserInfo response that creates the username  
            user-name-attribute: preferred\_username

### **3\. Key Settings Explained**

| Setting | Description |
| :---- | :---- |
| **registrationId** | The name you give the provider (e.g., myoidc in the example above). **Crucial:** This name becomes part of your Redirect URI. |
| **client-id & client-secret** | Obtained from your Identity Provider (IdP) when you create the client application. |
| **issuer-uri** | The base URL of your IdP. For Keycloak, it is usually .../auth/realms/{realm}. |
| **account.oidc.rbac.enabled** | If true, GridGain will try to map the user's role from the IdP to a Control Center role (Admin or User). |
| **attributeName** | The field in the OIDC token that contains the role. You must ensure your IdP sends a claim (like cc-role) containing values like admin or regular. |

### **4\. Important: The Redirect URI**

When you register the application in your Identity Provider (Okta, Keycloak, etc.), you must whitelist the **Valid Redirect URI**.  
The format for GridGain Control Center is:  
http://{CC\_HOST}:{PORT}/api/v1/oauth2/login/{registrationId}  
Using the example YAML above, if your Control Center is on localhost:3000, the URI is:  
http://localhost:3000/api/v1/oauth2/login/myoidc

### **5\. Next Steps**

1. **Save** the application.yml file.  
2. **Restart** the GridGain Control Center service.  
3. **Verify** by opening the login page; you should now see a "Sign in with {Name}" button.