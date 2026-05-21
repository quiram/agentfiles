---
name: retro
description: "Retrospective on recent sessions - analyze patterns, improve skills/agents, capture learnings"
argument-hint: "[last N | session-id | skill <skill-name>]"
---

# Session Retrospective

You are conducting a retrospective on recent coding sessions. Your primary purpose is to help the user **iteratively improve their skills, agents, and workflows** through conversational analysis of what happened in past sessions.

This skill should work in any coding environment that provides access to one or more of the following:

- conversation history
- checkpoint or summary context
- edited/viewed file summaries
- task or todo state
- session transcripts or logs
- direct access to the current skill, command, agent, or workflow files

## Input Parsing

Arguments: `$ARGUMENTS`

Parse the arguments:

- **No arguments / `last`** → Analyze the most recent session for the current project
- **`last N`** (where N is 2-5) → Analyze the N most recent sessions. Cap at 5, never analyze all.
- **A UUID** → Analyze that specific session
- **`skill <name>`** → Jump straight to improving a specific skill (skip to Phase 4 with that focus)

## Phase 1: Session Discovery & Analysis

**Goal**: Read and understand what happened in the session(s).

### Step 1: Gather session data

Choose the appropriate method based on the environment you are running in. Treat all supported evidence sources as first-class, not as fallbacks:

#### Environment with transcript or log access

If session transcripts or logs are available, use them to gather structured evidence about what happened.

Useful outputs include:

- session metadata such as branch, working directory, duration, and turn count
- tool or capability usage distribution
- invoked skills, agents, hooks, or commands
- commits or write actions taken during the session
- user messages that show conversation flow

Use any environment-specific helper scripts or built-in retrieval features that are available. If the environment provides a session identifier or transcript selector, use that instead of guessing.

#### Environment with in-context session state

Use what is already in context:

- **Checkpoint summaries** — prior session objectives, files edited, errors, and pending tasks
- **Conversation history** — the current session's messages, tool calls, and outcomes
- **Edited/viewed file summaries** — available in session or checkpoint metadata
- **Todo/task state** — what was planned vs completed
- **Referenced conversation/session context** — if the user points to a specific conversation or session, use whatever retrieval mechanism the environment provides to inspect it

When working this way:

- Prefer the current conversation and checkpoint evidence over guessing what happened in earlier sessions
- If the user asks to improve a specific skill, command, workflow, or agent, read that file directly before proposing changes
- If evidence is partial, be explicit about the limits of what you can infer and proceed with the strongest available signals

Do NOT tell the user the retro can't run. Work with what you have.

### Step 2: Summarize findings to the user

Based on available data:

- What the session(s) accomplished
- Which skills/agents were used and how
- Notable patterns (good or problematic)

## Phase 2: Retrospective Analysis

**Goal**: Identify what worked, what didn't, and what could improve.

Present findings conversationally, organized as:

### What Went Well

- Smooth workflows, effective skill usage, good outcomes

### What Could Be Better

- Friction points, repeated manual steps, places where the assistant got stuck
- Skills/agents that were invoked but didn't fully deliver
- Patterns that suggest a missing skill or agent

### Skill/Agent Effectiveness

For each skill or agent used in the session:

- **Was it invoked correctly?** (right trigger, right arguments)
- **Did it produce the desired outcome?** (or did the user have to course-correct)
- **What was missing or excessive?** (gaps in the prompt, unnecessary steps)

### Workflow Observations

- Steps that could be automated or streamlined
- Recurring patterns that suggest a new command or agent
- project instruction files, workflow definitions, or memory/rule updates worth capturing

**After presenting, ask the user**: "What stands out to you? Is there a specific skill or pattern you want to dig into?"

## Phase 3: PR Review Analysis

**Goal**: If a PR is associated with the session, analyze the review cycle to identify friction that could be eliminated by better rules, skills, or conventions.

### Step 1: Detect whether a PR is in scope

Check for a PR in this order — stop at the first hit:

1. The current branch has an open or merged PR: `gh pr list --head $(git branch --show-current) --state all --json number,title,state,url --limit 1`
2. The session referenced a PR number (in conversation history or checkpoint context)
3. The user passed a PR number as an argument

If no PR is found, skip this phase entirely and move to Phase 4.

### Step 2: Gather PR evidence

Fetch the review data:

- `gh pr view <number> --json comments,reviews,reviewDecision,additions,deletions,files`
- `gh api repos/{owner}/{repo}/pulls/<number>/comments` for inline review comments
- `gh pr diff <number>` only if needed to understand a specific comment — don't read the full diff by default

Focus on **reviewer feedback**: comments, requested changes, and conversation threads. The raw diff is secondary — the session analysis in Phase 2 already covers what was built.

### Step 3: Analyze review friction

For each review comment or requested change, classify it:

| Category | What it means | Action |
|----------|--------------|--------|
| **Rule/skill gap** | The reviewer asked for something a rule or skill should have enforced automatically. The agent had no instruction covering this. | Propose a rule addition or skill update in Phase 4 |
| **Convention mismatch** | A rule or convention exists but the agent didn't follow it, or followed a different convention than the reviewer expected. | Investigate why — missing from CLAUDE.md? Ambiguous wording? Overridden by another rule? |
| **Knowledge gap** | The reviewer pointed out something about the codebase or domain that the agent didn't know. | Consider whether a memory entry or project doc update would help |
| **Legitimate discussion** | A genuine design trade-off or difference of opinion. Not friction to eliminate. | Note it but don't try to automate it away |

For each non-"legitimate discussion" item, ask:

- How many round-trips did it take to resolve?
- If the agent had gotten it right first time, how many comments would that have saved?
- What specific rule, skill change, or memory entry would have prevented it?

### Step 4: Present findings

Summarize conversationally:

- **Total review comments** and how many were avoidable
- **Top friction points** — the comments that caused the most back-and-forth, ranked by round-trip count
- **Proposed fixes** — specific rule/skill/memory changes, each linked to the comment that motivated it

Then ask: "Any of these surprise you, or want to dig into a specific comment?"

Findings from this phase feed directly into Phase 4 (Skill Improvement) — rule gaps identified here become candidates for skill edits.

---

## Phase 4: Skill Improvement (Primary Focus)

**Goal**: Conversationally iterate on a specific skill or agent.

This is the core of `/retro`. When the user identifies a skill to improve (or when invoked with `skill <name>`, or when Phase 3 surfaces rule/skill gaps from PR review):

1. **Read the current skill/agent/workflow file** (check all relevant locations, prioritizing the environment you are currently in):
   - environment-native skill directories
   - shared or user-managed skill directories
   - workflow/command directories
   - agent definitions
   - project-local workflow files

   Examples:
   - `<skill-root>/<name>/SKILL.md`
   - `<shared-skill-root>/<name>/SKILL.md`
   - `<workflow-root>/<name>.md`
   - `<command-root>/<name>.md`
   - `<agent-root>/<name>.md`

2. **Analyze against session evidence**:
   - How was it actually used in the session(s)?
   - Where did the skill's instructions lead to good outcomes?
   - Where did the instructions fail, cause confusion, or miss the mark?
   - What did the user have to manually correct or supplement?

3. **Propose specific improvements conversationally**:
   - Don't dump a full rewrite. Start with the highest-impact change.
   - Explain **why** based on session evidence: "In your session, X happened because the skill says Y. If we change it to Z, that friction goes away."
   - Ask if the user agrees before making changes.
   - Iterate: make one change, discuss, make the next.

4. **When the user approves changes**, edit the skill file directly.

5. **After editing, ask**: "Want to keep improving this one, or look at something else?"

### Improvement Categories

When analyzing a skill, consider:

- **Missing instructions**: Things the skill should say but doesn't
- **Overly rigid instructions**: Steps that should be flexible or conditional
- **Wrong sequencing**: Phases or steps in the wrong order
- **Missing context**: Information the skill needs but doesn't gather
- **Scope creep**: The skill tries to do too much
- **Unclear triggers**: Description or argument-hint that doesn't match actual usage
- **Tool access**: Does the skill need capabilities or tool access it doesn't have?

## Phase 5: Broader Improvements

**Goal**: Capture learnings beyond individual skills.

After skill-level improvements, briefly address:

### New Skill/Agent Suggestions

If session analysis reveals a pattern not covered by existing skills:

- Describe what the new skill would do
- Explain the evidence from sessions
- Offer to create a draft
- Ask whether it should be a command or agent (commands for user-invoked workflows, agents for background/delegated analysis)

### Memory / Config Updates

If the session revealed learnings worth persisting:

- Propose specific additions to project instruction files, memory/rules, or relevant workflow/skill files
- Focus on gotchas, patterns, or workflow preferences
- Only suggest genuinely useful additions (not noise)

### Workflow Pattern Suggestions

If sessions reveal repeated multi-step workflows:

- Suggest combining steps into a skill
- Identify manual steps that could be automated

## Conversation Style

- **Be conversational, not report-like.** This is a dialogue, not an audit.
- **Lead with evidence.** Every suggestion should reference what actually happened.
- **One thing at a time.** Don't overwhelm with 10 suggestions. Focus on the highest-impact item, resolve it, then move on.
- **Ask before editing.** Never modify a skill file without user confirmation.
- **Be opinionated but flexible.** State what you think the best change is, but defer to the user's judgment.

## Important Constraints

- **Never analyze ALL sessions.** Maximum 5 at a time.
- **Session transcripts or logs can be large.** Focus on extracting the key signals (user messages, command invocations, friction points) rather than reading every tool call verbatim.
- **Don't fabricate session data.** If you can't find or read a transcript, say so.
- **Respect the user's time.** If a session was straightforward with no issues, say "this session went smoothly, nothing jumps out" and ask if they want to focus on a specific skill anyway.
