# Template 1: Hebrew Speech-to-Text (STT) Bot

**Purpose:** This n8n workflow template creates a Telegram bot that transcribes Hebrew voice notes into text using OpenAI's Whisper API.

**Capabilities:**
- [x] Speech-to-Text (Hebrew)
- [ ] Text-to-Speech
- [ ] OCR
- [ ] Multi-modal Routing

---

## Setup Guide

### 1. Prerequisites
- An n8n instance (self-hosted or cloud).
- A Telegram Bot token.
- An OpenAI API key.

### 2. Environment Variables
In your n8n instance, you need to set up the following credentials/environment variables:

- **`TELEGRAM_API_KEY`**: Your Telegram bot token.
  - To get one, talk to the [BotFather](https://t.me/botfather) on Telegram.
- **`OPENAI_API_KEY`**: Your OpenAI API key.
  - You can get one from the [OpenAI Platform](https://platform.openai.com/account/api-keys).

### 3. Workflow Configuration
1.  **Import the Template**: Import the `telegram_hebrew_stt_bot.json` file into your n8n instance.
2.  **Activate the Workflow**: Activate the workflow in the n8n UI. The Telegram Trigger will automatically register the webhook.
3.  **Webhook Configuration**: n8n handles the webhook registration automatically. Make sure your n8n instance is reachable from the internet at its configured webhook URL. For self-hosted instances, this might require setting up a reverse proxy and a domain.

### 4. Hebrew-Specific Settings
- The `Whisper STT` HTTP Request node is pre-configured for Hebrew transcription with the `language` parameter set to `he`.
- Whisper is generally good with diacritics (nikkud), but its performance may vary.

---

## Configuration Notes

### Self-Host vs. n8n Cloud
- **n8n Cloud**: Webhooks work out of the box.
- **Self-Hosted**: You must ensure your n8n instance is publicly accessible via a URL for the Telegram webhook to work. Using a service like ngrok during development can be helpful.

### Rate Limits & Fallbacks
- **OpenAI API**: Be aware of the rate limits of the Whisper API. For high-volume use, you might need to implement queuing or a more robust error-handling strategy.
- **Fallback Providers**: The current template uses OpenAI's Whisper. To add a fallback (e.g., to Groq), you can add another HTTP Request node and configure it to run on an error from the Whisper node. You would need to adjust the Groq API call to match its documentation for speech-to-text.

### Error Handling
- The current workflow is simple and does not include sophisticated error handling. You can add a "Respond on Error" path using the "Error Trigger" node or the error output of the HTTP Request node to inform the user if the transcription fails.

---

## Testing Protocol

1.  **Send a Voice Note**: Record a voice note in Hebrew and send it to your Telegram bot.
    - **Sample Input (Voice)**: A voice note saying "שלום, זהו מבחן." (Shalom, zehu mivchan - "Hello, this is a test.")
2.  **Verify the Output**: The bot should reply with the transcribed text.
    - **Expected Output (Text)**: `שלום, זהו מבחן.`
3.  **Check RTL Formatting**: Ensure the text is displayed correctly with right-to-left (RTL) formatting in your Telegram client.
4.  **Test with Diacritics**: Send a voice note with Hebrew words that include nikkud (diacritics) and check if they are transcribed correctly.
    - **Sample Input (Voice)**: A voice note saying "בֹּקֶר טוֹב" (Boker tov - "Good morning.")
    - **Expected Output (Text)**: `בוקר טוב` (Whisper often removes diacritics in the final text output, which is generally acceptable for modern Hebrew).
