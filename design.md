# Design Document

## Project Title
From Diagnosis to Financial Support: An AI Workflow Copilot for Rural Healthcare

---

## 1. System Overview

### Purpose
This system automates the workflow from clinical diagnosis documentation to financial assistance claim preparation for rural healthcare facilities. It leverages AI to extract, simplify, and translate medical information while matching patients with eligible financial schemes.

### Key Objectives
- Transform unstructured clinical notes into structured, actionable data
- Bridge communication gap between medical professionals and patients
- Automate financial scheme eligibility assessment
- Generate claim-ready documentation with minimal manual effort
- Operate efficiently in resource-constrained rural environments

### Target Users
- Primary: Doctors, Nurses, Healthcare Workers
- Secondary: Patients and their families
- Tertiary: Hospital administrators, scheme coordinators

---

## 2. High-Level Architecture

### Architecture Diagram (Conceptual)

```
┌─────────────────────────────────────────────────────────────────┐
│                         FRONTEND LAYER                          │
│                    React.js Web Application                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │ Data Upload  │  │  Results     │  │ PDF Download │         │
│  │   Screen     │  │  Dashboard   │  │    Screen    │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
└────────────────────────────┬────────────────────────────────────┘
                             │ HTTPS/REST API
┌────────────────────────────┴────────────────────────────────────┐
│                      API GATEWAY LAYER                          │
│                    Amazon API Gateway                           │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────┴────────────────────────────────────┐
│                    BACKEND ORCHESTRATION                        │
│                       AWS Lambda Functions                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │   Ingestion  │  │  Processing  │  │     PDF      │         │
│  │   Handler    │  │ Orchestrator │  │  Generator   │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
└─────────┬────────────────┬────────────────┬────────────────────┘
          │                │                │
┌─────────┴────────────────┴────────────────┴────────────────────┐
│                      STORAGE LAYER                              │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              Amazon S3 Buckets                           │  │
│  │  • clinical-data-bucket (input data)                     │  │
│  │  • generated-pdfs-bucket (output PDFs)                   │  │
│  └──────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              Amazon DynamoDB                             │  │
│  │  • schemes-table (financial schemes database)            │  │
│  │  • patient-records-table (processing metadata)           │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
          │                │                │
┌─────────┴────────────────┴────────────────┴────────────────────┐
│                      AI SERVICES LAYER                          │
│  ┌──────────────────────┐  ┌──────────────────────┐           │
│  │   Amazon Bedrock     │  │  Amazon Translate    │           │
│  │  • Data Extraction   │  │  • Language          │           │
│  │  • Summarization     │  │    Translation       │           │
│  │  • Simplification    │  │                      │           │
│  └──────────────────────┘  └──────────────────────┘           │
└─────────────────────────────────────────────────────────────────┘
```

### Architecture Principles
- **Serverless-First**: Minimize infrastructure management using AWS Lambda
- **Event-Driven**: Asynchronous processing for better scalability
- **Modular Design**: Independent components for easier maintenance
- **Cost-Effective**: Pay-per-use model suitable for variable rural healthcare loads
- **Secure by Default**: Encryption at rest and in transit

---

## 3. Component Design

### 3.1 Frontend Component (React.js)

#### Technology Stack
- React.js 18+
- React Router for navigation
- Axios for API calls
- Material-UI or Tailwind CSS for styling
- React-PDF for PDF preview

#### Key Components

**A. DataUploadComponent**
```javascript
// Responsibilities:
// - Accept text input for clinical data
// - Validate input format
// - Display upload progress
// - Handle errors gracefully

State Management:
- clinicalText: string
- uploadStatus: 'idle' | 'uploading' | 'success' | 'error'
- uploadProgress: number
```

**B. ResultsDashboardComponent**
```javascript
// Responsibilities:
// - Display extracted clinical information
// - Show simplified explanation
// - Present matched financial schemes
// - Provide language selection for translation

State Management:
- extractedData: object
- simplifiedExplanation: string
- matchedSchemes: array
- selectedLanguage: string
- translatedContent: string
```

**C. PDFPreviewComponent**
```javascript
// Responsibilities:
// - Preview generated PDF
// - Download PDF functionality
// - Share options

State Management:
- pdfUrl: string
- downloadStatus: 'ready' | 'downloading' | 'complete'
```

#### State Management
- Use React Context API or Redux for global state
- Store user session, processing status, and results

#### API Integration Layer
```javascript
// services/api.js
const API_BASE_URL = process.env.REACT_APP_API_URL;

export const uploadClinicalData = async (data) => {
  // POST /api/upload
};

export const getProcessingStatus = async (requestId) => {
  // GET /api/status/{requestId}
};

export const translateContent = async (text, targetLanguage) => {
  // POST /api/translate
};

export const downloadPDF = async (requestId) => {
  // GET /api/pdf/{requestId}
};
```

---

### 3.2 Backend Orchestration (AWS Lambda)

#### Lambda Function Architecture

**A. IngestionHandler Lambda**
```python
# Function: clinical-data-ingestion
# Trigger: API Gateway POST /api/upload
# Runtime: Python 3.11

def lambda_handler(event, context):
    """
    1. Validate incoming clinical data
    2. Generate unique request ID
    3. Store raw data in S3
    4. Create record in DynamoDB
    5. Trigger ProcessingOrchestrator
    6. Return request ID to client
    """
    
    # Input validation
    # S3 upload
    # DynamoDB record creation
    # Async invocation of next Lambda
    # Response generation
```

