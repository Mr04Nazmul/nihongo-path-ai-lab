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
