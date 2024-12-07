# Identity Actor Model Specification

## 1 Introduction

This specification defines the `Identity Actor Model` as a foundational concept within the Zero Trust Auth* (`ZTAuth*`) Framework.
The `ZTAuth*` Framework strictly adheres to and implements the principles of the `ZTAuth*` Architecture.

The goal of this specification is to provide the architecture and implementation details necessary for a `node` to securely act as an `Actor` on behalf of a `Principal`. This ensures secure and controlled operations, fully aligned with the principle of least privilege.

The `Identity Actor Model` is a transformative concept, offering clear guidance for implementing secure operations and inherently directing system architecture to build correct authorization models. These models are designed with Zero Trust Security as the foundational principle, ensuring that permissions and actions are always limited to what is strictly necessary.

Under this framework, the Identity Provider (IDP) can create roles associated with identities, which reference one or more `Identity Actor Models`. When combining `Identity Actor Models`, the system must allow flexibility to specify whether multiple actor models can be assumed simultaneously within a single session, if the actor model is exclusive to a session, or if the behavior reflects a form of intersection between models.

To clarify, roles can associate multiple `Identity Actor Models` using mathematical operations such as Union, Intersection, and Difference. This enables precise and flexible permission assignments, ensuring that every role operates within a clearly defined, auditable, and secure authorization context.

### 1.1 Clarification on Assumptions

`ZTAuth*`assumes that Identity Providers (IdPs) are now a standard and, as such, they are responsible for managing core identity aspects such as Users, Roles, and Groups.

`ZTAuth*`focuses solely on managing the metadata of identities provided by the IdP and implements the entire Authorization (AuthZ) layer.

This approach ensures that `ZTAuth*`remains streamlined and dedicated to authorization, while leveraging the capabilities of Identity Providers to handle authentication and identity management.

### 1.2 Zero Trust Architecture (ZTApp or ZTApplication)

