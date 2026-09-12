# kurdify-uptime

Off-machine reachability check for `app.kurdify.app`, run on GitHub's infrastructure.

## Why it is not inside the kurdify repo

Kurdify's own health probe runs on the Mac that serves the site. **143 of its 249 recorded
failures were the host itself unreachable**, which an on-host probe structurally cannot report:
the machine that is down cannot tell you it is down. This one runs somewhere else.

It lives in its own **public** repo because `Siavashsed/kurdify` is private, and private-repo
Actions bill per run rounded up to the minute. A 5-minute cron is 8,640 billed minutes a month
against a 2,000-3,000 allowance, roughly $53/month. Public repos get unlimited free minutes.
Nothing here is secret: `app.kurdify.app` is already public DNS, and this repo holds no source.

## What it checks

Two endpoints, every 5 minutes, because they fail independently:

| probe | proves |
|---|---|
| `https://app.kurdify.app/api/features` | the app is answering |
| `https://app.kurdify.app/` | the page a visitor lands on still renders |

Each probe retries once, 20 seconds later, before the run is called an outage. The on-host
script waits for two consecutive 5-minute ticks for the same reason, which costs it 5 minutes
of detection latency; retrying inside one run buys the same protection for 20 seconds.

## How you find out

1. **GitHub's own email.** GitHub emails the user who last edited the cron when a scheduled
   workflow fails. This needs no configuration and is the baseline channel.
2. **Telegram**, additive. Set repo secrets `TG_BOT_TOKEN` and `TG_ALERT_CHAT`
   (Settings, Secrets and variables, Actions). Until both exist the step logs one line and
   stays green, so a missing alert channel never becomes a second failure on top of the first.

## Two honest limits

- **GitHub cron is best-effort.** Scheduled runs are delayed under load, sometimes 10-15
  minutes. Read this as "checked several times an hour", never as a precise interval, and
  never as the source of an uptime percentage.
- **GitHub disables scheduled workflows after 60 days of repo inactivity,** and a workflow run
  does not count as activity. The `keepalive` job commits a heartbeat once a week so the
  monitor does not die silently after two months, which is the exact failure it exists to catch.

---
Copyright (c) Kavalsia Inc. All Rights Reserved.
Lead Architect & Engineer: Siavash Sadighi
