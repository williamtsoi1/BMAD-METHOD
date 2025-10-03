# ADK Implementation Checklist

Use this checklist to ensure your Google ADK agent implementation meets production standards.

## Environment Setup

- [ ] Google ADK installed (`pip install google-adk`)
- [ ] Gemini API access configured
- [ ] Project structure follows ADK conventions
- [ ] Dependencies listed in requirements.txt

## Agent Design

- [ ] Agent purpose clearly defined
- [ ] Appropriate agent type selected (LlmAgent vs BaseAgent vs Workflow)
- [ ] Instruction prompt is clear and specific
- [ ] Model selection appropriate for task (e.g., gemini-2.0-flash)
- [ ] Agent description provided

## Tools (if applicable)

- [ ] Tool functions have clear, descriptive names
- [ ] All parameters have type hints
- [ ] Comprehensive docstrings provided (LLM uses these)
- [ ] Tools follow single responsibility principle
- [ ] Tools return appropriate types
- [ ] Error handling implemented in tools
- [ ] Tools tested independently

## Schemas (if applicable)

- [ ] Input schema defined using Pydantic BaseModel (if structured input needed)
- [ ] Output schema defined using Pydantic BaseModel (if structured output needed)
- [ ] Field descriptions clear and complete
- [ ] Schema validation working correctly
- [ ] NOT combining output_schema with tools (mutually exclusive)

## Multi-Agent Systems (if applicable)

- [ ] Sub-agents clearly defined with distinct purposes
- [ ] Sub-agents list provided to parent agent
- [ ] Data flow designed using session.state
- [ ] output_key used for agent-to-agent communication
- [ ] Orchestration pattern appropriate (Sequential/Loop/Parallel/Custom)

## Custom Agents (if applicable)

- [ ] Pydantic field declarations for all sub-agents
- [ ] `model_config = {"arbitrary_types_allowed": True}` set
- [ ] `_run_async_impl` properly implemented as async generator
- [ ] All events from sub-agents properly yielded
- [ ] Session state properly managed
- [ ] Sub-agents list passed to super().**init**
- [ ] Error handling in orchestration logic

## State Management

- [ ] output_key used for storing agent results
- [ ] Session state keys documented
- [ ] Data flow between agents clearly defined
- [ ] State keys consistent across agent instructions

## Testing

- [ ] Agent tested with sample inputs
- [ ] Edge cases identified and tested
- [ ] Tool functions unit tested
- [ ] Multi-agent data flow verified
- [ ] Evaluation set created (.evalset.json)
- [ ] All tests passing

## Code Quality

- [ ] Type hints on all functions
- [ ] Docstrings on all classes and functions
- [ ] Code follows Python best practices
- [ ] No hardcoded credentials or secrets
- [ ] Proper error handling
- [ ] Logging implemented where appropriate

## Documentation

- [ ] Agent purpose documented
- [ ] Usage examples provided
- [ ] Tool descriptions clear
- [ ] Session state keys documented
- [ ] Deployment instructions included

## Deployment Readiness

- [ ] Configuration externalized (no hardcoded values)
- [ ] Environment variables for sensitive data
- [ ] Containerization prepared (if deploying to Cloud Run)
- [ ] Resource requirements identified
- [ ] Monitoring strategy defined

## Story Compliance

- [ ] All story task requirements met
- [ ] Acceptance criteria satisfied
- [ ] File List updated in story
- [ ] Change Log updated
- [ ] Story status set to "Ready for Review"

## Final Verification

- [ ] Code reviewed for security issues
- [ ] Performance acceptable for use case
- [ ] Agent behaves as intended
- [ ] No unnecessary complexity
- [ ] Follows ADK best practices
- [ ] Ready for production deployment
