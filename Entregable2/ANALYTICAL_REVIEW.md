# Analytical Review: Entregable 2 - Multiclass Object Classifier

**Reviewer Role**: Data Science and Machine Learning Expert specializing in applied classification problems, statistical validation, and model performance analysis

**Document Reviewed**: Entregable 2: Clasificador Multiclase de Objetos basado en Datos Radar 2D

**Review Date**: 2025-11-01

---

## Executive Summary

This deliverable presents a multiclass classification system for identifying different object types (cars, large vehicles, pedestrians, etc.) using radar-based features. While the work demonstrates solid machine learning fundamentals and good software engineering practices with optimizations, **there are critical misalignments between the assignment requirements and the actual implementation**.

**Overall Assessment**: ⭐⭐⭐☆☆ (3/5)

**Key Strengths**:
- ✅ Well-structured code with clear documentation
- ✅ Excellent performance optimizations (parallel processing, efficient data handling)
- ✅ Proper handling of class imbalance through weighted approaches
- ✅ Comprehensive model comparison (Random Forest, XGBoost, baseline)
- ✅ Good error analysis and visualization
- ✅ Consideration of missing classes (4, 6, 9) in the data

**Critical Issues**:
- ❌ **Class reassignments not performed**: Bus (3), Truck (2) → Large Vehicle (1); Group of People (8) → Person (7) were not applied as required
- ❌ **Incorrect data split**: Used random 70-30 stratified split instead of sequential first 200k (train) / next 200k (validation)
- ❌ **Missing RCS analysis**: No visualization of RCS median ± MAD for Person (7) vs Large Vehicle (1) by distance
- ❌ **Wrong metrics**: No computation of Average Precision (AP) per class or mean Average Precision (mAP) as specified
- ❌ **Missing spatial analysis**: No analysis of AP improvement with closer distances and azimuth angles near 0
- ❌ **Missing Person vs rest analysis**: No specific binary classification threshold selection for Person vs rest with precision prioritization

---

## 1. Data Preprocessing and Labeling

### 1.1 Class Reassignments

**Requirement**: Reassign labels as follows:
- Bus (3) and Truck (2) → Large Vehicle (1)
- Group of People (8) → Person (7)

**Implementation**: ❌ **NOT IMPLEMENTED**

**Assessment**: ⚠️ CRITICAL ISSUE

**Findings**:
- The code works directly with the original class labels (0, 1, 2, 3, 5, 7, 8, 10, 11)
- No reassignment logic is present in the preprocessing section
- This fundamentally changes the classification task and makes results non-comparable to requirements

**Impact**:
- Class imbalance analysis is performed on wrong class distribution
- Model is solving a different problem than specified
- Results cannot be interpreted in the context of the assignment requirements

**Required Fix**:
```r
# After loading data, reassign classes
y[y == 2] <- 1  # Truck → Large Vehicle
y[y == 3] <- 1  # Bus → Large Vehicle
y[y == 8] <- 7  # Group of People → Person

# Update factor levels accordingly
y_factor <- factor(y, levels = c(0, 1, 5, 7, 10, 11))
```

### 1.2 Independence Assumption of Detections

**Assumption**: "Asumir cada detección (registro) como un objeto independiente"

**Implementation**: ✅ CORRECTLY APPLIED

**Assessment**: ✅ ACCEPTABLE

**Findings**:
- Each row in the dataset is treated as an independent observation
- This is a reasonable simplification for initial model development
- No temporal or spatial grouping is attempted

**Limitations to Note**:
- In reality, multiple detections may correspond to the same physical object across time
- This assumption may introduce:
  - **Label leakage**: Same object appearing in train and test sets
  - **Autocorrelation**: Consecutive detections of the same object are not independent
  - **Optimistic performance estimates**: Model may perform worse on truly unseen objects
  
**Recommendation**: 
- For production systems, implement object tracking to group detections
- Consider temporal holdout validation (e.g., split by time periods rather than random sampling)
- Document this limitation clearly in final reports

### 1.3 Train/Validation Split

**Requirement**: 
- Train: First 200,000 records
- Validation: Next 200,000 records

**Implementation**: ❌ **INCORRECT**

**Assessment**: ⚠️ CRITICAL ISSUE

**Findings**:
```r
# Current implementation (WRONG):
train_index <- createDataPartition(y_factor, p = 0.7, list = FALSE)
X_train <- X[train_index, ]
X_test <- X[-train_index, ]
```

This uses:
- Random stratified 70-30 split
- Ensures balanced class distribution across splits
- Standard in machine learning but NOT what was requested

**Why This Matters**:
1. **Temporal considerations**: If data has temporal ordering, sequential split respects time
2. **Real-world simulation**: Sequential split better simulates model deployment on future data
3. **Assignment compliance**: Does not follow explicit instructions

**Statistical Soundness Assessment**:
- ✅ Random stratified split is statistically valid and generally preferred
- ✅ Maintains class proportions in both sets
- ❌ However, ignores potential temporal structure in data
- ❌ Does not match assignment requirements

**Required Fix**:
```r
# Sequential split as required
n_train <- 200000
n_val <- 200000

X_train <- X[1:n_train, ]
X_validation <- X[(n_train + 1):(n_train + n_val), ]
y_train <- y[1:n_train]
y_validation <- y[(n_train + 1):(n_train + n_val)]

# Check class distribution
cat("Train distribution:\n")
print(table(y_train))
cat("\nValidation distribution:\n")
print(table(y_validation))
```

**Implication**: If data has temporal ordering (e.g., collected sequentially over time), this split tests model generalization to future time periods, which is more realistic for production deployment.

---

## 2. Exploratory Data Analysis and Feature Inspection

### 2.1 RCS vs Distance Visualization for Person (7) and Large Vehicle (1)

**Requirement**: "Graficar el valor de Radar Cross Section (RCS) mediana ± mad para las clases Persona (7) y Vehículo grande (1) en función de la distancia (range) en el set de 'Train'."

**Implementation**: ❌ **NOT IMPLEMENTED**

**Assessment**: ⚠️ MAJOR OMISSION

**Findings**:
- Current visualizations include:
  - ✅ Boxplots of all features
  - ✅ Density plots by class for all features
  - ✅ Correlation matrix
  - ❌ NO specific RCS vs distance analysis for Person (7) vs Large Vehicle (1)

