# Complete POC Canvas

## Purpose

Guide users through systematic completion of the 8-square POC Canvas to validate AI agent concepts before building.

## Prerequisites

Before starting POC Canvas:

- [ ] Failure risk assessment completed (use `assess-failure-risks.md`)
- [ ] No critical (RED) risks identified
- [ ] Leadership buy-in secured or in progress
- [ ] You have access to real users for validation

**If prerequisites not met, STOP. Address gaps first.**

## The POC Canvas Flow

```
Product → Agent → Data → Model

Phase 1: Product Validation (Squares 1-2)
Phase 2: Agent Design (Squares 3-4)
Phase 3: Data Requirements (Squares 5-6)
Phase 4: External Model Integration (Squares 7-8)
```

## Instructions

### Step 1: Load the POC Canvas Template

Execute the create-doc task with the POC Canvas template:

```
Run task: create-doc.md with template: poc-canvas-tmpl.yaml
```

This will guide you through all 8 squares interactively.

### Step 2: Complete Phase 1 - Product Validation

Work with **Product Strategist** agent for Squares 1-2:

**Square 1: Product Vision & User Problem**

- Define specific workflow that frustrates users
- Identify who experiences pain and how often
- Articulate what success looks like
- Explain why users would prefer agent over alternatives

**Critical:** Demand evidence, not assumptions. Have you talked to real users?

**Square 2: User Validation & Interaction**

- Map complete interaction from start to finish
- Define interaction modality (chat, voice, GUI, etc.)
- Establish success metrics
- Identify adoption barriers and mitigation strategies

**Validation Gate:**

- [ ] At least 3 users validated the problem exists
- [ ] Users can articulate why agent is better than alternatives
- [ ] Success metrics are measurable
- [ ] Adoption barriers identified and addressable

**If validation fails:** STOP. Rethink the problem or solution.

### Step 3: Complete Phase 2 - Agent Design

Work with **Agent Architect** for Squares 3-4:

**Square 3: Agent Capabilities & Workflow**

- List specific actions agent must perform
- Break complex requests into agent-sized steps
- Define required tools and capabilities
- Establish decision vs. escalation boundaries

**Critical:** Remember 24% task completion rate. Scope appropriately.

**Square 4: Agent Interaction & Memory**

- Design conversation flow
- Define communication style
- Specify memory requirements (session vs. long-term)
- Plan error handling and graceful degradation

**Validation Gate:**

- [ ] All capabilities are within current AI limitations
- [ ] Complex tasks broken into manageable steps
- [ ] Decision boundaries clear (decide vs. escalate)
- [ ] Conversation style matches use case
- [ ] Memory requirements specified

**If validation fails:** Rescope capabilities to realistic AI limitations.

### Step 4: Complete Phase 3 - Data Requirements

Work with **Data Strategist** for Squares 5-6:

**Square 5: Knowledge Requirements & Sources**

- Map capabilities to required information
- Identify where knowledge currently exists
- Define update frequency needs
- Establish quality and accuracy requirements

**Critical:** Data requirements flow from agent capabilities, not vice versa.

**Square 6: Data Collection & Enhancement**

- Design initial data gathering methodology
- Identify highest-impact data
- Plan user feedback improvement loops
- Establish data freshness strategy

**Validation Gate:**

- [ ] All agent capabilities have mapped knowledge requirements
- [ ] Sources are identified and accessible
- [ ] Quality standards defined
- [ ] Update strategy established
- [ ] Feedback loops planned

**If validation fails:** Identify data gaps and acquisition plans.

### Step 5: Complete Phase 4 - Model Integration

Work with **Model Integration Specialist** for Squares 7-8:

**Square 7: Provider Selection & Prompt Engineering**

- Compare foundation models for your use case
- Design prompt engineering strategy
- Plan token optimization approach
- Calculate economics at scale

**Critical:** Model selection comes LAST, after Product → Agent → Data.

