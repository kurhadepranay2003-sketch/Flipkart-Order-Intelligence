

# Flipkart Order Intelligence & Support Assistant

## Overview

This project builds an end-to-end **Flipkart-style Order Intelligence and Customer Support Assistant** using machine learning, deep learning, document retrieval, and LangGraph.

The project is divided into three connected parts.

**Part 1** focuses on predicting whether an order is likely to be returned.

**Part 2** focuses on classifying product images into their respective categories.

**Part 3** brings everything together into a support assistant that can answer policy questions, use the return-risk model, classify product images, and maintain context during conversations.

The overall goal is to demonstrate how different machine-learning components can be combined into a practical AI-powered customer-support workflow.

---

# Project Architecture

The project follows this general flow:

```text
                    Customer
                       │
                       ▼
              ┌─────────────────┐
              │  Support Agent  │
              │   LangGraph     │
              └────────┬────────┘
                       │
          ┌────────────┼────────────┐
          │            │            │
          ▼            ▼            ▼
     Policy Query  Return Risk   Image Query
          │            │            │
          ▼            ▼            ▼
    Vector Store    Part 1 ML    Part 2 Model
          │            │            │
          └────────────┼────────────┘
                       │
                       ▼
                Final Response
```

This makes the project more than just a collection of individual models. The final support agent acts as the layer that connects the different capabilities.

---

# Repository Structure

The current repository is organised as follows:

```text
Flipkart-Order-Intelligence/
│
├── support_agent.py
├── part_2_train_evaluation.py
├── part_1_train_evaluation.py
├── generate_orders.py
│
├── transcripts/
│   ├── test_01_apparel_footwear_policy.txt
│   ├── test_02_electronics_policy.txt
│   ├── test_03_return_risk.txt
│   ├── test_04_product_image.txt
│   ├── test_05_multiturn_state.txt
│   ├── test_06_fresh_conversation.txt
│   ├── test_07_policy_retrieval.txt
│   ├── test_08_tool_usage.txt
│   └── test_09_mixed_support.txt
│
├── vector_store/
│   ├── index.faiss
│   └── index_info.json
│
├── Knowledge base/
│   ├── policies.json
│   └── chunks.json
│
├── data/
│   └── orders_dataset.csv
│
├── Model/
│   ├── return_risk_model.pkl
│   └── product_classifier.pt
│
└── orders_dataset.xlsx
```

---

# Part 1 — Return-Risk Scoring

The first part of the project focuses on predicting whether an order is likely to be returned.

A synthetic e-commerce dataset was generated with information such as:

* Product category
* Payment method
* Product price
* Discount
* Customer tenure
* Previous orders
* Previous returns
* Delivery distance
* Delivery time
* Weekend order indicator
* Customer rating

The dataset contains:

* **6,000 orders**
* **13 columns**
* Approximately **22.75% return rate**
* Approximately **13.05% missing values in `rating_given`**

The missingness in the rating variable was intentionally related to payment method so that the dataset could also demonstrate a realistic **Missing At Random (MAR)** pattern.

## Dataset Generation

The dataset can be regenerated using:

```bash
python generate_orders.py
```

The generated dataset is stored in:

```text
data/orders_dataset.csv
```

An Excel version of the dataset is also included:

```text
orders_dataset.xlsx
```

---

## Training and Evaluation

The Part 1 training and evaluation workflow is contained in:

```text
part_1_train_evaluation.py
```

The script handles the main steps of the machine-learning workflow, including:

1. Loading the order data
2. Data preprocessing
3. Handling missing values
4. Encoding categorical variables
5. Splitting the data
6. Training baseline and machine-learning models
7. Model evaluation
8. Model selection/tuning
9. Saving the final model

The final trained return-risk model is saved in:

```text
Model/return_risk_model.pkl
```

The saved model can then be loaded by the support agent without retraining it.

---

# Part 2 — Product Image Categoriser

The second part focuses on product image classification using deep learning and transfer learning.

The project uses the **Fashion-MNIST** dataset containing 10 product/clothing categories.

The dataset consists of:

* **60,000 training images**
* **10,000 test images**
* **10 classes**

The model was trained using GPU acceleration where available.

