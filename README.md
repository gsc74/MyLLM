<div align="center">

# MyLLM-1B

**A tiny, general-purpose math assistant that runs on your laptop.**

[![GGUF](https://img.shields.io/badge/format-GGUF-blue)](#download) [![Params](https://img.shields.io/badge/params-1.05B-green)](#model-summary) [![Runs on](https://img.shields.io/badge/runs%20on-LM%20Studio%20%7C%20llama.cpp-orange)](#usage) [![License](https://img.shields.io/badge/license-Apache--2.0-lightgrey)](#license)

</div>

> [!NOTE]
> Educational outcome of the tutorial **[How to train your first LLM?](https://gsc74.github.io/blogs/train_index.html)**.
> It's a **basic high-school math model**.

## Model Summary

A **1.05B-parameter** decoder-only model for basic math: pretrained on open math web text, SFT on math instructions, then aligned (DPO + GRPO).

| | |
|---|---|
| Parameters | 1.055B |
| Architecture | Llama-style, 28 layers, hidden 1792 |
| Attention | 14 query / 2 KV heads (GQA), head dim 128 |
| FFN / Norm | SwiGLU (4864) / RMSNorm |
| Positional | RoPE (θ = 500,000), 8,192 ctx |
| Vocabulary | 65,536 (byte-level BPE) |

## Download

| Format | Use for | Link |
|---|---|---|
| GGUF (BF16, ~2 GB) | LM Studio / llama.cpp | [Release asset](https://github.com/gsc74/MyLLM/releases/latest/download/MyLLM-1B-BF16.gguf) |
| Transformers (safetensors) | Python / PyTorch | [MyLLM-1B-HF/](https://github.com/gsc74/MyLLM/tree/main/MyLLM-1B-HF) + [weights from Release](https://github.com/gsc74/MyLLM/releases/latest) |

> The model weights (the `.gguf` and `.safetensors` files, ~2 GB each) are attached as **GitHub Release assets**. For the Transformers version, clone the repo and drop the `model-0000*.safetensors` files from the Release into `MyLLM-1B-HF/`.

## Usage

> [!IMPORTANT]
> Run with **temperature ≈ 0.7** and a **repeat penalty ≈ 1.3**. Pure greedy decoding makes this small model loop; these settings keep answers concise and let it stop cleanly.

### LM Studio (easiest, for laptops)

[LM Studio](https://lmstudio.ai) is a free desktop app (macOS, Windows, Linux) with a chat GUI.

1. **Install** LM Studio from [lmstudio.ai](https://lmstudio.ai) and open it.
2. **Add the model:** download `MyLLM-1B-BF16.gguf` from the [Release](https://github.com/gsc74/MyLLM/releases/latest), then in LM Studio go to **My Models -> Import** (or drop the `.gguf` into the LM Studio models folder).
3. **Load it:** open the chat tab, pick **MyLLM-1B** from the model selector at the top, and wait for it to load. The embedded chat template and stop token (`<|end|>`) are picked up automatically.
4. **Set sampling** in the right-hand panel: **Temperature 0.7**, **Repeat Penalty 1.3** (Top P 0.9, Top K 40), then chat.

### llama.cpp

```bash
llama-cli -m MyLLM-1B-BF16.gguf --jinja \
  -p "What is 2+2?" \
  --temp 0.7 --top-p 0.9 --top-k 40 --repeat-penalty 1.3
```

### Transformers (PyTorch)

```python
from transformers import AutoModelForCausalLM, AutoTokenizer

path = "MyLLM-1B-HF"
tok = AutoTokenizer.from_pretrained(path, trust_remote_code=True)
model = AutoModelForCausalLM.from_pretrained(path, trust_remote_code=True, torch_dtype="auto")

msgs = [{"role": "user", "content": "What is 2+2?"}]
inputs = tok.apply_chat_template(msgs, add_generation_prompt=True, return_tensors="pt", return_dict=True)
out = model.generate(**inputs, max_new_tokens=128, temperature=0.7, repetition_penalty=1.3)
print(tok.decode(out[0][inputs["input_ids"].shape[1]:], skip_special_tokens=True))
```

## Example

**Prompt:** `Give me the integration of x^2.`

```
The integral is: ∫ x² dx

So, we have: (1/3) x³ + C₀

Therefore: \boxed{\frac{1}{3}x^{3}}
```

## Training Data & Licenses

| Stage | Dataset | License |
|---|---|---|
| Pretraining | [OpenWebMath](https://huggingface.co/datasets/open-web-math/open-web-math) | ODC-By 1.0 |
| SFT | [MetaMathQA](https://huggingface.co/datasets/meta-math/MetaMathQA) | MIT |
| SFT | [NuminaMath-CoT](https://huggingface.co/datasets/AI-MO/NuminaMath-CoT) | Apache-2.0 |
| SFT | [OpenMathInstruct-2](https://huggingface.co/datasets/nvidia/OpenMathInstruct-2) | CC-BY-4.0 |
| GRPO reward | [GSM8K](https://huggingface.co/datasets/openai/gsm8k) | MIT |

## License

Weights and code are released under **Apache-2.0**. Use must also comply with the dataset licenses above (including attribution for OpenWebMath and OpenMathInstruct-2).

## Acknowledgements

Capstone artifact of my tutorial **[How to train your first LLM?](https://gsc74.github.io/blogs/train_index.html)**.
