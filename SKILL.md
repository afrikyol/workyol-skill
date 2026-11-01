---
name: workyol
description: Runs Workyol from Claude: prepares WhatsApp campaigns to opted-in lists, reads the WhatsApp/Instagram/Messenger inbox and drafts replies, adds auto-replies and comment-to-DM rules, and schedules Reels, photos, LinkedIn posts and YouTube videos. Use when the person asks about WhatsApp campaigns, an Instagram or Messenger inbox, auto-replies, comment-to-DM, or scheduling social posts with Workyol.
---

# Workyol

Workyol is a program that runs on the person's own computer. It works with Meta's official platforms (WhatsApp Business Platform, Instagram, Messenger), with LinkedIn's API and, where open, YouTube's API. This skill tells you how to use its MCP tools.

## When to use

Use this skill when the person wants to:
- send a WhatsApp template campaign to a contact list that opted in;
- read unread replies (WhatsApp, Instagram, Messenger) and answer them;
- add keyword auto-replies, or comment-to-private-message rules;
- schedule Reels, photos or carousels (Instagram and Facebook), LinkedIn posts, or YouTube videos;
- check their Workyol setup or numbers.

Do not use it for people who did not agree to be contacted.

## Requirement

1. Workyol is installed and **running** on the computer (it opens at `localhost:4747`).
2. Workyol's MCP server is connected to this assistant (see README: in Workyol, Help, "Use Workyol from Claude or another AI assistant", Copy).

**First step in every session:** call `check_setup` (what is done / missing in Set up) and `get_overview` (contacts, messages, unread replies, waiting posts). If the tools do not exist or answer "Workyol is not running", stop and tell the person to start Workyol and connect the MCP server. Do not guess.

Most tools take an optional `account` (name, Facebook Page or @instagram); without it the default account is used. Use `list_accounts` first when there are several brands or clients.

## Tools

Overview and setup
- `list_accounts`: the accounts (brands/clients) and what each has connected.
- `get_overview`: contacts, messages sent/delivered/read, unread replies, connected Page/Instagram/WhatsApp, waiting posts.
- `check_setup`: what is done and what is missing in Set up (licence, live mode, Meta app, WhatsApp, Page + Instagram, webhooks). Use it to tell the person the next step.
- `quick_connect`: connect a client's Facebook Page (and its Instagram) as a Workyol account in one step, from a token the person copied in Graph API Explorer. Parameters: `token`, `name`, `business_type` (general, dental, realestate, wholesale, restaurant, salon, school, none), `page_id`, `language` (en, fr, tr, ar, es), `confirm`. Ready-made automations are added switched OFF.

Inbox
- `list_replies`: messages people sent, newest first, grouped by person. Parameters: `unread_only`, `limit`.
- `send_reply`: answer one person on the same channel. Parameters: `channel` (whatsapp, instagram, messenger), `to` (the value from `list_replies`), `text`, `confirm`. WhatsApp allows free text only within 24 hours of the person's last message.

Automations
- `list_automations`: keyword auto-replies and comment-to-DM rules.
- `add_automation`: parameters `type` (`auto_reply` or `comment_to_dm`), `keywords`, `channel`, `reply` (auto_reply), `file_link` (optional https link to a picture, video, audio or PDF), `private_message` and `public_reply` (comment_to_dm), `name`, `confirm`. `{name}` in a text becomes the person's name.

WhatsApp campaigns
- `list_contact_lists`: lists of people who agreed to hear from the business.
- `list_templates`: WhatsApp templates approved by Meta (campaigns can only use these), with variables.
- `list_campaigns`: campaigns with sent / delivered / read / failed.
- `create_campaign`: send or schedule an approved template to an opted-in list. Parameters: `list_id`, `template_name`, `template_language`, `name`, `mapping`, `scheduled_at`, `confirm`. Without `confirm` it returns a preview (how many receive it, how many are skipped and why).

Instagram and Facebook posts
- `list_posts`: scheduled and published posts (filter `status`: waiting, published, failed, all).
- `schedule_post`: a Reel (`video_path`: vertical MP4, 9:16, max 100 MB, optional `cover_path`) or pictures (`image_paths`: 1 = photo, 2 to 10 = carousel; Instagram takes JPG only). Also `caption` (max 2200 characters), `at` (ISO 8601 with time zone), `platforms` (instagram, facebook), `from_workyol_com`, `confirm`.
- `import_content_plan`: schedule a whole plan from a folder with a CSV sheet (columns date, time, caption, files, optional channels) and the media. Without `confirm` it lists every row with its status. A row is never scheduled twice.
- `delete_post`: remove a scheduled post (a published one stays online).

