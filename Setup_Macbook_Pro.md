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
brew install python
# cmake (to build code)
brew install cmake
# Hugging face cli
curl -LsSf https://hf.co/cli/install.sh | bash

ROOT_DIR=~/Documents/code/llm_tools
mkdir -p ${ROOT_DIR}

#latest release of llama.cpp (official repo)
LLAMA_TAG=$(curl -s https://api.github.com/repos/ggml-org/llama.cpp/releases/latest | grep -Po '"tag_name": "\K[^"]+')
echo $LLAMA_TAG
curl -s https://api.github.com/repos/ggml-org/llama.cpp/releases/latest | \
  grep -o '"browser_download_url": "[^"]*macos-arm64[^"]*"' | \
  cut -d '"' -f 4 | \
  xargs curl -L -O
tar -xzf llama-${LLAMA_TAG}-bin-macos-*.tar.gz
ln -s ${ROOT_DIR}/llama-${LLAMA_TAG}/llama* ~/.local/bin/.

```

3. Chat template
```
# ROOT_DIR=~/Documents/code/llm_tools
CHAT_TEMPLATE_DIR=${ROOT_DIR}/chat_templates
mkdir -p ${CHAT_TEMPLATE_DIR}
HF_TEMPLATE_LOC="froggeric/Qwen-Fixed-Chat-Templates"
TEMPLATE_TAG=${HF_TEMPLATE_LOC////-}
hf download ${HF_TEMPLATE_LOC} chat_template.jinja --local-dir ${TEMPLATE_DIR}/${TEMPLATE_TAG}_chat_template.jinja

```

4. Model(s)
```
MODEL_DIR=${ROOT_DIR}/models
mkdir -p ${MODEL_DIR}

HF_PROVIDER_MODEL_TAG=AtomicChat/Qwen3.6-35B-A3B-UDT-MTP-GGUF
GGUF_TAG=Qwen3.6-35B-A3B-UDT-Q6_K_XL_MTP.gguf #smaller model to fit in mem
mkdir -p ${MODEL_DIR}/${HF_PROVIDER_MODEL_TAG}

hf download ${HF_PROVIDER_MODEL_TAG} ${GGUF_TAG} --local-dir ${MODEL_DIR}/${HF_PROVIDER_MODEL_TAG}
```

5. AtomicChat llama.cpp version
```
cd ${ROOT_DIR}
git clone https://github.com/AtomicBot-ai/atomic-llama-cpp-turboquant.git
cd atomic-llama-cpp-turboquant
git checkout feature/turboquant-kv-cache
cmake -B build -DGGML_METAL=ON -DGGML_METAL_EMBED_LIBRARY=ON -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release -j
```

6. Run model with AtomicChat

```

```
