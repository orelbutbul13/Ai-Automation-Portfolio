# Benny Bear Studio — AI Children's Content Production Pipeline

End-to-end AI content automation system that writes, illustrates, and narrates episodes of an original children's educational series using multi-model AI orchestration and workflow automation.

Personal project — active proof of concept.

---

## Project Documentation

- [View Project Case Study (PDF)](https://drive.google.com/drive/folders/1OVcr8CYaEQtnaDh7GASdgBHPXytTzf9K?usp=sharing)
- [Workflow Export (n8n JSON)](./Benny_Bear_Studio_workflow.json)

---

## Project Overview

This automation replaces a fully manual children's episode production process by:

- Generating an episode script and social-emotional lesson using GPT-5
- Automatically scoring the script for story quality, educational value, and age-appropriateness
- Generating one consistent illustration per scene using a fixed character reference
- Converting every line of dialogue into character-specific narration audio
- Organizing every output into a structured, trackable production system

The system demonstrates multi-model AI orchestration (text, image, and voice) combined with automated quality control in a single production pipeline.

---

## Business Problem

Producing a children's episode requires:

- Writing an age-appropriate story with a clear lesson
- Reviewing the story for quality and safety before production
- Illustrating every scene while keeping the main character visually consistent
- Recording narration in a distinct voice per character
- Tracking and organizing every asset produced

Manual execution is slow, expensive, and difficult to repeat on a daily basis.

---

## Solution Summary

The automation system consists of:

1. n8n — orchestrates the full workflow end-to-end
2. GPT-5 — generates the episode script and performs automated quality review
3. OpenAI Image API — generates consistent scene illustrations
4. OpenAI Text-to-Speech — generates character-specific narration audio
5. Google Sheets — episode tracking and quality scoring log
6. Google Drive — automated asset storage and organization

---

## Tech Stack

- n8n — workflow orchestration
- GPT-5 — story generation and automated quality scoring
- OpenAI Image / TTS APIs — illustration and voice generation
- Google Sheets — structured tracking layer
- Google Drive — automated asset storage

---

## Workflow Architecture

Trigger → Story Generation → Automated Quality Review → Scene & Voice Scripting → Illustration → Narration → Asset Storage

1. Manual or scheduled trigger executed
2. GPT-5 writes the episode script and lesson
3. A second AI pass scores the script; low scores stop the pipeline before production cost is spent
4. Episode is broken into scenes and a narration script
5. One consistent illustration generated per scene
6. Each line converted to character-specific voice audio
7. All assets organized into Drive and logged in Sheets

---

## Key Outcomes

- Fully automated script-to-asset production pipeline
- Automated quality gate prevents low-quality output from reaching production
- Consistent character appearance across every scene and episode
- Built-in retry handling for reliability against transient API failures

---

## Skills Demonstrated

- Multi-model AI orchestration (text, image, and voice in one pipeline)
- Automation-first system design, including quality control and error handling
- Workflow architecture design beyond single prompts
- Structured asset and data pipeline development
