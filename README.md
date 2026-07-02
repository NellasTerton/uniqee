# UniQee: AI-Driven Narrative Engine & LLM Arbiter

**Status:** Released in Production  
📱 [App Store](https://apps.apple.com/us/app/uniqee-meet-by-actions/id6744572287?l=ru) | 🤖 [Google Play](https://play.google.com/store/apps/details?id=app.aibrary.android.uniqee)

## Overview
This repository contains the high-level AI architecture, system prompts, and data structures for **UniQee** — a gamified application where an LLM acts as an autonomous "Game Master" (The Arbiter). 

My role as **Lead AI Systems Designer** was to architect a predictable, hallucination-free LLM pipeline that processes simultaneous blind inputs from two users, calculates tension, and generates dynamic narrative scenarios in real-time.

## 🛠 Tech Stack (Production 2026)
*   **LLM Engines:** 2025–2026 Frontier Models via API. Heavy emphasis on Claude and Gemini (via AI Studio), alongside GPT. 
*   **Media Generation:** Nano Banana Pro & GPT-driven image generation pipelines + my own Comfy Workflows.
*   **Prompt Architecture:** Strict XML-tagging, Multi-Agent workflow logic.
*   **Data Handling:** Structured JSON Mapping, Dynamic Variable Injection.

## 📂 Repository Structure
*(Note: Full source code is under NDA. This repo contains abstracted architecture logic and prompt frameworks).*

*   📄 `/prompts/01_The_Arbiter_System.md` — The core Game Master logic, routing, and state management.
*   📄 `/prompts/02_Persona_Constraints.md` — AI character psychotypes and anti-bot typing dynamics.
*   📄 `/data_schemas/persona_schema.json` — Example of how LLM output is structured for the backend.

## ⚙️ Key Engineering Solutions

### 1. The Arbiter (State & Logic Management)
Instead of relying on basic chat completion, the LLM is restricted by a strict XML-based `<disclosure_layer>` and `<protocol>`. It tracks tested variables (e.g., empathy, boundaries) and applies rotation constraints (Friction vs. Spontaneous events) to maintain narrative pacing without developer intervention.

### 2. Hallucination Control & Guardrails
To prevent the model from generating "cinematic" or illogical outputs, I implemented strict negative constraints:
*   **Costly Kindness Rule:** Forces the AI to generate measurable, shared resource costs (time, money, physical space) instead of abstract social awkwardness.
*   **Action Self-Check:** Forces the LLM to validate its own output against passive commentary before returning the string.

### 3. Human Typing Dynamics (Anti-Bot)
To solve the "plastic AI" problem, persona prompts include strict formatting rules: absolute limits on emojis, lowercase enforcement for specific psychotypes, and bans on theatrical D&D-style stage directions (e.g., `*sighs and looks away*`).

## 🚀 V2 Architecture: Interdependent Variable Engine
Currently upgrading the procedural generation pipeline to include dynamically linked contextual variables. 

Instead of static scene generation, the system now cross-references interdependent variables before prompting the Arbiter (e.g., `Location: Business District` + `Time: Morning` logically triggers `Event: Commuters in suits`, not a random occurrence).

**Business Value:** 
*   Eliminates repetitive plot patterns and AI hallucinations.
*   Ensures the exact same psychological criteria are tested across vastly different, realistic settings.
*   Drastically increases replayability and user engagement, allowing players to replay the game with different personas in completely unique, logically coherent environments.
