# Template 2: Hebrew Text-to-Speech (TTS) Bot

**Purpose:** This n8n workflow template creates a Telegram bot that converts Hebrew text messages into speech using a Text-to-Speech (TTS) API.

**Capabilities:**
- [ ] Speech-to-Text
- [x] Text-to-Speech (Hebrew)
- [ ] OCR
- [ ] Multi-modal Routing

---

## Setup Guide

### 1. Prerequisites
- An n8n instance (self-hosted or cloud).
- A Telegram Bot token.
- An API key for a TTS provider (the template uses OpenAI by default).

### 2. Environment Variables
In your n8n instance, set up the following credentials/environment variables:

- **`TELEGRAM_API_KEY`**: Your Telegram bot token.
- **`OPENAI_API_KEY`**: Your OpenAI API key. This is used for the default TTS provider.

### 3. Workflow Configuration
1.  **Import the Template**: Import the `telegram_hebrew_tts_bot.json` file into your n8n instance.
2.  **Activate the Workflow**: Activate the workflow in the n8n UI.
3.  **Webhook Configuration**: Ensure your n8n instance is publicly accessible for the Telegram webhook to function correctly.

### 4. Hebrew-Specific Settings
- The default `OpenAI TTS` node uses the `tts-1` model and the `shimmer` voice. While OpenAI's TTS supports Hebrew, the voice is not a native Hebrew speaker. For higher quality, you might need to use a different provider that offers native Hebrew voices (e.g., Google Cloud Text-to-Speech, ElevenLabs).
- The `response_format` is set to `opus`, which is the recommended format for Telegram voice notes.

---

## Configuration Notes

### How to Swap TTS Providers
This template uses an `HTTP Request` node, making it easy to switch to a different TTS provider.

1.  **Update the URL**: Change the URL in the `OpenAI TTS` node to the API endpoint of your new provider.
2.  **Update Authentication**: Change the authentication method and credentials to match the new provider.
3.  **Update the Body**: Modify the request body (in the `Body Parameters` section) to match the new provider's API specification. You will need to specify the correct parameters for voice, language, text, and format.
4.  **Rename the Node**: It's good practice to rename the node to reflect the new provider (e.g., "Google TTS").

### Audio Format
- The template is configured to request `opus` from the TTS provider. If your chosen provider does not support `opus` directly, you may need to add a media conversion step using a tool like FFmpeg. You could execute an FFmpeg command via the `Execute Command` node in n8n.

---

## Testing Protocol

1.  **Send a Text Message**: Send a text message in Hebrew to your Telegram bot.
    - **Sample Input (Text)**: `שלום, איך حالך?` (Shalom, eikh halekh? - "Hello, how are you?")
2.  **Verify the Output**: The bot should reply with a voice message.
    - **Expected Output (Voice)**: A voice note that speaks the Hebrew text you sent.
3.  **Check Pronunciation**: Listen to the voice note to verify that the Hebrew pronunciation is clear and correct.
4.  **Test with Mixed Languages**: Send a message containing both Hebrew and English text to see how the TTS engine handles it.
    - **Sample Input (Text)**: `זהו test message` (Zeh test message - "This is a test message.")
    - **Expected Output (Voice)**: A voice note that attempts to pronounce both the Hebrew and English words. The quality will depend on the TTS provider's capabilities.
