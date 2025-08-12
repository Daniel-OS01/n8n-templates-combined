# n8n Workflow: Hebrew OCR Bot (Tesseract & Gemini Vision)

This workflow extracts Hebrew text from images sent to a Telegram bot. It uses a robust, two-layer approach for Optical Character Recognition (OCR).

First, it attempts to process the image using a local, self-hosted **Tesseract OCR** engine, which is a powerful open-source tool. If Tesseract fails, produces no text, or is not installed, the workflow automatically falls back to the **Google Gemini Pro Vision API**, ensuring a high success rate for text extraction.

## Features

-   **Input**: Photo sent to a Telegram bot.
-   **Primary OCR**: Tesseract with Hebrew language pack (`heb`).
-   **Fallback OCR**: Google Gemini Pro Vision API.
-   **Processing**:
    1.  Downloads the highest resolution photo from the message.
    2.  Pipes the image data to the Tesseract command-line tool.
    3.  If Tesseract fails or returns empty text, it sends the image to Gemini Vision.
-   **Output**: Replies with the extracted Hebrew text, preserving line breaks where possible.

## Prerequisites

This workflow requires a self-hosted n8n instance to support the `Execute Command` node.

1.  **n8n Instance**: A self-hosted n8n setup (e.g., on a VPS or using Docker).
2.  **Tesseract OCR Installation**:
    -   You must install Tesseract OCR on the n8n host machine.
    -   On Debian/Ubuntu: `sudo apt update && sudo apt install tesseract-ocr`
    -   On CentOS/RHEL: `sudo yum install tesseract-ocr`
3.  **Tesseract Hebrew Language Pack**:
    -   You must also install the Hebrew language data for Tesseract.
    -   On Debian/Ubuntu: `sudo apt install tesseract-ocr-heb`
    -   On CentOS/RHEL: `sudo yum install tesseract-ocr-heb`
4.  **Telegram Bot Token**: From the [BotFather](https://t.me/botfather).
5.  **Google Cloud API Key**: For the Gemini fallback. Ensure the "Generative Language API" or a similar service is enabled for your key.

## Setup Guide

### 1. Environment Variables & Credentials

-   **In your n8n instance, go to `Credentials` > `New`:**
    1.  **Telegram**: Create a `Telegram` credential with your bot token. Note its ID for the `N8N_TELEGRAM_API_KEY_ID` environment variable.
    2.  **Google API**: Create a `Google API` credential using your API Key. Note its ID for the `N8N_GOOGLE_API_KEY_ID` environment variable.

-   Link these credentials to the workflow nodes either via environment variables or manually in the n8n UI after importing.

### 2. Import the Workflow

1.  Open your n8n canvas.
2.  Click `Import from File` and select this workflow's `workflow.json`.

### 3. Telegram Webhook Setup

1.  **Activate the workflow** in the n8n UI to get the live webhook URL.
2.  **Copy the Production URL** from the `Telegram Trigger` node.
3.  **Register the webhook** with Telegram:

    ```bash
    curl "https://api.telegram.org/bot<YOUR_TELEGRAM_BOT_TOKEN>/setWebhook?url=<YOUR_N8N_PRODUCTION_WEBHOOK_URL>"
    ```

## Configuration Notes

-   **Tesseract Command**: The `Tesseract OCR` node uses the command `tesseract stdin stdout -l heb --psm 4`.
    -   `stdin` and `stdout` are used to efficiently pipe data between nodes without writing to disk.
    -   `-l heb` explicitly tells Tesseract to use the Hebrew language model.
    -   `--psm 4` sets the Page Segmentation Mode to "Assume a single column of text of variable sizes," which is a good general-purpose setting.
-   **Fallback Logic**: The `Tesseract Failed?` IF node checks for two conditions:
    1.  If the Tesseract node itself threw an error (e.g., Tesseract is not installed).
    2.  If the Tesseract command ran but produced no text (`stdout` is empty).
    If either is true, it proceeds to the `Gemini Vision Fallback` node.
-   **Image Quality**: The workflow is configured to use the highest quality photo sent by the user (`message.photo[$json.message.photo.length-1]`). OCR accuracy is highly dependent on the quality of the source image.

## Testing Protocol

1.  **Setup**: Ensure the workflow is active, the webhook is set, and Tesseract with the Hebrew pack is installed.
2.  **Test Case 1: Tesseract Success**
    -   **Input**: Find a clear image containing printed Hebrew text (e.g., a screenshot of a news article or a picture of a book page). Send it to your bot.
    -   **Expected Output**: The bot should reply with the extracted Hebrew text.
    -   **Verification**: Check the n8n execution log to confirm the "false" path of the `Tesseract Failed?` node was taken.
3.  **Test Case 2: Gemini Fallback**
    -   **Input**: Send a more challenging image, perhaps with stylized fonts, handwriting, or some distortion. Alternatively, you can temporarily break the Tesseract command (e.g., change `-l heb` to `-l xxx`) to force the fallback.
    -   **Expected Output**: The bot should still reply with the extracted text, this time from Gemini.
    -   **Verification**: Check the n8n execution log to confirm the "true" path of the `Tesseract Failed?` node was taken, and the `Gemini Vision Fallback` node was executed.
4.  **Test Case 3: No Text**
    -   **Input**: Send a photo that contains no text (e.g., a landscape).
    -   **Expected Output**: The bot should reply with "No text found."
    -   **Verification**: Both Tesseract and Gemini should ideally find nothing, and the final expression in the `Send Extracted Text` node will fall back to the default message.

---
*This workflow provides a resilient OCR solution. The primary path is free (besides hosting costs), while the fallback ensures high accuracy on difficult images.*
