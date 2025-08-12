# n8n Workflow: Hebrew Speech-to-Text (STT) Telegram Bot

This workflow transcribes Hebrew voice notes sent to a Telegram bot. It uses OpenAI's Whisper API as the primary service and falls back to Groq's Whisper-large-v3 API if the primary one fails or returns an empty result.

![Workflow Diagram](https://i.imgur.com/example.png)  _Note: This is a placeholder image. The actual workflow follows the logic described._

## Features

-   **Input**: Telegram voice note (OGG/Opus format).
-   **Processing**: Downloads the voice note and sends it to an STT API.
-   **STT Providers**:
    1.  **Primary**: OpenAI Whisper (`whisper-1` model).
    2.  **Fallback**: Groq (`whisper-large-v3` model).
-   **Language**: Optimized for Hebrew (`he`).
-   **Output**: Replies to the original message with the transcription text.
-   **Resilience**: Automatically switches to the fallback provider upon failure of the primary provider.

## Prerequisites

Before you begin, ensure you have the following:

1.  **n8n Instance**: A running n8n instance (self-hosted is recommended to handle binary data and command-line tools).
2.  **Telegram Bot Token**: A token for your Telegram bot. You can get one by talking to the [BotFather](https://t.me/botfather).
3.  **API Keys**:
    -   OpenAI API Key
    -   Groq API Key
4.  **FFmpeg (Optional but Recommended)**: While this specific workflow might not need it (as APIs accept Opus), having `ffmpeg` installed on your n8n host machine is crucial for other audio-processing workflows (like TTS).

## Setup Guide

### 1. Environment Variables & Credentials

You need to configure the credentials in your n8n instance for the different services this workflow uses.

-   **In your n8n instance, go to `Credentials` > `New`:**
    1.  **Telegram**:
        -   Select `Telegram` from the list.
        -   **Credential Name**: `Telegram Account` (or a name of your choice).
        -   **Access Token**: Paste your Telegram Bot Token.
        -   Save the credential. Note its ID.
    2.  **OpenAI**:
        -   Select `OpenAI` from the list.
        -   **Credential Name**: `OpenAI Account`.
        -   **API Key**: Paste your OpenAI API Key.
        -   Save the credential. Note its ID.
    3.  **Groq (Bearer Token)**:
        -   Select `Header Auth` from the list.
        -   **Credential Name**: `Groq Account`.
        -   **Name**: `Authorization`.
        -   **Value**: `Bearer YOUR_GROQ_API_KEY` (replace `YOUR_GROQ_API_KEY` with your actual key).
        -   Save the credential. Note its ID.

-   **Update the `workflow.json` (or the imported workflow in the UI) to use these credentials.** The provided JSON uses placeholder expressions like `{{$env.N8N_TELEGRAM_API_KEY_ID}}`. You can either:
    -   Set these environment variables in the `docker-compose.yml` or `.env` file of your n8n instance.
    -   Or, manually link the credentials in the n8n UI after importing the workflow. Go to each node that has a credential (e.g., Telegram Trigger, OpenAI, HTTP Request for Groq) and select the credential you created from the dropdown menu.

### 2. Import the Workflow

1.  Open your n8n canvas.
2.  Click `Import from File` and select the `workflow.json` provided.
3.  The workflow will appear on your canvas.

### 3. Telegram Webhook Setup

For the Telegram Trigger to work, you must register your n8n webhook URL with Telegram.

1.  **Activate the workflow** in the n8n UI. This makes the webhook URL live.
2.  **Copy the Webhook URL**:
    -   Open the `Telegram Trigger` node.
    -   It will display two webhook URLs: Test and Production. Copy the **Production URL**.
3.  **Register the Webhook**: Construct and run the following command in your terminal or browser, replacing the placeholders:

    ```bash
    curl "https://api.telegram.org/bot<YOUR_TELEGRAM_BOT_TOKEN>/setWebhook?url=<YOUR_N8N_PRODUCTION_WEBHOOK_URL>"
    ```

    -   `<YOUR_TELEGRAM_BOT_TOKEN>`: Your bot's token.
    -   `<YOUR_N8N_PRODUCTION_WEBHOOK_URL>`: The URL you copied from n8n.

    You should receive a `{"ok":true,"result":true,"description":"Webhook was set"}` response.

## Configuration Notes

-   **Provider Swapping**: The workflow is pre-configured to use OpenAI first. If you prefer Groq as the primary, you can simply rewire the nodes. Connect the `Download Voice File` node directly to the `Groq Transcription Fallback` node.
-   **Error Handling**: The current fallback is triggered by an empty response from Whisper. You can make the `Is Whisper Empty?` IF node more robust by also checking for error codes or status messages from the `Whisper Transcription` node.
-   **File Handling**: The workflow downloads the voice note into n8n's binary data store. This is efficient for passing to the next API node. For self-hosted n8n, ensure your server has adequate storage and memory for the file sizes you expect.

## Testing Protocol

1.  **Setup**: Ensure the workflow is active and the webhook is set.
2.  **Test Case 1: Simple Hebrew Voice Note**
    -   **Input**: Record a voice note on Telegram and send it to your bot. Say something simple like "שלום, זוהי בדיקה" (Shalom, zohi bdika - "Hello, this is a test").
    -   **Expected Output**: The bot should reply with the transcribed text: `שלום, זוהי בדיקה`.
    -   **Verification**: Check that the response is accurate and timely.
3.  **Test Case 2: Fallback Trigger**
    -   **Input**: To test the fallback, you can temporarily disable the `Whisper Transcription` node or configure it with invalid credentials. Send another Hebrew voice note.
    -   **Expected Output**: The workflow should fail over to the `Groq Transcription Fallback` node and still return a valid transcription.
    -   **Verification**: Check the execution log in n8n to confirm the fallback path (the `true` output of the `Is Whisper Empty?` node) was taken.
4.  **Performance Benchmarks**:
    -   **Whisper API**: Typically responds in 5-10 seconds for short audio clips (<30s).
    -   **Groq API**: Known for its high speed, often responding in 1-3 seconds for the same clips.
    -   Monitor the execution times in your n8n instance to see what real-world performance you are getting.

---
*This template is ready for production use but can be customized further to include more advanced error handling, logging, or usage analytics.*
