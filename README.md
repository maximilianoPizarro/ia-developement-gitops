# ia-developement-gitops
<link rel="icon" href="https://raw.githubusercontent.com/maximilianoPizarro/botpress-helm-chart/main/favicon-152.ico" type="image/x-icon" >
<p align="left">
<img src="https://img.shields.io/badge/redhat-CC0000?style=for-the-badge&logo=redhat&logoColor=white" alt="Redhat">
<img src="https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white" alt="kubernetes">
<img src="https://img.shields.io/badge/helm-0db7ed?style=for-the-badge&logo=helm&logoColor=white" alt="Helm">
<a href="https://artifacthub.io/packages/search?org=community-charts"><img src="https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/n8n" alt="n8n" /></a>
<a href="https://artifacthub.io/packages/search?repo=librechat-openshift"><img src="https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/librechat" alt="Artifact Hub" /></a>
<a href="https://artifacthub.io/packages/search?repo=botpress"><img src="https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/botpress" alt="Artifact Hub" /></a>
<a href="https://www.linkedin.com/in/maximiliano-gregorio-pizarro-consultor-it"><img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white" alt="linkedin" /></a>
</p>

<div align="center">
  <img src="https://github.com/maximilianoPizarro/ia-developement-gitops/raw/main/snapshot/gitops.png" width="900"/>
</div>

This repository provides the setup for deploying the following components on OpenShift using GitOps practices:

- **Operators**: Red Hat Build of Keycloak, Red Hat Developer Hub, Red Hat Dev Spaces, path-operator, Red Hat 3scale and Red Hat Apicast
- **n8n**
- **LibreChat**
- **Botpress**

---

## Prerequisites

- An OpenShift cluster version **4.18 or higher**
- **OpenShift GitOps** (ArgoCD) installed to instantiate the `applicationset.yaml` and `applicationset-instance.yaml` files
- **Cluster-admin permissions** are required to instantiate these ApplicationSets

---

## ApplicationSet Files

`applicationset.yaml` and `applicationset-instance.yaml` define the GitOps deployment logic for the components listed above.  
These files are used by OpenShift GitOps to automate the creation and management of ArgoCD Applications.  
You must have **cluster-admin** privileges to apply these files, as they may create resources at the cluster scope.

Example install **all** components.

`applicationset.yaml`

```
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: ia-development
  namespace: openshift-gitops
spec:
  generators:
    - list:
        elements:
          - name: librechat
            namespace: librechat
            path: librechat
          - name: n8n
            namespace: n8n
            path: n8n
          - name: botpress
            namespace: botpress
            path: botpress
          - name: operators
            namespace: openshift-gitops
            path: operators
  template:
    metadata:
      name: '{{name}}'
    spec:
      project: default
      source:
        repoURL: 'https://github.com/maximilianoPizarro/ia-developement-gitops.git'
        targetRevision: main
        path: '{{path}}'
        helm:
          valueFiles:
            - helm-values.yaml
      destination:
        server: 'https://kubernetes.default.svc'
        namespace: '{{namespace}}'
      syncPolicy:
        automated:
          selfHeal: true
          prune: true
        syncOptions:
          - CreateNamespace=true
          - PruneLast=true
```

---

## Operators

<div align="left">
  <img src="https://github.com/maximilianoPizarro/ia-developement-gitops/raw/main/snapshot/operators.png" width="900"/>
</div>

### Automated Management with OpenShift Operators
Operators are a key component for automating the management of complex applications on OpenShift. They encapsulate the operational knowledge needed to install, upgrade, and maintain services seamlessly, which significantly reduces manual intervention and operational overhead.

In this repository, several operators are used to provide essential functionalities:

- Red Hat Build of Keycloak: For robust authentication.

- Red Hat Developer Hub and Red Hat Dev Spaces: For developer tooling.

- Red Hat 3scale and Red Hat Apicast: For API management.

- Path-operator: For intelligent routing.

