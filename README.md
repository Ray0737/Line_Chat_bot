# 📦 Inventory & Room Management Chatbot

A robust **Google Apps Script** backend for a **LINE Chatbot** integrated via **Dialogflow**. This system allows users to manage equipment inventory and room bookings in real-time using a simple Google Sheet as the database.



---

## 🚀 Key Features

* **Real-time Inventory:** Check stock levels and item details via LINE Flex Messages.
* **Automated Stock Management:** Borrowing (`Ebook`) and returning (`Ereturn`) automatically updates quantities.
* **Room Management:** Toggle room status between "Occupied" and "Unoccupied."
* **Fail-Safe Validation:** A built-in logic gate ensures no data is written to the log if the User ID is missing or remains as a placeholder.
* **Activity Logs:** Instant retrieval of transaction history for specific users.

---

## 🛠️ Commands & Usage

The bot responds to comma-separated commands. The UI provides buttons, but users can also type manually:

### Equipment Commands
| Command | Format | Action |
| :--- | :--- | :--- |
| **Check** | `Check,ITEM_ID` | Displays item image, name, and current stock. |
| **Borrow** | `Ebook,ITEM_ID,QTY,USER_ID` | Reduces stock and logs the "Book" action. |
| **Return** | `Ereturn,ITEM_ID,QTY,USER_ID` | Increases stock and logs the "Return" action. |
| **History** | `Elog,USER_ID` | Returns a list of all equipment borrowed by the user. |

### Room Commands
| Command | Format | Action |
| :--- | :--- | :--- |
| **Book Room** | `Rbook,ROOM_ID,USER_ID` | Sets room status to "Occupied" and logs "Enter". |
| **Leave Room** | `Rreturn,ROOM_ID,USER_ID` | Sets room status to "Unoccupied" and logs "Leave". |
| **Room Logs** | `Rlog,USER_ID` | Returns a history of room entries/exits for the user. |

---

## 🛡️ The Fail-Safe Logic

To prevent "dirty data" (logs containing the text "USER" or empty fields), the script performs a pre-check before any sheet operations occur:

1.  **Intercepts** the `doPost` request.
2.  **Validates** that the `USER_ID` index in the command array is not empty and has been changed from the default placeholder.
3.  **Aborts** the operation if invalid, sending a warning message back to the user without touching the Spreadsheet or changing inventory counts.

> **Note:** If the validation fails, the `appendRow` and `setValue` functions are never triggered.

---

## ⚙️ Setup Instructions

### 1. Spreadsheet Setup
Create a Google Sheet with three specific tabs:
1.  **`Inventory`**: Columns: ID, Name, Category, Description, **Quantity (Col 5)**, **Image URL (Col 6)**.
2.  **`Equip Log`**: Columns: User ID, Item ID, Qty, Timestamp, Action.
3.  **`Room Log`**: Columns: User ID, Room ID, Timestamp, Action.

### 2. Deployment
1.  Open **Extensions > Apps Script** in your Google Sheet.
2.  Paste the code from `Code.gs`.
3.  Replace the `ss` variable URL with your spreadsheet URL.
4.  Click **Deploy > New Deployment**.
    * **Type:** Web App
    * **Execute as:** Me
    * **Who has access:** Anyone
5.  Copy the **Web App URL**.

### 3. Dialogflow Integration
1.  Go to your **Dialogflow Console**.
2.  Navigate to **Fulfillment**.
3.  Enable **Webhook** and paste your Apps Script Web App URL.
4.  Ensure your Intents have "Enable webhook call for this intent" toggled on.

---



# CHATBOT MANUAL (LINE)

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

var ss = SpreadsheetApp.openByUrl("...");
var sheet = ss.getSheetByName("...");
function doPost(e) {
  var data = JSON.parse(e.postData.contents);
  var userMsg = data.originalDetectIntentRequest.payload.data.message.text;
  var values = sheet.getRange(2, 1, sheet.getLastRow(), sheet.getLastColumn()).getValues();
  for (var i = 0; i < values.length; i++) {
    if (values[i][0] == userMsg) {
      i = i + 2;
      var Data = sheet.getRange(i, 2).getValue();
      var result = {
        "fulfillmentMessages": [
          {
            "platform": "line",
            "type": 4,
            "payload": {
              "line": {
                "type": "text",
                "text": Data
              }
            }
          }
        ]
      }
      var replyJSON = ContentService.createTextOutput(JSON.stringify(result)).setMimeType(ContentService.MimeType.JSON);
      return replyJSON;
    }
  }
}
