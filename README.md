# GeetaDive System Architecture

This document provides a high-level system design overview of the GeetaDive platform, detailing how the different components interact in a production environment.

## Source Code Repositories & Live Hosting

The project is split into two primary codebases:
- **[Frontend](https://github.com/AmanCode22/geetadive_frontend)**: The React-based user interface.
  - Hosted via GitHub Pages: [amancode22.github.io/geetadive](https://amancode22.github.io/geetadive)
  - Hosted via Cloudflare Pages: [geetadive.pages.dev](https://geetadive.pages.dev)
- **[Backend](https://github.com/AmanCode22/geetadive_backend)**: The Flask and MySQL powered API.
  - Hosted on Serv00: [amantest.serv00.net](https://amantest.serv00.net)

---

## System Design Overview

GeetaDive is designed with a modern decoupled client-server architecture, built for speed, caching efficiency, and interactive user experiences.

### 1. Frontend Architecture
Built using React and Vite, the frontend focuses on delivering a rich, interactive user experience:
- **Component Design**: Modular components (`ShlokaWorkspace`, `LibraryDrawer`, `AIAvatar`) isolate concerns and promote reusability.
- **Audio Processing**: Utilizes the Web Audio API (`AudioContext`, `AnalyserNode`) to process Text-to-Speech buffers in real-time. It extracts frequency data to dynamically animate the AI Avatar in sync with the audio.
- **API Communication**: Communicates asynchronously with the backend API via the `fetch` API. The base URL is globally configured through environment variables (`VITE_API_BASE_URL`) to support seamless transitions between local development and production deployments.

### 2. Backend Architecture
The backend is a lightweight Python Flask application serving as the primary orchestration layer:
- **Data Layer (MySQL)**: A robust relational MySQL database schema stores hundreds of thousands of verses. It is heavily indexed on `category` and `section` columns to support lightning-fast full-text searches and pagination across massive Dharmic datasets.
- **AI Integrations**: 
  - **Cerebras AI**: Handles blazing-fast LLM-powered multi-language translations (Tamil, Hindi, English).
  - **FishAudio**: Provides high-quality Text-to-Speech (TTS) synthesis.
- **Caching Strategy**: Implements robust caching to drastically reduce external API load and latency on repeated requests:
  - **Audio Cache**: Filesystem caching for generated MP3 audio files.
  - **Translation Cache**: JSON caching for translated text responses mapped by MD5 text hashes.

### 3. Data Flow

1. **User Input / Selection**: A user inputs or selects a Sanskrit shloka from the Library Explorer.
2. **Translation Pipeline**: The frontend requests `/api/translate`. The backend checks its local `translation_cache.json`. On a cache miss, it queries the Cerebras LLM, parses the JSON response, caches it, and returns the multi-lingual data.
3. **Speech Synthesis Pipeline**: The frontend requests `/api/tts`. The backend checks the `audio_cache` directory. On a cache miss, it streams the text to FishAudio, saves the resulting MP3 to the disk, and returns the file stream.
4. **Playback & Animation**: The frontend decodes the MP3 via the Web Audio API. As the audio plays, the frontend feeds the raw frequency data into the `AIAvatar` component's `requestAnimationFrame` render loop to synchronize mouth movements with the speech.
