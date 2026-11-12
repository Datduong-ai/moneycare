# MoneyCare — API Reference (Mobile Shared Backend)

This web app is prepared to serve a mobile app. All endpoints are documented in `openapi.json` (OpenAPI 3.0.3). This file is a quick human-readable summary.

## Authentication

Mobile clients authenticate via the system-managed mobile-auth bridge:

1. `POST /api/mobile-auth/login` with `{ "email", "password" }` → `{ token, user }`.
2. Send the token on every subsequent request as `Authorization: Bearer <token>`.
3. The bridge (`proxy.ts` + `lib/mobile-bridge.ts`) converts the bearer token into this app's session cookie, so every existing route works unchanged.

`POST /api/mobile-auth/signup` and `GET /api/mobile-auth/me` are also available. These three mobile-auth routes and NextAuth's `/api/auth/[...nextauth]` are managed by the platform and are intentionally **not** listed in `openapi.json`.

Monetary amounts are integers in **VND**. BigInt values serialize to JSON numbers; dates to ISO 8601 strings.

## Endpoints

| Method | Path | operationId | Auth | Description |
|--------|------|-------------|------|-------------|
| POST | /api/auth/login | authLogin | No | Verify email/password, return basic user (no session) |
| POST | /api/signup | signup | No | Register a user + default wallet & categories |
| GET | /api/dashboard | getDashboard | Yes | KPIs, 6-month trend, category breakdown, recent transactions |
| GET | /api/reports | getReports | Yes | Monthly report with previous-period comparison (`month`, `year`) |
| GET | /api/wallets | listWallets | Yes | List wallets |
| POST | /api/wallets | createWallet | Yes | Create a wallet |
| GET | /api/categories | listCategories | Yes | List categories |
| POST | /api/categories | createCategory | Yes | Create a category |
| GET | /api/budgets | listBudgets | Yes | List budgets with spent amounts (`month`, `year`) |
| POST | /api/budgets | upsertBudget | Yes | Create/update a budget for category/month/year |
| GET | /api/transactions | listTransactions | Yes | Paginated transactions + filters |
| POST | /api/transactions | createTransaction | Yes | Create a transaction (updates wallet balance) |
| GET | /api/transactions/{id} | getTransaction | Yes | Get one transaction |
| PUT | /api/transactions/{id} | updateTransaction | Yes | Update a transaction (re-applies balance effect) |
| DELETE | /api/transactions/{id} | deleteTransaction | Yes | Delete a transaction (reverses balance effect) |
| GET | /api/goals | listGoals | Yes | List savings goals with contributions |
| POST | /api/goals | createGoal | Yes | Create a savings goal |
| POST | /api/goals/{id}/contribute | contributeToGoal | Yes | Add a contribution to a goal |
| GET | /api/members | listMembers | Yes | List household members |
| POST | /api/members | createMember | Yes | Create a household member |
| PUT | /api/members/{id} | updateMember | Yes | Update a household member |
| DELETE | /api/members/{id} | deleteMember | Yes | Delete a household member |
| POST | /api/upload/presigned | createPresignedUpload | Yes | Presigned cloud-storage upload URL |
| POST | /api/ai/categorize | aiCategorize | Yes | Suggest a category for a description (JSON) |
| POST | /api/ai/chat | aiChat | Yes | AI finance assistant — **streams** `text/plain` |
| GET | /api/ai/insights | aiInsights | Yes | AI insights — **streams** `text/plain` |
| POST | /api/ai/scan-receipt | aiScanReceipt | Yes | Scan a receipt image (multipart) — **SSE** `text/event-stream` |

### Query filters for `GET /api/transactions`
`page`, `limit`, `categoryId`, `walletId`, `type` (`income`/`expense`), `memberId`, `search` (matches description), `from`, `to` (dates).

### Streaming endpoints
`aiChat` and `aiInsights` return a `text/plain` chunked stream. `aiScanReceipt` returns Server-Sent Events; each event is JSON with `status` (`processing` | `completed` | `error`) and, when completed, a `result` object (`store`, `date`, `amount`, `categoryId`, `type`, `description`). Mobile clients must read these as streams, not as a single JSON body.

## Notes
- All data routes require a valid bearer token; without it they return `401 { "error": ... }`.
- Data is always scoped to the authenticated user; IDs in the URL never grant cross-user access.
- The full machine-readable contract (request bodies, response schemas, status codes) lives in `openapi.json`.
