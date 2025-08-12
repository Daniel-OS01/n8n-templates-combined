# Hebrew Multi-Modal Telegram Agent

A modular, multi-workflow n8n setup for a powerful, Hebrew-capable Telegram bot that can understand voice, images, and text.

This project provides a robust, scalable, and production-ready foundation for a multi-modal Telegram assistant that is fluent in Hebrew. It leverages a modern AI stack and a modular architecture that is easy to maintain and extend.

## Architecture

This project uses a multi-workflow ("agent-based") architecture. A central **Main Orchestrator** workflow acts as the brain, receiving messages from Telegram and delegating tasks to specialized, single-purpose **Agent** workflows. This design ensures separation of concerns and makes it easy to update or replace individual components (like an STT provider) without affecting the entire system.

### Architectural Flow Diagram

```
[User on Telegram]
       |
       v
[Telegram API]
       |
       v
[1. Main Orchestrator]--(receives message)
       |
       +---> [Switch: Is it Voice?] ---> [2. STT Agent] --(transcribed text)--> [Junction]
       |
       +---> [Switch: Is it Image?] --> [3. OCR Agent] --(extracted text)----> [Junction]
       |
       +---> [Switch: Is it Text?] -----------------------------------------> [Junction]
                                                                                  |
                                                                                  v
                                                                   [4. LLM Agent] --(AI response)
                                                                                  |
                                                                                  v
                                         [Switch: Was original input Voice?] --(no)--> [Send Text to User]
                                                                                  |
                                                                                (yes)
                                                                                  |
                                                                                  v
                                                            [5. TTS Agent] --(audio data)--> [Send Voice to User]
```

## Features

-   **Modular Architecture**: 5 separate workflows for maximum maintainability and scalability.
-   **Hebrew Speech-to-Text (STT)**: Transcribes Hebrew voice messages using the high-performance Groq (Whisper) API.
-   **Hebrew Optical Character Recognition (OCR)**: Extracts Hebrew text from images and documents using the powerful Gemini Vision API.
-   **AI-Powered Responses**: Leverages the Gemini Pro LLM to generate intelligent, context-aware responses in Hebrew.
-   **Hebrew Text-to-Speech (TTS)**: Replies with natural-sounding Hebrew voice messages using Google's TTS API.
-   **Multi-Modal**: Intelligently handles text, voice, and image inputs, and can reply in the appropriate format.

## Setup Instructions

Follow these steps carefully to deploy the entire agent system.

### Prerequisites

