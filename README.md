# Real-Time Speech-to-Text Streaming API
## Product Requirements Document (PRD)

### 1. Overview
#### 1.1 Product Purpose
To provide a high-performance, real-time speech-to-text API with low latency (< 200ms) that supports continuous audio streaming, voice activity detection, and wake word functionality.

# 1. WebSocket Channels
CONTROL_CHANNEL = "wss://api.example.com/v1/stream/control"
AUDIO_CHANNEL = "wss://api.example.com/v1/stream/audio"

# 2. Configuration Schema

# REQUIRED PARAMETERS
REQUIRED_CONFIG = {
    "language": "en",              # Language code
    "sample_rate": 16000,         # Audio sample rate
    "encoding": "PCM"             # Audio encoding format
}

# DEFAULT CONFIGURATION
DEFAULT_CONFIG = {
    # Core Settings
    "model": "base",              # tiny, base, small, medium, large-v1, large-v2
    "compute_type": "default",    # default, int8, float16
    "device": "cuda",             # cuda or cpu
    "gpu_device_index": 0,
    
    # Audio Processing
    "channels": 1,
    "buffer_size": 512,
    "handle_buffer_overflow": True,
    "allowed_latency_limit": 100,
    
    # Voice Activity Detection (VAD)
    "vad": {
        "enabled": True,
        "silero": {
            "enabled": True,
            "sensitivity": 0.4,
            "use_onnx": False
        },
        "webrtc": {
            "enabled": True,
            "sensitivity": 3
        },
        "post_speech_silence": 0.6,
        "min_speech_duration": 0.5
    },

    # Wake Word Detection
    "wake_word": {
        "enabled": False,
        "backend": "pvporcupine",      # pvporcupine or openwakeword
        "words": [],                   # List of wake words
        "sensitivity": 0.6,
        "activation_delay": 0.0,
        "timeout": 5.0,
        "buffer_duration": 0.1
    },

    # Enhanced Features
    "features": {
        # Transcription Features
        "interim_results": True,
        "speaker_diarization": False,
        
        # Analysis Features
        "sentiment_analysis": {
            "enabled": False,
            "granularity": "sentence",  # sentence or word
            "min_confidence": 0.6
        },
        "topic_detection": {
            "enabled": False,
            "min_confidence": 0.6
        },
        "text_summarization": {
            "enabled": False,
            "max_length": 100,
            "style": "bullets"          # bullets or paragraph
        },
        "intent_identification": {
            "enabled": False,
            "confidence_threshold": 0.7
        }
    },

    # Content Filtering
    "filters": {
        "profanity": {
            "enabled": False,
            "replacement_char": "*",
            "sensitivity": "medium"      # low, medium, high
        }
    },

    # Output Formatting
    "formatting": {
        "ensure_sentence_starting_uppercase": True,
        "ensure_sentence_ends_with_period": True,
        "print_transcription_time": False
    },

    # Stream Control
    "auto_stop": {
        "enabled": False,
        "timeout": 30,                  # Stop after N seconds of silence
        "max_duration":              # Maximum recording duration
    }
}

# 3. Message Types

## Control Channel Messages

# Start Stream
{
    "command": "start_stream",
    "config": {
        # Required parameters
        "language": "en",
        "sample_rate": 16000,
        "encoding": "PCM",
        
        # Optional features
        "wake_word": {
            "enabled": True,
            "words": ["hey assistant"]
        },
        "features": {
            "speaker_diarization": True,
            "sentiment_analysis": {
                "enabled": True,
                "granularity": "sentence"
            }
        }
    }
}

# Control Commands
{
    "command": "stop_stream"
}
{
    "command": "pause_stream"
}
{
    "command": "resume_stream"
}

## Audio Channel Messages

# Partial Results
{
    "type": "partial_result",
    "data": {
        "text": "partial transcription",
        "confidence": 0.85,
        "timestamp": "2024-03-20T10:30:45.123Z",
        "is_final": false,
        "speaker": "speaker_1",         # if speaker_diarization enabled
        "wake_word_detected": "hey assistant"  # if wake_word enabled
    }
}

# Final Results
{
    "type": "final_result",
    "data": {
        # Core Results
        "text": "final transcription",
        "is_final": true,
        "confidence": 0.95,
        "duration": 10.5,
        "timestamp": {
            "start": "2024-03-20T10:30:35.123Z",
            "end": "2024-03-20T10:30:45.123Z"
        },

        # Speaker Information (if enabled)
        "speakers": {
            "count": 2,
            "details": [
                {
                    "id": "speaker_1",
                    "speaking_time": 6.5,
                    "segments": [...]
                }
            ]
        },

        # Enhanced Analysis (if enabled)
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
                    "mentions": [...]
                }
            ],
            "summary": {
                "style": "bullets",
                "points": [
                    "First key point",
                    "Second key point"
                ]
            },
            "intent": {
                "type": "question",
                "confidence": 0.85
            }
        },

        # Content Filtering Results (if enabled)
        "filters": {
            "profanity": {
                "detected": false,
                "filtered_words": []
            }
        }
    }
}

# Status Updates
{
    "type": "status",
    "data": {
        "event": "recording_start|recording_stop|vad_detect|wake_word_detected",
        "timestamp": "2024-03-20T10:30:45.123Z",
        "details": {
            "wake_word": "hey assistant",  # if applicable
            "auto_stop_reason": "silence_timeout",  # if applicable
            "processing_latency": 150
        }
    }
}
