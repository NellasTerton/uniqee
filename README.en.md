# Uniqee: AI-Driven Narrative Engine & LLM Arbiter

[🇷🇺 Русский](README.md) · **🇬🇧 English**

**Status:** Released in production
📱 [App Store](https://apps.apple.com/us/app/uniqee-meet-by-actions/id6744572287?l=ru) | 🤖 [Google Play](https://play.google.com/store/apps/details?id=app.aibrary.android.uniqee)

## Overview

This repository holds the AI architecture, system prompts and data structures behind **Uniqee** — a gamified dating app where an LLM acts as an autonomous Game Master (the "Arbiter").

Two players read the same scene at the same time and answer blindly, without seeing each other's response. The Arbiter merges both answers, describes the consequence and continues the story — while covertly testing personal compatibility criteria the players never see.

My role as **AI Systems Designer**: architect a predictable, hallucination-free pipeline that processes two simultaneous blind inputs, holds narrative state, and generates scenes in real time.

**Scope of ownership:** the prompt and rules layer — how AI personas behave, what constrains them, the output format they must obey, the self-checks the model runs before answering, plus the scenarios and their variable banks. Backend infrastructure (hosting, database) belongs to the engineering team.

## 📂 Repository structure

*(Full source code is under NDA. This repo contains abstracted architecture logic and prompt frameworks.)*

* 📄 [`/prompts/01_The_Arbiter_System.md`](prompts/01_The_Arbiter_System.md) — core Game Master logic: routing, state, scene rules.
* 📄 [`/prompts/02_Persona_Constraints.md`](prompts/02_Persona_Constraints.md) — AI character psychotypes and anti-bot typing dynamics.
* 📄 [`/data_schemas/persona_schema.json`](data_schemas/persona_schema.json) — how LLM output is structured for the backend.
* 📄 [`/data_schemas/scenario_schema.json`](data_schemas/scenario_schema.json) — scenario payload and plot drivers.

## ⚙️ Key engineering solutions

### 1. The Arbiter: state management

Instead of plain chat completion, the model is locked inside a strict XML `<disclosure_layer>` and `<protocol>`. It receives named blocks (`expected_criteria`, `current_state`, `history`, `current_turn`) and must read them in order.

* **One criterion per scene.** The model tests strictly one hidden criterion per scene and never reveals it to the players.
* **Ping-pong.** Criteria alternate between player A and player B every scene.
* **Phase 2 (recheck).** Once both criteria lists are exhausted the story does not stop: the model consults `criteria_history` and re-tests the same criterion from a **new angle** that has not been used. For "attitude towards children": own children → other people's children in public → friends' children and responsibility → children as an inconvenience in a restaurant.

### 2. Scene type rotation

Every scene is one of five types: `FRICTION`, `SPONTANEOUS_FORK`, `PLEASANT_SURPRISE`, `INTERNAL_BEAT`, `AESTHETIC_MOMENT`. The model reads `recent_scene_types` and must rotate:

* across any 5 consecutive scenes — at least one FORK and one SURPRISE;
* never more than two FRICTION scenes in a row;
* target distribution 25/25/25/15/10;
* the first scene of a story can never be FRICTION.

This solves the core failure mode of such systems: without quotas an LLM slides into an endless chain of problems and punishment.

### 3. A toolbox of 10 scene-opening mechanics

Repetitive openings are the second reason AI stories read like a template. Every scene opens with one of 10 mechanics (`NPC_NONVERBAL`, `INFO_INJECT`, `ENVIRONMENT_SHIFT`, `SOCIAL_DYNAMICS`, `PHYSICAL_ACCIDENT`, `RESOURCE_STATUS`, `TIME_ANCHOR`, `SPATIAL_SHIFT`, `DETAIL_ZOOM`, `DIRECT_INTRUSION`). The mechanic must differ from the previous scene and is matched to the chosen scene type.

### 4. Hallucination control: hard negative constraints

* **Costly Kindness Rule.** Whenever a FRICTION scene involves helping a third party, the cost to the couple must be **shared, concrete and measurable**: money, food already ordered, a booking with a hard deadline, a physical spot they currently occupy, time measured against a concrete event. Vague costs ("some time", "social awkwardness") and reversible costs are banned. Cost types rotate too — "money" may not be the cost in more than 2 of any 5 scenes.
* **Dilemma Strength Check.** Before output the model must answer itself: "what does this scene tell me about the player if they answer A versus B?" If both answers carry the same information, the dilemma is too weak — rewrite.
* **Action Self-Check.** The model validates its own output against passive commentary and philosophising before returning the string.
* **Shared Scene Symmetry.** Every event must affect both players equally. Individual triggers ("your phone", "your boss") are banned — they turn the other player into a spectator.
* **Anti-Romance.** The model may never attribute actions, emotions or words to players that weren't in their answers. No forced intimacy, no "spark between you".
* **No Cascading Punishment.** After a friction scene the next environmental beat must be neutral or positive — unless the players' own actions caused the new problem. The world is not vindictive.
* **Split Prevention.** If players choose mutually exclusive actions, the plot freezes and demands negotiation instead of branching.

### 5. Human typing dynamics (anti-bot)

To kill the "plastic AI" voice, persona prompts constrain formatting hard: emoji ceilings, enforced lowercase for specific psychotypes, a ban on theatrical D&D-style stage directions (`*sighs and looks away*`), a ban on drawn-out letters ("pleassse", "nooo"), and a strict length limit.

**40 psychotypes** (20 female, 20 male), each with its own problem-solving logic, not just a tone of voice. The psychotype dictates **how** the character solves the problem: an alpha takes charge or pays, an anxious type retreats to safety, a pragmatist finds the cheapest exit, a troll escalates the awkwardness with a joke.

## 🎬 Scenarios and the variable engine

A scenario is not prose — it is a `<plot_prompt>` carrying context, atmosphere, plot drivers (`friction_sources`, `pleasant_surprise`, sources for `INTERNAL_BEAT` and `AESTHETIC`) and a blacklist of words and clichés.

Second-generation scenarios ship with **variable banks**: 5–7 variables per scenario, 10–20 concrete values each. Variables are injected into the plot text and are interdependent — not independent random rolls, but a nested geometry of the scene.

Example (GLAMPING): `glamping_aesthetics` (13 lodging variants) × `lost_luxury` (11 things the outage killed) × `weather_condition` (15) × `neighbor_attitude` (17) × `pleasant_discovery` (15) × `trip_reason` (16). The scenario rules explicitly require friction to be derived from `{weather_condition}` colliding with the dark `{glamping_aesthetics}`, and pleasant beats to come only from `{pleasant_discovery}`.

**Business value:** the exact same psychological criteria get tested across vastly different yet logically coherent settings. Repetitive plot patterns and hallucinations disappear, and replayability jumps — the same scenario played with a different persona produces a different story.

## 🖼 Imagery

Scenario covers and previews are generated through an image pipeline (Gemini image models + custom Comfy workflows). Originally an image was generated **for every scene** inside a session; production moved to covers only — per-scene generation broke reading pace and cost disproportionately. The `Story scene images` toggle remains in the admin panel.

## 🛠 Tech stack (production, 2026)

* **Text generation:** 2025–2026 frontier models via API. Primary is Claude (direct Anthropic API), alongside Gemini and GPT.
* **Image generation:** Gemini image models + custom Comfy workflows.
* **Auxiliary tasks:** cheap models via OpenRouter for binary judgements (e.g. "is this feed reply self-sufficient?").
* **Prompt architecture:** strict XML tagging, multi-agent logic, model self-checks before output.
* **Data handling:** structured JSON mapping, dynamic variable injection.

## 📊 Scope of work

| | |
|---|---|
| Scenarios written | 30+ |
| Of those, with the variable engine | 16 (94 variables, 1160+ values) |
| Character psychotypes | 40 |
| Persona cards in the database | 53 |
| Scene types | 5 with rotation quotas |
| Scene-opening mechanics | 10 with no-repeat rule |
