# Setup guide: your sales messaging bot

This gets your bot reading your WhatsApp and Lark, drafting replies in your voice, and sending only when you say "send". It takes about 20 minutes. You need your phone (with WhatsApp and the Lark app signed in) and your Grok Bot chat.

You do not need to type any commands yourself. Wherever it says "ask your bot", just paste that sentence into your chat with the bot.

## Before you start

- Ask your bot to download the kit onto its computer with this command, which unpacks it into `~/sales-bot-kit`:
  `curl -fsSL https://raw.githubusercontent.com/PaulbCatalyst/sales-messaging-assistant-kit/main/sales-messaging-assistant-kit.tgz | tar xz -C ~`
- Have your phone unlocked and nearby.

## Part 1: WhatsApp

### Step 1. Install

Ask your bot:

> Run `~/sales-bot-kit/install.sh` and tell me when it finishes.

This installs the tools it needs, downloads and builds the WhatsApp bridge, and sets it up to restart on its own after the bot's computer restarts. It does not connect to your WhatsApp yet. Running it twice is harmless.

### Step 2. Set your basics

Ask your bot:

> Open `~/whatsapp-mcp/kit/config.json`. Set my time zone to (your time zone, for example America/Chicago) and my quiet hours to (for example 23:00 to 07:00). Leave routine_url for now.

If there are WhatsApp groups you never want drafts for (community groups, friends, family), tell the bot part of their names and it will add them to `group_deny_needles`. If you only want specific groups, it can use `group_name_needles` instead.

### Step 3. Link your WhatsApp (one time only)

You are linking the bot like you would link WhatsApp Web. Your phone stays the main device.

**Option A: QR code (default)**

1. Ask your bot: "Run `~/whatsapp-mcp/kit/link-whatsapp.sh` and show me the QR code."
2. On your phone: open WhatsApp, go to **Settings > Linked devices > Link a device**.
3. Point your phone at the QR code the bot shows you. The code refreshes about every 20 seconds, so if it expires, ask the bot for the new one.
4. The bot will say "Linked. WhatsApp is connected."

**Option B: pairing code (if scanning is awkward)**

1. Ask your bot: "Run the WhatsApp link with a pairing code for my number (country code + number, digits only)."
   The bot runs `WA_PAIR_PHONE=<your number> ~/whatsapp-mcp/kit/link-whatsapp.sh`.
2. On your phone: **Settings > Linked devices > Link a device > Link with phone number instead**.
3. Type the 8-character code the bot shows you. It expires in a few minutes.

You only do this once. The bot will never relink on its own. If WhatsApp ever logs the bot out, it will tell you and ask before relinking.

### Step 4. Create the message routine

This is what turns a new WhatsApp message into a draft in your chat.

1. Ask your bot: "Create a webhook routine called whatsapp-message-draft that drafts replies using my sales-messaging-assistant skill, and give me its webhook URL."
2. When the routine is created you get a URL and a key. Save the key as a bot secret named exactly **WA_ROUTINE_WEBHOOK_KEY**. Never paste the key into a chat message or a document.
3. Ask your bot: "Run `~/whatsapp-mcp/kit/set-routine-key.sh`." That writes the key to a locked-down file on the bot's computer (`~/.config/sales-bot-kit/routine_key`, mode 0600) so the relay still works after a restart. The bot must not print the key.
4. Tell the bot: "Put the routine URL into routine_url in `~/whatsapp-mcp/kit/config.json` and restart the relay." (The bot runs `stop-bridge.sh` then `start-bridge.sh`.)

Suggested routine instructions (paste into the routine, edit to taste):

> You receive WhatsApp events from my local relay. Follow my sales-messaging-assistant skill. For each chat: read the recent thread from the local database, check who wrote each message, and draft a reply in my voice in chat with the full text. Do not send anything. If all messages are from me, only log deals (never infer amounts). If the event is a whatsapp_bridge_down alert, tell me in plain words. Respect quiet_hours: if true, hold non-urgent items until quiet hours end.

### Step 5. Test it

Have a friend send you a WhatsApp message. Within about two minutes your bot should show you who wrote, what they said, and a draft. Say "send" to send it, or tell the bot what to change.

## Part 2: Lark

Lark does not let the bot receive messages instantly the way WhatsApp does, so the bot checks Lark on a schedule (and whenever you ask).

### Step 1. Log in

1. Ask your bot: "Open Lark web messenger in your browser and show me the login QR code." It opens your company's Lark web address (your bot will ask for it if it does not know it).
2. On your phone, open the **Lark app**, tap the scan icon (usually top right of the Messages tab, or in the + menu), and scan the QR code.
3. Confirm the login on your phone. The bot should now see your chat list.

The bot stays logged in. It will never sign you out.

### Step 2. Choose check times

Tell the bot when to check, for example: "Check Lark at 8 AM, noon, and 4 PM on weekdays." Each check gives you a short list of new messages and drafts for anything that needs a reply.

Good to know: in Lark, opening a chat marks it as read. The bot works from the chat list and only opens a chat when it needs the full conversation to draft a reply.

### The admin option (optional, needs your Lark admin)

If your company's Lark admin prefers, they can create a **custom app** in the Lark developer console. That gives an **App ID** and **App Secret** and can deliver new messages instantly through the `im.message.receive_v1` event. Trade-offs:

- It needs a tenant admin to create and approve the app.
- Messages the bot sends this way come from the app (a bot account), not from you personally.
- The app only sees chats it has been added to.

For one-to-one selling in your own name, the web login above is usually the better fit.

## Part 3: Your voice

The first time you use the bot, it asks you to paste 3 to 5 messages you actually sent to contacts. It writes a short voice guide from them and shows it to you. Correct anything that is off. Every draft follows that guide.

How sending works, always:

1. The bot shows you the full draft.
2. You say "send" (a thumbs-up does not count).
3. The bot reads back exactly what it will send, to whom, and where, then sends it.

## Troubleshooting

| What you see | What to do |
|---|---|
| No drafts for new WhatsApp messages | Ask the bot: "Run `~/whatsapp-mcp/kit/watchdog.sh` and tell me what it says." |
| Watchdog says `no_session` | WhatsApp is not linked yet. Do Part 1, Step 3. |
| Watchdog says logged out | Your phone removed the link (or WhatsApp did). Ask the bot to relink with `link-whatsapp.sh --force-relink`, then scan again. |
| Bot says a send timed out | The bridge may be stuck. The watchdog restarts it within a few minutes; ask the bot to check health and retry. |
| `forward_error ... missing_key` in the relay log | The secret `WA_ROUTINE_WEBHOOK_KEY` is not set, or `set-routine-key.sh` has not been run. Add the bot secret and ask the bot to run `~/whatsapp-mcp/kit/set-routine-key.sh` (Part 1, Step 4). |
| `forward_error ... routine_url_not_set` | `routine_url` in config.json still has the placeholder. |
| Drafts for groups you do not care about | Tell the bot part of the group name to add to the deny list. |
| QR code expired before you scanned it | Ask the bot to run the link again. Each QR lasts about 20 seconds; the bot shows the newest one. |
| Lark shows a login screen | The web session expired. Ask the bot to open Lark and show the QR, then scan it from the Lark app. |
| Nothing works after the bot's computer restarted | Ask the bot to run `~/whatsapp-mcp/kit/boot.sh` in the background. Autostart normally does this for you. |
| WhatsApp on your phone shows a linked device you do not recognize | Remove it on your phone (Settings > Linked devices) and tell the bot. |
