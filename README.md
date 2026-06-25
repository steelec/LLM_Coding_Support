# Running todo
1. 
2. Test new jinja template on CODER COMPAT MTP, hoping it keeps the speed while correcting the looping. Performance should be appx equal.
   - this is, so far, very good w/ the atomicchat testing (UDT MTP) but not tested otherwise
4. Test AtomicChat build, which is more recently updated than the Tom one you are currently using
   - MoE versions may now work better with jinja template? if so then this could be a speedier option.
   - DONE: seems smart, not yet optimized (increase b / ub)
5. Try `AtomicChat/Qwen3.6-35B-A3B-UDT-MTP-GGUF`
   - this branch does not work quite as advertised, no specification of -md necessary for this here
   - A3B is v. fast (40-50 tok/s), quality not tested
   - web chat interface has a bug (does not allow scrolling)
   - --spec-draft-p-min 0.80  does not seem to work so set --draft-p-min 0.80 (older flag)
     - this ensures that low probability token drafts are not considered and can speed us up!
5. Try `/Users/${USER}/.lmstudio/models/unsloth/Qwen-AgentWorld-35B-A3B-GGUF/Qwen-AgentWorld-35B-A3B-UD-Q8_K_XL.gguf`
6. 
```
/Users/${USER}/Documents/code/atomic-llama-cpp-turboquant/build/bin/llama-server \
  -m /Users/${USER}/.lmstudio/models/AtomicChat/Qwen3.6-35B-A3B-UDT-MTP-GGUF/Qwen3.6-35B-A3B-UDT-Q8_K_XL_MTP.gguf \
  --spec-type draft-mtp \
  --spec-draft-n-max 2 \
  --spec-draft-p-min 0.80 \
  --draft-p-min 0.80 \
  -c 131072 \
  -b 1024 \
  -ub 1024 \
  -ngl 999 \
  -fa on \
  --cache-type-k q8_0 \
  --cache-type-v turbo3 \
  --cache-type-k-draft q8_0 \
  --cache-type-v-draft turbo3 \
  --host 0.0.0.0 \
  --port 8080
```

# Enhanced local model setup for llama w/ guardrails
To use forge-guardrails to improve local model peformance: https://github.com/antoinezambelli/forgehttps://github.com/steelec/LLM_Coding_Support/blob/main/README.md
- can be setup in proxy mode to just point to current server (e.g., LMStudio)
- or with the full llama.cpp backend setup (there are additional benefits to this)

## MAC-specific setup

### Monitoring tools
- monitor GPU and CPU processing and power usage
  - `sudo powermetrics --samplers cpu_power,gpu_power | grep -E "GPU|residency"`
- monitor GPU memory usage
  - `sudo footprint llama-server`
    - where:
      - `mapped file` is the model memmapped into RAM
      - `untagged VM_ALLOCATE` is the KV cache allocation
      - `MALLOC_LARGE` is the runtime engine overhead

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
- `brew install cmake`
- `curl -LsSf https://hf.co/cli/install.sh | bash`

### Example
- `hf download Jackrong/Qwopus3.6-27B-Coder-Compat-MTP-GGUF Qwopus3.6-27B-Coder-Compat-MTP-Q8_0.gguf --local-dir .`

## Install llama
https://github.com/ggml-org/llama.cpp/releases
- create directory `~/Documents/code/tools`
- download (e.g.): `curl -O https://github.com/ggml-org/llama.cpp/releases/download/b9437/llama-b9437-bin-macos-arm64.tar.gz -L`
- untar: `tar -xvf llama*`
- link to local bin (e.g.): `ln -s /Users/${USER}/Documents/code/tools/llama-b9437/llama* ~/.local/bin/.` 

## Run llama-server
- alternative, can run `llama-cli` for testing purposes
- can run turboquant compatible versions as well