**Why This Analysis Is Important**:
1. **Physical interpretation**: RCS (Radar Cross Section) measures the radar reflectivity of objects
   - Larger objects (vehicles) typically have higher RCS
   - Smaller objects (persons) have lower RCS
   - RCS varies with distance due to signal propagation

2. **Feature discriminability**: Understanding if RCS can separate these classes is crucial for model feature importance

3. **Distance dependency**: Analyzing RCS vs distance reveals:
   - If signal degradation affects class discrimination
   - If closer detections are more reliable
   - Optimal operating range for the classifier

**Expected Analysis**:

```r
# Prepare data for Person (7) and Large Vehicle (1) only
df_analysis <- data.frame(
  range_sc = X_train[, "range_sc"],
  rcs = X_train[, "radar_cross_section"],
  clase = y_train
)
df_analysis <- df_analysis[df_analysis$clase %in% c(1, 7), ]

# Create distance bins
df_analysis$range_bin <- cut(df_analysis$range_sc, 
                              breaks = seq(0, max(df_analysis$range_sc), by = 10),
                              include.lowest = TRUE)

# Calculate median ± MAD by distance bin
library(dplyr)
rcs_summary <- df_analysis %>%
  group_by(range_bin, clase) %>%
  summarise(
    median_rcs = median(rcs),
    mad_rcs = mad(rcs),
    n = n()
  ) %>%
  filter(n >= 30)  # Require minimum sample size

# Plot with error bars
ggplot(rcs_summary, aes(x = range_bin, y = median_rcs, color = factor(clase))) +
  geom_point() +
  geom_line(aes(group = clase)) +
  geom_errorbar(aes(ymin = median_rcs - mad_rcs, 
                    ymax = median_rcs + mad_rcs), 
                width = 0.2) +
  labs(title = "RCS (median ± MAD) vs Distance",
       subtitle = "Person (7) vs Large Vehicle (1)",
       x = "Distance Range (m)",
       y = "RCS (dBsm)",
       color = "Class") +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))
```

### 2.2 Statistical Evidence of RCS Difference

**Requirement**: "¿Existe diferencia significativa en términos de RCS entre la clase Persona (7) y Vehículo grande (1)? Justifica tu respuesta."

**Implementation**: ❌ **NOT ADDRESSED**

**Assessment**: ⚠️ MAJOR OMISSION

**Expected Analysis**:

To properly answer this question, the following statistical tests should be performed:

1. **Descriptive Statistics**:
```r
# Summary statistics by class
df_analysis %>%
  group_by(clase) %>%
  summarise(
    mean_rcs = mean(rcs),
    median_rcs = median(rcs),
    sd_rcs = sd(rcs),
    mad_rcs = mad(rcs),
    q25 = quantile(rcs, 0.25),
    q75 = quantile(rcs, 0.75)
  )
```

2. **Visual Comparison**:
```r
# Boxplot
ggplot(df_analysis, aes(x = factor(clase), y = rcs, fill = factor(clase))) +
  geom_boxplot() +
  labs(title = "RCS Distribution: Person vs Large Vehicle")

# Density plot
ggplot(df_analysis, aes(x = rcs, fill = factor(clase))) +
  geom_density(alpha = 0.5) +
  labs(title = "RCS Density: Person vs Large Vehicle")
```

3. **Statistical Tests**:
```r
# Welch's t-test (accounts for unequal variances)
t_test_result <- t.test(
  rcs ~ clase, 
  data = df_analysis, 
  var.equal = FALSE
)
print(t_test_result)

# Mann-Whitney U test (non-parametric alternative)
wilcox_result <- wilcox.test(
  rcs ~ clase, 
  data = df_analysis
)
print(wilcox_result)

# Cohen's d for effect size
library(effsize)
cohens_d <- cohen.d(
  df_analysis$rcs[df_analysis$clase == 7],
  df_analysis$rcs[df_analysis$clase == 1]
)
print(cohens_d)
```

4. **Interpretation Framework**:
   - **p-value < 0.05**: Statistically significant difference exists
   - **Cohen's d interpretation**:
     - Small effect: |d| = 0.2
     - Medium effect: |d| = 0.5
     - Large effect: |d| = 0.8
   - **Practical significance**: Even if statistically significant, is the difference large enough to be useful for classification?

**Expected Answer Structure**:
> "Analysis of RCS values between Person (7) and Large Vehicle (1) shows:
> - Mean RCS for Large Vehicle: [X] dBsm (± [SD])
> - Mean RCS for Person: [Y] dBsm (± [SD])
> - Statistical test: Welch's t-test yielded t = [value], p < [value]
> - Effect size: Cohen's d = [value] ([small/medium/large] effect)
> - **Conclusion**: [Yes/No], there is [statistically significant/no significant] difference in RCS between the two classes. The effect size is [small/medium/large], indicating [practical interpretation]."

### 2.3 Variability Interpretation (Median ± MAD)

**Requirement**: Assess whether variability was correctly interpreted

**Implementation**: ❌ NOT PERFORMED

**Assessment**: N/A - Analysis not present

**Expected Interpretation**:

MAD (Median Absolute Deviation) is a robust measure of statistical dispersion:

```r
MAD = median(|X_i - median(X)|)
```

**Interpretation Guidelines**:
1. **Large MAD**: High variability in RCS values
   - Could indicate measurement noise
   - Could reflect true diversity within class (e.g., different person sizes, clothing)
   - Makes class separation more challenging

2. **Small MAD**: Low variability in RCS values
   - More consistent measurements
   - Tighter class distribution
   - Easier class separation

3. **Overlapping intervals**: If (median ± MAD) ranges overlap between classes
   - Indicates ambiguity in classification
   - Some observations from both classes have similar RCS values
   - Perfect separation is not possible using RCS alone

4. **Distance effects**: If MAD increases with distance
   - Signal degradation at longer ranges
   - More measurement uncertainty
   - Suggests range-dependent feature engineering might help

---

## 3. Model Development

### 3.1 Modeling Approach

**Implementation Review**: ✅ GOOD APPROACH (but wrong metrics)

**Assessment**: ⭐⭐⭐⭐☆ (4/5)

**Findings**:

**Models Implemented**:
1. ✅ **Baseline Model**: Majority class prediction (good practice)
2. ✅ **Random Forest**: 200 trees, mtry=3, with class weights
3. ✅ **XGBoost**: 100 rounds, early stopping, with sample weights
4. ⚠️ **SVM**: Noted but set to eval=FALSE (reasonable due to computational cost)

