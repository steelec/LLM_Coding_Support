# Watching
- dflash (beellama)
- lucebox approach: https://github.com/Luce-Org/lucebox-hub

# Learnings
- beelama with 27B MTP is the current favourite
  - still potentuial issues with lost prompt context though
- k / v cache tradeoffs
  - "v is everything" - there is a paper for this, apparently showing that we should always prioritize v
  - the key (k) puts is in the correct or not location, but either way if v (value) is wrong we will be wrong 
- Dense model (27B) almost always better for complex coding
- 35B 3B MOE is v. fast and pretty good overall, but maybe not as good for complex tasks
  - potentially most useful for rapid iteration, followed by confirmation with the dense model
- AtomicChat is quite stable and works well, but hallucinates flags (`nextn` does not exist)
  - implementing it with CODER COMPAT MTP may be a big win, but testing reveals that it still slows down rapidly
    - tested and still seems to stop or loop
  - works, but is it trustworthy? it seems like it is 2ndary to their sales?
- Beellama looks great, but has not allowed me to run anything without crashing
  - keep an eye on it...?
  - v0.3.2 seems to mostly work, but I have very poor acceptance (max .17) with the current setup, could be optimized...
  - but still buggy and cannot get acceptance up --> **WAIT** until this is within the main llama.cpp repo
    - not ready for MAC metal
- Interesting approach for running these on limited hardware
  - definitely some takeaways for you: https://www.youtube.com/watch?v=8F_5pdcD3HY (--> speculative decoding does not improve in MOE models, makes it worse)
  - Dflash here, works only on v. specific conditions: https://www.youtube.com/watch?v=9vY4-Z-tkHs (--> works in specific conditions, if you can fit in VRAM and if you are coding)
- Jackrong's CODER COMPAT MTP still runs into looping issues even with new jinja template, currently not using
- `headroom wrap` combined with the extensive jinja template from `froggeric` and the unsloth MTP on standard llama.cpp seems to be a good workhorse with reasonable decode speed
  - b/c this does not have turboquant we hit context limits though, so sometimes this just stops
  - beelama

# Running todo
2. Test new jinja template on CODER COMPAT MTP, hoping it keeps the speed while correcting the looping. Performance should be appx equal.
   - [x] this is, so far, very good w/ the atomicchat testing (UDT MTP) but not tested otherwise
4. Test AtomicChat build, which is more recently updated than the Tom one you are currently using
   - MoE versions may now work better with jinja template? if so then this could be a speedier option.
   - [x] DONE: seems smart, not yet optimized (increase b / ub) 
5. Try `AtomicChat/Qwen3.6-35B-A3B-UDT-MTP-GGUF`
   - this branch does not work quite as advertised, no specification of -md necessary for this here
   - [x] A3B is v. fast (40-50 tok/s), quality not extensively tested but expected to be lower based on others experience - good for simple coding but may break down with more complex tasks
   - web chat interface has a bug (does not allow scrolling)
   - --spec-draft-p-min 0.80  does not seem to work so set --draft-p-min 0.80 (older flag)
     - this ensures that low probability token drafts are not considered and can speed us up!
6. Try DFlash (beellama)
   - this did not work as of Jun 25, 2026. Model loads, but first prompt leads to crash
   - after much testing of params, leave for the moment
5. Try `/Users/${USER}/.lmstudio/models/unsloth/Qwen-AgentWorld-35B-A3B-GGUF/Qwen-AgentWorld-35B-A3B-UD-Q8_K_XL.gguf`
   - downloaded, not tested
