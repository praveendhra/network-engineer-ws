# Job Application Portfolio — How to Recreate This for Any Role

A step-by-step playbook to build a fully personalized, standalone HTML work sample portfolio from a job description and a resume PDF. No frameworks, no build tools — just Python and a text editor.

---

## What You End Up With

| File | Purpose |
|---|---|
| `index.html` | Main teal-themed work sample — scenario-based technical proposal mapped to the JD |
| `network-workflows.html` | Standalone tabbed visual flow diagrams tied to the work sample |
| `approach.md` | Your private research doc — role breakdown, cert roadmap, domain notes |
| `work-sample.md` | Markdown draft (optional, used as source before HTML conversion) |

---

## Prerequisites

```bash
pip3 install pdfplumber --break-system-packages   # for resume parsing
```

Python 3.x, a browser, and a code editor are all you need.

---

## Step 1 — Analyze the Job Description

Read the JD carefully and extract:

- **Hard requirements** — certifications, years of experience, must-have tools
- **Soft requirements** — phrases like "ownership", "documentation", "cross-functional" (these tell you the culture)
- **Domain specifics** — healthcare, finance, manufacturing each add compliance/regulatory layers
- **Tech stack** — note every product named (Cisco, Palo Alto, SolarWinds, ServiceNow, etc.)
- **Top 3 pain points** — what problem is this team actually trying to solve?

Create `approach.md` with:
1. Role breakdown by section
2. Your gap analysis (what you know well vs. what you need to brush up on)
3. Cert roadmap relevant to the role
4. Domain-specific considerations (e.g. HIPAA for healthcare, PCI-DSS for finance)
5. KPIs and metrics you'll use in the work sample
6. Tooling stack the employer uses

---

## Step 2 — Create the Work Sample (Markdown First)

Write `work-sample.md` as a scenario-based technical proposal. Structure it around a realistic problem the team faces.

**Recommended sections (adapt to the role):**

1. **Executive Summary** — 2–3 sentences: what problem, what approach, what outcome
2. **Scenario** — describe the hypothetical environment (size, scale, constraints)
3. **Current State Assessment** — what's broken, what's the gap
4. **Proposed Architecture / Solution** — the core technical design
5. **Security** — how you bake security in from the start
6. **Key Technical Area 1** — deepest domain skill from the JD (e.g. SD-WAN, Kubernetes, CI/CD)
7. **Key Technical Area 2** — second domain skill
8. **Monitoring & Observability** — how you know it's working
9. **Change Management** — how you roll it out safely
10. **Root Cause Analysis** — walk through a real-style incident
11. **Disaster Recovery** — runbook-level, not just "we have backups"
12. **Automation** — what you'd automate and how
13. **Why I Stand Out** — 3–5 specific differentiators backed by evidence
14. **About Me & Fit** — bridge your actual experience to JD requirements

**Tips:**
- Every section should reference at least one specific JD requirement
- Use real product names, CLI commands, and metrics — vague is forgettable
- Include a concrete incident/RCA story (real or realistic composite)
- Add measurable outcomes: percentages, time savings, SLA numbers

---

## Step 3 — Convert to Standalone HTML

The HTML file is fully self-contained (no CDN, no external assets). Copy the structure from `index.html` and adapt:

### Layout skeleton

```
<body>
  <nav id="sidebar">          ← fixed left nav with scroll-aware active highlight
  <div id="main">
    <div id="hero">           ← gradient hero + 3 stat cards
    <section id="...">        ← one <section> per content block
```

### CSS custom properties to change for a different color theme

```css
:root {
  --teal-900: #0d3331;   /* swap these for any hue */
  --teal-700: #0e6655;
  --teal-500: #14967a;
  --teal-300: #4ecba8;
  --teal-100: #d0f5ec;
  --teal-50:  #f0fdf8;
  --dark:     #0a2826;
}
```

### Components available (copy from `index.html`)

| Class | What it renders |
|---|---|
| `.card-grid` | 2–3 column info cards |
| `.data-table` | Styled comparison/spec table |
| `.checklist` | CSS-drawn checkbox list |
| `.steps` | Auto-numbered step list |
| `.timeline` | Vertical timeline with dots |
| `.topology-box` | ASCII/text network diagram block |
| `.kpi-row` | Horizontal KPI metric strip |
| `.code-block` | Dark background CLI/config block |
| `.tag` | Small colored label pill |

### Sidebar nav

One `<a href="#section-id">` per section. The IntersectionObserver JS auto-highlights the active link on scroll. External links (e.g. to the workflows page) work normally — only `#anchor` links get smooth-scroll treatment.

### No-cache headers (keep for GitHub Pages / direct file serving)

```html
<meta http-equiv="Cache-Control" content="no-cache, no-store, must-revalidate" />
<meta http-equiv="Pragma" content="no-cache" />
<meta http-equiv="Expires" content="0" />
```

---

## Step 4 — Parse the Resume and Add Personal Details

```python
import pdfplumber

with pdfplumber.open("YourResume.pdf") as pdf:
    for page in pdf.pages:
        print(page.extract_text())
```

