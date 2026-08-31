# council-watch

GitHub Actions health backstop for a Council `/healthz` endpoint (called **M3** in this repo). Every five minutes it records whether the endpoint returned HTTP 200; after three consecutive failures it sends a Telegram alert.

This is independent personal infrastructure by [Dr Non Arkara](https://github.com/Nonarkara). It is not a government product and is not affiliated with or endorsed by any public agency.

## What it does

The workflow in [`.github/workflows/watch.yml`](.github/workflows/watch.yml):

1. Issues `GET {COUNCIL_HEALTH_URL}/healthz` with a 10-second timeout.
2. Treats HTTP **200** as healthy. Any other status — including a failed request, which `curl` reports as `000` — is a strike.
3. Writes the consecutive-failure count to [`.state/fail-count`](.state/fail-count) and commits that file so the next run can resume the count.
4. When the count reaches **3** (about 15 minutes down), posts a Telegram message using a bot token stored as a GitHub secret. A later 200 resets the counter to `0`.

The alert text in the workflow refers to an M3 sentinel and an M5 host. Those names are operational labels in this repo; they are not documented further here.

There is **no public live URL** in this repository. The host to ping is supplied at runtime through GitHub Actions secrets.

## How to run

This repository has no application server. It runs on GitHub Actions.

### Prerequisites

- A GitHub repository with Actions enabled and permission for the workflow to push to the default branch (`contents: write` is already set in the workflow).
- An HTTP service that answers `GET /healthz`.
- A Telegram bot token, if you want the three-strike alert.

### Repository secrets

Set these under **Settings → Secrets and variables → Actions**. Do not commit values.

| Secret | Purpose |
| --- | --- |
| `COUNCIL_HEALTH_URL` | Base URL of the service to monitor. The workflow appends `/healthz`. |
| `TG_TOKEN` | Telegram Bot API token. Used only when the failure count reaches 3. |

The Telegram destination chat is hardcoded in `.github/workflows/watch.yml`. Change it before enabling alerts on a fork.

### Enable and trigger

1. Push the workflow to the default branch.
2. Open **Actions → council-watch → Run workflow** (`workflow_dispatch`), or wait for the schedule (`*/5 * * * *`).
3. Confirm a run appears. If the endpoint is not returning 200, `.state/fail-count` should increment and a commit from `council-watch` should land on the default branch.

GitHub may delay scheduled workflows. Use **Run workflow** when you need an immediate check.

## Layout

```
.github/workflows/watch.yml   # ping, failure counter, optional Telegram alert
.state/fail-count             # consecutive non-200 count (written by the workflow)
LICENSE
```

`.state/fail-count` is operational state, not source. The workflow commits it so strike counts survive between runs.

## License

MIT. See [LICENSE](LICENSE).