**B. ProcessingOrchestrator Lambda**
```python
# Function: clinical-processing-orchestrator
# Trigger: Async invocation from IngestionHandler
# Runtime: Python 3.11

def lambda_handler(event, context):
    """
    1. Retrieve clinical data from S3
    2. Call Bedrock for extraction
    3. Call Bedrock for simplification
    4. Call eligibility matching logic
    5. Update DynamoDB with results
    6. Trigger PDF generation
    """
    
    # Orchestration logic
    # Error handling and retry logic
    # Status updates
```

**C. EligibilityMatcher Lambda**
```python
# Function: financial-eligibility-matcher
# Trigger: Invoked by ProcessingOrchestrator
# Runtime: Python 3.11

def lambda_handler(event, context):
    """
    1. Extract patient demographics and diagnosis
    2. Query DynamoDB for all schemes
    3. Apply rule-based matching logic
    4. Calculate eligibility scores
    5. Return ranked list of schemes
    """
    
    # Rule engine implementation
    # Scheme matching algorithms
```

**D. TranslationHandler Lambda**
```python
# Function: language-translation-handler
# Trigger: API Gateway POST /api/translate
# Runtime: Python 3.11

def lambda_handler(event, context):
    """
    1. Receive text and target language
    2. Call Amazon Translate
    3. Return translated content
    """
    
    # Translation service integration
```

**E. PDFGenerator Lambda**
```python
# Function: pdf-generator
# Trigger: Async invocation from ProcessingOrchestrator
# Runtime: Python 3.11
# Layers: ReportLab or WeasyPrint

def lambda_handler(event, context):
    """
    1. Retrieve processed data from DynamoDB
    2. Format data into PDF template
    3. Generate PDF document
    4. Upload to S3
    5. Update DynamoDB with PDF URL
    """
    
    # PDF generation logic
    # Template rendering
```

#### Lambda Configuration
```yaml
Memory: 512 MB - 1024 MB
Timeout: 30 seconds (processing), 60 seconds (PDF generation)
Environment Variables:
  - S3_BUCKET_DATA
  - S3_BUCKET_PDF
  - DYNAMODB_SCHEMES_TABLE
  - DYNAMODB_RECORDS_TABLE
  - BEDROCK_MODEL_ID
```

---

### 3.3 AI Layer Design

#### Amazon Bedrock Integration

**A. Clinical Data Extraction**
```python
# Use Case: Extract structured information from unstructured text

import boto3
import json

bedrock_runtime = boto3.client('bedrock-runtime', region_name='us-east-1')

def extract_clinical_data(clinical_text):
    """
    Extract: patient demographics, diagnosis, symptoms, 
    treatment plan, estimated costs
    """
    
    prompt = f"""
    Extract the following information from the clinical notes:
    - Patient Age
    - Gender
    - Primary Diagnosis
    - Secondary Diagnoses
    - Symptoms
    - Prescribed Treatment
    - Estimated Treatment Cost
    - Urgency Level
    
    Clinical Notes:
    {clinical_text}
    
    Return as JSON format.
    """
    
    request_body = {
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 2000,
        "messages": [
            {
                "role": "user",
                "content": prompt
            }
        ]
    }
    
    response = bedrock_runtime.invoke_model(
        modelId='anthropic.claude-3-sonnet-20240229-v1:0',
        body=json.dumps(request_body)
    )
    
    # Parse and return structured data
```

**B. Patient-Friendly Simplification**
```python
def simplify_medical_explanation(extracted_data):
    """
    Convert medical jargon to simple language
    """
    
    prompt = f"""
    Explain the following medical information in simple, 
    easy-to-understand language suitable for patients 
    with limited medical knowledge:
    
    Diagnosis: {extracted_data['diagnosis']}
    Treatment: {extracted_data['treatment']}
    
    Use simple words, avoid medical jargon, and be empathetic.
    """
    
    # Similar Bedrock invocation
    # Return simplified text
```

#### Amazon Translate Integration

```python
import boto3

translate_client = boto3.client('translate', region_name='us-east-1')

def translate_to_regional_language(text, target_language_code):
    """
    Translate simplified explanation to regional language
    
    Supported languages:
    - hi: Hindi
    - ta: Tamil
    - te: Telugu
    - bn: Bengali
    - mr: Marathi
    - gu: Gujarati
    """
    
    response = translate_client.translate_text(
        Text=text,
        SourceLanguageCode='en',
        TargetLanguageCode=target_language_code
    )
    
    return response['TranslatedText']
```

---

### 3.4 Financial Eligibility Module

#### Rule-Based Matching Engine

**Eligibility Criteria Structure**
```python
# eligibility_rules.py

class EligibilityRule:
    def __init__(self, scheme_id, criteria):
        self.scheme_id = scheme_id
        self.criteria = criteria
    
    def evaluate(self, patient_data):
        """
        Evaluate if patient meets scheme criteria
        Returns: (eligible: bool, score: float, reasons: list)
        """
        score = 0
        max_score = 0
        reasons = []
        
        # Age criteria
        if 'age_range' in self.criteria:
            max_score += 20
            if self.criteria['age_range'][0] <= patient_data['age'] <= self.criteria['age_range'][1]:
                score += 20
                reasons.append("Age criteria met")
        
        # Income criteria
        if 'max_income' in self.criteria:
            max_score += 30
            if patient_data.get('annual_income', 0) <= self.criteria['max_income']:
                score += 30
                reasons.append("Income criteria met")
        
        # Diagnosis criteria
        if 'covered_diagnoses' in self.criteria:
            max_score += 30
            if patient_data['diagnosis'] in self.criteria['covered_diagnoses']:
                score += 30
                reasons.append("Diagnosis covered")
        
        # Location criteria
        if 'eligible_states' in self.criteria:
            max_score += 20
            if patient_data.get('state') in self.criteria['eligible_states']:
                score += 20
                reasons.append("Location eligible")
        
        eligibility_percentage = (score / max_score * 100) if max_score > 0 else 0
        
        return {
            'eligible': eligibility_percentage >= 50,
            'score': eligibility_percentage,
            'reasons': reasons
        }
```

