# ☕ Fenjan Café — AI Omnichannel Order & Kitchen Automation (n8n + Groq + Meta + Google Sheets)

An end-to-end AI customer service, order management, and kitchen operations system for **Fenjan Café**. Customers message the café on **Facebook Messenger** or **Instagram DMs** in Egyptian Arabic, English, or Franco-Arabic; **Groq AI** extracts the structured order, **deterministic JavaScript** calculates prices and enforces business hours, **Google Sheets** acts as the live database, and **Telegram + n8n Forms** power real-time kitchen updates.

[![Live Demo](https://img.shields.io/badge/Live_Demo-GitHub_Pages-2ab3b1?style=for-the-badge)](https://adhmmabbas.github.io/Fenjan-Cafe-AI-Automation/)
![n8n](https://img.shields.io/badge/n8n-Self--Hosted-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)
![Groq](https://img.shields.io/badge/Groq_AI-Llama_3.1_8B-F55036?style=for-the-badge)
![Meta Graph API](https://img.shields.io/badge/Meta_Graph_API-Messenger_%26_IG-0081FB?style=for-the-badge&logo=meta&logoColor=white)
![Google Sheets](https://img.shields.io/badge/Google_Sheets-Live_DB-34A853?style=for-the-badge&logo=googlesheets&logoColor=white)
![Telegram](https://img.shields.io/badge/Telegram-Kitchen_%26_Alerts-26A5E4?style=for-the-badge&logo=telegram&logoColor=white)

---

## 📸 System Preview

| Interactive Live Portal | Live Orders Logged in Google Sheets |
| :---: | :---: |
| [<img src="assets/workflow-dm-orders.png" width="440"/>](https://adhmmabbas.github.io/Fenjan-Cafe-AI-Automation/) | [<img src="assets/sheet-orders.png" width="440"/>](assets/sheet-orders.png) |

---

## 🏗️ Architecture & Workflows

The system is organized into two modular workflows inside n8n:

| Workflow | Trigger(s) | Core Responsibility |
| :--- | :--- | :--- |
| **`Fenjan DM Orders`** | `Messenger Trigger`, `Instagram Trigger`, `n8n Form Trigger`, `Schedule Trigger (9 AM)` | Parses DMs via Groq AI, validates prices & hours in JS, manages carts & orders in Google Sheets, notifies kitchen on Telegram, sends kitchen status updates to customers, and publishes daily Facebook specials. |
| **`Fenjan Error Handler`** | `Error Trigger` | Captures any workflow or Meta API delivery failure, logs execution details to the `Errors` sheet tab, and dispatches instant alerts via Telegram. |

### 1. Omnichannel DM Orders, Daily Post & Kitchen Progress
![Fenjan DM Orders Workflow](assets/workflow-dm-orders.png)

* **AI Intent & Extraction (`Basic LLM Chain` + `Groq`):** Understands Egyptian Arabic, English, and Franco-Arabic (`"عايز ٢ سبانش لاتيه توصيل"` or `"3ayez 2 latte"`) and outputs strict JSON (`intent`, `items`, `fulfilment`, `address`, `language`).
* **Zero-Hallucination Business Logic (`Decide Reply`):** Prices, 25 EGP delivery fee, item availability (`Menu` sheet), and Cairo operating hours (`08:00 – 24:00`) are enforced purely in JavaScript.
* **5-Branch Smart Routing (`Switch`):**
  1. `confirm` → Generates Order ID (`F-YYMMDD-XXXX`), writes to `Orders`, closes `Carts`, alerts kitchen on Telegram, and replies to customer.
  2. `cart` → Upserts active items/address into `Carts` (`status: open`).
  3. `clear` → Empties and closes the customer's cart.
  4. `human` → Escalates complaints or unclear messages to staff on Telegram.
  5. `reply` → Handles greetings, menu requests, and missing order details directly.
* **Kitchen Progress Form (`Kitchen Update`):** Staff update status (`preparing`, `out_for_delivery`, `cancelled`) via an n8n Form; the workflow validates the order ID and messages the customer on their original platform.

### 2. Centralized Error Monitoring
![Fenjan Error Handler Workflow](assets/workflow-error-handler.png)

* Linked as the **Error Workflow** for `Fenjan DM Orders` and triggered automatically if Meta rejects a reply (e.g., outside the 24-hour messaging window) or any API call fails.

---

## 🗄️ Google Sheets Schema (`Fenjan Café · Orders by DM`)

| Tab Name | Purpose | Key Columns |
| :--- | :--- | :--- |
| **`Menu`** | Live catalog & keywords | `item_id`, `name_en`, `name_ar`, `category`, `price_egp`, `available`, `keywords` |
| **`Carts`** | Active customer sessions | `customer_id`, `platform`, `items_json`, `fulfilment`, `address`, `total_egp`, `status`, `updated_at` |
| **`Orders`** | Confirmed kitchen orders | `order_id`, `created_at`, `platform`, `customer_id`, `language`, `items`, `subtotal_egp`, `delivery_egp`, `total_egp`, `fulfilment`, `address`, `status`, `updated_at` |
| **`Specials`** | Scheduled promo posts | `date`, `text`, `posted`, `post_id` |
| **`Errors`** | Automated failure log | `time`, `workflow`, `node`, `message`, `execution_url` |

---

## 🚀 Quick Setup (Self-Hosted n8n)

1. **Import Workflows:** Download [`fenjan-dm-orders.json`](workflows/fenjan-dm-orders.json) and [`fenjan-error-handler.json`](workflows/fenjan-error-handler.json) and import them into your n8n instance (`Africa/Cairo` timezone).
2. **Configure Credentials:**
   * **Groq API:** Connect your Groq API key (`llama-3.1-8b-instant`).
   * **Meta Graph API:** Connect `Facebook Graph API (App)` for Messenger/Instagram triggers and `Facebook Graph API` (Page Token with `pages_messaging` & `pages_manage_posts`) for replies and daily posts.
   * **Google Sheets OAuth2:** Point all Google Sheets nodes to your copy of the `Fenjan Café` spreadsheet.
   * **Telegram Bot API:** Add your bot token and kitchen group `chatId`.
3. **Activate:** Publish `Fenjan Error Handler` first, set it as the Error Workflow in `Fenjan DM Orders` settings, then publish `Fenjan DM Orders`.

---

## 👨‍💻 Author

**Adham Abbas** — Computer Science & AI Student | Automation & AI Workflows Engineer  
🔗 GitHub: [@adhmmabbas](https://github.com/adhmmabbas)