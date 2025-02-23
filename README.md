# Real-Time Speech-to-Text Streaming API

trade off  between real time and main model : 
1. Real time model is optimized for low latency (< 200ms) and high throughput.
2. Main time model is optimized for high accuracy and low latency.

[Deepgram](https://developers.deepgram.com/docs/live-streaming-audio)
[AssemblyAI](https://www.assemblyai.com/docs/api-reference/streaming/realtime)


## Overview
A high-performance, real-time speech-to-text streaming API delivering transcriptions with ultra-low latency (<200ms).

### Models
- **Real-time Model**: Optimized for low latency (<200ms) and high throughput
- **Main Model**: Optimized for high accuracy with reasonable latency

## WebSocket Endpoints

```json
{
    "CONTROL_CHANNEL": "wss://api.com/v1/stream/control",
    "AUDIO_CHANNEL": "wss://api.com/v1/stream/audio"
}
```

## Authentication Requirements

```json
{
    "REQUIRED_CONFIG": {
        "API_OR_JWT": "Token",
        "client_id": "123",
        "session_id": "session_id"
    }
}
```

## Audio Requirements

```json
{
    "AUDIO_CONFIG": {
        "language": "en",
        "sample_rate": 16000,
        "encoding": "PCM",
        "channels": 1
        "chunk size" : 512.
        "audio_format": {
            "accepted_types": ["raw", "wav", "ogg", "mp3"],
            "streaming_chunk_format": "raw_pcm"
        },
        "streaming_config": {
            "max_chunk_size_bytes": 8192, (need to verify)
            "chunk_duration_ms": 100,(need to verify)
            "continuous_streaming": true(need to verify)
        }
    }
}
    }
}
```

## Configuration Options

```json
{
    "core_settings": {
        "model": "base",
        "compute_type": "default",
        "device": "cuda",
        "gpu_device_index": 0,
        "batch_size": 16,
        "language": "en",
        "initial_prompt": null
    },
    "audio_processing": {
        "channels": 1,
        "buffer_size": 512,
        "sample_rate": 16000,
        "handle_buffer_overflow": true,
        "allowed_latency_limit": 200,
        "input_device_index": null
    },
    "vad": {
        "enabled": true,
        "silero": {
            "enabled": true,
            "sensitivity": 0.4,
            "use_onnx": false
        },
        "webrtc": {
            "enabled": true,
            "sensitivity": 3
        },
        "post_speech_silence": 0.6,
        "min_speech_duration": 0.5
    },
    "features": {
        "interim_results": true,
        "speaker_diarization": false,
        "sentiment_analysis": {
            "enabled": false,
            "granularity": "sentence",
            "min_confidence": 0.6
        },
        "topic_detection": {
            "enabled": false,
            "min_confidence": 0.6
        }
    },
    "formatting": {
        "ensure_sentence_starting_uppercase": true,
        "ensure_sentence_ends_with_period": true,
        "print_transcription_time": false
    }
}
```

## API Communication

### Control Commands

```json
{
    "commands": {
        "start_stream": {
            "command": "start_stream",
            "audio_start": "2024-03-20T10:30:35.123Z"
        },
        "stop_stream": {
            "command": "stop_stream",
            "audio_end": "2024-03-20T10:30:45.123Z"
        },
        "pause_stream": {
            "command": "pause_stream"
        },
        "resume_stream": {
            "command": "resume_stream"
        }
    }
}
```

### Response Messages

#### Partial Results

```json
{
    "type": "partial_result",
    "data": {
        "text": "partial transcription",
        "confidence": 0.85,
        "timestamp": "2024-03-20T10:30:45.123Z",
        "is_final": false,
        "audio_start": "2024-03-20T10:30:35.123Z",
        "audio_current": "2024-03-20T10:30:45.123Z",
        "speaker": "speaker_1",
        "wake_word_detected": "hey assistant"
    }
}
```

#### Final Results

```json
{
    "type": "final_result",
    "data": {
        "text": "final transcription",
        "is_final": true,
        "confidence": 0.95,
        "audio": {
            "start": "2024-03-20T10:30:35.123Z",
            "end": "2024-03-20T10:30:45.123Z",
            "duration": 10.5
        },
        "timestamp": "2024-03-20T10:30:45.123Z",
        "speakers": {
            "count": 2,
            "details": [
                {
                    "id": "speaker_1",
                    "speaking_time": 6.5,
                    "segments": []
                }
            ]
        },
        "analysis": {
            "sentiment": {
                "overall": {
                    "label": "positive",
                    "score": 0.8
                },
                "by_sentence": [
                    {
                        "text": "This is great!",
                        "sentiment": "positive",
                        "score": 0.9
                    }
                ]
            },
            "topics": [
                {
                    "topic": "technology",
                    "confidence": 0.9,
                    "mentions": []
                }
            ]
        },
        "filters": {
            "profanity": {
                "detected": false,
                "filtered_words": []
            }
        }
    }
}
```


#### To Do (Storage of Data)