## Chat template(s)
Useful to reduce looping, which you saw a lot of in the Qwen 3.6 variants (not in the original dense model). Can also reduce token usage and improve agentic flow, apparently
## froggeric
- https://huggingface.co/froggeric/Qwen-Fixed-Chat-Templates/blob/main/chat_template.jinja
- `hf download froggeric/Qwen-Fixed-Chat-Templates chat_template.jinja --local-dir ~/Documents/code/.`

### Qwen 3.7 27B dense
- reliable, but heavy and slow so we use turboquant
- b/c of this, we can fit the full context in mem!
```
/Users/${USER}/Documents/code/llama-cpp-turboquant/build/bin/llama-server \
  --model /Users/${USER}/.lmstudio/models/Jackrong/Qwen3.6-27B-GGUF/Qwen3.6-27B-Q8_0.gguf \
  -c 262144 \
  -np 1 \
  -b 2048 \
  -ub 2048 \
  -ngl 999 \
  --temp 0.9 \
  --top-p 0.90 \
  --top-k 20 \
  --cache-type-k q8_0 \
  --cache-type-v turbo4 \
  -fa on \
  --host 0.0.0.0 \
  --port 8080
```

### Qwenopus 3.6 UDT MTP (AtomicChat) with turboquant
Trying to get the best of all worlds, as dense turboquant gets bogged down w/ large context (2 tok/s)
- if this does not work look at AtomicChat, this is more up-to-date it seems (but the improvements in tok/s seem not so great)
  - https://huggingface.co/AtomicChat/Qwen3.6-27B-UDT-MTP-GGUF
  - https://github.com/AtomicBot-ai/atomic-llama-cpp-turboquant/tree/feature/turboquant-kv-cache
- Setup:
1. `hf download AtomicChat/Qwen3.6-27B-UDT-MTP-GGUF Qwen3.6-27B-UDT-Q8_K_XL_MTP.gguf --local-dir .`
2. `cd ~/Documents/code/`
3. Download, build, and compile the fork (for macOS metal)
```
https://github.com/AtomicBot-ai/atomic-llama-cpp-turboquant.git`
cd atomic-llama-cpp-turboquant
git checkout feature/turboquant-kv-cache
cmake -B build -DGGML_METAL=ON -DGGML_METAL_EMBED_LIBRARY=ON -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release -j
```
4. Can now be run from `~/Documents/code/atomic-llama-cpp-turboquant/build/bin/llama-server`
- --draft-p-min 0.80 set to reduce time at high contexts (maybe?) 
- does not look like we actually need the --model-draft? it does it itself now?
  - removed in current call (see below)
- Assessment after 2+ days of testing:
  - Very useable as a daily reliable driver - no hand-holding required in testing (at all!)
  - Outputs seem accurate and controlled, no looping (note: currently using expanded chat template)
  - Initially fairly fast (~100 input and ~8ish decode tok/s) but quickly decreases as context expands
  - Falls to v. slow (~27-30 and ~1.8-2 tok/s) with fuller context (111k)
  - There is additional headroom w/in 56 GB for more context at current turbo3 v-chache compression
  - Did not play around with temperature at all (default here is 0.8, which is reasonable for this model)
  - There may be additional tuning that can happen with -b / -ub for given hardware (?)
  - Currently prompt processing requires a lot of re-computation on every prompt
    - Potentially due to agent (opencode) changing metadata
    - Apparently this interacts with the Qwen model architecture (hybrid/recurrent model maintains rolling memory record that relies on sequence order)
      - forcing full prompt re-processing due to lack of cache data (likely due to SWA or hybrid/recurrent memory, see https://git
hub.com/ggml-org/llama.cpp/pull/13194#issuecomment-2868343055) 
      - active area of discussion: https://github.com/ggml-org/llama.cpp/pull/24035
    - May be able to tune with `--ctx-checkpoints 48 --checkpoint-every-n-tokens 2048` to increase frequency and reduce size of checkpoints and therefore spend less than ~5 mins on every prompt
    - $$\text{Checkpoint Size (Bytes)} = 2 \times \text{Layers} \times \text{Embedding Dim} \times \text{Head Dim} \times \text{Precision (Bytes)}$$
      - Size (MB) = 2*64*5124*128*2 (/(1024*1024) for MiB --> 167.90 MB)
        - $+$ 1.5KB * current token position for data that keeps track of the checkpoints (e.g., $150\text{ MiB} + (114,545 \times 0.001438\text{ MiB}) = \mathbf{314.7\text{ MiB}}$ (where MiB is 1024*1024 bytes)
      - This is a balance though, as it takes up more and more mem!
- Why / When use this model?
  - Able to handle v. large context
  - Require accurate output
  - Slow when the context grows (but see above, this is due to be fixed upstream?)

```
Where the "Huge" Gains Actually Happen
The major speedups (+24% to +36%) you see highlighted for this fork occur on the Mixture-of-Experts (MoE) models, specifically the Qwen 3.6 35B-A3B (which only activates 3B parameters per token).

