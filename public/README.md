# Nihongo Path AI Lab
Independent browser-only Japanese tutor with Bengali explanations. No inference API, account, backend or quota. Download bandwidth, device storage, battery and RAM are still needed. Educational experiment: model accuracy is not guaranteed.

## Run
Node 24: `npm ci --ignore-scripts`, then `npm run dev -- --host 127.0.0.1`. Production: `npm run build`; serve `dist` over HTTPS (or localhost). Open Model, inspect exact file size, choose WebGPU or experimental WASM, then explicitly download. No weights download before that action. Enter a question after Ready appears. Stop interrupts generation; Unload terminates the worker and frees its resources.

## Model
Primary is **Qwen3.5-0.8B**, text-only ONNX conversion `onnx-community/Qwen3.5-0.8B-Text-ONNX`, pinned revision `25cdf33f386b487a25f3d743e86442517bdb77ff`, q4. Original model: https://huggingface.co/Qwen/Qwen3.5-0.8B (Apache-2.0). Conversion: https://huggingface.co/onnx-community/Qwen3.5-0.8B-Text-ONNX . q4 graph 482,783 bytes + external weights 550,764,544 bytes; tokenizer/config are extra. Transformers.js 4.3.2 implements qwen3_5_text; ONNX Runtime executes in the browser worker. Native llama.cpp is not embedded.

Optional **explicitly selected fallback**: Qwen2.5-0.5B-Instruct ONNX q4. It is never selected automatically. Its language quality may be weaker. Both model identities are visible in the UI and assistant messages.

## Privacy / offline
Chats are in this origin's localStorage. Do not enter sensitive information on a shared device. Prompts are not sent to an inference server. Hugging Face receives model downloads/metadata requests. A Google font is optional and uses a system-font fallback. Model files use Transformers.js Cache Storage; the service worker caches visited app assets. Offline is conditional on full caches surviving browser eviction, including the worker and WASM. Test offline only after one complete successful generation. Clear browser site data to remove weights and chats.

## Test
Open Model, verify size and no automatic weight download. Load chosen backend. Try: 〜ながら; 勉強; 〜てしまう; correction of 私は昨日学校に行きます; N3 〜ようにする; 学 readings; 昨日、図書館で本を読みました translation. Check Japanese and Bengali manually. Record tok/s and browser memory separately; JS heap is not total model/GPU memory. See REPORT.md for actual results and limitations.

GitHub Actions builds and deploys to Pages after Pages is configured to use GitHub Actions. No other repository or production app is connected.
