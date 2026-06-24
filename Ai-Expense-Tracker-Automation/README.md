# AI Expense Tracker Automation

AI-powered expense tracking system that captures Telegram messages, extracts structured financial data using large language models, and stores clean records in Google Sheets.

Developed as part of the TripleTen AI Automation Program.

---

## Project Documentation

- [View Project Documentation (PDF)](https://drive.google.com/file/d/1a7dLLtall2f6Oo45Y6I07zZdmf6KGZ44/view?usp=drive_link)

---

## Project Overview

This automation enables real-time expense logging by:

- Receiving expense entries via Telegram  
- Extracting structured financial fields using an AI model  
- Categorizing each transaction automatically  
- Writing structured records into Google Sheets  

The system converts unstructured text into analysis-ready financial data suitable for tracking and reporting.

---

## Business Problem

Manual expense tracking is inefficient and error-prone:

- Transactions are forgotten  
- Receipts are lost  
- Expenses are misclassified  
- Financial records become incomplete  

These inefficiencies reduce visibility and compromise data accuracy.

---

## Solution Summary

The automation system consists of:

1. Telegram Bot API — captures expense entries  
2. Webhook processing (Zapier) — manages incoming data  
3. AI model (LLM) — extracts amount, category, and metadata  
4. Google Sheets — stores structured financial records  

---

## Tech Stack

- Telegram Bot API — input capture  
- Zapier — workflow orchestration  
- AI by Zapier (LLM) — structured data extraction  
- CloudConvert — file handling and preprocessing  
- Google Sheets — structured data storage  

---

## Workflow Architecture

Telegram → Webhook Processing → AI Extraction → Categorization → Structured Storage

1. Telegram message received  
2. Webhook processes incoming data  
3. AI extracts financial details  
4. Categorization applied  
5. Structured record written to Google Sheets  

---

## Key Outcomes

- Automated expense logging  
- Clean, structured financial dataset  
- Real-time transaction capture  
- Reduced manual data entry  
- Scalable automation framework  

---

## Skills Demonstrated

- LLM-driven data extraction  
- Workflow automation architecture  
- Structured data pipeline design  
- Financial process automation  
- Operational efficiency optimization  