7. implement `headroom` around opencode harness, now that it can be used to wrap
   - (https://github.com/headroomlabs-ai/headroom/pull/1105)
   - not yet in pip

```
/Users/${USER}/Documents/code/atomic-llama-cpp-turboquant/build/bin/llama-server \
  -m /Users/${USER}/.lmstudio/models/AtomicChat/Qwen3.6-35B-A3B-UDT-MTP-GGUF/Qwen3.6-35B-A3B-UDT-Q8_K_XL_MTP.gguf \
  --spec-type draft-mtp \
  --spec-draft-n-max 2 \
  --spec-draft-p-min 0.75 \
  -np 1
  -c 131072 \
  -b 2048 \
  -ub 512 \
  -ngl 999 \
  -fa on \
  --cache-type-k q8_0 \
  --cache-type-v turbo3 \
  --cache-type-k-draft q8_0 \
  --cache-type-v-draft turbo3 \
  --host 0.0.0.0 \
  -ctxcp 48 \
  -cms 2048 \
  --cache-reuse 256 \
  --port 8080
```

### Beellama versions
#### 27B [CURRENT FAVOURITE]
- Jul 3, 2026 added --swa-full to try to increase prompt processing spd by retaining context
  - does not appear to have fixed the issue
  - now trying -ub 512 -b 512 --> which can apparently make block checking more reliable
    - (now written in the command below)
  - spec-draft-n-max 3 seems to be the best currently, tried 2,3,4
```
/Users/${USER}/Documents/code/beellama.cpp/build/bin/llama-server \
  --model /Users/${USER}/.lmstudio/models/unsloth/Qwen3.6-27B-MTP-GGUF/Qwen3.6-27B-UD-Q8_K_XL.gguf \
  --spec-type draft-mtp \
  --spec-draft-n-max 3 \
  -c 102400 \
  -b 512 \
  -ub 512 \
  -ngl 999 \
  --temp 0.6 \
  --top-p 1.0 \
  --top-k 20 \
  --min-p 0.0 \
  --chat-template-file /Users/${USER}/Documents/code/chat_template.jinja \
  --jinja \
  --cache-type-k q4_0 \
  --cache-type-v q4_0 \
  --cache-prompt \
  --host 0.0.0.0 \
  --port 8080 \
  -fa on \
  -np 1 \
  --metrics \
  -ctxcp 48 -cms 2048 --n-predict 16384 --swa-full
```

#### A3B
```
/Users/${USER}/Documents/code/beellama.cpp/build/bin/llama-server \
  --model /Users/${USER}/.lmstudio/models/unsloth/Qwen3.6-35B-A3B-GGUF/Qwen3.6-35B-A3B-Q8_0.gguf \
  --model-draft /Users/${USER}/.lmstudio/models/Anbeeld/Qwen3.6-35B-A3B-DFlash-GGUF/qwen36-35b-a3b-dflash-Q8_0.gguf \
  --spec-type dflash \
  --spec-dflash-cross-ctx 1024 \
  --port 8080 \
  -np 1 \
  --kv-unified \
  -ngl all \
  --spec-draft-ngl all \
  -b 2048 -ub 512 \
  --ctx-size 102400 \
  --cache-type-k q4_1 --cache-type-v q4_1 \
  --flash-attn on \
  --jinja \
  --chat-template-file /Users/${USER}/Documents/code/chat_template.jinja \
  --no-mmap --mlock \
  --reasoning on \
  --temp 0.6 --top-k 20 --top-p 1.0 --min-p 0.0 --host 0.0.0.0 --swa-full
```

### Llama.cpp
llama.cpp version of A3B
```
llama-server \
  --model /Users/${USER}/.lmstudio/models/unsloth/Qwen3.6-35B-A3B-GGUF/Qwen3.6-35B-A3B-Q8_0.gguf \
  --port 8080 \
  -np 1 \
  --kv-unified \
  -ngl all \
  --spec-draft-ngl all \
  -b 2048 -ub 512 \
  --ctx-size 102400 \
  --cache-type-k q5_1 --cache-type-v q8_0 \
  --flash-attn on \
  --jinja \
  --chat-template-file /Users/${USER}/Documents/code/chat_template.jinja \
  --no-mmap --mlock \
  --reasoning on \
  --temp 0.6 --top-k 20 --top-p 1.0 --min-p 0.0 --host 0.0.0.0 -ctxcp 48 -cms 2048 --cache-reuse 256
```
# Enhanced local model setup for llama (w/ guardrails if possible?)

## Headroom
Used to compress prompts (+ more) @ the harness level
- now wraps opencode harness: https://github.com/headroomlabs-ai/headroom/pull/1105
- `pip install headroom-ai[all]`

## Forge Guardrails
Use forge-guardrails to improve local model peformance: https://github.com/antoinezambelli/forgehttps://github.com/steelec/LLM_Coding_Support/blob/main/README.md

Can improve small model output with nudges etc, providing much higher output quaulity at the cost of time
- install uv
  - did this in `~/Documents/code/tools/`
- create venv, source it
  - `uv venv --python 3.13`
  - `source .venv/bin/activate`
- `uv pip install forge-guardrails`
- run it after starting model, point opencode at it
  - `forge-proxy --backend-url http://localhost:8080 --port 8081`

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
Excellent chat template that seems to smooth out Qwen usage, definitely use this or something like it
- https://huggingface.co/froggeric/Qwen-Fixed-Chat-Templates/blob/main/chat_template.jinja
- `hf download froggeric/Qwen-Fixed-Chat-Templates chat_template.jinja --local-dir ~/Documents/code/.`

### Qweb 3.6 DFlash speculative decoding with beellama
**Not currently working on your MAC M1 setup (after testing)**
- Get and build Beellama
  - https://github.com/Anbeeld/beellama.cpp.git
```
git clone https://github.com/Anbeeld/beellama.cpp.git
cd beellama.cpp
cmake -B build -DGGML_METAL=ON -DCMAKE_BUILD_TYPE=Release
cmake --build build -j
```
- Requires 3 files:
  - Target model (https://huggingface.co/unsloth/Qwen3.6-27B-GGUF)
  - Multimodal projector file (mmproj) (not required, unless wanting to feed raw images)
    -   `--mmproj /Users/${USER}/.lmstudio/models/Anbeeld/Qwen3.6-27B-DFlash-GGUF/Qwen3.6-27B-DFlash-bf16.gguf \`
        `--no-mmproj-offload \` (on MAC metal, this has to be handled by CPU)
  - DFlash draft model: (https://huggingface.co/Anbeeld/Qwen3.6-27B-DFlash-GGUF)
    - `hf download Anbeeld/Qwen3.6-27B-DFlash-GGUF --local-dir .`
```
/Users/${USER}/Documents/code/beellama.cpp/build/bin/llama-server \
  --model /Users/${USER}/.lmstudio/models/unsloth/Qwen3.6-27B-GGUF/Qwen3.6-27B-Q8_0.gguf \
  --spec-draft-model /Users/${USER}/.lmstudio/models/Anbeeld/Qwen3.6-27B-DFlash-GGUF/Qwen3.6-27B-DFlash-Q8_0.gguf \
  --spec-type dflash \
  --spec-dflash-cross-ctx 1024 \
  --port 8080 \
  -np 1 \
  --kv-unified \
  -ngl all \
  --spec-draft-ngl all \
  -b 2048 -ub 512 \
  --ctx-size 102400 \
  --cache-type-k q5_0 --cache-type-v q4_1 \
  --flash-attn on \
  --jinja \
  --chat-template-file /Users/${USER}/Documents/code/chat_template.jinja \
  --no-mmap --mlock \
  --reasoning on \
  --temp 0.6 --top-k 20 --top-p 1.0 --min-p 0.0 --host 0.0.0.0
```

#### 27B MTP
- https://unsloth.ai/docs/models/qwen3.6#mtp-guide#thinking-enable-disable--preserve-thinking
- these seem to work very well, but sometimes just stop working potentially due to memory full - but it is not clear
```
llama-server \
  --model /Users/${USER}/.lmstudio/models/unsloth/Qwen3.6-27B-MTP-GGUF/Qwen3.6-27B-UD-Q8_K_XL.gguf \
  --spec-type draft-mtp \
  --port 8080 \
  -np 1 \
  -ngl all \
  -b 2048 -ub 512 \
  --ctx-size  100000\
  --cache-type-k q4_1 --cache-type-v q4_1 \
  --flash-attn on \
  --jinja \
  --chat-template-file /Users/${USER}/Documents/code/chat_template.jinja \
  --reasoning on \
  --temp 0.6 --top-k 20 --top-p 0.95 --min-p 0.0 --host 0.0.0.0 --spec-draft-n-max 2 --metrics --n-predict 16384
```

- Another version
```
llama-server \
  --model /Users/${USER}/.lmstudio/models/unsloth/Qwen3.6-27B-MTP-GGUF/Qwen3.6-27B-UD-Q8_K_XL.gguf \
  --spec-type draft-mtp \
  --port 8080 \
  -np 1 \
  -ngl all \
  -b 2048 -ub 512 \
  --ctx-size 102400 \
  --cache-type-k q4_1 --cache-type-v q4_1 \
  --flash-attn on \
  --jinja \
  --chat-template-file /Users/${USER}/Documents/code/chat_template.jinja \
  --no-mmap --mlock \
  --reasoning on \
  --temp 0.6 \
  --top-k 20 \
  --top-p 0.95 \
  --min-p 0.0 \
  --host 0.0.0.0 \
  --spec-draft-n-max 2 \
  --metrics
```

#### Updated beellama Dflash
Switched to in-process v0.3.2 version of git code to be able to run on Mac, slight modifications to call
- note that in mac this is currently necessary and slows things down: `dflash: GPU cross ring unavailable; using CPU hidden capture`
- with this approach below, draft acceptance is VERY low (<20%)
- 
```
/Users/${USER}/Documents/code/beellama.cpp/build/bin/llama-server \                                                                                      
  --model /Users/${USER}/.lmstudio/models/unsloth/Qwen3.6-27B-GGUF/Qwen3.6-27B-Q8_0.gguf \
  --model-draft /Users/${USER}/.lmstudio/models/Anbeeld/Qwen3.6-27B-DFlash-GGUF/Qwen3.6-27B-DFlash-Q4_K_M.gguf \
  --spec-type dflash \
  --spec-dflash-cross-ctx 1024 \
  --port 8080 \
  -np 1 \
  --kv-unified \
  -ngl all \
  --spec-draft-ngl all \
  -b 2048 -ub 512 \
  --ctx-size 102400 \
  --cache-type-k q4_1 --cache-type-v q4_1 \
  --flash-attn on \
  --jinja \          
  --chat-template-file /Users/${USER}/Documents/code/chat_template.jinja \          
  --no-mmap --mlock \
  --reasoning on \
  --temp 0.6 --top-k 20 --top-p 1.0 --min-p 0.0 --host 0.0.0.0 --spec-draft-ngl all
```

### Qwen 3.7 27B dense
- reliable, but heavy and slow so we use turboquant
- b/c of this, we can fit the full context in mem!
- should try adding   `-ctxcp 48 -cms 2048 \` to force more cache reuse, (if it invalidates cache?)
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

### Qwenopus 3.6 UDT MTP With Turboquant
- if this does not work look at AtomicChat, this is more up-to-date it seems (but the improvements in tok/s seem not so great)
```
git clone https://github.com/TheTom/llama-cpp-turboquant
cd llama-cpp-turboquant
cmake --build build --clean-first
cmake -B build -DGGML_METAL=ON -DGGML_METAL_EMBED_LIBRARY=ON -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release -j
```

```
~/Documents/code/llama-cpp-turboquant/build/bin/llama-server --model /Users/${USER}/.lmstudio/models/Jackrong/Qwopus3.6-27B-Coder-COMPAT-MTP-GGUF/Qwopus3.6-27B-Coder-Compat-MTP-Q8_0.gguf \
  -c 262144 \
  -b 2048 \
  -ub 2048 \
  -ngl 999 \
  --temp 0.9 \
  --top-p 0.90 \
  --top-k 20 \
  --min-p 0.0 \
  --presence-penalty 0.0 \
  --repeat-penalty 1.0 \
  --chat-template-file /Users/${USER}/Documents/code/chat_template.jinja \
  --spec-draft-n-max 3 \
  --jinja \
  --cache-type-k q8_0 \
  --cache-type-v turbo4 \
  --cache-prompt \
  --host 0.0.0.0 \
  --port 8080 \
  -fa on \
  -np 1 \
  --metrics
```
### AtomicChat with turboquant
Trying to get the best of all worlds, as dense turboquant gets bogged down w/ large context (2 tok/s)
- this repo is in a bad state, referencing a decoding type (nextn) that does not exist
  - https://huggingface.co/AtomicChat/Qwen3.6-27B-UDT-MTP-GGUF
  - https://github.com/AtomicBot-ai/atomic-llama-cpp-turboquant/tree/feature/turboquant-kv-cache
- Setup:
1. `hf download AtomicChat/Qwen3.6-27B-UDT-MTP-GGUF Qwen3.6-27B-UDT-Q8_K_XL_MTP.gguf --local-dir .`
2. `cd ~/Documents/code/`
3. Download, build, and compile the fork (for macOS metal)
```
git clone https://github.com/AtomicBot-ai/atomic-llama-cpp-turboquant.git
cd atomic-llama-cpp-turboquant
git checkout feature/turboquant-kv-cache
cmake --build build --clean-first
cmake -B build -DGGML_METAL=ON -DGGML_METAL_EMBED_LIBRARY=ON -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release -j
```
4. Can now be run from `~/Documents/code/atomic-llama-cpp-turboquant/build/bin/llama-server`
- --draft-p-min 0.80 set to reduce time at high contexts (maybe?) 
- does not look like we actually need the --model-draft? it does it itself now?
  - removed in current call (see below)
- `nextn` is potentially hallucinated at the repo level, or not implemented yet (but documented!)
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
      - in this repo, we use: `-ctxcp 48 -cms 2048`
      - limited testing suggests that this does improve prompt processing by not clearing all of the previous context in the llama-server output
      - Yes, it seems to recover a lot of the context, in simple testing ~40-90% tested on up to 40k context
    - $$\text{Checkpoint Size (Bytes)} = 2 \times \text{Layers} \times \text{Embedding Dim} \times \text{Head Dim} \times \text{Precision (Bytes)}$$
      - Size (MB) = 2*64*5124*128*2 (/(1024*1024) for MiB --> 167.90 MB)
        - $+$ 1.5KB * current token position for data that keeps track of the checkpoints (e.g., $150\text{ MiB} + (114,545 \times 0.001438\text{ MiB}) = \mathbf{314.7\text{ MiB}}$ (where MiB is 1024*1024 bytes)
      - This is a balance though, as it takes up more and more mem!
      - 
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
  -ctxcp 48 \
  -cms 2048 \
  --cache-reuse 256 \
  --port 8080
```
alternative (current testing):
```
/Users/${USER}/Documents/code/atomic-llama-cpp-turboquant/build/bin/llama-server \
  -m /Users/${USER}/.lmstudio/models/AtomicChat/Qwen3.6-27B-UDT-MTP-GGUF/Qwen3.6-27B-UDT-Q8_K_XL_MTP.gguf \
  --ctx-size 262144 \
  -b 2048 \
  -ub 2048 \
  -ngl 999 \
  -fa on \
  --spec-type draft-mtp \
  --spec-draft-n-max 2 \
  --draft-p-min 0.80 \
  --cache-type-k q8_0 \
  --cache-type-v turbo3 \
  --cache-type-k-draft q8_0 \
  --cache-type-v-draft turbo3 \
  --temperature 0.6 \
  --jinja \
  --chat-template-file /Users/${USER}/Documents/code/chat_template.jinja \
  --host 0.0.0.0 \
  --port 8080 \
  -ctxcp 48 \
  -cms 2048 \
  --metrics --mlock
```

### QwenOpus 3.6 CODER COMPAT MTP (JackRong)
- supposed to fix looping, but does not seem to entirely
- use of new chat template seems to help, but not extensively tested
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
  --chat-template-file /Users/${USER}/Documents/code/chat_template.jinja \
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
  --chat-template-file /Users/${USER}/Documents/code/chat_template.jinja \
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


