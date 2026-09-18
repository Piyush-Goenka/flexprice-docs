---
name: docs
description: >
  Write, extend, and validate Flexprice documentation pages in Mintlify MDX format.
  Use this skill whenever the user asks to write a new doc, add a section to existing docs,
  create a guide, document a feature, or update the Flexprice docs site.
  Trigger when the user mentions "write a doc", "add docs", "documentation for", "create a guide",
  "update the docs", "add to docs", "document this feature", "threshold notifications",
  "alerts doc", or any request to produce or modify an .mdx file in the docs repo.
  This skill knows the Mintlify MDX component set, the docs.json navigation schema,
  the repo layout, the editorial voice, validation commands, and the dev server setup.
---

# Flexprice Docs — Writing and Validation Skill

Write new documentation pages and validate them against the live Mintlify build. Always follow the existing style of the repo — read nearby docs before writing to match tone and component usage.

---

## Repo Layout

```
flexprice-docs/
├── docs/                    ← All documentation pages (.mdx)
│   ├── customers/           ← Customer-related docs
│   ├── wallet/              ← Wallet docs
│   ├── product-catalogue/   ← Plans, Features, Coupons
│   ├── subscriptions/       ← Subscription workflows
│   ├── webhook/             ← Webhook reference
│   ├── event-ingestion/     ← Event & metering docs
│   └── ...
├── api-reference/           ← API Reference tab: introduction, pagination, error-responses, openapi.json
├── images/docs/             ← Screenshots referenced from docs
├── docs.json                ← Navigation config (Mintlify v2), including every API endpoint
├── scripts/sync-api-nav.py  ← Keeps the API Reference sidebar in docs.json in sync with openapi.json
├── .github/workflows/       ← sync-api-nav.yml runs that script in --check mode on PRs and main (flags only, never edits)
└── .claude/skills/          ← This skills directory
```

The docs site is at `https://docs.flexprice.io`. The `docs.json` at repo root controls all navigation.

---

## Step-by-Step: Writing a New Doc

### 1. Read Before Writing

Always read at least two nearby docs before writing — pick pages in the same section or covering similar topics. This ensures consistent tone, component choices, and structure.

```bash
# E.g. if writing a wallet doc:
cat docs/wallet/auto-top-up.mdx
cat docs/wallet/low-balance-alert.mdx
```

Also check `docs/webhook/webhooks.mdx` if the doc involves webhooks, and `docs/product-catalogue/features/wallet-balance-alert.mdx` for alert-related docs.

### 2. Pick the Right File Location

| Topic | Directory |
|-------|-----------|
| Customer management | `docs/customers/` |
| Wallet features | `docs/wallet/` |
| Plans, Features, Coupons | `docs/product-catalogue/` |
| Subscription workflows | `docs/subscriptions/` |
| Integrations | `integrations/<provider>/` |
| Webhooks / events | `docs/webhook/` or `docs/event-ingestion/` |
| Alerts & notifications | `docs/customers/` (customer-scoped) or nearest feature section |

### 3. Frontmatter

Every `.mdx` file starts with:

```mdx
---
title: "Page Title"
description: "One sentence describing what this page covers."
---
```

- `title`: Short noun phrase, title case
- `description`: Shown in search results and page meta — one sentence, no period

### 4. Opening Paragraph

Start with a plain prose paragraph (no heading) that explains what the feature is and why it matters. Keep it to 2–4 sentences. Do not repeat the `description` verbatim.

### 5. Benefits List (Optional)

For feature docs, a bold-bullet benefits list after the opening paragraph is conventional:

```mdx
**Benefits:**

- **Proactive management** — Description of why this matters
- **Granular control** — Description
- **Flexible conditions** — Description
```

### 6. Section Structure

Use `##` for top-level sections and `###` for subsections. Never use `#` (reserved for the page title) or go deeper than `###` in most docs.

---

## Mintlify MDX Components

### Callouts

```mdx
<Note>
  Informational note — use for helpful context or clarifications.
</Note>

<Info>
  Similar to Note but for more prominent informational content.
</Info>

<Warning>
  Use for gotchas, required conditions, or things that will break if ignored.
</Warning>

<Check>
  Use for best practices, recommendations, or "do this" guidance.
</Check>
```

### Steps (for configuration workflows)

```mdx
<Steps>
  <Step title="Navigate to the feature">
    Instructions here.
  </Step>

  <Step title="Configure settings">
    More instructions.
  </Step>
</Steps>
```

### Code Blocks

Single language:
```mdx
```json
{ "key": "value" }
```
```

Multiple languages side by side:
```mdx
<CodeGroup>
```bash cURL
curl ...
```

```javascript JavaScript
fetch(...)
```

```python Python
import requests
```
</CodeGroup>
```

### Tables

