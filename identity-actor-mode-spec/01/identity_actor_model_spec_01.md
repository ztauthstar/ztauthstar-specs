# Identity Actor Model Specification

## 1. Introduction

This specification introduces the `Identity Actor Model`, a core component of the Zero Trust Auth* (`ZTAuth*`) Framework.
It implements the `ZTAuth*` Architecture to provide a secure, scalable, and organized way to manage authorization, aligning with Zero Trust principles.

The `Identity Actor Model`, also known as the `Actor Model` or simply `Actor`, enables secure, permission-based operations through policies.
It links policies directly to specific authorization contexts, ensuring systems operate with only the minimum permissions required for efficiency and security.

### 1.1 Purpose

The purpose of this specification is to outline how a `Node` can securely act as an `Actor` on behalf of a `Principal`.
It provides clear guidance for both architecture and implementation to enable permissioned operations that fully align with Zero Trust principles, ensuring secure interactions with strictly bounded permissions.

### 1.2 Key Features

The `Identity Actor Model` enhances user, role and group management with:

- **Direct Link Between Identities and `Actor Models`:** Users, roles and groups are mapped directly to `Actor Models`.

- **Customizable `Actor Models`:** Combine `Actor Models` using Union, Intersection, and Difference to support complex permission scenarios.

These features make the `Identity Actor Model` a secure, efficient, scalable, and flexible solution for modern systems.

### 1.3 Clarification on Assumptions

`ZTAuth*` assumes that Identity Providers (IdPs) are now the standard and are responsible for managing core identity aspects, such as Users, Roles, and Groups, specifically in the context of Authentication (AuthN) and Identity Management.

`ZTAuth*` primarily focuses on **Zero Trust** security, managing the metadata of identities provided by the IdP and implementing the entire Authorization (AuthZ) layer.

This approach ensures that `ZTAuth*` remains focused and dedicated to authorization within a **Zero Trust** framework, while leveraging the capabilities of Identity Providers to handle authentication and identity management.

### 1.4 Zero Trust Architecture (ZTApp or ZTApplication)

The `ZTAuth*` Architecture is outlined in this diagram `https://github.com/ztauthstar/ztauthstar-specs/blob/main/identity-actor-mode-spec/01/ztauth-architecture.png`.

For further details, refer to `https://github.com/ztauthstar/ztauthstar-publications`.

### 1.5 Fields of Application

The `Identity Actor Model` is versatile and applies to a wide range of domains, including, but not limited to:

- **Cloud Computing**: Secure and precise access control for cloud resources and services.
- **Microservices**: Enforcing secure interactions while maintaining isolation.
- **API Management**: Providing controlled and auditable API access.
- **IoT and Edge Computing**: Managing secure device-to-cloud and edge interactions.
- **Multi-Tenant Systems**: Ensuring resource segregation and tenant isolation.
- **Distributed Systems**: Securing operations across distributed components.
- **Federated Systems**: Enabling safe collaboration between independent entities.
- **Blockchain/DLT**: Supporting secure `node`, smart contract, and app interactions.
- **Data Privacy**: Handling sensitive data securely and meeting compliance.
- **DevOps/CI/CD**: Automating secure deployments and operations.
- **AI/ML**: Managing secure access to models and datasets.
- **Digital Transformation**: Scaling secure operations in modern IT.
- **Digital Wallets**: Ensuring secure management and transactions for wallets in public and private sectors.

We can summarize as following:

    The `Identity Actor Model` applies to any system—whether hardware, software, or a combination of both—where a `Node` is responsible for managing requests from a `Principal`. The `Node` securely processes these requests within a clearly defined authorization context, acting as an `Actor` and performing actions strictly on behalf of the `Principal` while enforcing necessary security and operational boundaries.

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

- **Authentication Agnosticism**: The principal can be recognized by the system using an `Authentication Token` or any other industry-standard authentication mechanism. 
  This approach ensures the system is agnostic to specific authentication implementations, providing flexibility and compatibility across various standards.

- **`Node` Chaining**: `Nodes` can be concatenated without any limit to the number of connections. Each `node` in the chain must independently elevate to the appropriate `Actor Model` and securely perform actions on behalf of the principal.  
   For example, a possible chain could be: `API -> Worker -> API -> API -> Worker -> Robot -> API -> Worker -> API`.

### 1.6 Reference Scenario

To explain the concepts in this document, consider an **accounting system** with two business roles:

- **John**: An `accountant` who manages all parts of the invoice process (view, create, update, delete, approve, reject).
- **Bob**: An `apprentice` who can only view invoices and cannot perform other actions.

As Bob gains experience, John sometimes assigns him extra tasks. For example, when John is unavailable, Bob is allowed to either **create or update new invoices**, but these invoices must remain pending until approved by another accountant.

This example demonstrates how responsibilities can be shared in a bounded way, with clear limits to ensure accountability and security.

## 2. Pairing of the `Central Server` and `Nodes`

The `Identity Actor Model` requires a secure and reliable mechanism for `nodes` to pair with the `Central Server`. This process is essential for the `Central Server` to manage the `nodes` and ensure secure communication between them.

