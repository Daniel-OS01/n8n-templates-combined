# Comprehensive Hebrew Multi-Modal Assistant

**Purpose:** This n8n workflow template creates a single, advanced Telegram bot that functions as a multi-modal assistant. It can understand and respond to text, Hebrew voice notes, and images containing Hebrew text. It maintains a persistent, user-specific conversation history using Supabase and OpenAI's Assistants API.

**Capabilities:**
- [x] Speech-to-Text (Hebrew)
- [x] Text-to-Speech (via Assistant, optional addon)
- [x] OCR (Hebrew from images/PDFs)
- [x] Multi-modal Routing
- [x] Conversational Memory

---

## Setup Guide

This is an advanced workflow that requires several services to be configured.

### 1. Prerequisites
- An n8n instance (self-hosted or cloud) with public webhook access.
- A Telegram Bot token.
- An OpenAI API key.
- A Supabase account for the memory database.
- A running OCR service endpoint.

### 2. Environment Variables
In your n8n instance, set up the following credentials/environment variables:

- **`TELEGRAM_API_KEY`**: Your Telegram bot token from BotFather.
- **`OPENAI_API_KEY`**: Your OpenAI API key.
- **`OPENAI_ASSISTANT_ID`**: The ID of the OpenAI Assistant you want to use. See step 3b below.
- **`SUPABASE_CREDENTIALS`**: Your Supabase credentials (URL and anon key).
- **`OCR_API_URL`**: The URL for your self-hosted OCR service. See step 3d below.

### 3. Service Configuration

**a. Telegram Bot:**
- Create a bot using the [BotFather](https://t.me/botfather) on Telegram to get your `TELEGRAM_API_KEY`.

**b. OpenAI Assistant:**
- Go to the [OpenAI Assistants](https://platform.openai.com/assistants) page.
- Create a new Assistant. You can give it a name, instructions (e.g., "You are a helpful assistant for Hebrew-speaking users"), and choose a model.
- Once created, copy the Assistant ID and set it as the `OPENAI_ASSISTANT_ID` environment variable.

**c. Supabase Database (for Memory):**
1.  Create a new project in [Supabase](https://supabase.com/).
2.  Go to the `SQL Editor` and run the following query to create the `telegram_users` table. This table will store the mapping between Telegram user IDs and their corresponding OpenAI thread IDs.
    ```sql
    CREATE TABLE public.telegram_users (
      id UUID NOT NULL DEFAULT gen_random_uuid(),
      created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT (now() AT TIME ZONE 'utc'::text),
      telegram_id BIGINT NULL,
      openai_thread_id TEXT NULL,
      CONSTRAINT telegram_users_pkey PRIMARY KEY (id)
    );
    ```
3.  In your Supabase project settings, find your project URL and `anon` key to create the `SUPABASE_CREDENTIALS` in n8n.

**d. Self-Hosted OCR Service:**
- This workflow requires an external HTTP endpoint for OCR. You can follow the instructions in `telegram_hebrew_ocr_bot.md` to set up a simple Tesseract-based OCR server.
- Once your service is running, set its URL as the `OCR_API_URL` environment variable.

### 4. Workflow Configuration
1.  **Import the Template**: Import the `telegram_hebrew_multimodal_assistant.json` file into your n8n instance.
2.  **Verify Credentials**: Ensure all nodes that require credentials are correctly configured to use the environment variables you set up.
3.  **Activate the Workflow**: Activate the workflow. n8n will handle the webhook registration with Telegram.

---

## Workflow Logic Explained

1.  **Trigger & Memory**: The workflow triggers on any new message. It immediately looks up the `telegram_id` in Supabase.
    - If the user exists, it retrieves their `openai_thread_id`.
    - If not, it creates a new OpenAI thread, saves the new user and thread ID to Supabase, and then proceeds.
2.  **Routing**: A Router node checks the message content and directs the workflow to one of three branches:
    - **Voice Branch**: Downloads the voice note, transcribes it to Hebrew text using Whisper, and passes the text to the main flow.
    - **Image/Document Branch**: Downloads the file, extracts Hebrew text using the OCR service, and passes the text to the main flow.
    - **Text Branch**: Simply uses the incoming text directly.
3.  **Assistant Interaction**: The processed text (from any of the branches) is sent as a new message to the user's OpenAI thread. The workflow then runs the assistant, waits a few seconds for the processing to complete, and retrieves the latest message from the thread.
4.  **Response**: The assistant's text response is sent back to the user in Telegram.

---

## Testing Protocol

1.  **Test Text Input**:
    - **Input**: Send a simple text message: `שלום`
    - **Expected Output**: A greeting from your OpenAI assistant in Hebrew.

2.  **Test Voice Input**:
    - **Input**: Send a voice note saying: `מה מזג האוויר היום בתל אביב?` ("What is the weather today in Tel Aviv?")
    - **Expected Output**: The assistant should respond with the weather information for Tel Aviv.

3.  **Test Image Input**:
    - **Input**: Send an image containing the Hebrew text: `כמה עולה כרטיס רכבת לירושלים?` ("How much is a train ticket to Jerusalem?")
    - **Expected Output**: The assistant should answer the question based on the text it extracted from the image.

4.  **Test Conversation Memory**:
    - **Step 1 Input**: Send a text message: `שמי דוד` ("My name is David").
    - **Step 1 Output**: The assistant should acknowledge this.
    - **Step 2 Input**: Send another message: `מה שמי?` ("What is my name?").
    - **Step 2 Output**: The assistant should correctly respond with "דוד" ("David"), demonstrating that it remembers the context from the previous message in the same thread.
