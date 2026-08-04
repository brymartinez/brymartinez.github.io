# Staff Engineer and Fractional CTO Positioning Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Present Bryan as both a Staff engineer and fractional CTO, and make the Projects page support that position without adding unsupported claims.

**Architecture:** Keep the current Astro page structure and styles. Change only the home-page introduction and Projects page content. Merge the two related payment terminal entries into one main case study, then keep the other work as shorter supporting entries.

**Tech Stack:** Astro, TypeScript, Mermaid, CSS, npm

---

## File map

- Modify `src/pages/index.astro`: Update the hero message and project button label. Leave CV titles unchanged.
- Modify `src/pages/projects.astro`: Merge the payment terminal entries, improve project wording, and list the strongest case-study themes to document next.
- No CSS change is planned. The existing project, heading, diagram, and bullet styles cover the new content.

### Task 1: Update the home-page position

**Files:**
- Modify: `src/pages/index.astro:114-126`

- [ ] **Step 1: Record the current validation result**

Run:

```bash
npm run check
```

Expected: Exit code 0. This gives a clean baseline before the copy change.

- [ ] **Step 2: Replace the hero copy**

Change the hero section to:

```astro
<section class="hero">
  <h1>Bryan Louie Martinez</h1>
  <p>
    Staff engineer and fractional CTO who turns complex product and platform
    problems into reliable systems. I set technical direction, design cloud and
    event-driven architecture, and help teams deliver with confidence.
  </p>
  <div class="button-row">
    <a class="button button-primary" href="/projects">
      View selected engineering work
    </a>
    <a class="button button-primary" href="/notes">Read notes</a>
  </div>
</section>
```

Do not change the `experiences` array. Its roles are official job titles.

- [ ] **Step 3: Check the page source**

Run:

```bash
rg -ni "Staff engineer|Senior software engineer|View selected" src/pages/index.astro
```

Expected: The new Staff engineer message and button label appear. The old hero phrase does not appear. `Senior Software Engineer` remains in the CV data.

- [ ] **Step 4: Run the Astro check**

Run:

```bash
npm run check
```

Expected: Exit code 0 with no Astro errors.

- [ ] **Step 5: Commit the home-page change**

```bash
git add src/pages/index.astro
git commit -m "feat: position profile at staff level"
```

### Task 2: Reframe the Projects page

**Files:**
- Modify: `src/pages/projects.astro:5-130`

- [ ] **Step 1: Remove unused and split project data**

Delete the unused `telemetryDiagram` constant. Keep `onboardingDiagram`, `profileManagerDiagram`, and `arbitrageDiagram` unchanged.

Replace `projectLinks` with:

```ts
const projectLinks = [
  { label: "Payment Terminal Platform", href: "#payment-terminal-platform" },
  { label: "Arbitrage Trading", href: "#arbitrage-trading" },
  { label: "Public Products", href: "#public-products" },
  { label: "Next Case Studies", href: "#next-case-studies" },
];
```

- [ ] **Step 2: Replace the page introduction**

Keep the existing `BaseLayout` document title. Replace the opening content with:

```astro
<p class="eyebrow">Selected Engineering Work</p>
<p>
  Systems and products that show how I approach architecture, platform design,
  and delivery. Work projects use public or anonymized details.
</p>
```

- [ ] **Step 3: Merge the payment terminal entries**

Replace the separate `terminal-onboarding` and `profile-manager` articles with:

```astro
<article class="project-entry" id="payment-terminal-platform">
  <h2 class="project-title">
    <a href="#payment-terminal-platform">Payment Terminal Platform</a>
  </h2>
  <p>
    A group of services that provisions payment terminals before merchants use
    them. The platform coordinates onboarding, integrates with multiple terminal
    providers, and applies the correct device profile.
  </p>
  <h3>Onboarding flow</h3>
  <MermaidDiagram
    chart={onboardingDiagram}
    title="Terminal Onboarding Flow"
  />
  <p>
    SNS connects the provisioning services. Provider-specific proxies keep each
    external terminal API separate from the internal onboarding flow.
  </p>
  <h3>Profile design</h3>
  <MermaidDiagram
    chart={profileManagerDiagram}
    title="Profile Manager Flow"
  />
  <p>
    DynamoDB stores a hierarchy of shared and device-specific profiles. Common
    settings pass down to more specific terminal profiles.
  </p>
</article>
```

This wording must remain limited to facts already shown by the current copy and diagrams.

- [ ] **Step 4: Improve the supporting project copy**

Keep the existing Arbitrage Trading diagram. Replace its paragraph with:

```astro
<p>
  A market-making system that read continuous order book streams and made
  millisecond trading decisions across three exchanges.
</p>
```

Rename the `other-projects` article and anchor to `public-products`:

```astro
<article class="project-entry" id="public-products">
  <h2 class="project-title">
    <a href="#public-products">Public Products</a>
  </h2>
```

Keep the existing SoilMate and Raket.ph links inside that article.

- [ ] **Step 5: Add the project themes to document next**

Add this final article after Public Products:

```astro
<article class="project-entry" id="next-case-studies">
  <h2 class="project-title">
    <a href="#next-case-studies">Next Case Studies</a>
  </h2>
  <p>
    Strong candidates for future case studies, once their scope and results are
    ready to publish:
  </p>
  <ul class="project-bullets">
    <li>Engineering platforms, shared tooling, and delivery standards.</li>
    <li>Fractional CTO work covering product direction and team delivery.</li>
    <li>Cloud modernization with reliability, cost, or delivery results.</li>
    <li>Reliability programs that reduced incidents across systems.</li>
  </ul>
</article>
```

- [ ] **Step 6: Check anchors and removed IDs**

Run:

```bash
rg -n "terminal-onboarding|profile-manager|other-projects|payment-terminal-platform|public-products|next-case-studies" src/pages/projects.astro
```

Expected: The old IDs do not appear. Each new ID appears once in `projectLinks`, once on its article, and once in its heading link.

- [ ] **Step 7: Run page validation**

Run:

```bash
npm run check
npm run build
```

Expected: Both commands exit with code 0. The build creates the static home and Projects pages.

- [ ] **Step 8: Commit the Projects page change**

```bash
git add src/pages/projects.astro
git commit -m "feat: reframe selected engineering work"
```

### Task 3: Verify the finished site

**Files:**
- Verify: `src/pages/index.astro`
- Verify: `src/pages/projects.astro`

- [ ] **Step 1: Start the local site**

Run:

```bash
npm run dev
```

Expected: Astro prints a local URL and keeps running.

- [ ] **Step 2: Check the home page in a browser**

At a desktop width and a mobile width, confirm:

- The hero says `Staff engineer and fractional CTO`.
- The introduction wraps without clipping or overlap.
- The project button opens `/projects`.
- Every CV title is unchanged.

- [ ] **Step 3: Check the Projects page in a browser**

At a desktop width and a mobile width, confirm:

- The page heading says `Selected Engineering Work`.
- All four shortcut links move to the correct section.
- Both payment platform diagrams render inside one case study.
- The Arbitrage Trading diagram renders.
- SoilMate and Raket.ph links open their correct external pages.
- The future case-study list is readable.
- No content overflows the page.

- [ ] **Step 4: Run final automated validation**

Stop the development server, then run:

```bash
npm run check
npm run build
git diff --check
git status --short
```

Expected: Checks and build pass. `git diff --check` prints nothing. `git status --short` shows no uncommitted implementation files.
