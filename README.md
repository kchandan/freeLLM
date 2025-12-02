# FreeLLM 

This repository provides a crowd-sourced collection of LLM/SLM proxy endpoints, powered by LiteLLM, to help everyday developers experiment with AI without worrying about token limits or infrastructure costs.
These endpoints aggregate compute from volunteered machines — home PCs, laptops, edge devices, and more — to offer free (whenever possible) access for prototyping and learning.

⚠️ Disclaimer
These endpoints are intended strictly for experimentation and education.
There are no SLAs, no uptime guarantees, and usage is entirely at your own risk. Malicious or abusive usage is strictly prohibited.

# FreeLLM Architecture

<img width="888" height="736" alt="image" src="https://github.com/user-attachments/assets/a9ddd1a1-21ab-459a-a20b-0738babbb4f5" />


## Repository Layout

```
.
├── ansible/
│   ├── collections/         # Galaxy collection requirements
│   ├── inventory/           # Example inventory definitions
│   ├── playbooks/           # Entry-point playbooks (docker, firewall)
│   └── roles/               # Reusable automation roles
├── config.yaml              # FreeLLM proxy configuration
├── docker-compose.yml       # Local docker-compose example for the proxy stack
├── prometheus.yml           # Prometheus scrape configuration
└── README.md
```

## Prerequisites

- An Ubuntu 22.04/24.04 host that will run vLLM and the proxy.
- SSH access with a user that can escalate with `sudo` (default inventory user is `ubuntu`).
- Ansible 2.15+ on your control machine.
- Optional: NVIDIA GPU drivers and CUDA stack on the target if you plan to run GPU-backed models.

> **Tip:** Disable strict host key checking when iterating on a fresh machine: `export ANSIBLE_HOST_KEY_CHECKING=False`.

## Using the Ansible Automation

1. Install the required community collections:
   ```bash
   ansible-galaxy collection install -r ansible/collections/requirements.yml
   ```
2. Update `ansible/inventory/hosts.ini` with your host name or IP and SSH user.
3. Harden the host firewall and Docker networking:
   ```bash
   ansible-playbook \
     -i ansible/inventory/hosts.ini \
     ansible/playbooks/harden_firewall.yml \
     --private-key ~/workspace/<private key>.pem
   ```
4. Install Docker CE and supporting packages:
   ```bash
   ansible-playbook \
     -i ansible/inventory/hosts.ini \
     ansible/playbooks/install_docker.yml \
     --private-key ~/workspace/<private key>.pem
   ```

Role defaults keep SSH (TCP/22) open and lock down the proxy port (TCP/8001) to the CIDR list you supply via `allowed_8001_cidrs`. Override these variables in your inventory or on the command line to match your environment.

## Running the FreeLLM Proxy Locally

Bring the proxy stack up with Docker Compose (includes Redis and optional metrics exporters):
```bash
docker compose up -d
```

Test the API once the stack is up:
```bash
curl http://freellm.torontoai.io:4000/v1/models \
  -H "Authorization: Bearer torontoai"

curl http://freellm.torontoai.io:4000/v1/chat/completions \
  -H "Authorization: Bearer torontoai" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "Qwen/Qwen3-0.6B",
    "messages": [{"role": "user", "content": "Explain vector databases in two sentences."}]
  }'
```
curl http://<IP Address>:8001/v1/chat/completions \
  -H "Authorization: Bearer SECRETKEY123" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "Qwen/Qwen3-0.6B",
    "messages": [{"role": "user", "content": "Explain vector databases in two sentences."}]
  }'

## Running vLLM with Docker

Use the official image to start a GPU-enabled vLLM instance:
```bash
docker run --runtime nvidia --gpus all \
  -v ~/.cache/huggingface:/root/.cache/huggingface \
  --env "HF_TOKEN=$HF_TOKEN" \
  -p 8000:8000 \
  --ipc=host \
  vllm/vllm-openai:latest \
  --model Qwen/Qwen3-0.6B
```


sudo iptables -I INPUT -s 18.236.254.81 -p tcp --dport 8002 -j ACCEPT
sudo iptables -I INPUT -s 18.236.254.81 -p tcp --dport 8003 -j ACCEPT
