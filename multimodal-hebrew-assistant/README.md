# n8n Workflow: Multi-Modal Hebrew Assistant

This advanced workflow acts as a central router or "brain" for your Telegram bot, intelligently handling different types of media. It automatically detects whether an incoming message is a voice note, a photo, or a plain text message, and routes it to the appropriate specialized workflow or a conversational AI.

This assistant is designed to be modular, using the `Execute Workflow` node to call the **Hebrew STT Bot** and **Hebrew OCR Bot** workflows you have already set up. For text messages, it engages a conversational AI (Gemini Pro) and maintains a chat history for context-aware replies.

## Features

-   **Multi-Modal Input**: A single webhook endpoint handles voice, photo, and text messages.
-   **Intelligent Routing**: A `Switch` node directs the input to the correct processing path.
    -   **Voice Message**: Triggers the `Hebrew STT Bot` workflow.
    -   **Photo Message**: Triggers the `Hebrew OCR Bot` workflow.
    -   **Text Message**: Triggers a new conversational chat flow.
-   **Conversational AI**: Uses the Gemini Pro model for engaging, human-like text conversations in Hebrew.
-   **Context Memory**: Implements a simple, file-based chat history on the n8n server's local file system. This allows the bot to remember the last few exchanges with a user for more natural conversations.

## Prerequisites

This workflow consolidates the requirements of all previous templates.

1.  **All Previous Workflows**: You must have already imported and configured the `hebrew-stt-bot` and `hebrew-ocr-bot` workflows into your n8n instance. They should be activated.
2.  **Self-Hosted n8n**: Required for the file-based chat history and for the sub-workflows that use `Execute Command` nodes.
3.  **All Credentials**: Ensure your Telegram, OpenAI, Groq, and Google API credentials are set up in your n8n instance.
4.  **All Dependencies**: `ffmpeg` and `Tesseract` (with the Hebrew language pack) must be installed on the n8n host machine.

## Setup Guide

### 1. Get Workflow IDs

Before configuring this workflow, you need the IDs of the STT and OCR workflows you imported earlier.

1.  Open your n8n instance.
2.  Go to the `Workflows` list.
3.  Open the `Hebrew STT Bot` workflow. The ID is the last part of the URL in your browser's address bar (e.g., `.../workflow/123`).
4.  Note this ID. This is your `STT_WORKFLOW_ID`.
5.  Repeat the process for the `Hebrew OCR Bot` workflow to get your `OCR_WORKFLOW_ID`.

### 2. Environment Variables

This workflow is best configured using environment variables in your n8n `docker-compose.yml` or `.env` file.

```
# Credential IDs (get from n8n Credentials screen)
N8N_TELEGRAM_API_KEY_ID=your_telegram_credential_id
N8N_GOOGLE_API_KEY_ID=your_google_api_credential_id

# Workflow IDs (from step 1)
STT_WORKFLOW_ID=your_stt_workflow_id
OCR_WORKFLOW_ID=your_ocr_workflow_id
```

If you don't use environment variables, you must manually enter the workflow IDs and link credentials in the n8n UI after importing.

### 3. Import the Workflow

1.  Import the `multimodal-hebrew-assistant/workflow.json` file into your n8n canvas.
2.  If you did not set environment variables, manually edit the `Execute STT Workflow` and `Execute OCR Workflow` nodes, filling in the `Workflow ID` field for each. Link the credentials in the `Gemini Chat` and `Send Chat Response` nodes.

### 4. Telegram Webhook Setup

This is the only webhook you need to set for your bot, as it handles all inputs.

1.  **Deactivate other triggers**: If your STT and OCR workflows are active, open their `Telegram Trigger` nodes and deactivate them to avoid multiple triggers firing.
2.  **Activate this workflow**.
3.  **Copy the Production URL** from the `Telegram Trigger` node in this workflow.
4.  **Register the webhook** with Telegram:
    ```bash
    curl "https://api.telegram.org/bot<YOUR_TELEGRAM_BOT_TOKEN>/setWebhook?url=<YOUR_N8N_PRODUCTION_WEBHOOK_URL>"
    ```

## Configuration Notes

-   **Chat History Path**: The `Get Chat History` and `Update Chat History` Function nodes store conversation data in a file at `/home/node/.n8n/local-files/conversation_store.json`.
    -   This path is typically writable by default in standard n8n Docker containers. If you have a custom setup, ensure the n8n user has write permissions to this directory.
    -   This file will be created automatically on the first text message.
-   **History Length**: The `Update Chat History` node is configured to keep the last 20 messages (10 user messages and 10 bot responses). You can change the `maxHistory` variable inside the Function node's code to store more or less context.
-   **Modularity**: Using `Execute Workflow` is powerful but note that data must be passed and returned explicitly. These sub-workflows were designed to be self-contained and reply on Telegram directly. For a more advanced setup, you could modify them to *return* data to this main workflow instead of sending their own replies.

## Testing Protocol

1.  **Setup**: Ensure all workflows are imported, this main workflow is active with the correct webhook, and the other workflow triggers are inactive.
2.  **Test Case 1: Voice Message**
    -   **Input**: Send a Hebrew voice note.
    -   **Expected Output**: The bot should reply with the transcription.
    -   **Verification**: Check the n8n execution log. You should see the `Switch Input Type` routing to the `Execute STT Workflow` node.
3.  **Test Case 2: Image Message**
    -   **Input**: Send an image with Hebrew text.
    -   **Expected Output**: The bot should reply with the extracted OCR text.
    -   **Verification**: The execution log should show the `Execute OCR Workflow` path being taken.
4.  **Test Case 3: Text Conversation**
    -   **Input**: Send a text message: "ספר לי בדיחה" (Tell me a joke).
    -   **Expected Output**: The bot should reply with a joke from Gemini.
    -   **Verification**: The log should show the "Get History" -> "Gemini Chat" -> "Update History" path.
5.  **Test Case 4: Context Memory**
    -   **Input**: After the joke, send a follow-up: "זה היה מצחיק, ספר לי עוד אחת" (That was funny, tell me another one).
    -   **Expected Output**: The bot should understand the context ("another one" refers to a joke) and provide a new joke.
    -   **Verification**: Inspect the `Gemini Chat` node's input in the execution log. You should see that the `contents` array included the previous messages, providing context to the AI. Check the `conversation_store.json` file on your server to see the history being saved.

---
*This multi-modal assistant provides a powerful, centralized, and extensible foundation for your Telegram bot.*
