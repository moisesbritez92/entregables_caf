# How to Use the Entregable2 Reviewer Agent

## Overview

The **Entregable2 Reviewer Agent** is a specialized custom agent designed to review R outputs from the multiclass radar object classifier assignment. It provides expert-level analysis without requiring you to re-run experiments.

## What is a Custom Agent?

A custom agent is a specialized role with domain expertise that can be invoked to perform specific tasks. The Entregable2 Reviewer acts as a senior data scientist who:
- Reviews your R console outputs
- Verifies assignment requirements are met
- Provides structured, comprehensive feedback
- Identifies issues and suggests improvements
- Answers assignment questions based on your results

## When to Use This Agent

Use this agent when:
- ✅ You've completed your R analysis for Entregable 2
- ✅ You want to verify you've met all assignment requirements
- ✅ You need expert feedback on your methodology
- ✅ You want to check if your results are reasonable
- ✅ You're preparing your final submission

**Do NOT use** when:
- ❌ You haven't started the analysis yet (use it for review, not initial guidance)
- ❌ You want the agent to write code for you (it only reviews outputs)
- ❌ You need help debugging R errors (it reviews completed outputs)

## How to Prepare Your R Outputs

### Step 1: Run Your Analysis

Execute your `entregable2_corrected.Rmd` or equivalent R Markdown file and capture all outputs:

```r
# Option A: Knit to HTML and review console output
library(rmarkdown)
render("entregable2_corrected.Rmd", output_format = "html_document")

# Option B: Run chunks interactively and save output
# Make sure to capture all console messages, tables, and summaries
```

### Step 2: Collect Required Outputs

Gather the following from your console/output:

1. **Label Remapping**:
   - Before/after frequency tables showing class reassignments
   - Verification that bus(3) and truck(2) → large vehicle(1)
   - Verification that group(8) → person(7)

2. **Train/Validation Split**:
   - Dimensions of train and validation sets
   - Class distribution in each set
   - Confirmation of sequential split (first 200k, next 200k)

3. **EDA (RCS Analysis)**:
   - RCS median ± MAD statistics by distance bins
   - Statistical test results (t-test, Mann-Whitney, Cohen's d)
   - Interpretation of significance

4. **Model Metrics**:
   - AP per class (one-vs-rest) for train and validation
   - mAP (mean Average Precision) for train and validation
   - Table showing all values

5. **Spatial Analysis**:
   - AP values by distance bins (e.g., 0-30m, 30-60m, etc.)
   - AP values by azimuth bins (e.g., 0-0.2, 0.2-0.4, etc.)
   - Tables or summaries showing trends

6. **Person vs Rest Analysis**:
   - PR curve data or summary
   - Selected threshold value
   - Precision and Recall at that threshold (train)
   - Precision and Recall at same threshold (validation)
   - Comparison table

### Step 3: Format Your Request

Create a message like this:

```
Please review these R outputs from my Entregable 2 analysis using the entregable2_reviewer agent.

=== LABEL REMAPPING ===
[paste frequency tables showing before/after class counts]

=== TRAIN/VALIDATION SPLIT ===
[paste dimension verification and class distributions]

=== EDA: RCS ANALYSIS ===
[paste RCS statistics, test results, Cohen's d]

=== MODEL METRICS: AP/mAP ===
[paste AP per class table and mAP values]

=== SPATIAL ANALYSIS ===
[paste AP by distance and azimuth tables]

=== PERSON VS REST THRESHOLD ===
[paste threshold selection info and validation results]

=== ADDITIONAL OUTPUTS ===
[any other relevant console outputs]
```

## Example: Minimal Required Output

Here's an example of the minimum information needed:

```
=== LABEL REMAPPING ===
Distribution of classes ORIGINAL:
   0    1    2    3    5    7    8   10   11 
1234 5678 2341 1456 3456 6789 1234 4567 8901

Distribution of classes AFTER reassignment:
   0    1    5    7   10   11 
1234 9475 3456 8023 4567 8901

=== TRAIN/VALIDATION SPLIT ===
Dimensions Train: 200000 9
Dimensions Validation: 200000 9

Distribution in TRAIN:
   0     1     5     7    10    11 
 620  4750  1730  4012  2267  1145

=== MODEL METRICS ===
AP Results:
  Clase  AP_Train  AP_Validation  Diferencia
      0    0.8234        0.8156      0.0078
      1    0.7891        0.7723      0.0168
      5    0.6543        0.6398      0.0145
      7    0.6145        0.5987      0.0158
     10    0.7234        0.7089      0.0145
     11    0.8901        0.8823      0.0078

mAP Training:   0.7491
mAP Validation: 0.7363
Diferencia:     0.0128

[Continue with other sections...]
```

## What You'll Get Back

The agent will provide:

### A. Executive Findings (5-10 bullets)
Quick summary of key findings, strengths, and critical issues.

### B. Detailed Review (7 sections)
1. Preprocessing & Labeling verification
2. EDA (RCS vs Distance) analysis
3. Modeling approach assessment
4. Performance metrics (AP/mAP) evaluation
5. AP vs distance/azimuth analysis
6. PR threshold selection review
7. Quality & reproducibility assessment

### C. Direct Answers to Assignment Questions
Numbered answers to questions 2-7 from the assignment, citing your results.

### D. Actionable Recommendations
Concrete suggestions to improve reporting and analysis.

### E. Missing or Ambiguous Items
Checklist of information not provided that would strengthen the review.

## Tips for Best Results

1. **Be comprehensive**: Include all relevant outputs, not just tables
2. **Include context**: Label each section clearly
3. **Preserve formatting**: Copy tables and outputs as they appear
4. **Include statistical tests**: Don't just show p-values, include test names and interpretations
5. **Show intermediate steps**: Include any calculations or logic you used
6. **Be honest**: If you skipped something, mention it - the agent will help identify gaps

## Common Issues

### Issue: "Cannot determine if requirements are met"
**Solution**: You likely didn't provide enough output. Go back and include:
- Full frequency tables (before/after relabeling)
- Complete metrics tables with column headers
- Statistical test outputs with test names

### Issue: "Inconsistent results reported"
**Solution**: Check if you:
- Applied relabeling consistently throughout
- Used the correct train/validation split (sequential, not random)
- Calculated AP correctly (one-vs-rest, not multi-class at once)

### Issue: "Missing critical analysis"
**Solution**: You might have skipped a required section:
- Did you analyze AP vs distance bins?
- Did you analyze AP vs azimuth bins?
- Did you select a specific threshold for Person vs rest?
- Did you evaluate that threshold on validation?

## After the Review

Based on the agent's feedback:

1. **Address Critical Issues First**: Fix any missing requirements or incorrect implementations
2. **Improve Reporting**: Add missing tables, clarify labels, include confidence intervals
3. **Refine Interpretations**: Update your conclusions based on agent's physical intuition
4. **Document Limitations**: Add notes about assumptions and constraints
5. **Re-review if Needed**: If you make significant changes, request another review

## Questions?

If the agent's feedback is unclear or you need clarification:
- Ask specific follow-up questions
- Provide additional context or outputs
- Request explanation of specific recommendations

The agent is designed to help you produce high-quality, assignment-compliant analysis!
