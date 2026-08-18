# Flipkart Order Intelligence & Support Assistant

## Overview

This project is an end-to-end **Flipkart-style Order Intelligence and Customer Support Assistant** that combines machine learning, deep learning, document retrieval, and LangGraph.

The project is divided into three parts:

* **Part 1:** Predict whether an order is likely to be returned.
* **Part 2:** Identify the category of a product from an image.
* **Part 3:** Build a support agent that brings the previous two models together with a policy knowledge base.

The main idea was to build the project step by step and then connect everything into one support system. The final agent can answer policy questions, check return risk, classify product images, and handle follow-up questions in a conversation.

---

# Project Architecture

The overall workflow looks like this:

```text
                         Customer
                            │
                            ▼
                  ┌──────────────────┐
                  │  LangGraph       │
                  │  Support Agent   │
                  └────────┬─────────┘
                           │
             ┌─────────────┼─────────────┐
             │             │             │
             ▼             ▼             ▼
       Policy Query   Return Risk    Image Query
             │             │             │
             ▼             ▼             ▼
        FAISS Index     Part 1 ML      Part 2 Model
             │             │             │
             └─────────────┼─────────────┘
                           │
                           ▼
                     Final Answer
```

The support agent acts as the main layer connecting all three parts.

---

# Repository Structure

```text
Flipkart-Order-Intelligence/
│
├── README.md
├── generate_orders.py
├── part_1_train_evaluation.py
├── part_2_train_evaluation.py
├── support_agent.py
│
├── data/
│   └── orders_dataset.csv
│
├── Model/
│   ├── return_risk_model.pkl
│   └── product_classifier.pt
│
├── Knowledge base/
│   ├── policies.json
│   └── chunks.json
│
├── vector_store/
│   ├── index.faiss
│   └── index_info.json
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
└── orders_dataset.xlsx
```

---

# Part 1 — Return-Risk Scoring

The first part of the project focuses on predicting whether an order is likely to be returned.

I generated a synthetic e-commerce dataset containing information about the order, customer, payment method, product, and delivery.

Some of the features include:

* Product category
* Payment method
* Product price
* Discount percentage
* Customer tenure
* Previous orders
* Previous returns
* Delivery distance
* Delivery time
* Weekend order
* Customer rating

## Dataset

The generated dataset contains:

* **6,000 orders**
* **13 columns**
* **22.75% return rate**
* **13.05% missing values in `rating_given`**

The missing values in `rating_given` were intentionally generated based on the payment method. This was done to create a realistic **Missing At Random (MAR)** pattern.

## Generate the Dataset

The dataset can be regenerated using:

```bash
python generate_orders.py
```

The CSV file is saved in:

```text
data/orders_dataset.csv
```

An Excel version is also included:

```text
orders_dataset.xlsx
```

## Training and Evaluation

The complete Part 1 workflow is in:

```text
part_1_train_evaluation.py
```

The script covers:

1. Loading the dataset
2. Preprocessing the data
3. Handling missing values
4. Encoding categorical variables
5. Splitting the data
6. Training baseline and ML models
7. Evaluating the models
8. Tuning the final model
9. Saving the trained model

The final return-risk model is saved as:

```text
Model/return_risk_model.pkl
```

This saved model is later used by the support agent in Part 3.

---

# Part 2 — Product Image Categoriser

The second part of the project focuses on product image classification.

For this part, I used the **Fashion-MNIST** dataset and transfer learning to build a product image classifier.

The dataset contains:

* **60,000 training images**
* **10,000 test images**
* **10 classes**

The model was trained using GPU acceleration where available.

## Training and Evaluation

The complete Part 2 workflow is in:

```text
part_2_train_evaluation.py
```

The evaluation includes classification metrics and a confusion matrix to see which product categories the model is able to distinguish well and where it makes mistakes.

The trained model is saved as:

```text
Model/product_classifier.pt
```

This model is then loaded by the Part 3 support agent whenever an image needs to be classified.

---

# Part 3 — LangGraph Support Agent

Part 3 is the main user-facing part of the project.

The goal here was to take the models from Parts 1 and 2 and make them useful inside a single customer-support workflow.

The support agent is built using **LangGraph** and can work with:

* The policy knowledge base
* The Part 1 return-risk model
* The Part 2 image classifier
* Conversation history

The main application is:

```text
support_agent.py
```

## Example: Policy Question

A customer might ask:

```text
How long can I return apparel or footwear?
```

The agent searches the policy knowledge base and uses the retrieved information to answer the question.

## Example: Return-Risk Question

A customer might ask:

```text
Can you check the return risk for this order?
```

The agent loads:

```text
Model/return_risk_model.pkl
```

and uses the trained Part 1 model to make the prediction.

## Example: Image Question

A customer might ask:

```text
What product category is shown in this image?
```

The agent uses:

```text
Model/product_classifier.pt
```

to classify the image.

---

# Knowledge Base

For policy-related questions, I created a small Flipkart-style knowledge base.

The policy documents are stored in:

```text
Knowledge base/policies.json
```

The documents are broken into smaller chunks for retrieval and stored in:

```text
Knowledge base/chunks.json
```

The knowledge base covers topics such as:

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
* Delivery policies
* Return restrictions

The purpose of the knowledge base is to give the support agent a reliable source to use when answering policy questions instead of relying only on the language model.

---

# Vector Store

The policy chunks are converted into embeddings and stored in a **FAISS** vector index.

The vector store contains:

```text
vector_store/
├── index.faiss
└── index_info.json
```

`index.faiss` contains the searchable vector index, while `index_info.json` stores the information needed to connect retrieved vectors back to the original policy chunks.

The retrieval process is:

```text
policies.json
      ↓
Chunking
      ↓
chunks.json
      ↓
Embeddings
      ↓
FAISS
      ↓
Relevant Chunks
      ↓
Support Agent
      ↓
Answer
```

---

# Multi-Turn Conversations

Customer-support conversations are usually more than one question, so the agent also maintains conversation state.

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

This allows the user to ask follow-up questions without having to repeat all the information from the previous message.

---

# Testing

I included transcript-based tests to check how the support agent behaves in different situations.

The tests are stored in:

```text
transcripts/
```

The test cases cover:

* Apparel and footwear policy questions
* Electronics policy questions
* Return-risk prediction
* Product-image classification
* Multi-turn conversations
* Fresh conversations
* Policy retrieval
* Tool usage
* Mixed support queries

The tests are useful for checking not only the final answer but also whether the correct capability is being used for each type of request.

---

# Model Files

The trained models are stored in:

```text
Model/
├── return_risk_model.pkl
└── product_classifier.pt
```

### `return_risk_model.pkl`

This is the trained Part 1 return-risk model.

It is loaded by the support agent when a customer asks about the return probability of an order.

### `product_classifier.pt`

This is the trained Part 2 image classification model.

It is loaded when the customer provides a product image that needs to be classified.

Keeping the trained models as separate files means they do not have to be retrained every time the support agent is started.

---

# How to Run

## 1. Generate the Dataset

```bash
python generate_orders.py
```

This creates the order dataset used in Part 1.

---

## 2. Train Part 1

```bash
python part_1_train_evaluation.py
```

This trains and evaluates the return-risk model and saves:

```text
Model/return_risk_model.pkl
```

---

## 3. Train Part 2

```bash
python part_2_train_evaluation.py
```

This trains and evaluates the image classifier and saves:

```text
Model/product_classifier.pt
```

---

## 4. Check the Knowledge Base

Make sure these files are present:

```text
Knowledge base/
├── policies.json
└── chunks.json
```

and:

```text
vector_store/
├── index.faiss
└── index_info.json
```

These files are required for policy retrieval.

---

## 5. Start the Support Agent

```bash
python support_agent.py
```

The support agent can now handle policy questions and use the trained models when necessary.

---

# Technologies Used

| Technology            | Used For                                    |
| --------------------- | ------------------------------------------- |
| Python                | Main development                            |
| Pandas                | Data processing                             |
| NumPy                 | Dataset generation and numerical operations |
| Scikit-learn          | Machine learning                            |
| PyTorch               | Deep learning                               |
| Torchvision           | Fashion-MNIST and computer vision           |
| LangGraph             | Agent workflow and state                    |
| LangChain             | Retrieval and agent components              |
| FAISS                 | Vector search                               |
| Sentence Transformers | Text embeddings                             |
| Joblib                | Saving/loading the ML model                 |

---

# Reproducibility

The project can be run in the following order:

```text
Generate dataset
      ↓
Train Part 1
      ↓
Save return-risk model
      ↓
Train Part 2
      ↓
Save image classifier
      ↓
Prepare policy knowledge base
      ↓
Create/load FAISS index
      ↓
Run support agent
      ↓
Run transcript tests
```

Each part can be developed and tested separately, while Part 3 brings everything together into the final application.

---

# What This Project Demonstrates

This project covers several parts of a practical AI/ML workflow:

* Synthetic data generation
* Data preprocessing
* Missing-data handling
* Machine-learning model training
* Model evaluation and tuning
* Transfer learning
* Image classification
* Model saving and reuse
* Document chunking
* Semantic search
* FAISS vector retrieval
* LangGraph agent workflows
* Tool-based model usage
* Grounded policy responses
* Multi-turn conversations
* Application-level testing

---

# Conclusion

The **Flipkart Order Intelligence & Support Assistant** brings together the different parts of the project into one practical workflow.

In **Part 1**, I built a machine-learning model that predicts the likelihood of an order being returned.

In **Part 2**, I used transfer learning to build a model that can identify product categories from images.

In **Part 3**, I connected both models with a policy knowledge base and a LangGraph support agent. The agent decides what information or tool it needs depending on the customer's question.

For policy questions, it uses document retrieval. For return-risk questions, it uses the saved Part 1 model. For image questions, it uses the saved Part 2 model.

The project also includes multi-turn conversations and transcript-based testing so that the final system is evaluated as a support application rather than just as a collection of individual models.

Overall, the project demonstrates how **machine learning, deep learning, retrieval, and agent-based workflows can be combined to build a complete AI-powered e-commerce support system.**
