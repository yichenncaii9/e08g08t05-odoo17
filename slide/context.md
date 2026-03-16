# ESMOS Week 10 Proposal — Slide Deck Context

## File
`esmos-week10-proposal (1).html` — single self-contained HTML file, 21 slides, full-page scroll-snap presentation.

---

## Project Identity
- **Course**: IS214 Enterprise Solution Management (ESM), SMU AY2025-26 Term 2
- **Group**: e08g08t05 | **Instructor**: Kiru
- **Framework**: ITIL 4.0 (NOT v3 — never use lifecycle terminology like Service Strategy/Design/Transition/Operation/CSI)
- **Audience**: CEO of healthcare client (non-technical, business value framing)

## Team Roles
| Member | Role |
|---|---|
| Yichen | System Configurator |
| Seann | Support Manager |
| Zachary | Security Manager |
| Sahanya | Product Manager |
| Shawmya | Data Manager |

---

## Slide Index (21 slides)

| # | Title | Background | Layout | Key content |
|---|---|---|---|---|
| 1 | Title | dark | bottom-aligned | ESMOS Healthcare Go-Live Proposal |
| 2 | The Problem | light | split | Business context + architecture placeholder |
| 3 | The Hard Dependency | light | single-col | No Odoo without Moodle training |
| 4 | Who We Serve | light | single-col | 4 user groups (steps list) |
| 5 | Three Systems | dark | thirds | Odoo / Moodle / Helpdesk with real images |
| 6 | ITIL 4 | light | split | SVC explanation + service value chain image |
| 7 | Plan Decisions | light | single-col | 4 strategic decisions (steps list) |
| 8 | Later Phases | light | single-col | Obtain/Build → Design & Transition → Deliver & Support |
| 9 | AKS | dark | split | AKS overview + architecture placeholder |
| 10 | Why AKS | light | single-col | 5 justifications (bullet-grid) |
| 11 | Testing Proposal | light | single-col | 4 tests: Moodle capacity, helpdesk throughput, WAF latency, self-healing |
| 12 | RFC 1 — WAF + Nginx | light | split | WAF change + success criteria note + WAF diagram placeholder |
| 13 | ITIL Lifecycle Loop | dark | single-col | PM2 incident → Problem Mgmt → RFC (preventive) → CI |
| 14 | RFC 2 — MFA | light | split | MFA enforcement details + risk/source callouts |
| 15 | Operations | light | split-wide | 7-step onboarding workflow + workflow screenshot placeholder |
| 16 | Incident Handling | light | single-col | RASCI escalation + RFC process |
| 17 | SLAs | dark | thirds | 99.5% availability / <15m P1 / <24h requests |
| 18 | Monitoring Strategy | light | split | What we monitor (WAF + infra) / Why it matters (LTA + PM2 lessons) |
| 19 | Cost & Risk | light | split | Cost & value (CEO framing) / Key risks mitigated |
| 20 | Security Posture | dark | single-col | 6 bullet-grid items, no MITRE IDs |
| 21 | Closing | light | centered | Thank you / Q&A |

---

## CSS Design System

### Color Tokens
```
--cream: #faf8f4        (slide background)
--warm-white: #f2efe8   (card backgrounds)
--charcoal: #1d1b18     (dark slide bg, heading text)
--mid: #5c5750          (body text on light slides)
--light: #948e85        (labels, muted text)
--accent: #d4321c       (red — numbers, bullets, CTAs)
--accent-bg: #fdf0ed
--teal: #1a7a6d         (secondary accent)
--teal-bg: #edf7f5
```

### Typography
- Headings: Fraunces (Google Fonts, serif, italic-capable)
- Body: Work Sans (Google Fonts, sans-serif)
- All sizes use `clamp()` for responsive scaling

### Layout Classes
| Class | Description |
|---|---|
| `.slide` | Full-viewport scroll-snap section, light bg by default |
| `.slide.dark` | Dark background (`--charcoal`), light text |
| `.slide-content` | Centered flex column inside slide with padding |
| `.split` | 2-col grid (1fr 1fr), aligned center |
| `.split-wide` | 2-col grid (1.4fr 1fr), aligned start |
| `.thirds` | 3-col equal grid |

### List Classes
| Class | Usage | Notes |
|---|---|---|
| `.steps` | Numbered step lists | Counter auto-increments, number in `--accent` |
| `.clean-list` | Bullet dot lists | `::before` dot, good for short unlabelled items |
| `.bullet-grid` | Label — description lists | `max-content 1fr` grid, aligns all em-dashes; use `<strong>Label</strong><span>— description</span>` inside each `<li>`; `display:contents` on `li` |