**Square 8: API Integration & Validation**

- Design API integration architecture
- Plan output processing and error handling
- Test against functional requirements
- Document hardening needs for production

**Validation Gate:**

- [ ] Model providers selected with justification
- [ ] Prompts tested and optimized for consistency
- [ ] Economics validated at 10x and 100x scale
- [ ] API integration functional
- [ ] Error handling implemented
- [ ] Performance meets requirements

**If validation fails:** Revisit model selection or prompt engineering.

## Final POC Canvas Validation

Before proceeding to build POC, confirm ALL squares complete:

**Phase 1: Product Validation**

- [ ] Square 1: Product Vision & User Problem ✅
- [ ] Square 2: User Validation & Interaction ✅

**Phase 2: Agent Design**

- [ ] Square 3: Agent Capabilities & Workflow ✅
- [ ] Square 4: Agent Interaction & Memory ✅

**Phase 3: Data Requirements**

- [ ] Square 5: Knowledge Requirements & Sources ✅
- [ ] Square 6: Data Collection & Enhancement ✅

**Phase 4: Model Integration**

- [ ] Square 7: Provider Selection & Prompt Engineering ✅
- [ ] Square 8: API Integration & Validation ✅

**Overall Validation:**

- [ ] Real users validated the problem
- [ ] Agent capabilities within AI limitations (24% task completion)
- [ ] Data sources identified and accessible
- [ ] Economics work at scale
- [ ] All 8 squares have detailed answers (not placeholders)

## Decision Gate

**If ALL boxes checked:**
✅ **Proceed to build POC**

- Build minimum viable POC in days (not months)
- Test with real users
- Gather feedback
- Only scale what users validate

**If ANY boxes unchecked:**
⚠️ **Complete missing squares before building**

- 95% of projects fail by skipping validation
- Don't be part of the 95%

**If validation shows fundamental flaws:**
🚨 **Consider killing project**

- Better to fail fast on bad ideas
- Redirect resources to validated opportunities

## Next Steps After POC Canvas

### 1. Build Rapid POC

- Use foundation models for quick prototyping
- Build MVP in days, not months
- Focus on core value proposition
- Don't over-engineer

### 2. Test with Real Users

- At least 10-20 users minimum
- Observe actual usage (not just feedback)
- Measure success metrics defined in Square 2
- Gather qualitative and quantitative feedback

### 3. Validate Business Value

- Calculate actual ROI: (Value - Cost) / Cost
- Measure outcomes (time saved, errors reduced, etc.)
- Confirm economics work at scale
- Use ROI Analyst to validate

### 4. Refine Based on Feedback

- Iterate on prompts and agent behavior
- Improve data quality based on failures
- Adjust scope based on what works
- Kill features users don't value

### 5. Only Then Consider Production

If POC is validated:

- **Switch to Production Canvas** (11 squares)
- Work with Production Architect
- Plan robust architecture, security, governance
- Only scale what users demonstrated they want

If POC is not validated:

- **Pivot or kill the project**
- Don't throw good money after bad
- 80% of companies see no material earnings - don't join them

## Critical Reminders

- **Start with problems, not technology** - Technology-first = 95% failure
- **Validate before scaling** - Only scale what users want
- **Current AI: 24% task completion** - Scope appropriately
- **Economics must work at scale** - Calculate at 10x and 100x
- **Focus on business value** - Outcomes over sophistication

## Resources

- Product Strategist agent - Product validation expert
- Agent Architect agent - Agent design specialist
- Data Strategist agent - Knowledge requirements expert
- Model Integration Specialist agent - Model selection and integration
- ROI Analyst agent - Business value measurement

## Success Pattern

**You will know you succeeded when:**

- Real users validated problem and solution
- Agent scope matches AI capabilities
- Economics are viable at scale
- Clear path to business value
- Ready to build with confidence

**Join the successful 25% by completing POC Canvas systematically.**
