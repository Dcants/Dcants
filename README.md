# Demetrius McCants

Computer scientist and ML engineer. I build systems that have to keep up with something real: live RF streams off a software-defined radio, phone calls that must be answered and booked in a second or two, sports data feeding a calibrated prediction engine, and the web products in front of them. Robust products that hold up under real load, real users, and real deadlines — measured, not assumed.

- **Now:** AI Solutions & Cloud Architect at [DarkCom Enterprises](https://www.darkcom.co) — AI voice agents, workflow automation, and the cloud underneath them.
- **Summer 2026:** RF Machine Learning Engineer intern at NIWC Pacific (U.S. Navy) — real-time drone detection, from live RF capture through CNN inference back to the operator.
- **Summer 2025:** Cyber & RF Research intern, VICEROY MAVEN (AFRL).
- **School:** B.S. Computer Science, University of West Georgia (expected 2028).

The projects below are the work itself. Live links where there's something to click; private repos are marked — happy to walk through any of them.

## Tech & Tools

- **Signals & ML:** `Python` · `NumPy` · `PyTorch / TorchSig` · `SDR / VITA-49` · `GNU Radio` · `Wireshark` · `TCP/IP`
- **Backend & Data:** `FastAPI` · `PostgreSQL` · `TimescaleDB` · `SQLAlchemy` · `Celery` · `Redis` · `Docker`
- **Cloud & Automation:** `GCP (Cloud Run, Cloud SQL)` · `n8n` · `Twilio` · `ElevenLabs` · `Vercel`
- **Web:** `Next.js` · `React` · `TypeScript` · `Tailwind`
- **Also:** `Java` · `C` · `C#` · `Luau (Roblox)`

## Featured Work

### RF & Signal Processing

#### Sceptre SDR Ingest Pipeline
Live UDP ingestion for the Sceptre SDR platform: decodes its VITA-49.2 packet stream into `complex64` NumPy windows paired with typed RF context (center frequency, sample rate, bandwidth), ready for FFT, recording, and RF-ML consumers downstream.

- **Repo:** https://github.com/Dcants/sceptre-pipeline
- **Stack:** Python · NumPy (stdlib + NumPy only at runtime) · Docker / Podman · PyInstaller
- **Highlights:**
  - Producer/consumer design with a bounded, drop-oldest queue; demuxes multiple streams on one port by `stream_id` and flushes windows on context change, packet-counter gap, size, or age.
  - Wire format developed and tested entirely offline against recorded captures — 190+ tests, including one that holds the shipped library to stdlib + NumPy.
  - Four deployment paths, including an air-gapped wheelhouse and a per-OS standalone binary — after proving Docker Desktop can't deliver external UDP to containers, containers are Linux-only with host networking.

#### Sceptre Pipeline — Free-Threaded Rebuild
The same pipeline re-architected for free-threaded CPython 3.14t (no GIL), so parsing actually spreads across cores.

- **Repo:** https://github.com/Dcants/sceptre-pipeline-free-thread
- **Stack:** Python 3.14t · NumPy · Docker
- **Highlights:**
  - Four stages: batched receive → N stateless parse workers → reorder ring → one ordered assembly thread → convert/emit. Packet order is preserved end to end, and output is tested byte-identical across 1, 2, 4, and 8 workers.
  - **620 MS/s** on a 192-core server vs **68.85 MS/s** with the GIL forced back on — same image, one variable changed, **7.64×**.
  - Tuned from measurements: throughput peaks at three parse workers and falls past that as threads spread across NUMA nodes, so the default is capped at three. Linux `recvmmsg` was built, benchmarked at 0.82× the portable path, and left off by default.

### AI Voice & Automation

#### DarkCom Voice Agent — *private repo, in active build*
A multi-tenant AI voice receptionist for salons: a caller reaches the salon's number and books an appointment in English, Vietnamese, or Spanish with an agent that checks real availability and writes the booking. One set of workflows and one agent template serve every tenant.

- **Stack:** Twilio · ElevenLabs Agents · n8n (self-hosted on Cloud Run) · Cloud SQL PostgreSQL 17 · Google Calendar API · Python
- **Highlights:**
  - Postgres functions own availability and double-booking; Google Calendar is the display mirror technicians see, provisioned keylessly through domain-wide delegation.
  - Availability, booking, and call-init webhooks are live. Booking stayed published only after holding every call under 1.5 s with three concurrent callers across six rounds on the real stack.
  - Workflow code is tested offline in n8n's own expression engine and mutation-tested before it touches the instance; the agent, its tools, and its test suite live in the repo as code.
  - Runs on a self-hosted n8n stack I stood up on Cloud Run + Cloud SQL behind a load balancer, with uptime alerting and park/restore runbooks.

#### Healthcare Voice Intake — *client work*
A production serverless voice AI system answering 24/7 for a home-healthcare agency, replacing manual after-hours reception with automated intake and multi-channel notifications.

- **Stack:** Twilio · ElevenLabs · n8n · Docker · Google Cloud Run
- **Highlights:** intent-based call triage orchestrated through self-hosted n8n, containerized on Cloud Run with elastic auto-scaling, in a data-isolated private cloud.

### Web

#### DarkCom Enterprises
Marketing and lead-generation site for a multi-division engineering company — AI agent infrastructure, workflow automation, voice agents, and full-stack engineering.

- **Live:** https://www.darkcom.co
- **Stack:** Next.js 16 · React 19 · TypeScript · Tailwind CSS v4 · Vercel (private repo)
- **Highlights:**
  - Fully static — every route prerenders. Nav, mega-menu, home grid, the four division pages, and form options all generate from one content model, so they can't drift apart.
  - Five lead and intake forms on one submission hook that never fakes success, plus an unlisted, tokenized client-onboarding form with saved drafts.
  - Privacy and Terms written to the SMS (A2P 10DLC) registration checklist; a canvas voice-waveform hero; pixel-hinted favicons from 16 to 48 px.

#### 180 Parking Solutions
A premium marketing site for an Atlanta-based managed-valet company — luxury-hospitality positioning, hospitality-grade typography, custom motion, and a real in-page lead-capture flow.

- **Live:** https://180-parking-site.vercel.app
- **Stack:** Next.js 14 · TypeScript · Tailwind · Framer Motion · Lenis · Vercel (private repo)
- **Highlights:** custom Framer Motion design language, accessible form modals (focus trap, validation, click-outside), and a deep iOS Safari paint-performance pass — stripped GPU-expensive CSS and framer-motion hydration on touch devices to bring multi-second mobile load times back in line with desktop.

### Data & Prediction

#### Bandwagon — *private repo*
A calibrated NBA player-points projection engine with a social picks product on top: users lock picks, follow sharp bettors, earn reputation and badges, and see where the model disagrees with the market. Not a sportsbook — no wagers are placed or held.

- **Stack:** Python 3.12 · FastAPI · SQLAlchemy 2 + Alembic · PostgreSQL + TimescaleDB · Celery + Redis · NumPy / SciPy / scikit-learn / Optuna · Docker
- **Highlights:**
  - Bottom-up projection (possessions → shots → efficiency) with a top-down cross-check, a vectorized Monte Carlo simulation for the full points distribution, and an edge layer (de-vig → EV → Kelly staking).
  - Idempotent grading — a stat correction re-settles to the right result without double-counting — validated by a walk-forward backtest. The automated tuning loop promotes a challenger only when it beats the champion out-of-sample on both calibration and closing-line value, with rollback.
  - Full social backend: Sign in with Apple + email, pick locking, consensus %, a Brier-based Sharp Score, leaderboards, feed, badges, moderation, APNs push, and RevenueCat entitlements — 900+ tests behind ruff, mypy, and CI with a secret scan. Backend complete; the native iOS client is designed and next.

### Games

#### Jolly Horizon — *private repo, in progress*
A Roblox pirate adventure: crew up, captain a ship across a shared 28-player sea, fight over cargo, and hunt the cursed fruits that grant powers — with no pay-to-win.

- **Stack:** Luau (strict) · Rojo · Wally · Rokit · TestEZ
- **Highlights:**
  - Filesystem-first: the whole game is version-controlled and builds reproducibly from a clone, with every tool pinned.
  - Server-authoritative from day one — every remote is defined in one place and passes a validation and rate-limiting middleware chain before any handler runs.
  - Design settled before code: seat-locked ships so moving-platform physics is designed out, and "cargo lives at the dock or dies at sea." Foundation phase complete, with its test suite green in Studio.

### Smaller Builds

- **EXSSA USA Silver Jubilee Countdown** — a flip-card countdown page for an organization's 25th-anniversary celebration. Vanilla HTML/CSS/JS on GitHub Pages. [Live](https://dcants.github.io/exssa-usa-silver-jubilee-countdown/) · [Repo](https://github.com/Dcants/exssa-usa-silver-jubilee-countdown)

<!-- Add new project case studies under the matching section. Same shape:
#### Project Name
One-sentence pitch — what it does and who it's for.

- **Live / Repo:** https://...   (mark private repos as *private repo* next to the title)
- **Stack:** key technologies · dot · separated
- **Highlights:** 1–3 bullets focused on craft and specifics, not adjectives.
-->

## Currently

Building out the DarkCom voice agent. Open to RF/ML, data-engineering, and contract work.

- **Email:** djmccants23@gmail.com
- **LinkedIn:** https://www.linkedin.com/in/demetrius-mccants-83247936a/
- **Company:** https://www.darkcom.co
