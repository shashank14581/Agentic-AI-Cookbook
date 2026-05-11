# Agentic AI Pattern Website

Welcome to this guide on building agentic AI systems!  This site explains how to generate your own API key, how to navigate the repository of scripts, and the mental models behind each agentic pattern.  Whether you're exploring basic conversations or complex retrieval‑augmented agents, this page will help you understand the concepts and files.

## Getting a Gemini API key

To run any of these agents you need an API key for Google Gemini.  Follow these steps to create a key in **Google AI Studio**:

1. **Sign in to AI Studio** — go to [Google AI Studio](https://aistudio.google.com) and log in with your Google account【701672666700928†L105-L156】.  AI Studio is the console where you manage Gemini API access and projects.
2. **Open the API keys page** — once inside, choose **API keys** from the left navigation【701672666700928†L105-L156】.
3. **Create a key** — click **Get API key**.  When prompted, select an existing Google Cloud project or create a new one, then click **Create API key**【701672666700928†L105-L156】.
4. **Copy the key** — AI Studio will generate your Gemini API key; copy it and store it securely, as you’ll need it in your environment variables or configuration【701672666700928†L105-L156】.

With your key in hand, you can interact with the Gemini API from the Python scripts in this repository.

## How to read the scripts

The repository is organized into numbered files (e.g., `1_CreatingTalkingAgents.py`, `2_AgentsWithPersonalityAndMemory.py`) and each demonstrates a specific pattern.  Here’s a general strategy for understanding them:

* **Identify the agent classes.**  Most scripts define a `BaseAgent` or specialized agents that wrap the Gemini API.  The basic version maintains a conversation history and sends prompts to Gemini【955041144366331†L4-L41】.
* **Read the orchestration logic.**  Functions such as `orchestrate` or `run_conversation` coordinate which agent speaks when and handle messaging between agents.  For moderated or handoff patterns, look at the part that decides the next speaker【293824849925005†L104-L138】.
* **Observe memory and prompt construction.**  Some agents assemble a summary of recent exchanges and a set of extracted facts before calling Gemini【811023111516989†L13-L36】.  Look at how each message gets prepended with role names and system prompts to influence the model’s behavior.
* **Examine tool definitions.**  In scripts that use external functions, the tools are declared using JSON schemas and are passed to Gemini when constructing prompts【308481509569095†L31-L56】.  Understanding these declarations will help you know what parameters the agent expects and returns.
* **Check return types.**  For structured responses, the agent might force Gemini to return a JSON object instead of free‑form text【601815852495934†L44-L63】.

## Mental models for each pattern
Below is a high‑level mental model of the patterns demonstrated in the repository.  Use this as a guide when exploring the scripts.

### 1. Basic conversational agents

The first example introduces a `BaseAgent` that wraps the Gemini API and keeps track of previous messages【955041144366331†L4-L41】.  Two agents (a “human” and a “dog”) exchange messages via an `orchestrate` function that simply alternates between them【955041144366331†L56-L72】.  The mental model here is straightforward: each agent constructs a prompt from its memory and calls the language model; control alternates until the conversation ends.

### 2. Personality and memory

Building on the basic agent, this pattern stores a list of facts about each character—such as “I am…” or “I like…”—and extracts facts from previous utterances.  The agent summarizes recent conversation to stay within token limits and prepends the facts to the prompt【811023111516989†L13-L36】.  The mental model is: maintain a short‑term memory and a set of long‑term facts, then use both when generating responses.  The scripts also show how to customize the system prompt to give agents distinct personalities (e.g., a human, a Hindu god)【811023111516989†L72-L84】.

### 3. Moderated conversations

Rather than strictly alternating speakers, a **Moderator** agent decides which participant should speak next.  The moderator inspects the last message and returns a single word indicating the next agent (“Human”, “Dog”, “Philosopher”, “God”)【293824849925005†L104-L138】.  The `orchestrate` function then routes the conversation according to the moderator’s output【293824849925005†L143-L166】.  This pattern introduces conditional control flow: the model’s output directs which agent responds next.

### 4. Tools and structured outputs

Some agents can call external functions (tools).  Each tool is declared as a JSON schema specifying its name, description and parameters【308481509569095†L31-L56】.  When Gemini decides that using a tool is appropriate, it returns a JSON object indicating which function to call and with what arguments.  After the function executes, the agent passes the result back to Gemini.  Another variation forces Gemini to return a structured JSON response describing a conversation (speaker, mood, sentiment, topic and reply)【601815852495934†L44-L63】.  The mental model here is: define tools, present them to the model, call them when needed, and enforce structured outputs.

### 5. Reasoning and hand‑off

In the **handoff** pattern, a reasoning agent returns JSON with two fields: `message` and `handoff_to`.  The `message` contains what the agent wants to say, and `handoff_to` specifies which agent should speak next【976221262491800†L44-L74】.  The orchestrator simply routes messages according to this hand‑off.  This pattern separates reasoning (handled by Gemini) from control (handled by a simple router), allowing the model to plan its next move.

### 6. Supervisor pattern

A **Supervisor** agent coordinates specialized workers.  The supervisor tracks the state of a task (e.g., marketing content) and decides when to call a Planner, Analyst or Copywriter agent【712539935523758†L83-L117】.  Each specialized agent performs its part, and the supervisor integrates their outputs.  Think of this pattern as a simple state machine where the supervisor advances through stages of planning, analysis and writing.

### 7. Parallel and debate patterns

The **parallel** pattern runs multiple analyst agents concurrently (using Python’s `asyncio`).  Each analyst works independently on a part of the problem; the orchestrator then synthesizes their outputs【295666701086031†L80-L99】.  In the **debate** pattern, two debaters present positions, cross‑criticize each other, and a judge delivers a verdict【790854410646844†L74-L142】.  These patterns illustrate fan‑out/fan‑in and adversarial reasoning, respectively.

### 8. Long‑term and shared memory

To simulate long‑term memory, a news‑reading agent stores insights in an SQLite database.  It fetches news articles, decides whether to record an insight (or “SKIP”) and logs it in the database【133667219133308†L118-L129】.  The shared memory pattern uses a single in‑memory store so that multiple agents can read and write each other’s messages【124175675784994†L6-L30】.  The mental model is: persist information across sessions (database) or share a common memory among agents (in‑memory store).

### 9. Knowledge extraction

A specialized agent extracts entities such as people, companies, locations and products from text and writes them to a knowledge store【899284906468114†L23-L45】.  This demonstrates how an agent can transform unstructured data into structured knowledge as it processes information.

### 10. Chunking, embedding and retrieval (RAG)

Some scripts show how to **chunk** and **embed** documents into vectors and perform **vector search** to retrieve relevant context【974069080489756†L9-L27】【974069080489756†L119-L127】.  A **retrieval decision agent** determines whether retrieval is needed and either performs a search or answers directly【163180456072636†L130-L156】【163180456072636†L176-L215】.  The agentic RAG pattern adds this decision layer so that the system only retrieves context when necessary, improving efficiency and response quality.

These mental models should help you understand the design choices in each script.  As you explore the code, try to match the implementation details to the high‑level patterns outlined above.  With a Gemini API key and this guide, you’ll be well on your way to building your own agentic AI applications.
