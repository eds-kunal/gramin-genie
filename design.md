# Design Document

## Project Title
From Diagnosis to Financial Support: An AI Workflow Copilot for Rural Healthcare

---

## 1. System Overview

### Purpose
Automate the workflow from clinical diagnosis to financial assistance claim preparation using AI-powered extraction, simplification, translation, and financial eligibility matching.

---

## 2. System Architecture

### Architecture Diagram

```mermaid
graph LR


%% USER LAYER

subgraph Client_Layer [User Interface]
    A[User uploads data]
end

 

%% =========================
%% STORAGE LAYER
%% =========================
A --> D[Amazon S3 - Data ingestion ]

%% =========================
%% EVENT TRIGGER
%% =========================
D -- S3 Event Trigger --> E[AWS Lambda -Processor]

%% =========================
%% AI PROCESSING
%% =========================
subgraph AI_Intelligence [AI Processing Layer]
    E --> F[Amazon Bedrock - Summary & Extraction]
    F --> G[Amazon Translate - Language Translation]
end

%% =========================
%% FINANCIAL ELIGIBILITY
%% =========================
subgraph Financial_Module [Financial Eligibility Engine]
    E --> H[Eligibility Rules Engine]
    H --> I[(Scheme Database - DynamoDB)]
end

%% PDF GENERATION

subgraph Report_Generation [Report & Claim Assistance]
    G --> J[Lambda - PDF Generator]
    H --> J
    J --> K[Amazon S3 - Reports]
end

%% =========================
%% DELIVERY
%% =========================
K --> L[Downloadable Claim Assistance PDF]

%% =========================
%% Styling
%% =========================
style E fill:#ff9900,stroke:#232f3e,color:#fff
style F fill:#116611,color:#fff
style G fill:#116611,color:#fff
style H fill:#1f77b4,color:#fff
style J fill:#aa00aa,color:#fff
```



## 3. Data Flow & Processing Logic

### Workflow Flowchart

```mermaid
flowchart LR
    A[Start] --> B[User Uploads Data]
    B --> D[Extraction and Patient-Friendly Summarization]
    D --> E{Translate to Regional Language?}
    E -- Yes --> F[Translate Summary]
    E -- No --> G[Keep Original Language]
    F --> H{Check Financial Eligibility?}
    G --> H
    H -- Yes --> I[Match Patient with Eligible Schemes]
    H -- No --> J[Generate Clinical Summary Report]
    I --> J[Generate Downloadable Assistance Report]
    J --> M[End]
```



## 4. Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Frontend | React.js | User interface |
| API Gateway | AWS API Gateway | Request routing |
| Compute | AWS Lambda | Serverless functions |
| Storage | Amazon S3 | File storage |
| Database | Amazon DynamoDB | NoSQL database |
| AI - Extraction | Amazon Bedrock | Clinical data processing |
| AI - Translation | Amazon Translate | Language translation |
| PDF Generation | ReportLab (Python) | PDF creation |

---


## 5. Workflow design logic

Extraction: Structured data extracted from raw doctor notes via Amazon Bedrock.

Simplify & Translate: Turn jargon into simple understandable language and also translate into regional languages if required.

Match Financial Shemes: Rule-based engine maps patient profile to gov and non-gov schemes for financial aid.

Generate: One-click claim-ready PDF.



**Document Version**: 1.0  
**Last Updated**: February 15, 2026  
**Status**: Design Document for Prototype Implementation
