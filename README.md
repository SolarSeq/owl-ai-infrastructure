# Owl AI Infrastructure

Owl AI Infrastructure is a Docker Compose deployment for running local AI inference with llama.cpp and providing a browser-based interface through Open WebUI.

This project was built as part of Operation Owl to explore local model serving, Docker networking, resource limits, persistent storage, service separation, and application-to-model integration.

## Architecture

Open WebUI
    |
    | OpenAI-compatible HTTP API
    v
llama.cpp server
    |
    v
Local GGUF models

Both services run in Docker containers on a shared private bridge network.

The llama.cpp API is not published directly to the host in this configuration. Open WebUI communicates with it over the internal Docker network.

## Components

### llama.cpp

The `owl-ai-engine` service runs the official llama.cpp server image.

It provides:

- OpenAI-compatible API endpoints
- local GGUF model loading
- configurable context size
- API-key authentication
- CPU and memory limits
- support for text and vision-capable models

### Open WebUI

The `owl-ai-webui` service provides a browser interface for interacting with the local model server.

It is configured to use llama.cpp through the OpenAI-compatible API rather than Ollama.

## Docker Networking

The services share a Docker bridge network named:

`owl-ai-network`

Open WebUI reaches the model server internally at:

`http://owl-ai-engine:8080/v1`

The llama.cpp API itself does not need to be exposed directly to the host.

## Storage

The example deployment uses relative paths:

- `./data/models` for local model files
- `./data/webui` for Open WebUI application data

Model files, application databases, conversations, and runtime data are intentionally excluded from Git.

## Configuration

Copy `.env.example` to `.env` and replace the placeholder values.

Required variables:

- `LLAMA_API_KEY`
- `WEBUI_SECRET_KEY`

The real `.env` file is excluded from Git.

## Resource Limits

The example configuration limits resource usage so the AI stack can coexist with other services on the same Linux host.

### llama.cpp

- 3.5 CPUs
- 9 GB memory
- 8192-token context
- one model loaded at a time
- four inference threads
- parallelism set to one

### Open WebUI

- 0.5 CPU
- 1536 MB memory

These values were tuned for a small CPU-only home server and should be adjusted for other hardware.

## Deployment

1. Install Docker and Docker Compose.
2. Place GGUF model files under `data/models`.
3. Copy `.env.example` to `.env`.
4. Replace the placeholder secrets.
5. Start the services with Docker Compose.
6. Open the web interface on port `3001`.

The example Compose file uses:

`3001:8080`

for the Open WebUI host mapping.

## Security and Privacy

This repository intentionally excludes:

- real API keys
- WebUI secrets
- model files
- Open WebUI databases
- conversation history
- backups
- logs
- private server paths
- private LAN addresses

The public Compose file is a sanitized version of the working deployment.

## Lessons Learned

This project provided practical experience with:

- Docker Compose
- container networking
- persistent volumes
- local AI model serving
- OpenAI-compatible APIs
- CPU and memory limits
- Linux server administration
- service health and restart behavior
- separating applications from AI inference
- troubleshooting local AI performance

One important result was that small text workloads were usable on CPU-only hardware, while interactive vision and screenshot analysis were too slow for the intended user experience.

That limitation informed the decision to keep lightweight services on the existing server while planning GPU-backed inference for more demanding workloads.

## Project Background

This infrastructure stack supports experiments and applications developed under Operation Owl, including local AI assistants and Discord integrations.

The architecture separates the inference layer from client applications so bots, browser interfaces, and future applications can share a common model-serving backend.

This project was developed with AI-assisted configuration and troubleshooting. Architecture decisions, deployment, testing, server administration, and production troubleshooting were performed as part of the development process.
