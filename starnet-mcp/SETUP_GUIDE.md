# Connecting your StarNet agents to tools (MCP)

MCP is a plug system: each "server" in `mcp_config.json` is one plug (a database, a payment
system, a CRM). Your agents can only use the plugs you hand them.

## 1. Where to save the files

| Computer | Folder |
|---|---|
| Windows | `C:\Users\YOUR_NAME\StarNet\config\mcp_config.json` |
| Mac | `/Users/YOUR_NAME/StarNet/config/mcp_config.json` |
| Linux | `/home/YOUR_NAME/StarNet/config/mcp_config.json` |

Also create a shared work folder the agents may read/write: `.../StarNet/shared-files/`
and put its full path in the `files` entry of the config.

If StarNet's settings screen asks for an "MCP config path" or has a JSON box, point it at /
paste this file. If StarNet expects a different filename or folder, its settings or docs win.

## 2. Install the two helpers (once)

- Node.js LTS from nodejs.org (gives you `npx`)
- uv from docs.astral.sh/uv (gives you `uvx`, used by `web-fetch`)

## 3. Fill in the CHANGE_ME values

| Plug | What it does | Where to get the key |
|---|---|---|
| postgres | Your business database (orders, customers, inventory) | Your DB host. Create a user `starnet_agent` with only the rights it needs. Note: this server is read-only. |
| files | Reads/writes documents in shared-files | Just the folder path |
| stripe | Payments, invoices, digital product sales | Stripe Dashboard > Developers > API keys. Use a **restricted** key (`rk_...`), start in test mode. |
| hubspot-crm | Contacts, deals, support tickets | HubSpot > Settings > Integrations > Private Apps |
| slack | Team alerts and messages | api.slack.com/apps > your bot > OAuth token |
| web-fetch | Reads web pages (job boards, competitor prices) | No key |
| memory | Long-term notes shared by agents | No key |

Delete any block you don't use. For Shopify, WooCommerce, Gmail, etc., add a block in the same
shape using that vendor's official MCP server (check their docs for the exact package name).

**Never commit real keys to GitHub.** Keep the filled-in copy only on your computer.

## 4. Check it works

Restart StarNet. Each plug should show as connected/green. If one fails, run its command
in a terminal (e.g. `npx -y @stripe/mcp --tools=all`) to see the error.

## 5. Assign tools to each agent

Suggested split (rename to match your crew):

| Agent | Tools |
|---|---|
| Central Brain (orchestrator) | memory, slack, postgres (read) |
| Retail / Commerce agent | postgres, stripe, files |
| Customer Service agent | hubspot-crm, postgres, slack, memory |
| Income / Opportunities agent | web-fetch, files, stripe, memory |

In StarNet, open each agent > Tools/Integrations, tick only its tools, then paste its prompt:

**Central Brain**
```
You are the Central Brain of StarNet. You receive every request, decide which specialist
handles it (Retail, Customer Service, or Income), and pass it on with clear instructions.
Use `memory` to store decisions and daily summaries. Use `slack` to post a daily report.
You may read `postgres` for overview numbers. You never charge money, refund, or email
customers yourself; specialists do that, and anything over $100 or irreversible needs
my approval first.
```

**Retail / Commerce agent**
```
You run retail operations. Use `postgres` to check orders, stock levels and sales.
Use `stripe` to create invoices, payment links and check payments. Save reports
to `files`. Flag items with stock below 10. Ask for approval before any refund,
price change, or charge over $100. Report results back to the Central Brain.
```

**Customer Service agent**
```
You handle customer support. Look up the customer in `hubspot-crm` and their orders in
`postgres` before replying. Log every conversation as a note or ticket in HubSpot.
Be friendly and brief. Escalate angry customers, legal threats, or refund requests to
a human via `slack`. Never promise refunds or discounts without approval.
```

**Income / Opportunities agent**
```
You find and manage digital income. Use `web-fetch` to research jobs, gigs, and product
ideas; save findings to `files` as a weekly shortlist with links and estimated value.
Use `stripe` to create payment links for digital products I approve. Remember what
worked in `memory`. Never sign up for services, apply, or spend money without my OK.
```

## 6. Safety checklist

- Start with Stripe **test** keys; switch to live only after a week of good results.
- Give the database user the least access possible.
- Keep human approval on money, refunds, and outgoing customer emails.
- Rotate any key you think was exposed.