By leveraging these operators, we ensure that all applications are deployed in a consistent, scalable, and secure manner.

`applicationset.yaml`

```
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: ia-development-operators
  namespace: openshift-gitops
spec:
  generators:
    - list:
        elements:
          - name: operators
            namespace: openshift-gitops
            path: operators
  template:
    metadata:
      name: '{{name}}'
    spec:
      project: default
      source:
        repoURL: 'https://github.com/maximilianoPizarro/ia-developement-gitops.git'
        targetRevision: main
        path: '{{path}}'
        helm:
          valueFiles:
            - helm-values.yaml
      destination:
        server: 'https://kubernetes.default.svc'
        namespace: '{{namespace}}'
      syncPolicy:
        automated:
          selfHeal: true
          prune: true
        syncOptions:
          - CreateNamespace=true
          - PruneLast=true
```
### Example instance Red Hat Build of Keycloak and Red Hat Developer Hub with GitHub OAuth provider

> **Recommendation:**  
> Before proceeding with the instantiation steps, it is highly recommended to **fork this repository** into your own GitHub account. This allows you to customize configurations, manage your own deployment history, and safely update values (such as secrets and hostnames) according to

## Configuring OAuth for Developer Hub

To enable GitHub OAuth authentication in Red Hat Developer Hub, you must first configure an OAuth application in GitHub and set the credentials in a Kubernetes secret before instantiating Developer Hub.

### Steps to Configure GitHub OAuth

1. **Create a GitHub OAuth App**  
   - Go to your GitHub account settings → Developer settings → OAuth Apps.
   - Click "New OAuth App".
   - Set the application name and homepage URL.
   - For "Authorization callback URL", use:  
     `https://<your-developer-hub-route>/api/auth/github/callback`
   - After creation, note the **Client ID** and **Client Secret**.

2. **Create the Kubernetes Secret**

Below is an example of the `secrets-rhdh.yaml` file required for Developer Hub.  
**It is recommended to update these values directly from the OpenShift Console UI for better security and usability, rather than using `oc` commands.**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secrets-rhdh
  namespace: developer-hub
type: Opaque
data:
  OLLAMA_TOKEN: c3NoLWVkMjU1MTkgQUFBQUMzTnphQzFsWkRJMU5URTVBQUFBSURFb2EyYmlwZnNhelRyeGxyR1NrMTBvYUE1cFRUN0EwbFBab3VwaTV6UVc=
  KEYCLOAK_CLIENT_ID: YmFja3N0YWdl
  KEYCLOAK_REALM: YmFja3N0YWdl
...
```

<div align="left">
  <img src="https://github.com/maximilianoPizarro/ia-developement-gitops/raw/main/snapshot/operators.png" width="900"/>
</div>

**Important:**  
- The secret must be created in the `developer-hub` namespace.
- You should update the values (especially tokens, client IDs, secrets, and URLs) to match your environment and credentials.
- Use the OpenShift Console UI to edit and manage secret values securely.


3. **Set the Hostname for Keycloak Integration**  
   - Update the `hostname` field in your `keycloak.yaml` to point to the Red Hat Build of Keycloak route.
   - Example:
     ```
     spec:
       hostname: <your-keycloak-route>
     ```
   - This ensures Developer Hub can authenticate users via Keycloak.

**Note:**  
Both the GitHub OAuth secret and the correct Keycloak hostname are required prerequisites for a successful Developer Hub deployment.

`applicationset-instance.yaml`

```
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: ia-development-instance-operators
  namespace: openshift-gitops
spec:
  generators:
    - list:
        elements:
          - name: rhbk
            namespace: rhbk-operator
            path: rhbk
          - name: developer-hub
            namespace: developer-hub
            path: developer-hub
  template:
    metadata:
      name: '{{name}}'
    spec:
      project: default
      source:
        repoURL: 'https://github.com/maximilianoPizarro/ia-developement-gitops.git'
        targetRevision: main
        path: '{{path}}'
        kustomize:
          namePrefix: '' 
          nameSuffix: '' 
      destination:
        server: 'https://kubernetes.default.svc'
        namespace: '{{namespace}}'
      syncPolicy:
        automated:
          selfHeal: true
          prune: true
        syncOptions:
          - CreateNamespace=true
          - PruneLast=true
