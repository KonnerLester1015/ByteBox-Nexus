---
title: "AWS AI and ML"
---

## Executive Summary

AWS AI and ML services span pre-built AI APIs and fully managed ML tooling. Teams can adopt ready-made services or build custom models depending on skill and workload requirements. 

---

## Understanding AI and ML

### Artificial Intelligence (AI)

AI refers to systems that perform tasks requiring human-like intelligence, such as understanding language, identifying objects, or making decisions.

### Machine Learning (ML)

ML is a subset of AI where models learn patterns from historical data and use those patterns to make predictions on new data.

**Typical ML Workflow**

1. Collect and prepare data  
2. Train and evaluate a model  
3. Deploy the model  
4. Continuously monitor and improve performance  

---

## AI/ML Architecture Layers in AWS

AWS organizes its ML services into three tiers, each serving different user skill levels and sophistication requirements.

| Tier | Primary Service | Best For |
|------|----------------|----------|
| Tier 1 | Amazon Comprehend, Rekognition, Textract | Adding AI capabilities through APIs |
| Tier 2 | Amazon SageMaker | Building, training, and deploying custom models |
| Tier 3 | Amazon EC2, EKS, EMR | Full control over frameworks and infrastructure |

---

### Tier 1 — AI Services (Pre-Built Models)

AI services provide ready-made intelligence without requiring ML expertise. These services solve common business problems through simple API calls.

#### Language AI

- **Amazon Comprehend** — Extracts sentiment, key phrases, entities, and insights from text.  
- **Amazon Transcribe** — Converts speech to text with speaker identification and custom vocabularies.  
- **Amazon Translate** — Provides real-time and batch language translation.  
- **Amazon Polly** — Converts text into natural-sounding speech for accessibility or voice-enabled apps.

#### Vision and Search AI

- **Amazon Rekognition** — Detects faces, objects, scenes, and text in images and videos.  
- **Amazon Textract** — Extracts structured and unstructured data from forms and documents.  
- **Amazon Kendra** — Enterprise search using natural language understanding to deliver precise answers.

#### Conversational and Personalization AI

- **Amazon Lex** — Builds conversational chatbot experiences with ASR and NLU.  
- **Amazon Personalize** — Generates real-time, personalized recommendations using historical user behavior.

---

### Tier 2 — ML Services (Build, Train, Deploy Your Own Models)

#### Amazon SageMaker

SageMaker is a fully managed machine learning platform that simplifies the entire ML lifecycle—including data preparation, model training, deployment, and monitoring.

**Key Capabilities**

- SageMaker Studio IDE for managing ML workflows  
- Built-in algorithms and notebooks  
- Automated model tuning and training job tracking  
- Real-time and batch inference endpoints  
- SageMaker JumpStart for deploying pretrained models  

SageMaker is ideal for data scientists and ML engineers wanting more control without managing servers.

---

### Tier 3 — Frameworks and Custom ML Infrastructure

Advanced users can run their own ML frameworks using:

- TensorFlow  
- PyTorch  
- MXNet  

These run on services such as:

- Amazon EC2 ML-optimized instances  
- Amazon EMR for distributed data processing  
- Amazon ECS or Amazon EKS for containerized ML workloads  

This tier is designed for organizations that require complete control over training environments.

---

## Deep Learning and Generative AI

### Deep Learning

Deep learning uses multi-layer neural networks to recognize complex patterns. It powers many modern AI capabilities including voice assistants, image recognition, and NLP models.

### Generative AI

Generative AI leverages foundation models (FMs) which are large models pre-trained on massive datasets. They are highly adaptable, these models can execute a wide range of tasks, including text generation, summarization, image creation, and coding.

**AWS Generative AI Services**

- **Amazon Bedrock** — Fully managed service for deploying and customizing foundation models from AWS and third-party providers.  
- **SageMaker JumpStart** — Provides prebuilt FMs and templates for quick deployment.  
- **Amazon Q** — An enterprise-ready AI assistant for business and development workflows.

---

## Data Analytics

Data analytics is the process of transforming raw data into insights. Both AI/ML and analytics require clean, accessible data.

### The ETL Process

1. **Extract** — Gather data from source systems  
2. **Transform** — Clean, enrich, and standardize data  
3. **Load** — Store data into a destination like a data warehouse  

AWS supports ETL and data pipelines with services like AWS Glue, Amazon Kinesis, Amazon EMR, Lambda, and Step Functions.

## Key Terms Summary

| Term | Definition |
|------|-----------|
| AI | Systems that perform tasks requiring human-like intelligence |
| ML | Models that learn patterns from data |
| Deep Learning | Neural network-based machine learning |
| Foundation Model | Large pre-trained model adaptable to many tasks |
| ETL | Extract, Transform, Load |
| Inference | Using a trained model to make predictions |