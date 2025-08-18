# FSI reference architecture based on C64 and Azure AKS reference architecture

## Introduction

The goal of this project is to design a scalable and secure online banking platform that supports account management, payments, user authentication, and audit logging. The solution should ensure high availability, modular design, and extensibility for future services.

## System context online banking system

<img width="3633" height="1900" alt="diagram-export-8-15-2025-2_36_59-PM" src="https://github.com/user-attachments/assets/1dbd4fb1-23b6-483c-901a-9b5116ba8fcd" />

## Container: Online banking system

<img width="4155" height="3181" alt="diagram-export-8-15-2025-3_18_53-PM" src="https://github.com/user-attachments/assets/e2636ed7-1520-43de-97dc-10fa2e4bcdd1" />


## Component: Online banking system, backend component

<img width="6950" height="3922" alt="diagram-export-8-15-2025-3_57_26-PM" src="https://github.com/user-attachments/assets/49b886a2-ff83-4253-b698-255b87374fb8" />

## Azure technical reference architecture

<img width="5122" height="3024" alt="diagram-export-8-15-2025-1_56_37-PM" src="https://github.com/user-attachments/assets/3f9a9f34-f5b3-491b-9e72-2ae5f1d2bd47" />


### High availability and backup

1. Design for Multi-Region Redundancy
   
![aks-multi-cluster](https://github.com/user-attachments/assets/bf98cce0-50a3-4598-84ae-e1d826554916)

How:

- Multiple AKS cluster deployments in different regions.
- Front Door configuration: we can add both clusters to the same backend pool
- Leverage health probes (HTTP/HTTPS) at the service level (e.g., /healthz (Spring Boot actuator)) so Front Door routes traffic only to healthy clusters.
- It can be considered to build an active-active scenario to minimize latency or an active-passive setup for disaster recovery

2. Zone Redundancy within Each Region

How:

- Leverage AKS with availability zones enabled, which helps eliminate the risk of region outage.
- Node pools should spread across different availability zones.
- Leverage a zone redundant load balancer (Standard SKU) for the AKS microservice layer (Kubernetes Service type).


3. Application-Level Resilience

How:

- Leverage Kubernetes Deployments with replicas spread across zones.

- Enable PodDisruptionBudgets to avoid too many pods going down during maintenance.

- Implement liveness and readiness probes so Front Door sees unhealthy pods and routes traffic elsewhere.

- Store state in redundant services (Azure SQL, Redis with geo-replication).

4. Networking and Failover Mechanisms

How:

- We can expose AKS services via Azure Public Load Balancer (Standard SKU) or Azure Application Gateway Ingress Controller — these become the backend for Front Door.

- Private Link if needed for security, but be aware of failover complexity.

- Keep DNS TTLs low if we need to fallback to DNS-based routing


### Authentication and authorization

https://learn.microsoft.com/en-us/azure/aks/azure-ad-rbac?tabs=portal


### Audit logging requirement

The system needs to be able to log critical business events in a strongly consistent manner.
This involves implementing an append-only audit log, which can be a basis for future audits as well.
For this purpose, the main idea is to leverage Azure Confidential Ledger with the corresponding Java SDK.

https://github.com/Azure/azure-sdk-for-java/tree/azure-security-confidentialledger_1.0.30/sdk/confidentialledger/azure-security-confidentialledger/src/samples


### Compliance requirements

The following compliance requirements can arise based on the initial problem statements:

- PCI DSS
- GDPR
- PSD2
- ISO27001
- SOX (Within US)
