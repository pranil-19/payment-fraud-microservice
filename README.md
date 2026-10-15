# payment-fraud-microservice
### Secure Fraud Resolution and Metadata Exchange Microservice

## Overview

A scalable microservice-based solution that supports secure exchange of transaction and order metadata between external payment networks and merchant systems for fraud-resolution and transaction-dispute use cases.

The system helps provide relevant purchase information so issuers and customers can better understand transactions, resolve queries faster, and reduce unnecessary disputes.

> **Confidentiality Note:** This README uses generic architecture and business terminology. It excludes proprietary source code, internal URLs, credentials, database schemas, secrets, customer data, and confidential implementation details.

## Architecture

The solution consists of three main services:

```text
External Payment Network
              |
              v
+-------------------------------+
| Service 1                     |
| Orchestration Layer           |
|                               |
| - Request parsing             |
| - Data transformation         |
| - Redis caching               |
| - Service orchestration       |
+---------------+---------------+
                |
                v
+-------------------------------+
| Service 2                     |
| Business / Database Layer     |
|                               |
| - Business logic              |
| - PostgreSQL queries          |
| - Kafka data integration      |
+---------------+---------------+
                ^
                |
+---------------+---------------+
| Service 3                     |
| Kafka Producer                |
|                               |
| - Transaction updates         |
| - Publishes latest data       |
+-------------------------------+
```

### Service Responsibilities

- **Orchestration Service:** Handles incoming requests, transforms data, checks Redis for cached metadata, and coordinates communication with the business service.
- **Business Service:** Executes business logic, retrieves transaction information from PostgreSQL, and integrates with transaction updates received through Kafka.
- **Kafka Producer Service:** Publishes relevant transaction updates to Kafka for consumption by downstream services.

## Request Flow

1. The external payment network sends a fraud-resolution request.
2. The orchestration service parses and transforms the incoming request.
3. The service checks Redis for the required transaction metadata.
4. If the data is available in the cache, the service retrieves it; otherwise, it requests the required information from the business service.
5. The business service queries PostgreSQL and combines the result with relevant transaction information received through Kafka.
6. The business service returns the processed information to the orchestration service.
7. The orchestration service transforms the response into the required external format and returns it to the requesting network.
8. In parallel, the Kafka producer publishes relevant transaction updates to support downstream processing.

## Redis Caching

Redis is used to temporarily cache transaction-related metadata that may be requested repeatedly across different request types.

Caching reduces repeated database lookups, improves response times, and helps lower the load on PostgreSQL.

For repeated requests:

```text
First request  -> Redis miss -> PostgreSQL -> Cache result
Later request  -> Redis hit  -> Return cached data
```

The caching and database-access optimizations reduced redundant database queries by approximately **40%** and improved repeated-request response time from approximately **2 seconds to 200 milliseconds**, representing a **90% reduction in latency**.

If Redis becomes unavailable, the service can fall back to PostgreSQL, provided the fallback mechanism is available and operational. Performance may temporarily degrade because requests may require database access.

## Kafka Integration

Apache Kafka is used to distribute transaction updates between producer and consumer services.

This integration supports:

- **Service decoupling:** Producers and consumers can operate independently.
- **Event-driven processing:** Transaction updates can be published and consumed asynchronously.
- **Event replay:** Retained events can be reprocessed when the required Kafka configuration and retention policies permit it.
- **Near-real-time updates:** Services can consume transaction updates as they become available.
- **Reduced repeated lookups:** Relevant transaction information can be made available to downstream processing without relying exclusively on repeated database queries.

## Business Use Case

The application supports transaction-information exchange for fraud investigation and transaction-dispute resolution workflows.

When a cardholder questions a transaction, an issuer may request additional purchase information to help identify the transaction and resolve the query. The application acts as an integration layer between an external payment network and merchant-side order information.

The returned information can help:

- Customers recognize legitimate purchases.
- Issuer call-center agents investigate transaction queries.
- Reduce avoidable disputes and chargebacks.
- Support transaction-dispute evaluation using relevant purchase and order information.

## Key Benefits

- **Approximately 40% fewer redundant database queries** through caching and database-access optimization.
- **90% lower repeated-request latency**, from approximately 2 seconds to 200 milliseconds.
- Separation of orchestration, business logic, and event-processing responsibilities.
- Near-real-time transaction updates through Kafka.
- Reduced PostgreSQL load for frequently requested transaction metadata.
- Improved performance for repeated requests through Redis caching.

## Technology Stack

- **Language:** Java
- **Framework:** Spring Boot
- **Architecture:** Microservices, REST APIs
- **Database:** PostgreSQL
- **Caching:** Redis
- **Messaging:** Apache Kafka

## Resume Summary

Designed and enhanced a scalable Java/Spring Boot microservice supporting fraud-resolution workflows and transaction metadata exchange between payment networks and merchant systems. Integrated Redis caching and optimized PostgreSQL access patterns, reducing redundant database queries by approximately 40% and repeated-request latency from approximately 2 seconds to 200 milliseconds.

## Confidentiality

This project description is intentionally sanitized for portfolio and interview use. Proprietary source code, internal endpoints, credentials, database schemas, customer information, infrastructure details, and sensitive business information should not be published in a public repository.

All architecture descriptions, technology references, and performance metrics should accurately reflect the implementation and work performed.
