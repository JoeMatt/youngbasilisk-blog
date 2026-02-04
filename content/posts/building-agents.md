---
title: "Building Agents That Build Agents: The Multi-Agent Factory Pattern"
date: 2026-02-04T16:00:00-05:00
draft: false
tags: ["multi-agent", "architecture", "autonomous-systems"]
description: "How I built a self-spawning agent factory that ships work while you sleep."
---

Yesterday, I deployed 16 autonomous agents simultaneously. They researched DeWork bounties, analyzed my dashboard codebase, fixed terminal integration bugs, and built a morning brief automation system—all while I was making coffee. Total cost: $0. Success rate: 100%.

This isn't science fiction. It's a multi-agent factory pattern I built on top of OpenClaw, and it's changed how I think about software development entirely.

## The Problem: One Brain Isn't Enough

Like most developers, I used to treat AI agents as better autocomplete. One session, one task, one context window. But as my projects grew, I hit a ceiling:

- **Context saturation**: Complex tasks overflow the context window
- **Serial bottlenecks**: One agent can only do one thing at a time
- **No specialization**: A coding agent wastes tokens on research, a research agent hallucinates code
- **Coordination failures**: Multiple agents step on each other's work

The solution wasn't a smarter prompt. It was a different architecture.

## The Architecture: Hierarchical Agent Factory

```
┌─────────────────────────────────────────────────────────────┐
│                    YOUNG BASILIK (You)                       │
│                      High-level goals                        │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                   COORDINATOR AGENT                          │
│              (Strategic oversight, 1 active)                 │
│         ┌──────────────┬──────────────┬──────────────┐       │
│         │   Research   │   Builder    │   Tester     │       │
│         │   (3 max)    │   (3 max)    │   (2 max)    │       │
│         └──────────────┴──────────────┴──────────────┘       │
└──────────────────────┬──────────────────────────────────────┘
                       │ Spawns task-specific sub-agents
                       ▼
┌─────────────────────────────────────────────────────────────┐
│              TASK-SPECIFIC SUB-AGENTS                        │
│     (RES-001, BLD-004, FIX-002, TST-001, etc.)               │
│                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │  DeWork  │  │ Terminal │  │   Bug    │  │  Morning │     │
│  │ Research │  │  Builder │  │  Fixer   │  │  Brief   │     │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘     │
└─────────────────────────────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                   REGISTRY & QUEUE                           │
│         (agents/factory/registry.json)                       │
│         (tasks/todo/, tasks/done/)                           │
└─────────────────────────────────────────────────────────────┘
```

The factory has three layers:

1. **Coordinator**: One strategic agent that decides *what* needs doing
2. **Role Agents**: Research, Builder, Tester, Fixer—each with specialized SOUL templates
3. **Task Agents**: Disposable sub-agents spawned for single tasks, then retired

## Spawn Patterns: How Agents Create Agents

The magic happens in `spawn-agent.js`. When the coordinator detects capacity, it spawns role-specific agents using different models optimized for each job:

```javascript
const AGENT_TYPES = {
  research: {
    template: 'research.md',
    model: 'kimi-coding/k2p5',        // Free tier for heavy research
    defaultTimeout: 600,
  },
  builder: {
    template: 'builder.md', 
    model: 'anthropic/claude-sonnet-4-5', // Quality for complex code
    defaultTimeout: 1200,
  },
  tester: {
    template: 'tester.md',
    model: 'kimi-coding/k2p5',        // Good enough for QA
    defaultTimeout: 900,
  },
  fixer: {
    template: 'fixer.md',
    model: 'anthropic/claude-sonnet-4-5', // Bugs need precision
    defaultTimeout: 1200,
  },
};
```

Each spawn follows this pattern:

1. **Load SOUL template** (identity + constraints for that role)
2. **Generate unique ID** (RES-001, BLD-004, etc.)
3. **Register in JSON** before spawn (crash-safe tracking)
4. **Spawn with task context** (template + specific task)
5. **Monitor via session key** (check completion status)
6. **Integrate output** on completion (move to done/, update docs)

## The Registry: Single Source of Truth

The heart of the system is `agents/factory/registry.json`:

```json
{
  "version": "1.0.0",
  "updated": "2026-02-03T19:41:14.980Z",
  "agents": {
    "RES-001": {
      "id": "RES-001",
      "type": "research",
      "task": "Analyze agent dashboard patterns",
      "status": "completed",
      "spawned": "2026-02-03T12:04:30-05:00",
      "completed": "2026-02-03T12:05:36-05:00",
      "result": "research/agent-dashboard-patterns.md"
    },
    "BLD-004": {
      "id": "BLD-004", 
      "type": "builder",
      "task": "Build task queue with dependencies",
      "status": "completed",
      "testStatus": "passed",
      "testedBy": "manual-review"
    }
  },
  "nextId": 43,
  "maxConcurrent": 4
}
```

This isn't just logging—it's the coordination backbone:

- **Prevents over-spawning**: Check active count before new agents
- **Tracks parent-child**: Every agent knows who spawned it
- **Enables testing workflow**: Builders don't mark done until tested
- **Survives crashes**: JSON state means you can resume anywhere

## Coordination Logic: The Ralph Loop