**Strengths**:
- Multiple model comparison is good practice
- Baseline model provides reference point
- Both tree-based methods are appropriate for this task

**Weaknesses**:
1. **No Logistic Regression**: Assignment recommends adding variable interactions
   - Logistic regression with interactions would be interpretable
   - Could reveal which feature combinations are most predictive
   
2. **Limited Hyperparameter Tuning**: 
   - No systematic grid search or random search
   - Parameters seem reasonable but not optimized
   
3. **No Model Ensembling**: Could combine RF and XGBoost predictions

**Recommendation**:
```r
# Add multinomial logistic regression with interactions
library(nnet)

# Create interaction terms
X_train_interact <- model.matrix(~ . + range_sc:radar_cross_section + 
                                     azimuth_sc:range_sc + 
                                     x_cc:y_cc, 
                                  data = as.data.frame(X_train_scaled))

# Train multinomial logistic regression
multinom_model <- multinom(
  y_train ~ .,
  data = data.frame(X_train_interact),
  MaxNWts = 10000,
  maxit = 500
)
```

### 3.2 Variable Interactions and Feature Engineering

**Requirement**: "Se recomienda añadir interacciones entre variables explicativas"

**Implementation**: ❌ **NOT IMPLEMENTED**

**Assessment**: ⚠️ MAJOR OMISSION

**Findings**:
- Current implementation uses raw features only
- Tree-based models (RF, XGBoost) can implicitly learn interactions
- However, explicit interaction terms could improve linear models

**Why Interactions Matter for Radar Data**:

1. **Range × RCS**: Object detectability depends on both distance and reflectivity
   - Far + Small RCS → Difficult detection
   - Close + Large RCS → Easy detection

2. **Azimuth × Range**: Detection accuracy varies with angle
   - Center field of view (azimuth ≈ 0) → High accuracy
   - Peripheral angles → Lower accuracy

3. **Velocity × Range**: Radial velocity interpretation depends on distance
   - Doppler effect varies with range

**Recommended Feature Engineering**:
```r
# Create interaction features
X_train_engineered <- X_train_scaled
X_train_engineered$range_rcs_interaction <- X_train_scaled[,"range_sc"] * 
                                            X_train_scaled[,"radar_cross_section"]
X_train_engineered$azimuth_range_interaction <- abs(X_train_scaled[,"azimuth_sc"]) * 
                                                X_train_engineered[,"range_sc"]
X_train_engineered$position_magnitude <- sqrt(X_train_scaled[,"x_cc"]^2 + 
                                               X_train_scaled[,"y_cc"]^2)
X_train_engineered$azimuth_squared <- X_train_scaled[,"azimuth_sc"]^2

# Polynomial features for non-linear relationships
X_train_engineered$range_squared <- X_train_scaled[,"range_sc"]^2
X_train_engineered$rcs_squared <- X_train_scaled[,"radar_cross_section"]^2
```

### 3.3 Class Imbalance Handling

**Implementation**: ✅ EXCELLENT

**Assessment**: ⭐⭐⭐⭐⭐ (5/5)

**Findings**:

**Approach Used**:
```r
class_weights <- 1 / table(y_train)
class_weights <- class_weights / sum(class_weights) * length(class_weights)
```

**Strengths**:
- ✅ Inverse frequency weighting correctly implemented
- ✅ Normalized to maintain reasonable scale
- ✅ Applied to both Random Forest (classwt) and XGBoost (weight)
- ✅ Visualized weight distribution

**Why This Is Good**:
1. **Addresses extreme imbalance**: Class 8 (40.95%) vs Class 1 (0.38%) = 107:1 ratio
2. **Penalizes misclassification**: Model pays more attention to minority classes
3. **Alternative to resampling**: More efficient than SMOTE/undersampling for 400k observations

**Alternative Approaches Considered** (implicitly):
- ✅ SMOTE mentioned in conclusions but not used (appropriate, as SMOTE can be expensive)
- ✅ Code includes ROSE library (loaded but not used)

**Minor Suggestion**:
Consider cost-sensitive learning with asymmetric costs:
```r
# If misclassifying a Person (7) as Large Vehicle (1) is more costly than reverse
cost_matrix <- matrix(c(
  # True: 0   1   5   7   10  11
         0,  5,  1,  10, 1,  1,   # Pred: 0
         1,  0,  1,  5,  1,  1,   # Pred: 1
         1,  1,  0,  1,  1,  1,   # Pred: 5
         10, 5,  1,  0,  1,  1,   # Pred: 7
         1,  1,  1,  1,  0,  1,   # Pred: 10
         1,  1,  1,  1,  1,  0    # Pred: 11
), nrow = 6, byrow = TRUE)
```

---

## 4. Performance Evaluation

### 4.1 Average Precision (AP) and Mean Average Precision (mAP)

**Requirement**: "Obtener el AP (Average Precision) por cada clase vs resto y el mAP (mean Average Precision) en entrenamiento y en validación"

**Implementation**: ❌ **NOT IMPLEMENTED**

**Assessment**: ⚠️ CRITICAL OMISSION

**Findings**:

**Current Metrics Used**:
- ✅ Accuracy
- ✅ Kappa
- ✅ Sensitivity (Recall)
- ✅ Specificity
- ✅ Precision (Positive Predictive Value)
- ✅ F1 Score
- ✅ Balanced Accuracy
- ✅ Confusion Matrix

**Missing Metrics**:
- ❌ **Average Precision (AP) per class**: Not computed
- ❌ **mean Average Precision (mAP)**: Not computed
- ❌ **Precision-Recall Curves**: Not plotted per class

**Why AP/mAP Matter**:
1. **Class-specific performance**: AP evaluates each binary "class vs rest" problem
2. **Threshold-independent**: AP summarizes performance across all thresholds
3. **Handles imbalance well**: More informative than accuracy for imbalanced data
4. **Standard in object detection**: mAP is the gold standard metric for detection systems

**Required Implementation**:

```r
library(PRROC)
library(pROC)

# Function to calculate AP for one-vs-rest
calculate_ap_per_class <- function(y_true, y_pred_proba, class_label) {
  # Binary labels: 1 if class_label, 0 otherwise
  y_binary <- as.numeric(y_true == class_label)
  
  # Predicted probability for this class
  y_score <- y_pred_proba[, as.character(class_label)]
  
  # Calculate Average Precision
  pr_curve <- pr.curve(
    scores.class0 = y_score[y_binary == 1],
    scores.class1 = y_score[y_binary == 0],
    curve = TRUE
  )
  
  return(list(
    ap = pr_curve$auc.integral,
    curve = pr_curve
  ))
}

# Get predicted probabilities (not just class labels)
# For XGBoost, need to use predict with outputmargin or type="prob"
params_prob <- params
params_prob$objective <- "multi:softprob"

xgb_model_prob <- xgb.train(
  params = params_prob,
  data = dtrain,
  nrounds = 100,
  watchlist = list(train = dtrain, test = dtest),
  early_stopping_rounds = 10,
  verbose = 0
)

# Get probability predictions
y_pred_train_proba <- predict(xgb_model_prob, dtrain, reshape = TRUE)
y_pred_val_proba <- predict(xgb_model_prob, dtest, reshape = TRUE)

# Calculate AP for each class
classes <- unique(y_train)
ap_train <- sapply(classes, function(cls) {
  calculate_ap_per_class(y_train, y_pred_train_proba, cls)$ap
})
ap_val <- sapply(classes, function(cls) {
  calculate_ap_per_class(y_test, y_pred_val_proba, cls)$ap
})

# Calculate mAP
mAP_train <- mean(ap_train)
mAP_val <- mean(ap_val)

# Results table
ap_results <- data.frame(
  Class = classes,
  AP_Train = ap_train,
  AP_Validation = ap_val,
  Difference = ap_train - ap_val
)
print(ap_results)
cat("\nmAP Train:", mAP_train, "\n")
cat("mAP Validation:", mAP_val, "\n")
```

### 4.2 Train/Validation Consistency and Overfitting Assessment

**Requirement**: "Verificar consistencia entre métricas de train/validación e identificar signos de overfitting o underfitting"

**Implementation**: ⚠️ PARTIALLY ADDRESSED

**Assessment**: ⭐⭐⭐☆☆ (3/5)

**Findings**:

**Current Approach**:
- ✅ Reports train and test accuracy separately
- ✅ XGBoost uses watchlist to monitor train/test loss during training
- ✅ Early stopping prevents severe overfitting
- ❌ No explicit comparison of train vs validation metrics side-by-side
- ❌ No discussion of overfitting/underfitting patterns

**What Should Be Done**:

1. **Create Comparison Table**:
```r
overfitting_analysis <- data.frame(
  Metric = c("Accuracy", "Macro F1", "mAP"),
  Train = c(
    conf_matrix_rf_train$overall['Accuracy'],
    mean(metricas_rf_train$F1, na.rm = TRUE),
    mAP_train
  ),
  Validation = c(
    conf_matrix_rf$overall['Accuracy'],
    mean(metricas_rf$F1, na.rm = TRUE),
    mAP_val
  )
)
overfitting_analysis$Gap <- overfitting_analysis$Train - 
                            overfitting_analysis$Validation
overfitting_analysis$Gap_Percent <- (overfitting_analysis$Gap / 
                                      overfitting_analysis$Train) * 100

print(overfitting_analysis)
```

2. **Interpretation Guidelines**:
   - **No overfitting**: Train ≈ Validation (gap < 5%)
   - **Mild overfitting**: 5% < gap < 10% (acceptable)
   - **Moderate overfitting**: 10% < gap < 20% (concerning)
   - **Severe overfitting**: gap > 20% (problematic)
   
   - **Underfitting**: Both train and validation scores are low (< 70% accuracy)

3. **Learning Curves**:
```r
# Plot training history for XGBoost
xgb_history <- xgb_model$evaluation_log

ggplot(xgb_history, aes(x = iter)) +
  geom_line(aes(y = train_mlogloss, color = "Train")) +
  geom_line(aes(y = test_mlogloss, color = "Validation")) +
  labs(title = "XGBoost Learning Curve",
       x = "Iteration",
       y = "Multiclass Log Loss",
       color = "Set") +
  theme_minimal()
```

**Expected Analysis**:
Based on the code, XGBoost likely shows:
- Good convergence (both train and validation loss decrease)
- Early stopping suggests optimal point found
- Small gap between train/validation indicates good generalization

However, **without AP metrics computed, we cannot fully assess performance consistency**.

### 4.3 Distance and Azimuth Effects on AP

**Requirement**: "¿Los valores de AP por clase vs resto mejoran en distancias más cercanas y ángulos de azimuth más próximos a 0? Justifica la respuesta."

**Implementation**: ❌ **NOT IMPLEMENTED**

**Assessment**: ⚠️ CRITICAL OMISSION

**Why This Analysis Is Important**:

1. **Radar Physics**: 
   - **Signal degradation**: Radar signal strength decreases with distance (inverse square law)
   - **Angular resolution**: Central field of view (azimuth ≈ 0) has better resolution
   - **Side-lobe effects**: Peripheral angles have more noise

2. **Practical Implications**:
   - If AP improves at close range → Model can be more aggressive in close-range scenarios
   - If AP degrades at large azimuth → Need additional sensors or conservative decisions at periphery
   - Helps define operational envelope for the system

**Required Implementation**:

