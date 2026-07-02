# 🧬 Dendrites — High-Level System Architecture Diagrams

> Copy any of the Mermaid code blocks below and paste into [mermaid.live](https://mermaid.live) or any Mermaid-compatible renderer.

---

## 1. Full System Architecture (Bird's Eye View)

```mermaid
graph TB
    %% ─── CLIENT LAYER ───
    subgraph CLIENT["🌐 Client Layer"]
        direction LR
        Browser["🖥️ React + Vite SPA<br/>├─ Redux Toolkit (Auth/Explorer State)<br/>├─ TanStack React Query (Server State)<br/>├─ React Router v6<br/>├─ react-virtuoso (Virtual Scroll)<br/>└─ Firebase SDK (FCM Registration)"]
    end

    %% ─── LOAD BALANCER ───
    LB["⚖️ Nginx / Reverse Proxy<br/>├─ SSL Termination<br/>├─ Rate Limiting<br/>├─ Gzip Compression<br/>└─ Static Asset Serving"]

    %% ─── API GATEWAY ───
    subgraph BACKEND["🏗️ Backend Layer (Node.js + Express)"]
        direction TB

        MW["🛡️ Middleware Stack<br/>├─ CORS<br/>├─ Cookie Parser<br/>├─ Auth Middleware (JWT verify)<br/>├─ Error Handler<br/>└─ 🔜 Rate Limiter (per tier)"]

        subgraph GUARDRAILS["🛡️ NeMo Guardrails"]
            direction LR
            InputGuard["📥 Input Guard<br/>├─ Topic Filtering<br/>├─ Jailbreak Detection<br/>├─ PII Redaction<br/>└─ Prompt Injection Shield"]
            OutputGuard["📤 Output Guard<br/>├─ Hallucination Check<br/>├─ Toxicity Filter<br/>├─ Factual Grounding<br/>└─ Response Sanitization"]
        end

        subgraph CONTROLLERS["🎮 Controllers"]
            direction LR
            AuthCtrl["Auth"]
            ChatCtrl["Chat"]
            FolderCtrl["Folder"]
            RecallCtrl["Recall"]
            BranchCtrl["Branch"]
            AdminCtrl["🔜 Admin"]
            BillingCtrl["🔜 Billing"]
            SuggestCtrl["🔜 Suggestions"]
        end

        subgraph USECASES["⚙️ Application Layer (36 Use Cases)"]
            direction LR
            AuthUC["Auth<br/>6 use cases"]
            ChatUC["Chat<br/>10 use cases"]
            FolderUC["Folder<br/>4 use cases"]
            RecallUC["Recall<br/>6 use cases"]
            BranchUC["Branch<br/>2 use cases"]
            WorkerUC["Workers<br/>6 use cases"]
            SchedulerUC["Schedulers<br/>2 use cases"]
        end

        subgraph SERVICES["🔧 Services"]
            direction LR
            AISvc["AIService<br/>├─ Search Routing<br/>├─ Stream with Retry<br/>└─ Anchor Context Cache"]
            EmbSvc["EmbeddingService<br/>└─ Jina v2 Wrapper"]
            ChunkSvc["SemanticChunkingService<br/>├─ Markdown Parser<br/>├─ Boundary Detection<br/>└─ Token Enforcement"]
            FileSvc["FileUploadService<br/>├─ MIME Validation<br/>└─ Cloudinary Upload"]
            SearchSvc["SearchCacheService<br/>└─ Tavily + Qdrant Cache"]
            PubSubSvc["DocumentProgressPubSub<br/>└─ Redis Pub/Sub"]
        end
    end

    %% ─── ASYNC PIPELINE ───
    subgraph ASYNC["⚡ Async Processing Pipeline"]
        direction TB
        subgraph QUEUES["📬 BullMQ Queues"]
            direction LR
            Q1["embeddingQueue"]
            Q2["descriptionQueue"]
            Q3["summaryQueue"]
            Q4["stateQueue"]
            Q5["recallQueue"]
            Q6["documentChunkingQueue"]
        end

        subgraph WORKERS["👷 Workers (6)"]
            direction LR
            W1["Embedding Worker<br/>Code → Dual Vector"]
            W2["Description Worker<br/>Code → AI Description"]
            W3["Summary Worker<br/>Messages → Rolling Summary"]
            W4["State Worker<br/>Summary → Facts → Vectors"]
            W5["Recall Worker<br/>Card → Schedule + Notify"]
            W6["Document Worker<br/>PDF → Chunks → Vectors"]
        end

        subgraph CRON["⏰ Cron Sweepers (3)"]
            direction LR
            C1["Outbox Sweeper<br/>every 30s"]
            C2["Description Sweeper<br/>every 60s"]
            C3["Search Cache Sweeper<br/>every 6h"]
        end
    end

    %% ─── DATA LAYER ───
    subgraph DATA["💾 Data Layer"]
        direction LR
        subgraph MONGO["🍃 MongoDB"]
            direction TB
            Users["users"]
            Chats["chats"]
            Messages["messages"]
            Folders["folders"]
            CodeBlocks["codeblocks"]
            SubChats["subchats"]
            Recalls["recalls"]
            OutboxEvents["outboxevents"]
        end

        subgraph REDIS_DB["🔴 Redis"]
            direction TB
            SessionCache["Session Cache<br/>Anchor Context"]
            JobBroker["BullMQ Job Broker"]
            PubSubChannel["Pub/Sub Channels<br/>Document Progress"]
            OTPStore["🔜 OTP Store (TTL)"]
        end

        subgraph QDRANT["🔷 Qdrant Vector DB"]
            direction TB
            CodeColl["code_blocks<br/>Dual Vector (code + desc)<br/>768-dim"]
            SummaryColl["chat_summaries<br/>Single Vector<br/>768-dim"]
            SearchColl["search_cache<br/>Tavily Results<br/>768-dim"]
            DocColl["documents<br/>Semantic Chunks<br/>768-dim"]
        end
    end

    %% ─── EXTERNAL SERVICES ───
    subgraph EXTERNAL["☁️ External Services / Third-Party APIs"]
        direction LR
        Gemini["🤖 Google Gemini API<br/>├─ gemini-3-flash-preview (main)<br/>├─ gemini-2.5-flash-lite (router)<br/>└─ Multi-key rotation pool"]
        Infinity["🧮 Infinity Server (Local)<br/>├─ Jina v2 Base Code<br/>├─ 768-dim embeddings<br/>└─ Batch support"]
        Tavily["🔍 Tavily Search API<br/>└─ Real-time web search"]
        Cloudinary["☁️ Cloudinary CDN<br/>├─ Image uploads<br/>└─ Document storage"]
        LlamaParse["📄 LlamaParse<br/>├─ PDF → Markdown<br/>└─ Agentic tier"]
        Firebase["🔔 Firebase Cloud Messaging<br/>└─ Push Notifications"]
        Stripe["💳 🔜 Stripe<br/>├─ Checkout Sessions<br/>├─ Webhooks<br/>└─ Customer Portal"]
        NeMo["🛡️ NVIDIA NeMo<br/>├─ Guardrails Server<br/>├─ Colang Rules<br/>└─ Content Safety"]
    end

    %% ─── CONNECTIONS ───
    Browser -->|"HTTPS / SSE Streams"| LB
    LB -->|"Proxy Pass"| MW
    MW --> CONTROLLERS
    CONTROLLERS --> InputGuard
    InputGuard --> USECASES
    USECASES --> SERVICES
    USECASES --> OutputGuard
    OutputGuard -->|"Sanitized Response"| CONTROLLERS

    SERVICES --> MONGO
    SERVICES --> REDIS_DB
    SERVICES --> QDRANT
    SERVICES --> Gemini
    SERVICES --> Infinity
    SERVICES --> Tavily
    SERVICES --> Cloudinary
    SERVICES --> LlamaParse

    CONTROLLERS -->|"Enqueue Jobs"| QUEUES
    QUEUES --> WORKERS
    WORKERS --> MONGO
    WORKERS --> QDRANT
    WORKERS --> Infinity
    WORKERS --> Gemini
    WORKERS --> Firebase
    WORKERS --> NeMo

    CRON --> MONGO
    CRON --> QDRANT

    PubSubSvc --> PubSubChannel

    %% ─── STYLING ───
    classDef clientStyle fill:#1e3a5f,stroke:#3b82f6,stroke-width:2px,color:#e2e8f0
    classDef backendStyle fill:#1e293b,stroke:#8b5cf6,stroke-width:2px,color:#e2e8f0
    classDef asyncStyle fill:#1a2332,stroke:#f59e0b,stroke-width:2px,color:#e2e8f0
    classDef dataStyle fill:#1a2e1a,stroke:#22c55e,stroke-width:2px,color:#e2e8f0
    classDef externalStyle fill:#2d1b1b,stroke:#ef4444,stroke-width:2px,color:#e2e8f0
    classDef guardrailStyle fill:#2d1b3d,stroke:#ec4899,stroke-width:2px,color:#e2e8f0

    class CLIENT clientStyle
    class BACKEND backendStyle
    class ASYNC asyncStyle
    class DATA dataStyle
    class EXTERNAL externalStyle
    class GUARDRAILS guardrailStyle
```

---

## 2. Request Flow — Message Send Pipeline (with NeMo Guardrails)

```mermaid
sequenceDiagram
    autonumber
    participant U as 🖥️ Browser
    participant LB as ⚖️ Nginx
    participant MW as 🛡️ Auth Middleware
    participant IG as 🛡️ NeMo Input Guard
    participant C as 🎮 ChatController
    participant PM as ⚙️ PrepareMessage
    participant EMB as 🧮 Infinity (Jina v2)
    participant Q as 🔷 Qdrant
    participant R as 🔴 Redis
    participant AI as 🤖 Gemini API
    participant OG as 🛡️ NeMo Output Guard
    participant SM as ⚙️ SaveModelReply
    participant DB as 🍃 MongoDB
    participant BQ as 📬 BullMQ

    U->>LB: POST /chats/:id/message (SSE)
    LB->>MW: Proxy + Rate Check
    MW->>MW: Verify JWT Cookie
    MW->>IG: Forward user message

    rect rgb(45, 27, 61)
        Note over IG: NeMo Input Guardrails
        IG->>IG: Topic Filter (block off-topic)
        IG->>IG: Jailbreak Detection
        IG->>IG: PII Redaction
        IG-->>C: ✅ Approved / ❌ Blocked
    end

    C->>PM: execute(chatId, userId, message)

    par Dual Embedding
        PM->>EMB: embed(message, CODE_RETRIEVAL)
        PM->>EMB: embed(message, RETRIEVAL_QUERY)
    end

    par Triple Vector Search
        PM->>Q: searchSimilarCode(codeVec, descVec, chatIds[])
        PM->>Q: searchSimilarChatChunk(descVec, chatIds[])
        PM->>Q: searchDocuments(query, descVec, chatIds[])
    end

    PM->>DB: findRecentMessages(chatId, 20)

    opt Sliding Window Deficit
        PM->>DB: findRecentMessages(parentChatId, deficit)
        Note over PM,DB: Traverse contextParent chain
    end

    PM->>PM: Build System Prompt<br/>+ Inherited Summaries<br/>+ RAG Context<br/>+ Mode Instructions

    PM-->>C: { contents, chat }

    C->>AI: generateContentStream(contents)

    loop SSE Chunks
        AI-->>OG: Raw AI chunk

        rect rgb(45, 27, 61)
            Note over OG: NeMo Output Guardrails
            OG->>OG: Hallucination Check
            OG->>OG: Toxicity Filter
            OG->>OG: Factual Grounding
        end

        OG-->>U: data: {"text": "..."}
    end

    AI-->>C: Stream complete

    C->>SM: execute(chatId, fullResponse)
    SM->>DB: Save model message
    SM->>SM: Extract code blocks (SHA-256 dedup)
    SM->>DB: Save code blocks
    SM->>DB: Create OutboxEvents
    SM->>BQ: Enqueue: embedding, summary, state jobs

    C-->>U: data: {"type":"metadata", ids...}
    C-->>U: data: [DONE]
```

---

## 3. AI Memory System — 5-Layer Architecture

```mermaid
graph TB
    subgraph INPUT["📨 User Message Arrives"]
        MSG["'Explain SQL JOIN optimization'"]
    end

    subgraph LAYER1["🪟 Layer 1: Sliding Context Window (Short-Term)"]
        direction LR
        SW["Last 20 messages<br/>sent directly to Gemini"]
        SWD["If chat has < 20 messages:<br/>fill deficit from contextParent chain<br/>(max 300 ancestors)"]
    end

    subgraph LAYER2["📝 Layer 2: Rolling Summary (Mid-Term)"]
        direction LR
        RS["AI-compressed summary<br/>updated every 20 messages"]
        RSI["Injected as:<br/>[ACTIVE CONVERSATION STATE]<br/>in system prompt"]
    end

    subgraph LAYER3["🔍 Layer 3: Vector RAG (Long-Term)"]
        direction TB
        subgraph SEARCH["Triple Parallel Search"]
            S1["🔷 code_blocks<br/>Dual vector match<br/>(code + description)"]
            S2["🔷 chat_summaries<br/>Extracted facts<br/>similarity > 0.62"]
            S3["🔷 documents<br/>Semantic chunks<br/>from uploaded files"]
        end
        DEDUP["Deduplicate against<br/>sliding window content"]
    end

    subgraph LAYER4["🧠 Layer 4: Spaced Repetition (Permanent)"]
        direction LR
        SM2["SM-2 Algorithm<br/>├─ Learning phase (steps 0-3)<br/>├─ Review phase (intervals grow)<br/>└─ easeFactor adjustment"]
        FCM["🔔 Firebase Push Notification<br/>when card is due"]
    end

    subgraph LAYER5["🌍 Layer 5: Global Memory (Cross-Session)"]
        direction TB
        subgraph PROFILE["👤 User Profile (MongoDB)"]
            UP["User Preferences<br/>├─ Preferred language (Python/JS/etc)<br/>├─ Learning style (visual/textual)<br/>├─ Expertise level (beginner→expert)<br/>└─ Response format preference"]
        end
        subgraph GKNOW["🧬 Global Knowledge (Qdrant)"]
            GK1["🔷 user_knowledge<br/>Cross-chat entities, concepts,<br/>and relationships the user<br/>has learned over time"]
            GK2["🔷 user_queries<br/>Historical query patterns<br/>for inline suggestions<br/>and personalization"]
        end
        subgraph EXTRACTION["⚙️ Global Memory Worker"]
            GMW["Periodically scans all<br/>chat summaries per user →<br/>Extracts recurring themes,<br/>expertise signals, and<br/>cross-domain connections"]
        end
    end

    subgraph GUARDRAIL["🛡️ NeMo Guardrails"]
        direction LR
        PRE["Pre-Generation<br/>├─ Input sanitization<br/>└─ Topic enforcement"]
        POST["Post-Generation<br/>├─ Fact verification<br/>└─ Safety filtering"]
    end

    subgraph GEMINI["🤖 Gemini API"]
        PROMPT["Final Prompt =<br/>System Instruction<br/>+ 🌍 Global User Profile<br/>+ Folder Behavior 🔜<br/>+ Inherited Summaries<br/>+ Rolling Summary<br/>+ RAG Code Snippets<br/>+ RAG Facts<br/>+ RAG Document Chunks<br/>+ Global Knowledge Hits<br/>+ Visual Mode Rules<br/>+ Last 20 Messages"]
    end

    MSG --> LAYER1
    MSG --> LAYER3
    MSG -->|"Query user_knowledge<br/>+ user_queries"| LAYER5
    LAYER1 --> PROMPT
    LAYER2 --> PROMPT
    LAYER3 --> DEDUP --> PROMPT
    LAYER5 -->|"[GLOBAL USER PROFILE]<br/>+ cross-chat knowledge"| PROMPT
    PROMPT --> PRE --> GEMINI
    GEMINI --> POST --> RESPONSE["📤 Streamed Response"]
    RESPONSE -->|"User selects text"| LAYER4
    RESPONSE -->|"Post-response analysis"| EXTRACTION
    EXTRACTION -->|"Update profile +<br/>upsert knowledge vectors"| GKNOW
    LAYER4 --> FCM

    S1 --> DEDUP
    S2 --> DEDUP
    S3 --> DEDUP

    classDef layer1 fill:#1e3a5f,stroke:#3b82f6,stroke-width:2px,color:#e2e8f0
    classDef layer2 fill:#1a2e1a,stroke:#22c55e,stroke-width:2px,color:#e2e8f0
    classDef layer3 fill:#2d2a1a,stroke:#f59e0b,stroke-width:2px,color:#e2e8f0
    classDef layer4 fill:#2d1b1b,stroke:#ef4444,stroke-width:2px,color:#e2e8f0
    classDef layer5 fill:#1a1a2e,stroke:#818cf8,stroke-width:2px,color:#e2e8f0
    classDef guard fill:#2d1b3d,stroke:#ec4899,stroke-width:2px,color:#e2e8f0

    class LAYER1 layer1
    class LAYER2 layer2
    class LAYER3 layer3
    class LAYER4 layer4
    class LAYER5 layer5
    class GUARDRAIL guard
```

---

## 4. Async Processing Pipeline (Workers + Outbox)

```mermaid
flowchart LR
    subgraph TRIGGER["🎬 Trigger"]
        SMR["SaveModelReply<br/>use case"]
    end

    subgraph OUTBOX["📦 Outbox Pattern"]
        OE["MongoDB:<br/>OutboxEvent<br/>status: pending"]
        OS["⏰ Outbox Sweeper<br/>(every 30s)"]
    end

    subgraph REDIS_Q["🔴 Redis + BullMQ"]
        EQ["embeddingQueue"]
        DQ["descriptionQueue"]
        SQ["summaryQueue"]
        STQ["stateQueue"]
        RQ["recallQueue"]
        DCQ["docChunkingQueue"]
    end

    subgraph WORKERS["👷 Workers"]
        EW["Embedding Worker<br/>Code → Jina v2 →<br/>Dual vectors →<br/>Qdrant upsert"]
        DW["Description Worker<br/>Code → Gemini Flash →<br/>AI description →<br/>MongoDB update"]
        SW["Summary Worker<br/>Last 20 msgs →<br/>Gemini compress →<br/>Rolling summary"]
        STW["State Worker<br/>Summary → Extract<br/>pipe-delimited facts →<br/>Embed → Qdrant"]
        RW["Recall Worker<br/>SM-2 schedule →<br/>BullMQ delayed job →<br/>Firebase push notification"]
        DCW["Document Worker<br/>LlamaParse → Markdown →<br/>Semantic Chunking →<br/>Qdrant upsert"]
    end

    subgraph STORES["💾 Destinations"]
        MDB["🍃 MongoDB"]
        QDB["🔷 Qdrant"]
        FCM["🔔 Firebase FCM"]
    end

    SMR -->|"1. Create events"| OE
    SMR -->|"2. Enqueue jobs"| REDIS_Q
    OS -->|"Catch missed"| OE
    OE -->|"Re-enqueue"| REDIS_Q

    EQ --> EW --> QDB
    DQ --> DW --> MDB
    SQ --> SW --> MDB
    STQ --> STW --> QDB
    RQ --> RW --> FCM
    DCQ --> DCW --> QDB

    style TRIGGER fill:#1e293b,stroke:#8b5cf6,color:#e2e8f0
    style OUTBOX fill:#1a2e1a,stroke:#22c55e,color:#e2e8f0
    style REDIS_Q fill:#2d1b1b,stroke:#ef4444,color:#e2e8f0
    style WORKERS fill:#2d2a1a,stroke:#f59e0b,color:#e2e8f0
    style STORES fill:#1e3a5f,stroke:#3b82f6,color:#e2e8f0
```

---

## 5. NeMo Guardrails Integration Detail

```mermaid
flowchart TB
    subgraph INPUT_FLOW["📥 Input Pipeline"]
        UM["User Message"] --> TR["Topic Rail<br/>Colang: define user off_topic<br/>→ Block non-learning queries"]
        TR --> JD["Jailbreak Detector<br/>Colang: define user jailbreak<br/>→ Pattern match against<br/>known injection templates"]
        JD --> PII["PII Scanner<br/>→ Redact emails, phones,<br/>SSN, API keys before<br/>they reach the LLM"]
        PII --> FS["Fact-Check Setup<br/>→ Tag claims for<br/>post-generation verification"]
    end

    subgraph CORE["🤖 LLM Generation"]
        GEMINI["Gemini 3 Flash<br/>+ System Prompt<br/>+ RAG Context<br/>+ Memory Layers"]
    end

    subgraph OUTPUT_FLOW["📤 Output Pipeline"]
        HC["Hallucination Check<br/>→ Cross-reference response<br/>against RAG sources<br/>→ Flag unsupported claims"]
        TF["Toxicity Filter<br/>→ Block harmful,<br/>offensive, or biased<br/>content"]
        FG["Factual Grounding<br/>→ Verify code snippets<br/>against known patterns"]
        RS["Response Sanitizer<br/>→ Strip leaked system<br/>prompt fragments<br/>→ Clean formatting"]
    end

    subgraph CONFIG["⚙️ NeMo Config"]
        COLANG["📜 Colang Rules<br/>├─ rails/input.co<br/>├─ rails/output.co<br/>├─ rails/topics.co<br/>└─ rails/retrieval.co"]
        YAML["📄 config.yml<br/>├─ models: gemini-3-flash<br/>├─ rails: input, output<br/>├─ embedding: jina-v2<br/>└─ streaming: true"]
    end

    UM --> INPUT_FLOW
    INPUT_FLOW -->|"✅ Approved"| CORE
    INPUT_FLOW -->|"❌ Blocked"| BLOCK["Return: I can only help<br/>with learning-related topics"]
    CORE --> OUTPUT_FLOW
    OUTPUT_FLOW -->|"✅ Safe"| DELIVER["📤 Stream to Client"]
    OUTPUT_FLOW -->|"⚠️ Flagged"| MODIFY["Modify/Redact<br/>flagged sections"]
    MODIFY --> DELIVER

    CONFIG -.->|"Loads at startup"| INPUT_FLOW
    CONFIG -.->|"Loads at startup"| OUTPUT_FLOW

    style INPUT_FLOW fill:#1e3a5f,stroke:#3b82f6,color:#e2e8f0
    style CORE fill:#2d2a1a,stroke:#f59e0b,color:#e2e8f0
    style OUTPUT_FLOW fill:#1a2e1a,stroke:#22c55e,color:#e2e8f0
    style CONFIG fill:#2d1b3d,stroke:#ec4899,color:#e2e8f0
```

---

## 📋 Quick Reference: Copy-Paste Guide

| Diagram | What it shows | Best for |
|---|---|---|
| **Diagram 1** | Full system topology with all components | README, project overview |
| **Diagram 2** | Request lifecycle for sending a message | Technical documentation |
| **Diagram 3** | 4-layer memory architecture | Feature explanation |
| **Diagram 4** | Async worker pipeline | DevOps / infrastructure docs |
| **Diagram 5** | NeMo Guardrails integration | Security / safety documentation |

> 💡 **Tip:** Paste any code block into [mermaid.live](https://mermaid.live) to preview and export as SVG/PNG.
