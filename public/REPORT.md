# Compatibility and actual test report
Date: 2026-11-12. Status: implementation in verification; do not interpret source integration as a passed inference test.

## Architecture investigation
- Transformers.js 4.3.2 source registry contains qwen3_5_text → Qwen3_5ForCausalLM. Primary text-only ONNX config declares that architecture. Selected because it omits the unnecessary vision encoder.
- WebGPU is the preferred browser execution provider, but API presence alone does not establish adapter / operator compatibility.
- WASM is offered as an explicit experimental backend, one thread, without requiring SharedArrayBuffer or cross-origin isolation. Operator availability and speed must be measured.
- WebLLM requires an MLC-compiled supported model artifact; original Hugging Face weights or GGUF are not drop-in inputs. Not selected.
- llama.cpp is native; browser operation requires a dedicated WASM port (such as wllama) and compatible GGUF build. Native engine is not inserted into this site.
- ONNX q4 primary graph: 482,783 bytes; weights: 550,764,544 bytes. Actual total tokenizer/config download is obtained from Hugging Face metadata before consent. Working memory exceeds download size.

## Device matrix (physical devices not available)
| RAM | Current status | Guidance |
|---|---|---|
| 2 GB | Not verified; high risk | Browser may kill tab; no support guarantee |
| 3 GB | Not verified; experimental | Close other tabs, 512/1024 context |
| 4 GB | Not verified | Try WebGPU, 1024 context |
| 6 GB | Not verified | Try WebGPU, 1024/2048 context |

Context counts actual tokenizer input plus output reserve. Only one generation can run. History is trimmed from the oldest messages; an oversized last question is rejected. The model runs in a worker; Unload terminates it. Browser-wide OOM cannot be caught reliably. No fabricated RAM or speed readings are shown.

## Automated checks so far
- npm production build: PASS.
- Architecture registry/config inspection: PASS (source compatibility, not inference proof).
- Attempt to launch standalone Chromium with Playwright: blocked by executor socket() Operation not permitted before page load. Not a model failure.
- Shared Chrome cannot reach the executor's localhost URL (connection refused); deployed-browser test still required.
- Japanese/Bengali/kanji accuracy, inference speed and actual peak RAM: not measured yet.
- Physical Android tests: none. Mobile viewport checks cannot establish Android hardware support.

No prewritten assistant answers exist. Send remains disabled until real model load succeeds. Errors do not trigger an undisclosed model substitution.

## Browser observations (shared desktop Chrome)
- Public Pages URL loaded and rendered the complete interface; Send disabled before model readiness.
- Primary metadata size shown: 570,493,329 bytes, including tokenizer/config and q4 graph/weights (runtime extra).
- WebGPU API present, but requestAdapter returned no adapter. No WebGPU inference was executed.
- Primary q4 weights downloaded to 550.8 MB. WASM session creation FAILED with: `Could not find an implementation for GatherBlockQuantized(1) ... /model/embed_tokens/Gather_Quant`.
- This is a verified operator blocker in the selected ONNX Runtime WASM build. The updated UI blocks primary+WASM before downloading and asks users to explicitly choose a backend/model. Primary WebGPU remains an unverified candidate, not a supported-device claim.
- Explicit fallback selected for further testing: Qwen2.5-0.5B-Instruct q4, revision 22942cb7d7ba4cc81bb4673549ca4d4614469b5e. Model/config/tokenizer total 797,647,451 bytes. This export is larger than the primary despite fewer parameters; it is not advertised as a smaller download.

## Sources
- https://huggingface.co/Qwen/Qwen3.5-0.8B
- https://huggingface.co/onnx-community/Qwen3.5-0.8B-Text-ONNX
- https://github.com/huggingface/transformers.js/releases/tag/4.3.2
- https://huggingface.co/docs/transformers.js/main/en/guides/webgpu
- https://webllm.mlc.ai/docs/user/basic_usage.html
- https://github.com/ggml-org/llama.cpp

## Actual Qwen2.5 WASM generation
Model loaded successfully. Test prompt: “What does 勉強 mean? Give its hiragana and explain briefly in Bengali.” Output began `Bengali: さあ (Sāa) / Hiragana: さあ (Sāa) / English: 力争力 ...`. This is WRONG: 勉強 is べんきょう, meaning study. Bengali requirement failed. Observed 35 tokens at 63.2s (~0.6 tok/s), then 56 tokens at 98.2s; the initial cooperative Stop did not interrupt WASM promptly. Updated Stop now terminates the worker, retains partial output, and requires reload before continuing; this is an observed desktop WASM result, not an Android benchmark. Do not use this fallback as an authoritative tutor. More language tests have not passed. It does prove genuine local generation, not teaching quality.

Qwen3 0.6B q4 is added as a separately named experimental fallback for investigation; its graph is 919,096,585 bytes, so fewer parameters does not mean a smaller download with this export. No substitution happens automatically.