**Important**: On `.dark` slides, `strong` inside `.steps li` and `.clean-list li` will be invisible without the dark overrides already in CSS. Do NOT add new lists on dark slides without checking this.

### Block Classes
| Class | Description |
|---|---|
| `.accent-block` | Red left-border callout box (light bg only) |
| `.teal-block` | Teal left-border callout box (usable on light or dark) |
| `.stat` + `.stat-label` | Big number + label (used in slide 17 SLAs) |
| `.stat-card` | Card with left red border (available but not currently used) |
| `.divider` | Red horizontal rule, 3px |

### Image Classes
| Class | Usage |
|---|---|
| `.img-placeholder` | Dashed box placeholder (still used for missing images) |
| `.img-placeholder.wide` | 16:9 placeholder |
| `.slide-img` | Real images; `object-fit:contain` by default |
| `.slide-img.wide` | 16:9 ratio image |
| `.dark .slide-img` | Auto dark background (`#2a2724`) |

For real images, prefer inline `style="aspect-ratio:X/Y;object-fit:cover;object-position:top center;"` on `.slide-img` when you need crop-to-fill behaviour.

### Reveal Animation
Add class `reveal` to any element for fade-in-up on scroll into view. Delay staggering on `nth-child(2/3/4)` is automatic.

---

## Available Images (`img/` directory)
| File | Dimensions | Used in |
|---|---|---|
| `odoo.jpg` | 438×274 | Slide 5 (Three Systems) |
| `moodle.png` | 2974×1442 | Slide 5 (Three Systems) |
| `helpdesk.jpg` | 304×310 | Slide 5 (Three Systems) |
| `service value chain.png` | 866×590 | Slide 6 (ITIL 4) |

Still needed (placeholders remain):
- Architecture overview diagram (slide 2)
- AKS architecture diagram (slide 9)
- WAF architecture diagram (slide 12)
- Workflow screenshot (slide 15)

---

## Content Rules (Kiru / grading constraints)

1. **ITIL 4 only** — never v3 lifecycle terminology
2. **BECAUSE reasoning** — every architecture/cost decision must be justified with "X BECAUSE Y"
3. **CEO framing** — business value over technical config; no MITRE ATT&CK/D3FEND IDs
4. **No premature results** — load tests and WAF demos are Week 12 live demo; slide 11 is strategy not evidence
5. **Two RFCs** — RFC 1 (WAF + Nginx, non-functional, no workflow impact) and RFC 2 (MFA, process change, modifies onboarding BPMN, requires CAB)
6. **Onboarding hard dependency** — Odoo access only after verified Moodle completion, enforced through helpdesk workflow not policy
7. **SLA scope** — 99.5% business hours only (Mon–Fri 8am–8pm SGT), NOT 24/7
8. **Client notification** — only for Major/Critical incidents with business impact; informational events stay internal

## Key Facts to Preserve
- Infrastructure: AKS (Azure Kubernetes Service), Southeast Asia region
- Stack: Odoo + Moodle + PostgreSQL + Nginx Ingress + ModSecurity (OWASP CRS)
- Cost: ~$30/month, free AKS control plane, ~3 months runway on $100 Azure for Students credits
- SLAs: 99.5% uptime | P1 ack <15 min, resolve <2 hrs | Moodle access <4 hrs | Odoo access <24 hrs post-training
- PM2 incident: web defacement via compromised account with Settings-level privileges → motivated RFC 1 (WAF) and RFC 2 (MFA)
- LTA lesson: monitor service quality (what users experience), not just infrastructure uptime
- Monitoring tools: Azure Monitor (infra) + Better Stack (synthetic checks, port 8069, 30s interval)

---

## What NOT to Change
- The `.bullet-grid` implementation uses `display:contents` on `<li>` — this is intentional for column alignment
- Dark slides already have overrides for `.steps li strong` and `.clean-list li strong` — don't remove them
- The `service value chain.png` uses `aspect-ratio:866/590` (natural ratio) to avoid letterboxing
- Three system images (odoo/moodle/helpdesk) use `aspect-ratio:16/9;object-fit:cover;object-position:top center` — all identical size
- Slide 13 ITIL loop step 3 says "preventive measure" NOT "permanent fix"
- No inline `color:#8a857e` anywhere — was fixed to `#b8b3ab` for dark bg readability
