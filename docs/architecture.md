# System Architecture

## Overview

The automation is divided into four n8n workflows. The first two workflows detect procurement requirements, the third handles vendor quotation generation, and the fourth processes vendor responses.

```text
                    ┌──────────────────────┐
                    │   Pending Orders     │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Inventory Analysis   │
                    └──────────┬───────────┘
                               │
                               │
                    ┌──────────▼───────────┐
                    │ Procurement Items    │
                    └──────────┬───────────┘
                               │
             ┌─────────────────┴─────────────────┐
             │                                   │
             ▼                                   ▼
   ┌─────────────────────┐             ┌─────────────────────┐
   │ Vendor Resolution   │             │ Stock Threshold     │
   └──────────┬──────────┘             │ Check               │
              │                        └──────────┬──────────┘
              └──────────────┬────────────────────┘
                             ▼
                  ┌──────────────────────┐
                  │ Vendor Quotation     │
                  │ Processing           │
                  └──────────┬───────────┘
                             │
                             ▼
                  ┌──────────────────────┐
                  │ WhatsApp Request     │
                  └──────────┬───────────┘
                             │
                             ▼
                  ┌──────────────────────┐
                  │ Vendor Response      │
                  └──────────┬───────────┘
                             │
                             ▼
                  ┌──────────────────────┐
                  │ AI Classification &  │
                  │ Extraction           │
                  └──────────┬───────────┘
                             │
                    ┌────────┴────────┐
                    ▼                 ▼
                  Text            Quotation
                    │                 │
                    ▼                 ▼
                AI Reply        Save Quotation
                                      │
                                      ▼
                               Update Status
```

## Workflow 1: Pending Orders Check

This workflow identifies procurement requirements caused by pending sales orders.

### Process

1. Receive a workflow trigger.
2. Retrieve pending orders.
3. Retrieve current inventory.
4. Retrieve items already in procurement.
5. Retrieve items for which quotations have already been sent.
6. Aggregate demand by item code.
7. Compare demand against available stock.
8. Calculate projected stock levels.
9. Identify items requiring procurement.
10. Validate recipe/procurement availability.
11. Trigger the vendor quotation workflow.

### Main Engineering Logic

The workflow prevents duplicate procurement requests by excluding items that are already being processed or have already entered the quotation stage.

---

## Workflow 2: Stock Threshold Check

This workflow detects inventory that has reached its configured minimum threshold.

### Process

1. Retrieve inventory records.
2. Retrieve items already in procurement.
3. Retrieve items already sent for quotation.
4. Compare available quantity with the minimum threshold.
5. Calculate the quantity required to restore the minimum level.
6. Exclude items already being processed.
7. Validate procurement requirements.
8. Trigger vendor quotation processing.

This workflow provides a second procurement trigger independent of pending sales orders.

---

## Workflow 3: Vendor Quotation Processing

This workflow receives procurement items from the previous workflows and manages the vendor quotation lifecycle.

### Process

```text
Input Items
     ↓
Normalize Data
     ↓
Resolve Vendors
     ↓
Group By Vendor
     ↓
Deduplicate Items
     ↓
Create Procurement Demand
     ↓
Attach Demand ID
     ↓
Batch Vendor Items
     ↓
Prepare Messages
     ↓
Create Quotation Records
     ↓
Synchronize Quotation IDs
     ↓
Update Procurement State
```

### Data Transformation

Input records are normalized into a consistent structure containing:

* Item code
* Item name
* Required quantity
* Unit of measure
* Sales order number
* Sales order ID

Vendor responses are then grouped by vendor code to avoid sending separate requests for every individual item.

### Batching

Vendor items are processed in configurable chunks. This reduces message size and allows larger vendor requests to be split into manageable batches.

### Context Restoration

After parallel API operations and merge operations, the workflow restores the original item and vendor context before continuing downstream processing.

This prevents data loss when multiple branches operate on the same records.

---

## Workflow 4: Vendor WhatsApp Processing

This workflow handles incoming vendor messages through a webhook.

### Process

```text
WhatsApp Webhook
       ↓
Validate Message
       ↓
Store Incoming Message
       ↓
AI Classification
       ↓
Structured Output
       ↓
Retrieve Conversation Context
       ↓
AI Response / Quotation Extraction
       ↓
       ┌───────────────┐
       │               │
       ▼               ▼
     Text          Quotation
       │               │
       ▼               ▼
  Send Reply      Resolve Vendor
                       ↓
                 Extract Prices
                       ↓
                 Save Quotation
                       ↓
                 Update Status
```

## AI Layer

The AI layer performs two separate responsibilities.

### Message Classification

The first AI agent determines:

* Message type
* Language
* Mentioned item names
* Whether quotation information is present

Structured output is used so downstream nodes can reliably process the result.

### Vendor Response Processing

The second AI agent uses the classification and recent conversation context to:

* Generate short conversational replies
* Extract item code and price pairs
* Preserve the detected language
* Avoid inventing quotation information

Supported language categories include:

* English
* Urdu
* Roman Urdu

---

## Integration Architecture

The workflows communicate with external services through REST APIs and webhooks.

```text
n8n
 │
 ├── REST APIs
 │      ├── Inventory
 │      ├── Procurement
 │      ├── Vendor Data
 │      └── Quotation Records
 │
 ├── Webhooks
 │      └── WhatsApp Events
 │
 ├── AI APIs
 │      └── Message Classification / Extraction
 │
 └── Workflow Execution
        ├── Procurement Detection
        └── Vendor Quotation Processing
```

## Reliability Considerations

The workflows include validation and state checks to reduce duplicate processing.

Examples include:

* Checking whether procurement data exists
* Excluding already processed items
* Validating required identifiers
* Deduplicating vendor-item combinations
* Validating AI output structure
* Restoring context after merge operations
* Separating text responses from quotation responses

## Portfolio Version

The workflows in this repository have been sanitized for portfolio use.

Production endpoints, credentials, identifiers, vendor information, and private business data have been replaced with placeholders or synthetic examples.

The purpose of this repository is to demonstrate the **workflow architecture, data processing, API integration, AI processing, and automation engineering patterns** used by the system.