LinkedIn (the person's own profile)
- `linkedin_status`: connected or not, days of access left (60 days, then the person clicks Reconnect), posts waiting.
- `linkedin_schedule_posts`: one or many posts. Each: `text` (max 3000 characters, plain text, hashtags allowed), `at`, optional `media_path` (JPG/PNG picture or MP4), optional `lang`. Max 60 posts per call.
- `linkedin_list_posts`: scheduled, published and failed posts.
- `linkedin_delete_post`: remove a scheduled post (a published one stays on LinkedIn).

YouTube (only where YouTube is open for this licence; at the time of writing it is not open to customers yet, and the tools then answer "YouTube is not open yet")
- `youtube_status`, `youtube_playlists`, `youtube_list`, `youtube_schedule` (MP4/MOV up to 2 GB; title, description, tags, `publish_at`, `privacy`, `thumbnail_path`, `playlist`, `first_comment`; 3 uploads a day per channel, extra videos wait for the next day), `youtube_delete`.

## Workflows

### Prepare a WhatsApp campaign
1. `check_setup`, then `list_contact_lists` and `list_templates`.
2. Pick the list and an approved template with the person. Only lists with opt-in are shown.
3. Call `create_campaign` **without** `confirm`. Show the preview: who receives it, who is skipped and why.
4. Only if the person says yes, call it again with `confirm: true`. Offer `scheduled_at` if they want a later time.
5. Later: `list_campaigns` to report sent / delivered / read / failed.

### Answer unread replies
1. `list_replies` with `unread_only: true`.
2. Summarise by person and channel. Draft an answer for each.
3. Show every draft. Send each one with `send_reply` and `confirm: true` only after the person approves it.
4. On WhatsApp, if the last message is older than 24 hours, a free text will fail: tell the person a template is needed.

### Add a keyword auto-reply or comment-to-DM
1. `list_automations` to avoid duplicates.
2. Show the rule (keywords, channel, reply) as a table.
3. After a yes, `add_automation` with `confirm: true`.

### Schedule a week of Reels
1. Ask for the video files, captions and times (with time zone).
2. `list_posts` to see what is already waiting.
3. Show the full schedule. After a yes, call `schedule_post` for each post with `confirm: true`. Or, for a folder with a CSV sheet, use `import_content_plan` (preview first).

### Schedule a month of LinkedIn posts
1. `linkedin_status`: if not connected, or too few days left, tell the person to connect or reconnect in Workyol.
2. Write the posts in plain text (no markdown or bold). Show them all with their dates.
3. After a yes, `linkedin_schedule_posts` with `confirm: true`.

## Safety rules (follow them always)

- **Nothing is sent, published or changed until the person says yes.** Call the tool without `confirm` first, show the summary, and pass `confirm: true` only after a clear yes to that exact action.
- **Opted-in lists only.** Campaigns go only to contact lists imported in Workyol with the opt-in box ticked. No tool can add phone numbers. Never message people who did not ask.
- No spam, no bulk messages to strangers. Respect STOP: people who reply STOP are skipped by Workyol.
- You never see access tokens, the App secret or the licence key. Do not ask for them.
- Workyol's own rules still apply: the 24-hour WhatsApp window, opt-outs, approved templates, licence limits, and Meta's daily limits.
- Meta only allows a business to answer people who wrote first on Instagram and Messenger. There are no campaigns on those channels.
- Do not promise results such as "no ban". Say only that Workyol uses the official APIs.

## Example prompts

- "Check my Workyol setup and tell me what is missing."
- "How many unread replies do I have? Summarise them and draft answers."
- "Prepare a WhatsApp campaign with the approved template 'order_ready' for my 'Customers 2026' list. Show me the preview first."
- "Add an auto-reply: when someone writes CATALOG, answer with this text and send my PDF."
- "When someone comments PRICE under my post, send them a private message with our prices."
- "Schedule these 5 videos as Reels on Instagram and Facebook, one each day at 18:00."
- "Import the content plan from the folder C:\plans\november and show me the preview."
- "Schedule 12 LinkedIn posts about dental care, two per week, and show them to me first."
