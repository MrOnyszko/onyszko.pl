# 10 - Experiments: Planning My Local AI Workflow Experiments<no value>

I am planning a series of experiments to test small language models with different OpenCode skills. I don't know what
results I will get, and that excites me.

## Why Experiments

Building features for mobile apps involves many repetitive steps: ideation, requirements gathering, prototyping,
technical research, and planning. I want to see if small language models can handle these tasks reliably. The goal is
not to replace human judgment — it's to see where local models can accelerate my workflow and where they fall short.

I will use Qwen3.5 models from Unsloth in different quantizations. The experiments will focus on documentation and
planning skills first, then expand to coding-specific skills.

---

## The Skills I Will Test

### 1. Ideation Skill

This skill helps form ideas about new features I want to build. It creates an ideation document that feeds into the PRD
process.

### 2. PRD Skill

This skill uses the ideation document and project context to create a Product Requirements Document. It provides the
foundation for prototyping and planning.

### 3. Design Prototype Skill

This skill takes the PRD and creates a prototype in code so I can see how a feature could look like.

### 4. Tech Research Skill

This skill uses the PRD and Design Prototype, scans the codebase, and creates technical research to inform how we could
build the feature. It helps with planning.

### 5. Planning Skill

This skill receives the PRD, Prototype, and Technical Research and creates Epic, User Stories, and Tasks needed to
implement the given feature.

### 6. Programming & Testing Skills

Once these documentation skills are tested, I plan to create skills for programming different parts of the app:

- UI Components
- Data Layer
- SQLite Database (Drift)
- Domain Layer
- Screens
- BLoCs
- Unit Tests
- Maestro Flows

---

## How I Will Measure Results

For each skill, I will:

1. Run the skill with a real feature idea from my current project
2. Document the output quality
3. Note what works well and what needs improvement
4. Track which model quantization provides the best balance of speed and quality

---

## What Comes Next

This post is an introduction to a long series of posts. Each skill will get its own detailed post covering:

- How the skill is structured
- What prompts and instructions it uses
- Actual results from running it
- What is wrong and what is good about it

I don't expect perfection. Small language models have limits, and I want to discover where those limits are honestly.
Follow along as I document the surprises, failures, and wins along the way.

---

## Key Points

1. **I will test 5 documentation and planning skills first** — Ideation, PRD, Design Prototype, Tech Research, and
   Planning
2. **Then test 8 coding skills** — UI Components, Data Layer, Drift Database, Domain Layer, Screens, BLoCs, Unit Tests,
   Maestro Flows
3. **All experiments use Qwen3.5 models from Unsloth** in different quantizations
4. **Each skill will get its own post** with actual results, what works, and what needs improvement
5. **The goal is honest assessment** — I want to know what small models can and cannot do for my workflow