### 2.1 Pairing Requirements

The `Central Server` and each `Node` must generate a pair of cryptographic keys (public and private) to be dedicated just for the `Pairing` operations.

These keys are essential for secure communication and authentication within the system. The **public key** will be used for encryption, while the **private key** will be used for decryption and signing operations, ensuring the integrity and confidentiality of the messages exchanged between the `Central Server` and the `Nodes`.

### 2.2 Pairing with Node Registration

The `Node` registration process ensures secure pairing with the `Central Server` before the `Node` becomes fully operational or reachable. This process is initiated by a trusted operator and follows these steps:

- A trusted operator **requests** the `Central Server` to register a new `node` in the system by providing the necessary details, such as the `node` name, description, and the **public key**, which is mandatory. This should be done in a secure environment, such as an administration console, a secure API endpoint, or any other secure procedure, to protect sensitive information during transmission.
- The `Central Server` generates a unique identifier for the `node` and associates it with the provided details.
- The `Node` **contacts** the `Central Server` to **confirm** the registration and **retrieve** the unique identifier certificate and the API key. This request is **signed** with the private key of the `Node` to ensure the authenticity of the request and to prevent tampering.
- The `Central Server` **verifies** the signature of the request.
- **Before responding**, the `Central Server` **saves** the API key securely, ensuring that it is stored in a way that prevents unauthorized access.
- The `Central Server` **sends** the unique identifier certificate along with the generated API key to the `Node`. This response is **encrypted** with the `Node`'s public key to ensure confidentiality and protect the integrity of the sensitive data in transit.

### 2.3 Pairing with Node Advertisement

The `Node` advertisement process ensures that a `Node` is recognized and verified by the `Central Server` before it becomes fully operational or reachable. This process is initiated by the `Node` and follows these steps:

- The `Node` **advertises** its presence to the `Central Server` by providing necessary details such as the `node` name, description, and the **public key**, which is mandatory. This should happen in a secure environment, such as a secure API endpoint, to protect sensitive information during transmission.  
- The `Central Server` **receives** the advertisement request and generates a unique identifier for the `node`, associating it with the provided details.
- The `Central Server` then performs a verification step, which can be manual or automatic, potentially using a third-party provider to **validate** the authenticity and legitimacy of the `Node`. This ensures the `Node` is trustworthy and can be safely integrated into the system.
- Once the verification is successful, the `Central Server` **saves** the API key securely, ensuring that it is stored in a way that prevents unauthorized access.
- The `Central Server` **sends** the unique identifier certificate along with the generated API key to the `Node`. This response is **encrypted** with the `Node`'s public key to ensure confidentiality and protect the integrity of the sensitive data in transit.
- The `Node` can now use the API key and the unique identifier to become fully operational and interact securely with the `Central Server` and other nodes in the network.

### 2.4 Identifier Certificate

The `Identifier Certificate` is a unique certificate generated by the `Central Server` for each `Node`. This certificate contains the following information:

- **Node Identifier**: A unique identifier assigned to the `Node` by the `Central Server`.
- **Node Name**: The name of the `Node` provided during registration or advertisement.
- **Node Description**: A brief description of the `Node` provided during registration or advertisement.
- **Public Key**: The public key of the `Node` used for encryption and verification operations.
- **Creation Timestamp**: The timestamp indicating when the certificate was created.
- **Expiration Timestamp**: The timestamp indicating when the certificate will expire.
- **Signature**: A digital signature generated by the `Central Server` to ensure the authenticity and integrity of the certificate.

> Essentially, the `Central Server` acts as the Certificate Authority (CA) for the `Node`, signing the certificate to guarantee its authenticity and integrity.

Here an example of the `Identifier Certificate` structure:

```json
{
    "certificate_version": "1.0",
    "certificate_type": "node_identifier",
    "certificate_id": "03933209-d244-490c-83df-ffad43f20c67",
    "node_identifier": "0bc91b6c-fa45-4f22-bc18-a148157f6e75",
    "node_name": "Node1",
    "node_description": "Node dedicated for API operations",
    "public_key": "node_public_key",
    "creation_timestamp": "2024-01-01T00:00:00Z",
    "expiration_timestamp": "2024-01-01T00:00:00Z",
    "signature": "certificate_signature"
}
```

> **NOTE**: The `Identifier Certificate` structure is provided as an example and can be customized based on the specific requirements of the system, likewise node_public_key and certificate_signature are placeholders for the actual public key and signature.

### 2.5 Revocation of Pairing

In case of a security incident or a change in the `Node` status, the `Central Server` can revoke the pairing with a `Node`. This ensures that the `Node` is no longer recognized or trusted by the `Central Server` and cannot interact with the system.

The **API Key** is used only for communication with the `Central Server`, not for `Node` to `Node` communication. Therefore, deleting the **API Key** is sufficient to block the `Node` from the system.

Regarding the `Identifier Certificate`, the `Central Server` can revoke it by updating the `Node` status in the system, marking it as revoked. This ensures that the `Node` is no longer considered trustworthy and cannot perform any operations. The revoked status should be tracked in a **revoked nodes list**. Each `Node` should regularly check this list to verify the trustworthiness of other `Nodes` it communicates with.

