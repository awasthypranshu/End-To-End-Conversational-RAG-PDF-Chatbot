# 📚 Conversational RAG PDF Chatbot

A **Conversational Retrieval-Augmented Generation (RAG)** application built with **Streamlit and LangChain** that allows users to upload PDF documents and interact with their content through a context-aware chatbot.

Unlike a basic RAG system, this project maintains **chat history**, allowing follow-up questions such as:

> **User:** What is attention?
>
> **Assistant:** Attention is a mechanism that allows a model to focus on relevant parts of an input.
>
> **User:** What is its use?
>
> **Assistant:** It is used to help the model focus on important information and capture relationships between different parts of the input.

The application uses **Hugging Face embeddings**, **Chroma vector storage**, a **history-aware retriever**, and a **Groq-hosted Gemma model** to provide conversational question answering over uploaded PDFs.

---

## 🚀 Features

* 📄 Upload one or multiple PDF documents
* 🔍 Semantic document retrieval using vector embeddings
* 🧠 Chroma vector database for storing document embeddings
* 💬 Context-aware conversational question answering
* 🔄 History-aware query reformulation
* 🗂️ Session-based chat history
* 🤖 Groq LLM integration
* ⚡ Fast responses using Groq inference
* 🖥️ Interactive Streamlit interface
* 🔐 API key input through the UI
* 📚 Ask questions directly from uploaded research papers and documents

---

## 🏗️ Architecture

```text
                    ┌──────────────────┐
                    │    PDF Upload    │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │   PyPDFLoader    │
                    └────────┬─────────┘
                             │
                             ▼
                ┌─────────────────────────┐
                │ Recursive Text Splitter │
                │ chunk_size = 5000       │
                │ overlap = 500           │
                └───────────┬─────────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │ Hugging Face         │
                 │ Embeddings           │
                 │ all-MiniLM-L6-v2     │
                 └──────────┬───────────┘
                            │
                            ▼
                    ┌───────────────┐
                    │    Chroma     │
                    │ Vector Store  │
                    └───────┬───────┘
                            │
                            ▼
                       ┌──────────┐
                       │Retriever │
                       └────┬─────┘
                            │
                            ▼
              ┌─────────────────────────────┐
              │ History-Aware Retriever     │
              │                             │
              │ Chat History + User Query   │
              │            ↓                │
              │ Standalone Question         │
              └──────────────┬──────────────┘
                             │
                             ▼
                   ┌─────────────────┐
                   │ Retrieved       │
                   │ Context         │
                   └────────┬────────┘
                            │
                            ▼
                    ┌───────────────┐
                    │   Groq LLM    │
                    │  Gemma 2 9B   │
                    └───────┬───────┘
                            │
                            ▼
                        ┌───────┐
                        │Answer │
                        └───────┘

                 ┌───────────────────────┐
                 │   Session History    │
                 │   Streamlit State    │
                 └───────────────────────┘
```

---

## 🧠 How It Works

### 1. Upload PDFs

The application allows users to upload multiple PDF documents through Streamlit.

```python
uploaded_files = st.file_uploader(
    "Choose A PDf file",
    type="pdf",
    accept_multiple_files=True
)
```

Each uploaded PDF is temporarily stored and loaded using `PyPDFLoader`.

---

### 2. Document Processing

The extracted PDF content is split into smaller overlapping chunks using:

```python
RecursiveCharacterTextSplitter(
    chunk_size=5000,
    chunk_overlap=500
)
```

This makes the documents easier to search semantically while preserving surrounding context.

---

### 3. Generate Embeddings

The application uses the Hugging Face:

```text
all-MiniLM-L6-v2
```

embedding model to convert document chunks into vector representations.

```python
embeddings = HuggingFaceEmbeddings(
    model_name="all-MiniLM-L6-v2"
)
```

---

### 4. Store Vectors in Chroma

The generated embeddings are stored in a Chroma vector database:

```python
vectorstore = Chroma.from_documents(
    documents=splits,
    embedding=embeddings
)

retriever = vectorstore.as_retriever()
```

When the user asks a question, the retriever searches for the most relevant document chunks.

---

### 5. History-Aware Retrieval

This is the key feature that makes the application conversational.

The system receives:

```text
Chat History
      +
Latest User Question
```

and generates a **standalone question** that can be understood without the previous conversation.

For example:

```text
Previous:
"What is attention?"

Current:
"What is its use?"
```

The history-aware retriever can reformulate the query into something similar to:

```text
"What is the use of attention?"
```

This reformulated question is then sent to the retriever.

The project uses LangChain's:

```python
create_history_aware_retriever()
```

for this process.

---

### 6. Retrieve Relevant Context

The standalone question is used to search the Chroma vector store.

The most relevant document chunks become the context supplied to the LLM.

---

### 7. Generate the Answer

The retrieved context, chat history, and user question are passed to the QA chain.

The system prompt instructs the model to:

* Answer using the retrieved context
* Say that it does not know when the answer is unavailable
* Keep the answer concise
* Limit the response to three sentences

The final answer is generated using the Groq-hosted **Gemma 2 9B** model.

---

## 💬 Conversation Memory

The application maintains session-specific message history using Streamlit's session state:

```python
if 'store' not in st.session_state:
    st.session_state.store = {}
```

Each session receives its own `ChatMessageHistory` instance.