```
### Red Hat Build of Keycloak 26.0

<div align="left">
  <img src="https://github.com/maximilianoPizarro/ia-developement-gitops/raw/main/snapshot/rhbk.png" width="900"/>
  <img src="https://github.com/maximilianoPizarro/ia-developement-gitops/raw/main/snapshot/rhbk-2.png" width="900"/>
</div>

### Red Hat Developer Hub 1.7

<div align="left">
  <img src="https://github.com/maximilianoPizarro/ia-developement-gitops/raw/main/snapshot/developer-hub.png" width="900"/>
  <img src="https://github.com/maximilianoPizarro/ia-developement-gitops/raw/main/snapshot/developer-hub2.png" width="900"/>
</div>


---

## n8n

<div align="left">
  <img src="https://github.com/maximilianoPizarro/ia-developement-gitops/raw/main/snapshot/n8n.png" width="900"/>
</div>

### n8n: A Powerful Workflow Automation Tool
**n8n** is an extendable and flexible workflow automation tool that helps you connect various services and automate tasks with minimal effort. It features a user-friendly visual editor that makes it accessible for both technical and non-technical users to build everything from simple automations to complex workflows with conditional logic, loops, and error handling.

### Key Features and Use Cases
With n8n, you can orchestrate data flows between hundreds of APIs, databases, and other platforms. It's perfect for automating repetitive tasks, synchronizing data between systems, and triggering actions based on specific events. Its versatility makes it a valuable asset for a wide range of use cases, including DevOps, data engineering, and general business process automation.

As an open-source platform, n8n supports custom scripting, custom nodes, and community-driven enhancements, empowering teams to innovate faster by reducing manual work and errors.

### Scalability and Secure Deployment
For full control over your automation environment, n8n can be self-hosted and scaled horizontally to handle large volumes of workflows. This deployment integrates seamlessly with OpenShift for container orchestration, ensuring reliability and scalability.

Managed via GitOps, the setup provides version control and automated rollbacks. Security is also a priority, with support for environment variables, encrypted credentials, and role-based access control. Real-time monitoring and logging ensure transparency and traceability for all your automated processes.

<div align="left">
  <img src="https://github.com/maximilianoPizarro/ia-developement-gitops/raw/main/snapshot/gitops2.png" width="900"/>
</div>


`applicationset.yaml`


```
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: ia-development-n8n
  namespace: openshift-gitops
spec:
  generators:
    - list:
        elements:
          - name: n8n
            namespace: n8n
            path: n8n
  template:
    metadata:
      name: '{{name}}'
    spec:
      project: default
      source:
        repoURL: 'https://github.com/maximilianoPizarro/ia-developement-gitops.git'
        targetRevision: main
        path: '{{path}}'
        helm:
          valueFiles:
            - helm-values.yaml
      destination:
        server: 'https://kubernetes.default.svc'
        namespace: '{{namespace}}'
      syncPolicy:
        automated:
          selfHeal: true
          prune: true
        syncOptions:
          - CreateNamespace=true
          - PruneLast=true
