# Database Copilot & AI Visualizer

Welcome to the Database Copilot repository! This project features a Hybrid AI Query Router and an Interactive Data Visualizer designed to make complex relational data instantly accessible and analyzable through natural language.

---

## Important Links
* **Project Deployed Link:** [https://hack-latest.onrender.com](https://hack-latest.onrender.com)
* **Video Demonstration:** [Google Drive Link](https://drive.google.com/file/d/136c8UDNVpJMufVGaOpzrkEest5BsZ6OU/view?usp=sharing)

---

## About the Project

Database Copilot addresses the challenge of making complex relational data (like college placements and employee records) accessible to non-technical users. It acts as an intelligent query engine that dynamically routes user requests to the most accurate retrieval method.

**Main Features:**
* **Hybrid Query Routing:** Runs deterministic (NL-to-Query) and semantic (MongoDB Vector RAG) pipelines in parallel, using Gemini 1.5 Flash confidence scores to select the best output.
* **Interactive AI Data Visualizer:** A "Chat-to-Graph" dashboard where data scientists can upload CSVs and generate targeted comparison charts using natural language prompts.
* **Automated Vector Synchronization:** A background loop that instantly regenerates 768-dimensional embeddings upon database modifications, ensuring 0% stale context.
* **Secure Multi-Agent System:** Specialized Security and Validation agents prevent destructive database operations and guarantee query optimization.

---

## Deep Dive: Database Architecture & AI Engine

The core innovation of Database Copilot is how it bridges the gap between structured relational databases and unstructured semantic search. Here is a detailed breakdown of what happens under the hood.

### 1. The Problem with Standard AI Data Access
Typically, AI systems use either **Text-to-SQL/NoSQL** (which is great for exact math, like "Find users with salary > $50k", but fails on vague queries) OR **RAG (Retrieval-Augmented Generation)** (which is great for semantic search like "Tell me about the work culture", but fails at exact aggregations). 

Database Copilot solves this by running a **Hybrid Engine** that executes both, compares them, and returns the mathematically superior answer.

### 2. The Multi-Agent System (Deterministic Route)
When a user inputs a structured query (e.g., "Show me software engineers placed at Google with a CTC over 20 LPA"), the system triggers a swarm of specialized Gemini agents:
* **The Query Agent:** Translates natural language into highly optimized MongoDB Aggregation pipelines. It handles regex for case-insensitive searches and complex `$match` conditions.
* **The Validation Agent:** Evaluates the generated query against the database schema to ensure it will execute correctly. It assigns a **Confidence Score (0.0 to 1.0)** based on how specific and structured the user's prompt is.
* **The Security Agent:** Acts as a strict firewall. It scans the generated MongoDB query to ensure no destructive operations (`$drop`, `$delete`, `$unset`) are executed, preventing prompt-injection attacks.
* **The Update Agent:** If the user issues a data modification command, this agent executes the update and immediately triggers the Vector Synchronization loop.

### 3. Relational RAG & Vector Search (Semantic Route)
If a user asks a vague or conversational question (e.g., "Which students with good grades got into top tech companies?"), standard queries fail. This is where the Relational RAG pipeline takes over.

* **Vector Generation:** The system uses Google's `text-embedding-004` model. Instead of embedding flat text files, the script `create_placement_embeddings.js` joins relational data from three separate MongoDB collections (`students`, `companies`, and `offers`) into a single, rich text block, and generates a **768-dimensional vector embedding** for it.
* **MongoDB Atlas Vector Search:** The user's query is also converted into a vector. Using MongoDB Atlas's `$vectorSearch` stage, the database calculates the **Cosine Similarity** between the query vector and the document vectors. 
* **Dynamic Context Re-joining:** Once the top vector matches are found, the system uses MongoDB `$lookup` stages to dynamically re-fetch the exact, up-to-date relational data (Student Name, CGPA, Company Sector, etc.) to pass as pure context to the LLM.
* **The Gemini Brain:** Gemini 2.0 Flash receives this strictly scoped context and synthesizes a human-readable, conversational answer. The Cosine Similarity metric serves as the **RAG Confidence Score**.

### 4. The Routing Algorithm (The Copilot)
The Node.js backend acts as the ultimate decider. It compares the **Validation Agent's Score** against the **RAG Cosine Similarity Score**. 
* If the user asks for exact numbers/tables $\rightarrow$ Agent score wins $\rightarrow$ Returns raw JSON data.
* If the user asks for summaries/descriptions $\rightarrow$ RAG score wins $\rightarrow$ Returns a conversational response.

### 5. Automated Vector Synchronization
In traditional RAG, when a database is updated, the vector embeddings become stale, causing hallucinations. Database Copilot features a background synchronization loop. Whenever the Update Agent alters a document in the database, the system instantly recalculates and overwrites the `docEmbedding` array, ensuring **0% stale context**.

---

## Project Flow

1. **User Input:** The user asks a question or uploads a CSV.
2. **Parallel Execution:** The backend generates a query embedding and runs both the Vector Search and the Multi-Agent pipeline simultaneously.
3. **Score Comparison:** The router compares the Cosine Similarity Score vs. Agent Validation Score.
4. **Data Retrieval:** The winning pipeline extracts the data from MongoDB Atlas.
5. **Visualization/Response:** Gemini synthesizes the final JSON for chart rendering (in the Visualizer) or text for the conversational interface.

---

## Setup Instructions

Follow these steps to set up and run the project locally:

### 1. Clone the Repository

```sh
git clone [https://github.com/your-username/Database-Copilot.git](https://github.com/your-username/Database-Copilot.git)
cd Database-Copilot

```

### 2. Install Dependencies

```sh
npm install

```

### 3. Configure Environment Variables

Create a `.env` file in the project root and add your configuration details.

```env
PORT=3002
MONGODB_URI=mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/placements?retryWrites=true&w=majority
GEMINI_API_KEY=your_gemini_api_key_here

```

### 4. Initialize the Database Vectors

Before running the main server, execute the embedding script to generate vector embeddings for your existing MongoDB relational data.

```sh
node create_placement_embeddings.js

```

### 5. Start the Application

```sh
npm run dev
# OR
node placement_backend.js

```

The server will start running at `http://localhost:3002`.

---

## Usage

* **API Endpoint:** Send POST requests to `/api/rag-placements-answer` or `/api/hybrid-query` with a JSON body containing `"userInput"`.
* **Visualizer:** Upload a CSV via the frontend dashboard and use the chatbox to request specific data plots (e.g., "Show me a comparison chart for X and Y").

---

---

**Contact:** For any questions, feel free to open an issue or reach out to me directly.
