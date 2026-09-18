# CloseBot Calendar Integration: Connect GoHighLevel or HubSpot, Book the Right Calendar, and Fix Bookings That Never Land

Most people typing this into Google have one of two things going wrong. Either they're trying to work out whether CloseBot can book to their calendar at all, or the agent just told a lead "you're all set for Tuesday at 10" and nothing showed up on the calendar.

Both questions have the same answer underneath: CloseBot doesn't have a calendar of its own. It books into the calendar that lives inside whatever CRM it's connected to. Once that clicks, the rest of the setup makes a lot more sense, including the parts that quietly break bookings.

## There's no "CloseBot Google Calendar integration" — the calendar comes from your CRM

In CloseBot, the thing you connect isn't a calendar, it's a **Source**. A source is your CRM, and the docs describe the supported ones plainly: GoHighLevel sub-account, LeadConnector, HubSpot, and a generic Webhook.

That architecture is deliberate. As CloseBot's own documentation puts it, it doesn't integrate directly with Facebook Messenger, SMS, or WhatsApp — it piggybacks on whatever channels your CRM already handles. Calendars work the same way. The booking action pulls availability from a calendar **inside your source**, which is why the calendar ID field in the booking node is described as "provided by your source's calendar (found within the source CRM)."

So if you're imagining a setup where you point a bot at Calendly and let it loose, that's not this. What you get instead:

- If you already run GoHighLevel or HubSpot with a working calendar, you're a few clicks away.
- If your leads live in Instagram DMs and you have no CRM, you're actually shopping for two products, not one.
- If your CRM calendar syncs out to Google or Outlook at the CRM level, bookings follow that sync — but that's your CRM's job, not CloseBot's.

## Before you touch the booking node

Booking failures usually trace back to something missing **upstream** of the booking step. Get these in order first:

1. **A live calendar in your CRM.** Not a draft, not a deleted one. The troubleshooting doc is blunt about this because it happens.
2. **A phone number or email on the contact.** Conversational booking needs at least one of them on the contact record. If your flow doesn't collect it, add an objective before the booking action to grab it.
3. **A time zone on the contact**, if you book people outside your own time zone. More on why below.
4. **The source connected with field-write permission.** During the OAuth step there's a checkbox to allow CloseBot to create and update fields. Tick it, or your agent can't write qualification data back to the contact.

## Connecting it: source first, booking action second

### Step 1 — Add the source

Open **Sources → New Source**. You'll see a tile per supported integration. Pick HighLevel Sub-Account, LeadConnector, or HubSpot and hit Connect. An OAuth window opens; choose the workspace or sub-account you want to authorize, approve the permissions, then click **Add Source** back in CloseBot. The HubSpot flow is the same shape, with an extra step to name the account.

Read the "Sources" list afterwards. That's your confirmation the connection landed.

### Step 2 — Drop in the booking action

Inside your job flow, the booking action is what tells the agent "book an appointment here." It's a node, not a setting, so where you place it in the flow matters — which brings us to the single most common misconfiguration.

### Step 3 — Pick the calendar: by name, or by permanent ID

You've got two options, and the choice affects how flexible your agent is:

- **Calendar name.** Select it from the dropdown and CloseBot auto-fills the Title and Short Description. Fast, and fine for most single-calendar setups.
- **"Other – Use Calendar ID."** This reveals a field where you paste the calendar's **Permanent ID**. This is the option that lets one agent book into different calendars dynamically.

Either way, you can book to multiple sources with the same configuration — by name or by ID.

One field worth not skipping: the **Short Description**. Keep it short and specific, like "Book a 30 minute in-person appointment." It's supposed to say what kind of meeting and how long, nothing more.

### Step 4 — Advanced settings and the failure tag

The advanced field is for context the agent should only use while working on this booking. The docs' advice is "less is more" here, which is consistent with how the whole system behaves: extra instructions leak into other parts of the conversation.

You can also attach a **tag** to the contact when a booking fails. CloseBot counts two things as failures:

- No response from your CRM or calendar when checking availability
- No open slots on the calendar the agent is trying to book into

Tagging failures is a cheap way to spot a broken calendar before you lose a week of leads.

### Step 5 — Time zones, if your leads aren't local

Conversational booking checks the contact's time zone first. If the contact record doesn't have one, it falls back to your source's time zone. For a local business booking local people, you can leave that alone.

