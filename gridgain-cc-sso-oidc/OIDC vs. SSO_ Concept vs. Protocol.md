This is one of the most common questions in identity management. The short answer is **no, they are not the same thing.**  
However, they are very closely related.  
Here is the simplest way to understand the difference:

* **SSO (Single Sign-On)** is a **concept** or a **desired user experience**.  
* **OIDC (OpenID Connect)** is a specific **technical protocol** used to achieve that concept.

Think of it like this: **SSO is the destination; OIDC is the vehicle.**

### ---

**1\. What is SSO (Single Sign-On)?**

SSO is a capability. It is the ability for a user to authenticate once with a central provider and then access multiple, independent applications without having to log in again.

**The Goal of SSO:** To improve user experience (fewer passwords to remember) and security (centralized control over who has access).

**SSO is the "What":** "I want my users to log in once to Google and then be able to open Zoom, Slack, and GridGain without typing their password again."

### **2\. What is OIDC (OpenID Connect)?**

OIDC is an open standard authentication protocol built on top of OAuth 2.0. It allows clients (like GridGain Control Center) to verify the identity of the end-user based on the authentication performed by an Authorization Server (like Okta, Keycloak, or Google).  
\+1

OIDC uses JSON Web Tokens (JWTs), specifically an **ID Token**, to securely communicate user identity information.

**The Goal of OIDC:** To provide a modern, standardized, and secure way for one application to ask another application, "Who is this user?"  
**OIDC is the "How":** "I will use the OIDC standard protocol to securely pass an ID Token from Okta to GridGain so GridGain knows who just logged in."

### **The Key Differences**

| Feature | SSO (Single Sign-On) | OIDC (OpenID Connect) |
| :---- | :---- | :---- |
| **Nature** | A concept, capability, or user experience. | A technical standard and protocol used for authentication. |
| **Focus** | Convenience and centralized access management. | Verifying identity securely using modern web standards (JWT, JSON). |
| **Relationship** | The end goal. | A tool to achieve the goal. |
| **Alternatives** | You can have SSO *without* OIDC. | You can use OIDC *without* achieving full SSO across many apps. |

### **How they work together in your context**

In the GridGain configuration example we just discussed:

1. Your company desires **SSO** so that employees don't need a separate username/password just for GridGain.  
2. To achieve this **SSO** experience, you configured GridGain to act as an **OIDC** client.  
3. GridGain now uses the **OIDC** protocol to talk to your Identity Provider (e.g., Okta).

**Are there other ways to do SSO?**  
Yes. Before OIDC became popular, the primary protocol for corporate SSO was **SAML** (Security Assertion Markup Language). SAML is still widely used, but modern applications prefer OIDC because it is lighter weight and more mobile-friendly.  
\+1

**Summary:** OIDC is currently the most popular and modern way to implement SSO.