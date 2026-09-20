# Hello Agent — CSV FAQ Agent 🤖

A lightweight Agentic AI application that allows users to upload multiple CSV files and ask questions about their data using natural language.

The application uses **Python, Pandas, LangChain, OpenAI, and Streamlit** to create an AI-powered interface over structured business data.

> **Project:** Week 0 Mini Project — Applied Agentic AI for SWEs  
> **Purpose:** Warm-up project for understanding the fundamentals of AI agents working with tabular data.

---

## 📌 Overview

Many business teams maintain frequently accessed information in structured files such as CSVs:

- E-commerce FAQs
- Credit card terms and conditions
- Hospital policies
- SaaS product documentation

Support and operations teams often need to search these files manually to answer customer questions.

**Hello Agent** provides a simple natural-language interface over these datasets.

Instead of searching through rows manually, a user can upload one or more CSV files and ask questions such as:

> "What is the return policy for electronics?"

or

> "What is the API rate limit for the free plan?"

The application uses a LangChain Pandas DataFrame Agent to analyze the uploaded datasets and generate an answer based on the available data.

---

## 🎯 Project Goals

The application was designed around five core requirements:

1. **Upload CSV files**
   - Support one or multiple CSV files.
   - Load each file into a Pandas DataFrame.
   - Preview uploaded data.

2. **Ask questions in natural language**
   - Users can ask questions without knowing the structure of the CSV.

3. **Answer using the uploaded data**
   - The AI agent analyzes the DataFrames to identify relevant information.
   - Answers are generated using values contained in the uploaded datasets.

4. **Prevent unsupported answers**
   - The system prompt instructs the agent not to rely on general knowledge.
   - When information cannot be found in the uploaded data, the intended behavior is to clearly indicate that the information is unavailable.

5. **Keep the interface simple**
   - Streamlit provides a lightweight UI for uploading files and asking questions.

---

## 🏗️ Architecture

The application follows a simple pipeline:

```text
                    ┌──────────────────┐
                    │     User         │
                    │                  │
                    │ Upload CSV files │
                    │ Ask a question   │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │    Streamlit     │
                    │      UI          │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │      Pandas      │
                    │                  │
                    │ CSV → DataFrame  │
                    └────────┬─────────┘
                             │
                             ▼
              ┌─────────────────────────────┐
              │  LangChain DataFrame Agent  │
              │                             │
              │  • Selects relevant data   │
              │  • Analyzes DataFrames     │
              │  • Executes data operations │
              └──────────────┬──────────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │   OpenAI Model   │
                    │   gpt-4o-mini    │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │     Answer       │
                    │                  │
                    │ Clear response   │
                    │ based on data    │
                    └──────────────────┘
```

---

## 🧰 Technology Stack

| Technology | Purpose |
|---|---|
| **Python** | Application and agent logic |
| **Streamlit** | Web-based user interface |
| **Pandas** | Loading and manipulating CSV data |
| **LangChain** | Agent orchestration |
| **LangChain Experimental** | Pandas DataFrame Agent |
| **OpenAI** | Large language model |
| **gpt-4o-mini** | Chat model used by the agent |
| **gdown** | Downloading the sample datasets |

---

## 📂 Sample Datasets

The project uses four example datasets representing different business domains:

```text
data/
├── ecommerce_faqs.csv
├── credit_card_terms.csv
├── hospital_policy.csv
└── saas_docs.csv
```

The datasets represent:

### 🛒 E-commerce FAQs

Frequently asked questions about the online store, including topics such as returns and warranties.

### 💳 Credit Card Terms

Information related to credit card terms and conditions.

### 🏥 Hospital Policies

Information about hospital policies such as visiting hours, records, and payments.

### ☁️ SaaS Documentation

Product documentation covering features, limits, and support information.

---

## 🔑 Core Agent Configuration

The application initializes an OpenAI chat model with a low temperature:

```python
llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.0,
    api_key=api_key
)
```

A low temperature helps make responses more deterministic and predictable, which is desirable for a business FAQ application.

The Pandas DataFrame agent is then created over the uploaded DataFrames:

```python
agent = create_pandas_dataframe_agent(
    llm,
    dataframes,
    verbose=True,
    agent_type="openai-functions",
    allow_dangerous_code=True
)
```

The user's question is combined with the application instructions before being passed to the agent:

