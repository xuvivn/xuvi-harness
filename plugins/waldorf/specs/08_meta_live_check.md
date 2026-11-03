# Meta live integration evidence

Date: 2026-11-03. Executed by orchestrator with the user-supplied `.env`, secret kept in process memory only. These checks verify provider access directly; application-route checks are separate in `04_verification.md`.

- `GET https://api.meta.ai/v1/models`: success; includes `muse-wonder-1.0`, `muse-spark-1.3`, `muse-image-1.0`, `muse-voice-transcribe-1.0`, `sam-3.1`.
- `POST /v1/chat/completions`, `muse-spark-1.3`: success, Vietnamese response to a generic story-preparation sentence.
- `POST /v1/asr/transcribe`, `muse-voice-transcribe-1.0`: success with multipart `request` JSON and `audio` WAV. Test audio synthesized locally with macOS Linh voice, converted by ffmpeg to mono 24kHz PCM16; no personal recording used. Audio duration 8,640ms. Vietnamese transcript correctly recovered the sentence about a storyteller, a seed, watering and a first leaf. Returned turn timestamps 220–8,140ms.
- `POST /v1/images/generations`, `muse-image-1.0`: success, 433,232-byte WebP watercolor garden illustration (sniffed from actual bytes, not assumed PNG). Visually reviewed. A sample is stored at `client/public/images/garden-watercolor.webp` for app presentation. It is an illustration, not a student's work or evidence of learning.

Provider references checked directly: https://dev.meta.ai/docs/models, https://dev.meta.ai/docs/speech-to-text, https://dev.meta.ai/docs/image-generation. The ASR endpoint requires mono PCM16 WAV at 16kHz or 24kHz; browser WebM must be converted. User input goes only to the explicitly invoked provider action, not the entire repository.