```mdx
| Column | Column |
|--------|--------|
| Value  | Value  |
```

Pipe-align all columns. Use bold (`**text**`) in first column when listing settings or fields.

### Cards and Links

```mdx
<Card icon="book-open" horizontal={true} href="/docs/..." title="Related Doc Title" />
```

Use `horizontal={true}` for inline card links at the bottom of a page.

### Frames (screenshots)

```mdx
<Frame>
  ![Alt text](/images/docs/Section/Page/image.png)
</Frame>
```

Only include `<Frame>` blocks when actual screenshots exist in the repo at the referenced path. **Do not include placeholder image references** — broken image links will fail the `mint broken-links` check.

---

## Webhook Payload Sections

When documenting a feature that emits webhooks, follow this structure:

```mdx
### Webhook Payload

**Event type:** `event.name.here`

```json
{
  "event_type": "event.name.here",
  "alert_type": "descriptor",
  "alert_status": "warning",
  ...
}
```

### Webhook Fields

| Field | Description |
|-------|-------------|
| `field_name` | What it contains |
| `nested.field` | Description |
```

Model new webhook events on existing ones — see `docs/wallet/low-balance-alert.mdx` (event: `wallet.alert`) and `docs/product-catalogue/features/wallet-balance-alert.mdx` (event: `feature.wallet_balance.alert`). For usage-based alerts, follow the `customer.usage.alert` / `feature.usage.alert` pattern established in `docs/customers/threshold-notifications.mdx`.

---

## Navigation: docs.json

After writing a new file, **always add it to `docs.json`**.

The navigation lives in `navigation.tabs[0].groups` (the Documentation tab). Structure:

```json
{
  "group": "Group Name",
  "icon": "icon-name",
  "pages": [
    "docs/path/to/page",
    {
      "group": "Sub-group Name",
      "pages": [
        "docs/path/to/nested-page"
      ]
    }
  ]
}
```

**Rules:**
- Flat page paths are relative to repo root, no `.mdx` extension
- Sub-groups use the same `{ "group": "...", "pages": [...] }` shape — no `icon` on sub-groups
- Always add new pages immediately after the most relevant existing page in the same section
- Never create a new top-level group without checking if an existing group is the right home

**Common icon names:** `users`, `wallet`, `webhook`, `layer-group`, `refresh`, `file-text`, `gear`, `bell`, `shield`, `code`, `book-open`, `plug`

---

## API Reference Navigation (docs.json)

The API Reference tab (`navigation.tabs[1]`) does **not** auto-generate its sidebar. Every endpoint is listed explicitly so the sidebar shows one collapsible group per resource, Stripe-style:

```json
{
  "tab": "API Reference",
  "openapi": { "source": "/api-reference/openapi.json", "directory": "api-reference" },
  "groups": [
    { "group": "API Documentation", "pages": ["api-reference/introduction", "..."] },
    {
      "group": "Resources",
      "pages": [
        {
          "group": "Add-ons",
          "pages": [
            "POST /addons",
            "PUT /addons/{id}",
            "DELETE /addons/{id}",
            "GET /addons/lookup/{lookup_key}",
            "GET /addons/{id}",
            "POST /addons/search"
          ]
        }
      ]
    }
  ]
}
```

**How it works:**
- An entry of the form `"METHOD /path"` renders the endpoint page for that operation from `openapi.json`. The path is the spec path, without the `/v1` base URL, with `{param}` placeholders kept as-is.
- Only nested groups collapse with a chevron. Top-level groups are always-open headers. That is why every resource sits inside the single top-level `Resources` group.
- Mintlify stops auto-populating endpoint pages as soon as any explicit `"METHOD /path"` entry exists. An endpoint that is in `openapi.json` but not in `docs.json` does not appear in the sidebar and its page 404s.
- Generated page URLs are `/api-reference/<tag-kebab>/<summary-kebab>` (for example `/api-reference/addons/create-addon`). The tag and summary come from the spec; the sidebar label of the endpoint is the spec `summary`.

**When `openapi.json` changes, `docs.json` must gain or lose the matching `"METHOD /path"` lines.** This is automated:

- `scripts/sync-api-nav.py` compares the spec with the sidebar. It adds missing endpoints to the sub-group whose label matches the tag (creating the sub-group in alphabetical position if none exists), removes entries whose endpoint left the spec, and leaves everything else alone (hand ordering, labels, object pages). New endpoints are placed by kind: create (POST), update (PUT/PATCH), delete, retrieve (GET by id), list, then `/search` and other actions. The map of tag to sidebar label lives in its `LABELS` dict (`Addons` to `Add-ons`, `AlertSettings` to `Alert Settings`).
- `.github/workflows/sync-api-nav.yml` runs it in `--check` mode on pull requests and on pushes to `main`. It never edits files: it fails the check and lists the missing or stale entries (also in the job summary) when `docs.json` and `openapi.json` disagree. Whoever fixes the PR runs the script below.

