## This is a Chaotic Puma Wrapper Based feature selection for IoMT based CyberSecurity implemented using Logistic Regression and Random Forest as Base Models
## We were able to achieve  encouraging results using MedSec-25 dataset

# Network Intrusion Detection with Chaotic Puma Feature Selection

This project focuses on building a robust network intrusion detection system using machine learning techniques. The core objective is to classify network traffic into various attack types or benign activity. A key aspect of this project involved implementing a novel metaheuristic feature selection algorithm, 'Chaotic Puma', under aggressive time constraints to identify the most relevant features for classification.

## Dataset

The project utilizes a network traffic dataset, `MedSec-25.csv`, which includes various flow features and a 'Label' column indicating the type of traffic (e.g., Reconnaissance, Initial access, Exfiltration, Lateral movement, Benign).

## Methodology

The project followed these key steps:

1.  **Data Loading and Initial Exploration:** The dataset was loaded into a pandas DataFrame. Initial exploration included checking dimensions, column names, unique values in the 'Label' column, missing values, descriptive statistics, and data types.
2.  **Feature Engineering:**
    *   The 'Timestamp' column was converted to datetime objects, and new time-based features (`Hour`, `DayOfWeek`, `Month`) were extracted.
    *   `Flow ID` was dropped due to a high number of unique values.
    *   `Src IP` and `Dst IP` were one-hot encoded to convert categorical IP addresses into numerical features.
    *   The 'Label' column (target variable) was label encoded into numerical values.
3.  **Data Preprocessing:**
    *   Numerical features (excluding the time-based and target features) were scaled using `StandardScaler`.
    *   The original 'Timestamp' column was dropped.
4.  **Train-Test Split:** The data was split into training and testing sets (`X_train`, `X_test`, `y_train`, `y_test`) with a `test_size=0.2` and stratified sampling to maintain class distribution.
5.  **Feature Leakage Mitigation:** Features highly correlated with the target due to potential data leakage (`DayOfWeek`, `Hour`, `Src_IP_192.168.1.108`, and `Month`) were identified and removed, creating `X_reduced`.
6.  **Re-split and Re-scale Reduced Data:** The reduced dataset was split again (`X_train_reduced`, `X_test_reduced`, `y_train_reduced`, `y_test_reduced`) and scaled appropriately.
7.  **Chaotic Puma Feature Selection (on downsampled data):**
    *   To meet a strict 20-minute time constraint for feature selection, a **5% stratified subset** of `X_train_reduced` and `y_train_reduced` was created (`X_train_puma`, `y_train_puma`).
    *   A simplified objective function, `evaluate_subset`, was defined, employing a single 80/20 train-validation split and a lightweight `LogisticRegression` model (`max_iter=100`, `solver='liblinear'`) for rapid evaluation of feature subsets.
    *   The 'Chaotic Puma' algorithm was implemented with aggressive parameters (`population_size = 15`, `num_iterations = 7`).
    *   This process identified a subset of **97 optimal features**.
8.  **Model Training and Evaluation (with selected features):**
    *   The selected 97 features were applied to the *full* `X_train_reduced` and `X_test_reduced` datasets, creating `X_train_selected` and `X_test_selected`.
    *   Two classification models were trained and evaluated on this feature-reduced dataset:
        *   **Logistic Regression**
        *   **Random Forest Classifier**

## Models Implemented and Results

### 1. Logistic Regression Model

*   **Training Data:** `X_train_selected` (full training data with 97 features) and `y_train_reduced`.
*   **Evaluation Data:** `X_test_selected` and `y_test_reduced`.
*   **Performance Metrics:**
    *   **Accuracy Score:** **0.9313**
    *   **Classification Report:**