**Matching Algorithm**
```python
def match_financial_schemes(patient_data, schemes_list):
    """
    Match patient against all available schemes
    """
    matched_schemes = []
    
    for scheme in schemes_list:
        rule = EligibilityRule(scheme['scheme_id'], scheme['criteria'])
        result = rule.evaluate(patient_data)
        
        if result['eligible']:
            matched_schemes.append({
                'scheme_id': scheme['scheme_id'],
                'scheme_name': scheme['name'],
                'scheme_type': scheme['type'],
                'coverage_amount': scheme['coverage_amount'],
                'eligibility_score': result['score'],
                'match_reasons': result['reasons'],
                'application_process': scheme['application_process'],
                'required_documents': scheme['required_documents']
            })
    
    # Sort by eligibility score
    matched_schemes.sort(key=lambda x: x['eligibility_score'], reverse=True)
    
    return matched_schemes
```

---

### 3.5 PDF Generation Module

#### PDF Template Structure

```python
from reportlab.lib.pagesizes import A4
from reportlab.lib.styles import getSampleStyleSheet, ParagraphStyle
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, Table, TableStyle
from reportlab.lib.units import inch
from reportlab.lib import colors
import io

def generate_claim_assistance_pdf(data):
    """
    Generate comprehensive PDF for claim assistance
    """
    
    buffer = io.BytesIO()
    doc = SimpleDocTemplate(buffer, pagesize=A4)
    story = []
    styles = getSampleStyleSheet()
    
    # Title
    title_style = ParagraphStyle(
        'CustomTitle',
        parent=styles['Heading1'],
        fontSize=24,
        textColor=colors.HexColor('#1a73e8'),
        spaceAfter=30
    )
    story.append(Paragraph("Medical Financial Assistance Report", title_style))
    story.append(Spacer(1, 0.2*inch))
    
    # Patient Information Section
    story.append(Paragraph("Patient Information", styles['Heading2']))
    patient_data = [
        ['Patient ID:', data['patient_id']],
        ['Age:', str(data['age'])],
        ['Gender:', data['gender']],
        ['Location:', data['location']],
        ['Report Date:', data['report_date']]
    ]
    patient_table = Table(patient_data, colWidths=[2*inch, 4*inch])
    patient_table.setStyle(TableStyle([
        ('BACKGROUND', (0, 0), (0, -1), colors.lightgrey),
        ('TEXTCOLOR', (0, 0), (-1, -1), colors.black),
        ('ALIGN', (0, 0), (-1, -1), 'LEFT'),
        ('FONTNAME', (0, 0), (-1, -1), 'Helvetica'),
        ('FONTSIZE', (0, 0), (-1, -1), 10),
        ('BOTTOMPADDING', (0, 0), (-1, -1), 12),
        ('GRID', (0, 0), (-1, -1), 1, colors.black)
    ]))
    story.append(patient_table)
    story.append(Spacer(1, 0.3*inch))
    
    # Clinical Summary Section
    story.append(Paragraph("Clinical Summary", styles['Heading2']))
    story.append(Paragraph(f"<b>Diagnosis:</b> {data['diagnosis']}", styles['Normal']))
    story.append(Spacer(1, 0.1*inch))
    story.append(Paragraph(f"<b>Treatment Plan:</b> {data['treatment']}", styles['Normal']))
    story.append(Spacer(1, 0.1*inch))
    story.append(Paragraph(f"<b>Estimated Cost:</b> ₹{data['estimated_cost']}", styles['Normal']))
    story.append(Spacer(1, 0.3*inch))
    
    # Simplified Explanation Section
    story.append(Paragraph("Patient-Friendly Explanation", styles['Heading2']))
    story.append(Paragraph(data['simplified_explanation'], styles['Normal']))
    story.append(Spacer(1, 0.3*inch))
    
    # Eligible Schemes Section
    story.append(Paragraph("Eligible Financial Assistance Schemes", styles['Heading2']))
    
    for idx, scheme in enumerate(data['matched_schemes'], 1):
        story.append(Paragraph(f"<b>{idx}. {scheme['scheme_name']}</b>", styles['Heading3']))
        scheme_details = [
            ['Scheme Type:', scheme['scheme_type']],
            ['Coverage Amount:', f"₹{scheme['coverage_amount']}"],
            ['Eligibility Score:', f"{scheme['eligibility_score']:.1f}%"],
            ['Match Reasons:', ', '.join(scheme['match_reasons'])]
        ]
        scheme_table = Table(scheme_details, colWidths=[2*inch, 4*inch])
        scheme_table.setStyle(TableStyle([
            ('BACKGROUND', (0, 0), (0, -1), colors.lightblue),
            ('GRID', (0, 0), (-1, -1), 1, colors.black),
            ('FONTSIZE', (0, 0), (-1, -1), 9),
            ('BOTTOMPADDING', (0, 0), (-1, -1), 8)
        ]))
        story.append(scheme_table)
        story.append(Spacer(1, 0.2*inch))
    
    # Required Documents Section
    story.append(Paragraph("Required Documents Checklist", styles['Heading2']))
    all_docs = set()
    for scheme in data['matched_schemes']:
        all_docs.update(scheme['required_documents'])
    
    for doc in all_docs:
        story.append(Paragraph(f"☐ {doc}", styles['Normal']))
    
    story.append(Spacer(1, 0.3*inch))
    
    # Footer
    story.append(Paragraph(
        "This report is generated by AI Workflow Copilot for Rural Healthcare. "
        "Please verify all information with respective scheme authorities.",
        styles['Italic']
    ))
    
    # Build PDF
    doc.build(story)
    buffer.seek(0)
    
    return buffer
```

