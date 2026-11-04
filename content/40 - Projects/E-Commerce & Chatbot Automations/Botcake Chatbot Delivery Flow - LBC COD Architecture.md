---
title: "Botcake Chatbot Delivery Flow: LBC COD Architecture"
tags:
  - chatbot
  - botcake
  - ecommerce
  - automation
  - lbc
  - cod
  - sop
date: 2026-10-28
---

# 🤖 Botcake Delivery Flow: LBC Cash on Delivery (COD) Architecture

An operational blueprint and flow structure for automated conversational checkout via third-party Messenger/Instagram tools (**Botcake**, ManyChat, Chatfuel) for courier deliveries (LBC Express COD).``

---

## 🎯 Strategic Objective
1. **Reduce Friction:** Eliminate messy back-and-forth exchanges by offering clear Quick Reply / Button choices.
2. **Prevent Failed Deliveries (RTS):** Require exact addresses with landmarks and require an address-bearing Valid ID before booking to filter out bogus buyers.
3. **Branching Logic:** Separate **Branch Pick-up (COP/COD)** from **Door-to-Door Delivery (COD)** early in the conversational tree.``

---

## 🧭 Flow Architecture & Message Sequence

```mermaid
flowchart TD
    A["Step 1: Payment & Fulfillment Trigger (COD / Delivery Selected)"] --> B{"Step 2: Delivery Option (Quick Replies)"}
    B -->|Branch Pick Up| C["Step 3A: Branch Details Form"]
    B -->|Door to Door| D["Step 3B: Exact Delivery Address Form"]
    C --> E["Step 4: Identity & Address Verification (Upload Valid ID)"]
    D --> E
    E --> F{"Valid ID Attached?"}
    F -->|Yes| G["Step 5: Order Confirmation Summary + Tag (#order_pending_review)"]
    F -->|No / Text Only| H["Reminder: ID photo with address is required to dispatch"]
    H --> E
`````

---

## 💬 Step-by-Step Message Scripts & Node Configuration

> [!NOTE] Botcake Editor Compatibility
> Messages below feature **Unicode bold text** and contextual emojis designed to display cleanly without formatting breakage inside Facebook Messenger and Instagram DM. Each message includes structured button configurations compliant with Botcake's limit (0 to 3 buttons max).``

---

### Step 1: Fulfillment Confirmation & Delivery Selection
* **Message Type:** Text Card with Quick Reply / Block Buttons
* **Objective:** Establish terms and let customer pick the dispatch channel.
* **Button Count:** 2 buttons (Min: 0, Max: 3)

```text
🎉 Thank you for your order! 

To process your 𝗖𝗮𝘀𝗵 𝗼𝗻 𝗗𝗲𝗹𝗶𝘃𝗲𝗿𝘆 (𝗖.𝗢.𝗗) package via 𝗟𝗕𝗖 𝗘𝘅𝗽𝗿𝗲𝘀𝘀, please select your preferred delivery option below: 👇
```

* **Configured Buttons:**
  * 🔘 **Button 1:** 🏢 LBC Branch Pick Up *(Branches to Step 2A)*
  * 🔘 **Button 2:** 🚚 Door-to-Door Delivery *(Branches to Step 2B)*

---

### Step 2A: Branch Pick Up Route (Branch Pick Up)
* **Trigger:** Customer taps 🏢 LBC Branch Pick Up
* **Botcake Node:** User Input (Text / Multiline Form)
* **Variable Mapping:** {{lbc_branch_name}}, {{customer_fullname}}, {{contact_number}}
* **Button Count:** 1 button (Min: 0, Max: 3) — Quick switch to switch to Door Delivery if tapped by mistake.

