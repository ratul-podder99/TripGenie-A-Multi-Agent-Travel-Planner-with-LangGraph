# 🧞 TripGenie — A Multi-Agent Travel Planner with LangGraph

TripGenie is an AI-powered travel planning assistant that turns a single natural-language trip request into a complete, structured travel plan — flight options, hotel suggestions, a day-by-day itinerary, and an estimated budget — using a **multi-agent workflow orchestrated with LangGraph**, served through a **FastAPI** backend.

Instead of one generic LLM call, TripGenie breaks trip planning into specialized agents that each do one job well, then hands the combined results to a final agent that writes a polished, human-readable travel plan.

---

## ✨ Features

- **Multi-agent orchestration** — dedicated agents for flights, hotels, itinerary building, and final response composition, coordinated as a LangGraph `StateGraph`.
- **Live flight search** — a custom flight tool resolves cities/countries into IATA airport codes (via `airportsdata` and `pycountry`) and queries flight data.
- **Web-grounded hotel & travel info** — uses the **Tavily Search API** to pull up-to-date hotel and destination information instead of relying purely on model knowledge.
- **Conversational memory** — trip planning sessions are persisted using a **PostgreSQL-backed LangGraph checkpointer**, so a `thread_id` can be reused to continue a conversation.
- **FastAPI web app** — a lightweight HTML/JS frontend (Jinja2 templates + static assets) talks to a JSON API.
- **Groq-powered LLM** — uses `llama-3.3-70b-versatile` via `langchain-groq` for fast, low-cost inference.

---

## 🧠 How It Works

TripGenie models the trip-planning process as a directed graph of agents, each mutating a shared `TravelState`:

```
        START
          │
          ▼
   ┌───────────────┐
   │ Flight Agent  │  → searches flights for the query
   └───────┬───────┘
           ▼
   ┌───────────────┐
   │ Hotel Agent   │  → searches hotels via Tavily
   └───────┬───────┘
           ▼
   ┌───────────────┐
   │Itinerary Agent│  → LLM drafts a day-by-day plan
   └───────┬───────┘
           ▼
   ┌───────────────┐
   │ Final Agent   │  → LLM composes the final formatted answer
   └───────┬───────┘
           ▼
          END
```

Each node updates a shared state object (`messages`, `user_query`, `flight_results`, `hotel_results`, `itinerary`, `llm_calls`), and the final response is formatted into six sections: **Trip Summary, Flight Information, Hotel Suggestions, Day-by-Day Itinerary, Estimated Budget, and Final Recommendations.**

Conversation state is checkpointed to PostgreSQL via `langgraph-checkpoint-postgres`, keyed by a `thread_id`, so a user can continue refining a plan across multiple requests.

---

## 🏗️ Tech Stack

| Layer              | Technology                                          |
|---------------------|------------------------------------------------------|
| Agent orchestration  | [LangGraph](https://github.com/langchain-ai/langgraph) |
| LLM framework        | LangChain + `langchain-groq`                         |
| LLM provider         | Groq (`llama-3.3-70b-versatile`)                     |
| Web search           | Tavily API (`langchain-tavily`, `tavily-python`)     |
| Backend API          | FastAPI + Uvicorn                                    |
| Frontend             | Jinja2 templates + static HTML/CSS/JS                |
| Persistence          | PostgreSQL (via `psycopg`, `langgraph-checkpoint-postgres`) |
| Airport/country data | `airportsdata`, `pycountry`                          |
| Config               | `python-dotenv`                                      |

---

## 📁 Project Structure

```
TripGenie-A-Multi-Agent-Travel-Planner-with-LangGraph/
├── app.py                  # FastAPI app: routes, request/response handling
├── backend.py              # LangGraph state graph, agents, and PostgreSQL checkpointer
├── tools/                  # Agent tools (flight search, Tavily web search)
├── static/                 # Frontend static assets (CSS/JS)
├── templates/               # Jinja2 HTML templates (index.html, etc.)
├── requirements.txt         # Python dependencies
└── .gitignore
```

---

## ✅ Prerequisites

- Python 3.10+
- A PostgreSQL database (e.g. a free instance on [Render](https://render.com), [Supabase](https://supabase.com), or local Postgres)
- API keys:
  - [Groq API key](https://console.groq.com/) (LLM inference)
  - [Tavily API key](https://tavily.com/) (web search)

---

## ⚙️ Installation

1. **Clone the repository**

   ```bash
   git clone https://github.com/ratul-podder99/TripGenie-A-Multi-Agent-Travel-Planner-with-LangGraph.git
   cd TripGenie-A-Multi-Agent-Travel-Planner-with-LangGraph
   ```

2. **Create and activate a virtual environment**

   ```bash
   python -m venv venv
   source venv/bin/activate      # On Windows: venv\Scripts\activate
   ```

3. **Install dependencies**

   ```bash
   pip install -r requirements.txt
   ```

4. **Configure environment variables**

   Create a `.env` file in the project root:

   ```env
   GROQ_API_KEY=your_groq_api_key
   TAVILY_API_KEY=your_tavily_api_key
   DATABASE_URL=postgresql://user:password@host:5432/dbname
   ```
   > `DATABASE_URL` is required — the app raises an error on startup if it's missing. If your connection string doesn't include `sslmode`, the app automatically appends `sslmode=require`.

5. **Run the application**

   ```bash
   python app.py
   ```
   or
   ```bash
   uvicorn app:app --reload
   ```

   The app will be available at **http://127.0.0.1:8000**.

---

## 🔌 API Reference

### `GET /`
Renders the web UI (`templates/index.html`).

### `POST /api/travel`
Runs the multi-agent travel planning workflow.

**Request body:**
```json
{
  "message": "Plan a 5-day trip to Cox's Bazar from Dhaka on a mid-range budget",
  "thread_id": null
}
```

**Response:**
```json
{
  "success": true,
  "thread_id": "user_xxxxxxxx",
  "answer": "Full formatted travel plan...",
  "flight_results": "...",
  "hotel_results": "...",
  "itinerary": "...",
  "llm_calls": 4
}
```

Pass the returned `thread_id` back in on the next call to continue the same conversation with retained context.

### `GET /health`
Simple health-check endpoint returning `{"status": "ok"}`.

---

## 🗺️ Roadmap Ideas

- Add authentication and rate limiting for the public API
- Cache repeat flight/hotel queries to reduce API usage
- Support multi-city and round-trip itineraries
- Add a supervisor/routing agent for conditional tool selection instead of a fixed linear graph
- Return structured (typed/Pydantic) itinerary data in addition to the formatted text answer

---

## 🤝 Contributing

Contributions, issues, and feature requests are welcome. Feel free to fork the repo and open a pull request.

## 📄 License

No license file is currently included in this repository — check with the repository owner before reuse or redistribution.