### **Azure AI Fundamentals: Artificial Intelligence Principles Cheatsheet**

---

### **1. The Six Principles of Responsible AI**
Microsoft defines six foundational principles to guide the ethical development and deployment of AI systems:
- **Fairness**: AI must minimize bias and treat all individuals equitably, avoiding discrimination.
- **Reliability & Safety**: Systems must exhibit fault tolerance, include fallback capabilities, and perform consistently within defined parameters. 
- **Privacy & Security**: AI must protect sensitive data from exploitation and adhere to "privacy by design" standards.
- **Inclusiveness**: AI should empower diverse user groups and avoid representational harm or opportunity denial.
- **Transparency**: AI behavior must be understandable. Tools like **Google Model Cards** (documentation artifacts) and **Explainable AI (XAI)** help interpret "black box" non-deterministic models.
- **Accountability**: Organizations must maintain governance, audit trails, and clear policies to answer for their AI system outcomes.

---

### **2. AI Bias & Exclusion Risks**
Bias in training data or system design can lead to severe societal impacts, such as representational harm, opportunity denial, and disproportionate product failures.
- **Dataset Bias**: Training data is not representative of the target population.
- **Automation Bias**: Human tendency to blindly trust algorithmic decisions over human judgment.
- **Association Bias**: The system makes unjustified correlations between attributes and outcomes.
- **Interaction Bias**: AI treats users differently based on demographics or past interactions.
- **Confirmation Bias**: The algorithm reinforces existing societal stereotypes or prejudices.
- **Proxy Variables**: Indirect attributes that correlate with sensitive data, leading to hidden discrimination if not removed.

---

### **3. Privacy & Compliance Frameworks**
Responsible AI must align with major global data protection regulations:
- **CCPA (California)**: Grants consumers the right to know, delete, opt-out of data sales, and receive non-discriminatory treatment.
- **HIPAA (Healthcare)**: Mandates strict confidentiality, integrity, and security safeguards for Protected Health Information (PHI) handled by covered entities and business associates.
- **GDPR (European Union)**: Requires lawful processing, explicit consent, and data minimization. Mandates Data Protection Impact Assessments (DPIAs), prompt breach notifications, and the appointment of a Data Protection Officer (DPO) for large-scale operations.

---

### **4. Azure AI Capabilities & Tools**
Azure provides a hybrid ecosystem of prebuilt APIs and customizable environments supporting multiple languages (Python, .NET, Java, JavaScript).
- **Prebuilt Services**: Ready-to-use Cognitive Services and Bot Services for vision, speech, language, and decision-making.
- **Azure Machine Learning Workspaces**: Centralized, collaborative environments for managing datasets, models, experiments, and pipelines.
- **Azure ML SDK (Python)**: Code-first approach featuring the `command()` function for interactive training and **Automated ML (AutoML)** for automatic algorithm selection and hyperparameter tuning.
- **Azure ML Designer**: A drag-and-drop web UI allowing users to build and train models with no-code/low-code visually.
- **Azure ML CLI**: Command-line interface for scripting and automating ML tasks and deployment jobs.
- **Azure AI Content Moderator**: An API resource used to scan text, images, and video to filter out offensive, racy, or inappropriate material.

---

### **5. MLOps & Model Management**
Machine Learning Operations (MLOps) applies DevOps principles (CI/CD) to machine learning to automate, scale, and govern the model lifecycle.
- **Reproducibility & Environments**: MLOps utilizes reusable software environments (like Docker or Conda) to ensure consistent execution across platforms.
- **Azure ML Pipelines**: Independently executable, automated workflows that standardize data preparation, training, and deployment at scale.
- **Azure ML Assets**: Categorized into Data assets (raw/labeled data), Model assets (trained models/metrics), and Component assets (reusable code modules).
- **Git Integration**: Azure ML seamlessly tracks commits, history, and artifacts without requiring a centralized repository, integrating with GitHub, Azure Repos, or Bitbucket.
