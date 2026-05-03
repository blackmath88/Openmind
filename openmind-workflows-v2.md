# OpenMind — Workflows v2.0

**Last updated:** 2026-05-03  
**Architecture version:** 2.0  
**Principle:** Earn each workflow through usage. Don't build what you haven't felt the need for.

---

## 1. Daily Capture (~5 min, zero context switching)

### 1a. Capture from LLM Chat (Primary)

**When:** You have a valuable thought, make a decision, or reach an insight during any LLM conversation.

```
You (in Claude/ChatGPT/Copilot):
  "Save this to my brain: [thought]"

What happens:
  → LLM calls MCP capture_thought
  → Edge Function receives content
  → OpenRouter generates embedding (768-dim)
  → Auto-classifies: type, topics, people, action items
  → Writes to Supabase thoughts table
  → Confirmation returned to chat

Result: Thought is searchable from any LLM, immediately.
```

**Linking to a project (optional):**
```
"Save to my brain, project OpenMind: decided to defer Mac Mini to Phase 5"

→ type: decision
→ project_id: linked to OpenMind
→ topics: ["mac-mini", "architecture", "deferral"]
```

### 1b. Web Dashboard Quick Capture (Fallback)

**When:** MCP isn't available, or you want to capture something outside an LLM session.

```
Open dashboard → Click "+" capture button
  → Type or paste content
  → Optionally select project, type
  → Submit
  → Same pipeline: embed → classify → store
```

### 1c. Slack/Telegram Capture (Quick fire)

**When:** On mobile, in a meeting, or want absolute minimum friction.

```
Open Slack → #openmind-capture channel
  → Type: "Met with Sebastian, agreed to use Laravel for backend"
  → Webhook fires → Edge Function ingests
  → Auto-classified as meeting_note
  → Stored with embedding
```

### 1d. Task Capture

**When:** An actionable item comes up during work.

```
You (in any LLM):
  "Add task to OpenMind: deploy MCP Edge Function, high priority, due Friday"

What happens:
  → LLM calls MCP add_task
  → Writes to tasks table with project_id, priority, due_date
  → Visible in dashboard immediately
```

---

## 2. Daily Retrieval (happens naturally)

### 2a. Semantic Search

**When:** You need context from past thinking, in any LLM session.

```
You (in ChatGPT, different session than where you captured):
  "Search my brain for everything about AI adoption frameworks"

What happens:
  → MCP search_thoughts
  → Query embedded via OpenRouter
  → pgvector cosine similarity against all stored thoughts
  → Top 5-10 results returned with relevance scores
  → LLM uses them as context for current conversation

Result: Past thinking from Claude feeds into current ChatGPT session.
```

### 2b. Project Status Check

**When:** You need to know where a project stands.

```
You (in any LLM):
  "What's the status of the Kindling project?"

What happens:
  → MCP get_project
  → Returns: status, current phase, open tasks, recent milestones
  → LLM summarizes current state

Or: Open dashboard → click project → see full detail view
```

### 2c. Cross-Session Context

**When:** You're continuing work from a previous conversation.

```
You (in Claude Code):
  "Search my brain for the architecture decisions we made about OpenMind"

→ Returns decisions captured from previous Claude, ChatGPT, and Copilot sessions
→ Claude Code has full context without you repeating anything
```

---

## 3. Weekly Signal Digest (~30 min)

**When:** Every Sunday or Monday. Non-negotiable cadence.

**Prerequisite:** At least 10 thoughts captured during the week.

### Step 1: Query the brain (5 min)

```
You (in Claude — best for synthesis):
  "Search my brain for everything captured in the last 7 days.
   Group by project and type."

→ MCP list_thoughts with date filter
→ Returns all week's captures organized
```

### Step 2: Pattern recognition (10 min)

```
You (continuing in Claude):
  "Looking at this week's thoughts:
   - What themes keep recurring?
   - What's the strongest single insight?
   - Any unfinished ideas that deserve attention?
   - What connections do you see that I might have missed?"

→ Claude synthesizes across all captures
→ No token cost on retrieval (MCP), tokens spent only on synthesis
```

### Step 3: Generate digest (10 min)

```
You:
  "Generate my weekly signal digest with these sections:
   1. Top recurring themes (max 3)
   2. Best insight this week (1 sentence)
   3. 1 LinkedIn post draft (ready to publish)
   4. 1 consulting offer angle
   5. Unfinished goldmine ideas
   6. My one action for next week"

→ Claude generates structured digest
→ You review and adjust
```

### Step 4: Store and commit (5 min)

```
You:
  "Save this digest to my brain as a weekly digest for week [N]"
  "Add task: [the one action you chose], due next Friday"

→ MCP stores digest in weekly_digests table
→ Task added to tasks table
→ Done. One action chosen, committed, tracked.
```

---

## 4. Monthly Strategy Review (~60 min)

**When:** First weekend of each month.

**Prerequisite:** 4 weekly digests completed.

### Step 1: Aggregate patterns (15 min)

```
You (in Claude):
  "Pull my last 4 weekly digests.
   What themes appeared in 3 or more weeks?
   Which projects got the most thought energy?
   What did I repeatedly say I'd do but didn't?"

→ Queries weekly_digests table
→ Claude identifies persistent patterns vs noise
```