-   An active **n8n instance** (Cloud or self-hosted).
-   A **Telegram Bot Token**. Talk to the [BotFather](https://t.me/botfather) to create a bot and get your token.
-   **API Keys** for the following services:
    -   **Groq**: For STT. Get your key from [GroqCloud](https://console.groq.com/keys).
    -   **Gemini (Google AI)**: For OCR and LLM. Get your key from [Google AI Studio](https://aistudio.google.com/app/apikey).
    -   **Google Cloud**: For TTS. You need a Google Cloud project with the Text-to-Speech API enabled. Get your API key from the [Google Cloud Console](https://console.cloud.google.com/apis/credentials).

### Step 1: Import All Workflows

1.  Download the 5 workflow JSON files from this directory:
    -   `main-orchestrator.json`
    -   `agent-stt.json`
    -   `agent-ocr.json`
    -   `agent-llm.json`
    -   `agent-tts.json`
2.  In your n8n instance, go to the "Workflows" page and click "Import from File" for each of the 5 files.

### Step 2: Configure Credentials

1.  In your n8n instance, go to "Credentials" in the left-hand menu.
2.  Click "Add credential" and create the following credentials:
    -   **Telegram**:
        -   Credential Type: `Telegram`
        -   Credential Name: `Telegram` (or a name of your choice)
        -   API Key: Your Telegram Bot Token
    -   **Groq**:
        -   Credential Type: `Header Auth`
        -   Credential Name: `Groq`
        -   Header Name: `Authorization`
        -   Header Value: `Bearer YOUR_GROQ_API_KEY`
    -   **Gemini**:
        -   Credential Type: `Generic Credential Type` -> `API Key Auth`
        -   Credential Name: `Gemini`
        -   API Key: Your Gemini API Key
3.  The Google TTS API key will be stored as an environment variable, not a credential, for simplicity in the HTTP Request node.

### Step 3: Deploy and Configure the Agents

For each of the four agent workflows (`agent-stt`, `agent-ocr`, `agent-llm`, `agent-tts`):

1.  Open the workflow in n8n.
2.  Click the "Active" toggle in the top right to activate the workflow.
3.  Click on the "Webhook" trigger node at the start of the workflow.
4.  Copy the **Production URL** for the webhook. This is the public address for your agent.
5.  **Save the URL**. You will need these four URLs for the next step.

### Step 4: Configure and Deploy the Main Orchestrator

1.  Create a `.env` file in your n8n environment (if you don't have one already) or configure the environment variables directly in your n8n instance.
2.  Add the following variables to your environment, filling in the values you've collected:

    ```
    # From Prerequisites
    TELEGRAM_API_KEY=
    GEMINI_API_KEY=
    GROQ_API_KEY=
    GOOGLE_API_KEY=

    # From Step 3
    STT_AGENT_WEBHOOK_URL=
    OCR_AGENT_WEBHOOK_URL=
    LLM_AGENT_WEBHOOK_URL=
    TTS_AGENT_WEBHOOK_URL=
    ```
3.  Open the `main-orchestrator` workflow in n8n.
4.  Activate the workflow using the "Active" toggle.

### Step 5: Connect Your Telegram Bot

1.  Get the webhook URL for the **Main Orchestrator** workflow (click the "Telegram Trigger" node and copy the Production URL).
2.  Set your Telegram bot's webhook to this URL by running the following command in your terminal. Replace `<YOUR_TELEGRAM_API_KEY>` and `<YOUR_ORCHESTRATOR_WEBHOOK_URL>` with your actual values.

    ```bash
    curl --request POST \
         --url https://api.telegram.org/bot<YOUR_TELEGRAM_API_KEY>/setWebhook \
         --header 'content-type: application/json' \
         --data '{"url": "<YOUR_ORCHESTRATOR_WEBHOOK_URL>"}'
    ```

3.  If successful, you will see a response like `{"ok":true,"result":true,"description":"Webhook was set"}`. Your bot is now live!

## Environment Variables

| Variable                  | Description                                                                                             |
| ------------------------- | ------------------------------------------------------------------------------------------------------- |
| `TELEGRAM_API_KEY`        | Your API key for the Telegram Bot.                                                                      |
| `STT_AGENT_WEBHOOK_URL`   | The full production webhook URL for your deployed `agent-stt` workflow.                                 |
| `OCR_AGENT_WEBHOOK_URL`   | The full production webhook URL for your deployed `agent-ocr` workflow.                                 |
| `LLM_AGENT_WEBHOOK_URL`   | The full production webhook URL for your deployed `agent-llm` workflow.                                 |
| `TTS_AGENT_WEBHOOK_URL`   | The full production webhook URL for your deployed `agent-tts` workflow.                                 |
| `GEMINI_API_KEY`          | Your API key for Google Gemini (used for OCR and LLM).                                                  |
| `GROQ_API_KEY`            | Your API key for GroqCloud (used for STT).                                                              |
| `GOOGLE_API_KEY`          | Your API key for a Google Cloud project with the Text-to-Speech API enabled.                            |

## Usage

Once deployed, you can interact with your bot in Telegram:

-   **Send a text message**: The bot will reply with a text message.
-   **Send a voice message (in Hebrew)**: The bot will transcribe it and reply with a voice message.
-   **Send an image or document with Hebrew text**: The bot will perform OCR and reply with a text message containing the extracted text.
