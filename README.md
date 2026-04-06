# Binox FDE Assessment: Autonomous Sales AI Pipeline

## Overview
This repository contains a multi-agent, self-improving AI sales orchestration pipeline built in n8n. The system autonomously conducts sales simulations, grades its own performance, and iteratively rewrites its core system prompt to handle new objections dynamically.

Unlike standard static chatbots, this architecture is **stateful**. It remembers past failures and patches its own logic without human intervention, proving out a foundational "AI Flywheel" concept for enterprise sales automation.

### Tech Stack
* **Orchestration:** n8n
* **LLMs:** Google Gemini 1.5 Flash (Agent, Analyst, Director nodes)
* **Voice / TTS:** Deepgram Aura
* **Database:** Supabase (PostgreSQL)

---

## Architecture & Key Engineering Wins

### 1. Resilient API Failover (Graceful Degradation)
During the initial build phase, the primary TTS provider (ElevenLabs) triggered a 401 IP/Rate Limit block. Instead of allowing the pipeline to crash, the system was dynamically re-routed to use **Deepgram Aura**. This pivot ensured sub-250ms audio latency and uninterrupted execution, demonstrating a core Forward Deployed Engineering principle: *never let external dependencies break the prototype.*

### 2. Dynamic State Management
The AI Agent does not rely on a hardcoded, static prompt. A pre-execution Postgres node utilizes the following query:
`SELECT * FROM Scripts ORDER BY version DESC LIMIT 1;`
This guarantees the system dynamically fetches the most evolved script state before every single execution. 

### 3. The Autonomous Self-Improvement Loop
The workflow utilizes a 3-tier multi-agent architecture:
* **The Caller:** Executes the script and generates the conversational transcript.
* **The Analyst:** Evaluates the transcript, extracts a boolean `success_status`, and isolates the specific prospect `objection` into structured JSON format.
* **The Director:** Triggered conditionally via an n8n If-Node on failed calls. It parses the previous version string, mathematically increments it (e.g., v1.1 -> v1.2), dynamically patches the master prompt with a targeted rebuttal, and deploys the new code back to the production database.

---

## Future Scalability (Production Considerations)

For the scope of this prototype, the pipeline executes sequentially. However, to scale this to a production environment and handle multi-turn conversations without overloading the database, I would implement the following architectural updates:

1. **Decouple the Loops:** Separate the real-time conversational loop (Frontend) from the asynchronous analytical loop (Backend).
2. **Websockets vs. Webhooks:** Connect the AI Agent to a websocket (e.g., Twilio/Retell) for real-time, multi-turn voice interaction. Trigger the Analyst/Director pipeline via a webhook *only* after the `call_status` hits 'completed' to prevent the AI from mutating the system prompt mid-conversation.
3. **RAG Integration:** Instead of endlessly appending rebuttals to the main prompt (which would eventually hit token limits), the Analyst would save successful rebuttals to a vector database. The Agent would then use Retrieval-Augmented Generation to pull only the relevant objection-handling logic in real-time.

---

## Demo & Proof of Execution

* **Video Walkthrough:**
https://www.loom.com/share/2ec3a817ecdb4fc19053db43ab490625 
* **Workflow Canvas:**
<img width="1086" height="322" alt="n8n architecture" src="https://github.com/user-attachments/assets/e7e55e35-132c-4d41-8f8b-fabba32ba58f" />

* **Database State (Proof of Loop):**
<img width="1440" height="772" alt="Screenshot 2026-04-06 at 10 47 18 AM" src="https://github.com/user-attachments/assets/98e2e1a8-e709-4a8a-92aa-1a1cd51f8901" />


---
*Built for the Binox Forward Deployed Engineer Assessment.*
