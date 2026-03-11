# Setup
- LMStudio (mac version)
- tailcale (secure connections between devices)

## Tools
### AnythingLLM
- desktop app: https://anythingllm.com/
- https://github.com/Mintplex-Labs/anything-llm
### LMStudio
- run remote, link through tailscale as necessary (they now have a discoverable secure login w/ device discovery, which I have not tried)
#### Unload models remotely:
- curl http://<URL:PORT>/api/v1/models/unload   -H "Authorization: "   -H "Content-Type: application/json"   -d '{
    "instance_id": "qwen3-coder-next"
  }'
## Models
- 
## Links


