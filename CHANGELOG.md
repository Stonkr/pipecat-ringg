# Changelog

All notable changes to the Ringg AI STT integration for Pipecat.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2026-05-19

### Added

- Initial release of `RinggSTTService`, a streaming Speech-to-Text
  integration backed by the official [`ringglabs`](https://pypi.org/project/ringglabs/)
  SDK.
- SDK-managed WebSocket streaming (connection, handshake, and event parsing
  delegated to `ringglabs`).
- Streaming interim (`InterimTranscriptionFrame`) and final
  (`TranscriptionFrame`) transcripts.
- Client-driven VAD endpointing: forwards Pipecat
  `VADUserStartedSpeakingFrame` / `VADUserStoppedSpeakingFrame` to the
  server as `start_speaking` / `stop_speaking` cues for `on_final` mode.
- Server-side capitalization and punctuation support.
- Per-utterance processing metrics and STT tracing support.
- Configurable language, encoding, mode, and server VAD parameters via
  `RinggSTTParams`.
- Foundational example (`example.py`) and unit tests.

### Compatibility

- Tested with Pipecat v1.2.1.
