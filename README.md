# Enhanced local model setup for llama w/ guardrails
To use forge-guardrails to improve local model peformance: https://github.com/antoinezambelli/forgehttps://github.com/steelec/LLM_Coding_Support/blob/main/README.md
- can be setup in proxy mode to just point to current server (e.g., LMStudio)
- or with the full llama.cpp backend setup (there are additional benefits to this)

## MAC-specific setup
### Programs
- `tailscale` (add to autostart!)
- `brew` (if you want to be doing some more custom things!)

### Shared memory settings
- by default, mac has a hard-coded amount of mem that is available to the GPU
  - the full memory is therefore **not available** to your model, which will crash performance if you are loading large contexts and/or k/v cache with higher precision
- set to 75% of available memory as a safe setting (leaving other for system resources)
  - set to 56GB (for 64 GB mac): `sudo sysctl iogpu.wired_limit_mb=57344`
  - default is `sysctl iogpu.wired_limit_mb=0`

## Install hugging face cli
Allows you to download models directly from cli
- `brew install python`
- `curl -LsSf https://hf.co/cli/install.sh | bash`

### Example
- `hf download Jackrong/Qwopus3.6-27B-Coder-Compat-MTP-GGUF Qwopus3.6-27B-Coder-Compat-MTP-Q8_0.gguf --local-dir .`

## Install llama
https://github.com/ggml-org/llama.cpp/releases
- create directory `~/Documents/code/tools`
- download (e.g.): `curl -O https://github.com/ggml-org/llama.cpp/releases/download/b9437/llama-b9437-bin-macos-arm64.tar.gz -L`
- untar: `tar -xvf llama*`
- link to local bin (e.g.): `ln -s /Users/csteele/Documents/code/tools/llama-b9437/llama* ~/.local/bin/.` 

## Run llama-server
- alternative, can run `llama-cli` for testing purposes
### QwenOpus 3.6 CODER COMPAT MTP (JackRong)
- supposed to fix looping
- `hf download Jackrong/Qwopus3.6-27B-Coder-Compat-MTP-GGUF Qwopus3.6-27B-Coder-Compat-MTP-Q8_0.gguf --local-dir .`
llama-server --model /Users/${USER}/.lmstudio/models/Jackrong/Qwopus3.6-27B-Coder-COMPAT-MTP-GGUF/Qwopus3.6-27B-Coder-Compat-MTP-Q8_0.gguf \
  -c 80000 \
  -b 2048 \
  -ub 2048 \
  -ngl 999 \
  --temp 0.9 \
  --top-p 0.90 \
  --top-k 20 \
  --min-p 0.0 \
  --presence-penalty 0.0 \
  --repeat-penalty 1.1 \
  --chat-template-kwargs '{"preserve_thinking": true}' \
  --spec-type draft-mtp \
  --spec-draft-n-max 3 \
  --jinja \
  --cache-type-k q4_0 \
  --cache-type-v q4_0 \
  --cache-prompt \
  --host 0.0.0.0 \
  --port 8080 \
  -fa on \
  --metrics
```

### QwenOpus 3.6 CODER MTP (JackRong)
- here we run an MTP-style for agent usage, with full offloading to GPU (QwenOpus 3.6 MTP from JackRong)
- this looped like nobody's business
```
llama-server \                                                                                              
  --model /Users/${USER}/.lmstudio/models/Jackrong/Qwopus3.6-27B-Coder-MTP-GGUF/Qwopus3.6-27B-Coder-MTP-Q8_0.gguf \
  --flash-attn on \
  -c 200000 \
  -ngl 999 \
  --temp 0.9 \
  --top-p 0.95 \
  --top-k 20 \
  --min-p 0.0 \
  --presence-penalty 0.0 \
  --repeat-penalty 1.0 \
  --chat-template-kwargs '{"preserve_thinking": true}' \
  --spec-type draft-mtp \
  --spec-draft-n-max 3 \
  --jinja \
  --cache-type-k q8_0 \
  --cache-type-v q8_0 \
  --host 0.0.0.0 \
  --port 8080 \
  --metrics
```


## Install forge-guardrails
python package, requires version 3.12: https://github.com/antoinezambelli/forge
- install uv: `curl -LsSf https://astral.sh/uv/install.sh | sh`
- create venv in `~/Documents/code`: `uv venv env_forge --python 3.12`
- activate it: `source env_forge/bin/activate`
- install forge: `uv pip install forge-guardrails`

## Connect LMStudio<-forge<-opencode
1. run forge wrapper (simple default mode): `python -m forge.proxy   --backend-url http://127.0.0.1:8080  --port 8081 --budget-tokens 262144`
2. test from local machine (if ports are visible and bound): `curl http://localhost:8081/v1/models`
3. test from the remote machine: `curl http://<remote_ip>:8081/v1/models`
  - should have a response like: `{"object": "list", "data": [{"id": "forge", "object": "model"}]}`
4. Update `.opencode/opencode.json` to point to newly wrapped interface:
- this does not currently work as advertised
- model name is not correctly passed through (Jun 2, 2026) but the model can be accessed at least partially (fails at some point)
```
{
  "$schema": "https://opencode.ai/config.json",
  "model": "unsloth/qwen3.6-35b-a3b@q8_k_xl",
  "provider": {
    "lmstudio": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "LM Studio (local)",
      "options": {
        "baseURL": "http://<remote_ip>:8081/v1"
      }
      }
      }
    }
```

# General Setup with LMStudio
- useful for downloading models and testing parameters
  - parameter testing is faster with `llama-cli`
- LMStudio (mac version)
- tailcale (secure connections between devices)
- opencode (command-line based coding harness)

## Tools
### Harness(es)
#### Opencode
- Installation
  - `curl -fsSL https://opencode.ai/install | bash`
  - To provide lookup of models direct from local/networked lmstudio instance:
    - `npm install opencode-lmstudio`
  - if you do not have `npm`, it must be installed (see below)
- config files live in `~/.opencode/`
- update to point at your local LLM server, you can add models specifically or a provider
  - you can aslo add specific models to this directly, but having `opencode-lmstudio` installed allows generation the list of available based on the baseURL (use `/models` command in opencode)
  - `vim ~/.opencode/opencode.json`
```
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": [
    "opencode-lmstudio@latest"
  ],
  "provider": {
    "lmstudio": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "LM Studio (local)",
      "options": {
        "baseURL": "http://<IP>:<port>/v1"
      }
    }
  }
}
```
##### Install npm to get access to the whole world of skills
- Skills give superpowers to your harness
- many available at https://skills.sh/
  - `sudo apt install npm`
- Install specific skill to a specific agent harness
  - `npx skills add https://github.com/forrestchang/andrej-karpathy-skills -a opencode`

### AnythingLLM
- desktop app: https://anythingllm.com/
- https://github.com/Mintplex-Labs/anything-llm
- setup by pointing at LMStudio
  - can select all models available from LMStudio and load directly in interface (using the ?brain? icon)
### LMStudio
- run remote, link through tailscale as necessary (they now have a discoverable secure login w/ device discovery, which I have not tried)
#### Unload models remotely:
- leaving authorization blank, if not set https://lmstudio.ai/docs/developer/rest/unload
- `curl http://<URL:PORT>/api/v1/models/unload   -H "Authorization: "   -H "Content-Type: application/json"   -d '{"instance_id": "qwen3-coder-next"}'`
## Models
- 
## Links


