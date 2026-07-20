# Ollama Setup Guide

This project provides Docker Compose configurations for running [Ollama](https://ollama.com/) locally, either standalone or alongside Open WebUI.

---

## Prerequisites

- **Docker** (latest version)
- **Docker Compose v2+**
- **GPU support:** AMD ROCm devices (`/dev/dri`, `/dev/kfd`) for accelerated inference on Linux

> **Note:** The default images use `rocm` variant optimized for AMD GPUs. For NVIDIA GPU users, change the image tag to `:latest`.

---

## Option 1: Run Ollama Standalone

Use this setup if you want to run just Ollama without a web UI (e.g., connecting via API from your own application).

### Directory Structure
```
ollama-setup/
├── docker-compose.yml           # Standalone Ollama configuration
├── volumes/
│   └── ollama                  # Persistent storage for models/data
└── README.md                   # This file
```

### Start Ollama

1. Ensure the `volumes` directory exists:
   ```bash
   mkdir -p volumes/ollama
   ```

2. Pull required images and start services:
   ```bash
   docker compose up -d --build
   ```

3. Check that Ollma is running:
   ```bash
   docker compose ps
   ```

### Connect to Ollama API

Once started, access the local LLM via REST API at `http://localhost:11434`:

- **API Docs:** http://localhost:11434/docs  
- **Status check:** Open a browser or curl GET request against that URL.

#### Example Usage with Curl
```bash
# Create model
curl http://localhost:11434/api/tag -d '{ "name": "my-model" }'

# Pull LLM models (e.g., llama2)
curl http://localhost:11434/api/pull -d '{ "model": "llama2", "stream": true }'

# Run inference
curl http://localhost:11434/api/generate -d '{ 
  "model": "my-model",
  "prompt": "Explain quantum computing in one sentence.",
  "stream": false
}'

# Create chat session and send messages
SESSION_ID=$(curl -s http://localhost:11434/api/chat/create -d '{"name":"session"}' | jq -r '.id')
echo $SESSION_ID  # Store this for later use

curl http://localhost:11434/api/chat -X POST \
     --header "Content-Type: application/json" \
     -d "{\"model\":\"my-model\",\"messages\":[{\"role\":\"user\"},{\"content\":\"Hello!\"}], \"stream\":false}" | jq .response  # Note this session ID

curl http://localhost:11434/api/chat/messages/$SESSION_ID 
```

### Stop Ollama
```bash
docker compose down -v   # Removes volumes too; remove `-v` to persist data
```

---

## Option 2: Run with Open WebUI

Use this setup if you want a graphical web interface (Open WebUI) paired with Ollma.

### Directory Structure
```
ollama-setup/
├── docker-compose-web-ui.yml   # Ollama + Open WebUI configuration
├── volumes/
│   ├── ollama                  # Persistent storage for models/data
│   └── open-webui              # UI settings, sessions, etc.
└── README.md                   # This file
```

### Start with Both Services

1. Create `volumes/open-webui` directory:
   ```bash
   mkdir -p volumes/open-webui
   ```

2. Pull images and start services together:
   ```bash
   docker compose -f docker-compose-web-ui.yml up -d --build
   ```

3. Check service status:
   ```bash
   docker compose -f docker-compose-web-ui.yml ps
   ```

### Access Open WebUI

Navigate to http://localhost:3000 in your browser or run this command instead of the URL:
```bash
curl localhost:3000 
```

**Note:** The first launch may take a few minutes as Ollama pulls models and initializes containers.

#### Example with Curl for Open WebUI API (via Docker)
```bash
# Use host.docker.internal to access services from the container's perspective
curl http://host.docker.internal:3000/api/tags  # List loaded models via UI/API
```

### Stop Both Services
```bash
docker compose -f docker-compose-web-ui.yml down -v 
```

## License & Attribution

This project uses official Docker Compose configurations from:  
- Ollama: https://ollama.com   
- Open WebUI: ghcr.io/open-webui/open-webui  

