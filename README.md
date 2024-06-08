# web-management-cloud-env
- This repository contains the architecture and deployment details for managing a web project in a cloud environment. It includes components such as Jenkins for CI/CD, SonarQube for code quality, Docker and Trivy for container management, and Azure services like AKS, Key Vault, and Monitor for comprehensive cloud management and security.
- The web I use to manage is [Ekart](https://github.com/Meraviglioso8/Ekart)
## 1. Architecture

![Picture3](https://github.com/Meraviglioso8/web-management-cloud-env/assets/46748862/f3bedef1-1265-4d20-b1f3-46bc316f757e)

### 1.1 Components

- **Developers**: Write and maintain code for applications.
- **VS Code**: A popular integrated development environment (IDE) used by developers to write and edit code.
- **GitHub**: A platform for version control and collaboration. Developers push their code to GitHub repositories.
- **Jenkins**: A CI/CD tool that automates the building, testing, and deploying of applications. Jenkins pulls code from GitHub and orchestrates the pipeline flow.
- **Snyk**: A security tool integrated into the CI/CD pipeline to scan code for vulnerabilities. Jenkins sends the code to Snyk for scanning.
- **SonarQube**: A tool for continuous inspection of code quality to perform automatic reviews, detecting bugs, code smells, and security vulnerabilities. Jenkins sends code to SonarQube for analysis.
- **Docker**: A platform for developing, shipping, and running applications in containers. Jenkins builds Docker images of the application code.
- **Trivy**: An open-source vulnerability scanner for container images. Jenkins scans the Docker images using Trivy to detect vulnerabilities.
- **Terraform**: An infrastructure as code (IaC) tool used to define and provision infrastructure using configuration files. Manages the infrastructure in the Azure Resource Group.
- **Azure Resource Group**: A container for managing and grouping resources in Azure. Terraform manages the infrastructure within this group.
- **Container Registry**: A service to store and manage container images. The pipeline pushes built Docker images to the Azure Container Registry.
- **Azure Kubernetes Service (AKS)**: A managed Kubernetes service for running containerized applications. Pulls images from the Azure Container Registry and deploys them.
- **Azure Key Vault**: A cloud service for securely storing and managing secrets, keys, and certificates. Manages secrets required by the applications running on AKS.
- **Azure Monitor**: A comprehensive monitoring service that provides metrics and logs for the applications and infrastructure. Creates dashboards for monitoring.
- **Prometheus**: An open-source systems monitoring and alerting toolkit. Collects metrics from the applications running on AKS and sends them to Azure Monitor and Grafana.
- **Grafana**: An open-source platform for monitoring and observability. Creates dashboards for visualizing metrics collected by Prometheus.


## 2. Deployment (1)
![Picture1](https://github.com/Meraviglioso8/web-management-cloud-env/assets/46748862/850fb0f2-9934-4178-a27e-249a9dae858f)


### 2.1 Components

1. **huyen (SSH key)**: For secure VM access, ensuring only authorized logins.
2. **vm-jenkins-and-tools_key (SSH key)**: SSH key for secure access to the Jenkins VM.
3. **vm-jenkins-and-tools (Virtual machine)**: Main VM hosting Jenkins and tools for CI/CD.
4. **akscloud (Container registry)**: Azure registry storing Docker images for CI/CD.
5. **cs11003200186c955ee (Storage account)**: Azure Storage for build artifacts, logs, and data.
6. **vm-jenkins-and-tools_OsDisk (Disk)**: OS disk for the Jenkins VM, storing OS and applications.
7. **vm-jenkins-and-tools645_z1 (Network interface)**: NIC connecting the Jenkins VM to the network.
8. **vm-jenkins-and-tools-nsg (Network security group)**: NSG controlling VM traffic with security rules.
9. **vm-jenkins-and-tools-ip (Public IP address)**: Public IP for internet access to the VM.
10. **vm-jenkins-and-tools-vnet (Virtual network)**: Isolated network environment for the VM.

### 2.2 Interactions and Relationships

- **SSH Keys (huyen and vm-jenkins-and-tools_key)**: Secure access keys for the Jenkins VM.
- **vm-jenkins-and-tools VM**: Central component interacting with:
  1. **Disk (vm-jenkins-and-tools_OsDisk)**: Attached OS disk storing system and apps.
  2. **Network Interface (vm-jenkins-and-tools645_z1)**: NIC managing network traffic.
  3. **Public IP Address (vm-jenkins-and-tools-ip)**: Enables internet access.
  4. **Network Security Group (vm-jenkins-and-tools-nsg)**: Security rules for NIC.
- **Azure Container Registry (akscloud)**: Stores Docker images used by the VM.
- **Storage Account (cs11003200186c955ee)**: Stores build artifacts and logs.
- **Virtual Network (vm-jenkins-and-tools-vnet)**: Secure network environment for the VM.


## 3. Deployment (2) - AKS only

![image](https://github.com/Meraviglioso8/web-management-cloud-env/assets/46748862/470461cb-6007-4e97-8d34-6d6de3bda10e)


### 3.1 Components

1. **defaultazuremonitorworkspace (Azure Monitor Workspace)**
   - **Function**: Central workspace in Azure Monitor where metrics and logs are collected and analyzed. It aggregates data from different sources to provide a unified view of the system's performance and health.
2. **Prometheus Rule Groups (NodeRecordingRulesRuleGroup, NodeAndKubernetesRecordingRules, KubernetesRecordingRules, etc.)**
   - **Function**: Define recording and alerting rules. Recording rules precompute frequently needed queries and store the results as new time series. Alerting rules trigger alerts based on specified conditions.
3. **Prometheus Data Collection Rule (MSCI-southeastasia-akscloud, Prom-southeastasia-akscloud)**
   - **Function**: Define how data is collected from the Prometheus endpoints and sent to the Azure Monitor workspace, ensuring metrics from the Kubernetes cluster are ingested into Azure Monitor.
4. **Prometheus Data Collection Endpoint (Prom-southeastasia-akscloud)**
   - **Function**: Represents the target from which Prometheus scrapes metrics, serving as the data source for the Prometheus rule groups and data collection rules.
5. **Metric Alert Rules (CPU Usage Percentage - akscloud, Memory Working Set Percentage - akscloud)**
   - **Function**: Monitor specific metrics, such as CPU usage and memory working set percentage, and trigger alerts when values exceed defined thresholds, helping to identify potential issues early.
6. **Alert Group (RecommendedAlertRules-Akscloud)**
   - **Function**: Aggregates multiple alert rules and defines actions to be taken when alerts are triggered, such as sending notifications or executing automated remediation scripts.
7. **Grafana (grafana-20240517224939)**
   - **Function**: Azure Managed Grafana is used to visualize metrics and logs collected in Azure Monitor, providing dashboards for monitoring the health and performance of the Kubernetes cluster.
8. **Azure Key Vault (akscloud)**
   - **Function**: Securely stores secrets, keys, and certificates used by applications running on the AKS cluster, ensuring secure access to sensitive information.
9. **AKS Cluster (akscloud)**
   - **Function**: Azure Kubernetes Service (AKS) provides a managed Kubernetes environment, running containerized applications and integrating with various Azure services for monitoring, security, and management.

### 3.2 Interactions and Relationships

- **Prometheus Rule Groups**: Define logic for recording and alerting based on metrics collected from the AKS cluster, sending data to the Azure Monitor workspace for aggregation and analysis.
- **Azure Monitor Workspace**: Collects metrics and logs from Prometheus and other sources, acting as the central repository for monitoring data, enabling analysis and alerting.
- **Data Collection Rules and Endpoints**: Define how data is ingested from Prometheus endpoints into the Azure Monitor workspace, with endpoints providing real-time metrics.
- **Metric Alert Rules**: Continuously monitor critical metrics such as CPU and memory usage, triggering alerts when thresholds are breached to notify administrators or take automated actions.
- **Alert Group**: Consolidates multiple alert rules and specifies actions to be taken when an alert is triggered, ensuring a coordinated response to potential issues.
- **Grafana**: Accesses data from the Azure Monitor workspace to create visual dashboards, providing insights into the performance and health of the Kubernetes cluster.
- **Azure Key Vault**: Stores and manages secrets required by applications in the AKS cluster, ensuring secure access and enhancing security posture.
- **AKS Cluster**: Runs containerized applications, integrating with Azure Monitor and Prometheus for monitoring, and relying on Azure Key Vault for secure secret management.