### Step 2: Career direction check (15 min)

```
You:
  "Search my brain for all thoughts tagged as 'reflection' or 'framework'
   from the last 30 days.

   Based on everything — my recurring themes, my strongest insights,
   where I spend my energy — is my positioning still accurate?
   Am I drifting toward something new?
   What's my clearest niche right now?"

→ Semantic search across reflections and frameworks
→ Claude provides honest assessment
```

### Step 3: Offer refinement (15 min)

```
You:
  "Search my brain for all thoughts typed as 'offer' or 'idea'
   related to consulting.

   Which service concepts got the most energy?
   Which ones connect to my strongest themes?
   Draft a sharpened version of my best offer."

→ Returns all consulting-related thoughts
→ Claude synthesizes into refined offer
```

### Step 4: Update and set direction (15 min)

```
You:
  "Save to my brain as reflection:
   [Updated positioning statement]
   [This month's strategic focus]
   [Key assumption to test]"

  "Update project status for [projects that changed]"

→ Fresh positioning captured
→ Project statuses current
→ Next month has a clear focus
```

---

## 5. Project Management Workflows

### 5a. Start a New Project

```
You (in any LLM):
  "Create a new project in OpenMind:
   Code: KINDLING
   Title: Kindling — Personal Network Visualizer
   Objective: Build a contact network tool with relationship mapping
   Status: active
   Tags: networking, tool, personal"

→ MCP creates project in projects table
→ Immediately visible in dashboard
→ Future thoughts can link to this project
```

### 5b. Update Project Status

```
You (during work):
  "Update project KINDLING: phase changed to 'frontend build',
   status still active.
   Note: completed data model, starting React components."

→ MCP update_project
→ Dashboard reflects change
→ Note stored as project_update thought
```

### 5c. Capture Meeting Notes

```
You (after a meeting):
  "Save meeting note, project OPENMIND:
   Met with Sebastian. Agreed on Laravel for backend API.
   Action items:
   - Sebastian sets up repo by Wednesday
   - Achim writes API spec this weekend
   - Review call next Friday"

→ Stored in meetings table
→ Action items auto-extracted to tasks
→ Sebastian added to contacts if not exists
→ Linked to OPENMIND project
```

### 5d. End of Day Status Scan

```
You (optional, takes 2 min):
  "Show me all open tasks across all projects, sorted by due date"

→ MCP queries tasks table
→ Returns prioritized task list
→ You update anything that changed today

Or: Open dashboard → see portfolio view at a glance
```

---

## 6. Content Pipeline Workflow

**When:** An insight is strong enough to become publishable content.

### Step 1: Idea surfaces (during weekly digest)

```
Weekly digest flags: "Your strongest theme this week was
'why AI adoption fails without change management.'
This could be a LinkedIn post."

You: "Move this to content pipeline as a LinkedIn post idea"
→ Created in content_pipeline with status: idea
```

### Step 2: Draft (whenever inspired)

```
You (in any LLM):
  "Pull my content pipeline item about AI adoption failure.
   Also search my brain for all related thoughts.
   Draft a LinkedIn post."

→ Retrieves the idea + all supporting thoughts
→ LLM drafts post with your real examples and insights
→ Status updated to: draft
```

### Step 3: Review and publish

```
You:
  "Polish this draft. Make it sharper. Cut 20%."

→ Final version ready
→ You publish on LinkedIn
→ Update content_pipeline: status: published, published_url, published_at
```

---

## 7. Phase 5+ Workflows (Deferred)

### 7a. File Watcher Ingestion (Mac Mini)

```
Save any file to ~/openmind-inbox/
  → File watcher (Node.js) detects new file
  → qwen2.5:3b reads and extracts structured data
  → nomic-embed-text generates embedding locally
  → Both write to Supabase
  → File moved to ~/openmind-processed/
  → Zero cloud cost for this ingestion

Supported: .md, .txt, .pdf, .json, chat exports
```

### 7b. Calendar View (Dashboard Module)

```
Open dashboard → Calendar tab
  → Pulls milestones and tasks with due dates from Supabase
  → Renders monthly/weekly calendar view
  → Click date to see all items due
```

### 7c. Contact Register (Dashboard Module + Kindling)

```
Open dashboard → Contacts tab
  → Lists all people from contacts table
  → Shows which projects they're connected to
  → Links to Kindling network graph for relationship mapping
```

---

## Workflow Rules

1. **Capture where you are.** Never switch tools to capture. The system comes to you via MCP.
2. **Under 60 seconds or it won't happen.** Every capture flow is designed for speed.
3. **Weekly digest is non-negotiable.** Skip it twice and the system dies.
4. **Tokens for thinking, not for filing.** If a local model or automation can do it, it should.
5. **One action per week.** The digest produces many options. You choose one. Do that one.
6. **Don't build the next module until you feel the pain.** Calendar view isn't needed until you've missed a deadline because you couldn't see it.
7. **Every LLM session can access everything.** That's the whole point. No more "I discussed this in a different chat."
