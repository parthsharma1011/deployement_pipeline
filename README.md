# LinkedIn Post Generator — Multi-Agent AI

An AI-powered LinkedIn post generator that uses a multi-agent pipeline to research, write, and validate professional LinkedIn content.

---

## How It Works

The app runs three CrewAI agents in sequence:

```
Research Agent → Writer Agent → Validator Agent
```

| Agent | Role |
|---|---|
| **Research Agent** | Searches the web for recent trends, statistics, and angles on your topic using Tavily |
| **Writer Agent** | Uses research insights to write an engaging LinkedIn post following best practices |
| **Validator Agent** | Fact-checks the post, scores it 1–10, and suggests improvements |

---

## Tech Stack

- **[CrewAI](https://www.crewai.com/)** — multi-agent orchestration
- **[Gemini 2.0 Flash](https://deepmind.google/technologies/gemini/)** — LLM powering all three agents
- **[Tavily](https://tavily.com/)** — real-time web search for research
- **[Gradio](https://www.gradio.app/)** — web UI
- **Docker** — containerised deployment
- **Render** — cloud hosting

---

## Features

- Choose your **tone**: Professional, Casual, Thought-Leader
- Choose your **post type**: Story, Hot-Take, Announcement, Lesson-Learned, Thought-Leader
- Real-time web research on your topic before writing
- Validation score + improvement suggestions on every post
- Follows LinkedIn best practices (hook, short paragraphs, hashtags at the bottom, no cringe phrases)

---

## Local Setup

### 1. Clone the repo

```bash
git clone https://github.com/parthsharma1011/test_deployment.git
cd test_deployment
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Set environment variables

Copy `.env.example` to `.env` and fill in your keys:

```bash
cp .env.example .env
```

```env
GEMINI_API_KEY=your_gemini_api_key_here
TAVILY_API_KEY=your_tavily_api_key_here
```

- Get a Gemini API key at [aistudio.google.com](https://aistudio.google.com)
- Get a Tavily API key at [tavily.com](https://tavily.com)

### 4. Run the app

```bash
python gradio_app.py
```

Open `http://localhost:7860` in your browser.

---

## Docker

```bash
docker build -t linkedin-generator .
docker run -p 7860:7860 \
  -e GEMINI_API_KEY=your_key \
  -e TAVILY_API_KEY=your_key \
  linkedin-generator
```

---

## Deployment (Render)

This project deploys automatically to [Render](https://render.com) via GitHub Actions.

**Pipeline:**
```
git push to main → GitHub Actions → triggers Render deploy hook → Render rebuilds Docker image → live
```

**To set up:**
1. Create a Web Service on Render and connect this repo
2. Add `GEMINI_API_KEY` and `TAVILY_API_KEY` as environment variables on Render
3. Copy the Deploy Hook URL from Render → Settings
4. Add it as a GitHub secret named `RENDER_DEPLOY_HOOK_URL`

---

## Project Structure

```
.
├── gradio_app.py          # Gradio web UI
├── linkedin_generator.py  # Core multi-agent logic
├── requirements.txt       # Python dependencies
├── Dockerfile             # Container config
├── render.yaml            # Render service config
├── .env.example           # Environment variable template
├── .github/
│   └── workflows/
│       └── deploy.yml     # GitHub Actions CI/CD
└── .gitignore
```

---

## Environment Variables

| Variable | Description |
|---|---|
| `GEMINI_API_KEY` | Google Gemini API key |
| `TAVILY_API_KEY` | Tavily search API key |
| `PORT` | Port to run on (default: 7860, set automatically by Render) |
