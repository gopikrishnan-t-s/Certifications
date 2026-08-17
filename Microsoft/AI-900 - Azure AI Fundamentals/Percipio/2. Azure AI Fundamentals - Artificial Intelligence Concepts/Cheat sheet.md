### **Azure AI Fundamentals: Artificial Intelligence Concepts Cheat sheet (AI-900 Prep)**

---

### **1. Core Machine Learning (ML) Types**
* **Supervised Learning**: Models train on a **labeled dataset** to map input features to known target outputs.
* **Unsupervised Learning**: Models analyze **unlabeled data** to discover hidden structures and groupings without explicit human-provided labels.
* **Semi-Supervised Learning**: Combines a small amount of **labeled data** with a larger pool of **unlabeled data** to balance high performance with lower labeling costs.
* **Self-Supervised Learning**: Models autonomously generate **pseudo-labels** and supervisory signals internally from unstructured/unlabeled data.
* **Reinforcement Learning**: Software agents interact with an environment through trial-and-error, optimizing behavior based on **rewards** (positive feedback) or **penalties** (negative feedback).

---

### **2. Data Handling & Dataset Partitioning**
* **Data Ingestion**: Standardizing inputs from databases, cloud storage, or APIs using streaming (real-time) or batch-processing pipelines.
* **Exploratory Data Analysis (EDA)**: Profiling raw data to understand distributions, patterns, and anomalies.
* **The Three Dataset Splits**:
  - **Training Dataset**: Used directly to train the model and adjust parameters (weights).
  - **Validation Dataset**: Used during training to fine-tune hyperparameters and prevent overfitting.
  - **Test Dataset**: An independent benchmark used after training to evaluate the final model's real-world predictive accuracy.

---

### **3. Labeled vs. Unlabeled Data**
* **Labeled Data**: Annotated with explicit tags serving as ground truth.
  - *Multi-label Data*: Individual instances that belong to multiple non-exclusive categories simultaneously (e.g., an image labeled as both "dog" and "animal").
  - *Quality Metrics*: Measured via gold standard benchmarks, consensus audits, or the **Intersection over Union (IOU)** metric in computer vision (evaluating overlap between predicted and actual bounding boxes).
* **Unlabeled Data**: Lacks annotations entirely. Primarily processed via clustering, dimensionality reduction, and anomaly detection.

---

### **4. Data Features & Feature Selection**
Features are measurable properties or characteristics of your data (numeric, string, or graph values). Selecting optimal features prevents model overfitting (mitigating the bias/variance trade-off).

* **Filtering Methods**: Evaluate and select features based on independent statistical properties (e.g., calculating P-values).
* **Wrapper Methods**: Train models on various subset combinations to optimize directly against a model performance metric.
* **Hybrid & Embedded Methods**: Incorporate selection directly into the training algorithm (e.g., Random Forest, XGBoost, or genetic algorithms).
* **Execution Techniques**:
  - *Forward Selection*: Starts with empty features and adds the most informative one iteratively.
  - *Backward Elimination*: Starts with all features and iteratively removes the least informative.
  - *Recursive Feature Elimination (RFE)*: Recursively ranks and prunes features to find the optimal subset.

---

### **5. Key ML Methods & Algorithms**

#### **A. Regression (Continuous Outcomes)**
Predicts continuous numerical values along a spectrum (answers "how much?" or "how many?").
* **Key Algorithms**:
  - *Linear Regression*: Models relationships using a straight-line equation.
  - *Polynomial Regression*: Models non-linear relationships using exponential terms.
  - *Lasso Regression*: Penalizes the absolute size of coefficients to promote sparsity (feature selection).
  - *Ridge Regression*: Penalizes the squared magnitude of coefficients to reduce overfitting.
  - *Elastic Net*: Combines Lasso and Ridge penalties.
* **Evaluation Metrics**:
  - **MAE (Mean Absolute Error)**: Average absolute difference between predicted and actual values.
  - **MSE (Mean Squared Error)**: Emphasizes larger errors by squaring the differences.
  - **RMSE (Root Mean Squared Error)**: Square root of MSE, returning the metric to the original unit scale.
  - **R² (Coefficient of Determination)**: The proportion of variance explained by the model (goodness of fit).

#### **B. Classification (Categorical Outcomes)**
Categorizes data into distinct classes or groups.
* **Types of Classification**:
  - *Binary*: Two exclusive categories (e.g., Yes/No, Diabetic/Not Diabetic).
  - *Multi-class*: More than two exclusive categories (e.g., Car, Truck, or Motorcycle).
  - *Multi-label*: Multiple overlapping categories simultaneously (e.g., Genre: Action and Sci-Fi).
  - *Imbalanced*: Datasets where one class heavily outnumbers the other.
* **Key Evaluation Metrics**:
  - **Confusion Matrix**: Table mapping True Positives (TP), True Negatives (TN), False Positives (FP), and False Negatives (FN).
  - **Accuracy**: $$(TP + TN) / \text{Total}$$
  - **Precision**: $$TP / (TP + FP)$$ (Ability to avoid false positives).
  - **Sensitivity / Recall**: $$TP / (TP + FN)$$ (Ability to find all true positive cases).
  - **F1 Score**: Harmonic mean of Precision and Recall, providing a balanced metric.
  - **AUC-ROC**: Curve showing the model's ability to discriminate classes across thresholds.

#### **C. Clustering (Group Finding)**
Groups unlabeled data points together based on inherent mathematical similarities.
* **Key Approaches**:
  - *Centroid-based (K-Means)*: Partitions data into K clusters around central points (centroids).
  - *Density-based (DBSCAN, OPTICS)*: Groups dense regions separated by low-density noise. Highly robust to outliers.
  - *Distribution-based (Gaussian Mixture Models - GMM)*: Groups points based on probability densities.
  - *Hierarchical (BIRCH, Agglomerative)*: Organizes data points into nested tree structures (dendrograms).

---

### **6. Deep Learning & Artificial Neural Networks (ANNs)**
Deep learning is an advanced subset of ML that mimics the human brain's interconnected neural structure.

* **Structure**: Composed of an **input layer**, one or more **hidden layers** (making it a Deep Neural Network, or DNN), and an **output layer**.
* **The 7-Step Learning Loop**:
  1. Training and validation features are fed into the **input layer**.
  2. Hidden layers apply **weights** (transformation functions) to the data.
  3. The **output layer** produces calculated predictions ($$\hat{y}$$).
  4. A **loss function** calculates the difference between predicted values ($$\hat{y}$$) and known actual values ($$y$$).
  5. An optimization function (using calculus/gradient descent) evaluates how changing each weight affects the overall loss.
  6. Changes to the weights are **back-propagated** through the network to update them.
  7. This entire process is repeated over multiple iterations (**epochs**) until the loss is minimized.
