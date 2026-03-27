# Classwork | LINE Inventory Tracker Chatbot

A **LINE Messenger chatbot** for managing equipment inventory and room bookings in real-time, built with **Google Apps Script**, **Dialogflow**, and **Google Sheets** as the backend database.

---

## Overview

This project was the culminating assignment for the E-AI chatbot module. The bot allows users to check, borrow, and return equipment, manage room availability, view activity logs, and add new inventory — all through natural conversation on LINE Messenger.

---

## System Architecture

```
User (LINE) → LINE Messaging API → Dialogflow (NLP) → Google Apps Script (Webhook) → Google Sheets (Database)
```

1. A user sends a message to the LINE Official Account.
2. LINE forwards the message to **Dialogflow**, which identifies the intent.
3. Dialogflow triggers the **fulfillment webhook** (Google Apps Script).
4. The script reads/writes to **Google Sheets** and returns a response (text, Flex Message, or Carousel).

---

## Key Features

| Feature | Description |
| :--- | :--- |
| **Real-time Inventory Check** | Query item stock levels with image and status via LINE Flex Messages. |
| **Equipment Booking & Return** | Automatically adjusts stock quantities and logs every transaction with timestamps. |
| **Room Management** | Toggle room status between "Occupied" and "Unoccupied" with entry/exit logging. |
| **Activity Logs** | Retrieve transaction history for any user ID — filterable by equipment or room. |
| **Add New Items** | Admin-level command to register new inventory items with duplicate-ID protection. |
| **Fail-Safe Validation** | Pre-checks ensure User ID is valid before any database write occurs. |
| **Carousel Menu UI** | An interactive card-based menu displayed on bot start for quick command access. |

---

## Commands & Syntax

### Equipment
| Command | Syntax | Action |
| :--- | :--- | :--- |
| Check | `Check,ITEM_ID` | Shows item image, name, and current stock. |
| Borrow | `Equipment booking,ITEM_ID,QTY,USER_ID` | Deducts stock and logs a "Book" action. |
| Return | `Equipment return,ITEM_ID,QTY,USER_ID` | Adds stock back and logs a "Return" action. |
| Logs | `Equipment log,USER_ID` | Displays the user's equipment transaction history. |
| Add | `Add,ID,NAME,QTY,ADMIN_NAME` | Registers a new item (admin only). |

### Room
| Command | Syntax | Action |
| :--- | :--- | :--- |
| Book | `Room book,ROOM_ID,USER_ID` | Sets room to "Occupied" and logs "Enter". |
| Leave | `Room return,ROOM_ID,USER_ID` | Sets room to "Unoccupied" and logs "Leave". |
| Logs | `Room log,USER_ID` | Displays the user's room entry/exit history. |

---

## Repository Structure

| File | Description |
| :--- | :--- |
| `Chat Prototype.txt` | Early prototype script — basic stock add/remove with Flex Message card response. Served as proof-of-concept for the Dialogflow-to-Sheets pipeline. |
| `Chat Final Task (MAIN).txt` | The complete production script with all 7 command handlers, carousel UI, error handling, modular response builders (`Result0`–`Result3`), and timestamp logging. |
| `Chat Final Task (1-5).txt` | Incremental development versions showing the evolution from prototype to final. |
| `Line QR.png` | QR code for the LINE Official Account. |
| `Spreadsheet.png` | Screenshot of the Google Sheets database structure. |

---

## How It Was Built

- **Backend:** Google Apps Script (JavaScript) deployed as a Web App.
- **NLP Engine:** Dialogflow ES — handles intent recognition and routes commands as comma-separated strings.
- **Database:** Google Sheets with 3 tabs: `Inventory` (stock data + image URLs), `Equip Log` (borrow/return records), and `Room Log` (room entry/exit records).
- **Frontend:** LINE Messaging API — responses rendered as Flex Messages (rich cards) and Carousel templates.
- **Development Process:** Started with a simple prototype (`Chat Prototype.txt`) handling only add/remove stock with a Flex card response, then iteratively expanded through 5 versions into the final multi-feature system with error handling, input validation, and modular response functions.

---

## Setup Instructions

### 1. Google Sheets
Create a spreadsheet with three tabs:
- **Inventory**: `ID | Name | Category | Description | Quantity | Image URL`
- **Equip Log**: `User ID | Item ID | Qty | Timestamp | Action`
- **Room Log**: `User ID | Room ID | Timestamp | Action`

### 2. Google Apps Script
1. Open **Extensions > Apps Script** in the spreadsheet.
2. Paste the code from `Chat Final Task (MAIN).txt`.
3. Update the spreadsheet URL in the `ss` variable.
4. Deploy as **Web App** (Execute as: Me, Access: Anyone).

### 3. Dialogflow
1. Create an agent at [Dialogflow Console](https://dialogflow.cloud.google.com/).
2. Enable **LINE Integration** with your Channel ID, Secret, and Access Token.
3. Enable **Fulfillment Webhook** with the Apps Script Web App URL.

### 4. LINE Developers
1. Create a Messaging API channel at [LINE Developers](https://developers.line.biz/).
2. Paste the Dialogflow Webhook URL into the channel settings.