If people book from other time zones, add an objective **before** the booking action to collect and write the contact's time zone. Interview-style: ask where they're based, update the field, then let the booking run.

### Step 6 — Turn rescheduling on if you want it

Conversational rescheduling is **off by default**. Turn it on under **Job Flow Settings → Important Business Information**. Once enabled, the agent can reschedule any appointment it finds on the contact — including appointments it didn't book itself. That's either exactly what you want or a surprise you don't, depending on how your team handles rescheduling.

## One agent, several calendars: three ways to route it

If you sell more than one type of meeting — a phone call versus an in-person visit, or two different locations — the docs lay out three routing patterns.

| Method | How it works | Best for |
| --- | --- | --- |
| **Agent Node** | You explain in the node which calendar the agent should book to, and it handles the choice | The most open-ended setups, where the branch logic isn't worth hardcoding |
| **True/False or Switch** | The agent asks a routing question and the flow branches | One-time bookings — initial sales calls, phone vs. Zoom, in-person vs. virtual |
| **Custom Scenario** | The flow listens for a specific intent, then follows a path tied to a specific calendar | Repeat customers — salons, clinics, mechanics, anything where people rebook monthly or quarterly |

The docs' own guidance is worth repeating: use **true/false or switch** when the AI should "get in, qualify, and get out really quickly," and **custom scenarios** when you want a returning contact to be able to say "I'd like to book again" and land in the right place.

There's no hard limit sitting in the middle of this. In a GoHighLevel community thread, someone asked whether CloseBot could handle roughly 80 calendars across 12 groups; CloseBot's reply was that it handles unlimited calendars through the job flow builder. That's the vendor answering about its own product, not an independent test — but the mechanism matches what the docs describe.

## When the agent confirms a booking and nothing appears

This is the frustrating one, and CloseBot's troubleshooting doc names the most likely cause first: **the agent wasn't actually supposed to be booking yet.**

CloseBot actions run in sequence. Agents are blind to your availability **unless they're actively on a booking objective**. If you mention "appointment" or "booking" in Important Business Information, in Why the Conversation is Happening, or anywhere else outside the booking node, the agent will happily talk about slots it can't see.

How to check: open the message in the dashboard, see which goal the bot was pursuing at that point, and look for the calendar icon. Hovering it shows the availability the agent pulled in. No icon means it wasn't on a booking objective, and any booking talk was improvising.

Other entries in the same doc, in rough order of how often they bite:

- **No slots found when slots exist.** Use the **permanent** Calendar ID, not a name, if you're on the custom ID route. Confirm the calendar isn't deleted or in draft. If several sources feed the same agent, the calendar name must match **exactly** across all of them.
- **The agent offers times that aren't actually open.** Hit the reasoning button on the message to see the logs, including the time zone and availability it used. Then open that calendar in your CRM and compare its availability settings with what the log shows.
- **Time zone drift.** Contact time zone wins if it's set. Otherwise it's the location time zone. If the location time zone can't be read, it falls back to Eastern Time — worth knowing if your bookings look shifted.
- **The agent mixing up weekdays and dates.** CloseBot posted a dated note in the troubleshooting article: agents were saying "today is Tuesday the 7th" when it was the 8th, apparently isolated to Anthropic as the provider. Their recommended fix was switching providers.

A related point from a third-party review of CloseBot: when a calendar throws an error mid-booking, the agent retries rather than replying with the "sorry, that slot is taken" dead end. That review attributes an up-to-20%-more-bookings figure to CloseBot's own reporting — treat it as a vendor claim, but the retry behavior itself shows up in CloseBot's own feature comparisons.

## Rules that keep the calendar connection healthy

Three habits prevent most of the above:

- **Keep "appointment" language inside booking nodes only.** This is the single biggest cause of an agent inventing availability.
- **Enforce the contact phone/email rule in every flow** that ends in a booking, using an objective rather than hoping the CRM already has it.
- **Watch the failure tag.** If it starts filling up, your calendar settings changed, not your AI.

## What the calendar connection actually costs

CloseBot's pricing has two shapes: all-inclusive business plans where message costs sit inside the base price, and agency plans built around rebilling. Here's the current lineup from the official plans page.

