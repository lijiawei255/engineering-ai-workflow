# Engineering AI Workflow: A 4-Step Hub-and-Spoke Method

> **[English](./README.md)** | **[简体中文](./README_ZH.md)** | **[Français](./README_FR.md)** | **[Русский](./README_RU.md)**
>
> **Author: Jiawei Li** | **License: MIT**

## Background

When applying AI to practical engineering—mechatronics, automation, instrumentation, embedded systems, or core architecture design—engineers usually run into two common bottlenecks:

1. **The Single-Window Grind**: Pasting code and specs back and forth in a single chat window. After five or six turns, context pollution sets in: fixing one bug breaks two existing features. Worse, asking a high-end reasoning model to write code directly often triggers compulsive over-engineering—stuffing a clean, minimal module full of heavy CI scripts, defensive boilerplate, and sprawling edge-case tests.
2. **Unchecked Multi-Agent Loops**: Running automated multi-agent frameworks where a "Coder AI" and a "Reviewer AI" converse in a closed loop. Without a human circuit breaker in the middle, the Coder blindly accepts every pedantic nitpick from the Reviewer. Within a few rounds, a lean design mutates into bloated enterprise spaghetti.

This repository documents a model-agnostic, **4-step Hub-and-Spoke workflow** built around three distinct behavioral roles (**1 Persistent Cloud Hub + 2 Stateless Local Agents**).

The goal is simple: **let engineers with domain and physical intuition turn a rough concept into a clean, integrated, production-ready system—without hand-writing long prompts and without reading pages of pedantic review jargon.**

---

## 1. Role Topology: 1 Cloud Hub + 2 Local Agents

In this setup, **you (the human) only ever interact with AI #1**. AI #2 and AI #3 act as isolated, stateless subroutines dispatched via AI #1. They never talk directly to each other, and roles are strictly separated by both **behavioral traits** and **deployment environments**:

| Role      | Behavioral Persona                                           | Deployment Environment      | Context Window Rule                                          | Core Responsibility                                          |
| :-------- | :----------------------------------------------------------- | :-------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **Human** | **Lead Engineer / Steering**                                 | Host Workstation            | Low-frequency, high-level                                    | Provides initial domain intuition; keeps AI #1 from drifting; makes the final call on what stays and what gets cut. |
| **AI #1** | **The Conciliatory AI**<br>(Master Hub / Prompt Compiler / Filter) | **Cloud Conversational AI** | **Single Persistent Window**<br>(Receives only distilled summaries, keeping token usage low and design intent continuous) | Expands your initial intuition into concrete plans; writes all prompts for AI #2 and AI #3; translates and filters AI #3's critiques into plain language. |
| **AI #2** | **The Pure Worker AI**<br>(Silent Executor)                  | **Local Agent**             | **Fresh Session Every Run**<br>(Stateless; close immediately after use) | Keeps quiet and builds. Modifies local files, runs verification, and invokes host tools as specified—outputs a brief execution & anomaly report. |
| **AI #3** | **The Stringent AI**<br>(Ruthless Auditor / Nitpicker)       | **Local Agent**             | **Fresh Session Every Run**<br>(Prevents historical bias and local optima) | Hunts for flaws across top-level architecture, low-level implementation, and boundary conditions. **Strictly forbidden from modifying files—outputs an audit report only.** |