| Label                   | Precision | Recall | F1-Score | Support |
| :---------------------- | :-------- | :----- | :------- | :------ |
| **Benign (0)**          | 0.94      | 0.90   | 0.92     | 2469    |
| **Exfiltration (1)**    | 0.86      | 0.68   | 0.76     | 5183    |
| **Initial access (2)**  | 0.87      | 0.88   | 0.88     | 20418   |
| **Lateral movement (3)**| 0.61      | 0.85   | 0.71     | 2500    |
| **Reconnaissance (4)**  | 0.96      | 0.96   | 0.96     | 80337   |
| **Macro Avg**           | 0.85      | 0.85   | 0.85     | 110907  |
| **Weighted Avg**        | 0.93      | 0.93   | 0.93     | 110907  |

*   **Observations:** The Logistic Regression model performed well for majority classes (`Reconnaissance`, `Benign`, `Initial access`) but showed noticeable struggles with minority classes (`Exfiltration`, `Lateral movement`), as indicated by lower F1-scores and off-diagonal elements in the confusion matrix.

### 2. Random Forest Classifier

*   **Training Data:** `X_train_selected` (full training data with 97 features) and `y_train_reduced`.
*   **Evaluation Data:** `X_test_selected` and `y_test_reduced`.
*   **Performance Metrics:**
    *   **Accuracy Score:** **0.9958**
    *   **Classification Report:**

| Label                   | Precision | Recall | F1-Score | Support |
| :---------------------- | :-------- | :----- | :------- | :------ |
| **Benign (0)**          | 0.99      | 0.99   | 0.99     | 2469    |
| **Exfiltration (1)**    | 0.98      | 0.96   | 0.97     | 5183    |
| **Initial access (2)**  | 1.00      | 1.00   | 1.00     | 20418   |
| **Lateral movement (3)**| 0.92      | 0.94   | 0.93     | 2500    |
| **Reconnaissance (4)**  | 1.00      | 1.00   | 1.00     | 80337   |
| **Macro Avg**           | 0.98      | 0.98   | 0.98     | 110907  |
| **Weighted Avg**        | 1.00      | 1.00   | 1.00     | 110907  |

*   **Observations:** The Random Forest classifier demonstrated exceptionally strong performance across all classes, including the minority ones. Its F1-scores were uniformly high, close to 1.0, and the confusion matrix showed minimal misclassifications, indicating a highly robust and accurate model.

## Model Comparison Summary

| Metric        | Logistic Regression | Random Forest |
| :------------ | :------------------ | :------------ |
| **Accuracy**  | 0.9313              | 0.9958        |
| **Precision** | 0.85                | 0.98          |
| **Recall**    | 0.85                | 0.98          |
| **F1-score**  | 0.85                | 0.98          |

The Random Forest model significantly outperformed Logistic Regression across all macro-averaged metrics, establishing itself as the superior choice for this intrusion detection task.

## Chaotic Puma Feature Selection: Compromises & Observations

To adhere to a strict 20-minute time constraint for feature selection, significant compromises were made:

*   **Data Downsampling:** Only a 5% stratified subset of the training data was used for the Chaotic Puma algorithm, drastically reducing the search space and computational load.
*   **Simplified Objective Function:** Feature subsets were evaluated using a single 80/20 train-validation split and a lightweight Logistic Regression model (`max_iter=100`) instead of more thorough cross-validation or complex models. This prioritized speed over optimal evaluation.
*   **Aggressive Parameters:** The Chaotic Puma algorithm itself was configured with minimal iterations (7) and a small population size (15).

Despite these aggressive limitations, the algorithm successfully identified a subset of **97 features** that yielded a final global best score of **0.9394** on the downsampled data during its internal evaluation. While the time constraint was met, it is important to note that the quality of the selected feature subset might be suboptimal compared to an unconstrained run. However, the subsequent performance of the Random Forest model on the full dataset, even with this constrained feature selection, highlights the effectiveness of the chosen features.

## Conclusion

This project successfully implemented and evaluated an intrusion detection system using a constrained Chaotic Puma feature selection approach. While the feature selection process involved compromises due to time limits, the resulting **Random Forest classifier demonstrated outstanding performance**, achieving an accuracy of **0.9958** and near-perfect F1-scores across all traffic types, including challenging minority classes. This indicates that even with limited computational resources for feature selection, powerful ensemble models can achieve highly effective results for network intrusion detection.