| Plan | What you get | Price | Billing cycle | Get started |
| --- | --- | --- | --- | --- |
| **Free** | 100 messages/month, 1 agent, 1 user seat, 1 MB upload storage, unlimited account connections | $0 | Always free | [Start on CloseBot's free plan](https://app.closebot.com/a?fpr=li87) |
| **Core – Business** | Message costs included; base 500 messages/month with higher ceilings selectable (500 up to 100K+); 15+ templates, 50+ on annual; human support; extra seats $5 each; extra storage as an add-on | $64/mo monthly, or $53/mo billed annually as $640/yr | Monthly or annual | [See the Core business plan](https://app.closebot.com/a?fpr=li87&plan=business&toggle=monthly) |
| **Core – Agency** | Unlimited agents across unlimited sources; messages billed at **$0.012 each and rebillable**; white-label client portal; rebill all costs; same seat and storage add-ons | $397/mo monthly ($331/mo equivalent on annual billing) | Monthly or annual | [Compare the agency plan](https://app.closebot.com/a?fpr=li87&plan=agency&toggle=monthly) |
| **Growth** | Custom: HIPAA compliance, quarterly audits, 99.99% priority uptime, priority support, SLAs, 50+ templates | Custom quote | Contract | [Ask CloseBot about Growth](https://app.closebot.com/a?fpr=li87) |

A few pricing details that matter more than the headline numbers:

- **The free plan is genuinely free forever** as long as you stay at 100 messages a month. Over that, it's $0.08 per message.
- **Business plan message costs are included.** If you exceed your ceiling, you enable wallet-based overage protection rather than getting a surprise invoice.
- **One message equals one segment**, unless you switch on the Agent Node's "unlimited potential" — heavy tools and long instructions move you to token-based billing, and a single message can consume several segments.
- **No bring-your-own API key.** CloseBot frames this as a security decision. Model spend sits inside the plan.
- **No refunds**, but every paid plan carries a 7-day trial, and plans are month-to-month with no contract.
- **Annual billing gives you roughly two months free** ($64 down to $53, $397 down to $331).

Worth flagging: CloseBot's older help doc still lists agency messages at $0.006, while the current plans page FAQ states $0.012. Trust the pricing page — the docs lag.

And the cost people forget: **the CRM underneath.** A solo business running 500-ish messages a month on a $64 business plan still needs a GoHighLevel or HubSpot subscription behind it. On the agency side, though, that's the point — the per-message cost is a line item you rebill at your own markup, with the white-label portal attached.

## Who should bother with this setup

The honest split, based on what the product is built to do and what reviewers keep pointing out:

**It's a good fit if** you already run GoHighLevel or HubSpot, you're losing leads to slow follow-up, and your conversion goal is a booked meeting. Agencies reselling AI setting get the most out of it — rebilling and white-labeling are core to the agency plan rather than add-ons.

**It's the wrong tool if** you have no CRM and your pipeline is entirely Instagram and WhatsApp DMs. You'd be buying an agent plus a CRM to run a job a DM-native tool handles alone. Also skip it if you want a fixed all-in monthly price with nothing underneath, or you want to plug in your own model API key to control spend.

Also worth being clear-eyed: this fills your calendar. It doesn't close deals. High-ticket negotiation still needs a human on the call.

## Quick answers to the questions people actually search

**Can CloseBot book to Google Calendar?** Not directly. It books to calendars inside your connected CRM, and whether that syncs outward to Google or Outlook is your CRM's behaviour.

**Does the agent need a phone number or email to book?** Yes — at least one of the two on the contact record. Add an objective to collect it beforehand if your flow doesn't already.

**Why did it say "booked" with nothing on my calendar?** Nine times out of ten, "appointment" or "booking" is mentioned somewhere outside the booking node, so the agent talked about availability it couldn't see.

**Can one agent use more than one calendar?** Yes, three documented ways: Agent Nodes, true/false or switch branches, and custom scenarios keyed to specific calendars.

**Is rescheduling automatic?** No. It's off by default; enable it under Job Flow Settings → Important Business Information.

## The short version

CloseBot's calendar integration is a CRM integration that happens to end in a booking. Connect the source, place the booking node where it belongs, decide between calendar name and permanent calendar ID, and keep appointment language out of every other field. Do that and conversational booking is genuinely hands-off.

Do it halfway and you'll get the most common failure in the product — an agent confidently offering times it never checked. If you want to test the calendar behaviour before paying, the free plan's 100 messages are enough to run a booking flow end to end:

👉 [Try CloseBot's free plan and test a booking flow yourself](https://app.closebot.com/a?fpr=li87)
