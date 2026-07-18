<div align="center">

# 🎬 Flockery

</div>

> A community platform on top of Discord for content creators, built and operated end to end: the site, its database, its storage, its login, the bot, the creator panel, all of it. It started as a clip pipeline for a single YouTuber with ~250K subscribers and grew into a **multi-tenant SaaS: one codebase, N deployments**, live in production. Source kept private.

<div align="center">

[![Website](https://img.shields.io/badge/Website-flockery.app-BB9AF7?style=for-the-badge&labelColor=1A1B26)](https://flockery.app/)
[![Demo](https://img.shields.io/badge/Demo-demo.flockery.app-9ECE6A?style=for-the-badge&labelColor=1A1B26)](https://demo.flockery.app/)
[![Live deployment](https://img.shields.io/badge/Live%20deployment-clip%20portal-7AA2F7?style=for-the-badge&labelColor=1A1B26)](https://jimmycruck.com/clips)

</div>

<p align="center">
  <img src="docs/screenshot_portal.png" width="70%" alt="Clip portal" />
</p>

<p align="center">
  <img src="docs/screenshot_web.png" width="45%" alt="Web" />
    <img src="docs/screenshot_email.png" width="47%" alt="Creator email notification" />
  <img src="docs/screenshot_discord.png" width="45%" alt="Discord" />
</p>

## The problem

Some creators build entire videos out of clips their community posts in Discord. Getting those clips out used to be fully manual: scroll the server, open each message, download each video one by one, rename them, sort them by who sent what. Hours per video, before editing even starts. The content was already there. The workflow to collect it was the bottleneck.

I didn't want to hand anyone a download script. I built the whole thing for one creator first — the site from scratch, its backend, its storage, the login, and the Discord bot that feeds it — and then did the harder part: turned it into a product any creator can run, without forking a single line. The part I'm proud of isn't any single feature. It's that I own every layer end to end, and that the same layers now serve multiple communities from one codebase.

## What it does

Three sides that share one pipeline, plus a panel that runs it all.

### For the creator: curation without the busywork

They do one thing: **react to a clip in their Discord.** That's the entire workflow. Two reactions, two outcomes:

- **Approve**: the clip goes live on the site *and* gets queued to pull down to disk for the next video.
- **Web only**: the clip goes live on the site for the community, but stays out of the editing batch.

The moment they react, the clip is captured and it's on the web, instantly, no manual downloading. The bot extracts a poster frame with ffmpeg at capture time, so the clip lands on the site with a real thumbnail, and it reacts back so the creator can see at a glance what's been saved. They can change their mind at any time: the last emoji always wins. Which emojis do what, whether the bot confirms, and who counts as an approver are all **per-deployment configuration, editable from the panel** — not code.

Then, when they're ready to edit, **one command pulls every newly-approved clip to a local folder, sorted by creator, filenames intact, ready to drop straight into the timeline.** Only the new ones, never re-downloading, and it never touches the public gallery.

**And there's a creator panel** (React) where they run the rest without touching code: branding and theme (colors, fonts, wordmark, copy), which channels get a tab on the wall, module toggles, stats dashboards, clip moderation with reversible soft-delete, transactional email templates, and audience newsletters with granular unsubscribe. Every save is validated server-side against an allowlist, written with compare-and-set (concurrent edits get a clean 409, never a silent overwrite), and followed by a targeted edge-cache purge — a change is visible on the public site in seconds.

### For the community: a reason to come back

The clips don't just disappear into an editor. They live on a **portal where fans log in with their Discord account** and browse everything that made the cut, each clip shown with the creator's real Discord avatar and name, grouped by channel. They can **like** clips, drop **emoji reactions** from a fixed palette (Discord style: tap to add, tap again to remove, validated server-side so with no free text there's nothing to moderate), see **view counts**, and pull up **their own submissions** in one place.

**A reel that opens different every time.** The vertical feed's default mode takes the trending ranking and shuffles it in position space with a per-session seed: the good stuff stays near the top, but the opener rotates instead of always being the same number one. The browser also remembers which clips you've already watched (locally, anonymously, 30-day expiry) and the feed sinks those below everything you haven't seen. Trending itself is a Hacker News style score, likes and views with time decay, with fresh and top-of-the-week tabs alongside it. Pagination is keyset with opaque cursors, so clips landing mid-scroll never cause duplicates or skips, and only the active clip and its neighbors ever hold a loaded video.

**Participation modules, switchable per client.** Beyond the feed, communities get competitive mechanics — each one a module the platform can turn on per deployment:

- **👑 Clip of the Week**: a public race by likes since the last coronation; the system ranks the candidates, a human puts the crown from the panel. The winner gets a hero card, a gold badge, a permanent crown counter and a congratulation email. The reigning clip sits out the next race; crowning stamps the next window automatically — no cron jobs.
- **🏆 Contests**: time-boxed events over a channel where fans rate clips 1–10 (one vote per fan per clip, structurally — it's the primary key), with a derived podium, frozen scores at close, and winner emails.
- **🎯 Bounties**: challenges with a prize ("best clip of X wins Y") where attempts are derived from the same capture pipeline and the creator picks the winner from the panel.

**Links that unfurl.** Every clip has its own share page, server-rendered with per-clip Open Graph meta: title, description, poster thumbnail and og:video, so a link dropped in Discord or WhatsApp shows a real preview instead of a gray box. For a person, the same URL opens the reel pinned to that exact clip. Any clip can be downloaded with its original filename through a proxy that only ever serves objects from the deployment's own bucket.

**Honest numbers for creators.** An anonymous analytics pipeline (no cookies, no PII, a random per-tab session id, batched through sendBeacon) tracks impressions, plays, watch depth by quartile, completions, loops and shares. Downloads are counted server-side, so nobody can inflate them from the client. Logged-in fans get a profile with their history; the creator gets aggregate dashboards in the panel.

And the loop closes automatically: a fan who saved their email gets a message the first time the creator picks their clip, another when they make a podium, another when they win the crown — all in the creator's voice (the copy is a template they can edit from the panel), without the creator lifting a finger. The portal is **installable as a PWA**, and the reaction that saves a clip for editing is the same signal that surfaces it to everyone, so the workflow that makes the videos doubles as a retention engine.

### For sponsors: native slots in the same feed

The reel is a TikTok-style vertical feed, and brands can buy space inside it. A sponsored clip enters through the exact same pipeline: post the brand's video in Discord with the campaign data in the message (name, destination link, rotation weight), react with the sponsor emoji, and it starts rotating through the feed at a configurable cadence.

- **Weighted rotation.** A brand that pays more gets a higher weight and proportionally more slots. Editing the message and re-reacting updates the campaign live, no redeploy.
- **Transparent by design.** Every sponsored clip carries a visible "Sponsored" badge and a call-to-action link. It never poses as a community member, never enters the Clip of the Week race, and can't be liked, shared or downloaded.
- **Campaign analytics.** Impressions, plays, watch depth, completions and CTA clicks are tracked per campaign, so a brand gets a real report with CTR included.

## The multi-tenant retrofit

The technically interesting part is what happened when a one-off became a product — without forking, and without downtime for the community already using it:

- **One configuration table per deployment** is the heart of the model: branding, copy, theme, channels, module state, bot behavior. The public site's HTML is intercepted at the edge by a middleware that renders **neutral templates from the repo** filled from that table — the failure mode is neutrality, never another client's brand — and zero client identity ships in the browser JS or the repo.
- **A module system with two owners**: platform flags (what tier a client bought) × creator tunables (how the module behaves), composed into a single signal server-side. Turning a module off turns its surfaces off but keeps its data intact.
- **The bot reads the same table** (60-second TTL, best-effort: the database being down never stops capture), so approval emojis, confirmations and the approver list propagate to a resident process without redeploys.
- **Secrets never live in that table** — credentials stay in per-service env vars with minimal scopes; the config API serves a public view through an allowlist.
- **Onboarding a new community is configuration, not code**: a bootstrap schema, a seed, and a set of managed accounts. Deployment #2 went live from scratch on the same commit as deployment #1, verified end to end the same day.

## Built with

Nearly everything runs on free tiers at this scale; the bot's always-on host is the only paid line item per deployment.

- **The web platform**: built from scratch, deployed from a GitHub repo through **Cloudflare Pages**, with the API layer running as serverless **Pages Functions** at the edge. Anonymous traffic to the hot endpoints is edge-cached, so a viral share wave never touches the database.
- **The creator panel**: **React + Vite**, same repo, same Pages project. Passwordless auth with **magic links**, CSRF-hardened writes, and a save flow that polls the edge cache until the change is confirmed live.
- **Neon (Postgres)**: one project per deployment, the single source of truth for clips, likes, ratings, analytics events, crown history, fan profiles and configuration.
- **Cloudflare R2**: object storage that serves the video and the poster frames, one bucket per deployment behind its own domain. Zero egress fees, which is the whole reason a video product like this runs cheaply.
- **Discord OAuth2**: fans log in with the identity they already use in the community. CSRF-state protection, signed sessions, minimal `identify` scope, no passwords to store.
- **Resend**: transactional and broadcast email, with per-deployment sending domains.
- **ffmpeg**: poster frame extraction at capture time, with an idempotent backfill script for the back catalogue.
- **The bot**: Node.js + discord.js, running 24/7 as an always-on worker on **Railway**, one service per deployment from the same repo. It streams uploads straight through, so a 5 MB clip and a 500 MB clip are handled the same way.

## Why there's no source here (and no wiring diagram)

Two reasons, and I'd rather be upfront about both.

One: this is a commercial product running live communities, holding real user data and credentials. That code doesn't belong in public.

Two: I'm deliberately not spelling out how the pieces connect. Not because it's a secret nobody could reverse-engineer, but because I know this pipeline end to end and standing it up for a new community is the product itself. You can see it running for real: the [website](https://flockery.app/), the [public demo](https://demo.flockery.app/), and a [live deployment](https://jimmycruck.com/clips) serving a ~250K-subscriber community. Reach me at the bottom.

## What I'd still do differently

- Aggregate analytics into daily rollups instead of scanning the raw event log (thresholds are written down; the data hasn't earned it yet).
- Self-service asset uploads for the creator panel, so theme images stop being an operator step.
- Automate more of the onboarding playbook — today it's a documented manual runbook, deliberately, until the third deployment tells me which steps are worth the tooling.

## AI assistance

Parts of this project and this write-up were built with AI assistance, disclosed here as in all my repos.

---

<div align="center">

<em>If you run a community and this is the kind of thing you wish you had, that's <a href="https://flockery.app/">flockery.app</a>. If you want to talk shop about how it's built, reach out.</em>

<br/><br/>

<a href="mailto:absravdev@gmail.com">📧 absravdev@gmail.com</a> · <a href="https://abstractraven.com/">🌐 abstractraven.com</a> · <sub>🐦‍⬛ <a href="https://github.com/absravdev">AbstractRaven</a></sub>

</div>