---

## 4. Data Flow Description

### End-to-End Processing Flow

```
Step 1: Data Upload
┌─────────────────────────────────────────────────────────────┐
│ User uploads clinical text via React frontend              │
│ → POST /api/upload with clinical_text payload              │
└────────────────────────┬────────────────────────────────────┘
                         ↓
Step 2: Ingestion
┌─────────────────────────────────────────────────────────────┐
│ IngestionHandler Lambda receives request                   │
│ → Validates input                                           │
│ → Generates unique request_id (UUID)                        │
│ → Stores raw text in S3: clinical-data/{request_id}.txt    │
│ → Creates DynamoDB record with status='processing'          │
│ → Returns request_id to frontend                            │
└────────────────────────┬────────────────────────────────────┘
                         ↓
Step 3: AI Processing
┌─────────────────────────────────────────────────────────────┐
│ ProcessingOrchestrator Lambda triggered asynchronously     │
│ → Retrieves clinical text from S3                          │
│ → Calls Bedrock for data extraction                        │
│   • Extracts: age, gender, diagnosis, treatment, cost      │
│ → Calls Bedrock for simplification                         │
│   • Generates patient-friendly explanation                 │
│ → Updates DynamoDB with extracted_data and explanation     │
└────────────────────────┬────────────────────────────────────┘
                         ↓
Step 4: Eligibility Matching
┌─────────────────────────────────────────────────────────────┐
│ EligibilityMatcher Lambda invoked                          │
│ → Queries DynamoDB schemes-table for all schemes           │
│ → Applies rule-based matching logic                        │
│ → Calculates eligibility scores                            │
│ → Ranks schemes by score                                   │
│ → Updates DynamoDB with matched_schemes array              │
└────────────────────────┬────────────────────────────────────┘
                         ↓
Step 5: PDF Generation
┌─────────────────────────────────────────────────────────────┐
│ PDFGenerator Lambda triggered                               │
│ → Retrieves all processed data from DynamoDB               │
│ → Formats data using PDF template                          │
│ → Generates PDF document                                   │
│ → Uploads to S3: generated-pdfs/{request_id}.pdf           │
│ → Updates DynamoDB with pdf_url and status='completed'     │
└────────────────────────┬────────────────────────────────────┘
                         ↓
Step 6: Results Retrieval
┌─────────────────────────────────────────────────────────────┐
│ Frontend polls GET /api/status/{request_id}                │
│ → Returns processing status and results when complete      │
│ → Frontend displays results dashboard                      │
│ → User can download PDF via GET /api/pdf/{request_id}      │
└─────────────────────────────────────────────────────────────┘

Optional Step 7: Translation
┌─────────────────────────────────────────────────────────────┐
│ User selects regional language                             │
│ → POST /api/translate with text and target_language        │
│ → TranslationHandler calls Amazon Translate                │
│ → Returns translated content to frontend                   │
└─────────────────────────────────────────────────────────────┘
```

### Detailed Step-by-Step Processing

#### Phase 1: Data Ingestion (0-2 seconds)
1. User submits clinical text through React form
2. Frontend validates non-empty input
3. API Gateway authenticates request (if auth enabled)
4. IngestionHandler Lambda:
   - Generates UUID for tracking
   - Sanitizes input text
   - Uploads to S3 with server-side encryption
   - Creates DynamoDB record:
     ```json
     {
       "request_id": "uuid-123",
       "status": "processing",
       "created_at": "2026-02-15T10:30:00Z",
       "clinical_text_s3_key": "clinical-data/uuid-123.txt"
     }
     ```
   - Invokes ProcessingOrchestrator asynchronously
   - Returns response to frontend:
     ```json
     {
       "request_id": "uuid-123",
       "status": "processing",
       "message": "Clinical data uploaded successfully"
     }
     ```

#### Phase 2: AI Extraction (5-10 seconds)
1. ProcessingOrchestrator retrieves text from S3
2. Calls Amazon Bedrock with extraction prompt
3. Bedrock returns structured JSON:
   ```json
   {
     "patient_age": 45,
     "gender": "Female",
     "primary_diagnosis": "Type 2 Diabetes Mellitus",
     "secondary_diagnoses": ["Hypertension"],
     "symptoms": ["Increased thirst", "Frequent urination"],
     "treatment_plan": "Metformin 500mg twice daily, lifestyle modifications",
     "estimated_cost": 15000,
     "urgency": "moderate"
   }
   ```
4. Calls Bedrock again for simplification
5. Receives patient-friendly explanation
6. Updates DynamoDB with extracted data

#### Phase 3: Eligibility Assessment (2-3 seconds)
1. EligibilityMatcher retrieves patient data
2. Scans DynamoDB schemes-table
3. For each scheme, evaluates:
   - Age eligibility
   - Income eligibility (if available)
   - Diagnosis coverage
   - Geographic eligibility
4. Calculates weighted score
5. Filters schemes with score ≥ 50%
6. Sorts by score descending
7. Updates DynamoDB with results

#### Phase 4: PDF Creation (5-8 seconds)
1. PDFGenerator retrieves complete data
2. Renders PDF with all sections
3. Uploads to S3 with public-read or signed URL
4. Updates DynamoDB with final status

#### Phase 5: Frontend Display (immediate)
1. Frontend polls status endpoint every 2 seconds
2. When status='completed', displays results
3. Shows extracted data, explanation, schemes
4. Provides PDF download link

---

## 5. Database Design

### DynamoDB Schema

#### Table 1: patient-records-table

**Purpose**: Track processing status and store results