The Part 2 training and evaluation workflow is contained in:

```text
part_2_train_evaluation.py
```

The trained model is saved as:

```text
Model/product_classifier.pt
```

The evaluation process includes classification metrics and a confusion matrix to understand how well the model distinguishes between the different product categories.

---

# Part 3 — LangGraph Support Agent

Part 3 is the main user-facing part of the project.

The support assistant is implemented using **LangGraph** and connects the policy knowledge base with the two trained machine-learning models.

The main application is:

```text
support_agent.py
```

The assistant can handle different types of customer queries.

For example:

### Policy Question

```text
How long can I return apparel or footwear?
```

The agent retrieves the relevant policy information from the vector store before answering.

### Return-Risk Question

```text
Can you check the return risk for this order?
```

The agent uses the saved Part 1 model:

```text
Model/return_risk_model.pkl
```

### Product Image Question

```text
What product category is shown in this image?
```

The agent uses the saved Part 2 model:

```text
Model/product_classifier.pt
```

---

# Knowledge Base

The support agent uses a small policy knowledge base created for this project.

The source policy documents are stored in:

```text
Knowledge base/policies.json
```

The policy documents are divided into smaller chunks for retrieval and stored in:

```text
Knowledge base/chunks.json
```

The knowledge base contains information related to areas such as:

* Apparel
* Footwear
* Electronics
* Home products
* Beauty products
* Return windows
* Refunds
* Replacements
* Damaged products
* Product eligibility
* Delivery-related policies
* Return restrictions

The purpose of the knowledge base is to ensure that policy-related responses are based on the information provided to the system.

---

# Vector Store

The policy chunks are converted into embeddings and stored in a FAISS vector index.

The vector store contains:

```text
vector_store/
├── index.faiss
└── index_info.json
```

`index.faiss` contains the searchable vector index.

`index_info.json` contains the supporting information required to map retrieved results back to the relevant policy chunks.

The retrieval flow is:

```text
policies.json
      ↓
Document Chunking
      ↓
chunks.json
      ↓
Embeddings
      ↓
FAISS
      ↓
index.faiss
      ↓
Relevant Policy Chunks
      ↓
Support Agent
      ↓
Final Answer
```

---

# Support Agent Workflow

The support agent decides which capability is needed based on the user's request.

```text
                         User
                           │
                           ▼
                  ┌─────────────────┐
                  │  LangGraph      │
                  │  Support Agent  │
                  └────────┬────────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
   Policy Retrieval   Return-Risk Tool   Image Tool
          │                │                │
          ▼                ▼                ▼
    Vector Store      Part 1 Model       Part 2 Model
          │                │                │
          └────────────────┼────────────────┘
                           │
                           ▼
                    Final Response
```

This tool-based design allows each component to focus on one specific responsibility.

---

# Multi-Turn Conversations

Customer-support conversations are often not limited to a single question.

For example:

```text
User:
What is the return policy for footwear?

Agent:
[Provides footwear return policy]

User:
What if the product is damaged?

Agent:
[Uses the previous conversation context and answers the follow-up]
```

The LangGraph workflow maintains the conversation state so that follow-up questions can be handled using the information already discussed.

This is particularly useful when a customer provides information across multiple messages instead of giving everything in one request.

---

# Testing

The project includes transcript-based tests to verify different aspects of the support agent.

The test transcripts are stored in:

```text
transcripts/
```

The tests cover scenarios such as:

* Apparel and footwear policy questions
* Electronics policy questions
* Return-risk prediction
* Product-image classification
* Multi-turn conversation state
* Fresh conversations
* Policy retrieval
* Tool usage
* Mixed support queries

These tests help verify not only whether the agent produces an answer, but also whether it uses the appropriate retrieval or machine-learning tool when required.

---

# Model Artifacts

The trained models are stored in the `Model` directory:

```text
Model/
├── return_risk_model.pkl
└── product_classifier.pt
```

### `return_risk_model.pkl`

This is the trained Part 1 return-risk model.

It is loaded by the support agent when an order-return prediction is required.

### `product_classifier.pt`

This is the trained Part 2 image classification model.

It is loaded by the support agent when a product image needs to be classified.

