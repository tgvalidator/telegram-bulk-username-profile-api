# Telegram bulk username profile check — official API examples | TG Validator

Return available Telegram user ID, activity-day and avatar URL signals for an authorized list of usernames.

This repository covers the **asynchronous, file-based** product `tg_username_profile_batch`. Submit a file, keep the returned task ID, poll it, then use the result URL after success. It is separate from synchronous `/api/v1/check` and `/api/v1/batch-check` integrations.

- **Product:** https://tgvalidator.com/products/tg_username_profile_batch
- **API documentation:** https://tgvalidator.com/api-docs
- **API base URL:** `https://tgvalidator.com`
- **Authentication:** `X-API-Key`
- **Get an API key:** https://tgvalidator.com/register?utm_source=github&utm_medium=referral&utm_campaign=tgvalidator_telegram-bulk-username-profile-api

## Typical uses
- Tiering username lists by activity
- Spotting long-silent accounts
- Splitting Premium from non-Premium
- Filling in user ids and avatars

## Submit an authorized list

Upload a `.txt` or one-column `.csv` file with **1,000–500,000 valid username entries**. Use one username per line. This product does not require a `country` field.

Send `service_type` and, for phone products, `country` **before `file`** in the multipart body. The request field is `service_type`; the response still calls the selected product `data.product`.

```bash
curl -X POST 'https://tgvalidator.com/api/v1/bulk-tasks' \
  -H 'X-API-Key: YOUR_API_KEY' \
  -F 'service_type=tg_username_profile_batch' \
  -F 'file=@usernames.txt'
```

For a retry of the same submission, use the same previously chosen `Idempotency-Key` header (up to 128 characters). Do not automatically resubmit after a timeout: the original request may already have created a task.

The included `usernames.txt` is a tiny **format fixture**, not a production-sized list. Supply an authorized file that meets the product minimum; do not pad it with duplicates or invented identifiers. Example clients accept `FILE` to select your file.

## Poll the same task and obtain results

```bash
curl -fsS 'https://tgvalidator.com/api/v1/bulk-tasks/TASK_ID' \
  -H 'X-API-Key: YOUR_API_KEY'
```

Read the `code`, then `data.status`. Poll **no more often than once every 30 seconds**:

- `processing`: keep the task ID and wait. `preparing=true` means input preprocessing is not complete; `total` is preliminary.
- `success`: final `success_cnt` and `failure_cnt` are available. Use `result_url` to download the result. If the link is temporarily absent, query this task again after 30 seconds; do not submit a replacement task.
- `failed`: report `reason` / `reason_detail` and stop polling. An HTTP 200 containing `data.status=failed` is a failed task.

The API does not return a progress percentage or pricing/reservation quote. Request fields, response fields, and examples are specified in [openapi.yaml](openapi.yaml).

## Exported result fields

The current brand-side export uses these columns, in this order. Preserve their names when parsing the result; do not substitute upstream CSV headers.

| Column | Example | Meaning |
|---|---|---|
| `identifier` | `alex_kim` | Submitted identifier, used to join the result to the input. Phone identifiers include their country code. |
| `activated` | `true` | The determined positive/negative signal for this product; interpret it in the product context. |
| `uid` | `1234567890` | Platform user ID when available. |
| `activedays` | `9` | Activity-day signal when available. Telegram privacy may coarsen it to 0, 7, 30 or 1000; it is not an exact login timestamp. |
| `avatar_url` | `https://cdn5.telesco.pe/file/example.jpg` | Public avatar URL when available; an empty value does not reveal a private image. |

These are point-in-time product signals. Empty or unknown values are not proof of a negative fact; avatar attributes are image-derived estimates, not verified facts about a person.

## Limits, billing and errors

| Contract | Value |
|---|---|
| `service_type` | `tg_username_profile_batch` |
| Input | `username` |
| Valid entries per task | 1,000–500,000 |
| Country | Not required |
| Poll interval | At least 30 seconds |
| File formats | `.txt`, one-column `.csv` |

The input amount is reserved on submission. Settlement charges checked entries and returns the remainder; failed tasks are refunded. See [current pricing](https://tgvalidator.com/pricing) rather than assuming a fixed price. `GET /api/v1/balance` returns USD micros in `data.balance_micros`.

`40000`/`40001` cover invalid requests or inputs; `40100` is an invalid key; `40200` is insufficient balance; `42900`/`42901` are rate/concurrency limits; `50300` is temporary maintenance. Honor `Retry-After`. Do not treat a request error or undetermined item as a determined negative result.

## Language examples

Run from the repository root; use the commands and prerequisites in each language README.

| Language | Example | Coverage |
|---|---|---|
| Python | [examples/python](examples/python) | Submit and poll; print result metadata / URL |
| Node.js | [examples/nodejs](examples/nodejs) | Submit and poll; print result metadata / URL |
| Shell | [examples/shell](examples/shell) | Submit and poll; print result metadata / URL |
| Go | [examples/go](examples/go) | Submit; prints the task ID in its JSON response |
| PHP | [examples/php](examples/php) | Submit; prints the task ID in its JSON response |
| Java | [examples/java](examples/java) | Query one existing task ID |
| C# | [examples/csharp](examples/csharp) | Query one existing task ID |

Read the key from `TGVALIDATOR_API_KEY` on a trusted server. `API_BASE_URL` is available for a trusted development/mock endpoint; never send a production key to an untrusted override. Do not expose keys in browser code or commit them.

## Official resources

- Product: https://tgvalidator.com/products/tg_username_profile_batch
- Documentation: https://tgvalidator.com/api-docs
- Registration: https://tgvalidator.com/register?utm_source=github&utm_medium=referral&utm_campaign=tgvalidator_telegram-bulk-username-profile-api
- Pricing: https://tgvalidator.com/pricing
- OpenAPI: [openapi.yaml](openapi.yaml)
- Sample code license: [MIT](LICENSE)

Process only identifiers you are authorized to use. Third-party trademarks do not imply affiliation or endorsement.

*Last reviewed: 2026-10-30 · Maintained by TG Validator.*