```python
def get_session_history(session: str) -> BaseChatMessageHistory:
    if session_id not in st.session_state.store:
        st.session_state.store[session_id] = ChatMessageHistory()

    return st.session_state.store[session_id]
```

The conversation is connected to the RAG chain using:

```python
RunnableWithMessageHistory(
    rag_chain,
    get_session_history,
    input_messages_key="input",
    history_messages_key="chat_history",
    output_messages_key="answer"
)
```

This allows the chatbot to retain previous user and assistant messages within the session.

---

## 🛠️ Tech Stack

| Technology        | Purpose                         |
| ----------------- | ------------------------------- |
| **Python**        | Core programming language       |
| **Streamlit**     | Web application interface       |
| **LangChain**     | RAG and conversational pipeline |
| **Chroma**        | Vector database                 |
| **Hugging Face**  | Text embeddings                 |
| **Groq**          | LLM inference                   |
| **Gemma 2 9B**    | Language model                  |
| **PyPDFLoader**   | PDF document extraction         |
| **python-dotenv** | Environment variable management |

---

## 📁 Project Structure

```text
PDF-Chatbot/
│
├── app.py
├── requirements.txt
├── .env
├── README.md
│
└── research_papers/
    ├── paper1.pdf
    ├── paper2.pdf
    └── ...
```

> The current Streamlit implementation supports PDF uploads directly through the interface, so a fixed `research_papers` folder is not required for normal application usage.

---

## ⚙️ Installation

### 1. Clone the repository

```bash
git clone https://github.com/your-username/pdf-rag-chatbot.git
cd pdf-rag-chatbot
```

### 2. Create a virtual environment

```bash
python -m venv venv
```

Activate it on Windows:

```bash
venv\Scripts\activate
```

Activate it on Linux/macOS:

```bash
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

---

## 🔑 API Key

The application asks for a **Groq API key** through the Streamlit interface.

You can obtain an API key from the Groq platform.

The application also loads a Hugging Face token from environment variables:

```python
load_dotenv()

os.environ["HF_TOKEN"] = os.getenv("HF_TOKEN")
```

Create a `.env` file:

```env
HF_TOKEN=your_huggingface_token
```

Do **not** commit API keys or tokens to GitHub.

Add this to `.gitignore`:

```gitignore
.env
.venv/
venv/
__pycache__/
*.pyc
```

---

## ▶️ Run the Application

Start Streamlit with:

```bash
streamlit run app.py
```

The application will open in your browser.

Then:

```text
1. Enter your Groq API key
2. Enter a Session ID
3. Upload one or more PDFs
4. Ask questions about the documents
5. Continue asking follow-up questions
```

---

## 🧪 Example Conversation

```text
User:
What is attention?

Assistant:
Attention is a mechanism that allows a neural network to focus on
relevant parts of an input while processing information.

User:
Why is it useful?

Assistant:
Attention is useful because it allows models to capture relationships
between different parts of an input and focus on the most relevant
information.

User:
How is this used in transformers?

Assistant:
...
```

The second and third questions can depend on the previous conversation because the application maintains chat history and uses a history-aware retriever.

---

## 🔄 RAG Pipeline

The complete flow can be summarized as:

```text
PDF
 ↓
Load
 ↓
Split into chunks
 ↓
Generate embeddings
 ↓
Chroma Vector Store
 ↓
User Question
 ↓
Combine with Chat History
 ↓
Reformulate into Standalone Question
 ↓
Retrieve Relevant Documents
 ↓
Inject Retrieved Context
 ↓
Groq / Gemma LLM
 ↓
Final Answer
```

---

## 🎯 What This Project Demonstrates

This project demonstrates practical implementation of:

**Retrieval-Augmented Generation**

Rather than relying only on the model's internal knowledge, the application retrieves relevant information from user-provided documents before generating the answer.

**Conversational RAG**

The system goes beyond basic RAG by using previous messages to understand follow-up questions and reformulate them into standalone queries.

**Vector Search**

Documents are converted into embeddings and stored in Chroma, enabling semantic retrieval rather than simple keyword matching.

**Session-Based Memory**

Conversation history is maintained separately for each session using Streamlit's session state.

**LLM Application Development**

The project combines document loading, embeddings, vector search, retrieval, prompt engineering, memory, and LLM inference into a complete application.

---

## ⚠️ Limitations

* Uploaded PDFs are processed during the Streamlit session.
* The current vector store is created from the uploaded documents during processing.
* Session memory is stored in Streamlit's in-memory session state rather than a persistent external database.
* Responses depend on the quality and relevance of retrieved document chunks.
* The application currently instructs the model to keep answers within three sentences.

---

## 🔮 Future Improvements

* Add persistent vector storage
* Add persistent chat history using PostgreSQL/Redis
* Add document deletion and management
* Display document/page citations with answers
* Add streaming LLM responses
* Add chat UI using `st.chat_message`
* Add configurable retrieval parameters such as `k`
* Support additional document formats such as DOCX and TXT
* Add authentication and multiple user accounts
* Deploy using Streamlit Cloud, Docker, or another cloud platform

---

## 👨‍💻 Author

**Pranshu Awasthy**

Built as a hands-on project to explore **Generative AI, LangChain, RAG, vector databases, conversational memory, and LLM application development**.

---

## ⭐ If You Like This Project

Give the repository a ⭐ on GitHub and feel free to explore, modify, and extend the application.