The `ZTAuth*` Architecture is outlined here. For further details, please refer to other specifications, if available, or consult the publications in the [ztauthstar-publications](https://github.com/ztauthstar/ztauthstar-publications) repository.

![`ZTAuth*`Architecture](ztauth-architecture.png "Zero Trust Auth* Architecture")

### 1.3 Fields of Application

 The `Identity Actor Model` is designed for diverse applications across multiple domains, including but not limited to:

- **Cloud Computing**: Providing secure and precise access control to cloud resources and services.
- **Microservices Architecture**: Facilitating secure interactions between microservices while maintaining isolation and trust boundaries.
- **API Management**: Enforcing secure, controlled, and auditable access to APIs.
- **IoT and Edge Computing**: Managing secure communications between devices, gateways, and cloud or edge services.
- **Multi-Tenant Systems**: Ensuring secure collaboration, resource segregation, and isolation between multiple tenants in shared environments.
- **Distributed Systems**: Maintaining secure and controlled operations across geographically or logically distributed components.
- **Federated Systems**: Enabling secure integration and collaboration between entities managed by different organizations or applications.
- **Blockchain and Distributed Ledger Technology (DLT)**: Supporting secure interactions between `nodes`, smart contracts, and decentralized applications.
- **Data Privacy and Compliance**: Allowing secure access and processing of sensitive data while meeting regulatory requirements.
- **DevOps and CI/CD**: Ensuring secure and controlled automation for software deployment and operations pipelines.
- **Machine Learning and AI**: Managing secure interactions between models, training datasets, and inference systems.
- **Digital Transformation**: Supporting secure and scalable operations in modern, evolving IT environments.
- **Digital Wallets**: Providing secure and controlled operations for digital wallets, applicable to both private and public sectors.

We can summarize as following:

    The `Identity Actor Model` applies to any system—whether hardware, software, or a combination of both—where a `node` is responsible for managing requests from a principal. The `node` securely processes these requests by operating within a clearly defined authorization context, performing actions strictly on behalf of the principal while enforcing the necessary security and operational boundaries.

Below are two illustrative examples to help understand the concept. These are purely figurative and not exhaustive, meant only to provide practical context:

- An API receives a request from a client and identifies the principal using the provided Authentication Token. Once the principal is identified, the system elevates to a specific `Actor Model` and performs the action on behalf of the principal. This action may involve various operations, such as sending a message to a message broker like Kafka in a signed and certified manner. Subsequently, a Worker `node`, operating with its own authentication token as a service account, retrieves the message from the message broker. The Worker `node` verifies the message, elevates to the target `Actor Model`, and executes the intended action on behalf of the principal, ensuring secure and controlled operations throughout the process.

```text
# API to Worker Node Communication

    +------+                    +--------+
    |  API |                    | Worker |
    +------+                    +--------+
        |                             ^
        v                             |
    +----------------+             +----------------+
    | Kafka Producer | ----------> | Kafka Consumer |
    +----------------+             +----------------+
                Kafka (Message Exchange)
```

- An AI-powered robot with sensors acts as a field agent and receives commands directly from a client (principal). The robot identifies the principal using an Authentication Token and elevates to a specific `Actor Model`. The elevated context ensures the robot can perform actions securely on behalf of the principal. The robot processes the input, interacts with its onboard devices (e.g., cameras or actuators), and performs tasks such as collecting data or performing physical actions in the field. If a secondary operation is required, such as delegating tasks to another robot or device, the first robot communicates securely with the second device. This communication uses the same principle of verifying messages, elevating to the required `Actor Model`, and performing actions securely and within the bounds of the assigned authorization.

```text
# Client to Robot Communication

+------------+                 +-------------+
|   Client   |                 |    Robot    |
| (Principal)|                 | (AI Agent)  |
+------------+                 +-------------+
       |                             ^
       |                             |
       |                         +----------------+
       |------------------------>| Robot Sensors  |
                                 +----------------+
                           Field Operations Secondary Device
                                           |
                                           v
                                  +----------------+
                                  |  Other Device  |
                                  +----------------+
```

Important Notes:

1. **Authentication Agnosticism**:  
   The principal is recognized by the system using an `Authentication Token` or any other industry-standard authentication mechanism. This approach ensures the system is agnostic to specific authentication implementations, providing flexibility and compatibility across various standards.

2. **`Node` Chaining**:  
   `nodes` can be concatenated without any limit to the number of connections. Each `node` in the chain must independently elevate to the appropriate `Actor Model` and securely perform actions on behalf of the principal.  
   For example, a possible chain could be:  
   `API -> Worker -> API -> API -> Worker -> Robot -> API -> Worker -> API`.

## 2 Registration and Discovery

Defines the processes through which `nodes` either self-advertise or register with a `Central Server`, using predefined keys or dynamic discovery mechanisms.

## 3 Trusted Elevation

Details the secure elevation of a `node` to a specific `Identity Actor`, enabling it to operate on behalf of a principal within the defined security boundaries of an `Authorization Context`.

## 4 Trusted Operation Chaining

Covers the process of securely chaining multiple `nodes`, ensuring that each `node` elevates to the appropriate `Identity Actor` and performs actions on behalf of the principal.  
This process ensures the safe chaining of multiple `nodes`, even in the absence of communication with the `Central Server`.

## 5 Trusted Federation

Describes the secure integration of multiple `nodes` across federated environments, ensuring that each `node` operates within the security boundaries of its `Identity Actor` and maintains secure communication with the relevant `Central Servers`.  
By federated environments, we refer to multiple `Central Servers` that may belong to the same organization/application or to different organizations/applications.

## Appendix A. Terminology

Here is the list of terms used in this document:

- **`Identity`**: A unique representation of a user, service account, or any other entity recognized by the system. An `Identity` is the foundational concept that defines who or what is interacting with the system, often provided or verified by an Identity Provider (IDP).
- **`Principal`**: A specific `Identity` actively engaged in an authenticated session or action within the system. A `Principal` represents the operational context of an `Identity` during its interaction with the system, including its roles, permissions, and any assumed responsibilities.
- **`node`**: A hardware or software component in the system architecture. It receives requests from a principal and performs actions within an authorization context on behalf of that principal.
- **`Central Server`**: A `Central Server` is a server that implements the Zero Trust Application (`ZTApplication`) as defined by the `ZTAuth*` Architecture.
- **`Actor`**: A type of `delegated identity` that a principal can temporarily assume.
- **`Role-Based Actor`**: A representation of a predefined role with specific permissions for a particular task or function.
- **`Digital Twin Actor`**: A virtual representation of an identity that operates independently. It mirrors the behavior and responsibilities of its associated identity, enabling autonomous operations such as scheduled tasks or API interactions.
- **`Zero Trust Elevation (ZT Elevation)`**: The process of dynamically establishing a secure, isolated authorization context for a specific principal. This process leverages the authorization model of the target actor to create a temporary sandbox. The sandbox enforces strict security boundaries, allowing the principal to execute only a predefined set of actions required for a specific task or operation. This ensures adherence to the principle of least privilege, minimizing risk and providing full auditability during execution.
- **`Zero Trust Delegation (ZT Delegation)`**: The resulting secure and temporary authorization context created through the elevation process. This delegated context is tied to a specific principal and derived from the target actor's authorization model. The actor represents the isolated and controlled set of permissions necessary for the operation, ensuring the principal operates within clearly defined, auditable, and temporary boundaries.

## Appendix B. Definitions

### Appendix B.1. Actor Model Definition

An `Actor` is a type of `delegated identity` that can be `temporarily assumed` by a `Principal` (e.g., a user or an entity represented by an Identity Provider - IdP). `Actors` are designed to `encapsulate specific` and `isolated contexts`, allowing `limited` and `purpose-driven permissions` for particular tasks. They can be configured for the following scenarios:

- **Role-Based Actor**: represents a predefined role with specific permissions for a given task or function, such as `Approvals Manager` or `Compliance Reviewer.`

    >Example: A Principal with the `Finance Specialist` role temporarily elevates their permissions to `Approvals Manager` to approve budget requests without gaining broader permissions.

- **Digital Twin Actor**: a virtual representation of a Principal or Service Account that operates independently. A Digital Twin can assume its own delegated identity to perform specific tasks, such as scheduled processes or API operations, reflecting the capabilities of the original Principal.
    > Example: An application uses a Digital Twin configured by the IdP to represent an `Agent` performing periodic compliance checks across multiple tenants.

Key Features and Benefits of the Actor Model:

- **Zero Trust Security**: Temporary elevation to an Actor enables the Principal to operate with the least privileged set of permissions necessary to complete a specific task. This approach minimizes risks, prevents unauthorized privilege escalation, and ensures full auditability of operations.
    > Example: A user elevates to the `Compliance Reviewer` role to access sensitive reports. Once the task is completed, the elevated permissions are automatically revoked.

- **Role Isolation**: Each Actor is strictly confined to its assigned role or context. This isolation guarantees a clear separation of responsibilities and limits access to resources outside the scope of the active role.
    > Example: A Developer Actor cannot access production operations, while a `Production Deployer` Actor is restricted to deployments only.

- **Future Federation**: The Actor model lays the groundwork for integrating permissions and identities across federated environments. Well-defined roles and permissions allow secure collaboration between systems managed by different IdPs while maintaining security boundaries
    > Example: An organization shares a Digital Twin Actor with a partner to access a subset of sensitive data without exposing the full identity of the original Principal.

Advantages for Distributed and Multi-Organization Systems:

- **Security**: Reduces the attack surface through least privilege and controlled elevation.
- **Flexibility**: Supports dynamic roles, digital twins, and integrations with Identity Providers.
- **Scalability**: Suitable for complex and multi-tenant systems, with centralized management of granular permissions.

## Appendix C. Document History

- 01 - Initial version

## Appendix C. Notices

This project is licensed under the [Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)](https://creativecommons.org/licenses/by-sa/4.0/).

> The work is attributed to Nitro Agility Srl as the sole author and responsible entity, except for referenced content, patterns, or other widely recognized knowledge attributed to their respective authors or sources.
> All content, concepts, and implementation were conceived, designed, and executed by Nitro Agility Srl.
> Any disputes or claims regarding this project should be addressed exclusively to Nitro Agility Srl.
> Nitro Agility Srl assumes no responsibility for any errors or omissions in referenced content, which remain the responsibility of their respective authors or sources.

[![License: CC BY-SA 4.0](https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-sa/4.0/)

> The trademark `ZTAuth*` and its associated logo(s), as contained in this repository, are the exclusive intellectual property of Nitro Agility Srl.
> All rights reserved. Unauthorized use, reproduction, or distribution of the `ZTAuth*` trademark and its associated logo(s) is strictly prohibited.
> For inquiries regarding the use of the `ZTAuth*` trademark and its associated logo(s), please contact Nitro Agility S.r.l. at <opensource@nitroagility.com>.
> Copyright © 2024 Nitro Agility S.r.l. All Rights Reserved.

## Appendix D. Authors

Authored by:

- Nicola Gallo, acting solely as the primary creator and ideator on behalf of Nitro Agility Srl.

All contributors to this document, including its current and future authors, act strictly as representatives of Nitro Agility Srl.

Nitro Agility Srl retains full ownership, responsibility, and liability for the content, actions, and implications arising from this work. Under no circumstances shall listed authors or any other contributors, past, present, or future, be held personally liable for any disputes, claims, or liabilities related to this document or its usage. All such matters shall be addressed exclusively to Nitro Agility Srl.
