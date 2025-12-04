
## Running the FreeLLM Proxy with Ollama Locally (MacOS)

Install Ollama

Run Llama 3.2:1b

```bash
ollama run llama3.2:1b
```

```bash
cd litellm
docker compose up -d
```

Test the API once:

```bash
curl http://localhost:4000/v1/models \
  -H "Authorization: Bearer freellm"

curl http://localhost:4000/v1/chat/completions \
  -H "Authorization: Bearer freellm" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "ollama/llama3.2:1b",
    "messages": [{"role": "user", "content": "Explain vector databases in two sentences."}]
  }'
```

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
