# Binox FDE Assessment: Autonomous Sales AI Pipeline

## Overview

This repository contains a multi-agent, stateful AI sales orchestration pipeline built in n8n. The system autonomously conducts sales simulations, grades its own performance, and iteratively rewrites its core system prompt to handle new objections dynamically.

Unlike static chatbots, this architecture implements an **"AI Flywheel"**—a self-improving loop that minimizes human intervention and maximizes sales conversion over time.

### Tech Stack

  * **Orchestration:** n8n
  * **LLMs:** Google Gemini 1.5 Flash (Agent, Analyst, Director)
  * **Voice / TTS:** Deepgram Aura
  * **Database:** Supabase (PostgreSQL)

-----

##  Architecture

The workflow follows a closed-loop architectural pattern:

1.  **Fetch State:** Query the latest script version from Postgres.
2.  **Execute:** Generate AI response and convert to speech via Deepgram.
3.  **Analyze:** Extract objections into structured JSON.
4.  **Optimize:** If the call failed, the "Director" agent patches the script and saves it as a new version.

-----

##  Setup & Reproducibility

To reproduce this workflow, follow these steps:

### 1\. Database Setup (Supabase/Postgres)

Create two tables in your public schema:

  * **`scripts`**: Columns: `id` (uuid), `version` (numeric), `system_prompt` (text).
      * *Initial Data:* Insert one row with version `1.0` and your base sales pitch.
  * **`outcomes`**: Columns: `id` (uuid), `script_version` (numeric), `transcript` (text), `success_status` (text), `improvement_suggestion` (text).

### 2\. n8n Configuration

1.  Import the `Binox_Sales_Pipeline.json` file into your n8n instance.
2.  Configure the following **Credentials**:
      * **Postgres:** Connect to your Supabase instance.
      * **Google Gemini:** Provide your Google AI Studio API key.
3.  In the **HTTP Request (Deepgram)** node, replace the header value with your Deepgram API Key: `Token YOUR_KEY_HERE`.

-----

##  Criteria-Based Self-Assessment

### 1\. Technical Execution 

  * **State Management:** Utilized `ORDER BY version DESC LIMIT 1` to ensure the system is truly dynamic and not relying on hardcoded prompts.
  * **Error Handling:** Implemented `onError: continueErrorOutput` on critical API nodes to prevent workflow crashes and ensure outcomes are logged even if voice generation fails.
  * **Clean Logic:** Used structured JSON output from LLM chains to drive conditional logic (If-Nodes), ensuring high reliability in the decision-making loop.

### 2\. Creativity & Constraint Handling 

  * **The Pivot:** Encountered a 401 rate-limit block with ElevenLabs during development.
  * **Trade-off Documentation:** To maintain the delivery timeline and ensure a working prototype, I pivoted to **Deepgram Aura**. This was a strategic choice; Deepgram offers significantly lower latency (\<250ms), which is superior for real-time sales applications.

### 3\. Business Impact Reasoning 

  * **ROI focused:** This solution directly addresses the high cost of manual sales training. By allowing an AI to autonomously learn how to handle objections like "Too busy" or "Too expensive," we reduce the cost-per-lead and increase the "speed to lead" for clients.
  * **Scalability:** The architecture proves that a single engineer can manage a fleet of "evolving" agents that get smarter with every failed call.

-----

## Demo & Proof of Execution

  * **Video Walkthrough:** [Loom Demo](https://www.loom.com/share/2ec3a817ecdb4fc19053db43ab490625)
  * **Workflow Canvas:**<img width="1086" height="322" alt="n8n architecture" src="https://github.com/user-attachments/assets/5ac5da20-ddb7-4c55-8d00-456c3b0b2a17" />

   
  * **Database Proof (Iteration Cycle):**
   
<img width="1440" height="743" alt="Screenshot 2026-04-06 at 2 12 24 PM" src="https://github.com/user-attachments/assets/ea931230-c3ee-405d-a95f-fe3aa7eecf5b" />

-----

*Built for the Binox Forward Deployed Engineer Assessment.*
