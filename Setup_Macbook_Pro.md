# Overview
- for mac M1, with 32 GB of RAM
# General Installation

1. Basic command-line tools and llama.cpp (official)
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew install python
# cmake (to build code)
brew install cmake
# Hugging face cli
curl -LsSf https://hf.co/cli/install.sh | bash

mkdir -p ~/Documents/code/llm_tools

#latest release of llama.cpp (official repo)
LLAMA_TAG=$(curl -s https://api.github.com/repos/ggml-org/llama.cpp/releases/latest | grep -Po '"tag_name": "\K[^"]+')
echo $LLAMA_TAG
curl -s https://api.github.com/repos/ggml-org/llama.cpp/releases/latest | \
  grep -o '"browser_download_url": "[^"]*macos-arm64[^"]*"' | \
  cut -d '"' -f 4 | \
  xargs curl -L -O
tar -xzf llama-${LLAMA_TAG}-bin-macos-*.tar.gz
ln -s /Users/${USER}/Documents/code/llm_tools/llama-${LLAMA_TAG}/llama* ~/.local/bin/.

```

3. Chat template
```
TEMPLATE_LOC="froggeric/Qwen-Fixed-Chat-Templates"
TEMPLATE_TAG=${TEMPLATE_LOC////-}
hf download ${TEMPLATE_LOC} chat_template.jinja --local-dir ~/Documents/code/llm_tools/chat_templates/_chat_template.jinja
```
4. Model(s)
```
HF_PROVIDER_MODEL_TAG=AtomicChat/Qwen3.6-35B-A3B-UDT-MTP-GGUF
GGUF_TAG=Qwen3.6-35B-A3B-UDT-Q8_K_XL_MTP.gguf
```
