# Overview
- for mac M1, with 32 GB of RAM
# General Installation
0. Metal setup
Set dedicated GPU memory (assuming 32GB RAM)
```
sudo sysctl iogpu.wired_limit_mb=28000 #leave 4 GB for system
```

2. Basic command-line tools and llama.cpp (official)
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew install python -y
echo 'export PATH="/opt/homebrew/opt/python@3.14/libexec/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
# cmake (to build code)
brew install cmake -y
# grep is out of date or not quite correct
brew install grep -y
PATH="/opt/homebrew/opt/grep/libexec/gnubin:$PATH"
# Hugging face cli
curl -LsSf https://hf.co/cli/install.sh | bash

ROOT_DIR=~/Documents/code/llm_tools
mkdir -p ${ROOT_DIR}
cd ${ROOT_DIR}
#latest release of llama.cpp (official repo)
LLAMA_TAG=$(curl -s https://api.github.com/repos/ggml-org/llama.cpp/releases/latest | grep -Po '"tag_name": "\K[^"]+')
echo $LLAMA_TAG
curl -s https://api.github.com/repos/ggml-org/llama.cpp/releases/latest | \
  grep -o '"browser_download_url": "[^"]*macos-arm64[^"]*"' | \
  cut -d '"' -f 4 | \
  xargs curl -L -O
tar -xzf llama-${LLAMA_TAG}-bin-macos-*.tar.gz
echo 'export PATH="/Users/csteele/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
ln -s ${ROOT_DIR}/llama-${LLAMA_TAG}/llama* ~/.local/bin/.

```

3. Chat template
```

ROOT_DIR=~/Documents/code/llm_tools
CHAT_TEMPLATE_DIR=${ROOT_DIR}/chat_templates
mkdir -p ${CHAT_TEMPLATE_DIR}
HF_TEMPLATE_LOC="froggeric/Qwen-Fixed-Chat-Templates"
TEMPLATE_TAG=${HF_TEMPLATE_LOC////-}
hf download ${HF_TEMPLATE_LOC} chat_template.jinja --local-dir ${CHAT_TEMPLATE_DIR}/${TEMPLATE_TAG}_chat_template.jinja

```
# Forge Guardrails
Can improve small model output with nudges etc, providing much higher output quaulity at the cost of time
- install uv
- create venv, source it
  - `uv venv --python 3.13`
  - `source .venv/bin/activate`
- `uv pip install forge-guardrails`
- run it after starting model, point opencode at it
  - `forge-proxy --backend-url http://localhost:8080 --port 8081`
# Llama.cpp versions

## Beellama llama.cpp version
- Allows us access to both turboquant and speculative decoding
```
ROOT_DIR=~/Documents/code/llm_tools
cd ${ROOT_DIR}
git clone https://github.com/Anbeeld/beellama.cpp.git
cd beellama.cpp
cmake -B build -DGGML_METAL=ON -DGGML_METAL_EMBED_LIBRARY=ON -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release -j
```

## AtomicChat llama.cpp version
Not very stable, and some of it seems to be vaporware
```
ROOT_DIR=~/Documents/code/llm_tools
cd ${ROOT_DIR}
git clone https://github.com/AtomicBot-ai/atomic-llama-cpp-turboquant.git
cd atomic-llama-cpp-turboquant
git checkout feature/turboquant-kv-cache
cmake -B build -DGGML_METAL=ON -DGGML_METAL_EMBED_LIBRARY=ON -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release -j
```
# Model(s)
```
MODEL_DIR=${ROOT_DIR}/models
mkdir -p ${MODEL_DIR}

HF_PROVIDER_MODEL_TAG=AtomicChat/Qwen3.6-35B-A3B-UDT-MTP-GGUF
GGUF_TAG=Qwen3.6-35B-A3B-UDT-Q4_K_XL_MTP.gguf #smaller model to fit in mem
mkdir -p ${MODEL_DIR}/${HF_PROVIDER_MODEL_TAG}

hf download ${HF_PROVIDER_MODEL_TAG} ${GGUF_TAG} --local-dir ${MODEL_DIR}/${HF_PROVIDER_MODEL_TAG}
```
## Running models
6. Run model with Beellama
```
ROOT_DIR=~/Documents/code/llm_tools
${ROOT_DIR}/beellama.cpp/build/bin/llama-server \
  -m ${ROOT_DIR}/models/unsloth/Qwen3.6-35B-A3B-MTP-GGUF/Qwen3.6-35B-A3B-MTP-GGUF \
  --spec-type draft-mtp \
  --spec-draft-n-max 3 \
  --temperature 0.6
  --top-p 0.95
  --min-p 0.0
  --top-k 20
  --presence-penalty 0.0
  --repetition-penalty 1.1
  -np 1
  -c 131072 \
  -b 2048 \
  -ub 512 \
  -ngl 999 \
  -fa on \
  --cache-type-k q8_0 \
  --cache-type-v q6_0 \
  --cache-type-k-draft q8_0 \
  --cache-type-v-draft q6_0 \
  --host 0.0.0.0 \
  -ctxcp 48 \
  -cms 2048 \
  --cache-reuse 256 \
  --cache-prompt
  --chat-template-file CHAT_TEMPLATE_DIR=${ROOT_DIR}/chat_templates/froggeric-Qwen-Fixed-Chat-Templates_chat_template.jinja \
  --jinja \
  --port 8080

```
6. Run model with AtomicChat

```
${ROOT_DIR}/atomic-llama-cpp-turboquant/build/bin/llama-server \
  -m ${ROOT_DIR}/models/AtomicChat/Qwen3.6-35B-A3B-UDT-MTP-GGUF/Qwen3.6-35B-A3B-UDT-Q4_K_XL_MTP.gguf \
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
