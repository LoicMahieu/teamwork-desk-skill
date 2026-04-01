---
name: teamwork-desk
description: Query and interact with Teamwork Desk API v2 for support ticket management. Use when the user asks to list tickets, read ticket details, read messages or notes, post notes, or perform any Teamwork Desk helpdesk operation.
---

# Teamwork Desk API

Interact with Teamwork Desk API v2 via `curl` commands.

## Configuration

Requires two values (check env vars, ask user if missing):

| Variable | Description | Example |
|---|---|---|
| `TEAMWORK_DESK_DOMAIN` | Subdomain | `mycompany.teamwork.com` |
| `TEAMWORK_DESK_API_KEY` | API key (Bearer token) | `tkn_...` |

Base URL: `https://{TEAMWORK_DESK_DOMAIN}/desk/api/v2`

Auth header: `Authorization: Bearer {TEAMWORK_DESK_API_KEY}`

## Endpoints

### 1. List tickets

```bash
curl -s "https://${TEAMWORK_DESK_DOMAIN}/desk/api/v2/tickets.json?orderBy=createdAt&orderMode=desc&pageSize=20" \
  -H "Authorization: Bearer ${TEAMWORK_DESK_API_KEY}" | jq .
```

**Common filters** (pass via `filter` query param, URL-encoded JSON):

```bash
# Tickets created in the last 7 days
FILTER='{"createdAt":{"$gt":"2025-01-01T00:00:00Z"}}'
curl -s "https://${TEAMWORK_DESK_DOMAIN}/desk/api/v2/tickets.json?orderBy=createdAt&orderMode=desc&pageSize=20&filter=$(python3 -c "import urllib.parse; print(urllib.parse.quote('${FILTER}'))")" \
  -H "Authorization: Bearer ${TEAMWORK_DESK_API_KEY}" | jq .
```

**Filter operators**: `$eq`, `$ne`, `$lt`, `$lte`, `$gt`, `$gte`, `$in`, `$nin`, `$and`, `$or`, `$contains`

**Filterable fields**: `id`, `subject`, `status`, `state`, `priority`, `inbox`, `agent`, `contact`, `company`, `customer`, `source`, `type`, `createdAt`, `updatedAt`, `deletedAt`, `sla`, `slaBreachedAt`

**Includes** (embed related data): add `&includes=customers,inboxes,users,ticketstatuses`

**Pagination**: `page`, `pageSize` (default 20), `pageOffset`. Response `meta.page` has `hasMore`, `pages`, `records`.

### 2. Get ticket detail

```bash
curl -s "https://${TEAMWORK_DESK_DOMAIN}/desk/api/v2/tickets/${TICKET_ID}.json?includes=customers,inboxes,users,tags,ticketstatuses,messages" \
  -H "Authorization: Bearer ${TEAMWORK_DESK_API_KEY}" | jq .
```

### 3. Get ticket messages (threads)

Messages include replies, notes, and system events.

```bash
curl -s "https://${TEAMWORK_DESK_DOMAIN}/desk/api/v2/tickets/${TICKET_ID}/messages.json?orderBy=createdAt&orderMode=asc&includes=users,files" \
  -H "Authorization: Bearer ${TEAMWORK_DESK_API_KEY}" | jq .
```

**threadType values** (integer):

| ID | Name | Description |
|---|---|---|
| 1 | `message` | Customer/agent reply |
| 2 | `forward` | Forwarded message |
| 3 | `note` | Internal note (private) |
| 4 | `eventInfo` | System event |

Filter notes only:
```bash
curl -s "https://${TEAMWORK_DESK_DOMAIN}/desk/api/v2/tickets/${TICKET_ID}/messages.json?filter=$(python3 -c "import urllib.parse; print(urllib.parse.quote('{\"threadType\":3}'))")&orderBy=createdAt&orderMode=asc&includes=users" \
  -H "Authorization: Bearer ${TEAMWORK_DESK_API_KEY}" | jq .
```

### 4. Post a note on a ticket

A note is a message with `threadType: 3`.

```bash
curl -s -X POST "https://${TEAMWORK_DESK_DOMAIN}/desk/api/v2/tickets/${TICKET_ID}/messages.json" \
  -H "Authorization: Bearer ${TEAMWORK_DESK_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "message": "<p>Your note content here (HTML supported)</p>",
    "threadType": 3
  }' | jq .
```

**Optional fields**: `isDraft` (bool), `isPinned` (bool)

## Output guidelines

- When listing tickets: show a concise table with id, subject, status, inbox, created date, and customer name.
- When showing ticket detail: display subject, status, priority, assignee, customer, tags, and creation date.
- When showing messages: display each thread chronologically with sender, date, type (reply/note/event), and a trimmed body preview.
- Always inform the user before posting/modifying data (notes, replies).

## Reference

- [API docs](https://apidocs.teamwork.com/docs/desk)
- [Authentication](https://apidocs.teamwork.com/guides/desk/authentication)
- [Filtering](https://apidocs.teamwork.com/guides/desk/filtering-api-results)
- [Webhook payloads](https://apidocs.teamwork.com/guides/desk/teamwork-desk-webhook-payload-samples)
