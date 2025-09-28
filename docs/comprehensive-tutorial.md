# Skyvern Comprehensive Tutorial & Architecture Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Architecture Overview](#architecture-overview)
3. [Core Components Deep Dive](#core-components-deep-dive)
4. [Installation & Setup](#installation--setup)
5. [Basic Usage Tutorial](#basic-usage-tutorial)
6. [Advanced Features](#advanced-features)
7. [Workflow Development](#workflow-development)
8. [Integration Patterns](#integration-patterns)
9. [Performance & Optimization](#performance--optimization)
10. [Troubleshooting Guide](#troubleshooting-guide)

## Introduction

Skyvern is a revolutionary web automation platform that leverages Large Language Models (LLMs) and computer vision to automate browser-based workflows. Unlike traditional automation tools that rely on brittle XPath selectors and DOM parsing, Skyvern uses AI to understand and interact with websites like a human would.

### Key Differentiators
- **AI-Powered Navigation**: Uses vision LLMs to understand web pages
- **Layout-Resistant**: Adapts to website changes without code modifications
- **Universal Compatibility**: Works on websites never seen before
- **Complex Reasoning**: Handles nuanced interactions and form filling

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        Skyvern Platform                         │
├─────────────────────────────────────────────────────────────────┤
│  Frontend (React)          │  Core Engine (Python)             │
│  ┌─────────────────────┐   │  ┌─────────────────────────────┐   │
│  │ Workflow Builder    │   │  │ Task Orchestrator           │   │
│  │ Task Manager        │   │  │ Agent Swarm                 │   │
│  │ Live Streaming      │   │  │ LLM Integration             │   │
│  └─────────────────────┘   │  └─────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────┤
│  Web Eye (Browser Automation)                                   │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │ Playwright Integration │ DOM Analysis │ Computer Vision     │ │
│  │ Element Detection      │ Action Exec  │ Screenshot Analysis │ │
│  └─────────────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────────┤
│  Storage & Services                                             │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │ PostgreSQL │ Redis Cache │ File Storage │ Credentials Mgmt │ │
│  └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Core Components Deep Dive

### 1. Web Eye Engine (`skyvern/webeye/`)

The Web Eye is Skyvern's browser automation engine built on Playwright.

**Key Files:**
- `domUtils.js`: DOM manipulation and element detection
- `scraper/`: Web page analysis and data extraction
- `actions/`: Browser action implementations
- `utils/dom.py`: Python-JavaScript bridge for DOM operations

**Capabilities:**
- **QuadTree Optimization**: Efficient spatial indexing for element detection
- **Incremental DOM Updates**: Real-time page change detection
- **Visibility Detection**: Smart element visibility calculations
- **Action Caching**: Performance optimization for repeated actions

```python
# Example DOM interaction
from skyvern.webeye.utils.dom import SkyvernElement, DomUtil

# Initialize DOM utility
dom_util = DomUtil(scraped_page, page)

# Find and interact with elements
element = await dom_util.get_skyvern_element_by_id("element_id")
await element.click_in_javascript()
```

### 2. Forge Engine (`skyvern/forge/`)

The Forge contains Skyvern's core automation logic and AI orchestration.

**Key Components:**
- **Agent System**: Multi-agent architecture for task execution
- **Prompt Engineering**: LLM prompt templates and optimization
- **Task Management**: Workflow orchestration and state management
- **Action Planning**: AI-driven action sequence generation

**Agent Types:**
1. **Navigation Agent**: Handles web page navigation
2. **Extraction Agent**: Performs data extraction
3. **Action Agent**: Executes form filling and clicks
4. **Validation Agent**: Verifies task completion

### 3. Workflow System (`fern/workflows/`)

Skyvern's workflow system enables chaining multiple tasks into complex automation sequences.

**Workflow Blocks:**
- `TaskBlock`: Core web automation
- `ForLoopBlock`: Iteration over data sets
- `CodeBlock`: Custom Python execution
- `TextPromptBlock`: LLM text generation
- `FileParserBlock`: Document processing
- `EmailBlock`: Email automation
- `UploadToS3Block`: File storage

**Example Workflow:**
```yaml
title: E-commerce Product Research
description: Extract product information from multiple sites
workflow_definition:
  parameters:
    - key: search_terms
      parameter_type: workflow
      workflow_parameter_type: array
  blocks:
    - block_type: for_loop
      label: product_search_loop
      loop_over: search_terms
      blocks:
        - block_type: task
          label: search_products
          url: "https://example-store.com/search"
          navigation_goal: "Search for {loop_item} and extract top 10 results"
          data_extraction_goal: "Extract product name, price, rating, and URL"
```

### 4. Frontend Application (`skyvern-frontend/`)

React-based UI for workflow management and task monitoring.

**Key Features:**
- **Visual Workflow Builder**: Drag-and-drop workflow creation
- **Live Browser Streaming**: Real-time task execution viewing
- **Task History**: Comprehensive execution logs and analytics
- **Credential Management**: Secure storage of authentication data

## Installation & Setup

### Prerequisites
- Python 3.11+
- Node.js 18+
- PostgreSQL 12+
- Redis 6+

### Quick Start

```bash
# Install Skyvern
pip install skyvern

# Initialize configuration
skyvern quickstart

# Start all services
skyvern run all
```

### Advanced Setup

```bash
# Clone repository for development
git clone https://github.com/skyvern-ai/skyvern.git
cd skyvern

# Install dependencies
poetry install

# Setup environment
cp .env.example .env
# Edit .env with your configuration

# Initialize database
alembic upgrade head

# Start services
skyvern run server  # Backend API
skyvern run ui      # Frontend interface
```

### Configuration

Essential environment variables:

```bash
# LLM Configuration
LLM_KEY=OPENAI_GPT4O
OPENAI_API_KEY=your_openai_key

# Database
DATABASE_URL=postgresql://user:pass@localhost:5432/skyvern

# Redis
REDIS_URL=redis://localhost:6379

# Browser Configuration
BROWSER_TYPE=chromium
ENABLE_BROWSER_RECORDING=true

# Security
SECRET_KEY=your_secret_key
ENABLE_AUTHENTICATION=true
```

## Basic Usage Tutorial

### 1. Your First Task

```python
from skyvern import Skyvern

# Initialize Skyvern client
skyvern = Skyvern()

# Create a simple task
task = await skyvern.run_task(
    prompt="Navigate to news.ycombinator.com and find the top story title",
    url="https://news.ycombinator.com"
)

print(f"Task completed: {task.status}")
print(f"Extracted data: {task.extracted_data}")
```

### 2. Form Filling Example

```python
# Fill out a contact form
task = await skyvern.run_task(
    prompt="""
    Fill out the contact form with the following information:
    - Name: John Doe
    - Email: john.doe@example.com
    - Subject: Product Inquiry
    - Message: I'm interested in learning more about your services.
    Then submit the form.
    """,
    url="https://example.com/contact"
)
```

### 3. Data Extraction with Schema

```python
# Extract structured data
task = await skyvern.run_task(
    prompt="Extract product information from this page",
    url="https://example-store.com/product/123",
    data_extraction_schema={
        "type": "object",
        "properties": {
            "product_name": {"type": "string"},
            "price": {"type": "number"},
            "availability": {"type": "string"},
            "description": {"type": "string"},
            "images": {
                "type": "array",
                "items": {"type": "string"}
            }
        }
    }
)
```

## Advanced Features

### 1. Authentication & Credentials

Skyvern supports multiple authentication methods:

```python
# Using stored credentials
task = await skyvern.run_task(
    prompt="Login and download recent invoices",
    url="https://portal.example.com",
    credentials={
        "type": "bitwarden",
        "identifier": "portal.example.com"
    }
)

# 2FA Support
task = await skyvern.run_task(
    prompt="Login with 2FA",
    url="https://secure.example.com",
    totp_identifier="example_totp",
    totp_verification_url="https://secure.example.com/verify"
)
```

### 2. File Handling

```python
# File download
task = await skyvern.run_task(
    prompt="Download the latest report PDF",
    url="https://reports.example.com",
    complete_on_download=True,
    download_suffix=".pdf"
)

# File upload
task = await skyvern.run_task(
    prompt="Upload the document to the form",
    url="https://upload.example.com",
    file_uploads=[{
        "path": "/path/to/document.pdf",
        "field_name": "document"
    }]
)
```

### 3. Error Handling & Retries

```python
task = await skyvern.run_task(
    prompt="Complete the complex form",
    url="https://complex-form.example.com",
    max_retries=3,
    error_code_mapping={
        "captcha_required": "Please solve the CAPTCHA manually",
        "form_validation_error": "Check form fields for errors"
    },
    terminate_criterion="If you see 'Access Denied', stop immediately"
)
```

## Workflow Development

### 1. Basic Workflow Structure

```yaml
title: Invoice Processing Workflow
description: Download and process invoices from multiple vendors
workflow_definition:
  parameters:
    - key: vendor_urls
      parameter_type: workflow
      workflow_parameter_type: array
    - key: date_range
      parameter_type: workflow
      workflow_parameter_type: object
      
  blocks:
    - block_type: for_loop
      label: vendor_loop
      loop_over: vendor_urls
      blocks:
        - block_type: task
          label: vendor_login
          url: "{loop_item.url}"
          navigation_goal: "Login using stored credentials"
          
        - block_type: task
          label: invoice_download
          navigation_goal: "Navigate to invoices and download all invoices from {date_range.start} to {date_range.end}"
          complete_on_download: true
          
    - block_type: upload_to_s3
      label: store_invoices
      path: "SKYVERN_DOWNLOAD_DIRECTORY"
      
    - block_type: send_email
      label: notification
      to: ["admin@company.com"]
      subject: "Invoice Download Complete"
      body: "All vendor invoices have been downloaded and stored."
```

### 2. Conditional Logic

```yaml
- block_type: task
  label: check_inventory
  data_extraction_goal: "Check if product is in stock"
  data_schema:
    type: object
    properties:
      in_stock: {type: boolean}
      
- block_type: conditional  # Coming soon
  condition: "previous_block.in_stock == true"
  if_true:
    - block_type: task
      label: place_order
      navigation_goal: "Add to cart and proceed to checkout"
  if_false:
    - block_type: task
      label: setup_notification
      navigation_goal: "Setup stock notification"
```

### 3. Custom Code Integration

```yaml
- block_type: code
  label: process_data
  code: |
    import pandas as pd
    
    # Process extracted data
    df = pd.DataFrame(context.previous_blocks['extract_data'].output)
    df['processed_at'] = pd.Timestamp.now()
    
    # Calculate metrics
    metrics = {
        'total_records': len(df),
        'avg_price': df['price'].mean(),
        'categories': df['category'].unique().tolist()
    }
    
    return metrics
```

## Integration Patterns

### 1. API Integration

```python
from skyvern import Skyvern
import requests

class SkyvernAPIIntegration:
    def __init__(self, api_key):
        self.skyvern = Skyvern(api_key=api_key)
        
    async def scrape_and_enrich(self, url, enrichment_api):
        # Scrape data
        task = await self.skyvern.run_task(
            prompt="Extract all product information",
            url=url,
            data_extraction_schema={
                "type": "object",
                "properties": {
                    "products": {
                        "type": "array",
                        "items": {
                            "type": "object",
                            "properties": {
                                "name": {"type": "string"},
                                "sku": {"type": "string"},
                                "price": {"type": "number"}
                            }
                        }
                    }
                }
            }
        )
        
        # Enrich with external API
        enriched_data = []
        for product in task.extracted_data['products']:
            enrichment = requests.get(
                f"{enrichment_api}/enrich",
                params={"sku": product['sku']}
            ).json()
            
            enriched_data.append({
                **product,
                "category": enrichment.get('category'),
                "brand": enrichment.get('brand')
            })
            
        return enriched_data
```

### 2. Database Integration

```python
import asyncpg
from skyvern import Skyvern

async def automated_data_pipeline():
    # Connect to database
    conn = await asyncpg.connect("postgresql://user:pass@localhost/db")
    
    # Get URLs to process
    urls = await conn.fetch("SELECT url FROM pending_scrapes WHERE status = 'pending'")
    
    skyvern = Skyvern()
    
    for url_record in urls:
        try:
            # Process with Skyvern
            task = await skyvern.run_task(
                prompt="Extract all data from this page",
                url=url_record['url']
            )
            
            # Store results
            await conn.execute("""
                INSERT INTO scraped_data (url, data, scraped_at)
                VALUES ($1, $2, NOW())
            """, url_record['url'], task.extracted_data)
            
            # Update status
            await conn.execute("""
                UPDATE pending_scrapes 
                SET status = 'completed' 
                WHERE url = $1
            """, url_record['url'])
            
        except Exception as e:
            await conn.execute("""
                UPDATE pending_scrapes 
                SET status = 'failed', error = $2
                WHERE url = $1
            """, url_record['url'], str(e))
```

---

This comprehensive guide provides the foundation for understanding and using Skyvern effectively. The next sections will cover the specialized automation forks and advanced implementation patterns.