> **What is the difference between a "Cloud Conversational AI" and a "Local Agent"?**
> * **Cloud Conversational AI (AI #1)**: A standard chat interface (such as a web browser chat or a pure Q&A window) that has **no direct access to your local hard drive**. It cannot scan your project folders or run commands on your computer; you only talk to it via text (and manually upload a specific key document or snippet only when strictly necessary).
> * **Local Agent (AI #2 & AI #3)**: This does **not** mean running a small open-source model on your local GPU. Instead, it refers to modern **desktop agent applications, IDE integrations, or CLI tools** installed on your computer. While still powered by top-tier models, a Local Agent is granted permission to directly read and edit your local project files, execute terminal commands, and invoke compilers or engineering software on your machine—eliminating the need to manually copy and paste code files back and forth.

---

## 2. The 4-Step Iteration Loop

### Step 1: Shape the Concept with AI #1 (Cloud Conversational Hub)
* Start with an initial engineering intuition—even if you haven't worked out every implementation detail, your domain experience tells you what approach makes sense and what won't hold up in practice.
* Pitch this raw idea to **AI #1** in the cloud chat window. Ask it to evaluate how to structure it properly and fill in the missing pieces.
* Discuss back and forth until you land on a clean preliminary design that you are happy with.

### Step 2: Compile the Build Prompt & Execute via AI #2 (Local Worker Agent)
* Once the preliminary idea is set, tell **AI #1**: *"We are assigning AI #2 as a pure execution worker. Write a prompt instructing it to build this design locally, and require it to output a report afterward detailing what it did and any strange behaviors observed during coding or runtime."*
* Copy that prompt into a **brand-new session in your Local Agent (AI #2)** and let it directly edit files, run builds, and invoke local verification tools.
* Copy AI #2's execution report back into **AI #1's persistent cloud window** so AI #1 stays synced with the actual state of the project.

### Step 3: Non-Leading Full-System Audit via AI #3 (Local Auditor Agent)
* Once AI #1 is synced, tell **AI #1**: *"Write an audit prompt for AI #3, which is an extremely harsh, nitpicking reviewer. Do NOT make the prompt leading or narrow (do not just ask it to check the latest changes, or it will get stuck in a local optimum). Instruct it to run a comprehensive audit—from top-level architectural patterns down to low-level implementation and boundary conditions—and output a full report."*
* Open a **brand-new session in your Local Agent (AI #3)**, let it inspect the local codebase and run any needed checks, and collect its unfiltered critique report.

### Step 4: High-Signal Filtering via AI #1 (Keeping Fixes, Dropping Bloat)
You **do not** need to read AI #3's exhausting wall of critique yourself. Instead, dump AI #3's raw report straight back into **AI #1's cloud window** with a clear framing instruction: *"This report comes from a purely harsh, fault-finding critic. Review it against our original design intent and tell me what to keep and what to throw away."*

AI #1 then handles the triage:
1. **Separate Signal from Noise**: Protecting your original design, AI #1 flags which points are genuine bugs or boundary failures worth fixing, and which are over-engineered bloat (like unnecessary test frameworks or redundant abstractions) that should be discarded, summarizing the verdict in plain human language.
2. **Compile the Next Worker Prompt**: Once you approve the filtered list, AI #1 writes a concise, integrated prompt for the next build pass.
3. **Repeat Until Complete**: Paste the new prompt into a **fresh local AI #2 session** $\rightarrow$ sync AI #2's report back to AI #1 $\rightarrow$ audit in a **fresh local AI #3 session** $\rightarrow$ filter via AI #1.

Step by step, the project converges toward its **most accurate, concise, and integrated form** without accumulating bloat.

---

## 3. The Human Role in the Loop

Beyond shaping the initial concept in Step 1, your involvement comes down to three light-touch actions:

1. **Course-Correct AI #1**: Because AI #1 is your main workspace and has an agreeable persona, it can easily drift into flattery, exaggeration, or local optima. Whenever it starts over-complicating things, drop a short instruction to pull it back on track.
2. **Summarize and Prune via AI #1**: Since AI #1 is good at speaking plain human language, use it to digest dry technical reports and strip out unhelpful or over-complicated suggestions before they touch your codebase.
3. **Inject New Intuition on the Fly**: Whenever a new insight or engineering constraint comes to mind mid-project, tell AI #1 in plain conversational language and let it dispatch the updated instructions to the two local agents below.

---

## 4. Why It Is Designed This Way

### 1. Why use a Cloud Chat AI for #1, and Local Agents for #2 and #3?
* **Why AI #1 stays isolated from local files**: If the master planning AI is hooked directly into your local directory, it will inevitably read through directory trees, build logs, and large source files, quickly blowing up and polluting its context window. Keeping AI #1 as a pure cloud conversational AI—and manually uploading a key document or core snippet only when strictly necessary—keeps its context clean, lightweight, and focused on high-level architecture.
* **Why AI #2 and #3 must be Local Agents**: Engineering work requires more than generating text; it requires modifying local project files, compiling code, running verification suites, and calling various desktop or engineering tools installed on your computer. Pure cloud chat interfaces—and even cloud-hosted VMs/sandboxes—do not work for this, because cloud VMs are almost always generic, stripped-down Linux containers that lack your actual host OS environment, hardware drivers, and local engineering toolchains.

### 2. Why never let the same AI (especially the Stringent AI) both build and review?
* **Self-Review Blindness**: Any model reviewing its own output naturally favors its own habits—reviewing your own code is always comfortable, which makes it easy to miss structural flaws.
* **Defensive Bloat**: Stringent models have a habit of writing heavy defensive boilerplate and sprawling, chaotic tests. If you let AI #3 write code directly, it will pack the project with unnecessary clutter. **Keep AI #2 on the build, and keep AI #3 strictly on audit duty.**

### 3. Why keep AI #1 in a single window, while opening fresh sessions for AI #2 and AI #3?
* **One window for AI #1 preserves continuity**: AI #1 only reads compact summaries and reports, so token usage stays low. Even if context compression eventually triggers on long projects, it doesn't hurt high-level architectural decisions, keeping the core train of thought continuous.
* **Fresh sessions for #2 and #3 prevent context pollution**: Reusing sessions for coding or auditing leaves failed attempts and prior compromises sitting in the model's context, dragging it into local optima. A fresh session guarantees an unpolluted builder (#2) and a sharp, unbiased auditor (#3) every time.