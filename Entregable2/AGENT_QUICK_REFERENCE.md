# Entregable2 Reviewer Agent - Quick Reference

## 🎯 What It Does
Reviews R outputs from Entregable 2 radar classifier assignment and provides expert feedback on all requirements.

## 📍 Agent Location
`.github/agents/entregable2_reviewer.md`

## 🚀 Quick Start

### 1. Prepare Your R Outputs
Run your analysis and collect these outputs:
- ✅ Label remapping tables (before/after)
- ✅ Train/validation split verification
- ✅ RCS statistics (median±MAD, statistical tests)
- ✅ AP/mAP per class (train & validation)
- ✅ AP vs distance and azimuth analysis
- ✅ Person vs rest threshold results

### 2. Format Your Request
```
Please review my Entregable 2 R outputs using the entregable2_reviewer agent.

=== LABEL REMAPPING ===
[paste your frequency tables]

=== TRAIN/VALIDATION SPLIT ===
[paste dimensions and distributions]

=== EDA: RCS ANALYSIS ===
[paste statistics and test results]

=== MODEL METRICS: AP/mAP ===
[paste AP table and mAP values]

=== SPATIAL ANALYSIS ===
[paste distance/azimuth results]

=== PERSON VS REST THRESHOLD ===
[paste threshold selection and validation metrics]
```

### 3. Get Comprehensive Review
Receive structured feedback in 5 sections:
- **A. Executive Findings** - Key takeaways
- **B. Detailed Review** - 7 in-depth sections
- **C. Direct Answers** - Assignment questions answered
- **D. Recommendations** - Actionable improvements
- **E. Missing Items** - What's not provided

## 📚 Full Documentation
- **Usage Guide**: `Entregable2/AGENT_USAGE_GUIDE.md`
- **Example Review**: `Entregable2/EXAMPLE_AGENT_REVIEW.md`
- **Agent Instructions**: `.github/agents/entregable2_reviewer.md`

## ✅ What Gets Reviewed

### Data Preprocessing (Section B1)
- ✓ Class relabeling verification (bus/truck→large vehicle, group→person)
- ✓ Sequential split correctness (200k/200k)
- ✓ Class distribution balance

### EDA Analysis (Section B2)
- ✓ RCS median±MAD comparison (Person vs Large Vehicle)
- ✓ Statistical significance tests
- ✓ Effect size interpretation

### Modeling (Section B3)
- ✓ Model approach identification
- ✓ Feature interactions assessment
- ✓ Physical plausibility

### Performance (Section B4)
- ✓ AP per class extraction
- ✓ mAP calculation
- ✓ Overfitting analysis

### Spatial Effects (Section B5)
- ✓ AP vs distance trends
- ✓ AP vs azimuth trends
- ✓ Physics-based validation

### Threshold Selection (Section B6)
- ✓ Precision prioritization check
- ✓ Train vs validation comparison
- ✓ Generalization assessment

### Quality (Section B7)
- ✓ Consistency checks
- ✓ Reproducibility assessment
- ✓ Improvement suggestions

## 🎓 Assignment Coverage
Covers all 9.5 points of Entregable 2:
- **Q1** (0.5 pts): Load data + relabel classes
- **Q2** (0.5 pts): Sequential split
- **Q3** (1.5 pts): RCS analysis + statistical test
- **Q4** (3 pts): Multiclass model + AP/mAP
- **Q5** (2 pts): Spatial analysis (distance/azimuth)
- **Q6** (2 pts): Person vs rest threshold + evaluation

## 💡 Pro Tips
1. **Be complete** - Include all console outputs, not just tables
2. **Label clearly** - Use section headers to organize outputs
3. **Show tests** - Include full statistical test outputs
4. **Preserve format** - Copy tables as they appear
5. **Be honest** - Flag anything you skipped or approximated

## ⚠️ What Agent Does NOT Do
- ❌ Write or modify code
- ❌ Re-run experiments
- ❌ Generate new analyses
- ❌ Access your R environment
- ✅ **Reviews completed outputs only**

## 📧 Need Help?
- Read the full usage guide: `AGENT_USAGE_GUIDE.md`
- See example review: `EXAMPLE_AGENT_REVIEW.md`
- Check agent instructions: `.github/agents/entregable2_reviewer.md`

---

**Last Updated**: 2025-11-01  
**Version**: 1.0  
**Reviewer Expertise**: Senior Data Scientist (R, Statistics, ML, Radar Systems)
