# Compatibility and actual test report
Date: 2026-11-12. Status: public experimental website deployed; actual browser inference demonstrated with explicit fallbacks. Japanese–Bengali tutor accuracy has NOT passed validation. Primary Qwen3.5 WebGPU remains unverified.

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
- Initial pre-deployment checks did not measure language quality. Subsequent real generation observations are recorded below. Actual peak RAM remains unmeasured.
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

## Offline / UI verification limits
The updated production build precaches the small app shell and report, and runtime/worker assets are cached when fetched after explicit Load. Metadata is retained for an offline reopen. Full offline reload has not been tested in the shared browser; cache eviction remains possible. History restoration after page reload was observed. Theme, mobile drawer, context controls and error states are implemented; physical Android hardware has not been tested. The streaming renderer updates only the current answer instead of rebuilding the entire conversation per token.


## Actual Qwen3 0.6B WASM test
- Model downloaded and loaded successfully in shared desktop Chrome. Metadata total: 932,679,247 bytes; q4 graph: 919,096,585 bytes.
- Prompt: `勉強 এর বাংলা অর্থ ও হিরাগানা কী? সংক্ষেপে বলো।`
- Partial streamed output: `বাংলার অর্থ এর হিরাগানা কী` (no useful answer yet). Observed 31 tokens at 83.9s, then 34 at 91.0s (~0.4 tok/s).
- Stopped before full completion because of slow, unhelpful output. This is NOT a completed accuracy benchmark and is not evidence of a correct Bengali explanation. No usable answer was obtained in this short attempt.
- Updated hard Stop immediately showed “Stopped · model unloaded. Reload cached model to continue.” Send stayed disabled after unloading.

## Requested learning checks
| Check | Primary Qwen3.5 | Qwen2.5 fallback | Qwen3 fallback |
|---|---|---|---|
| 勉強 meaning / hiragana / Bengali | WASM blocked; WebGPU unavailable here | Incorrect streamed content, stopped | No usable answer before stop; incomplete |
| 〜ながら explanation | Not run | Not run after failed basic check | Not run |
| 〜てしまう examples | Not run | Not run | Not run |
| Sentence correction | Not run | Not run | Not run |
| N3 grammar | Not run | Not run | Not run |
| 学 onyomi / kunyomi | Not run | Not run | Not run |
| Japanese → Bengali translation | Not run | Not run | Not run |

No claim is made that these educational checks passed. The next technical requirement for primary-model validation is a real WebGPU adapter on a compatible device. No physical Android tests were performed. Small local models should not be presented as verified N5–N1 teachers based on a successful download alone.

## Final scope and deployment
Only the new Mr04Nazmul/nihongo-path-ai-lab repository was created/modified. GitHub Pages public URL was opened successfully in real Chrome: https://mr04nazmul.github.io/nihongo-path-ai-lab/ . The source includes exact runtime dependencies, pinned model revisions, mobile CSS, Bengali-capable fonts, themes, persistent conversations, streaming, download consent, file progress, context/token controls, worker termination, and conditional offline caching. No inference backend, paid API, production database, existing app, authentication, ads, rewards or user data is connected. This is a real local-inference experiment; educational quality and physical-device compatibility remain blockers to calling it a production-ready Japanese teacher.
