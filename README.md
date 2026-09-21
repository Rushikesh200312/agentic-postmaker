# agentic-postmaker
# Autonomous Social Media Strategy Agent (Postmaker)

**Postmaker** is an autonomous agentic workflow built with **LangGraph**, **LangChain**, and **ChatGroq**. It ingests article content from an arbitrary web URL, drafts a synchronized multi-platform social media campaign (Twitter/X thread and LinkedIn post), and iteratively self-corrects using automated guardrail tools until all platform-specific formatting and character constraints pass.

## 🎯 Key Features

* **Automated Web Extraction:** Scrapes and isolates readable article bodies with BeautifulSoup, removing boilerplate overhead (scripts, navbars, headers, footers).
* **High-Throughput Reasoning Engine:** Powered by Groq's low-latency inference utilizing `openai/gpt-oss-safeguard-20b`.
* **Deterministic Tool Calling:** Employs explicit parameter bindings to guarantee tool invocation integrity.
* **Automated Validation Guardrails:** Programmatically validates tweet length limits ($\le 280$ characters) and minimum hashtag thresholds ($\ge 3$ hashtags).
* **Cyclic Agentic Reflection:** Leverages LangGraph's feedback loop to re-feed validation error messages directly to the LLM for automated revisions before delivering the final output.

## 🏗️ Architecture & Workflow

```
       +-----------------------+
       |         START         |
       +-----------+-----------+
                   |
                   v
       +-----------+-----------+
+----->|      Agent Node       |<-----+
|      |    (ChatGroq LLM)     |      |
|      +-----------+-----------+      |
|                  |                  |
|          (tools_condition)          |
|         /                 \         |
|    Tool Call               No Tools |
|       v                     v       |
|  +----+------+          +---+---+   |
|  | Tool Node |          |  END  |   |
|  +----+------+          +-------+   |
|       |                             |
+-------+-----------------------------+
   (Observation / Reflection Loop)
```

1. **Start:** Target URL and instructions are sent via `SystemMessage` and `HumanMessage`.
2. **Fetch Content:** Model calls `fetch_webpage_content` to parse clean text from the target link.
3. **Draft Campaign:** Model writes a 3-tweet thread and a LinkedIn post.
4. **Validation:** Draft is routed through `validate_social_draft`.
5. **Self-Correction:**
   * If validation **FAILS**, failure diagnostics route back into `messages`, prompting an immediate rewrite.
   * If validation **PASSES**, the agent outputs the finalized campaign and terminates at `END`.

## 📦 Prerequisites & Installation

### 1. Clone the Repository

```bash
git clone https://github.com/<your-username>/postmaker.git
cd postmaker
```

### 2. Create and Activate a Virtual Environment

```bash
# macOS/Linux
python -m venv venv
source venv/bin/activate

# Windows
python -m venv venv
venv\Scripts\activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

If creating `requirements.txt` manually:

```text
langchain
langchain-core
langchain-groq
langgraph
requests
beautifulsoup4
python-dotenv
```

## 🔑 Environment Configuration

Create a `.env` file in the root directory:

```env
GROQ_API_KEY=your_groq_api_key_here
```

> **Note:** If `GROQ_API_KEY` is not present in `.env`, the script will automatically prompt you securely via `getpass`.

## 🚀 Running the Notebook

1. Launch Jupyter Notebook or JupyterLab:
   ```bash
   jupyter notebook
   ```
2. Open `postmaker.ipynb`.
3. Set your desired target URL in Cell 4:
   ```python
   target_url = "https://en.wikipedia.org/wiki/Gut_microbiota"
   ```
4. Run all cells sequentially. The final deliverable will print at the bottom of the last cell.

## 🛠️ Tool Specifications

| Tool Name | Type | Purpose |
| :--- | :--- | :--- |
| `fetch_webpage_content` | Extraction | Requests the page, strips non-body tags, and returns the first 8,000 characters of readable text. |
| `validate_social_draft` | Guardrail | Validates that all tweets stay under 280 characters and verifies that the draft includes at least 3 hashtags. |

## 🛡️ Guardrails & Failsafes

* **Context Window Protection:** Ingested webpage text is bounded to 8,000 characters to prevent prompt bloat.
* **Deterministic Recursion Cap:** LangGraph execution is capped using `recursion_limit: 12` to prevent infinite loops during self-correction.
* **Low Temperature Setting:** `temperature=0.2` ensures schema adherence during tool dispatch.
