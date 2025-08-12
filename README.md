# Production-Ready n8n Templates for a Hebrew Telegram Bot

This project contains a suite of four production-ready n8n workflow templates for building a multi-functional, Hebrew-speaking Telegram bot. The workflows are designed to be modular, robust, and easy to deploy on a self-hosted n8n instance.

They prioritize open-source solutions (`Tesseract`, `ffmpeg`) and provide fallbacks to powerful APIs (`Groq`, `Google Gemini`) to ensure high availability and performance.

## Workflow Templates

This repository is organized into folders, one for each workflow. Each folder contains:
- `workflow.json`: The n8n workflow file, ready for import.
- `README.md`: A detailed guide covering prerequisites, setup, configuration, and testing for that specific workflow.

---

### 1. [Hebrew Speech-to-Text (STT) Bot](./hebrew-stt-bot/)

-   **Description**: Transcribes Hebrew voice notes sent to the bot.
-   **Features**: Uses OpenAI Whisper as the primary STT engine and falls back to Groq's Whisper-large-v3 for speed and redundancy.
-   **Priority**: 1

### 2. [Hebrew Text-to-Speech (TTS) Bot](./hebrew-tts-bot/)

-   **Description**: Converts Hebrew text messages into natural-sounding voice replies.
-   **Features**: Uses the Google Cloud Text-to-Speech API and an on-host `ffmpeg` process to encode the audio into the Opus format required by Telegram.
-   **Priority**: 2

### 3. [Hebrew OCR Document Bot](./hebrew-ocr-bot/)

-   **Description**: Extracts Hebrew text from images.
-   **Features**: Prioritizes a local Tesseract OCR engine for free and fast processing, with a seamless fallback to the Gemini Vision API for more challenging images.
-   **Priority**: 3

### 4. [Multi-Modal Hebrew Assistant](./multimodal-hebrew-assistant/)

-   **Description**: An advanced, all-in-one bot that acts as a central router.
-   **Features**:
    -   Detects input type (voice, photo, or text).
    -   Calls the appropriate STT or OCR workflow as a sub-process.
    -   Engages in conversational chat using Gemini.
    -   Maintains conversation history for context-aware responses.

## General Requirements

-   **Platform**: A self-hosted n8n instance is required for workflows using the `Execute Command` node or local file storage for context memory.
-   **Language**: All workflows are configured for Hebrew (`he` or `he-IL`).
-   **Dependencies**: The TTS and OCR bots require `ffmpeg` and `Tesseract` (with the Hebrew language pack) to be installed on the n8n host machine. Please see the individual `README.md` files for full instructions.

## Getting Started

1.  Choose a workflow you want to implement from the list above.
2.  Navigate to its directory.
3.  Follow the instructions in its `README.md` file to set up the prerequisites, credentials, and webhook.
4.  Import the `workflow.json` into your n8n instance and activate it.

For the full experience, set up the STT and OCR bots first, then deploy the Multi-Modal Assistant, which integrates them.