When `openapi.json` changes in a session, run it yourself so the check passes:

```bash
python3 scripts/sync-api-nav.py          # updates docs.json
python3 scripts/sync-api-nav.py --check  # exit 1 and a list if anything is missing
mintlify validate
```

**Object pages (optional):** to add a Stripe-style "The X object" page at the top of a resource group, create `api-reference/<tag-kebab>/the-<x>-object.mdx` with only frontmatter, then list its path as the first entry of that group:

```mdx
---
title: "The add-on object"
description: "Fields returned for an add-on by the Flexprice API"
openapi-schema: AddonResponse
---
```

`openapi-schema` names a schema under `components.schemas` in `openapi.json`; Mintlify renders its fields and a JSON example. None exist today.

---

## Validation Commands

Run these **before opening a PR**. The same two checks run in CI on every pull request (`.github/workflows/docs-checks.yml`: `mint validate` then `mint broken-links`, on Node 22), together with the API sidebar check in `sync-api-nav.yml`. All three are read-only and must be green before merge. Both use Node 22 (mintlify does not support Node 25+).

### Check for broken links

```bash
PATH="/opt/homebrew/opt/node@22/bin:$PATH" \
  /opt/homebrew/opt/node@22/bin/node \
  /opt/homebrew/lib/node_modules/mint/index.js \
  broken-links
```

- **Pass**: no broken links in your new file — proceed to PR
- **Fail**: fix every broken link reported in your file; pre-existing broken links in other files are not your responsibility
- The most common cause: referencing a screenshot path in `<Frame>` that doesn't exist in `images/`

### Validate build structure

```bash
PATH="/opt/homebrew/opt/node@22/bin:$PATH" \
  /opt/homebrew/opt/node@22/bin/node \
  /opt/homebrew/lib/node_modules/mint/index.js \
  validate
```

- Confirms `docs.json` is valid, all referenced pages exist, and no structural errors
- Must exit 0 with `success build validation passed`; any warning fails CI

### Dev server (visual check)

```bash
PATH="/opt/homebrew/opt/node@22/bin:$PATH" \
  /opt/homebrew/opt/node@22/bin/node \
  /opt/homebrew/lib/node_modules/mint/index.js \
  dev --port 3333
```

The `.claude/launch.json` in this repo is configured to use this exact path. Use `preview_start` with the `"docs"` configuration.

**Node version note:** The system has Node 25 as default (`/opt/homebrew/bin/node`), which is unsupported by mintlify. Always prefix with `PATH="/opt/homebrew/opt/node@22/bin:$PATH"` or use the absolute node@22 binary path.

---

## Editorial Voice

| Do | Don't |
|----|-------|
| "Wallet balance alerts fire when…" | "We've added exciting new alerts that…" |
| "Configure thresholds per wallet" | "Powerful per-wallet configuration" |
| "Set `alert_enabled: true` to activate" | "Simply toggle the switch to enable" |
| Present tense: "Alerts trigger on…" | Future: "Alerts will trigger on…" |
| Second-person: "You can configure…" | First-person: "We allow you to…" |

- **No em dashes**: never use `—` in docs content. Split the sentence, or use a comma, colon, or parentheses instead. Applies to prose, callouts, tables, and bullet lists.
- **No marketing adjectives**: skip "powerful", "flexible", "seamless", "robust", "easy"
- **Name the mechanism**: say what the API field or UI element is called
- **One idea per sentence**: split compound sentences
- **No trailing summaries**: don't end with "In summary, …" or "By using X, you can…"

---

## Checklist Before Opening a PR

- [ ] File is in the right directory (`docs/<section>/`)
- [ ] Frontmatter has `title` and `description`
- [ ] Page is added to `docs.json` in the correct group
- [ ] If `api-reference/openapi.json` changed, `python3 scripts/sync-api-nav.py --check` passes (see "API Reference Navigation")
- [ ] No em dashes: `grep -n "—" docs/path/to/page.mdx` returns nothing
- [ ] No `<Frame>` blocks reference images that don't exist in the repo
- [ ] `mint broken-links` passes with no new errors
- [ ] `mint validate` passes with no warnings
- [ ] Dev server renders the page correctly (check heading hierarchy, code block syntax, table alignment)
- [ ] Internal links use `/docs/...` paths (not relative `../` paths)

---

## Known Pre-Existing Issues (Do Not Fix Unless Asked)

- None at the moment. The old `components/Callout.tsx` react-import warning was removed on 2026-09-18 (the unused `import React` line was dropped), so `mint validate` must exit 0. A new warning means something in the change under review.
