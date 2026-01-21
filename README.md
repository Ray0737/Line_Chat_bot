# Dialogflow + LINE Messenger + Google Sheets Integration

This repository contains the documentation and boilerplate code to build a smart chatbot that uses **Dialogflow** for Natural Language Processing (NLP), **LINE Messenger** as the interface, and **Google Sheets** as a lightweight database.

---

## System Architecture

1.  **User** sends a message to the LINE Official Account.
2.  **LINE** triggers the **Dialogflow Integration**.
3.  **Dialogflow** identifies the intent and sends a payload to the **Fulfillment Webhook**.
4.  **Google Apps Script** (the webhook) processes the data and reads/writes to **Google Sheets**.
5.  The response is sent back through the chain to the User.

---

## Getting Started

### 1. LINE Developers Console
* Create a **Messaging API** channel at [LINE Developers](https://developers.line.biz/).
* Issue a **Channel Access Token** (long-lived).
* Keep your **Channel Secret** handy.

### 2. Dialogflow (ES) Setup
* Create an agent in the [Dialogflow Console](https://dialogflow.cloud.google.com/).
* Navigate to **Integrations** > **LINE**.
* Enter your Channel ID, Channel Secret, and Access Token.
* **Important:** Copy the `Webhook URL` provided by Dialogflow and paste it into the **Messaging API** settings in your LINE Developers Console.

### 3. Google Sheets & Apps Script
* Create a new Google Sheet.
* Go to **Extensions** > **Apps Script**.
* Paste the fulfillment code (see `code.gs` in this repo).
* Click **Deploy** > **New Deployment** > **Web App**.
* **Set Access to:** "Anyone".
* Copy the **Web App URL**.

### 4. Enable Fulfillment
* In Dialogflow, go to **Fulfillment** in the left sidebar.
* Enable **Webhook**, paste your Google Apps Script URL, and click **Save**.
* In your specific **Intents**, scroll to the bottom and toggle "Enable webhook call for this intent."

---

## Sample Fulfillment Code (Google Apps Script)

```javascript
function doPost(e) {
  const contents = JSON.parse(e.postData.contents);
  const intentName = contents.queryResult.intent.displayName;
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  
  // Example: Log every message to a sheet
  const userMessage = contents.queryResult.queryText;
  sheet.appendRow([new Date(), intentName, userMessage]);

  const response = {
    "fulfillmentMessages": [
      {
        "text": {
          "text": ["I've recorded your message in the spreadsheet! "]
        }
      }
    ]
  };

  return ContentService.createTextOutput(JSON.stringify(response))
    .setMimeType(ContentService.MimeType.JSON);
}
