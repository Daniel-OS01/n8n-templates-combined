# n8n Workflow: Hebrew Text-to-Speech (TTS) Telegram Bot

This workflow converts Hebrew text messages into high-quality voice notes and sends them back to the user on Telegram. It is designed for a self-hosted n8n environment where command-line tools can be installed.

The workflow uses the Google Cloud Text-to-Speech API for generating the audio and `ffmpeg` for converting the audio into the Opus format required for Telegram voice notes.

## Features

-   **Input**: Hebrew text message from Telegram.
-   **Processing**:
    1.  Calls the Google Cloud TTS API to generate an MP3 audio file from the text.
    2.  Decodes the Base64 audio response into a binary file.
    3.  Uses the `Execute Command` node to run `ffmpeg`, converting the MP3 to a high-quality Opus file.
    4.  Sends the Opus file as a voice note reply.
-   **Language**: Optimized for Hebrew (`he-IL` voices).
-   **Output**: Telegram voice note.

## Prerequisites

1.  **n8n Instance**: A **self-hosted** n8n instance is required to use the `Execute Command` node.
2.  **`ffmpeg` Installation**: You must have `ffmpeg` installed on the same machine that hosts your n8n instance.
    -   To install on Debian/Ubuntu: `sudo apt update && sudo apt install ffmpeg`
    -   To install on CentOS/RHEL: `sudo yum install ffmpeg`
    -   For Docker-based n8n setups, you may need to create a custom `Dockerfile` that extends the base n8n image and installs ffmpeg. Example `Dockerfile`:
        ```Dockerfile
        FROM n8nio/n8n
        USER root
        RUN apk add --no-cache ffmpeg
        USER node
        ```
3.  **Telegram Bot Token**: From the [BotFather](https://t.me/botfather).
4.  **Google Cloud API Key**: A Google Cloud Platform project with the "Text-to-Speech API" enabled. You will need to generate an API Key.

## Setup Guide

### 1. Environment Variables & Credentials

-   **In your n8n instance, go to `Credentials` > `New`:**
    1.  **Telegram**:
        -   Create a `Telegram` credential with your bot token (if you haven't already).
        -   Note its ID to set `N8N_TELEGRAM_API_KEY_ID`.
    2.  **Google Cloud API**:
        -   Select `Google API` from the list.
        -   **Credential Name**: `Google API Account`.
        -   **Authentication**: `API Key`.
        -   **API Key**: Paste your Google Cloud API Key.
        -   Save and note its ID to set `N8N_GOOGLE_API_KEY_ID`.

-   **Update the workflow to use these credentials** by either setting the environment variables in your n8n host environment or by manually linking them in the UI after import.

### 2. Import the Workflow

1.  Open your n8n canvas.
2.  Click `Import from File` and select the `workflow.json` for this bot.
3.  The workflow will appear on your canvas.

### 3. Telegram Webhook Setup

1.  **Activate the workflow** in the n8n UI.
2.  **Copy the Production Webhook URL** from the `Telegram Trigger` node.
3.  **Register the webhook** with Telegram by running this command in your terminal or browser:

    ```bash
    curl "https://api.telegram.org/bot<YOUR_TELEGRAM_BOT_TOKEN>/setWebhook?url=<YOUR_N8N_PRODUCTION_WEBHOOK_URL>"
    ```

    You should see a success message.

## Configuration Notes

-   **Changing the Voice**: You can change the Hebrew voice by editing the `Google Cloud TTS` node. In the `Body` parameter, modify the `name` field.
    -   Current voice: `he-IL-Wavenet-C` (female).
    -   Other options include `he-IL-Wavenet-A` (male), `he-IL-Standard-C` (female), and `he-IL-Standard-A` (male). Refer to the [Google Cloud TTS documentation](https://cloud.google.com/text-to-speech/docs/voices) for the latest list.
-   **Audio Encoding**: The `Convert MP3 to Opus` node uses the command `ffmpeg -i pipe:0 -c:a libopus -b:a 64k -f opus pipe:1`. This takes the input from the previous node (`pipe:0`), converts it to the `libopus` codec with a bitrate of `64k`, and pipes the output (`pipe:1`) to the next node. This is highly efficient as it avoids writing files to disk.
-   **Error Handling**: For production, you should add error handling. For example, add an `If` node after the `Google Cloud TTS` node to check if the API call was successful before proceeding.

## Testing Protocol

1.  **Setup**: Ensure the workflow is active, the webhook is set, and `ffmpeg` is installed correctly.
2.  **Test Case 1: Simple Hebrew Text**
    -   **Input**: Send a text message to your bot, e.g., "כמה עולה גלידה?" (Kama ole glida? - "How much does ice cream cost?").
    -   **Expected Output**: The bot should reply with a voice note that speaks the Hebrew text.
    -   **Verification**: Listen to the voice note. It should be clear, in Hebrew, and match the text you sent.
3.  **Test Case 2: Long Text**
    -   **Input**: Send a longer paragraph of Hebrew text.
    -   **Expected Output**: The bot should generate and send a longer voice note.
    -   **Verification**: This tests the limits of the TTS API (Google has a 5000-character limit per request) and the performance of the `ffmpeg` conversion. Check the execution log in n8n for any errors or timeouts.

---
*This template provides a powerful TTS capability. Ensure your self-hosted environment is properly configured for it to function correctly.*