**Primary Key**: 
- Partition Key: `request_id` (String)

**Attributes**:
```json
{
  "request_id": "uuid-123",
  "status": "completed | processing | failed",
  "created_at": "2026-02-15T10:30:00Z",
  "updated_at": "2026-02-15T10:31:00Z",
  "clinical_text_s3_key": "clinical-data/uuid-123.txt",
  "extracted_data": {
    "patient_age": 45,
    "gender": "Female",
    "primary_diagnosis": "Type 2 Diabetes Mellitus",
    "secondary_diagnoses": ["Hypertension"],
    "symptoms": ["Increased thirst", "Frequent urination"],
    "treatment_plan": "Metformin 500mg twice daily",
    "estimated_cost": 15000,
    "urgency": "moderate"
  },
  "simplified_explanation": "You have been diagnosed with...",
  "matched_schemes": [
    {
      "scheme_id": "PMJAY-001",
      "scheme_name": "Pradhan Mantri Jan Arogya Yojana",
      "eligibility_score": 85.5,
      "coverage_amount": 500000
    }
  ],
  "pdf_s3_key": "generated-pdfs/uuid-123.pdf",
  "pdf_url": "https://s3.amazonaws.com/...",
  "error_message": null
}
```

**Indexes**:
- GSI: `status-created_at-index` for querying by status

**Capacity**:
- On-Demand pricing for variable load
- Or Provisioned: 5 RCU, 5 WCU for prototype

---

#### Table 2: schemes-table

**Purpose**: Store financial assistance scheme information

**Primary Key**:
- Partition Key: `scheme_id` (String)

**Attributes**:
```json
{
  "scheme_id": "PMJAY-001",
  "scheme_name": "Pradhan Mantri Jan Arogya Yojana",
  "scheme_type": "Central Government",
  "description": "Health insurance scheme for economically vulnerable families",
  "coverage_amount": 500000,
  "criteria": {
    "age_range": [0, 100],
    "max_income": 100000,
    "covered_diagnoses": [
      "Type 2 Diabetes Mellitus",
      "Hypertension",
      "Cancer",
      "Cardiac diseases"
    ],
    "eligible_states": ["All"],
    "family_size_max": 10
  },
  "application_process": "Visit nearest Ayushman Bharat center with documents",
  "required_documents": [
    "Aadhaar Card",
    "Income Certificate",
    "Medical Records",
    "BPL Card"
  ],
  "contact_info": {
    "website": "https://pmjay.gov.in",
    "helpline": "14555",
    "email": "support@pmjay.gov.in"
  },
  "active": true,
  "last_updated": "2026-01-15T00:00:00Z"
}
```

**Sample Schemes to Populate**:
1. Pradhan Mantri Jan Arogya Yojana (PM-JAY)
2. State-specific schemes (e.g., Karnataka Arogya Bhagya)
3. NGO schemes (e.g., Tata Memorial Hospital schemes)
4. Private foundation schemes

**Indexes**:
- GSI: `scheme_type-index` for filtering by type
- GSI: `active-index` for querying active schemes only

**Capacity**:
- On-Demand or Provisioned: 5 RCU, 2 WCU (mostly read-heavy)

---

## 6. API Design

### REST API Endpoints

**Base URL**: `https://api.healthcare-copilot.com/v1`

---

#### 1. Upload Clinical Data

**Endpoint**: `POST /api/upload`

**Request**:
```json
{
  "clinical_text": "Patient is a 45-year-old female presenting with...",
  "metadata": {
    "facility_id": "RURAL_CLINIC_001",
    "uploaded_by": "Dr. Smith"
  }
}
```

**Response** (202 Accepted):
```json
{
  "request_id": "uuid-123",
  "status": "processing",
  "message": "Clinical data uploaded successfully",
  "estimated_completion_time": "30 seconds"
}
```

**Error Response** (400 Bad Request):
```json
{
  "error": "Invalid input",
  "message": "clinical_text field is required"
}
```

---

#### 2. Get Processing Status

**Endpoint**: `GET /api/status/{request_id}`

**Response** (200 OK - Processing):
```json
{
  "request_id": "uuid-123",
  "status": "processing",
  "progress": {
    "current_step": "eligibility_matching",
    "steps_completed": 2,
    "total_steps": 4
  }
}
```

**Response** (200 OK - Completed):
```json
{
  "request_id": "uuid-123",
  "status": "completed",
  "results": {
    "extracted_data": { ... },
    "simplified_explanation": "You have been diagnosed...",
    "matched_schemes": [ ... ],
    "pdf_url": "https://s3.amazonaws.com/..."
  }
}
```

**Error Response** (404 Not Found):
```json
{
  "error": "Request not found",
  "message": "No record found for request_id: uuid-123"
}
```

---

#### 3. Translate Content

**Endpoint**: `POST /api/translate`

**Request**:
```json
{
  "text": "You have been diagnosed with Type 2 Diabetes...",
  "target_language": "hi",
  "source_language": "en"
}
```

**Supported Language Codes**:
- `hi`: Hindi
- `ta`: Tamil
- `te`: Telugu
- `bn`: Bengali
- `mr`: Marathi
- `gu`: Gujarati

**Response** (200 OK):
```json
{
  "translated_text": "आपको टाइप 2 मधुमेह का निदान किया गया है...",
  "source_language": "en",
  "target_language": "hi"
}
```

---

#### 4. Download PDF

**Endpoint**: `GET /api/pdf/{request_id}`

**Response** (200 OK):
- Content-Type: `application/pdf`
- Content-Disposition: `attachment; filename="claim_report_{request_id}.pdf"`
- Binary PDF data

**Alternative**: Return signed S3 URL
```json
{
  "pdf_url": "https://s3.amazonaws.com/generated-pdfs/uuid-123.pdf?signature=...",
  "expires_in": 3600
}
```

