# Vendor Procurement & Quotation Automation

An event-driven procurement automation system built with **n8n, REST APIs, JavaScript, AI agents, and database integrations**.

The system identifies procurement requirements from pending orders and inventory thresholds, resolves suitable vendors, creates quotation requests, communicates with vendors through WhatsApp, and processes incoming quotation responses using AI.

## Architecture

```text
Pending Orders ─────┐
                    │
                    ▼
             Inventory Analysis
                    │
                    ▼
             Procurement Items
                    │
                    ▼
             Vendor Resolution
                    │
                    ▼
            Quotation Generation
                    │
                    ▼
             WhatsApp Request
                    │
                    ▼
             Vendor Response
                    │
                    ▼
          AI Classification Agent
                    │
              ┌─────┴─────┐
              ▼           ▼
            Text      Quotation
              │           │
        Chat Memory   Extract Items
              │           │
              ▼           ▼
          AI Reply    Save Quotation
                          │
                          ▼
                    Update Status
```

## Workflows

### 1. Pending Orders Check

Analyzes pending sales orders against current inventory.

* Retrieves pending orders
* Retrieves current stock levels
* Excludes items already in procurement or quotation processing
* Aggregates demand by item
* Calculates projected stock levels
* Identifies procurement requirements
* Checks recipe availability
* Triggers the vendor quotation workflow

### 2. Stock Threshold Check

Detects items that have reached or fallen below their minimum stock threshold.

* Retrieves inventory data
* Checks minimum stock levels
* Excludes items already being processed
* Calculates required quantities
* Validates procurement requirements
* Triggers vendor quotation processing

### 3. Vendor Quotation Processing

Handles the procurement workflow after items requiring quotation have been identified.

* Normalizes incoming item data
* Resolves vendors for each item
* Groups items by vendor
* Deduplicates vendor-item combinations
* Creates procurement demand records
* Batches vendor requests
* Prepares WhatsApp quotation messages
* Creates quotation records
* Synchronizes quotation IDs across services
* Updates procurement status

### 4. Vendor WhatsApp Processing

Processes vendor responses using AI.

* Receives WhatsApp webhook events
* Classifies incoming messages
* Detects quotation vs. normal text
* Detects English, Urdu, and Roman Urdu
* Extracts item codes and quoted prices
* Retrieves recent conversation context
* Generates contextual replies
* Resolves vendor information
* Saves quotation responses
* Updates quotation reply status

## Technical Highlights

* n8n workflow orchestration
* REST API integration
* Webhook-driven architecture
* Workflow-to-workflow execution
* JavaScript data transformation
* Data normalization
* Grouping and deduplication
* Conditional branching
* Parallel processing
* Merge and context restoration
* Batch processing
* AI structured output
* Multilingual AI processing
* Conversation memory
* WhatsApp API integration
* Database synchronization
* Asynchronous event handling
* Error and validation handling

## Example Flow

A simplified procurement cycle:

```text
Inventory / Orders
       ↓
Identify Required Items
       ↓
Check Procurement Status
       ↓
Resolve Vendors
       ↓
Create Demand
       ↓
Prepare Vendor Requests
       ↓
Send WhatsApp Request
       ↓
Receive Vendor Response
       ↓
AI Classification
       ↓
Extract Item + Price
       ↓
Save Quotation
       ↓
Update Procurement Records
```

## Repository Structure

```text
vendor-quotation-automation/
│
├── README.md
│
├── workflows/
│   ├── pending-orders-check.json
│   ├── stock-threshold-check.json
│   ├── vendor-quotation-processing.json
│   └── vendor-whatsapp-processing.json
│
├── screenshots/
│   ├── 01-pending-orders-flow.png
│   ├── 02-stock-threshold-flow.png
│   ├── 03-vendor-processing-flow.png
│   └── 04-whatsapp-ai-processing.png
│
├── sample-data/
│   ├── pending-orders.json
│   ├── stock-data.json
│   ├── vendor-message.json
│   └── quotation-response.json
│
└── docs/
    └── architecture.md
```

## Data Flow

The automation separates procurement detection from vendor communication and quotation processing.

**Procurement Detection**

Orders and inventory are analyzed to determine which items require procurement.

**Vendor Resolution**

Required items are mapped to available vendors and grouped to reduce unnecessary requests.

**Quotation Processing**

Quotation requests are prepared and tracked through the procurement workflow.

**AI Response Processing**

Incoming vendor messages are classified and structured using AI before quotation data is persisted.

**Synchronization**

Quotation identifiers and statuses are synchronized across connected backend services.

## AI Processing

The WhatsApp processing workflow uses AI for two main tasks:

### Message Classification

Incoming messages are classified by:

* Message type
* Language
* Mentioned item codes
* Quotation information

### Vendor Communication

The AI can:

* Retrieve recent conversation context
* Understand vendor messages
* Extract quotation information
* Respond in the vendor's language
* Return structured quotation data

The workflow is designed to avoid inventing item or quotation information and uses structured outputs for downstream processing.

## Sample Data

The `sample-data/` directory contains synthetic examples showing the expected structure of:

* Pending orders
* Inventory data
* Vendor messages
* Processed quotation responses

No production credentials or private business data are included.

## Security & Privacy

This repository contains a sanitized portfolio version of the automation.

Production-specific information has been replaced with placeholders or synthetic data, including:

* API endpoints
* Credentials
* Vendor information
* Employee identifiers
* Company-specific identifiers
* Production records
* Private configuration

The workflow files demonstrate the **engineering architecture and automation patterns** without exposing production data.

## Key Engineering Problems Solved

This project demonstrates practical automation engineering beyond simple API connections.

It handles:

* Multiple asynchronous workflows
* Large nested JSON structures
* Data normalization between systems
* Vendor grouping and deduplication
* State tracking across procurement stages
* Parallel database/API operations
* Context restoration after merge operations
* AI-to-API structured data conversion
* Multilingual vendor communication
* Webhook-based event processing
* Cross-workflow orchestration

## Stack

**Automation:** n8n

**Programming:** JavaScript

**AI:** OpenAI API

**Communication:** WhatsApp API

**Integration:** REST APIs, Webhooks

**Data:** JSON, relational database integrations

## Disclaimer

This repository is a sanitized portfolio representation of a production-style procurement automation system.

All examples and sample data are synthetic and are provided for demonstration purposes.
