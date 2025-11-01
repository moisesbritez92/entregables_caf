# Custom Agents for Entregables CAF

This directory contains custom agent instructions for specialized review tasks.

## Available Agents

### 1. Entregable2 Reviewer (`entregable2_reviewer.md`)

**Purpose**: Review R outputs from Entregable 2 (Multiclass Object Classifier Based on 2D Radar Data)

**Expertise**:
- Senior data scientist specialized in R, statistical inference, and classification metrics
- Domain knowledge in radar-based perception systems
- Expert in AP/mAP metrics, PR curves, and model validation

**Use Case**: 
When you need to review completed R analysis outputs for the Entregable 2 assignment without re-running experiments.

**How to Use**:
1. Run your R analysis and capture all console outputs
2. Collect outputs including:
   - Label remapping tables
   - Train/validation split verification
   - EDA results (RCS analysis, statistical tests)
   - Model metrics (AP per class, mAP)
   - Spatial analysis (AP vs distance/azimuth)
   - PR curve and threshold selection results
3. Paste the R outputs and request a review using this agent

**Output**: Structured review with:
- Executive findings
- Detailed analysis of 7 key areas
- Direct answers to assignment questions
- Actionable recommendations
- List of missing/ambiguous items

**Example Invocation**:
```
Please review these R outputs from my Entregable 2 analysis using the entregable2_reviewer agent:

[paste R console outputs here]
```

## Agent Design Principles

These custom agents follow these principles:
1. **Specialized expertise** - Deep domain knowledge in specific areas
2. **Output-only review** - Review finished work without re-running experiments
3. **Structured format** - Consistent, comprehensive review structure
4. **Actionable feedback** - Concrete suggestions for improvement
5. **Context-aware** - Understand assignment requirements and evaluation criteria

## Adding New Agents

To add a new custom agent:

1. Create a new `.md` file in this directory
2. Include these sections:
   - Role & Expertise
   - Context and domain information
   - Specific tasks/responsibilities
   - Output format specification
   - Style guidelines
   - Evaluation criteria
   - Example outputs

3. Update this README with agent description

4. Test the agent with sample inputs to verify behavior

## Notes

- Agents are instruction sets for specialized review roles
- They do not have access to repository code directly
- They rely on information provided by the user
- They should be precise, concise, and actionable in their feedback
