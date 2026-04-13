# 🔌 Endpointer

> A production-grade, event-driven microservices backend platform built with Node.js — featuring an API Gateway, message queuing, real-time event streaming, and multi-tier cloud architecture.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
  - [1. System Overview — API Gateway & Microservices](#1-system-overview--api-gateway--microservices)
  - [2. Event-Driven Streaming — Kafka Pipeline](#2-event-driven-streaming--kafka-pipeline)
  - [3. IoT Event-Driven Communication Pattern](#3-iot-event-driven-communication-pattern)
  - [4. Retail Microservices — Node.js Services](#4-retail-microservices--nodejs-services)
  - [5. Cloud Deployment — Azure SaaS Architecture](#5-cloud-deployment--azure-saas-architecture)
  - [6. Three-Tier Application Architecture](#6-three-tier-application-architecture)
- [Tech Stack](#tech-stack)
- [Services & Responsibilities](#services--responsibilities)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [API Reference](#api-reference)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

**Endpointer** is a scalable backend platform designed around the principles of microservices, event-driven architecture, and asynchronous communication. It serves as the backbone for high-throughput applications — from retail e-commerce to IoT systems — by decoupling services through message queues, routing traffic intelligently via an API Gateway, and deploying on Azure with SaaS-grade multi-tenancy.

### Key Highlights

- **API Gateway** as a single entry point for all client interactions
- **Message Queue** for asynchronous, decoupled inter-service communication
- **Kafka-based event streaming** for real-time analytics and data pipelines
- **Node.js microservices** for independent scalability of each domain
- **Azure-native deployment** with per-customer processing stamps and billing integration
- **Three-tier separation** of presentation, business logic, and data storage

---

## Architecture

### 1. System Overview — API Gateway & Microservices

```
Client (HTTPS Request)
       │
       ▼
 ┌─────────────┐
 │ API Gateway │
 └──────┬──────┘
        │  (via Message Queue)
        ▼
 ┌──────────────────┐     ┌─────────────────┐
 │ Order Processing │◄───►│ Payment Service │
 │     Service      │     └─────────────────┘
 └──────────────────┘
        │
        ▼
 ┌─────────────────────────────────────────┐
 │             Message Queue               │
 └──────┬──────────────┬───────────────────┘
        │              │              │
        ▼              ▼              ▼
 ┌────────────┐ ┌────────────┐ ┌──────────────────┐
 │ Inventory  │ │ Shipment   │ │  Notification    │
 │  Service   │ │  Service   │ │     Service      │
 └────────────┘ └────────────┘ └──────────────────┘
```

The client communicates exclusively with the **API Gateway** over HTTPS. The gateway routes requests into the system via a **Message Queue**, ensuring the client never directly calls a downstream service. The **Order Processing Service** coordinates with the **Payment Service** synchronously, while the Message Queue asynchronously fans out events to the **Inventory**, **Shipment**, and **Notification** services.

**Benefits of this pattern:**
- Client is decoupled from implementation details of individual services
- Failed downstream services do not affect the client response cycle
- Each service can be scaled, deployed, and updated independently

---

### 2. Event-Driven Streaming — Kafka Pipeline

```
 ┌─────────┐    events    ┌───────────┐
 │  edm-ui │◄────HTTPS───►│ edm-relay │
 │ (Mock   │              │(Publishes │
 │  e-comm │              │to Kafka)  │
 │   UI)   │              └─────┬─────┘
 └─────────┘                    │
                                ▼
                   ┌────────────────────────┐
                   │      Heroku Kafka       │
                   │                        │
                   │  ┌──────────────────┐  │
                   │  │ edm-ui-clicks    │  │
                   │  │    topic         │  │
                   │  └──────────────────┘  │
                   │  ┌──────────────────┐  │
                   │  │ edm-ui-payloads  │  │
                   │  │    topic         │  │
                   │  └──────────────────┘  │
                   └──────────┬─────────────┘
                              │  All topics
                   ┌──────────┴──────────┐
                   │                     │
                   ▼                     ▼
           ┌──────────────┐    ┌──────────────────┐
           │  edm-ui-     │    │    edm-stats      │
           │  stream      │    │ (Generates &      │
           │ (socket.io)  │    │  persists stats)  │
           └──────┬───────┘    └────────┬──────────┘
                  │ Text (HTTPS)        │ Read/Write
                  ▼                     ▼
           ┌──────────────┐    ┌──────────────────┐
           │ edm-dashboard│    │    Postgres       │
           │ (Admin UI,   │    │    Database       │
           │  live & hist)│    └──────────────────┘
           └──────────────┘
```

This pipeline captures **user interaction events** (clicks, page loads) from a mock e-commerce UI and streams them through Kafka topics into analytics consumers.

| Service | Role |
|---|---|
| `edm-ui` | Mock frontend that tracks click & page-load events |
| `edm-relay` | Publishes captured events to Kafka topics |
| `edm-ui-clicks` | Kafka topic for UI click events |
| `edm-ui-payloads` | Kafka topic for full event payloads |
| `edm-ui-stream` | Streams events to dashboard via socket.io |
| `edm-stats` | Aggregates and persists event statistics to Postgres |
| `edm-dashboard` | Admin UI with historical and live data visualisation |

---

### 3. IoT Event-Driven Communication Pattern

```
                              ┌─────────────────────┐
                              │  Existing Enterprise │
                              │       Systems        │
                              └──────────┬───────────┘
                                         │
 ┌──────────────┐              ┌─────────▼──────────────────────────┐
 │ Remote Assets│◄────────────►│            Event Medium            │
 │ (Industrial  │              │                                    │
 │  machinery)  │              │  ┌──────────────────────────────┐  │
 └──────────────┘              │  │          Microservices       │  │
                               │  │  ┌─────────────────────────┐ │  │
                               │  │  │ Remote Asset Monitoring │ │  │
                               │  │  ├─────────────────────────┤ │  │
                               │  │  │ Asset Health Analysis   │ │  │
                               │  │  ├─────────────────────────┤ │  │
                               │  │  │ Work Order Generation   │ │  │
                               │  │  ├─────────────────────────┤ │  │
                               │  │  │ Maintenance Scheduling  │ │  │
                               │  │  └─────────────────────────┘ │  │
                               │  └──────────────────────────────┘  │
                               │                                    │
                               │  ┌──────────────────────────────┐  │
                               │  │  Datalake & Advanced Analytics│  │
                               │  └──────────────────────────────┘  │
                               └────────────────────────────────────┘
                                         │
                              ┌──────────▼───────────┐
                              │      Client Apps      │
                              │  - Admin Staff        │
                              │  - Engineers          │
                              │  - Maintenance Staff  │
                              │  - External Clients   │
                              └──────────────────────┘
```

This pattern demonstrates how **Endpointer** supports IoT use cases. Remote industrial assets publish events to a central **Event Medium** (message broker), which fans them out to specialized microservices. Results are consumed by a datalake for advanced analytics and served to various client app roles.

---

### 4. Retail Microservices — Node.js Services

```
                    ┌─────────────────────────────────────────────────────┐
                    │              Node.js Microservices                  │
                    │                                                     │
        ┌──────┐    │  ┌─────────────────────┐          ┌────────────┐   │
        │ User │────┼─►│   Search Service    │──────────►│ Database  │   │
        └──────┘    │  └─────────────────────┘          └────────────┘   │
                    │                                                     │
                    │  ┌─────────────────────┐                           │
                    ├─►│ Suggestion Service  │                           │
                    │  └─────────────────────┘                           │
                    │                                                     │
                    │  ┌─────────────────────┐          ┌────────────┐   │
                    ├─►│  Checkout Service   │          │            │   │
                    │  └─────────────────────┘──────────► Database  │   │
                    │                                   │            │   │
                    │  ┌─────────────────────┐          │            │   │
                    └─►│    Cart Service     │──────────►            │   │
                       └─────────────────────┘          └────────────┘   │
                       └─────────────────────────────────────────────────┘
```

The retail layer exposes four independent Node.js services, each handling a specific domain:

| Service | Responsibility | Database |
|---|---|---|
| **Search Service** | Full-text and filtered product search | Dedicated DB (read-optimised) |
| **Suggestion Service** | Personalised product recommendations | In-memory / shared |
| **Checkout Service** | Order placement and payment processing | Shared transactional DB |
| **Cart Service** | Shopping cart state management | Shared transactional DB |

Each service owns its own data, preventing tight coupling and allowing independent scaling.

---

### 5. Cloud Deployment — Azure SaaS Architecture

```
Customer ────────────────────► Azure Marketplace ◄──────────────── Publisher
   │         Purchase SaaS         │                Publish SaaS offer
   │                               │ Fulfillment API
   │                               ▼
   │                     Publisher Subscription
   │                     ┌──────────────────────────────────────────────────┐
   │                     │   Control Resource Group                         │
   │                     │   ┌─────────────────────────────────────────┐   │
   │                     │   │  UX (Web App)  ►  Orchestration         │   │
   │                     │   │  Landing Page     ┌──────────────────┐  │   │
   │                     │   │                   │ Management       │  │   │
   │                     │   │                   │ Deployment       │  │   │
   │                     │   │                   │ Database         │  │   │
   │                     │   │                   │ Container Reg.   │  │   │◄── Entra
   │                     │   │                   │ Billing ─────────┼──┼──► Azure Billing
   │                     │   │                   └──────────────────┘  │   │   Metering API
   │                     │   └─────────────────────────────────────────┘   │
   │                     │                                                  │
   │                     │   Processing Stamp (one per customer)            │
   │                     │   ┌──────────────────────────────────────────┐   │
   │ Customer Subscription│   │  Processing Logic Containers            │   │
   │ ┌────────────────┐  │   │  Monitoring                             │   │
   │ │ /raw  (read)   │◄─┼───┤                                         │   │
   │ │ /processed (W) │──┼──►│                                         │   │
   │ └────────────────┘  │   └──────────────────────────────────────────┘   │
   └─────────────────────┴──────────────────────────────────────────────────┘
```

Endpointer is deployable as a **multi-tenant SaaS solution on Azure**. The architecture separates concerns into:

- **Azure Marketplace** — handles customer onboarding via the Fulfillment API
- **Control Resource Group** — manages the UX, orchestration, container registry, and billing integration
- **Processing Stamp** — one per customer, reads from `/raw` and writes to `/processed` in the customer's own subscription
- **Entra** — cross-tenant buyer authentication
- **Azure Billing Metering API** — reports custom usage meters for pay-as-you-go billing

> **Note:** Processing stamps are deployed in the same region as the customer's data to minimise networking costs.

---

### 6. Three-Tier Application Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  ┌──────────────────┐       ┌──────────────────┐    ┌────────────┐  │
│  │  Presentation    │  API  │  Business Logic   │    │   Data     │  │
│  │      Tier        │──────►│      Tier         │───►│  Storage  │  │
│  │  (Client)        │       │  (Server)         │    │   Tier    │  │
│  │                  │◄──────│                   │◄───│           │  │
│  │  Browser / App   │ Resp. │  Node.js +        │    │ Postgres  │  │
│  │                  │       │  Express          │    │ / MongoDB │  │
│  └──────────────────┘       └──────────────────┘    └────────────┘  │
│         Request ──────► API Call ──► SELECT FROM ──► Requested Data  │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

At the foundation, every Endpointer service follows a **three-tier pattern**:

| Tier | Technology | Responsibility |
|---|---|---|
| **Presentation** | React / Mobile App | Renders UI, sends API requests |
| **Business Logic** | Node.js + Express | Processes requests, applies rules, coordinates services |
| **Data Storage** | PostgreSQL / MongoDB | Persists and retrieves data |

Communication between tiers is strictly one-directional: the presentation tier never talks directly to the database.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Runtime | Node.js (LTS) |
| Framework | Express.js |
| Message Queue | Apache Kafka (Heroku Kafka) / RabbitMQ |
| Databases | PostgreSQL, MongoDB |
| Real-time | Socket.io |
| Cloud | Microsoft Azure (Marketplace, Entra, Billing) |
| Containerisation | Docker, Azure Container Registry |
| Authentication | Microsoft Entra ID (cross-tenant) |
| Monitoring | Azure Monitor |
| API Style | REST over HTTPS |

---

## Services & Responsibilities

| Service | Port | Description |
|---|---|---|
| `api-gateway` | 8080 | Entry point — routes, authenticates, and rate-limits all requests |
| `order-service` | 3001 | Manages order lifecycle and coordinates with payment |
| `payment-service` | 3002 | Handles payment processing and confirmation |
| `inventory-service` | 3003 | Tracks stock levels and reservations |
| `shipment-service` | 3004 | Manages delivery logistics |
| `notification-service` | 3005 | Sends email/SMS/push notifications |
| `search-service` | 3006 | Full-text product search |
| `suggestion-service` | 3007 | Personalised recommendations |
| `checkout-service` | 3008 | Order placement and payment coordination |
| `cart-service` | 3009 | Shopping cart CRUD operations |
| `edm-relay` | 4001 | Publishes frontend events to Kafka |
| `edm-ui-stream` | 4002 | Streams Kafka events to dashboard via socket.io |
| `edm-stats` | 4003 | Aggregates and persists event statistics |
| `edm-dashboard` | 4004 | Admin dashboard for live and historical analytics |

---

## Getting Started

### Prerequisites

- Node.js >= 18.x
- Docker & Docker Compose
- Apache Kafka (or a Heroku Kafka add-on)
- PostgreSQL >= 14

### Clone the repository

```bash
git clone https://github.com/Chirag-2199/Endpointer.git
cd Endpointer
```

### Install dependencies

```bash
npm install
```

### Start all services with Docker Compose

```bash
docker-compose up --build
```

### Start a specific service

```bash
cd services/order-service
npm run dev
```

---

## Environment Variables

Create a `.env` file at the project root (or per service):

```env
# API Gateway
PORT=8080
JWT_SECRET=your_jwt_secret

# Kafka
KAFKA_BROKER=localhost:9092
KAFKA_CLIENT_ID=endpointer
KAFKA_GROUP_ID=endpointer-group

# PostgreSQL
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_USER=endpointer
POSTGRES_PASSWORD=yourpassword
POSTGRES_DB=endpointer_db

# Azure (for cloud deployment)
AZURE_CLIENT_ID=your_client_id
AZURE_TENANT_ID=your_tenant_id
AZURE_CLIENT_SECRET=your_secret

# Socket.io
SOCKET_PORT=4002
```

> **Never commit `.env` files.** Use `.env.example` as a template.

---

## API Reference

All requests are made to the **API Gateway** at `http://localhost:8080`.

### Orders

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/orders` | Create a new order |
| `GET` | `/api/orders/:id` | Get order by ID |
| `PATCH` | `/api/orders/:id/status` | Update order status |

### Cart

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/cart` | Get current user's cart |
| `POST` | `/api/cart/items` | Add item to cart |
| `DELETE` | `/api/cart/items/:itemId` | Remove item from cart |

### Search

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/search?q=keyword` | Search products |
| `GET` | `/api/suggestions` | Get personalised suggestions |

### Checkout

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/checkout` | Initiate checkout |
| `POST` | `/api/checkout/confirm` | Confirm and place order |

---

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a new branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -m 'Add some feature'`)
4. Push to the branch (`git push origin feature/your-feature`)
5. Open a Pull Request

Please make sure your code follows the existing style and all services remain independently runnable.

---

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---
<img width="867" height="445" alt="postman 1" src="https://github.com/user-attachments/assets/0d4bc5cc-cf96-448e-ba4d-68341bf2c5b9" />
<img width="961" height="608" alt="postman2" src="https://github.com/user-attachments/assets/bf597cb9-b1ca-4453-8c77-07f5237657b4" />
![postman3](https://github.com/user-attachments/assets/54a1ec19-cc03-44b5-b145-209e2cb7c551)
![postman5](https://github.com/user-attachments/assets/4a34f198-f058-4997-beaa-9681321df6f1)
<img width="1572" height="591" alt="postman6" src="https://github.com/user-attachments/assets/60aff554-d91a-4c0d-aeab-157a6ff46dbf" />
<img width="1290" height="501" alt="postman7" src="https://github.com/user-attachments/assets/3c2146b5-20c4-4595-8a01-9fa06395a78b" />

