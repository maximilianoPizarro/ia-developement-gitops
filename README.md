# ia-developement-gitops
<link rel="icon" href="https://raw.githubusercontent.com/maximilianoPizarro/botpress-helm-chart/main/favicon-152.ico" type="image/x-icon" >
<p align="left">
<img src="https://img.shields.io/badge/redhat-CC0000?style=for-the-badge&logo=redhat&logoColor=white" alt="Redhat">
<img src="https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white" alt="kubernetes">
<img src="https://img.shields.io/badge/helm-0db7ed?style=for-the-badge&logo=helm&logoColor=white" alt="Helm">
<img src="https://img.shields.io/badge/shell_script-%23121011.svg?style=for-the-badge&logo=gnu-bash&logoColor=white" alt="shell">
<a href="https://www.linkedin.com/in/maximiliano-gregorio-pizarro-consultor-it"><img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white" alt="linkedin" /></a>
<a href="https://artifacthub.io/packages/search?repo=n8n"><img src="https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/n8n" alt="n8n" /></a>
<a href="https://artifacthub.io/packages/search?repo=librechat-openshift"><img src="https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/librechat" alt="Artifact Hub" /></a>
<a href="https://artifacthub.io/packages/search?repo=botpress"><img src="https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/botpress" alt="Artifact Hub" /></a>
</p>

<div align="center">
  <img src="https://github.com/maximilianoPizarro/ia-developement-gitops/raw/main/snapshot/gitops.png" width="600"/>
  <img src="https://github.com/maximilianoPizarro/ia-developement-gitops/raw/main/snapshot/gitops2.png" width="600"/>
</div>

This repository provides the setup for deploying the following components on OpenShift using GitOps practices:

- **Operators**: Red Hat Build of Keycloak, Red Hat Developer Hub, Red Hat Dev Spaces, path-operator, Red Hat 3scale and Red Hat apicast
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
You must have cluster-admin privileges to apply these files, as they may create resources at the cluster scope.

Example install all components.

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

Operators are essential for automating the management of complex applications on OpenShift.  
They encapsulate operational knowledge and best practices, enabling seamless installation, upgrades, and maintenance of services.  
In this repository, operators such as Red Hat Build of Keycloak, Red Hat Developer Hub, Red Hat Dev Spaces, path-operator, Red Hat 3scale and Red Hat apicast are used to provide authentication, developer tooling, API management, and routing capabilities.  
Using operators ensures that applications are deployed in a consistent, scalable, and secure manner, reducing manual intervention and operational overhead.

<div align="center">
  <img src="https://github.com/maximilianoPizarro/ia-developement-gitops/raw/main/snapshot/operators.png" width="600"/>
  <img src="https://github.com/maximilianoPizarro/ia-developement-gitops/raw/main/snapshot/rhbk.png" width="600"/>
  <img src="https://github.com/maximilianoPizarro/ia-developement-gitops/raw/main/snapshot/rhbk-2.png" width="600"/>
</div>

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
Example instance Red Hat Build of Keycloak and Red Hat DevSpaces

