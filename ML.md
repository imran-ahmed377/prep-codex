# Table of Contents

- [Machine Learning Q&A](#machine-learning-qa)
    - [Overfitting](#overfitting)
    - [Underfitting](#underfitting)
    - [Bias and Variance](#bias-and-variance)
    - [Supervised Learning](#supervised-learning)
    - [Unsupervised Learning](#unsupervised-learning)
    - [Difference Between Classification & Regression](#difference-between-classification--regression)
    - [Cross Validation](#cross-validation)
    - [Regularization](#regularization)
    - [Precision & Recall](#precision--recall)
    - [Confusion Matrix](#confusion-matrix)
    - [F1 Score](#f1-score)
    - [scikit-learn vs XGBoost](#scikit-learn-vs-xgboost)
    - [Gradient boosting (high level)](#gradient-boosting-high-level)
    - [Hyperparameters](#hyperparameters)
        

- [MLOps Q&A](#mlops-qa)
    - [Versioning Dataset, Code, and Model](#versioning-dataset-code-and-model)
    - [MLOps Lifecycle](#mlops-lifecycle)
    - [Deploying Model into Production](#deploying-model-into-production)
    - [Monitoring Model in Production](#monitoring-model-in-production)
    - [Monitoring Data Drift](#monitoring-data-drift)

- [Hallucinations Handling](#hallucinations-handling)






# Machine Learning Q&A

## Overfitting
- **Definition**: High training accuracy caused by noise and random patterns in training data, resulting in low test accuracy.

## Underfitting
- **Definition**: The model is too simple to capture the underlying patterns. It performs poorly on both training and testing data.

## Bias and Variance
- **Bias**: Error due to overly simple assumptions (causes **underfitting**).
- **Variance**: Error due to too much complexity (causes **overfitting**).

## Supervised Learning
- **Definition**: Uses labeled data to train the model.
- **Examples**: Classification, Regression.

## Unsupervised Learning
- **Definition**: Uses unlabeled data to find hidden patterns.
- **Examples**: Clustering, Dimensionality Reduction.

## Difference Between Classification & Regression
- **Classification**: Predicts discrete categories (e.g., Spam or Not Spam).
- **Regression**: Predicts continuous numbers (e.g., House prices).

## Cross Validation
- **Definition**: Splitting data into multiple parts and training/testing multiple times to ensure model stability.
- **Example**: K-Fold Cross Validation.

## Regularization
- **Definition**: Adding a penalty term to the model complexity to reduce overfitting. It slightly “punishes” large weights (coefficients), so the model avoids relying too heavily on any one feature.
- Imagine you’re packing a bag:
    - L1 (Lasso): You throw out some items completely → lighter bag
    - L2 (Ridge): You keep everything but reduce weight of each item → more balanced bag

## Precision & Recall
- **Precision**: Out of all predicted positives, how many are actually correct.
- **Recall**: Out of all actual positives, how many were correctly predicted.

## Confusion Matrix
- **Definition**: A table used to evaluate the performance of classification models.

| | Predicted Positive | Predicted Negative |
| :--- | :---: | :---: |
| **Actual Positive** | **TP** (True Positive) | **FN** (False Negative) |
| **Actual Negative** | **FP** (False Positive) | **TN** (True Negative) |

## F1 Score
- **Definition**: The harmonic mean of precision and recall, providing a balance between the two metrics.

## scikit-learn vs XGBoost
- scikit-learn is a general ML library. XGBoost is specialized for fast, high-performance gradient boosting.

## Gradient boosting (high level)
- Build models sequentially. Each new model focuses on correcting errors from the previous ones, combining them into a strong final model.

## Hyperparameters
- Hyperparameters are settings you choose before training the model (not learned automatically).
Examples:

    - Learning rate (how fast the model learns)
    - Number of trees in a model
    - Regularization strength (L1/L2)

### Common tuning methods
#### Grid Search

    Try all combinations of values. Example:
    - Learning rate: [0.01, 0.1]
    - Regularization: [0.1, 1]

    Try:

    (0.01, 0.1)\
    (0.01, 1)\
    (0.1, 0.1)\
    (0.1, 1)

    👉 Best but can be slow

#### Random Search

    Try random combinations instead of all.

    👉 Faster, often works just as well

#### Smarter methods (advanced)
    Bayesian optimization (chooses better values based on past results)

    Imagine baking a cake:

    Hyperparameters: Oven temperature, Baking time

    You try:

    180°C for 30 min
    200°C for 25 min
    170°C for 40 min

    👉 Then taste each cake and pick the best one.



# MLOps Q&A

## Versioning Dataset, Code, and Model
*   **Git**: Used for versioning code.
*   **DVC (Data Version Control)**: Used for versioning datasets and large files.
*   **MLflow**: Used for tracking experiments and versioning models.

---

## MLOps Lifecycle
The standard lifecycle follows these key stages:
1.  **Data Collection**
2.  **Preprocessing**
3.  **Training**
4.  **Evaluation**
5.  **Deployment**
6.  **Monitoring**

---

## Deploying Model into Production
*   **Containerization**: Use **Docker** to package the model and its dependencies.
*   **API Exposure**: Use frameworks like FastAPI or Flask to expose the model via an API.
*   **Orchestration**: Deploy and manage containers using **Kubernetes**.

---

## Monitoring Model in Production
To ensure the model remains reliable, track the following metrics:
*   **Accuracy**: Is the model still making correct predictions?
*   **Latency**: How long does it take for the model to respond?
*   **Errors**: Monitoring for system crashes or failed requests.
*   **Drift**: Monitoring changes in data or model performance over time.
*   **Tools**: Use Dashboards, Logging, and Alerts.

---

## Monitoring Data Drift
Data drift occurs when the input data changes over time, leading to model decay.
*   **Continuous Monitoring**: Watch for statistical shifts in input features.
*   **Regular Retraining**: Schedule model training sessions on new data.
*   **Trigger Alerts**: Automatically notify the team when performance drops below a threshold.

    ### Data Drift Detection Techniques
    1. Compare averages (mean): Check if the average value of a feature has changed.
    2. Compare distributions (histograms): Look at how values are spread, not just the average.
    3. Statistical tests: Example: Kolmogorov-Smirnov (KS) Test Compare two dataset cumulative distribution (CDF) curves at every point, check the difference between them.

---

# Hallucinations Handling:
I track hallucinations using RAGAS faithfulness metrics and query logs, and I reduce them through strong retrieval, strict prompting, confidence thresholds, and continuous evaluation.

1. How I Track:
I used the RAGAS framework to measure answer faithfulness, which checks whether the generated response is actually grounded in the retrieved context. I also monitor context precision and recall to ensure the model was retrieving the right information before generating answers. Also, I logged user queries and flagged cases where the response contained information not present in retrieved documents, The confidence score (similarity score) was below a defined threshold. I occasionally reviewed these failed or low-confidence cases to identify patterns of hallucination. 

2. How I Reduced:
a. Strong Retrieval (Most Important)
I improved retrieval quality using top-K vector search with cosine similarity in Azure AI Search. 
I also used hybrid search (BM25 + vector search) to ensure both semantic and keyword relevance. 

b. Prompt Engineering Controls
I designed strict system prompts such as:
“Answer only based on the provided context. If the information is not available, say ‘I don’t have enough data.’” 
This significantly reduced the model’s tendency to “make up” answers. 

c. Confidence Threshold + Fallback
I applied a similarity score threshold (e.g., cosine similarity cutoff). 
If no retrieved documents met the threshold, the system returned a fallback response instead of generating a hallucinated answer. 

d. Context Grounding
I ensured that only relevant, high-quality chunks (500–1000 tokens) were passed to the model. 
I avoided overloading the prompt with noisy or irrelevant data, which can increase hallucination risk. 

e. Model Choice
I used gpt-4o because it has stronger grounding and instruction-following capabilities compared to smaller models, which helped reduce hallucinations. 

f. Continuous Feedback Loop
I logged hallucinated responses and used them to: 
Improve prompt design 
Adjust retrieval parameters (top-K, chunking) 
Refine embeddings 

Result / Impact
These techniques significantly improved answer faithfulness and trustworthiness 
Reduced hallucination cases over time through iterative tuning 
Increased user confidence in the system for real-time decision-making 

---
