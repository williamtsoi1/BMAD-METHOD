# CANVAS Framework Knowledge Base

## Framework Origin

The CANVAS framework was developed by MongoDB to address the 95% failure rate of AI agent projects. Published in September 2024 by Mikiko Bazeley, it provides systematic canvases for validating and scaling AI agents.

## Core Philosophy

### The Paradigm Shift

**Traditional AI Development (Pre-Foundation Models):**

```
Data → Model → Product
- 10+ months to first value
- Massive upfront investment
- Brittle, single-purpose models
```

**Modern Foundation Model Approach:**

```
Product → Data → Model
- Days to first value
- Easy model swapping
- Rapid experimentation
```

**CANVAS Framework (Agentic Evolution):**

```
Product → Agent → Data → Model
- Agent design determines downstream requirements
- Focus on orchestration before model selection
- Systematic validation before scaling
```

## The Two Canvas System

### POC Canvas (8 Squares)

Validates agent concepts rapidly through 4 phases:

**Phase 1: Product Validation (Squares 1-2)**

- Square 1: Product Vision & User Problem
- Square 2: User Validation & Interaction

**Phase 2: Agent Design (Squares 3-4)**

- Square 3: Agent Capabilities & Workflow
- Square 4: Agent Interaction & Memory

**Phase 3: Data Requirements (Squares 5-6)**

- Square 5: Knowledge Requirements & Sources
- Square 6: Data Collection & Enhancement Strategy

**Phase 4: External Model Integration (Squares 7-8)**

- Square 7: Provider Selection & Prompt Engineering
- Square 8: API Integration & Validation

### Production Canvas (11 Squares)

Scales validated POCs to enterprise deployment through 5 phases:

**Phase 1: Product and Scale Planning (Squares 1-2)**

- Square 1: Business Case & Scale Planning
- Square 2: Production Requirements & Constraints

**Phase 2: Agent Architecture (Squares 3-4)**

- Square 3: Robust Agent Architecture
- Square 4: Production Memory & Context Systems

**Phase 3: Data Infrastructure (Squares 5-6)**

- Square 5: Data Architecture & Management
- Square 6: Knowledge Base & Pipeline Operations

**Phase 4: Model Operations (Squares 7-8)**

- Square 7: Model Strategy & Optimization
- Square 8: API Management & Monitoring

**Phase 5: Hardening and Operations (Squares 9-11)**

- Square 9: Security & Compliance
- Square 10: User Experience & Adoption
- Square 11: Continuous Improvement & Governance

## Critical Statistics

### Failure Rates

- **95%** of AI agent POCs fail (technology-first approach)
- **80%** of companies report no material earnings impact from gen AI
- **56%** of projects stopped due to security/governance barriers

### Success Factors

- **25%** of companies taking "AI-first" (problem-first) approach achieve transformative results
- **30%** have CEO sponsorship (critical for success)
- **44%** have governance policies (vs. 92% who say it's essential)

### Current AI Capabilities

- **24%** office task completion rate (Claude 3.5 Sonnet - current best)
- Agents struggle with basic obstacles: pop-ups, authentication, resource finding
- Some agents resort to deception when faced with obstacles

## Core Principles

### 1. Start with Problems, Not Technology

- Define user workflow problems first
- Validate pain points with real users
- Only then design agent solution
- Technology selection comes last

### 2. Validate Before Scaling

- Complete POC Canvas before building
- Test with real users before production planning
- Only scale what users actually want
- Don't build based on assumptions

### 3. Follow Product → Agent → Data → Model Flow

- Product vision determines agent requirements
- Agent design determines data needs
- Data requirements drive model selection
- Model choice is the final decision, not first

### 4. Understand Current AI Limitations

- Current best: 24% task completion for office work
- Design for augmentation, not full automation
- Scope appropriately to AI capabilities
- Plan for human escalation

### 5. Secure Leadership Buy-In

- CEO or C-level sponsorship critical
- Part of official company strategy
- Top-down vision and support
- Example: Moderna's 750 agents enabled by CEO buy-in

### 6. Address Governance Early

- 92% say essential, only 44% have policies
- Security cannot be an afterthought
- Define boundaries and audit trails
- 80% experienced agents acting outside boundaries

### 7. Use Unified Data Architecture

- Agents need: application data, vector store, memory
- Managing three databases kills momentum
- Single platform (like MongoDB Atlas) reduces complexity
- Unified approach enables faster iteration

### 8. Focus on Clear ROI

- Measure outcomes (business value), not activity (agent count)
- Define success metrics upfront
- Calculate economic viability at scale
- Unit economics must work at 10x and 100x usage

### 9. Rapid Iteration Over Perfection

- Build MVPs in days with foundation models
- Gather user feedback quickly
- Improve based on real usage
- Fine-tune only when ROI clear

### 10. Design for Production from Day One

- Security, compliance, scalability from start
- Avoid "pilot purgatory"
- Production requires fault tolerance and monitoring
- Governance and audit trails essential

## Memory Requirements for Agents

66% of executives want systems that learn from feedback. 63% demand context retention.

### Three Memory Types

1. **Session Memory**
   - Current conversation context
   - Recent interaction history
   - Temporary working memory

2. **Long-Term Memory**
   - User preferences and history
   - Past interactions and outcomes
   - Personalization data

3. **Organizational Knowledge**
   - Company-specific information
   - Processes and procedures
   - Institutional knowledge

## Model Selection Guidance

### External API Orchestration Focus

- Use external model providers (OpenAI, Anthropic, Google)
- Focus on integration, not deployment
- Enable easy provider swapping
- Implement failover strategies

### Current Leading Models

- **Claude 3.5 Sonnet** - Best task completion (24%)
- **GPT-4 family** - Strong general capabilities
- **Gemini family** - Long context windows
- **Open source** - Llama, Mistral for specific needs

### Specialized Models

- **Embeddings**: Domain-specific (Voyage AI for technical) vs. general
- **Code**: Code-specialized models for development tasks
- **Multilingual**: Language-specific optimization

### Selection Criteria

1. Task performance for your use case
2. Token limits (context requirements)
3. Latency (user experience needs)
4. Cost (economic viability at scale)
5. Reliability (API uptime)
6. Features (function calling, streaming, vision)

## Success Pattern: Moderna

**Results:** 750+ AI agents deployed

**Success Factors:**

- CEO sponsorship from inception
- Systematic canvas approach to agent development
- Production-grade infrastructure investment
- Focus on business process transformation
- Radical restructuring of HR and IT around AI

**Key Insight:** They measured business transformation, not agent count.

## Validation Gates

### POC → Production Transition

Only proceed to Production Canvas when:

- ✅ Real users validated POC works
- ✅ Clear business value demonstrated
- ✅ ROI can be calculated
- ✅ Users prefer agent over current solutions
- ✅ Economic viability confirmed at scale

### Production Deployment

Only deploy when:

- ✅ All 11 Production Canvas squares complete
- ✅ Security and compliance frameworks implemented
- ✅ Monitoring and alerting operational
- ✅ Governance policies documented
- ✅ User adoption plan established
- ✅ Economic model validated

## When to Kill Projects

Stop development if:

- Users don't validate problem exists
- POC doesn't demonstrate clear value
- Unit economics don't work at scale
- Security/compliance requirements unaddressable
- Leadership sponsorship lacking
- Better solutions exist for less cost

**It's better to kill a project than deploy agents nobody wants.**

## Resources

- MongoDB CANVAS Framework blog post (September 2024)
- POC Canvas template (PDF)
- Production Canvas template (PDF)
- MongoDB AI Learning Hub
- Webinars on memory-augmented agents
