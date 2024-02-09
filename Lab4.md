# Engineering Design Document: Kubernetes Deployment Architecture for Docker-Ethereum Application

#### Overview

This document outlines the design for deploying a Docker-Ethereum application within a Kubernetes environment, leveraging Minikube for local development and testing. The application consists of three main components: a Ganache Ethereum simulator, a Node.js backend server, and a React frontend.

#### Deployment Options

- **Ganache**: Deployed as a **StatefulSet** to ensure stable storage of blockchain data. While Ganache typically does not require persistent storage for ephemeral development environments, using StatefulSets allows us to explore advanced scenarios, such as persistent testing environments or simulations that require data retention across pod restarts.

- **Node.js Backend (DApp)**: Deployed as a **Deployment** with ReplicaSets. The backend is stateless, handling requests between the frontend and the Ganache blockchain simulator. Scaling is managed via ReplicaSets to accommodate fluctuating loads. However, we nuderstand that we wish to preserve the deployed contract address across restarts and found it easier to config maps
  - **Configuration and Secrets**: Contract addresses can be stored in ConfigMaps for non-sensitive data or Secrets for sensitive data. These Kubernetes objects allow you to externalize configuration and state from your application's containers, making it accessible to pods regardless of their statefulness.

- **React Frontend**: Also deployed as a **Deployment** with ReplicaSets. The frontend serves static content and communicates with the backend, requiring no stateful storage and benefiting from horizontal scaling.

#### Storage

- **Ganache StatefulSet** will be configured with a **PersistentVolumeClaim (PVC)** to ensure that blockchain data persists across pod restarts or failures. The PVC will request a modest amount of storage, given the development nature of Ganache's blockchain.

- **Node.js Backend and React Frontend** do not require persistent storage, as their states are either ephemeral or stored externally (in the blockchain or client-side).

#### Scaling

- **Horizontal Pod Autoscaler (HPA)** will be applied to the backend and frontend deployments. The HPA will scale the number of pods based on CPU utilization, ensuring responsive application performance under varying loads.

#### Load Balancing

- **Service of type LoadBalancer** for the frontend to distribute incoming traffic evenly across frontend pods, ensuring high availability and responsiveness.

- **ClusterIP Services** for the backend and Ganache, facilitating internal communication within the cluster. The frontend accesses the backend through its ClusterIP Service, and the backend communicates with Ganache similarly.

#### Secrets

- **Kubernetes Secrets** will be used to store sensitive information, such as API keys or configuration details required by the application. This ensures that sensitive data is not hard-coded into the application's source code or Docker images.

#### User Roles and Access Control

- **Role-Based Access Control (RBAC)** configurations will define specific roles for deployment and management tasks within the Kubernetes cluster:
  - A **Service Account** for CI/CD pipelines, enabling automated deployment with restricted permissions.
  - **Roles** and **RoleBindings** for developers and operations teams, ensuring access is granted based on the principle of least privilege.

#### Implementation

- **YAML Configuration Files** will be created for each Kubernetes object required by the architecture. This includes the StatefulSet for Ganache, Deployments for the backend and frontend, PVCs, Services, HPA configurations, Secrets, and RBAC rules.

- **Deployment to Minikube** involves applying these configurations to a local Minikube environment, using `kubectl apply -f <filename>.yaml` for each file.

### Conclusion

This design leverages Kubernetes' features to create a scalable, secure, and maintainable deployment for a Docker-Ethereum application. By carefully selecting deployment options, managing storage, and implementing scaling and load balancing, the architecture supports the development and testing of Ethereum applications within a local Kubernetes environment.

Screenshots Showing the working application.container

We used port forwarding for the application to send traffic from one cluster to another. In production scenarios using ingress plus a loadbalancer would be more appropriate.
