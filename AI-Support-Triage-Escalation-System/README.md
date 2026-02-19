# AI Support Triage & Escalation Automation System

End-to-end AI-powered support triage system that classifies service requests, determines urgency, and dynamically routes actions across multiple operational channels using workflow orchestration and structured large language model outputs.

Developed as part of the TripleTen AI Automation Program.

---

## Project Documentation

- [Download Full Case Study (PDF)](https://drive.google.com/file/d/12yzJobpccB5bWdeV-cyrWbPrfAVACMZs/view?usp=drive_link)

---

## Project Overview

This automation replaces a fully manual inbound support triage process by:

- Receiving service requests via webhook  
- Classifying service type using an AI model  
- Determining urgency level through structured rules  
- Generating professional confirmation responses  
- Logging structured records into Google Sheets  
- Escalating urgent cases automatically via Slack and SMS  

The system demonstrates scalable, AI-driven operational automation.

---

## Business Problem

Handling inbound service requests manually requires:

- Reviewing unstructured customer messages  
- Interpreting service intent  
- Determining urgency  
- Logging requests into tracking systems  
- Notifying internal teams  
- Escalating emergency situations  

Manual triage leads to inconsistent classification, delayed responses, and limited scalability as request volume increases.

---

## Solution Summary

The automation system consists of:

1. Webhook trigger — captures inbound requests  
2. Make (workflow orchestration layer) — manages routing logic  
3. OpenAI (LLM) — performs structured classification and response generation  
4. Prompt engineering — enforces deterministic JSON output  
5. Google Sheets — logs structured support records  
6. Slack + Twilio — handle internal notifications and urgent escalations  

---

## Tech Stack

- Make (workflow orchestration)  
- OpenAI API (LLM classification)  
- Google Sheets API (structured logging)  
- Slack API (internal notifications)  
- Twilio API (SMS escalation)  
- Prompt engineering (structured JSON enforcement)  

---

## Workflow Architecture

Webhook Trigger → AI Classification → JSON Parsing → Data Logging → Conditional Routing → Multi-Channel Notification

1. Inbound request received via webhook  
2. AI model analyzes request and returns structured JSON  
3. Structured fields validated and parsed  
4. Record written to Google Sheets  
5. Urgency-based router executes conditional logic  
6. Slack notification sent  
7. Twilio SMS triggered for high-priority cases  

---

## Key Outcomes

- Fully automated service request classification  
- Deterministic urgency-based routing  
- Real-time escalation handling  
- Structured operational logging  
- Reduced manual triage workload  
- Scalable AI-integrated support infrastructure  

---

## Skills Demonstrated

- LLM-driven classification systems  
- Deterministic prompt engineering  
- Event-driven workflow automation  
- Conditional routing architecture  
- Multi-channel integration design  
- Structured data pipeline implementation  
