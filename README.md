# 🐠 Babel Fish — Language Translator

> A voice-enabled **English-to-Spanish translator** powered by **Mistral** (via IBM watsonx) for translation, **IBM Watson Speech-to-Text** for voice input, and **IBM Watson Text-to-Speech** for spoken output — inspired by the Babel Fish from *The Hitchhiker's Guide to the Galaxy*.

Built as part of **Course 8 – Building Generative AI-Powered Applications with Python** in the [IBM AI Developer Professional Certificate](https://www.coursera.org/professional-certificates/applied-artifical-intelligence-ibm-watson-ai).

---

## ⚠️ Note on API Keys

> [!IMPORTANT]
> This project was built inside the **IBM Skills Network lab environment**, where API keys and authentication for IBM watsonx and Watson Speech services are **automatically handled by the platform**. The code does not include API keys because they were not required within the lab — the services are pre-configured and accessible without explicit authentication.
>
> If you want to run this project **outside** the lab environment, you will need to:
> 1. Obtain an [IBM watsonx.ai API key](https://cloud.ibm.com/iam/apikeys) and uncomment/set the `API_KEY` variable in `worker.py`
> 2. Set up your own IBM Watson STT/TTS service instances (or use the self-hosted Docker models in the `models/` directory)

---

## 📌 Overview

This project combines three AI services into a seamless translation experience. Users can **type or speak** an English sentence, and the app will:

1. **Transcribe** the voice input to text (if using the microphone)
2. **Translate** the English text to Spanish using Mistral LLM
3. **Speak** the Spanish translation aloud using a selectable voice

### Architecture

```
┌──────────────────────────────────────────────────────────┐
│                        Browser                           │
│  ┌────────────────────────────────────────────────────┐  │
│  │  index.html + style.css + script.js                │  │
│  │  • Type English text OR record via microphone      │  │
│  │  • Voice selection (17 voices, incl. Spanish)      │  │
│  │  • Light/Dark mode toggle                          │  │
│  │  • Audio playback of translated response           │  │
│  └─────────┬────────────────────────┬─────────────────┘  │
│    POST /speech-to-text     POST /process-message        │
│    (audio binary)           (text + voice preference)    │
└────────────┼────────────────────────┼────────────────────┘
             ▼                        ▼
┌──────────────────────────────────────────────────────────┐
│               Flask Backend (server.py)                   │
│                                                          │
│  worker.py                                               │
│  ├── speech_to_text()       ──→ IBM Watson STT API       │
│  ├── watsonx_process_message() ──→ Mistral (IBM watsonx) │
│  └── text_to_speech()       ──→ IBM Watson TTS API       │
└──────────────────────────────────────────────────────────┘
```

---

## 📂 Project Structure

```
Babel-Fish-Language-Translator/
├── Lab - Babel Fish (Language Translator) with LLM, STT, & TTS.pdf
└── translator-with-voice-and-watsonx/
    ├── server.py                 # Flask server — routes for STT, translation, TTS
    ├── worker.py                 # Core logic — Watson STT/TTS + Mistral translation
    ├── Dockerfile                # Docker container configuration
    ├── requirements.txt          # Python dependencies
    ├── example.flac              # Sample audio file (English)
    ├── hola.mp3                  # Sample audio file (Spanish)
    ├── output.wav                # Sample TTS output
    ├── templates/
    │   └── index.html            # Chat UI with voice recording & translation
    ├── static/
    │   ├── script.js             # Frontend — recording, messaging, audio playback
    │   └── style.css             # Light/dark mode styling with animations
    └── models/
        ├── stt/                  # IBM Watson Speech-to-Text Docker model configs
        │   ├── Dockerfile
        │   ├── prepareModels.sh
        │   ├── README.md
        │   └── chuck_var/        # STT configuration (en-US & fr-FR models)
        └── tts/                  # IBM Watson Text-to-Speech Docker model configs
            ├── Dockerfile
            ├── prepareModels.sh
            ├── README.md
            └── config/           # TTS configuration (multiple voice models)
```

---

## 🚀 Getting Started

### Prerequisites

- Python 3.11+
- Access to **IBM watsonx.ai** and **IBM Watson Speech** services (provided in the IBM Skills Network lab environment)

### Run Locally (Lab Environment)

```bash
# Clone the repository
git clone https://github.com/Arhaan-DB47/Babel-Fish-Language-Translator.git
cd Babel-Fish-Language-Translator/translator-with-voice-and-watsonx

# Install dependencies
pip install -r requirements.txt

# Start the server
python server.py
```

The app will be available at `http://0.0.0.0:8000`.

### Run with Docker

```bash
cd Babel-Fish-Language-Translator/translator-with-voice-and-watsonx

# Build the Docker image
docker build -t babel-fish-translator .

# Run the container
docker run -p 8000:8000 babel-fish-translator
```

---

## 📖 How It Works

### Backend

#### `server.py` — Flask Server

Exposes three routes:

| Route | Method | Description |
|-------|--------|-------------|
| `/` | GET | Serves the translator chat interface |
| `/speech-to-text` | POST | Receives raw audio binary, transcribes it via Watson STT |
| `/process-message` | POST | Receives English text + voice preference, translates to Spanish via Mistral, converts translation to speech via Watson TTS |

#### `worker.py` — Translation Pipeline

Contains three core functions:

| Function | Service | What It Does |
|----------|---------|-------------|
| `speech_to_text()` | IBM Watson STT | Sends audio to Watson's `en-US_Multimedia` model, returns transcribed English text |
| `watsonx_process_message()` | Mistral (IBM watsonx) | Sends a strict translation prompt to Mistral: *"Translate the following English sentence into Spanish. Reply ONLY with the translation."* |
| `text_to_speech()` | IBM Watson TTS | Converts the Spanish translation to WAV audio using the user's selected voice |

**Translation prompt design:**
```
Translate the following English sentence into Spanish.
Reply ONLY with the translation, no explanations, no formatting, no extra text.

English: {user_message}
Spanish:
```

**Model parameters:**
- `DECODING_METHOD: GREEDY` — deterministic output for consistent translations
- `MAX_NEW_TOKENS: 1024` — allows long translations
- `MIN_NEW_TOKENS: 1` — ensures at least one token is generated

### Frontend — `index.html` + `script.js` + `style.css`

A chat-style translator interface built with Bootstrap 4 and jQuery:

- **Dual input modes** — Type English text OR click 🎤 to record voice
  - Send button dynamically switches between 🎤 (microphone) and ✈️ (send) icons
  - Microphone turns red while recording
- **Voice selection** — Dropdown with 17 voices including Spanish options:
  - Laura (Castilian Spanish), Sofia (Latin American Spanish), Enrique (Castilian Spanish)
  - Plus English, French, Italian, and Portuguese voices
- **Audio playback** — Translated responses auto-play as speech; each message has a 🔊 replay button
- **Light/Dark mode** — Toggle switch for theme preference
- **Loading animations** — Bouncing dots during STT and translation processing

### Available Voices

| Voice | Language |
|-------|----------|
| Michael, Henry, Kevin | English (US) |
| Lisa, Emily, Allison, Olivia | English (US) |
| Kate, Charlotte, James | English (GB) |
| Louise | French (CA) |
| Renee, Nicolas | French (FR) |
| Francesca | Italian |
| Isabela | Brazilian Portuguese |
| **Laura** | **Castilian Spanish** |
| **Sofia** | **Latin American Spanish** |
| **Enrique** | **Castilian Spanish** |

---

## 🧠 AI Services Used

| Service | Provider | Purpose |
|---------|----------|---------|
| **Mistral Medium** | Mistral AI (via IBM watsonx) | English → Spanish translation |
| **Speech-to-Text** | IBM Watson | Transcribes English voice input (en-US_Multimedia model) |
| **Text-to-Speech** | IBM Watson | Speaks the Spanish translation aloud (17 voice options) |

---

## 🐳 Self-Hosted Watson Models

The `models/` directory includes Docker configurations to run IBM Watson STT and TTS locally:

- **`models/stt/`** — Speech-to-Text container supporting English and French
- **`models/tts/`** — Text-to-Speech container with 14+ voice models

See the README files within each directory for setup instructions. Requires an [IBM Entitlement Key](https://myibm.ibm.com/products-services/containerlibrary).

---

## 🛠️ Technologies & Libraries

| Technology | Purpose |
|------------|---------|
| [Flask](https://flask.palletsprojects.com/) | Python web server |
| [Flask-CORS](https://flask-cors.readthedocs.io/) | Cross-origin request support |
| [IBM Watson Machine Learning SDK](https://ibm.github.io/watson-machine-learning-sdk/) | Mistral model access via watsonx.ai |
| [IBM Watson Speech Services](https://www.ibm.com/products/speech-to-text) | STT and TTS APIs |
| [Bootstrap 4](https://getbootstrap.com/docs/4.5/) | Responsive UI framework |
| [jQuery](https://jquery.com/) | DOM manipulation and AJAX |
| [Font Awesome](https://fontawesome.com/) | Microphone, speaker, and send icons |
| [Docker](https://www.docker.com/) | Containerized deployment |
| [Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/MediaRecorder) | Browser microphone recording |

---

## 🔗 Related Projects

This project is part of a series built during the IBM AI Developer Professional Certificate (Course 8):

| Project | Description |
|---------|-------------|
| [Image Captioning with BLIP & Gradio](https://github.com/Arhaan-DB47/Img_captioning) | AI-powered image captioning — from local inference to automated web scraping |
| [Simple Chatbot Using LLM](https://github.com/Arhaan-DB47/Simple-chatbot-using-LLM) | Terminal-based chatbots using BlenderBot and SmolLM2 |
| [Integrate Chatbot into Web App](https://github.com/Arhaan-DB47/Integrate-chatbot-into-web-application) | Full-stack web chatbot with Flask + custom UI + Docker |
| [Voice Assistant](https://github.com/Arhaan-DB47/Voice-Assistant) | Voice-enabled assistant with OpenAI GPT + IBM Watson STT/TTS |
| [AI Meeting Companion](https://github.com/Arhaan-DB47/AI-Meeting-Companion) | Audio transcription + LLM key-point extraction with Whisper + LLaMA |
| [Chatbot for Your Data](https://github.com/Arhaan-DB47/Chatbot-for-Data) | RAG chatbot — upload PDFs and ask questions with LLaMA + ChromaDB |
| **Babel Fish Translator** *(this repo)* | Voice-enabled English → Spanish translator with Mistral + Watson STT/TTS |

---

## 📜 License

This project was created for educational purposes as part of the IBM AI Developer Professional Certificate on Coursera.

---

## 👤 Author

**Arhaan Khan** — [@Arhaan-DB47](https://github.com/Arhaan-DB47)