```text
🏢 Great choice! Picking up from your local branch is 𝗳𝗮𝘀𝘁𝗲𝗿, 𝘀𝗮𝗳𝗲𝗿, 𝗮𝗻𝗱 𝗮𝘃𝗼𝗶𝗱𝘀 𝗺𝗶𝘀𝘀𝗲𝗱 𝗱𝗲𝗹𝗶𝘃𝗲𝗿𝗶𝗲𝘀. ⚡

Please reply with the following details in 𝗼𝗻𝗲 𝗺𝗲𝘀𝘀𝗮𝗴𝗲: 📋

👤 𝗙𝗨𝗟𝗟 𝗡𝗔𝗠𝗘: 
📱 𝗖𝗢𝗡𝗧𝗔𝗖𝗧 𝗡𝗨𝗠𝗕𝗘𝗥: 
📍 𝗣𝗥𝗘𝗙𝗘𝗥𝗥𝗘𝗗 𝗟𝗕𝗖 𝗕𝗥𝗔𝗡𝗖𝗛: (e.g., LBC SM North EDSA / LBC Ayala Center)
🏙️ 𝗖𝗜𝗧𝗬 / 𝗣𝗥𝗢𝗩𝗜𝗡𝗖𝗘: 

💡 𝗤𝘂𝗶𝗰𝗸 𝗧𝗶𝗽: Please make sure your name matches the valid ID you will present when claiming at the branch!
```

* **Configured Buttons (Optional Helper):**
  * 🔘 **Button 1:** 🚚 Switch to Door-to-Door *(Redirects to Step 2B)*

---

### Step 2B: Door-to-Door Route (Door to Door)
* **Trigger:** Customer taps 🚚 Door-to-Door Delivery
* **Botcake Node:** User Input (Text / Multiline Form)
* **Variable Mapping:** {{customer_fullname}}, {{contact_number}}, {{complete_delivery_address}}
* **Button Count:** 1 button (Min: 0, Max: 3) — Quick switch to Branch Pick Up if customer prefers branch pickup.

```text
🚚 Noted! To guarantee accurate arrival and prevent rider dispatch delays, please provide your 𝗰𝗼𝗺𝗽𝗹𝗲𝘁𝗲 𝗱𝗲𝗹𝗶𝘃𝗲𝗿𝘆 𝗮𝗱𝗱𝗿𝗲𝘀𝘀: 📦

Please reply with the following in 𝗼𝗻𝗲 𝗺𝗲𝘀𝘀𝗮𝗴𝗲: 📝

👤 𝗙𝗨𝗟𝗟 𝗡𝗔𝗠𝗘: 
📱 𝗣𝗥𝗜𝗠𝗔𝗥𝗬 𝗖𝗢𝗡𝗧𝗔𝗖𝗧 #: 
📞 𝗕𝗔𝗖𝗞𝗨𝗣 𝗖𝗢𝗡𝗧𝗔𝗖𝗧 #: 
🏡 𝗖𝗢𝗠𝗣𝗟𝗘𝗧𝗘 𝗔𝗗𝗗𝗥𝗘𝗦𝗦: (House/Unit #, Street, Barangay, City, Province, Postal Code)
📍 𝗡𝗘𝗔𝗥𝗘𝗦𝗧 𝗟𝗔𝗡𝗗𝗠𝗔𝗥𝗞: (e.g., In front of Brgy. Hall, Beside 7-Eleven, Yellow Gate)
```

* **Configured Buttons (Optional Helper):**
  * 🔘 **Button 1:** 🏢 Switch to Branch Pick Up *(Redirects to Step 2A)*

---

### Step 3: Crucial Verification (Address-Matching Valid ID)
* **Condition:** Executes immediately after Step 2A or Step 2B input submission.
* **Botcake Node:** User Input (File / Media upload filter)
* **Tagging:** tag: awaiting_id_upload
* **Button Count:** 2 buttons (Min: 0, Max: 3) — Support & Alternative submission options.