```r
# Create range and azimuth bins
df_evaluation <- data.frame(
  range = X_validation[, "range_sc"],
  azimuth = X_validation[, "azimuth_sc"],
  y_true = y_validation,
  y_pred_proba = y_pred_val_proba
)

# Define bins
df_evaluation$range_bin <- cut(df_evaluation$range, 
                                breaks = c(0, 30, 60, 90, Inf),
                                labels = c("0-30m", "30-60m", "60-90m", ">90m"))

df_evaluation$azimuth_bin <- cut(abs(df_evaluation$azimuth),
                                  breaks = c(0, 0.2, 0.4, 0.6, Inf),
                                  labels = c("0-0.2rad", "0.2-0.4rad", 
                                            "0.4-0.6rad", ">0.6rad"))

# Calculate AP by range bin
ap_by_range <- df_evaluation %>%
  group_by(range_bin) %>%
  do({
    data <- .
    classes <- unique(data$y_true)
    ap_values <- sapply(classes, function(cls) {
      y_binary <- as.numeric(data$y_true == cls)
      y_score <- data$y_pred_proba[, as.character(cls)]
      pr.curve(scores.class0 = y_score[y_binary == 1],
               scores.class1 = y_score[y_binary == 0])$auc.integral
    })
    data.frame(
      class = classes,
      ap = ap_values,
      n_samples = nrow(data)
    )
  })

# Calculate AP by azimuth bin
ap_by_azimuth <- df_evaluation %>%
  group_by(azimuth_bin) %>%
  do({
    data <- .
    classes <- unique(data$y_true)
    ap_values <- sapply(classes, function(cls) {
      y_binary <- as.numeric(data$y_true == cls)
      y_score <- data$y_pred_proba[, as.character(cls)]
      pr.curve(scores.class0 = y_score[y_binary == 1],
               scores.class1 = y_score[y_binary == 0])$auc.integral
    })
    data.frame(
      class = classes,
      ap = ap_values,
      n_samples = nrow(data)
    )
  })

# Visualize
ggplot(ap_by_range, aes(x = range_bin, y = ap, color = factor(class), group = class)) +
  geom_line() +
  geom_point() +
  labs(title = "AP vs Distance Range",
       x = "Distance Range",
       y = "Average Precision",
       color = "Class") +
  theme_minimal()

ggplot(ap_by_azimuth, aes(x = azimuth_bin, y = ap, color = factor(class), group = class)) +
  geom_line() +
  geom_point() +
  labs(title = "AP vs Azimuth Angle",
       x = "Azimuth Range (absolute value)",
       y = "Average Precision",
       color = "Class") +
  theme_minimal()
```

**Expected Findings**:
- **Distance effect**: AP likely decreases with distance due to:
  - Lower SNR (Signal-to-Noise Ratio)
  - More measurement uncertainty
  - Smaller apparent RCS
  
- **Azimuth effect**: AP likely decreases at large angles due to:
  - Reduced antenna gain at periphery
  - Lower angular resolution
  - Increased multipath effects

**Justification Should Include**:
1. Quantitative comparison (e.g., "AP for Person class drops from 0.85 at 0-30m to 0.65 at >90m")
2. Statistical significance (e.g., "95% confidence intervals do not overlap")
3. Physical explanation (e.g., "consistent with radar propagation theory")
4. Practical recommendations (e.g., "operate conservatively beyond 80m range")

---

## 5. Precision-Recall Analysis and Threshold Selection

### 5.1 Person vs Rest Binary Classification

**Requirement**: "Elegir un punto de operación (umbral) de la curva de PR asociada a la clase persona vs resto en entrenamiento y comprobar los valores de Precision y Recall en validación correspondientes al umbral previamente definido en entrenamiento. Se pide priorizar la precisión del modelo."

**Implementation**: ❌ **NOT IMPLEMENTED**

**Assessment**: ⚠️ CRITICAL OMISSION

**Why This Analysis Is Critical**:

1. **Safety implications**: Person detection in autonomous/assisted driving systems
   - False Negative (missed person) → Potentially catastrophic
   - False Positive (false alarm) → Nuisance but safe

2. **Precision priority**: "Priorizar la precisión" suggests:
   - When model predicts "Person", it should be highly confident (high precision)
   - Acceptable to miss some persons (lower recall) to avoid false alarms
   - Typical in applications where acting on false positives is costly

**Required Implementation**:

```r
library(PRROC)

# Step 1: Create binary labels (Person vs Rest)
y_train_binary <- as.numeric(y_train == 7)  # 1 if Person, 0 otherwise
y_val_binary <- as.numeric(y_validation == 7)

# Step 2: Get probability scores for Person class
# Assuming XGBoost with softprob objective
y_score_train <- y_pred_train_proba[, "7"]  # Probability of class 7
y_score_val <- y_pred_val_proba[, "7"]

# Step 3: Calculate Precision-Recall curve on training set
pr_train <- pr.curve(
  scores.class0 = y_score_train[y_train_binary == 1],
  scores.class1 = y_score_train[y_train_binary == 0],
  curve = TRUE
)

# Step 4: Plot PR curve
plot(pr_train, main = "Precision-Recall Curve: Person vs Rest (Training)")

# Step 5: Select threshold prioritizing precision
# Option A: Target precision (e.g., 0.90)
target_precision <- 0.90
pr_data <- data.frame(
  threshold = pr_train$curve[, 1],
  recall = pr_train$curve[, 2],
  precision = pr_train$curve[, 3]
)
# Find threshold that achieves target precision
selected_row <- pr_data %>%
  filter(precision >= target_precision) %>%
  arrange(desc(recall)) %>%
  slice(1)
selected_threshold <- selected_row$threshold

cat("Selected threshold:", selected_threshold, "\n")
cat("Training - Precision:", selected_row$precision, 
    "Recall:", selected_row$recall, "\n")

# Step 6: Apply threshold to validation set
y_pred_val_binary <- as.numeric(y_score_val >= selected_threshold)

# Step 7: Evaluate on validation set
conf_matrix_val <- confusionMatrix(
  factor(y_pred_val_binary, levels = c(0, 1)),
  factor(y_val_binary, levels = c(0, 1)),
  positive = "1"
)

cat("\nValidation Results:\n")
cat("Precision:", conf_matrix_val$byClass["Pos Pred Value"], "\n")
cat("Recall:", conf_matrix_val$byClass["Sensitivity"], "\n")
cat("F1:", conf_matrix_val$byClass["F1"], "\n")

# Step 8: Compare train vs validation
comparison <- data.frame(
  Metric = c("Precision", "Recall", "F1"),
  Train = c(selected_row$precision, 
            selected_row$recall,
            2 * selected_row$precision * selected_row$recall / 
              (selected_row$precision + selected_row$recall)),
  Validation = c(conf_matrix_val$byClass["Pos Pred Value"],
                 conf_matrix_val$byClass["Sensitivity"],
                 conf_matrix_val$byClass["F1"])
)
comparison$Difference <- comparison$Train - comparison$Validation
print(comparison)
```

### 5.2 Threshold Selection Strategy

**Best Practices for Threshold Selection**:

1. **Business-Driven Approach** (Recommended):
   - Define minimum acceptable precision (e.g., 90%)
   - Among thresholds meeting precision requirement, maximize recall
   - Balances safety (high precision) with utility (reasonable recall)

2. **F-beta Score Approach**:
   ```r
   # F2 score weights recall higher (beta=2)
   # F0.5 score weights precision higher (beta=0.5)
   # For prioritizing precision, use F0.5
   
   f_beta <- function(precision, recall, beta = 0.5) {
     (1 + beta^2) * precision * recall / (beta^2 * precision + recall)
   }
   
   pr_data$f0.5 <- f_beta(pr_data$precision, pr_data$recall, beta = 0.5)
   optimal_threshold <- pr_data$threshold[which.max(pr_data$f0.5)]
   ```

