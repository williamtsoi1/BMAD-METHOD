# AI Agent Development Expansion Pack - Installation Guide

## What You Just Created

A comprehensive AI agent development expansion pack based on MongoDB's CANVAS framework, integrated into BMAD-METHOD™.

## Package Contents

### 📦 7 Specialized Agents

1. **Agent Orchestrator** (`agent-orchestrator.md`) - Primary guide through CANVAS framework
2. **Product Strategist** (`product-strategist.md`) - User problem validation specialist
3. **Agent Architect** (`agent-architect.md`) - Agent capabilities and behavior designer
4. **Data Strategist** (`data-strategist.md`) - Knowledge requirements and data pipeline expert
5. **Model Integration Specialist** (`model-integration-specialist.md`) - LLM selection and API integration
6. **Production Architect** (`production-architect.md`) - Enterprise scaling specialist
7. **Security & Governance Advisor** (`security-governance-advisor.md`) - Compliance and policy expert
8. **ROI Analyst** (`roi-analyst.md`) - Business value measurement specialist

### 📋 2 Canvas Templates

1. **POC Canvas** (`poc-canvas-tmpl.yaml`) - 8-square validation canvas
2. **Production Canvas** (`production-canvas-tmpl.yaml`) - 11-square scaling canvas

### ✅ Tasks & Checklists

- `assess-failure-risks.md` - Identify 6 critical failure patterns
- `complete-poc-canvas.md` - Guided POC Canvas workflow
- `complete-production-canvas.md` - Guided Production Canvas workflow
- `production-readiness-checklist.md` - 90+ item production validation

### 📚 Knowledge Base

- `canvas-framework-kb.md` - Complete CANVAS framework methodology

## Installation

### Option 1: Via BMAD Installer (Recommended)

```bash
npx bmad-method install
# Select "AI Agent Development Studio" when prompted
```

### Option 2: Manual Installation

```bash
# From BMAD-METHOD root directory
npm run build -- --expansions-only

# Bundles created in:
# dist/expansion-packs/bmad-ai-agent-dev/agents/
```

## Usage

### In IDE (Cursor, VS Code with Claude Code)

```bash
# Activate the orchestrator
@agent-orchestrator start
```

### In Web UI (Gemini, ChatGPT)

1. Navigate to `dist/expansion-packs/bmad-ai-agent-dev/agents/`
2. Upload `agent-orchestrator.txt`
3. Set instructions: "Your critical operating instructions are attached, do not break character as directed"
4. Start with: `*start` or `*help`

## Quick Start Workflow

### For New AI Agent Projects

1. **Start with Agent Orchestrator**

   ```
   *start
   ```

2. **Run Failure Risk Assessment**

   ```
   *failure-check
   ```

3. **Complete POC Canvas (8 Squares)**

   ```
   *poc-canvas
   ```

   - Work through: Product → Agent → Data → Model
   - Validate with real users before building
   - Build POC in days, not months

4. **Test POC with Users**
   - At least 10-20 real users
   - Measure actual business value
   - Only scale what works

5. **Complete Production Canvas (11 Squares)**
   ```
   *production-canvas
   ```

   - Only after POC validation
   - Address security, governance, scalability
   - Deploy with confidence

### For Existing Projects

1. **Assess Current State**

   ```
   *failure-check
   ```

2. **Identify Gaps**
   - Which canvas squares are incomplete?
   - Which failure patterns exist?
   - What's missing for production?

3. **Backfill Validation**
   - Complete missing POC Canvas squares
   - Validate with real users
   - Address security/governance gaps

4. **Scale Systematically**
   - Use Production Canvas for gaps
   - Deploy when ready (not before)

## Key Concepts

### The Product → Agent → Data → Model Flow

**Traditional (95% Failure):**

```
Data → Model → Product
❌ Technology-first
❌ 10+ months to value
❌ Massive investment
```

**CANVAS (25% Success):**

```
Product → Agent → Data → Model
✅ Problem-first
✅ Days to value
✅ Validate before scaling
```

### The 6 Critical Failure Patterns

1. **Technology-First Trap** - 95% failure rate
2. **Capability Reality Gap** - Ignoring 24% task completion reality
3. **Leadership Vacuum** - No CEO/C-level sponsorship
4. **Security & Governance Barriers** - 56% project killer
5. **Infrastructure Chaos** - Three-database management hell
6. **ROI Mirage** - 80% see no business earnings

### Success Statistics

- **25%** achieve transformative results (problem-first approach)
- **Moderna**: 750+ agents deployed successfully
- **Current AI**: 24% office task completion (Claude 3.5 Sonnet)

## File Structure

```
expansion-packs/bmad-ai-agent-dev/
├── config.yaml                     # Pack metadata
├── README.md                       # Comprehensive guide
├── docs/
│   └── INSTALLATION.md            # This file
├── agents/                         # 8 specialized agents
│   ├── agent-orchestrator.md
│   ├── product-strategist.md
│   ├── agent-architect.md
│   ├── data-strategist.md
│   ├── model-integration-specialist.md
│   ├── production-architect.md
│   ├── security-governance-advisor.md
│   └── roi-analyst.md
├── templates/                      # Canvas templates
│   ├── poc-canvas-tmpl.yaml
│   └── production-canvas-tmpl.yaml
├── tasks/                          # Key workflows
│   ├── assess-failure-risks.md
│   ├── complete-poc-canvas.md
│   └── complete-production-canvas.md
├── checklists/
│   └── production-readiness-checklist.md
└── data/
    └── canvas-framework-kb.md     # Framework knowledge
```

## Build Output

After running `npm run build`:

```
dist/expansion-packs/bmad-ai-agent-dev/
└── agents/
    ├── agent-orchestrator.txt          # 305 lines
    ├── product-strategist.txt          # 200+ lines
    ├── agent-architect.txt             # 150+ lines
    ├── data-strategist.txt             # 150+ lines
    ├── model-integration-specialist.txt
    ├── production-architect.txt        # 200+ lines
    ├── security-governance-advisor.txt
    └── roi-analyst.txt                 # 200+ lines
```

## Validation

The expansion pack passed all validation checks:

```bash
npm run validate
✓ All configurations are valid!

npm run build -- --expansions-only
✓ Built 8 agent bundles successfully
```

## Next Steps

1. **Read the README.md** - Comprehensive expansion pack guide
2. **Try the orchestrator** - Upload `agent-orchestrator.txt` to web UI
3. **Run a POC Canvas** - Practice with a test project
4. **Share feedback** - Contribute improvements

## Support

- 📖 [Full README](../README.md)
- 🎯 [CANVAS Framework KB](../data/canvas-framework-kb.md)
- 🌐 [MongoDB CANVAS Framework](https://www.mongodb.com/company/blog/technical/build-ai-agents-worth-keeping-canvas-framework)
- 💬 [BMad Discord Community](https://discord.gg/gk8jAdXWmj)

## Attribution

Based on MongoDB's CANVAS Framework by Mikiko Bazeley (September 2024).
Adapted for BMAD-METHOD™ by the BMad community.

---

**Remember:** Start with problems, not technology. Validate before scaling. Only build agents worth keeping.

**Goal:** Join the successful 25%, not the failing 75%.
