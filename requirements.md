# Requirements Document

## Project Title
From Diagnosis to Financial Support: An AI Workflow Copilot for Rural Healthcare

## Core Features

### 1. Simple Clinical Data Uploading

**Requirements**:
- Support text-based data input as the initial implementation
- Design architecture to accommodate future enhancements:
  - Form-based structured input
  - Excel/CSV file uploads
  - Integration with existing hospital management systems
- Secure storage of patient information.


### 2. Patient-Friendly Explanation & Language Translation

**Requirements**:
- Extract key medical information from clinical data
- Generate simplified explanations of:
  - Diagnosis
  - Treatment plan
  - Prognosis
  - Required procedures
- Translate explanations into regional languages on demand.

### 3. Financial Eligibility Matching Module

**Requirements**:
- Maintain comprehensive database of schemes including:
  - Central government schemes (e.g., Ayushman Bharat, PM-JAY)
  - State-level healthcare schemes
  - Non-governmental organization (NGO) programs
  - Private foundation assistance programs
- Implement rule-based matching logic based on:
  - Patient demographics (age, income, location)
  - Medical condition and diagnosis
  - Treatment costs
  - Scheme-specific eligibility criteria
- Display matched schemes with eligibility.

### 4. Claim Assisting PDF Generator

**Requirements**:
- Generate after inclusion of the following sections in generated PDF:
  - Patient demographic information
  - Diagnosis summary (in medical and simplified language)
  - Treatment details and estimated costs
  - List of eligible schemes with details
  - Supporting documentation checklist
- Format PDF for professional presentation to hospitals and support centers

## Technical Architecture

### Frontend

**Technology**: React.js

### Backend and Orchestration

**1. AWS Lambda**:
- Serverless functions for core workflow logic

**2. Amazon S3**:
- Store uploaded clinical data securely
- Store generated PDF documents

### AI and Language Processing

**1. Amazon Bedrock**:
- Extract structured information from clinical text
- Generate simplified medical explanations
- Summarize diagnosis and treatment plans

**2. Amazon Translate**:
- Translate medical explanations to regional languages

### Financial Eligibility Engine

**1. Custom Rule-Based Logic** (Python/Node.js)

**2. Amazon DynamoDB**:
- Store government and non-gov scheme information

## Non-Functional Requirements

### Performance
- Process clinical data and generate results within 30 seconds
- Support concurrent users (minimum 50 simultaneous sessions)
- PDF generation within 10 seconds

### Scalability
- Design for horizontal scaling to support multiple healthcare facilities
- Handle increasing scheme database without performance degradation

### Usability
- Minimal training required for healthcare workers
- Clear error messages and guidance
- Support for low-bandwidth environments

