### **Azure AI Fundamentals: Azure Machine Learning Fundamentals Cheat sheet (AI-900 Prep)**

---

### **1. Core Artificial Intelligence (AI) Paradigms**
* **Assisted Intelligence**: AI systems aid human decision-making by offering insights and recommendations based on data patterns.
* **Augmented Intelligence**: AI enhances human capabilities, allowing individuals to accomplish tasks more efficiently.
* **Autonomous Intelligence**: Systems operate and make decisions independently without human intervention (e.g., self-driving cars, autonomous drones).

---

### **2. Machine Learning (ML) Training Methods**
ML blends computer science with mathematics (statistics and linear algebra) to train models through two main phases: **training** (learning patterns from data) and **inferencing** (making predictions on new data).

* **Supervised Learning**: Model is trained on **labeled data** (inputs paired with correct outputs).
  - **Classification**: Predicting discrete categories or classes (e.g., dog vs. cat).
  - **Regression**: Predicting continuous numerical values (e.g., housing prices).
  - *Common Algorithms*: Decision trees, support vector machines, logistic regression, neural networks.
* **Unsupervised Learning**: Model operates on **unlabeled data** to discover hidden structures.
  - **Clustering**: Grouping data points based on similarity metrics (e.g., grouping vehicles into cars, trucks, and motorcycles).
  - **Dimensionality Reduction & Anomaly Detection**: Identifying outliers or unusual patterns.
* **Semi-Supervised Learning**: Combines a small amount of **labeled data** with a large pool of **unlabeled data** to significantly reduce expensive labeling costs while improving model performance.
* **Reinforcement Learning**: Software agents learn optimal behaviors through trial and error by interacting with an environment to maximize cumulative **rewards** (positive feedback) and minimize **penalties** (negative feedback).

---

### **3. Microsoft Azure AI Pillars & Services**

#### **Computer Vision**
Replicates human visual perception to extract and analyze standalone images or video streams.
* **Key Tasks**: 
  - *Object Detection*: Locating objects in an image and drawing bounding boxes around them.
  - *Object Identification*: Tracking and assigning unique identifiers to individual object instances.
  - *Motion Analysis*: Tracking movement over time (egomotion estimation, optical flow analysis).
* **Azure Services**: Image Analysis (classification/segmentation), Face Recognition (identity verification), and Optical Character Recognition (OCR - text extraction from images).

#### **Natural Language Processing (NLP)**
Enables computers to understand, interpret, and generate human voice and text.
* **Key Tasks**: Morphological analysis (lemmatization), syntactical analysis (parsing, part-of-speech tagging), semantic understanding, and discourse analysis (coherence).
* **Azure Speech Capabilities**: Speech-to-text (transcription), speech tagging, and meaning disambiguation (resolving homophones).

#### **Knowledge Mining**
Sifts through vast structured (databases) and unstructured (documents, videos, images) data pools to uncover concealed patterns using AI pipelines.
* **Key Uses**: Digital content organization, customer sentiment analysis, compliance auditing, and automated data extraction.

#### **Document Intelligence**
Automates document handling workflows by converting unstructured documents into usable, structured data.
* **Extraction Capabilities**: Text, tables, headers/footers, and key-value pairs (form fields).
* **Azure Models**: Offers both prebuilt models for common document types and customizable models.

#### **Generative AI**
Uses deep neural network architectures trained iteratively on large datasets to generate synthetic media (text, images, audio, video, 3D models) based on **natural language prompts**.
* **Key Concerns**: Output errors/inaccuracies, bias perpetuation, copyright/intellectual property infringement, and synthetic media risks (misinformation and deepfakes).

---

### **4. AI Security, Ethics, and Risks**
* **AI Security**: Requires robust intrusion detection (encryption, access controls), proactive threat identification (predictive analytics), and rapid event response.
* **Explainable AI (XAI)**: Frameworks designed to make AI decision-making transparent, interpretable, and auditable for regulators and stakeholders.
* **Common Risks**: Job displacement due to automation, overreliance undermining human judgment, cybersecurity vulnerabilities (malware, adversarial attacks), and legal/regulatory liability.

---

### **5. The 6 Principles of Responsible AI**
To ensure the ethical development and deployment of AI technologies, organizations must design systems around these six core pillars:

- **Fairness**: AI systems must treat everyone equitably without bias or discrimination based on race, gender, or socioeconomic status.
- **Reliability and Safety**: Systems must perform as intended within safe operational parameters, validated through rigorous testing and quality assurance.
- **Privacy and Security**: AI must integrate data protection, encryption, and "privacy by design" to safeguard sensitive user information.
- **Inclusiveness**: AI must empower and be accessible to people of all backgrounds, abilities, and physical capacities.
- **Transparency**: Systems must be understandable, providing clear explanations of algorithms, data sources, and decision processes.
- **Accountability**: Clear boundaries, organizational governance, and human oversight mechanisms must exist to answer for the outcomes of AI systems.
