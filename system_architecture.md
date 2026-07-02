# Nurons System Architecture

Nurons is designed as a highly scalable, decoupled web application. It utilizes a Service-Oriented Architecture (SOA), background task queues, and the Transactional Outbox pattern to ensure eventual consistency and reliability under heavy AI workloads.

## High-Level Architecture Diagram

```mermaid
graph TB
  classDef default fill:#1E293B,stroke:#475569,stroke-width:2px,color:#F8FAFC;
  classDef external fill:#0F172A,stroke:#3B82F6,stroke-width:2px,color:#93C5FD;
  classDef database fill:#064E3B,stroke:#10B981,stroke-width:2px,color:#A7F3D0;
  classDef queue fill:#7F1D1D,stroke:#EF4444,stroke-width:2px,color:#FECACA;

  subgraph ClientEnv [Client Environment]
    Browser([Web Browser])
    CF[AWS CloudFront]
    S3[AWS S3 - React SPA]
    
    Browser -->|HTTPS Request| CF
    CF -.->|Fetch Static Assets| S3
  end

  subgraph ServerEnv [Backend Environment]
    Nginx[Nginx Reverse Proxy]
    Gateway[Express API Gateway]
    
    CF -->|API Traffic| Nginx
    Nginx --> Gateway
    
    subgraph SOA [Service Oriented Architecture]
      Auth[Auth & Identity Service]
      Chat[Chat & Agent Service]
      RAG[Document & RAG Service]
      Recall[Recall & Spaced Repetition]
      Billing[Billing Service]
      
      Gateway --> Auth
      Gateway --> Chat
      Gateway --> RAG
      Gateway --> Recall
      Gateway --> Billing
    end
    
    subgraph Reliability [Eventual Consistency]
      Outbox[Transactional Outbox]
      BullMQ[BullMQ Background Workers]
      
      Chat --> Outbox
      RAG --> Outbox
      Recall --> Outbox
      Outbox -->|Async Dispatch| BullMQ:::queue
    end
    
    subgraph DataLayer [Data Persistence]
      Mongo[(MongoDB - App State)]:::database
      Redis[(Redis - Caching/Jobs)]:::database
      Qdrant[(Qdrant - Vector DB)]:::database
      
      SOA --> Mongo
      BullMQ --> Redis
      BullMQ --> Qdrant
      SOA --> Qdrant
    end
  end
  
  subgraph ExternalAPIs [External Integrations]
    Gemini[Google Gemini]:::external
    Voyage[Voyage AI / Reranker]:::external
    Cloudinary[Cloudinary Storage]:::external
    Paddle[Paddle Billing]:::external
    
    Chat --> Gemini
    RAG --> Voyage
    RAG --> Cloudinary
    Billing --> Paddle
  end
```

## Core Architectural Concepts

1. **Frontend Edge Delivery:** The React frontend is compiled and stored in an S3 bucket, served globally via AWS CloudFront for sub-second load times.
2. **Service-Oriented Architecture (SOA):** The Express backend delegates business logic to specialized services (`ChatService`, `RAGService`). This separation of concerns allows for easier testing and future microservice extraction.
3. **Transactional Outbox & Eventual Consistency:** To avoid dual-write failures (e.g., saving a user to MongoDB but failing to enqueue a Redis job), database writes and job dispatches are tied together using the Outbox pattern. Background workers process heavy tasks (like semantic chunking) to maintain eventual consistency.
4. **Vector & Semantic Search Pipeline:** Qdrant is used in tandem with Voyage AI embeddings to process uploaded documents asynchronously, enabling lightning-fast Retrieval-Augmented Generation (RAG).