```python
final_query = system_prompt + "\n\nQuestion: " + user_input

response = agent.invoke(final_query)["output"]
```

---

## 🧠 System Prompt

The application uses a system prompt to establish the expected behavior of the agent:

```text
You are a smart data assistant capable of reading multiple CSV files.

- You have access to 4 different datasets: SaaS Docs, Credit Card Terms,
  Hospital Policy, and Ecommerce FAQs.
- When asked a question, determine which DataFrame is most relevant.
- Do NOT answer from general knowledge.
- Answer in plain English.
```

This demonstrates an important Agentic AI concept:

> **The LLM is not simply being asked a question. Its behavior is being constrained by instructions and tools that allow it to work with external data.**

---

## 🔄 How the Agent Works

A typical request follows this flow:

### 1. User uploads CSV files

The application receives one or more CSV files.

### 2. CSV files become DataFrames

Pandas loads each CSV:

```python
df = pd.read_csv(file)
```

The resulting DataFrames are stored and provided to the agent.

### 3. User asks a natural-language question

For example:

```text
What is the API rate limit for the free plan?
```

### 4. Agent determines which data is relevant

The agent has access to the uploaded DataFrames and can work with their contents.

For example:

```text
Question
   ↓
SaaS DataFrame is relevant
   ↓
Inspect relevant rows/columns
   ↓
Extract required information
```

### 5. OpenAI generates the final response

The model converts the information found in the data into a clear natural-language response.

---

## 💡 Example Questions

The application can be used with questions such as:

```text
What is the return policy for electronics?

What does the extended warranty cover?

What are the visiting hours in the hospital?

What is the API rate limit for the free plan?
```

It can also be experimented with using questions that require working with numeric values or multiple rows.

---

## 🚫 Data-Only Rule

One of the key requirements of the project is that the agent should answer from the uploaded data rather than relying on general world knowledge.

The intended behavior is:

```text
User Question
      │
      ▼
Can the answer be found in uploaded data?
      │
   ┌──┴──┐
   │     │
  Yes    No
   │     │
   ▼     ▼
Answer   Clearly state that
from     the information
the data  was not found
```

For example:

```text
I could not find this information in the uploaded files.
```

This is an important principle when building AI systems for business use:

**An AI assistant should have clearly defined boundaries around the data it is expected to use.**

---

## 🚀 Running the Project

### Prerequisites

You will need:

- Python 3.x
- An OpenAI API key
- Internet connectivity
- The required Python packages

### Install Dependencies

```bash
pip install langchain-experimental
pip install langchain-openai
pip install pandas
pip install streamlit
pip install gdown
```

Or install them together:

```bash
pip install langchain-experimental langchain-openai pandas streamlit gdown
```

### Configure the API Key

Set your OpenAI API key as an environment variable:

```bash
export OPENAI_API_KEY="your-api-key"
```

On Windows:

```powershell
setx OPENAI_API_KEY "your-api-key"
```

Alternatively, the notebook/application can obtain the key interactively.

**Do not commit your API key to GitHub.**

---

## ▶️ Running the Notebook

The project can be explored interactively using the provided Jupyter notebook.

Open:

```text
Hello_Agent_CSV_FAQ_Agent.ipynb
```

Run the cells in order, provide your API key when prompted, load the datasets, and interact with the agent.

---

## 🌐 Running as a Streamlit Application

The intended application interface can be implemented using Streamlit.

Run:

```bash
streamlit run app.py
```

The application provides a simple workflow:

```text
Upload CSV files
       ↓
Preview uploaded data
       ↓
Enter question
       ↓
Run AI agent
       ↓
Display answer
```

---

## 🔐 Security Considerations

This project is intentionally small and is designed as a learning exercise.

The Pandas DataFrame Agent requires:

```python
allow_dangerous_code=True
```

This capability should be treated carefully in a production application because the agent can execute Python-based operations while working with the DataFrames.

For a production system, additional controls would be appropriate, such as:

- Sandboxed execution
- Strict tool permissions
- Input validation
- File-type and file-size validation
- Authentication and authorization
- Audit logging
- Rate limiting
- Secrets management
- Restricted execution environments
- Monitoring and observability

Therefore, this implementation should be considered a **learning/demo application rather than a production-ready security architecture**.

---

## 🧪 What I Learned

This project provided a practical introduction to several Agentic AI concepts.

### 1. LLM + Tools