---

#### 5. List Available Schemes (Optional)

**Endpoint**: `GET /api/schemes`

**Query Parameters**:
- `type`: Filter by scheme type (government, ngo, private)
- `active`: Filter active schemes only (true/false)

**Response** (200 OK):
```json
{
  "schemes": [
    {
      "scheme_id": "PMJAY-001",
      "scheme_name": "Pradhan Mantri Jan Arogya Yojana",
      "scheme_type": "Central Government",
      "coverage_amount": 500000
    }
  ],
  "total_count": 15
}
```

---

### API Gateway Configuration

**Throttling**:
- Rate: 100 requests per second
- Burst: 200 requests

**CORS Configuration**:
```json
{
  "allowOrigins": ["https://healthcare-copilot.com"],
  "allowMethods": ["GET", "POST", "OPTIONS"],
  "allowHeaders": ["Content-Type", "Authorization"],
  "maxAge": 3600
}
```

**Authentication** (Optional for prototype):
- API Key authentication
- Or AWS Cognito for user management

---

## 7. Scalability & Performance Considerations

### Scalability Strategy

#### Horizontal Scaling
- **Lambda Auto-Scaling**: Automatically scales to handle concurrent requests
  - Default: 1000 concurrent executions per region
  - Can request increase for production
- **DynamoDB On-Demand**: Automatically scales read/write capacity
- **S3**: Inherently scalable for storage

#### Performance Optimizations

**1. Lambda Optimization**
- Use Lambda layers for shared dependencies (ReportLab, boto3)
- Implement connection pooling for DynamoDB
- Reuse execution contexts (global variables for clients)
- Optimize memory allocation (1024 MB recommended)

**2. Caching Strategy**
- Cache scheme data in Lambda memory (refresh every 1 hour)
- Use CloudFront for PDF distribution
- Cache translated content for common phrases

**3. Asynchronous Processing**
- Use Lambda async invocation for non-blocking operations
- Implement SQS for reliable message queuing (future enhancement)
- Use Step Functions for complex workflows (future enhancement)

**4. Database Optimization**
- Use DynamoDB batch operations where possible
- Implement efficient query patterns with GSIs
- Use projection expressions to fetch only required attributes

### Performance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Upload Response Time | < 2 seconds | API Gateway to Lambda |
| Total Processing Time | < 30 seconds | Upload to PDF ready |
| PDF Generation | < 10 seconds | Data to PDF |
| Translation | < 3 seconds | Per request |
| API Availability | 99.5% | Monthly uptime |
| Concurrent Users | 50+ | Simultaneous processing |

### Load Testing Recommendations
- Use Apache JMeter or Locust for load testing
- Test scenarios:
  - 10 concurrent uploads
  - 50 concurrent status checks
  - 20 concurrent PDF downloads

---

## 8. Security & Data Handling

### Data Security Measures

#### 1. Data Encryption
**At Rest**:
- S3: Server-side encryption (SSE-S3 or SSE-KMS)
- DynamoDB: Encryption at rest enabled
- Lambda environment variables: Encrypted with KMS

**In Transit**:
- HTTPS/TLS 1.2+ for all API communications
- Signed URLs for S3 access

