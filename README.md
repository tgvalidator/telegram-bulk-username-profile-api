# Telegram Bulk Username Profile Check — `tg_username_profile_batch` | TG Validator

Telegram bulk username profile check returns user id, last-seen time, active days, the Premium flag and the avatar URL on top of confirming the account exists. Input is usernames, not numbers; no country to choose, one username per line.

This is the official TG Validator example repository for **one** bulk product, `tg_username_profile_batch`. It is an asynchronous task: upload a file, get a task id immediately, poll the task, and download the result file when it finishes.

- **Product page:** https://tgvalidator.com/products/tg_username_profile_batch
- **API documentation:** https://tgvalidator.com/api-docs
- **API base URL:** `https://tgvalidator.com`
- **Authentication:** `X-API-Key`
- **Get an API key:** https://tgvalidator.com/register

## What is it usually used for?

- Tiering username lists by activity
- Spotting long-silent accounts
- Splitting Premium from non-Premium
- Filling in user ids and avatars

## What does the result file contain?

| Column | Example | Meaning |
|---|---|---|
| `username` | `alex_kim` | The submitted username (without @). |
| `activated` | `yes` | Whether the username belongs to an existing Telegram account. |
| `uid` | `1234567890` | Telegram user id. |
| `lastseen` | `2026-04-26 17:01:17` | The most recent online time observed. |
| `activedays` | `130` | Days since that activity — smaller is more recent. |
| `member` | `no` | Whether the account has Telegram Premium. |
| `avatar_url` | `<image url>` | Avatar URL, empty when the account has none. |

The result is a **point-in-time signal**, not a verdict, and not identity data. It describes what the provider reported at the moment the task ran.

## How do I submit a task?

Upload a `.txt` or `.csv` with **one Telegram username per line (the leading `@` is optional)**, 1,000–100,000 valid entries. There is **no country** on username tasks: do not send the `country` field.

```bash
curl -X POST 'https://tgvalidator.com/api/v1/bulk-tasks' \
  -H 'X-API-Key: YOUR_API_KEY' \
  -F 'product=tg_username_profile_batch' \
  -F 'file=@usernames.txt'
```

The response returns the task id and `status=processing`, along with the server-side quote taken before processing starts.

## How do I poll a task and download the result?

```bash
curl -fsS 'https://tgvalidator.com/api/v1/bulk-tasks/TASK_ID' -H 'X-API-Key: YOUR_API_KEY'
```

`status` is `processing`, `success` or `failed` — there is no progress percentage to poll for. **Do not poll more often than once every 30 seconds.** When the task succeeds the response carries the result-file download link.

## Bulk task or realtime check?

`POST /api/v1/bulk-tasks` (this repository) takes a file of 1,000–100,000 entries and answers later — that is the shape for list cleaning, campaign preparation and enrichment runs. A **realtime** check (`POST /api/v1/check`, or `POST /api/v1/batch-check` for up to 100 identifiers) answers inside the same HTTP response, for a signup form or a live lookup. The two are separate endpoints and are not interchangeable; see the other repositories under [tgvalidator](https://github.com/tgvalidator).

## What about billing?

The whole file is reserved on submit. When the task finishes you are charged only for the entries that were actually checked and the remainder is returned. A failed task is refunded in full.

## What are the limits and error codes?

| Limit | Value |
|---|---|
| Entries per task | 1,000–100,000 |
| File formats | `.txt`, `.csv`, one entry per line |
| Products per task | 1 (`tg_username_profile_batch`) |
| Country | not applicable — omit the field |
| Poll interval | no more than once per 30 seconds |

`40000`/`40001` are request or file errors; `40100` is an invalid key; `40200` is insufficient balance; `42900` is rate limited — honour `Retry-After`; `50300` is temporary maintenance. `GET /api/v1/balance` returns the current balance in `data.balance_micros`.

Keep the key on a trusted server and read it from `TGVALIDATOR_API_KEY`. Never commit it or expose it in production browser code.

## Runnable examples in seven languages

| Language | Example |
|---|---|
| Python | [`examples/python`](examples/python) |
| Node.js | [`examples/nodejs`](examples/nodejs) |
| Go | [`examples/go`](examples/go) |
| Java | [`examples/java`](examples/java) |
| C# | [`examples/csharp`](examples/csharp) |
| PHP | [`examples/php`](examples/php) |
| Shell / curl | [`examples/shell`](examples/shell) |

## Official resources and responsible use

- **This product:** https://tgvalidator.com/products/tg_username_profile_batch
- **All products:** https://tgvalidator.com/products
- **Documentation:** https://tgvalidator.com/api-docs
- **Registration:** https://tgvalidator.com/register
- **Pricing:** https://tgvalidator.com/pricing
- **OpenAPI contract:** [`openapi.yaml`](openapi.yaml)
- **License:** [MIT](LICENSE) for the sample code

Submit only data you are authorized to process, and comply with applicable privacy laws and platform terms. Report the signal you received as a signal; do not relabel it downstream as a conclusion it does not support.

---

*Last reviewed: 2026-09-22 · Maintained by TG Validator. Canonical documentation: https://tgvalidator.com/api-docs*
