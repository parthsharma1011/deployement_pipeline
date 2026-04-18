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

## Deployment Architecture

I wanted a deployment setup that was clean, automated, and didn't require me to manually trigger anything after a code push. Here's the architecture I landed on and why I made each decision.

### The Pipeline

```
Local Machine
     │
     │  git push origin main
     ▼
GitHub Repository
     │
     │  triggers on push to main
     ▼
GitHub Actions (CI/CD)
     │
     │  curl POST to Render deploy hook
     ▼
Render Cloud
     │
     │  pulls latest code, rebuilds Docker image
     ▼
Live App — https://test-deployment-cfby.onrender.com
```

### Why This Stack?

**Docker** — I containerised the app so that the environment is consistent everywhere. Locally, in CI, and on Render it all runs the same. No more "works on my machine" issues. The `Dockerfile` uses `python:3.11-slim` to keep the image small and installs only what's in `requirements.txt`.

**GitHub Actions** — Acts as the middle layer between my code and the deployment. Every push to `main` triggers the workflow in `.github/workflows/deploy.yml`. Right now it does one thing — fire the Render deploy hook — but this is where I'd add tests, linting, or any other checks before a deploy goes out.

**Render** — I chose Render over alternatives because it natively supports Docker-based deployments with zero config. Point it at a repo, give it a `Dockerfile`, set your env vars, and it handles the rest. It also auto-assigns HTTPS and manages the port through the `PORT` environment variable, which the app reads at startup.

**Deploy Hook** — Instead of giving GitHub Actions full Render API access, I use a scoped deploy hook URL. It's a single-purpose URL that only triggers a redeploy of this specific service. It lives as a GitHub Actions secret (`RENDER_DEPLOY_HOOK_URL`) so it's never exposed in the code or logs.

### How the Pieces Connect

1. I push code to the `main` branch on GitHub
2. GitHub Actions picks it up immediately and runs `.github/workflows/deploy.yml`
3. The workflow checks out the code, then fires a `curl POST` to the Render deploy hook URL
4. Render receives the hook, pulls the latest code from GitHub, and rebuilds the Docker image from scratch
5. Once the new container passes Render's health check, traffic switches over to it — zero downtime

### Environment Variables

Secrets never touch the codebase. `GEMINI_API_KEY` and `TAVILY_API_KEY` are set directly in the Render dashboard under Environment Variables. Render injects them at runtime into the container. Locally I use a `.env` file (which is in `.gitignore` and never committed).

### Render Configuration

The `render.yaml` file at the root tells Render exactly how to run the service:

```yaml
services:
  - type: web
    name: linkedin-generator
    env: docker
    dockerfilePath: ./Dockerfile
    plan: free
    envVars:
      - key: GEMINI_API_KEY
        sync: false
      - key: TAVILY_API_KEY
        sync: false
```

`sync: false` means these values are managed manually in the Render dashboard and not synced from any external source — keeps secrets out of version control entirely.

### Setting It Up From Scratch

If you're forking this and want the same pipeline:

1. **Create a Web Service on Render** — connect your GitHub repo, Render will detect the `Dockerfile` automatically
2. **Add environment variables on Render** — `GEMINI_API_KEY` and `TAVILY_API_KEY` under Environment in your service settings
3. **Get the Deploy Hook URL** — Render dashboard → your service → Settings → Deploy Hook → copy the URL
4. **Add it as a GitHub secret** — repo → Settings → Secrets and variables → Actions → New secret → name it `RENDER_DEPLOY_HOOK_URL`, paste the URL
5. **Push to main** — GitHub Actions will fire, hit the hook, and Render will deploy

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
