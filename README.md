<p align="center">
  <img src="docs/hero-banner.png" alt="council-watch — manga-style civic chamber seen from the doorway. The tablet HUD is illustration only, not live agenda or voting data." width="100%">
</p>

# council-watch

A public GitHub Actions backstop that knocks on a Council `/healthz` door every five minutes. If the door stays shut for three strikes, it rings Telegram.

Independent civic-studio infrastructure by [Dr Non Arkara](https://github.com/Nonarkara). **Not a government product.** Not affiliated with or endorsed by any city council, legislature, or public agency.

The banner is a manga illustration of civic watching — a chamber, a gallery, a tablet held at the door. **The HUD (agenda list, voting-trail dots, sparkline) is illustration only.** This repository does not scrape agendas, record votes, or publish proceedings.

---

## What this is

council-watch is a small, inspectable watchdog for a private Council health endpoint (called **M3** in this repo). It is not an application server, not a dashboard, and not a municipal product.

What ships here:

| Path | Role |
| --- | --- |
| [`.github/workflows/watch.yml`](.github/workflows/watch.yml) | Ping, strike counter, optional Telegram alert |
| [`.state/fail-count`](.state/fail-count) | Consecutive non-200 count (written by the workflow) |
| [`docs/hero-banner.png`](docs/hero-banner.png) | README illustration |
| [`LICENSE`](LICENSE) | MIT |

There is **no public live URL** in this repository. The host to ping is supplied at runtime through GitHub Actions secrets. `.state/fail-count` is operational state, not source — the workflow commits it so strike counts survive between runs.

This is the public half of a civic studio habit: keep the monitor readable, keep the monitored host unpublished.

---

## Philosophy

A system that matters should be watched from outside itself.

The Council this repo pings is a private service, not a legislature. The civic move is still the same: put the watch on a clock that does not live on the same machine, and leave the rules in a public file. Anyone can read how the ping works, how strikes accumulate, and when a human is paged. That is the transparency — the monitor is inspectable. The endpoint stays a secret so a fork cannot knock on someone else's door by accident.

Three consecutive failures is about fifteen minutes of silence. The point is not a status page. The point is: if you are away and the council is dark, someone still hears it.

The hero image is the metaphor, not the product. A person at the chamber door, holding a light. The work in this repo is the knock.

---

## Ethical use

This work sits in a **civic-transparency** practice: make the watchdog public, do not impersonate the institution.

**Do**

- Fork this as an outside health backstop for a `/healthz` service you operate.
- Keep the workflow readable. Keep secret values out of git.
- Say clearly that your fork is independent personal or studio infrastructure.

**Do not**

- Present this repository, the banner, or a fork as an **official council product** — not a city council, not a government, not a public-agency system.
- Treat the illustrated HUD as live agenda, voting, budget, or public-safety data. It is drawing, not evidence.
- Use the name or imagery to impersonate a clerk, legislature, or municipal communications channel.
- Publish someone else's health URL, bot token, or chat destination — including values you find in Actions logs or in a private fork.

council-watch does not speak for any council. It only asks *are you there?* and, after three unanswered knocks, tells a human.

---

## How it works

The workflow in [`.github/workflows/watch.yml`](.github/workflows/watch.yml) runs on a five-minute cron (`*/5 * * * *`) and on manual `workflow_dispatch`.

1. Issues `GET {COUNCIL_HEALTH_URL}/healthz` with a 10-second timeout.
2. Treats HTTP **200** as healthy. Any other status — including a failed request, which `curl` reports as `000` — is a strike.
3. Writes the consecutive-failure count to [`.state/fail-count`](.state/fail-count) and commits that file so the next run can resume the count.
4. When the count reaches **3** (about 15 minutes down), posts a Telegram message using a bot token stored as a GitHub secret. A later 200 resets the counter to `0`.

The alert text in the workflow refers to an M3 sentinel and an M5 host. Those names are operational labels in this repo; they are not documented further here.

```
schedule / dispatch
        │
        ▼
 GET {secret}/healthz  ──►  200?  ──yes──►  fail-count = 0  ──►  commit
        │
       no
        ▼
 fail-count += 1  ──►  commit  ──►  count == 3?  ──yes──►  Telegram
```

---

## How to run / fork

This repository has no application server. It runs on GitHub Actions.

### Prerequisites

- A GitHub repository with Actions enabled and permission for the workflow to push to the default branch (`contents: write` is already set in the workflow).
- An HTTP service that answers `GET /healthz`.
- A Telegram bot token, if you want the three-strike alert.

### Fork

1. Fork [Nonarkara/council-watch](https://github.com/Nonarkara/council-watch).
2. In the workflow, change the Telegram destination chat before you enable alerts — it is hardcoded for this studio's watch, not for yours.
3. Set **your** secrets (below). Do not copy values from this account, from Actions logs, or from any other fork.
4. Push to the default branch, or use **Actions → council-watch → Run workflow**.

### Repository secrets

Set these under **Settings → Secrets and variables → Actions**. Do not commit values. This README does not invent, guess, or reprint secret contents.

| Secret | Purpose |
| --- | --- |
| `COUNCIL_HEALTH_URL` | Base URL of the service to monitor. The workflow appends `/healthz`. |
| `TG_TOKEN` | Telegram Bot API token. Used only when the failure count reaches 3. |

### Enable and trigger

1. Push the workflow to the default branch.
2. Open **Actions → council-watch → Run workflow** (`workflow_dispatch`), or wait for the schedule (`*/5 * * * *`).
3. Confirm a run appears. If the endpoint is not returning 200, `.state/fail-count` should increment and a commit from `council-watch` should land on the default branch.

GitHub may delay scheduled workflows. Use **Run workflow** when you need an immediate check.

---

## License

MIT. See [LICENSE](LICENSE). Copyright (c) 2026 Dr Non Arkara.

Fork the watch. Keep the ethic. Do not pretend you are the chamber.