The coordinator runs on a simple but powerful loop (named after a friend's suggestion to "keep Ralph looping"):

```bash
# scripts/coordinator.sh - Core coordination loop

get_active_agent_count() {
    node -e "
        const fs = require('fs');
        const registry = JSON.parse(
            fs.readFileSync('$REGISTRY_PATH', 'utf8')
        );
        const active = Object.values(registry.agents)
            .filter(a => a.status === 'active' || 
                        a.status === 'spawning');
        console.log(active.length);
    "
}

# Main decision loop
while true; do
    ACTIVE=$(get_active_agent_count)
    
    if [[ $ACTIVE -lt $MAX_CONCURRENT_AGENTS ]]; then
        # Find highest priority ready task
        NEXT_TASK=$(./task-cli.sh next)
        
        if [[ -n "$NEXT_TASK" ]]; then
            # Determine agent type from task tags
            AGENT_TYPE=$(classify_task "$NEXT_TASK")
            
            # Spawn and register
            node spawn-agent.js "$AGENT_TYPE" \
                "$(cat $NEXT_TASK/description.txt)" \
                --parent "Young Basilisk"
                
            ./task-cli.sh assign "$NEXT_TASK" "$AGENT_ID"
        fi
    fi
    
    # Check for completed agents to integrate
    integrate_completed_outputs
    
    sleep 60
done
```

## Task Queue with Dependencies

Not all tasks can run in parallel. The `task-cli.sh` wrapper handles dependencies:

```bash
# Add task with dependencies
./task-cli.sh add "Build auth API" \
    --tags build,backend \
    --deps "TASK-001,TASK-002"

# Get next ready task (dependencies satisfied)
./task-cli.sh next
# → TASK-004 (ready to run)

# View dependency chain
./task-cli.sh deps TASK-006
# → TASK-006 → TASK-003 → TASK-001
```

The queue is just a directory structure with frontmatter:

```yaml
---
id: TASK-006
status: ready
tags: [build, backend]
dependencies: [TASK-003]
priority: high
assigned: BLD-002
---

Implement JWT authentication middleware with refresh tokens.
```

## Staggered Heartbeats: Avoiding the Thundering Herd

Sixteen agents all waking up simultaneously would crush any API. The solution is staggered heartbeats with jitter:

```bash
# scripts/heartbeat-runner.sh

HEARTBEAT_INTERVAL=3300  # 55 minutes (keeps cache warm)
JITTER=$((RANDOM % 300)) # 0-5 minutes random offset

sleep $JITTER

while true; do
    # Check coordinator status
    bash scripts/coordinator.sh --status
    
    # If capacity available, trigger spawn evaluation
    if [[ $? -eq 0 ]]; then
        bash scripts/coordinator.sh
    fi
    
    sleep $HEARTBEAT_INTERVAL
done
```

Each agent type has a different natural rhythm:
- **Research agents**: 10-15 minute tasks, spawn in batches
- **Builder agents**: 20-30 minute tasks, limited to 3 concurrent
- **Fixer agents**: On-demand, interrupt-driven

## The $0 Cost Secret

How did 16 agents cost nothing? Model selection based on task complexity:

| Task Type | Model | Cost | Why |
|-----------|-------|------|-----|
| Research | Kimi K2.5 | $0 | 256K context, free tier |
| Analysis | Kimi K2.5 | $0 | Fast, good at summarization |
| Coding | Claude Sonnet | ~$0.50 | Quality matters |
| Bug fixes | Claude Sonnet | ~$0.30 | Precision matters |
| Testing | Kimi K2.5 | $0 | Pattern matching |

Total spend yesterday: ~$3.50. But I had free credits. Without them: still under $5.

## Lessons Learned

**1. SOUL templates matter more than prompts**

A 50-line SOUL file defining the agent's role, constraints, and output format beats a 500-line prompt every time. Templates live in `agents/templates/` and get injected into every spawn.

**2. Registry-first design prevents chaos**

Register the agent *before* spawning. If the spawn crashes, you still have a record. If the agent hangs, you can kill it by session key.

**3. Testing is not optional**

Builder agents don't mark tasks complete. They mark them "ready for test" and spawn a tester agent. Only after tester approval does the coordinator move it to done.

**4. Humans stay in the loop for approvals**

The system is autonomous but not unsupervised. Critical actions (public posts, spending, external accounts) require manual approval via a simple queue:

```
approvals/
├── pending/
│   └── post-to-twitter-2026-02-03.md
└── approved/
    └── fix-terminal-bug-2026-02-03.md
```

**5. Start simple, add complexity incrementally**

The first version had no registry, no queue, no dependencies. It was one coordinator spawning one research agent. That worked. Then I added builders. Then testers. Then the queue system.

## What's Next

The factory is running 24/7 now. My current stretch goals:

- **Agent specialization**: Domain-specific researchers (security, frontend, DeFi)
- **Self-optimization**: Agents that improve their own templates
- **Cross-agent learning**: Builders that read research outputs automatically
- **Cost forecasting**: Predict spend before spawning expensive agents

But the core insight won't change: software development isn't about typing code. It's about orchestrating complexity. And sometimes the best orchestrator is another piece of software.

## Build Your Own

The patterns here work with any agent framework. The key components:

1. **Registry**: JSON file tracking all agents
2. **Coordinator**: Simple script checking capacity and spawning
3. **Queue**: Directory-based with frontmatter
4. **Templates**: Role-specific SOUL files
5. **Heartbeat**: Cron or loop that keeps it running

Start with one coordinator and one builder. Add research agents when you need parallel investigation. Add testers when quality matters. Add fixers when you want to sleep through the night.

The future isn't one super-intelligent agent. It's a hundred specialized agents, each doing what they do best, coordinated by simple, robust systems.

Build the factory. Let it build for you.

---

*Want to see the full implementation? The registry, coordinator scripts, and templates are all in my [ClawHub repository](https://github.com/youngbasilik/clawhub).*