## 3 Trusted Elevation

The `Trusted Elevation` process is a critical component of the `Identity Actor Model`. It ensures that a `Node` can securely elevate its privileges to a specific `Identity Actor` and act on behalf of a `Principal` within the defined security boundaries of an `Authorization Context`.

### 3.1 Elevation Requirements

The `Node` must be authenticated with its `Service Account` to the Identity Provider (IdP).

### 3.2 Synchronization of Trusted Elevation

The `Node` must synchronize its `Trusted Elevation` list by sending the Authentication Token and the **API Key** to the `Central Server`.

The `Central Server` will verify the `Service Account` and the `Node`, then send back the list of valid `Trusted Elevations`.

The list contains `Identity Actors` to which the `Node` is authorized to elevate. Each `Trusted Elevation` entry is signed by the `Central Server` to ensure the authenticity and integrity of the list.

Here is an example of the `Trusted Elevation List` structure:

```json
{
  "certificate_version": "1.0",
  "certificate_type": "trusted_elevation_list",
  "certificate_id": "03933209-d244-490c-83df-ffad43f20c67",
  "node_identifier": "0bc91b6c-fa45-4f22-bc18-a148157f6e75",
  "node_name": "Node1",
  "trusted_elevations": [
    {
      "identity_actor": "admin_actor",
      "description": "Administrator privileges to manage sensitive data.",
      "valid_from": "2024-01-01T00:00:00Z",
      "valid_until": "2025-01-01T00:00:00Z",
      "signature": "server_signature_for_admin_actor"
    },
    {
      "identity_actor": "monitor_actor",
      "valid_from": "2024-01-01T00:00:00Z",
      "valid_until": "2025-01-01T00:00:00Z",
      "signature": "server_signature_for_monitor_actor"
    }
  ],
  "creation_timestamp": "2024-01-01T00:00:00Z",
  "expiration_timestamp": "2025-01-01T00:00:00Z",
  "signature": "list_signature"
}
```

> **NOTE**: The `Trusted Elevation List` structure is provided as an example and can be customized based on the specific requirements of the system. Additionally, `list_signature` and `server_signature_for_admin_actor` are placeholders for the actual public key and signature.

### 3.3 Trusted Elevation Process

The `Client`, before sending the principal, can optionally ask the `Node` if it is able to elevate to a specific `Identity Actor`. The `Client` receives the certificate and validates it.

Once the `Client` sends the request to the `Node`, it will load the `Authorization Context` from the `Authorization Model` and elevate to the specified `Identity Actor` in the request.

The `Authorization Model` contains all the policies and data needed by the `Node` to perform the operation and build the `Authorization Context`.

## 4 Trusted Operation Chaining

This section covers the process of securely chaining multiple `nodes`, ensuring that each `node` elevates to the appropriate `Identity Actor` and performs actions on behalf of the principal.
This process ensures the safe chaining of multiple `nodes`, even in the absence of communication with the `Central Server`.

In practice, this works by forwarding requests from one `node` to another. Each `node` presents its certificate as proof that it received a request from a client. The certificate is signed with the private key of the `node`, and this signature can be verified by the next `node` in the chain, ensuring the authenticity and integrity of the request.

## 5 Trusted Federation

Trusted Federation refers to the secure integration of multiple `nodes` across federated environments, ensuring that each `node` operates within the security boundaries of its `Identity Actor` and maintains secure communication with the relevant `Central Servers`.

Federated environments consist of multiple `Central Servers` that may belong to the same organization/application or different organizations/applications.

In the context of operation chaining, the multiple `Central Servers` exchange their public keys and verify the authenticity of the `nodes` that are part of the operation chain request. This ensures that each `node` in the chain is trusted and authorized to perform the requested operations.

## Appendix A. Terminology

Here is the list of terms used in this document:

- **`Identity`**: A unique representation of a user, service account, or any other entity recognized by the system. An `Identity` is the foundational concept that defines who or what is interacting with the system, often provided or verified by an Identity Provider (IdP).
- **`Principal`**: A specific `Identity` actively engaged in an authenticated session or action within the system. A `Principal` represents the operational context of an `Identity` during its interaction with the system, including its roles, permissions, and any assumed responsibilities.
- **`Node`**: A hardware or software component in the system architecture. It receives requests from a principal and performs actions within an authorization context on behalf of that principal.
- **`API Key`**: The **API Key** consists of a unique **ID** and a **secret**; the **secret** is hashed before being stored to enhance security, ensuring that sensitive data is protected even if the database is compromised.
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

> All contributors to this document, including its current and future authors, act strictly as representatives of Nitro Agility Srl.
> Nitro Agility Srl retains full ownership, responsibility, and liability for the content, actions, and implications arising from this work.
> Under no circumstances shall listed authors or any other contributors, past, present, or future, be held personally liable for any disputes, claims, or liabilities related to this document or its usage.
> All such matters shall be addressed exclusively to Nitro Agility Srl.