```text
⚠️ ━━━━━━━━━━━━━━━━━━━
🛡️ 𝗙𝗶𝗻𝗮𝗹 𝗦𝘁𝗲𝗽: 𝗦𝗲𝗰𝘂𝗿𝗶𝘁𝘆 & 𝗔𝗻𝘁𝗶-𝗙𝗿𝗮𝘂𝗱 𝗩𝗲𝗿𝗶𝗳𝗶𝗰𝗮𝘁𝗶𝗼𝗻
━━━━━━━━━━━━━━━━━━━ ⚠️

To approve your 𝗖𝗮𝘀𝗵 𝗼𝗻 𝗗𝗲𝗹𝗶𝘃𝗲𝗿𝘆 (𝗖.𝗢.𝗗) shipment and prevent bogus bookings, please reply with a clear 📸 𝗣𝗵𝗼𝘁𝗼 / 𝗦𝗰𝗿𝗲𝗲𝗻𝘀𝗵𝗼𝘁 𝗼𝗳 𝘆𝗼𝘂𝗿 𝗩𝗮𝗹𝗶𝗱 𝗜𝗗 showing your address.

✅ 𝗔𝗰𝗰𝗲𝗽𝘁𝗲𝗱 𝗜𝗗𝘀:
• Driver's License 🪪
• National ID / UMID 🇵🇭
• Postal ID 📬
• Passport / PRC ID 🛂
• Barangay Clearance (with photo & address) 🏛️

🔒 𝗬𝗼𝘂𝗿 𝗣𝗿𝗶𝘃𝗮𝗰𝘆 𝗠𝗮𝘁𝘁𝗲𝗿𝘀: Your details are strictly confidential and used solely for courier dispatch verification.
```

* **Configured Buttons:**
  * 🔘 **Button 1:** ❓ Why do I need an ID? *(Sends quick explainer card)*
  * 🔘 **Button 2:** 🙋 Speak to Live Agent *(Triggers live agent handoff)*

---

### Step 4: Final Order Summary & Auto-Handoff
* **Trigger:** Customer sends the ID image.
* **Botcake Actions:**
  1. Add Tag: COD_LBC_Pending_Review
  2. Send Notification to Admin / Live Agent Inbox.
  3. Pause Automated Sequence (Live Chat Mode).
* **Button Count:** 2 buttons (Min: 0, Max: 3) — Immediate customer self-service actions.

```text
🎉 ━━━━━━━━━━━━━━━━━━━
📦 𝗢𝗿𝗱𝗲𝗿 𝗖𝗼𝗻𝗳𝗶𝗿𝗺𝗲𝗱 & 𝗥𝗲𝗰𝗲𝗶𝘃𝗲𝗱!
━━━━━━━━━━━━━━━━━━━ 🎉

Thank you! We have securely received your delivery details and verification ID. 🙌

🚀 𝗪𝗵𝗮𝘁 𝗛𝗮𝗽𝗽𝗲𝗻𝘀 𝗡𝗲𝘅𝘁:
1️⃣ Our dispatch team is preparing your package for 𝗟𝗕𝗖 𝗘𝘅𝗽𝗿𝗲𝘀𝘀 pick-up.
2️⃣ You will receive an automated message right here containing your 𝗟𝗕𝗖 𝗧𝗿𝗮𝗰𝗸𝗶𝗻𝗴 𝗡𝘂𝗺𝗯𝗲𝗿 once it is encoded.

💬 Need assistance or changes? Tap a button below or reply directly to this chat! 👇
```

* **Configured Buttons:**
  * 🔘 **Button 1:** ✏️ Change Delivery Details *(Allows customer to modify)*
  * 🔘 **Button 2:** 💬 Chat with Human Agent *(Alerts support team)*

---

## ⚙️ Best Practices in Botcake Configuration

| Feature | Recommended Setting | Rationale |
| :--- | :--- | :--- |
| **User Input Timeout** | 4 to 6 Hours | Re-engages customers who abandoned the flow before uploading their valid ID. |
| **Fallback / Human Takeover** | Trigger on keywords: *"Talk to human"*, *"Agent"*, *"Change address"* | Automatically pauses the bot to prevent conflicting automated responses. |
| **Data Attributes** | Store custom attributes (`LBC_Option`, `Full_Address`, `ID_Uploaded_Bool`) | Enables one-click export into spreadsheet/courier shipping bulk tools. |
| **Anti-Fraud Filter** | Flag mismatch between recipient name and ID name | Protects the store from unclaimed shipments and courier return fees. |``

---

## 🔗 Related Notes
* [[Personal Knowledge Hub (Map of Content)]]
* [[Agentic Workflow The Future of Intelligent Automation - Anti Gravity (Larry Mark Somocor)]]
