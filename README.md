# OpenClaw VPS: How much RAM it actually needs, what a year costs, and which SixtyNet plan clears the bar (plan table + hardening checklist)

A VPS that runs OpenClaw properly starts at **2 vCPU, 4 GB RAM and roughly 100 GB of NVMe storage**. Your bill has two lines only: the server, and your own model API usage. OpenClaw itself is free software, so there is no licence fee, no per-message charge and no third cost to budget for.

On SixtyNet's current VPS price list, the cheapest plan that clears the 4 GB memory floor is **LAX-MICRO at $400/year** (listed down from $480), with 4 vCPU, 4 GB RAM and 80 GB NVMe. The $200 LAX-STARTER sitting next to it has half that RAM, and 2 GB is where people get into trouble once a second channel and a couple of scheduled automations are live.

What follows: the full SixtyNet plan table, why the 2 GB tiers fall short, what closed ports 80/443 mean for a Gateway that should never face the internet anyway, and the startup settings that keep a small box from crawling.

> **Quick pick:** SixtyNet LAX-MICRO, 4 vCPU / 4 GB RAM / 80 GB NVMe, **$400 per year** billed annually, Los Angeles, DDoS protection included. Right size for one agent, a few messaging channels and a handful of automations.
> 👉 [See the LAX-MICRO plan and current annual price](https://sixtynet.net/aff.php?aff=39)

## SixtyNet's plans, sorted by whether they can run OpenClaw

SixtyNet sells three families: residential-IP VPS, a DDoS-protected LAX series, and a China-optimised network product. Only the LAX series lines up cleanly with what OpenClaw needs. Prices below are the annual figures shown on the VPS page, with the struck-through list price in brackets.

| Plan | Core specs | Price (billed annually) | Fit for OpenClaw | Order |
| --- | --- | --- | --- | --- |
| Residential-STARTER | 2 cores / 2 GB RAM / 40 GB NVMe, 5 TB traffic, residential IP, ports 80 & 443 closed | $260/yr (list $300) | Below the 4 GB floor | [Order Residential-STARTER](https://sixtynet.net/aff.php?aff=39) |
| LAX-STARTER | 2 cores / 2 GB RAM / 40 GB NVMe, 5 TB traffic, DDoS protection | $200/yr (list $240) | Below the 4 GB floor | [Order LAX-STARTER](https://sixtynet.net/aff.php?aff=39) |
| **LAX-MICRO** | **4 cores / 4 GB RAM / 80 GB NVMe, 10 TB traffic, DDoS protection** | **$400/yr (list $480)** | **Entry point: meets CPU and RAM minimums** | [Order LAX-MICRO](https://sixtynet.net/aff.php?aff=39) |
| LAX-MEDIUM | 8 cores / 8 GB RAM / 160 GB NVMe, 20 TB traffic, DDoS protection | $800/yr (list $960) | Comfortable for several channels and heavier automation | [Order LAX-MEDIUM](https://sixtynet.net/aff.php?aff=39) |
| LAX-LARGE | 8 cores / 16 GB RAM / 320 GB NVMe, 40 TB traffic, DDoS protection | $1,600/yr (list $1,920) | More than one agent needs | [Order LAX-LARGE](https://sixtynet.net/aff.php?aff=39) |

Annual billing matters here. At $400/year the LAX-MICRO works out to about $33 a month, but you pay the twelve months up front, so the real question is whether you'll still want the box in month nine.

A few plans don't appear in the table for practical reasons:

- **Larger residential tiers.** The page also lists 4, 8 and 16-core residential plans with 4, 8 and 16 GB RAM (80/160/320 GB NVMe). Their price blocks weren't visible in what I could read, so check the order page before assuming.
- **The AS9929 residential variant.** Same hardware with a China-optimised route and a stricter network posture: it rejects external probes and scans, and only whitelisted or port-forwarded devices get in. That's a manual setup step before you can even SSH, and refunds on it are issued as account balance rather than money back.
- **Los Angeles China Unicom-Premium.** $2,500/year (from $3,000), 100M baseline with burst and 95th-percentile billing. This is a bandwidth and BGP product for China-facing traffic, not an OpenClaw host.

All of these sit in Los Angeles. If you need a European or Asian region, SixtyNet isn't the answer yet.

## How small is too small for an OpenClaw VPS?

**4 GB of RAM is the practical floor, and two independent hosting providers put the same number on it.** Bluehost calls 2 vCPU, 4 GB DDR5 and 100 GB NVMe its functional minimum, and says OpenClaw stops running reliably below that once multiple channels and several automations are connected. OVHcloud's answer to the same question is 2 vCores and 4 GB for basic onboarding, with more memory and storage recommended for heavy browser automation.

That's why the $200 plans are a false economy for this workload. A 2 GB box will boot OpenClaw and answer a message. Add a second channel, a scheduled job that fires while a browser session is open, and the OOM killer starts deciding what dies. The official Linux docs even ship a dedicated page on OOM victim selection and diagnosing `exit 137`, which tells you how often this happens in the wild.

Shared hosting is worse than small. It typically blocks root and Docker, and rarely offers persistent volumes, so your skills and run history reset. OpenClaw needs three things from a server: **full root access, Docker support and persistent storage**. Root is what lets you install and roll back plugins, ClawHub skills and MCP connectors. Persistent storage is what keeps sessions and skills across a reboot.

SixtyNet's listings don't spell out root access. Its VPS run on KVM with a control panel offering 400+ OS templates including Docker, which normally means full root, but confirm it with support before you pay for a year, along with the persistent disk. Both are one-line questions and both decide whether the install works at all.

👉 [Check the current SixtyNet VPS price list and specs](https://sixtynet.net/aff.php?aff=39)

## Do closed ports 80 and 443 break OpenClaw?

No, and this is the part people misread on SixtyNet's residential plans.

OpenClaw's secure default is to keep the Gateway on loopback and reach it through an SSH tunnel or Tailscale Serve. You are not supposed to expose the dashboard on a public port. The official docs are explicit: if you bind the Gateway to `lan` or `tailnet`, it requires a shared secret (`gateway.auth.token` or `gateway.auth.password`) unless authentication is delegated to a trusted proxy.

So a residential plan with 80/443 closed can still run a Gateway fine, because nothing about that setup needs inbound web ports. What you can't do is host a public website on the same box, and SixtyNet says as much: residential IPs are for looking like a home connection, and the LAX series is for anything public-facing.

What does bite is the stricter AS9929 variant. "Rejects all external probes and scans" sounds great until you realise your own SSH session is an external probe. Budget time to whitelist your address or set up port forwarding before you start.

## Three things to do before you install anything

**1. Harden admin access first.** Before OpenClaw goes on a public VPS, decide how you'll administer the machine. Install Tailscale, join the VPS to your tailnet, verify a second SSH session over the Tailscale IP or MagicDNS name, then restrict public SSH. If you'd rather not use Tailscale, apply the equivalent hardening to your SSH path before exposing more services. This is a separate job from Gateway access.

**2. Keep the Gateway on loopback.** Tunnel in or use Tailscale Serve for the Control UI.

**3. Set a token the moment you bind wider.** Bind to `lan` or `tailnet` without `gateway.auth.token` and you've built an open relay to your own machine.

## Tuning a small or ARM box so it starts fast

OpenClaw's docs give three settings worth adding on modest hardware. The standard `openclaw onboard --install-daemon` path installs a systemd **user** unit, so edit it as your own user:

ini
# systemctl --user edit openclaw-gateway.service
[Service]
Environment=OPENCLAW_NO_RESPAWN=1
Environment=NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache
TimeoutStartSec=90


What each one does:

- `NODE_COMPILE_CACHE` speeds up repeated command and worker startups, and the first run warms the cache. The Gateway main thread deliberately stays uncached so synchronous cache writes don't delay startup and shutdown.
- `OPENCLAW_NO_RESPAWN=1` keeps routine Gateway restarts in-process, which avoids extra process handoffs and keeps PID tracking simple on small hosts.
- `TimeoutStartSec=90` stops slow hosts from being killed mid-boot. The managed unit already carries `Restart=always` and `RestartSec=5`.

Put state and cache paths on SSD-backed disks. SixtyNet's NVMe and the quoted 12,500 IOPS on the LAX-MICRO are the relevant number here, since cold starts are a random-I/O problem.

If you're running a system unit instead, the same edit goes through `sudo systemctl edit openclaw-gateway.service`. Raspberry Pi specifics live in a separate install doc.

## Will your automations survive a reboot?

**Yes, if the box comes back predictably.** OpenClaw logs separately whether each run fired, delivered and completed. Work missed during a restart resumes, and finished jobs don't run twice. That's the difference between a scheduler you trust and one you check by hand every morning.

The dependency is the server. A VPS that reboots in seconds and comes back on its own is fine. A box that hangs on boot is not, which is another reason to keep the install lean and the disk healthy on an 80 GB plan.

## Sharing one host with a team

A single agent for a whole team is a valid setup when everyone is inside the same trust boundary and the agent is business-only. The official requirements: keep it on a dedicated runtime with a dedicated OS user, and don't sign that runtime into personal Apple or Google accounts or a personal browser and password-manager profile. If users are adversarial to each other, split by gateway, host or OS user instead of sharing.

Approvals stay bound to the exact request, command, session and person that granted them, so permissions can't be replayed out of context. Removing or re-pairing a device revokes its old access immediately, and role-based access controls who can create and change automations.

## What the rest of the market charges

Worth knowing before you commit a year, since SixtyNet is not the cheapest per gigabyte:

- **GoDaddy** starts at $8.99/month for its Starter OpenClaw plan, one-click deployment with no SSH or Docker, daily backups, 99.9% uptime, root retained. Account provisioning can take up to 24 hours or longer.
- **Contabo** lists Cloud VPS 4 (4 vCPU / 8 GB / 100 GB SSD) at €5.50, or €4.40/month equivalent on a 24-month term. The page shows both figures, so treat the effective price as unconfirmed until you reach checkout, and note the 24-month commitment.
- **Bluehost** uses 2 vCPU / 4 GB / 100 GB NVMe (NVMe 4) as its stated floor but doesn't publish that plan's price in the material I reviewed.

SixtyNet's case rests on location, the residential IP option and annual pricing without a multi-year lock-in. If the lowest possible monthly number is the only criterion, a 24-month Contabo term wins on paper.

## FAQ

**Which models can I connect?** OpenAI, Anthropic Claude, or local models through Ollama, using your own API keys. Contabo also lists Gemini.

**Which messaging channels work?** Slack, Discord, Telegram, WhatsApp, Signal, Microsoft Teams and Twilio SMS. Accepted messages persist through restarts, and connection state reports as usable, recovering or blocked.

**Who handles OpenClaw updates?** You do. On Docker, that usually means pulling the latest image and recreating the container. Hosting providers are consistent on this point: the VPS is theirs, the application is yours.

**Can one box run multiple agents for different clients?** Yes, one instance can hold several agents configured per client. Size the RAM for the total load, and split hosts if the clients shouldn't see each other.

**Do I need Docker?** Root access is the hard requirement. Docker is the documented route for plugins, skills and MCP connectors; a bare-metal `npm install` also works.

**Which version will I get?** Bluehost's page states it deploys OpenClaw 2.0, version v2026.8.1, but the official docs don't publish a version number, so check what your own install pulls rather than assuming.

## How to decide, and what to confirm before paying

One agent, a few channels, a handful of automations: **LAX-MICRO at $400/year**. It's the smallest SixtyNet plan that meets the 4 GB memory floor, and 80 GB of NVMe is enough for state and workspace if you back them up and keep an eye on disk as browser automation artifacts pile up.

Several channels plus heavy scheduled work, or multiple agents on one host: **LAX-MEDIUM at $800/year**, where 8 GB of RAM and 160 GB of disk give you room to grow without rebuilding.

Two questions to send support before you pay for twelve months: does the plan include full root access, and is the storage persistent across reboots. Then check which payment methods the checkout actually offers; a third-party review reports card payments being suspended for fraud screening, and the official page notes that refunds on one residential product are issued as account balance. Get the current refund terms in writing before committing a year up front.

👉 [Pick your SixtyNet plan and start the OpenClaw install](https://sixtynet.net/aff.php?aff=39)