From the extracted text, identify:
- Name, location, contact
- Each role: company, title, dates, bullet points
- Skills and certifications

Then add an **About Me & Fit** section at the bottom of `index.html` with:

1. **Profile card** — name, title, contact, short bio
2. **Skill tags** — visual pills for each relevant skill
3. **Experience bridge grid** — 2-column cards: left = your actual experience, right = how it maps to a specific JD requirement

Example bridge card:
```
Your experience                 →   JD requirement
"Zoho: 3-year F5 NGINX load     →   "Experience with load balancers
 balancer administration"            and traffic management"
```

Aim for 5–7 bridge cards covering the JD's top requirements.

---

## Step 5 — Build the Visual Workflow Diagrams

Create `network-workflows.html` as a separate tabbed page. One tab per major process flow.

**Suggested tabs (adapt to the role):**

| Tab | What to show |
|---|---|
| Core technical flow | The main system/data flow from your work sample |
| Authentication / access flow | How identities or requests are verified |
| Change management lifecycle | From request to post-review |
| Incident response + RCA | Detection → fix → prevention |
| DR / failover runbook | Step-by-step with decision branches |

### Flow diagram components

```html
<!-- Single node -->
<div class="n n-proc">
  <div class="n-title">Step Name</div>
  <div class="n-sub">Detail text</div>
</div>

<!-- Connector (animated flowing line) -->
<div class="conn"><div class="conn-line"></div><div class="conn-head"></div></div>

<!-- Branch split -->
<div class="branch-wrap">
  <div class="conn short">...</div>
  <div class="split-bar" style="--bar-w:80%;"></div>
  <div class="branches" style="width:80%;margin:0 auto">
    <div class="branch-col"> ... </div>
    <div class="branch-col"> ... </div>
  </div>
  <div class="merge-bar" style="--bar-w:80%;"></div>
</div>
```

**Critical rule:** `--bar-w` on `.split-bar` and `.merge-bar` must exactly match the `width` on `.branches`. This keeps the vertical connector lines from each branch column aligned with the horizontal bar.

### Node color classes

| Class | Meaning |
|---|---|
| `n-start` | Entry point / trigger |
| `n-end` | Terminal / success |
| `n-proc` | Process / action step |
| `n-dec` | Decision / branch point |
| `n-ok` | Success path |
| `n-err` | Failure / escalation |
| `n-info` | Documentation / logging |
| `n-teal` | Highlight / real example callout |

---

## Step 6 — Humanize the Content

After drafting, do a pass to:

- Remove section number prefixes (`1.2 —` style headings → plain headings)
- Replace formal/passive voice with direct statements ("The system monitors..." → "It monitors...")
- Remove em dashes (`—`) — replace with `,` or `:` depending on context
- Make the About Me bridge items sound like a conversation, not a résumé bullet
- Check that every heading could be understood by a smart non-specialist

Quick Python to strip em dashes across both HTML files:

```python
import re, pathlib

for fname in ["index.html", "network-workflows.html"]:
    p = pathlib.Path(fname)
    text = p.read_text(encoding="utf-8")
    text = re.sub(r'\s*\u2014\s*', ', ', text)   # em dash → comma
    p.write_text(text, encoding="utf-8")
```

---

## Step 7 — Final Checks

- [ ] Every sidebar nav link scrolls to the correct section
- [ ] "Visual Workflows" link opens the workflows page (must start with a filename, not `#`)
- [ ] No broken internal anchor IDs (check `href="#x"` matches `id="x"`)
- [ ] Checklist icons render as CSS boxes, not Unicode glyphs
- [ ] Timeline dots align with the vertical line (adjust `left` on `.tl-item::before` if off)
- [ ] Branch split/merge bars have matching widths with their `.branches` container
- [ ] Both files have no-cache meta tags
- [ ] Resume PDF is not committed to the repo if the repo is public

---

## Step 8 — Deploy to GitHub Pages

```bash
git init
git add index.html network-workflows.html
git commit -m "initial work sample"
git remote add origin https://github.com/USERNAME/REPO.git
git push -u origin main
```

In the repo settings → Pages → set source to `main` branch, root `/`. Your portfolio will be live at `https://USERNAME.github.io/REPO/`.

Add a `CNAME` file if you have a custom domain.

---

## Adapting to a Different Role

| Role type | Sections to emphasize | Diagrams to build |
|---|---|---|
| Network Engineer | SD-WAN, NAC, VoIP QoS, DR runbook | Traffic routing, 802.1X auth flow |
| DevOps / Platform | CI/CD pipeline, IaC, observability | Deploy pipeline, incident response |
| Cloud Architect | Multi-region design, FinOps, IAM | Account structure, data flow, DR |
| Security Engineer | Threat model, SIEM, IR playbook | Attack path, detection pipeline |
| Full-Stack Engineer | System design, API design, scaling | Request lifecycle, data model |

The structure stays the same. Only the domain vocabulary, tools, and diagram flows change.