3. **Cost-Based Approach**:
   ```r
   # Define costs
   cost_fn <- 1000  # Cost of False Negative (missed person)
   cost_fp <- 10    # Cost of False Positive (false alarm)
   
   # Calculate expected cost for each threshold
   pr_data$expected_cost <- (1 - pr_data$recall) * cost_fn + 
                            (1 - pr_data$precision) * pr_data$recall * 
                            cost_fp / (1 - pr_data$precision)
   optimal_threshold <- pr_data$threshold[which.min(pr_data$expected_cost)]
   ```

### 5.3 Generalization Assessment

**Requirement**: "¿Los resultados son similares entre training y validation? Justifica la respuesta"

**Expected Answer Structure**:

> **Comparison of Person vs Rest Classification:**
> 
> **Training Performance** (at selected threshold = [value]):
> - Precision: [X]%
> - Recall: [Y]%
> - F1: [Z]%
> 
> **Validation Performance** (at same threshold):
> - Precision: [X']%
> - Recall: [Y']%
> - F1: [Z']%
> 
> **Differences**:
> - Precision gap: [X - X']% ([small/moderate/large])
> - Recall gap: [Y - Y']% ([small/moderate/large])
> - F1 gap: [Z - Z']% ([small/moderate/large])
> 
> **Interpretation**:
> - If gaps < 5%: "Results are highly consistent, indicating excellent generalization"
> - If gaps 5-10%: "Results are reasonably consistent, with minor overfitting"
> - If gaps > 10%: "Significant discrepancy suggests overfitting or distribution shift"
> 
> **Possible Causes** (if results differ):
> 1. Overfitting to training set characteristics
> 2. Different class distributions between train/validation (if sequential split)
> 3. Temporal drift in data characteristics
> 4. Threshold calibration issues
> 
> **Recommendation**:
> [Specific actions based on findings, e.g., "Re-calibrate threshold on validation set" or "Use Platt scaling for better probability estimates"]

---

## 6. Overall Analytical Assessment

### 6.1 Analytical Rigor

**Rating**: ⭐⭐⭐☆☆ (3/5)

**Strengths**:
- ✅ Systematic workflow from data loading to model evaluation
- ✅ Multiple model comparison
- ✅ Proper handling of class imbalance
- ✅ Good code optimization and documentation
- ✅ Consideration of computational efficiency

**Weaknesses**:
- ❌ **Fundamental misalignment**: Does not implement key assignment requirements
- ❌ **Missing critical analyses**: RCS vs distance, AP/mAP, spatial effects
- ❌ **Wrong data split**: Random instead of sequential
- ❌ **No class reassignments**: Working with wrong class structure
- ❌ **Limited statistical testing**: No formal hypothesis tests for RCS differences
- ❌ **No feature engineering**: Despite explicit recommendation for interactions

### 6.2 Modeling Coherence

**Rating**: ⭐⭐⭐⭐☆ (4/5)

**Strengths**:
- ✅ Logical progression: baseline → simple models → complex models
- ✅ Appropriate algorithms for multiclass classification
- ✅ Proper preprocessing (normalization, handling missing classes)
- ✅ Weighted learning to address imbalance
- ✅ Error analysis to understand failure modes

**Weaknesses**:
- ⚠️ No hyperparameter tuning shown (grid search, random search, Bayesian optimization)
- ⚠️ No ensemble methods (stacking, blending)
- ⚠️ No feature selection or dimensionality reduction explored
- ⚠️ SVM mentioned but not evaluated (understandable due to computational cost)

### 6.3 Interpretability

**Rating**: ⭐⭐⭐⭐☆ (4/5)

**Strengths**:
- ✅ Feature importance plots (Random Forest, XGBoost)
- ✅ Confusion matrices with visualizations
- ✅ Per-class metrics reported
- ✅ Error analysis identifying most confused classes
- ✅ Clear documentation and comments

**Weaknesses**:
- ⚠️ No SHAP or LIME values for individual predictions
- ⚠️ No partial dependence plots to show feature effects
- ⚠️ Limited discussion of feature importance implications
- ⚠️ No visualization of decision boundaries (difficult in 9D, but could project to 2D)

### 6.4 Statistical Validity

**Rating**: ⭐⭐☆☆☆ (2/5)

**Strengths**:
- ✅ Appropriate metrics for imbalanced classification
- ✅ Stratified splitting maintains class proportions
- ✅ Multiple metrics reported (not just accuracy)

**Weaknesses**:
- ❌ **No confidence intervals** on performance metrics
- ❌ **No statistical significance testing** of model differences
- ❌ **No cross-validation** (k-fold or stratified)
- ❌ **Wrong split method** for assignment requirements
- ❌ **No bootstrap or permutation tests**
- ❌ **No analysis of prediction uncertainty**

**Required Improvements**:

1. **Confidence Intervals via Bootstrap**:
```r
library(boot)

bootstrap_accuracy <- function(data, indices) {
  d <- data[indices, ]
  acc <- sum(d$pred == d$true) / nrow(d)
  return(acc)
}

results_df <- data.frame(pred = y_pred_xgb, true = y_test)
boot_result <- boot(data = results_df, statistic = bootstrap_accuracy, R = 1000)
ci <- boot.ci(boot_result, type = "bca")

cat("Accuracy:", mean(boot_result$t), "\n")
cat("95% CI: [", ci$bca[4], ",", ci$bca[5], "]\n")
```

2. **McNemar's Test for Model Comparison**:
```r
# Compare if RF and XGBoost differ significantly
contingency_table <- table(
  RF_correct = y_pred_rf == y_test,
  XGB_correct = y_pred_xgb == y_test
)
mcnemar.test(contingency_table)
```

3. **Stratified K-Fold Cross-Validation**:
```r
library(caret)

train_control <- trainControl(
  method = "cv",
  number = 5,
  classProbs = TRUE,
  summaryFunction = multiClassSummary,
  savePredictions = "final"
)

cv_model <- train(
  x = X_train_scaled,
  y = y_train,
  method = "xgbTree",
  trControl = train_control,
  metric = "Mean_F1"
)

print(cv_model$resample)  # Performance across folds
```

---

## 7. Recommendations and Action Plan

### 7.1 Critical Fixes Required (Must Do)

**Priority 1: Align with Assignment Requirements**

1. **Implement Class Reassignments**
   ```r
   # After loading data
   y[y == 2] <- 1  # Truck → Large Vehicle
   y[y == 3] <- 1  # Bus → Large Vehicle  
   y[y == 8] <- 7  # Group of People → Person
   y_factor <- factor(y, levels = c(0, 1, 5, 7, 10, 11))
   ```
   **Impact**: Changes entire class structure and model task

2. **Change to Sequential Split**
   ```r
   X_train <- X[1:200000, ]
   X_validation <- X[200001:400000, ]
   y_train <- y[1:200000]
   y_validation <- y[200001:400000]
   ```
   **Impact**: Tests temporal generalization, matches requirements

3. **Add RCS vs Distance Analysis**
   - Plot median ± MAD for Person (7) vs Large Vehicle (1)
   - Perform statistical test (Welch's t-test, Mann-Whitney U)
   - Calculate and interpret effect size (Cohen's d)
   **Impact**: Core analysis requirement

4. **Implement AP/mAP Computation**
   - Calculate AP for each class vs rest
   - Compute mAP for train and validation
   - Compare to assess generalization
   **Impact**: Assignment requires specific metrics

5. **Spatial AP Analysis**
   - Bin data by range (distance) and azimuth
   - Calculate AP within each bin
   - Visualize and interpret trends
   **Impact**: Required spatial performance analysis

6. **Person vs Rest Threshold Selection**
   - Generate PR curve for Person class
   - Select threshold prioritizing precision
   - Evaluate on validation set
   - Compare train/validation performance
   **Impact**: Core requirement for operational deployment

### 7.2 Important Improvements (Should Do)

**Priority 2: Enhance Modeling**

1. **Add Feature Interactions**
   ```r
   # Implement as recommended in assignment
   X_enhanced <- model.matrix(
     ~ . + range_sc:radar_cross_section + 
          azimuth_sc:range_sc +
          x_cc:y_cc,
     data = as.data.frame(X_train_scaled)
   )
   ```

2. **Implement Multinomial Logistic Regression**
   - Provides interpretable coefficients
   - Works well with interaction terms
   - Baseline for comparison

3. **Add Statistical Tests**
   - Confidence intervals on metrics
   - Significance tests for model comparisons
   - Cross-validation for robust estimates

4. **Stratified Sampling for Training**
   - If computational resources are limited
   - Maintain class proportions
   - Document sampling strategy

### 7.3 Nice to Have (Could Do)

**Priority 3: Advanced Enhancements**

1. **Model Ensembling**
   ```r
   # Weighted average of predictions
   ensemble_proba <- 0.5 * y_pred_rf_proba + 0.5 * y_pred_xgb_proba
   ```

2. **Calibration Analysis**
   ```r
   library(CalibrationCurves)
   val.prob.ci.2(y_pred_proba, y_true)
   ```

3. **SHAP Values for Interpretability**
   ```r
   library(shapviz)
   shap_values <- shapviz(xgb_model, X_test_scaled)
   sv_importance(shap_values)
   ```

4. **Deployment Considerations**
   - Model size optimization
   - Inference time benchmarking
   - Quantization for edge devices

### 7.4 Documentation Improvements

1. **Add Assignment Compliance Section**
   - Checklist of all requirements
   - Status of each requirement
   - Justification for any deviations

2. **Enhance Conclusions Section**
   - Explicitly answer all assignment questions
   - Quantitative results for each analysis
   - Clear recommendations with supporting evidence

3. **Include Limitations Section**
   - Independence assumption of detections
   - Missing classes in data (4, 6, 9)
   - Temporal drift considerations
   - Generalization to different radar systems

---

## 8. Summary and Final Grade

### 8.1 Compliance with Assignment

| Requirement | Status | Notes |
|------------|--------|-------|
| Load data and reassign classes | ❌ Not Done | Critical omission |
| Sequential train/validation split | ❌ Not Done | Used random 70-30 instead |
| RCS vs distance plot for Person/Large Vehicle | ❌ Not Done | Core visualization missing |
| Statistical test for RCS difference | ❌ Not Done | Question not answered |
| Multiclass model with interactions | ⚠️ Partial | Model built but no interactions |
| AP per class and mAP computation | ❌ Not Done | Wrong metrics used |
| Distance/azimuth effect on AP | ❌ Not Done | Analysis not performed |
| Person vs rest threshold selection | ❌ Not Done | Binary classification missing |
| Prioritize precision | ❌ Not Done | Threshold strategy not implemented |
| Compare train/validation for Person class | ❌ Not Done | Comparison not made |

**Compliance Score**: 1/10 requirements fully met

### 8.2 Technical Quality Assessment

| Aspect | Rating | Comments |
|--------|--------|----------|
| Code Quality | ⭐⭐⭐⭐⭐ | Excellent structure, optimization, documentation |
| Machine Learning Skills | ⭐⭐⭐⭐☆ | Good algorithms and imbalance handling |
| Statistical Rigor | ⭐⭐☆☆☆ | Missing formal tests and validation |
| Assignment Compliance | ⭐☆☆☆☆ | Major deviation from requirements |
| Domain Understanding | ⭐⭐⭐☆☆ | Good ML but limited radar-specific analysis |
| Interpretability | ⭐⭐⭐⭐☆ | Good visualizations and error analysis |
| **Overall Technical Grade** | **⭐⭐⭐☆☆** | **3/5 - Good work but wrong assignment** |

### 8.3 Final Recommendation

**Current State**: 
This is a well-executed general multiclass classification project with excellent code quality and ML best practices. However, it does not address the specific requirements of Entregable 2.

**Path Forward**:

**Option A: Complete Rewrite (Recommended)**
- Implement all missing requirements from scratch
- Use existing code as reference for structure and optimizations
- Estimated effort: 2-3 days

**Option B: Incremental Fix**
- Keep current models but add all required analyses
- Risk: Some analyses may reveal need for different approach
- Estimated effort: 1-2 days + potential rework

**Option C: Hybrid Approach**
- Create new notebook addressing requirements explicitly
- Keep current notebook as "alternative approach" appendix
- Estimated effort: 2 days

**Recommendation**: **Option A** - The misalignment is too fundamental to patch. Starting fresh with requirements in mind will produce cleaner, more coherent work.

---

## 9. Detailed Action Items

### Phase 1: Data Preparation (2 hours)
- [ ] Implement class reassignments (bus, truck → large vehicle; group → person)
- [ ] Verify new class distribution
- [ ] Implement sequential split (first 200k train, next 200k validation)
- [ ] Document differences from current approach

### Phase 2: Exploratory Analysis (3 hours)
- [ ] Create RCS vs distance plot with median ± MAD
- [ ] Perform Welch's t-test for RCS difference
- [ ] Calculate Cohen's d effect size
- [ ] Write interpretation paragraph answering assignment question

### Phase 3: Modeling (4 hours)
- [ ] Create interaction features
- [ ] Train multinomial logistic regression with interactions
- [ ] Retrain Random Forest and XGBoost with new classes and split
- [ ] Compare all three models

### Phase 4: Performance Evaluation (4 hours)
- [ ] Implement AP calculation for each class vs rest
- [ ] Calculate mAP for train and validation
- [ ] Create AP comparison table
- [ ] Analyze train/validation gaps

### Phase 5: Spatial Analysis (3 hours)
- [ ] Bin data by distance ranges
- [ ] Bin data by azimuth angles
- [ ] Calculate AP within each bin
- [ ] Create visualizations showing AP trends
- [ ] Write interpretation answering assignment question

### Phase 6: Threshold Selection (3 hours)
- [ ] Generate PR curve for Person vs rest (train)
- [ ] Select threshold prioritizing precision (≥90%)
- [ ] Apply threshold to validation set
- [ ] Compare precision/recall train vs validation
- [ ] Write interpretation paragraph

### Phase 7: Documentation (2 hours)
- [ ] Update all sections with new results
- [ ] Write conclusions explicitly answering all questions
- [ ] Add limitations and recommendations sections
- [ ] Review for clarity and completeness

**Total Estimated Effort**: ~21 hours

---

## Appendix A: Code Template for Missing Analyses

### A.1 Class Reassignments
```r
# After: load("Entregable2.RData")
cat("Original class distribution:\n")
print(table(y))

# Reassign classes
y[y == 2] <- 1  # Truck → Large Vehicle
y[y == 3] <- 1  # Bus → Large Vehicle
y[y == 8] <- 7  # Group of People → Person

cat("\nNew class distribution:\n")
print(table(y))

# Define factor with correct levels
y_factor <- factor(y, levels = c(0, 1, 5, 7, 10, 11))
```

### A.2 Sequential Split
```r
# Sequential split: first 200k train, next 200k validation
n_train <- 200000
n_val <- 200000

X_train <- X[1:n_train, ]
X_validation <- X[(n_train + 1):(n_train + n_val), ]
y_train <- y_factor[1:n_train]
y_validation <- y_factor[(n_train + 1):(n_train + n_val)]

cat("Train set size:", nrow(X_train), "\n")
cat("Validation set size:", nrow(X_validation), "\n")
cat("\nTrain class distribution:\n")
print(table(y_train))
cat("\nValidation class distribution:\n")
print(table(y_validation))
```

### A.3 RCS Analysis
```r
# Filter for Person (7) and Large Vehicle (1)
mask <- y_train %in% c(1, 7)
df_rcs <- data.frame(
  range = X_train[mask, "range_sc"],
  rcs = X_train[mask, "radar_cross_section"],
  clase = y_train[mask]
)

# Create distance bins
df_rcs$range_bin <- cut(df_rcs$range, 
                        breaks = seq(0, max(df_rcs$range), by = 10),
                        include.lowest = TRUE)

# Calculate median ± MAD
library(dplyr)
rcs_stats <- df_rcs %>%
  group_by(range_bin, clase) %>%
  summarise(
    median_rcs = median(rcs),
    mad_rcs = mad(rcs),
    n = n()
  ) %>%
  filter(n >= 30)

# Plot
library(ggplot2)
ggplot(rcs_stats, aes(x = range_bin, y = median_rcs, color = factor(clase))) +
  geom_point() +
  geom_line(aes(group = clase)) +
  geom_errorbar(aes(ymin = median_rcs - mad_rcs, 
                    ymax = median_rcs + mad_rcs), width = 0.2) +
  labs(title = "RCS (median ± MAD) vs Distance",
       subtitle = "Person (7) vs Large Vehicle (1) - Training Set",
       x = "Distance Range (m)",
       y = "Radar Cross Section (dBsm)",
       color = "Class") +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))

# Statistical test
t_test <- t.test(rcs ~ clase, data = df_rcs, var.equal = FALSE)
print(t_test)

# Effect size
library(effsize)
cohens <- cohen.d(df_rcs$rcs[df_rcs$clase == 7],
                  df_rcs$rcs[df_rcs$clase == 1])
print(cohens)
```

### A.4 AP/mAP Calculation
```r
library(PRROC)

# Function to calculate AP for one class vs rest
calc_ap <- function(y_true, y_pred_proba, target_class) {
  y_binary <- as.numeric(y_true == target_class)
  y_score <- y_pred_proba[, as.character(target_class)]
  
  pr <- pr.curve(
    scores.class0 = y_score[y_binary == 1],
    scores.class1 = y_score[y_binary == 0]
  )
  
  return(pr$auc.integral)
}

# Get probability predictions
# (Requires training model with objective = "multi:softprob")
y_pred_train_proba <- predict(xgb_model_prob, dtrain, reshape = TRUE)
y_pred_val_proba <- predict(xgb_model_prob, dtest, reshape = TRUE)

# Calculate AP for each class
classes <- levels(y_train)
ap_train <- sapply(classes, function(c) 
  calc_ap(y_train, y_pred_train_proba, c))
ap_val <- sapply(classes, function(c) 
  calc_ap(y_validation, y_pred_val_proba, c))

# Mean AP
mAP_train <- mean(ap_train, na.rm = TRUE)
mAP_val <- mean(ap_val, na.rm = TRUE)

# Results
ap_results <- data.frame(
  Class = classes,
  AP_Train = ap_train,
  AP_Validation = ap_val,
  Difference = ap_train - ap_val
)
print(ap_results)
cat("\nmAP Train:", mAP_train, "\n")
cat("mAP Validation:", mAP_val, "\n")
```

---

**End of Review**

*This review was conducted with the goal of providing constructive, actionable feedback to improve the deliverable's alignment with requirements and analytical rigor. The author demonstrates strong technical skills; with focused effort on the specific requirements, this can become an excellent submission.*