```
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: ia-development-instance
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


---

## n8n

**n8n** is an extendable workflow automation tool that enables users to connect various services and automate tasks with minimal effort.  
It supports a wide range of integrations, allowing you to orchestrate data flows between APIs, databases, and other platforms.  
n8n is designed for flexibility, enabling both simple automations and complex workflows with conditional logic, loops, and error handling.  
It features a visual editor for building workflows, making it accessible for both technical and non-technical users.  
With n8n, you can automate repetitive tasks, synchronize data between systems, and trigger actions based on events.  
It supports custom scripting and can be self-hosted for full control over your automation environment.  
n8n is ideal for DevOps, data engineering, and business process automation, providing transparency and traceability for all automated processes.  
Its open-source nature allows for community-driven enhancements and integrations.  
Security is a priority, with support for environment variables, encrypted credentials, and role-based access control.  
n8n can be scaled horizontally to handle large volumes of workflows and tasks.  
It integrates seamlessly with OpenShift, leveraging container orchestration for reliability and scalability.  
The deployment in this repository ensures n8n is managed via GitOps, enabling version control and automated rollbacks.  
n8n's modular architecture allows for easy extension with custom nodes and plugins.  
It supports real-time monitoring and logging for workflow execution.  
The platform is suitable for automating notifications, data transformations, and third-party integrations.  
n8n can be used to build chatbots, automate CI/CD pipelines, and synchronize cloud resources.  
Its intuitive interface reduces the learning curve for new users.  
n8n is a key component for modern automation strategies in cloud-native environments.  
It empowers teams to innovate faster by reducing manual work and errors.  
The deployment is accompanied by visual guides and snapshots for easy onboarding.

<div align="center">
  <img src="https://github.com/maximilianoPizarro/ia-developement-gitops/raw/main/snapshot/n8n.png" width="600"/>
</div>

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

**LibreChat** is an open-source chat platform designed for secure, scalable, and customizable messaging experiences.  
It supports real-time communication, group chats, and integrations with external services.  
LibreChat is built with privacy in mind, offering end-to-end encryption and robust access controls.  
The platform is highly extensible, allowing for custom plugins and integrations with AI services.  
LibreChat can be used for team collaboration, customer support, and community engagement.  
It features a modern user interface, supporting multimedia messages, file sharing, and notifications.  
The deployment leverages OpenShift for high availability and scalability.  
LibreChat integrates with authentication providers and databases for seamless user management.  
It supports webhooks and API integrations for automation and third-party connectivity.  
The platform is suitable for both internal and external communication needs.  
LibreChat is managed via GitOps in this repository, ensuring consistent deployments and easy updates.  
Security features include audit logs, role-based permissions, and encrypted storage.  
The application can be customized to match organizational branding and workflows.  
LibreChat supports mobile and desktop clients for broad accessibility.  
It is ideal for organizations seeking a self-hosted, privacy-focused chat solution.  
The deployment process includes database initialization and synchronization steps.  
After the databases are running, you must finish the synchronization and perform a **sync prune + force** operation in ArgoCD to start the application.  
Snapshots and guides are provided to assist with setup and troubleshooting.  
LibreChat empowers teams to communicate efficiently while maintaining control over data and infrastructure.  
Its open-source nature encourages community contributions and rapid innovation.

<div align="center">
  <img src="https://github.com/maximilianoPizarro/ia-developement-gitops/raw/main/snapshot/librechat.png" width="600"/>
</div>

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

**Botpress** is a powerful open-source platform for building, deploying, and managing conversational AI applications.  
It provides a visual flow editor for designing chatbots and virtual assistants with complex logic and integrations.  
Botpress supports natural language understanding, multi-channel deployment, and analytics for conversation tracking.  
The platform is modular, allowing for custom extensions and integrations with external APIs and databases.  
Botpress is suitable for customer support, lead generation, and internal automation use cases.  
It features built-in tools for intent recognition, entity extraction, and dialogue management.  
Botpress can be deployed on-premises or in the cloud, ensuring data privacy and compliance.  
The platform integrates with messaging channels such as web chat, Slack, and Microsoft Teams.  
Botpress supports version control and collaborative development workflows.  
It offers real-time monitoring and debugging tools for chatbot performance.  
The deployment in this repository leverages OpenShift for scalability and reliability.  
Botpress is managed via GitOps, enabling automated updates and rollbacks.  
Security features include authentication, authorization, and encrypted storage.  
Botpress can be extended with custom modules for advanced functionality.  
It supports multilingual bots and rich media interactions.  
The platform is ideal for organizations looking to automate interactions and improve user engagement.  
Botpress provides detailed analytics and reporting for continuous improvement.  
Its open-source nature allows for rapid innovation and community-driven enhancements.  
Botpress empowers teams to deliver personalized, efficient, and scalable conversational experiences.  
Visual guides and snapshots are included to assist with deployment and configuration.

<div align="center">
  <img src="https://github.com/maximilianoPizarro/ia-developement-gitops/raw/main/snapshot/botpress-overview.png" width="600"/>
  <img src="https://github.com/maximilianoPizarro/ia-developement-gitops/raw/main/snapshot/botpress.png" width="600"/>
</div>

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