```


---

## LibreChat

<div align="left">
  <img src="https://github.com/maximilianoPizarro/ia-developement-gitops/raw/main/snapshot/librechat.png" width="900"/>
</div>

### LibreChat: A Secure, Open-Source Chat Platform
LibreChat is a robust, open-source chat platform built for secure, scalable, and customizable messaging. Designed with privacy in mind, it offers end-to-end encryption and strong access controls, making it an ideal choice for organizations seeking a self-hosted communication solution.

### Core Features and Flexibility
The platform supports a wide range of features, including real-time communication, group chats, and multimedia sharing. Its modern user interface makes it easy for teams to collaborate, share files, and receive notifications across both web and mobile clients.

LibreChat is also highly extensible, allowing for custom plugins and seamless integration with AI services and external APIs via webhooks. This flexibility makes it suitable for diverse use cases, from team collaboration and customer support to community engagement. The platform can be customized with organizational branding and workflows to fit your specific needs.

### Scalable and Secure Deployment
For high availability and seamless management, this deployment leverages OpenShift and is managed via GitOps. This ensures consistent deployments and easy updates. Security features like audit logs, role-based permissions, and encrypted storage provide peace of mind.

The deployment process includes specific steps for database initialization and synchronization. After the databases are running, a sync **prune** + **dry run** + **force** and **refresh** operation in ArgoCD is required to start the application for first time. Guides and snapshots are provided to assist with this setup, empowering teams to maintain full control over their data and infrastructure.

<div align="left">
  <img src="https://github.com/maximilianoPizarro/ia-developement-gitops/raw/main/snapshot/gitops-sync.png" width="600"/>
</div>

`applicationset.yaml`

```
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: ia-development-librechat
  namespace: openshift-gitops
spec:
  generators:
    - list:
        elements:
          - name: librechat
            namespace: librechat
            path: librechat
  template:
    metadata:
      name: '{{name}}'
    spec:
      project: default
      source:
        repoURL: 'https://github.com/maximilianoPizarro/ia-developement-gitops.git'
        targetRevision: main
        path: '{{path}}'
        helm:
          valueFiles:
            - helm-values.yaml
      destination:
        server: 'https://kubernetes.default.svc'
        namespace: '{{namespace}}'
      syncPolicy:
        automated:
          selfHeal: true
          prune: true
        syncOptions:
          - CreateNamespace=true
          - PruneLast=true
```


---

## Botpress

<div align="left">
  <img src="https://github.com/maximilianoPizarro/ia-developement-gitops/raw/main/snapshot/botpress.png" width="900"/>
</div>

### Botpress: A Powerful Conversational AI Platform
Botpress is a robust, open-source platform designed to help teams build, deploy, and manage conversational AI applications. It provides a visual flow editor that simplifies the creation of chatbots and virtual assistants, even with complex logic and integrations.

### Core Features and Flexibility
The platform features built-in tools for natural language understanding (NLU), intent recognition, and dialogue management, allowing for powerful and dynamic conversations. Botpress supports multi-channel deployment, integrating with popular messaging platforms like web chat, Slack, and Microsoft Teams.

Its modular architecture allows for custom extensions and integrations with external APIs and databases, ensuring it can adapt to any use case, from customer support and lead generation to internal automation. The platform also supports multilingual bots and rich media interactions, empowering teams to deliver personalized and engaging user experiences.

### Scalability and Secure Deployment
Designed for scalability and reliability, Botpress can be deployed on-premises or in the cloud. This particular repository leverages OpenShift for automated, scalable deployments managed via GitOps, enabling automated updates and rollbacks. With features like authentication, authorization, and encrypted storage, Botpress ensures data privacy and security.

For continuous improvement, the platform offers real-time monitoring, debugging tools, and detailed analytics for tracking chatbot performance and conversations.

`applicationset.yaml`

```
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: ia-development-botpress
  namespace: openshift-gitops
spec:
  generators:
    - list:
        elements:
          - name: botpress
            namespace: botpress
            path: botpress
  template:
    metadata:
      name: '{{name}}'
    spec:
      project: default
      source:
        repoURL: 'https://github.com/maximilianoPizarro/ia-developement-gitops.git'
        targetRevision: main
        path: '{{path}}'
        helm:
          valueFiles:
            - helm-values.yaml
      destination:
        server: 'https://kubernetes.default.svc'
        namespace: '{{namespace}}'
      syncPolicy:
        automated:
          selfHeal: true
          prune: true
        syncOptions:
          - CreateNamespace=true
          - PruneLast=true
```

---
For more details on each component, refer to their respective vendor documentation.
Build Here Go Anywhere.