On an MoE model, the main model's verification pass is computationally heavy relative to its active parameter size, allowing the drafting compute to completely hide inside the parallel processing pipeline.

Why use UDT and NextN on the 27B then?
The main reason to use the UDT framework on the 27B dense model isn't a massive speed jump over a standard baseline—it's regression prevention.

Without UDT's tensor masks, running NextN decoding while simultaneously squeezing the KV cache down to 3-bits creates so much math noise that the draft head guesses wrong constantly. This drops the acceptance rate so low that standard NextN setups actually run 7% to 12% slower than regular decoding.

The combination of a UDT file and NextN just allows you to squeeze your context memory down to turbo3 without suffering that speed penalty, keeping you at a net-positive (albeit modest) gain.
```

```
/Users/${USER}/Documents/code/atomic-llama-cpp-turboquant/build/bin/llama-server \
  --model /Users/${USER}/.lmstudio/models/AtomicChat/Qwen3.6-27B-UDT-MTP-GGUF/Qwen3.6-27B-UDT-Q8_K_XL_MTP.gguf \
  -c 131072 \
  -np 1 \
  -b 1024 \
  -ub 1024 \
  -ngl 999 \
  -fa on \
  --spec-type draft-mtp \
  --spec-draft-n-max 2 \
  --draft-p-min 0.80 \
  --cache-type-k q8_0 \
  --cache-type-v turbo3 \
  --cache-type-k-draft q8_0 \
  --cache-type-v-draft turbo3 \
  --jinja \
  --chat-template-file /Users/${USER}/Documents/code/chat_template.jinja \
  --host 0.0.0.0 \
  --port 8080
```

### QwenOpus 3.6 CODER COMPAT MTP (JackRong)
- supposed to fix looping, but does not seem to entirely
- `hf download Jackrong/Qwopus3.6-27B-Coder-Compat-MTP-GGUF Qwopus3.6-27B-Coder-Compat-MTP-Q8_0.gguf --local-dir .`
```
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
  -np 1 \
  --metrics
```
### QwenOpus 9B
- https://huggingface.co/Jackrong/Qwopus3.5-9B-v3-GGUF
- test downloads with `--dry-run` to make sure you are getting what you want
- `hf download Jackrong/Qwopus3.5-9B-v3-GGUF Qwopus3.5-9B-v3.Q8_0.gguf --local-dir .`

- not tested yet, but could be good to try!
```
llama-server \
  --model /Users/${USER}/.lmstudio/models/Jackrong/Qwopus3.5-9B-v3-GGUF/Qwopus3.5-9B-v3.Q8_0.gguf \
  -c 128000 \
  -np 1 \
  -b 2048 \
  -ub 2048 \
  --chat-template-kwargs '{"preserve_thinking": true}' \
  --spec-type draft-mtp \
  --spec-draft-n-max 3 \
  --jinja \
  -ngl 999 \
  --temp 1.0 \
  --top-p 0.95 \
  --cache-type-k q4_0 \
  --cache-type-v q4_0 \
  -fa on \
  --host 0.0.0.0 \
  --port 8082
