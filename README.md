# Ringg AI STT Integration for Pipecat

This repository provides an integration of [Ringg AI's](https://ringg.ai) streaming Speech-to-Text (STT) service with the [Pipecat](https://github.com/pipecat-ai/pipecat) framework.

## About

`RinggSTTService` is a **streaming STT integration** that delegates the WebSocket connection, handshake, and event parsing to the official [`ringglabs`](https://pypi.org/project/ringglabs/) Python SDK. It follows Pipecat's **SDK-based streaming STT** pattern (the provider SDK manages the connection internally, the service subclasses `STTService`).

A key feature is **client-driven VAD endpointing**: the service forwards Pipecat's `VADUserStartedSpeakingFrame` / `VADUserStoppedSpeakingFrame` to the server as `start_speaking` / `stop_speaking` cues. With `accept_client_vad_events=True` (the default), the server uses these for endpointing in `on_final` mode instead of relying solely on its own VAD.

**Company Attribution:** This integration is developed and maintained by the **Ringg AI** team, the provider of the underlying STT service. It will be actively maintained alongside the `ringglabs` SDK.

## Features

- ✅ Streaming interim (`InterimTranscriptionFrame`) and final (`TranscriptionFrame`) transcripts
- ✅ Client-driven VAD endpointing via Pipecat VAD frames
- ✅ Server-side capitalization and punctuation
- ✅ Per-utterance processing metrics and STT tracing
- ✅ Configurable language, encoding, mode, and server VAD parameters
- ✅ Robust error handling (API, transport, protocol, and timeout errors)

## Installation

### Install from GitHub

```bash
# With pip
pip install git+https://github.com/Stonkr/pipecat-ringg.git

# With uv (recommended)
uv pip install git+https://github.com/Stonkr/pipecat-ringg.git
```

> A PyPI package (`pip install pipecat-ringg`) will be published after community review.

### Development installation

```bash
git clone https://github.com/Stonkr/pipecat-ringg.git
cd pipecat-ringg

# Editable install with dev + example extras
uv pip install -e ".[dev,example]"
# or
pip install -e ".[dev,example]"
```

## Prerequisites

- Python 3.11 or higher
- A Ringg AI API key — get one at [ringg.ai](https://ringg.ai)

## Usage

### Usage with a Pipecat Pipeline

```python
import os

from pipecat.pipeline.pipeline import Pipeline
from pipecat.audio.vad.silero import SileroVADAnalyzer
from pipecat.audio.vad.vad_analyzer import VADParams
from pipecat.processors.audio.vad_processor import VADProcessor

from pipecat_ringg import RinggSTTParams, RinggSTTService

stt = RinggSTTService(
    base_url=os.environ.get("RINGG_BASE_URL"),  # optional
    params=RinggSTTParams(
        api_key=os.environ["RINGG_API_KEY"],
        language="hi",
        mode="on_final",  # "stream" for interim transcripts
    ),
)

# A VADProcessor upstream of the STT service produces the VAD frames the
# service forwards to the server for endpointing.
pipeline = Pipeline([
    transport.input(),
    VADProcessor(vad_analyzer=SileroVADAnalyzer(params=VADParams(confidence=0.55))),
    stt,
    context_aggregator.user(),
    llm,
    tts,
    transport.output(),
    context_aggregator.assistant(),
])
```

See [`example.py`](example.py) for a complete, runnable example.

### Configuration (`RinggSTTParams`)

| Parameter                   | Type    | Default   | Description                                                                 |
| --------------------------- | ------- | --------- | --------------------------------------------------------------------------- |
| `api_key`                   | `str`   | `""`      | Ringg API key for authentication.                                           |
| `encoding`                  | `str`   | `"int16"` | Audio encoding (signed 16-bit PCM).                                          |
| `language`                  | `str`   | `"hi"`    | Transcription language code.                                                |
| `mode`                      | `str`   | `"stream"`| `"on_final"` emits a final transcript on `stop_speaking`; `"stream"` emits interim transcripts. |
| `vad_tail_sil_ms`           | `int`   | `200`     | Trailing silence (ms) for server VAD.                                       |
| `vad_confidence`            | `float` | `0.55`    | Server VAD confidence threshold (0.0–1.0).                                   |
| `enable_cap_punc`           | `bool`  | `True`    | Enable server-side capitalization/punctuation.                              |
| `accept_client_vad_events`  | `bool`  | `True`    | Use client-sent VAD events for endpointing.                                 |

Constructor parameters:

```python
RinggSTTService(
    base_url: str | None = None,        # optional API base URL override
    sample_rate: int | None = None,     # taken from the StartFrame if omitted
    params: RinggSTTParams | None = None,
)
```

## Running the Example

1. Install dependencies:

   ```bash
   uv pip install -e ".[example]"
   ```

2. Set up your environment:

   ```bash
   cp .env.example .env
   # Edit .env and add your RINGG_API_KEY (and transport credentials)
   ```

3. Run the example:

   ```bash
   python example.py
   # or
   uv run python example.py
   ```

   The example starts a bot that transcribes incoming audio and prints each
   final transcription to the console.

## Compatibility

- **Pipecat version:** Tested with **Pipecat v1.2.1**.
- **Python version:** 3.11+
- **Dependencies:**
  - `pipecat-ai[websockets-base] >= 1.0.0`
  - `ringglabs >= 0.1.0, < 1`
  - `loguru >= 0.7.0`
  - `pydantic >= 2.0.0`

## Support

- **Issues:** [GitHub Issues](https://github.com/Stonkr/pipecat-ringg/issues)
- **Ringg AI:** [ringg.ai](https://ringg.ai)
- **Ringg SDK:** [`ringglabs` on PyPI](https://pypi.org/project/ringglabs/) · [ringglabs-sdk repo](https://github.com/Stonkr/ringglabs-sdk)
- **Pipecat docs:** [docs.pipecat.ai](https://docs.pipecat.ai)

## License

Licensed under the BSD 2-Clause License — see [LICENSE](LICENSE).

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history.

## Acknowledgments

- Thanks to the [Pipecat](https://github.com/pipecat-ai/pipecat) team for the framework and the community integrations program.
