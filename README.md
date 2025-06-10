# ☁️ Cloud-Native Dirt Detection System

An end-to-end **serverless AI solution** for detecting dirt on public surfaces, enabling large-scale sanitation audits and smart city applications using AWS and Google Vertex AI.

---

## 🧩 Project Overview

This project implements a **cloud-native, serverless computer vision pipeline** to detect and analyze dirt on public surfaces like floors and walls. It ingests images from client devices, processes them using advanced AI models, and returns structured cleanliness reports.

Built with **AWS Lambda**, **API Gateway**, **S3**, **DynamoDB**, and **CloudWatch**, and enhanced by **Google's Vertex AI**, the system offers high scalability, low operational overhead, and intelligent insight generation.
---
## 🏗️ Architecture Diagram

![Architecture Diagram](architecture.png)
---

## 🏗️ Architecture Overview

The system uses a **modular, event-driven serverless design**. Below is a high-level breakdown:

### 🔹 1. Image Upload (Client → S3)
- Clients upload images directly to Amazon S3 using pre-signed URLs.
- Reduces Lambda/API Gateway cost by avoiding binary payloads.

### 🔹 2. Metadata Submission (Client → API Gateway)
- Clients submit metadata (image path, area type, audit ID) to API Gateway.
- Triggers a Lambda that stores metadata in DynamoDB.

### 🔹 3. Metadata Lambda
- Writes metadata to `ImagesTable`.
- Triggers the AI Inference Lambda asynchronously.

### 🔹 4. AI Inference Lambda
- Retrieves unprocessed image metadata.
- Downloads images from S3.
- Applies:
  - **Gemini Pro Vision** for image captioning and cleanliness classification.
  - **YOLOv8** for dirty area localization.
- Calculates cleanliness scores and writes results to various DynamoDB tables.

### 🔹 5. YOLOv8 Lambda
- Invoked for dirty wall/floor surfaces.
- Returns bounding boxes and annotated images.

### 🔹 6. Results Query Lambda
- Handles client queries.
- Retrieves summaries, scores, annotations from DynamoDB.
- Logs to CloudWatch.

### 🔹 7. Observability
- All Lambda functions log to CloudWatch for monitoring, performance, and alerts.

---

## 📌 Design Highlights

- **Serverless**: Auto-scaled with no infrastructure management.
- **Event-Driven**: Functions invoked only on trigger events.
- **Decoupled Workflow**: Enhances reliability and system throughput.
- **Multi-Level Summarization**: Insights aggregated by image, element, area, and user.

---

## 🤖 AI Logic Summary

| Task                                | Tool Used                           |
|-------------------------------------|-------------------------------------|
| Captioning & Cleanliness Classification | Gemini Pro Vision 1.5               |
| Dirt Region Detection (Bounding Boxes) | YOLOv8 via AWS Lambda               |
| Summary Generation                  | Gemini Pro 1.0                      |
| Cleanliness Scoring                 | Custom logic (scale 1–5)            |

---

## 📊 DynamoDB Table Schemas

### `ImagesTable`

| Column             | Type      | Description                          |
|--------------------|-----------|--------------------------------------|
| ImageID            | String PK | Unique image identifier              |
| AuditID            | String    | Related audit                        |
| AreaID             | String    | Area where image was taken           |
| UserID             | String    | Uploading user                       |
| ImagePath          | String    | S3 path to image                     |
| IsProcessed        | Boolean   | Processing status                    |
| IsClean            | Boolean   | Cleanliness result                   |
| ImageCaption       | String    | Auto-generated caption               |
| BoundingBoxes      | List      | Dirty region coordinates             |
| ImagePathofYOLO    | String    | Annotated image path (if applicable) |
| AreaElementType    | String    | Type of surface                      |
| AreaElementID      | String    | Area element reference               |
| LastUpdateTimestamp| String    | ISO timestamp                        |

### `AreasTable`

| Column               | Type      | Description                     |
|----------------------|-----------|---------------------------------|
| AreaID               | String PK | Unique area ID                  |
| AuditID              | String SK | Related audit                   |
| AreaCaptionSummary   | String    | Summary of all images           |
| AreaScore            | Number    | Average cleanliness score       |
| ProcessingState      | String    | Status flag                     |
| BuildingID           | String    | Associated building             |
| LastUpdateTimestamp  | String    | ISO timestamp                   |

### `AreaElementsTable`

| Column                    | Type       | Description                     |
|---------------------------|------------|---------------------------------|
| AreaElementID             | Number PK  | Unique surface type instance    |
| AuditID                   | String     | Associated audit                |
| AreaID                    | String     | Related area                    |
| AreaElementType           | String     | Surface type (e.g., Wall)       |
| AreaElementCaptionSummary| String     | Caption summary for this type   |
| AreaElementScore          | Number     | Cleanliness score (1–5)         |
| ProcessingState           | String     | Current status                  |
| Timestamp                 | String     | Audit timestamp                 |
| LastUpdateTimestamp       | String     | ISO timestamp                   |

### `UsersTable`

| Column                   | Type       | Description                      |
|--------------------------|------------|----------------------------------|
| UserID                   | String PK  | Unique user ID                   |
| Timestamp                | String SK  | Processing date                  |
| UserOverallCaptionSummary | String    | Summary across all areas         |
| UserOverallScore         | Number     | Cleanliness score (1–5)          |
| ProcessingState          | String     | Should be "processed"            |
| LastUpdateTimestamp      | String     | ISO timestamp                    |

---

## 🧰 AWS Services Used

| Category         | Tool                     | Purpose                                        |
|------------------|--------------------------|------------------------------------------------|
| API Management   | Amazon API Gateway       | Client request entry point                     |
| Compute          | AWS Lambda (x3)          | Serverless execution for processing tasks      |
| Storage          | Amazon S3                | Image upload and retrieval                     |
| Database         | Amazon DynamoDB          | Metadata storage and tracking                  |
| Monitoring       | Amazon CloudWatch        | Logs for debugging and observability           |
| Permissions      | AWS IAM Roles            | Secure service access                          |
| AI / ML          | Vertex AI (Gemini)       | Captioning, summarization, classification      |
|                  | YOLOv8 ONNX via Lambda   | Dirty region detection                         |

---

## 📈 Scoring Strategy

- **Element Score**: (% clean images) × 5
- **Area Score**: Avg. of all element scores
- **User/Building Score**: Avg. of audited areas

---

## 🔐 Security Practices

- IAM roles restrict Lambda access to specific AWS services.
- S3 access controlled by policies.
- Secrets (like Vertex AI keys) are stored securely in env variables or secret managers.

---

## 🏁 Conclusion

This system demonstrates a production-grade cloud-native design for automating public cleanliness audits using cutting-edge AI and serverless compute. It transforms raw image uploads into insightful reports at scale — without managing any backend infrastructure.

---