#### 2. Access Control
**IAM Policies**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::clinical-data-bucket/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:UpdateItem",
        "dynamodb:Query"
      ],
      "Resource": "arn:aws:dynamodb:region:account:table/patient-records-table"
    }
  ]
}
```

**Principle of Least Privilege**:
- Each Lambda has minimal required permissions
- S3 buckets have restricted access
- DynamoDB tables use fine-grained access control

#### 3. Data Privacy

**Synthetic/Public Data Usage**:
- For prototype/hackathon: Use only synthetic or anonymized data
- No real patient information (PHI/PII)
- Generate test data using Faker library

**Data Retention**:
- Clinical data: 30 days retention policy
- PDFs: 90 days retention policy
- Automatic deletion using S3 lifecycle policies

**Anonymization**:
- Remove or mask identifiable information
- Use patient IDs instead of names
- Redact sensitive details in logs

#### 4. Audit Logging
- Enable CloudTrail for API calls
- Log all data access events
- Monitor suspicious activities with CloudWatch

#### 5. Input Validation
- Sanitize all user inputs
- Validate file sizes (max 10 MB for text)
- Prevent injection attacks
- Rate limiting to prevent abuse

### Compliance Considerations

**For Production** (Not required for prototype):
- HIPAA compliance for healthcare data
- GDPR compliance for EU users
- Indian data protection laws
- Regular security audits

---

## 9. Future Enhancements

### Phase 2 Enhancements (3-6 months)

#### 1. Multi-Modal Input Support
- **Form-Based Input**: Structured forms for clinical data entry
  - Pre-defined fields for common diagnoses
  - Dropdown menus for standardized inputs
  - Auto-complete for medications and procedures
- **Excel/CSV Upload**: Bulk patient data processing
  - Template-based Excel sheets
  - Batch processing capabilities
  - Progress tracking for bulk uploads
- **OCR Integration**: Scan handwritten prescriptions
  - Amazon Textract for document scanning
  - Handwriting recognition
  - Validation and correction interface

#### 2. Enhanced AI Capabilities
- **Multi-Language Support**: Expand beyond major Indian languages
  - Support for 20+ regional languages
  - Dialect-specific translations
- **Voice Input**: Voice-to-text for clinical notes
  - Amazon Transcribe Medical integration
  - Support for medical terminology
  - Multi-speaker recognition
- **Predictive Analytics**: Suggest likely schemes before full processing
  - Machine learning model for scheme prediction
  - Historical data analysis
  - Recommendation engine

#### 3. Integration Capabilities
- **Hospital Management System (HMS) Integration**
  - HL7/FHIR standard support
  - Real-time data sync
  - Bidirectional communication
- **Government Scheme APIs**
  - Direct integration with PM-JAY portal
  - Real-time eligibility verification
  - Automated application submission
- **Payment Gateway Integration**
  - Track claim status
  - Payment reconciliation
  - Financial reporting

#### 4. Mobile Application
- **React Native App**: Mobile version for field workers
  - Offline mode support
  - Camera integration for document capture
  - Push notifications for status updates
- **Progressive Web App (PWA)**: Lightweight mobile experience
  - Works on low-bandwidth networks
  - Installable on mobile devices

#### 5. Advanced Features
- **Patient Portal**: Self-service for patients
  - View their own reports
  - Track application status
  - Upload additional documents
- **Analytics Dashboard**: For administrators
  - Usage statistics
  - Success rate metrics
  - Scheme utilization reports
  - Geographic distribution analysis
- **Chatbot Assistant**: AI-powered help
  - Answer FAQs about schemes
  - Guide through application process
  - Multi-language support

#### 6. Workflow Automation
- **AWS Step Functions**: Complex workflow orchestration
  - Better error handling
  - Retry mechanisms
  - Visual workflow monitoring
- **Event-Driven Architecture**: Real-time notifications
  - SNS for email/SMS notifications
  - EventBridge for event routing
  - WebSocket for real-time updates

#### 7. Machine Learning Enhancements
- **Custom ML Models**: Train on historical data
  - Improve scheme matching accuracy
  - Predict approval likelihood
  - Optimize document requirements
- **Feedback Loop**: Learn from outcomes
  - Track successful applications
  - Refine eligibility rules
  - Improve AI prompts

### Phase 3 Enhancements (6-12 months)

#### 1. Blockchain Integration
- Immutable record keeping
- Secure document verification
- Transparent claim tracking

#### 2. Telemedicine Integration
- Video consultation support
- Remote diagnosis assistance
- Specialist referral system

#### 3. IoT Device Integration
- Wearable health device data
- Remote patient monitoring
- Real-time health metrics

#### 4. Advanced Security
- Biometric authentication
- Multi-factor authentication
- Zero-trust architecture

---

## 10. Assumptions & Limitations

### Assumptions

#### Technical Assumptions
1. **AWS Services Availability**: All required AWS services are available in the deployment region
2. **Internet Connectivity**: Rural healthcare facilities have basic internet access (minimum 2 Mbps)
3. **Browser Compatibility**: Users have access to modern web browsers (Chrome, Firefox, Safari, Edge)
4. **Data Format**: Clinical notes are in English or can be translated to English for processing
5. **Scheme Data Accuracy**: Financial scheme information is manually curated and kept up-to-date
6. **Synthetic Data**: Prototype uses synthetic/anonymized data only

#### Business Assumptions
1. **User Training**: Healthcare workers receive basic training on system usage
2. **Scheme Participation**: Government and NGO schemes are willing to be listed
3. **Legal Compliance**: System usage complies with local healthcare regulations
4. **Cost Model**: AWS costs are acceptable for the target user base
5. **Adoption Rate**: Healthcare facilities are willing to adopt digital solutions

#### Data Assumptions
1. **Clinical Note Quality**: Uploaded clinical notes contain sufficient information for extraction
2. **Patient Consent**: Patients consent to data processing for scheme matching
3. **Scheme Eligibility**: Scheme criteria can be represented as rule-based logic
4. **Document Availability**: Patients can provide required documents for applications

### Limitations

#### Technical Limitations

**1. AI Accuracy**
- Bedrock extraction may not be 100% accurate for complex medical cases
- Simplification may lose some medical nuance
- Translation quality varies by language and medical terminology
- Requires human review for critical decisions

**2. Text-Only Input (Phase 1)**
- No support for structured forms initially
- No image or document upload
- Manual text entry can be time-consuming
- Prone to typos and formatting issues

**3. Rule-Based Matching**
- Cannot handle complex eligibility criteria requiring human judgment
- May miss schemes with nuanced requirements
- Requires manual rule updates when schemes change
- No learning from historical outcomes initially

**4. Processing Time**
- 20-30 seconds total processing time
- Not suitable for emergency situations
- Requires stable internet connection throughout
- No offline mode in Phase 1

**5. Language Support**
- Limited to major Indian languages initially
- Medical terminology translation may be imperfect
- Regional dialects not fully supported
- English required for initial processing

#### Functional Limitations

**1. Scheme Coverage**
- Only includes schemes in the database
- Manual updates required for new schemes
- May not cover all state-specific or local schemes
- Private insurance schemes may be limited

**2. Eligibility Verification**
- System provides suggestions, not guarantees
- Final eligibility determined by scheme authorities
- Cannot verify income or other documents
- No real-time integration with government databases (Phase 1)

**3. Application Submission**
- PDF generation only, no direct submission
- Manual submission to scheme authorities required
- No tracking of application status
- No integration with scheme portals initially

**4. Data Privacy**
- Prototype uses synthetic data only
- Not HIPAA compliant in Phase 1
- Limited audit capabilities
- No patient consent management system

#### Operational Limitations

**1. Scalability**
- Designed for small to medium rural facilities
- May require optimization for large hospitals
- Concurrent user limit based on AWS quotas
- Cost increases with usage

**2. Maintenance**
- Requires regular scheme database updates
- AI model prompts may need refinement
- Manual monitoring of processing quality
- Dependency on AWS service availability

**3. User Support**
- Limited to documentation and basic training
- No 24/7 support in prototype phase
- Requires technical knowledge for troubleshooting
- No dedicated helpdesk

**4. Integration**
- No HMS integration in Phase 1
- Manual data entry required
- No automated follow-up system
- Limited reporting capabilities

### Risk Mitigation

**Technical Risks**:
- Regular testing of AI accuracy
- Fallback mechanisms for service failures
- Comprehensive error handling
- Performance monitoring and optimization

**Data Risks**:
- Use only synthetic data for prototype
- Implement encryption and access controls
- Regular security audits
- Clear data retention policies

**Operational Risks**:
- User training programs
- Clear documentation
- Phased rollout approach
- Feedback collection and iteration

---

## Appendix

### A. Technology Stack Summary

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Frontend | React.js 18+ | User interface |
| UI Framework | Material-UI / Tailwind | Styling and components |
| API Gateway | AWS API Gateway | REST API management |
| Compute | AWS Lambda | Serverless functions |
| Storage | Amazon S3 | File storage |
| Database | Amazon DynamoDB | NoSQL database |
| AI - Extraction | Amazon Bedrock | Clinical data processing |
| AI - Translation | Amazon Translate | Language translation |
| PDF Generation | ReportLab (Python) | PDF creation |
| Monitoring | CloudWatch | Logging and metrics |
| Security | AWS IAM, KMS | Access control and encryption |

### B. Development Timeline (Hackathon/Prototype)

**Day 1-2: Setup & Core Backend**
- AWS account setup and service configuration
- Lambda functions for ingestion and orchestration
- S3 bucket creation
- DynamoDB table setup

**Day 3-4: AI Integration**
- Bedrock integration for extraction
- Bedrock integration for simplification
- Amazon Translate integration
- Testing with sample clinical notes

**Day 5-6: Eligibility Engine**
- Rule-based matching logic implementation
- Scheme database population
- Testing matching algorithms

**Day 7-8: PDF Generation**
- PDF template design
- PDF generation Lambda
- S3 upload and URL generation

**Day 9-10: Frontend Development**
- React app setup
- Upload interface
- Results dashboard
- PDF download functionality

**Day 11-12: Integration & Testing**
- End-to-end testing
- Bug fixes
- Performance optimization
- Documentation

### C. Sample Test Data

**Sample Clinical Note**:
```
Patient: 45-year-old female
Chief Complaint: Increased thirst and frequent urination for 3 months
History: No prior history of diabetes. Family history of Type 2 Diabetes.
Examination: BMI 28, BP 140/90
Lab Results: Fasting glucose 180 mg/dL, HbA1c 8.5%
Diagnosis: Type 2 Diabetes Mellitus, Hypertension
Treatment Plan: Metformin 500mg BD, Lifestyle modifications, Diet counseling
Estimated Cost: ₹15,000 for initial 3 months treatment
```

**Expected Extraction**:
```json
{
  "patient_age": 45,
  "gender": "Female",
  "primary_diagnosis": "Type 2 Diabetes Mellitus",
  "secondary_diagnoses": ["Hypertension"],
  "estimated_cost": 15000
}
```

**Expected Simplified Explanation**:
```
You have been diagnosed with Type 2 Diabetes, which means your body 
has difficulty controlling blood sugar levels. You also have high 
blood pressure. Your doctor has prescribed medicine called Metformin 
to help control your blood sugar. You will need to make some changes 
to your diet and lifestyle. The treatment will cost approximately 
₹15,000 for the first three months.
```

### D. Deployment Checklist

- [ ] AWS account created and configured
- [ ] IAM roles and policies set up
- [ ] S3 buckets created with encryption
- [ ] DynamoDB tables created
- [ ] Lambda functions deployed
- [ ] API Gateway configured
- [ ] Bedrock access enabled
- [ ] Translate service enabled
- [ ] Frontend deployed (S3 + CloudFront or Amplify)
- [ ] Environment variables configured
- [ ] Scheme database populated
- [ ] Testing completed
- [ ] Documentation finalized
- [ ] Demo prepared

### E. Cost Estimation (Monthly - Prototype)

| Service | Usage | Estimated Cost |
|---------|-------|----------------|
| Lambda | 10,000 invocations | $0.20 |
| S3 | 10 GB storage, 1000 requests | $0.50 |
| DynamoDB | On-Demand, 1M reads, 100K writes | $1.50 |
| Bedrock | 100K tokens | $3.00 |
| Translate | 100K characters | $1.50 |
| API Gateway | 10,000 requests | $0.35 |
| **Total** | | **~$7.05/month** |

*Note: Costs scale with usage. Production costs will be higher.*

---

## Conclusion

This design document provides a comprehensive blueprint for building an AI-powered workflow copilot for rural healthcare. The system leverages modern cloud technologies and AI services to automate the journey from clinical diagnosis to financial assistance claim preparation.

The modular architecture ensures scalability, maintainability, and extensibility for future enhancements. The use of serverless technologies keeps operational overhead low while providing the flexibility to scale as needed.

For hackathon or prototype implementation, focus on core features (text upload, AI processing, eligibility matching, PDF generation) and use synthetic data. Future phases can add advanced features like multi-modal input, HMS integration, and mobile applications.

**Key Success Factors**:
1. Simple, intuitive user interface
2. Accurate AI extraction and simplification
3. Comprehensive scheme database
4. Fast processing times
5. Reliable PDF generation
6. Clear documentation and training

**Next Steps**:
1. Set up AWS environment
2. Implement core Lambda functions
3. Integrate AI services
4. Build frontend interface
5. Test with synthetic data
6. Prepare demo and documentation

---

**Document Version**: 1.0  
**Last Updated**: February 15, 2026  
**Author**: AI Workflow Copilot Team  
**Status**: Draft for Hackathon/Prototype Implementation
