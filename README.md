# Setup
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