An LLM becomes more useful when it can interact with external tools and data.

In this project:

```text
LLM
 +
Pandas DataFrames
 +
LangChain Agent
 =
Data-aware AI assistant
```

### 2. Agents vs. Simple LLM Calls

A traditional LLM call might look like:

```text
Question → LLM → Answer
```

The DataFrame agent introduces an additional reasoning/tool-use layer:

```text
Question
   ↓
Agent
   ↓
Determine what data operation is needed
   ↓
Interact with DataFrame
   ↓
Use result
   ↓
Generate answer
```

### 3. Prompt Engineering

The system prompt establishes behavioral constraints such as:

- Which datasets are available
- How the agent should use them
- Not relying on general knowledge
- Returning clear English responses

### 4. Structured Data + Natural Language

The project demonstrates how users can interact with structured business data without knowing SQL or Pandas.

Instead of:

```sql
SELECT ...
FROM ...
WHERE ...
```

the user can ask:

```text
What is the API rate limit for the free plan?
```

### 5. Business Requirement → Technical Design

A simple business requirement:

> "Support agents need faster access to FAQ and policy information."

can be translated into:

```text
CSV files
   ↓
Pandas
   ↓
DataFrame Agent
   ↓
LLM
   ↓
Natural-language answer
```

This is an important pattern when designing AI-powered business applications.

---

## 🔮 Potential Future Improvements

This project intentionally keeps the architecture simple. A production-oriented version could introduce several improvements.

### Data & Retrieval

- Add semantic search for larger datasets
- Introduce embeddings and a vector database
- Support PDFs, Word documents, and web content
- Add document metadata and source references
- Implement hybrid keyword + semantic retrieval

### Agent Improvements

- Add specialized tools for different data sources
- Introduce a routing agent
- Add structured output
- Add validation of generated answers
- Add confidence/source indicators

### Application Improvements

- User authentication
- Conversation history
- File management
- Better error handling
- Streaming responses
- Chat-based UI
- Upload validation

### Production Engineering

- Containerize with Docker
- Deploy to AWS/Azure/GCP
- Add CI/CD
- Add automated tests
- Add logging and monitoring
- Add metrics for latency, token usage, and failures
- Add rate limiting and cost controls

---

## 🗺️ Evolution Toward a Production Agent

The current project intentionally uses a simple architecture:

```text
CSV
 ↓
Pandas
 ↓
LangChain DataFrame Agent
 ↓
LLM
```

A more scalable enterprise architecture could evolve toward:

```text
                    ┌─────────────────┐
                    │   User / UI     │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │  Agent / Router │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
        ┌──────────┐   ┌──────────┐   ┌──────────┐
        │ FAQ Tool │   │ Policy   │   │ Product  │
        │          │   │ Tool     │   │ Tool     │
        └──────────┘   └──────────┘   └──────────┘
              │              │              │
              └──────────────┼──────────────┘
                             ▼
                    ┌─────────────────┐
                    │ Retrieval/Data  │
                    │ Layer           │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │       LLM       │
                    └─────────────────┘
```

This would allow the simple CSV FAQ agent to become the foundation for a more sophisticated enterprise AI assistant.

---

## 📁 Project Structure

A possible repository structure is:

```text
hello-agent-csv-faq/
│
├── README.md
├── Hello_Agent_CSV_FAQ_Agent.ipynb
├── app.py
├── data/
│   ├── ecommerce_faqs.csv
│   ├── credit_card_terms.csv
│   ├── hospital_policy.csv
│   └── saas_docs.csv
│
└── requirements.txt
```

---

## 📚 Key Concepts Demonstrated

- Agentic AI
- LLM tool usage
- LangChain
- Pandas DataFrame agents
- Prompt engineering
- Structured data analysis
- Natural-language interfaces
- Streamlit
- Business-focused AI applications
- Data-grounded responses
- AI safety and execution boundaries

---

## ⚠️ Disclaimer

This repository is a **learning project** created as part of an Applied Agentic AI learning exercise.

It is intentionally simplified and should not be considered a production-ready implementation for handling sensitive business, financial, healthcare, or customer data.

---

## 👩‍💻 Author

**Neha Kakkar**

Software Engineer | Distributed Systems | AI & Agentic Engineering

This project is part of my hands-on learning journey into building practical **Agentic AI systems for software engineering applications**.
