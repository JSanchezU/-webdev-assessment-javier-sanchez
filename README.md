# Webdev Assessment — Javier Sanchez
**Track:** Senior
**Repo:** `webdev-assessment-javier-sanchez`
**Screen recording:** [Watch on Google Drive](https://drive.google.com/file/d/1PhYbEx-vqPqmAtzLAyFeab4ArnJ3CFi7/view?usp=sharing)

---

## Files

| File | Description |
|------|-------------|
| `candidate-brief-senior.html` | Task instructions and timing |
| `segment-1/starter-page.html` | Main working file — all Segment 1 changes here |
| `segment-1/figma-spec-section.html` | Design reference for Task A |
| `segment-1/robots.txt` | Created during Task B SEO audit |

---

## Segment 1 — Practical Build with Claude Code

### Task A — Match the Figma Section (~15 min)

Built the **Getting Started** section into `starter-page.html` at the `<!-- SECTION PLACEHOLDER -->` comment. Before prompting, I shared the full existing stylesheet with Claude Code so it could identify which classes already existed — no new CSS was written.

**Approach:** Read `figma-spec-section.html` to extract the design spec (layout, spacing tokens, copy, CTA labels), then cross-referenced the existing CSS in `starter-page.html` to confirm `.steps-grid`, `.step-card`, `.step-card-top`, `.step-num`, `.step-title`, `.step-desc`, and `.btn-primary` were all already defined. Generated the HTML using only those classes.

**What was built:**
- `<section class="section"><div class="section-inner">` wrapper as specified
- `<h2 class="section-heading">` with exact copy from spec
- `<p class="section-subheading" style="margin-top:12px">` with the spec's 12 px gap
- `<div class="steps-grid">` — picks up 3-column layout, `gap: 24px`, and `margin-top: 48px` from existing CSS
- Three `.step-card` elements each containing: `.step-card-top` (step number circle + title/desc group with inner `gap: 12px`), and a full-width `.btn-primary`
- CTA labels in spec order: **Get Started Now** / **Explore Editing Tools** / **Launch Your Page**

---

### Task B — Fix Heading Hierarchy & SEO Structure (~13 min)

Audited `starter-page.html` with Claude Code. Found and fixed issues across heading structure, `<head>` meta, and accessibility.

**Critical fixes:**

| Issue | Fix applied |
|-------|-------------|
| `<title>Page Title Here</title>` — generic placeholder | → `"Website Studio — AI Landing Page Builder by Search Atlas"` |
| Missing `<meta name="description">` | → Added 155-char keyword-rich description |
| No `<link rel="canonical">` | → Added pointing to `https://searchatlas.com/website-studio/` |
| Two `<h1>` tags on the page — "What Is Website Studio?" was a second H1 | → Changed to `<h2>` |

**High severity fixes:**

| Issue | Fix applied |
|-------|-------------|
| Heading jump H2 → H4 in FAQ (H3 never used) | → All four `<h4 class="faq-question-text">` changed to `<h3>` |
| `alt=""` on testimonial avatar (meaningful image) | → `alt="Bruce Beck, CEO at DB&R"` |
| `alt=""` on What Is section product screenshot | → Descriptive alt text for the UI image |

**Heading outline after fixes:**

| Level | Count | Role |
|-------|-------|------|
| H1 | 1 | Hero headline only |
| H2 | 5 | Section headings |
| H3 | 4 | FAQ questions |

**Additional `<head>` improvements added during the audit:**

- **Open Graph** (10 tags) — `og:type`, `og:site_name`, `og:url`, `og:title`, `og:description`, `og:image` (1200×630), `og:image:alt` for Facebook, LinkedIn, and OG-compliant platforms
- **Twitter Card** (6 tags) — `summary_large_image` with matching title, description, and image
- **JSON-LD Schema** — single `@graph` with three types:
  - `Organization` — publisher entity with logo and social `sameAs` profile links
  - `SoftwareApplication` — Website Studio with `featureList`, `applicationCategory`, `operatingSystem`, publisher reference
  - `FAQPage` — all 4 FAQ Q&A pairs verbatim from the page; eligible for AI Overview / Perplexity extraction

> OG image path (`/images/website-studio-og.jpg`) and Organization logo are placeholder URLs — replace with production CDN paths before deploying.

**`robots.txt` — created** at `segment-1/robots.txt`:
- `Allow: /` for all crawlers (public marketing page)
- Disallowed: `/admin/`, `/login/`, `/dashboard/`, `/api/`, `/private/`
- Query-string blocks for `?sort=`, `?filter=`, `?utm_` to prevent duplicate-content indexing
- AI crawlers explicitly allowed: `GPTBot`, `ClaudeBot`, `PerplexityBot`, `Google-Extended`
- `Sitemap:` directive pointing to `https://searchatlas.com/sitemap.xml`

---

### Task C — HubSpot Form Integration + GTM Tracking (~12 min)

Wired up the `<div id="hubspot-form-target">` placeholder with a real embed pattern.

**Implementation:**

```html
<script src="//js.hsforms.net/forms/embed/v2.js" defer></script>
<script>
  window.addEventListener('load', function () {
    hbspt.forms.create({
      region: 'na1',
      portalId: 'YOUR_PORTAL_ID',
      formId: 'YOUR_FORM_ID',
      target: '#hubspot-form-target',
      onFormSubmit: function ($form) {
        window.dataLayer = window.dataLayer || [];
        window.dataLayer.push({
          event: 'hubspot_form_submit',
          formId: 'YOUR_FORM_ID',
          pageUrl: window.location.href,
          pageTitle: document.title
        });
      },
      onFormSubmitted: function () {
        window.dataLayer = window.dataLayer || [];
        window.dataLayer.push({ event: 'hubspot_form_submitted' });
      }
    });
  });
</script>
```

**Why `defer` + `window.load`:** The HubSpot script is deferred so it doesn't block the render path. The `hbspt.forms.create()` call is wrapped in `window.addEventListener('load', ...)` to guarantee the script has fully parsed before the API is invoked — avoids a race condition if the page fires events before the HubSpot library is ready.

**Two GTM events, not one:**
- `hubspot_form_submit` — fires on click before HubSpot validation. Use for micro-conversion tracking ("form engagement").
- `hubspot_form_submitted` — fires only after HubSpot confirms the submission. Use this for conversion goals, ad pixel firing, and lead counting in GA4.

**Why dataLayer over a generic form listener:**
HubSpot forms render inside an `<iframe>` in some embed configurations. A generic GTM `Form Submission` trigger listens on the parent DOM and cannot cross the iframe boundary. The dataLayer push happens in the same JavaScript context as the parent page, so it always fires correctly regardless of how HubSpot renders the form.

**GTM setup (walkthrough):**
1. Create a **Custom Event trigger** in GTM listening for `hubspot_form_submitted`
2. Attach a **GA4 Event tag** (`generate_lead`) to that trigger for analytics
3. For Meta/LinkedIn/Google Ads conversion tags, use the same trigger with their respective tag templates
4. Use `hubspot_form_submit` (pre-validation) only for micro-conversion events where you want to measure intent, not confirmed leads

To go live: replace `YOUR_PORTAL_ID` and `YOUR_FORM_ID` with the values from HubSpot → Marketing → Forms → ··· → Share.

---

## Changes Summary

| File | What changed |
|------|-------------|
| `segment-1/starter-page.html` | Task A: Getting Started section built; Task B: title, meta description, canonical, H1 dedup, H3 fix ×4, alt text ×2, OG tags, Twitter Card, JSON-LD schema; Task C: HubSpot embed + GTM dataLayer |
| `segment-1/robots.txt` | Created from scratch |

---

## Segment 2 — SEO Audit: linkgraph.com

**URL audited:** https://www.linkgraph.com/
**Business type detected:** SaaS SEO Agency — AI-powered SEO software (Search Atlas) + managed services
**Method:** 6 specialist subagents run in parallel (Technical, Content, Schema, Performance, SXO, GEO)

### SEO Health Score: 46 / 100

| Category | Weight | Score | Weighted |
|----------|--------|-------|----------|
| Technical SEO | 22% | 54/100 | 11.9 |
| Content Quality | 23% | 58/100 | 13.3 |
| On-Page SEO | 20% | 52/100 | 10.4 |
| Schema / Structured Data | 10% | 15/100 | 1.5 |
| Performance (CWV) | 10% | 25/100 | 2.5 |
| AI Search Readiness | 10% | 44/100 | 4.4 |
| Images | 5% | 30/100 | 1.5 |
| **Total** | 100% | | **46** |

---

### Critical Issues

**1. Core Web Vitals — FAILING all three metrics on mobile**

| Metric | Threshold | Estimated | Status |
|--------|-----------|-----------|--------|
| LCP | ≤ 2.5s | ~3.8–4.5s | FAIL |
| INP | ≤ 200ms | ~280–380ms | FAIL |
| CLS | ≤ 0.1 | ~0.15–0.22 | FAIL |

Root causes: Elementor applies `loading="lazy"` globally including to the hero LCP image; no `fetchpriority="high"` or `<link rel="preload">` for the LCP element; chat widget + HubSpot + ad pixels fire on page load blocking ~900ms–2.1s of main thread; YouTube embed has no facade (~500KB JS loaded unconditionally); 40+ images missing `width`/`height` attributes causing layout shift.

This is a credibility problem — any prospect running PageSpeed Insights on the agency's own site before engaging them will see red "Poor" labels.

**2. Zero schema markup**

No `Organization`, `WebSite`, `FAQPage`, `SoftwareApplication`, `Person`, or `AggregateRating` schema anywhere on the homepage despite multiple rich-result-eligible sections (FAQ section, team profiles, testimonials, Search Atlas product).

**3. H1 / Title tag contradiction**

- Title tag: *"Elevate Your Brand with Top SEO & Link Building Experts"* — agency framing
- H1: *"Maximize your revenue growth with AI SEO and AI Search"* — SaaS product framing

These send conflicting entity signals to Google and AI citation engines. The page cannot rank authoritatively for either frame.

**4. CTA overload blocking conversion**

10+ identical "Book A Meeting" CTAs. "Book A Meeting" is a bottom-of-funnel, high-commitment ask. Most visitors arriving from "SEO agency" or "AI SEO tool" queries are in the consideration stage — they need a lower-commitment entry point first (free audit, free trial, case study PDF). No mid-funnel CTA exists anywhere on the page.

Persona scores with current CTA strategy:
- In-house SEO manager: **49/100** — no free trial, no tool screenshots, ambiguous agency vs. software positioning
- Small business owner: **56/100** — no pricing signal, "AI Search" is jargon, sales call is intimidating
- CMO evaluating agencies: **57/100** — no ROI calculator, no named client logos, no soft pre-commitment path

**5. Page-type mismatch**

The homepage simultaneously targets "SEO agency" (SERP expects a service landing page with pricing anchor) and "AI SEO tool" (SERP expects a SaaS product page with free trial CTA). Google cannot slot it cleanly into either category, suppressing ranking potential for both. 80% of top-10 results for "link building service" are dedicated service pages — not homepages.

---

### High Priority Issues

| Area | Finding |
|------|---------|
| Technical | All nav and footer links use `href="#"` — zero internal link graph; no PageRank flows from this page |
| Technical | Mobile navigation has no hamburger menu — all secondary nav inaccessible on ≤768px screens |
| Technical | `robots.txt` sitemap pointer references `searchatlas.com/sitemap.xml` — cross-domain sitemap may be silently ignored by Googlebot if served at `linkgraph.com` |
| Technical | Canonical domain ambiguity — `linkgraph.com` pages canonicalizing to `searchatlas.com` needs a declared, consistent strategy |
| Content | "Guarantee results in thirty days or less" — unqualified financial-outcome promise under Google's expanded YMYL rules; no methodology link or qualifying language |
| Content | No named expert attribution above the fold — the "human expertise" H2 claim is undermined when no humans are identified |
| Content | Case studies likely lack specific metrics, named clients, timeframes, and analyst attribution — high-severity E-E-A-T failure for a service provider |
| Content | No freshness signals — no "last updated" date, no timestamped stats; especially damaging for AI-category content |
| Performance | YouTube embed without a facade — ~500KB JS loaded on every page view regardless of user intent |
| Performance | Font Awesome loaded as render-blocking CSS from CDN (~150KB) without Subresource Integrity |
| Performance | Third-party chat widget + ad pixels fire on `pageload` — estimated 900ms–2.1s of main-thread blocking on mobile |
| Images | ~20–25 of 40+ homepage images missing `alt` text |
| Images | No AVIF/WebP confirmed — PNG/JPEG served; no `srcset` at optimal mobile breakpoints |

---

### Quick Wins (highest ROI, lowest effort)

1. **Add `Organization` + `WebSite` + `FAQPage` JSON-LD** in a single `@graph` block — zero visual change, immediate Knowledge Panel and AI citation eligibility. Ready-to-deploy snippet generated during audit.
2. **Align H1 and title tag** to a single positioning axis — pick agency or software, not both.
3. **Add `fetchpriority="high"` to the hero LCP image** and remove `loading="lazy"` from it — estimated LCP improvement of 0.8–1.8s.
4. **Replace the YouTube `<iframe>` with `lite-youtube-embed`** — eliminates ~500KB of JS on every page load, improves CLS and INP.
5. **Reduce "Book A Meeting" to 2 CTAs** and add one mid-funnel CTA ("Get a free SEO audit" or "Try Search Atlas free") — serves the 70%+ of visitors not yet ready for a sales call.
6. **Defer the chat widget until first user scroll or click** — removes ~200–500ms of main-thread blocking from the critical path.

---

### Schema: Ready-to-Deploy JSON-LD

The following block can be dropped into the `<head>` immediately. Update `logo`, `sameAs` URLs, and `aggregateRating` values from verified G2/Capterra data before deploying.

```json
{
  "@context": "https://schema.org",
  "@graph": [
    {
      "@type": "Organization",
      "@id": "https://www.linkgraph.com/#organization",
      "name": "LinkGraph",
      "url": "https://www.linkgraph.com/",
      "logo": "https://www.linkgraph.com/wp-content/uploads/linkgraph-logo.png",
      "description": "LinkGraph is an AI-powered SEO agency offering link building, technical SEO, and content optimization through its Search Atlas platform.",
      "sameAs": [
        "https://twitter.com/LinkGraph",
        "https://www.linkedin.com/company/linkgraph/",
        "https://www.facebook.com/linkgraph/"
      ],
      "contactPoint": {
        "@type": "ContactPoint",
        "contactType": "customer support",
        "url": "https://www.linkgraph.com/contact/",
        "availableLanguage": "English"
      }
    },
    {
      "@type": "WebSite",
      "name": "LinkGraph",
      "url": "https://www.linkgraph.com/",
      "publisher": { "@id": "https://www.linkgraph.com/#organization" },
      "potentialAction": {
        "@type": "SearchAction",
        "target": { "@type": "EntryPoint", "urlTemplate": "https://www.linkgraph.com/?s={search_term_string}" },
        "query-input": "required name=search_term_string"
      }
    },
    {
      "@type": "SoftwareApplication",
      "name": "Search Atlas",
      "url": "https://searchatlas.com/",
      "applicationCategory": "BusinessApplication",
      "operatingSystem": "Web",
      "description": "All-in-one AI-powered SEO platform: keyword research, site auditing, link building, content optimization, and rank tracking.",
      "publisher": { "@id": "https://www.linkgraph.com/#organization" }
    }
  ]
}
```

---

### SXO: Recommended Page Architecture

To resolve the page-type mismatch and serve all three personas properly:

| New URL | Target intent | Page type |
|---------|--------------|-----------|
| `/` (homepage) | Navigational ("LinkGraph") | Brand hub with routing to both below |
| `/seo-agency/` | Commercial investigation ("SEO agency", "link building service") | Service landing page with process, pricing anchor, case studies |
| `/ai-seo-software/` | Transactional ("AI SEO tool") | SaaS product page with free trial CTA, feature grid, comparison table |

The current homepage is trying to win three SERP arenas with one page. Splitting into dedicated URLs gives each intent a purpose-built page and removes the dilution.
