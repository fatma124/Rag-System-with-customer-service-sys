# Customer Service Telegram Bot with Automated Knowledge Base (RAG)

An automated **n8n workflow** that ingests knowledge base documents (e.g., Return Policies, FAQs) from Google Drive into a **Pinecone vector database** and uses a **Google Gemini-powered AI Agent** to automatically respond to customer inquiries on **Telegram** using Retrieval-Augmented Generation (RAG).

---

## 📐 Architecture & Workflow Pipeline

This n8n project is divided into two primary execution pipelines:

```
[ Google Drive Folder ] ──> [ Download File ] ──> [ Gemini Embeddings ] ──> [ Pinecone Vector Index ]
                                                                                   │
                                                                                   ▼
[ Telegram User Query ] ──> [ AI Agent + Memory ] <────────────────────────────────┘
                                    │
                                    ▼
                        [ Telegram Bot Response ]
```

### 1. Document Ingestion Pipeline (ETL / Knowledge Indexing)
* **Google Drive Trigger**: Polls a specific Google Drive folder every minute (`folderId: 1Y-syA3s5f9D_mM0f7163j-3ELE6M3k9t`) for newly uploaded documents (e.g., `Return Policy.pdf`).
* **Download File**: Downloads the file content from Google Drive as binary data.
* **Default Data Loader**: Parses and extracts textual metadata and content from the document binary.
* **Google Gemini Embeddings**: Converts the document text into dense vector embeddings.
* **Pinecone Vector Store (`rag-ai`)**: Upserts document vector embeddings into the `rag-ai` Pinecone index.

### 2. Customer Service Conversational Pipeline (RAG Agent)
* **Telegram Trigger**: Triggers upon receiving incoming messages from Telegram users.
* **AI Agent (Google Gemini)**: Formulates accurate, grounded responses based on system prompt rules.
* **Simple Memory (Window Buffer)**: Retains conversation context per Telegram `chat.id`.
* **Pinecone Retrieval Tool**: Enables the AI Agent to query the `rag-ai` vector store before answering any user question to prevent hallucination.
* **Send Telegram Message**: Delivers the generated AI response back to the user on Telegram.

---

## 🛠️ Required Credentials & Integrations

To run this workflow, ensure the following credentials and tools are set up in your n8n environment:

| Service / Tool | Credential Type | Description |
| :--- | :--- | :--- |
| **Google Drive** | `googleDriveOAuth2Api` | OAuth2 authentication to monitor and download files. |
| **Google Gemini API** | `googlePalmApi` | API Key for Gemini Chat Model and Embedding models. |
| **Pinecone** | `pineconeApi` | API Key with access to the `rag-ai` index. |
| **Telegram Bot** | `telegramApi` | Bot Token obtained from [@BotFather](https://t.me/BotFather). |

---

## 🚀 Setup & Installation Instructions

### Step 1: Import Workflow to n8n
1. Open your n8n canvas.
2. Select **Workflows** > **Import from File** or **Paste from Clipboard**.
3. Paste the workflow JSON and save.

### Step 2: Configure Vector Store Index
1. Log in to your [Pinecone Console](https://app.pinecone.io/).
2. Create an index named **`rag-ai`** with the dimension size corresponding to Google Gemini Embeddings (typically `768` dimensions).

### Step 3: Configure Node Credentials & Settings
1. **Google Drive Trigger & Download**:
   * Re-link your `googleDriveOAuth2Api` credential.
   * Update the `folderToWatch` ID to target your specific Google Drive folder.
2. **Pinecone Nodes**:
   * Link your `pineconeApi` credential.
   * Ensure index name is set to `rag-ai`.
3. **Google Gemini Nodes**:
   * Re-link your `googlePalmApi` credential.
4. **Telegram Nodes**:
   * Connect your Telegram Bot token via `telegramApi`.

### Step 4: Activate Workflow
1. Click the **Active** toggle in top right corner of the n8n interface.

---

## 💻 How It Works in Production

1. **Adding Knowledge**: Whenever you add or update PDF documents (like company policies or manuals) to your watched Google Drive folder, n8n automatically embeds and indexes them into Pinecone.
2. **Asking Questions**: A customer sends a message to your Telegram Bot (e.g., *"What is your return policy?"*).
3. **Grounding & Answering**: The AI Agent queries Pinecone for relevant chunks from your document, evaluates the context with Gemini, and replies with a precise answer directly in the Telegram chat.

---

## 📄 License
This workflow project is open-source and customizable for enterprise customer support workflows.
