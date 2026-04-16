# Policy Inbox Tracker

A purpose-built inbox management tool for policy and research teams. Emails arrive, AI classifies them, your team reviews and acts — all in one place.

**[View live demo →](https://pghook93.github.io/policy-inbox-demo/)**

---

## What it does

Policy teams receive a constant stream of government announcements, consultation requests, funding opportunities, and sector updates. This tool gives the whole team a single structured view of that inbox — with clear ownership, read tracking, program area tags, and full follow-up visibility.

### Core features

- **AI classification** — on refresh, new emails are automatically categorised by type (e.g. Government announcement, Submission request) and tagged with one or more program areas. An admin reviews and confirms before publishing to the team.
- **Staff assignment** — tag staff as FYI or Action on any email. Each person sees their workload at a glance.
- **Action status tracking** — Not started / In progress / Done, visible across the whole team.
- **Session picker** — no passwords. Staff select their name on load. Action item badges show unread items per person.
- **Program area tags** — multiple tags per email, fully configurable. Filter the inbox by area in one click.
- **Person filter** — filter the entire inbox by staff member to see their assignments.
- **Expandable email view** — read the full email in-app without switching tools.
- **Admin overview table** — one view shows every email × every staff member: who has read it, their action status, and who owns what.
- **Overflow menu** — admins can edit, recategorise, or remove any email via a clean action menu.
- **Unreviewed state** — emails fetched on refresh appear as unreviewed (amber badge) until the admin confirms classification. Only published emails are visible to the team.

---

## Tech

This repo contains two things:

| Path | What it is |
|---|---|
| `index.html` | Self-contained prototype — single HTML file, no build step, runs in any browser |
| `inbox-digest/` | React/TypeScript component package for integration into an existing dashboard |

The React component is built for integration into a **React + TypeScript + Vite + Express + Drizzle ORM + SQLite** stack (e.g. the JSS Impact Dashboard). See setup steps below.

---

## Running the prototype

No setup required. Just open `index.html` in a browser — or visit the [live demo](https://pghook93.github.io/policy-inbox-demo/).

The prototype uses mock data and simulates the full workflow including AI classification, session switching, and admin actions.

---

## Integrating the React component

The `inbox-digest/` folder contains everything needed to drop the Policy Inbox Tracker into an existing dashboard as a new tab.

### Files

| File | Purpose |
|---|---|
| `InboxDigest.tsx` | Main React component — drop into `client/src/components/` |
| `schema.additions.ts` | Two new SQLite tables to add to `shared/schema.ts` |
| `routes.additions.ts` | Server routes to add to `server/routes.ts` |
| `Home.tsx.diff` | Changes to apply to `client/src/pages/Home.tsx` |
| `Sidebar.tsx.diff` | Changes to apply to `client/src/components/Sidebar.tsx` |
| `SETUP.md` | Full step-by-step integration guide |

### Quick start

**1. Copy the component**
```bash
cp inbox-digest/InboxDigest.tsx <your-project>/client/src/components/
```

**2. Add schema tables**

Paste the two table definitions from `schema.additions.ts` into `shared/schema.ts`, then run:
```bash
npx drizzle-kit push
```

**3. Add server routes**
```bash
cp inbox-digest/routes.additions.ts <your-project>/server/inboxRoutes.ts
```
Then in `server/routes.ts`:
```ts
import { registerInboxRoutes } from "./inboxRoutes";
// inside registerRoutes():
registerInboxRoutes(app);
```
Install the Gmail dependency:
```bash
npm install googleapis
```

**4. Apply diff files**

Follow the comments in `Home.tsx.diff` and `Sidebar.tsx.diff`. Each diff is annotated with exactly where to paste.

**5. Set environment variables**
```env
GMAIL_CLIENT_ID=your_google_oauth_client_id
GMAIL_CLIENT_SECRET=your_google_oauth_client_secret
GMAIL_REDIRECT_URI=https://your-domain.com/api/inbox/oauth/callback
```

**6. Connect the Gmail inbox (one-time)**

Visit `https://your-domain.com/api/inbox/oauth/start` and authenticate with the central Gmail address. Tokens are saved to the database and auto-refresh. See `SETUP.md` for the full Gmail OAuth setup walkthrough.

---

## Email forwarding model

The recommended approach for client deployments:

1. Create one Gmail address (e.g. `policyinbox@yourorg.com.au`)
2. Client sets up forwarding rules in their existing inbox for relevant senders — government mailing lists, sector newsletters, etc.
3. Connect that Gmail address via OAuth
4. The app reads that inbox — no client credential sharing required

For each new client: duplicate the repo, update `INBOX_CONFIG` at the top of `InboxDigest.tsx`, and connect a different Gmail account.

---

## Customising per client

Everything you need to change between deployments is in `INBOX_CONFIG` at the top of `InboxDigest.tsx`:

```ts
export const INBOX_CONFIG = {
  inboxLabel: "Policy Inbox",       // displayed in the header
  programAreas: [                   // AI classification categories
    "NDIS and foundational supports",
    "Safeguarding and human rights",
    "Housing and infrastructure",
    // ...
  ],
  emailTypes: [                     // AI classification categories
    "Government announcement",
    "Submission / consultation request",
    "Funding opportunity",
    // ...
  ],
  fetchDaysBack: 7,                 // how far back to pull on refresh
};
```

Nothing else needs to change between deployments.

---

## Hiding from clients during development

In `Home.tsx`, set:
```ts
const showInboxDigest = false;
```
This removes the Mail icon from the sidebar entirely. The component and routes remain in the background — flip to `true` when ready to reveal.

---

## Admin account

The admin user is set by `ADMIN_USER_ID` at the top of `InboxDigest.tsx`. The admin sees the unreviewed email queue, can confirm or edit AI classifications, and has access to the full staff overview table.

---

*Built by [Policy Patch](mailto:penny@pennyhookconsulting.com)*