Keeping these models as separate saved artifacts makes the system easier to reuse and avoids retraining them every time the support assistant starts.

---

# How to Run the Project

## Step 1 — Generate the Dataset

Run:

```bash
python generate_orders.py
```

This creates the order dataset.

---

## Step 2 — Train and Evaluate Part 1

Run:

```bash
python part_1_train_evaluation.py
```

This trains and evaluates the return-risk model and saves the resulting model under:

```text
Model/return_risk_model.pkl
```

---

## Step 3 — Train and Evaluate Part 2

Run:

```bash
python part_2_train_evaluation.py
```

This trains and evaluates the product image classification model and saves:

```text
Model/product_classifier.pt
```

---

## Step 4 — Prepare the Knowledge Base

The policy documents and retrieval chunks are stored under:

```text
Knowledge base/
```

The generated FAISS index is stored under:

```text
vector_store/
```

Make sure these files are available before running the support agent.

---

## Step 5 — Run the Support Agent

Start the final support assistant using:

```bash
python support_agent.py
```

The support agent can then answer policy questions and use the trained ML models when required.

---

# Technologies Used

The project uses a combination of data-science and AI technologies:

* **Python** — Main programming language
* **Pandas** — Data manipulation
* **NumPy** — Numerical operations and dataset generation
* **Scikit-learn** — Machine-learning models and evaluation
* **PyTorch** — Deep-learning model development
* **Torchvision** — Computer-vision utilities and Fashion-MNIST
* **LangGraph** — Agent workflow and state management
* **LangChain** — Retrieval and agent components
* **FAISS** — Vector similarity search
* **Sentence Embeddings** — Converting policy documents into searchable vectors
* **Joblib** — Saving and loading the return-risk model

---

# Reproducibility

The complete project can be reproduced in the following order:

```text
Generate order dataset
        ↓
Train and evaluate Part 1
        ↓
Save return-risk model
        ↓
Train and evaluate Part 2
        ↓
Save product classifier
        ↓
Prepare policy knowledge base
        ↓
Create retrieval chunks
        ↓
Build/load FAISS vector store
        ↓
Run LangGraph support agent
        ↓
Run transcript-based tests
```

This structure keeps the project modular while allowing all three parts to work together as a single application.

---

# Conclusion

The **Flipkart Order Intelligence & Support Assistant** demonstrates how different areas of artificial intelligence can be combined to solve a realistic e-commerce support problem.

The project starts with **Part 1**, where customer and order information is used to predict the likelihood of an order being returned. This part demonstrates the complete machine-learning lifecycle, from generating and preparing data to training, evaluating, and saving a model that can later be reused by another application.

**Part 2** extends the project into computer vision. A transfer-learning model is trained to identify product categories from images. The trained model is saved as a reusable PyTorch artifact rather than being limited to the training environment.

The most important part of the project is **Part 3**, where these individual capabilities are brought together through a LangGraph-based support assistant. The agent is able to understand what type of help the customer needs and use the appropriate capability. A policy question can be answered using the retrieval system, an order-related question can be handled using the return-risk model, and an image-related question can be handled using the product classifier.

The project also highlights the importance of **grounded responses** in customer-support applications. Instead of relying entirely on generated answers, the support agent retrieves information from a defined policy knowledge base. This makes the system easier to maintain because policy information can be updated in the knowledge base without changing the entire agent workflow.

Another important aspect is the use of **reusable model artifacts**. The models trained in Parts 1 and 2 are saved and later loaded by Part 3. This reflects a more realistic machine-learning deployment process, where model training and application usage are separate stages.

The transcript-based testing further adds a practical layer to the project. It checks different types of interactions, including policy questions, model-based requests, image classification, tool usage, and multi-turn conversations. This helps evaluate the support assistant as an actual application rather than only checking individual model metrics.

Overall, the project demonstrates a complete journey from **data generation and model development to retrieval, tool integration, and an interactive AI support assistant**. It shows how traditional machine learning, deep learning, vector search, and agentic AI can work together to create a more capable e-commerce support system.

The final application is therefore not simply a set of independent models. It is a connected system where each component has a clear role, and the LangGraph support agent acts as the layer that brings those capabilities together into one customer-facing workflow.