```

#### Turboquant
For faster prefill and slightly less mem
```
/Users/${USER}/Documents/code/llama-cpp-turboquant/build/bin/llama-server \
  --model /Users/${USER}/.lmstudio/models/Jackrong/Qwopus3.5-9B-v3-GGUF/Qwopus3.5-9B-v3.Q8_0.gguf \
  -c 128000 \
  -np 1 \
  -b 2048 \
  -ub 2048 \
  -ngl 999 \
  --temp 1.0 \
  --top-p 0.95 \
  --cache-type-k turbo3 \
  --cache-type-v turbo3 \
  -fa on \
  --host 0.0.0.0 \
  --port 8082
```

### VibeCoder 3B
For math, science, and programming reasoning. Not a tool caller. Likely not very good at writing actual code(?).
Initial testing suggests that this is both fast and **not very accurate** (Reimmanian manifold question failure in details).

>> See here for updated wrapping for tool calling, with testing: https://github.com/ricardodeazambuja/vibethinker_tests/tree/main
- https://huggingface.co/prithivMLmods/VibeThinker-3B-GGUF
- create directory in models: `prithivMLmods/VibeThinker-3B-GGUF`
- also grabbed the reasoning template from JohnRoger (still cannot use tool calls though!)
- `hf download prithivMLmods/VibeThinker-3B-GGUF VibeThinker-3B.Q8_0.gguf --local-dir .`

models/prithivMLmods/VibeThinker-3B-GGUF/VibeThinker-3B.Q8_0.gguf
models/prithivMLmods/VibeThinker-3B-GGUF/2026_06_reasoning_JohnRoger.jinja

```
llama-server \
  --model /Users/${USER}/.lmstudio/models/prithivMLmods/VibeThinker-3B-GGUF/VibeThinker-3B.Q8_0.gguf \
  -c 64000 \
  -np 1 \
  -b 2048 \
  -ub 2048 \
  -ngl 999 \
  --temp 1.0 \
  --top-p 0.95 \
  --min-p 0.05 \
  --chat-template-file /Users/${USER}/.lmstudio/models/prithivMLmods/VibeThinker-3B-GGUF/2026_06_reasoning_JohnRoger.jinja \
  -fa on \
  --host 0.0.0.0 \
  --port 8081
```

### QwenOpus 3.6 CODER MTP (JackRong)
- **LOOPY** on current setup
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

## Take advantage of rotorquant
Allows much faster interaction (prefill only?) and lower quantization of k / v caches without loss of precision
- install git repo and build (`brew install cmake` if necessary)
```
git clone https://github.com/johndpope/llama-cpp-turboquant.git
cd llama-cpp-turboquant
git checkout feature/planarquant-kv-cache

# Build with native Apple Silicon Metal support
cmake -B build -DGGML_METAL=ON -DGGML_METAL_EMBED_LIBRARY=ON -DCMAKE_BUILD_TYPE=Release
cmake --build build -j
```

### Run model
We cannot use the MTP models, as it does not work on that architecture
```
./build/bin/llama-server --model /Users/${USER}/.lmstudio/models/Jackrong/Qwen3.6-27B-GGUF/Qwen3.6-27B-Q8_0.gguf \
  -c 80000 \
  -np 1 \
  -b 2048 \
  -ub 2048 \
  -ngl 999 \
  --temp 0.9 \
  --top-p 0.90 \
  --top-k 20 \
  --cache-type-k iso3 \
  --cache-type-v iso3 \
  --spec-type draft-mtp \
  --spec-draft-n-max 3 \
  -fa on \
  --host 0.0.0.0 \
  --port 8080
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


