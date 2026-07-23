# LIREI Website Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the bilingual (FR default / EN) static academic website for LIREI (UQTR) with team, research, projects and publications catalogs, deployable to a self-hosted server.

**Architecture:** Astro 5 static site with built-in i18n routing (FR at `/`, EN at `/en/`). All page copy lives in `src/i18n/fr.json` / `en.json`; each page is implemented ONCE as a shared component in `src/components/pages/`, and thin route files per locale pass `lang`. Structured data (team, publications, projects) lives in typed Astro content collections. `npm run build` doubles as the type/schema verification.

**Tech Stack:** Astro 5, TypeScript, Zod (via astro:content), vanilla JS for filters, custom CSS. No UI framework.

**Verification command throughout:** `npm run build` (must exit 0). Dev server: `npm run dev`.

---

## File Structure

```
lireiweb/
├── astro.config.mjs            # i18n config, configurable base path
├── package.json / tsconfig.json
├── public/favicon.svg
├── src/
│   ├── styles/global.css       # design system (academic palette, typography)
│   ├── i18n/
│   │   ├── fr.json  en.json    # ALL UI strings and page copy
│   │   └── index.ts            # t() helper, route map, href(), Lang type
│   ├── layouts/Base.astro      # <html>, meta, hreflang, Header, Footer
│   ├── components/
│   │   ├── Header.astro Footer.astro LangSwitch.astro
│   │   ├── MemberCard.astro PublicationItem.astro ProjectCard.astro
│   │   └── pages/              # one shared component per page (Home.astro, Team.astro, …)
│   ├── content.config.ts       # collections: team, publications, projects
│   ├── content/
│   │   ├── team/*.json         # one file per member
│   │   ├── publications/*.json # seeded from Semantic Scholar
│   │   └── projects/*.md
│   └── pages/
│       ├── index.astro laboratoire.astro equipe.astro recherche.astro
│       │   projets.astro publications.astro contact.astro          (FR routes)
│       └── en/index.astro laboratory.astro team.astro research.astro
│           projects.astro publications.astro contact.astro         (EN routes)
└── README.md
```

Research-axis keys used everywhere: `smartgrids`, `residential`, `ml`, `hydrogen`, `flexibility`, `ev`.

---

### Task 1: Scaffold Astro project

**Files:** Create `package.json`, `astro.config.mjs`, `tsconfig.json`, `.gitignore`, `src/pages/index.astro`, `public/favicon.svg`

- [ ] **Step 1: package.json + install**

```json
{
  "name": "lireiweb",
  "type": "module",
  "version": "0.1.0",
  "scripts": {
    "dev": "astro dev",
    "build": "astro build",
    "preview": "astro preview"
  }
}
```

Run: `npm install astro` — expected: astro ^5 added to dependencies.

- [ ] **Step 2: astro.config.mjs**

```js
import { defineConfig } from 'astro/config';

export default defineConfig({
  site: 'https://www.uqtr.ca',
  base: process.env.BASE_PATH ?? '/',
  i18n: {
    defaultLocale: 'fr',
    locales: ['fr', 'en'],
    routing: { prefixDefaultLocale: false },
  },
});
```

- [ ] **Step 3: tsconfig.json**

```json
{
  "extends": "astro/tsconfigs/strict",
  "include": [".astro/types.d.ts", "**/*"],
  "exclude": ["dist"]
}
```

- [ ] **Step 4: .gitignore** — `node_modules/`, `dist/`, `.astro/`

- [ ] **Step 5: placeholder `src/pages/index.astro`** (`<h1>LIREI</h1>` minimal) and `public/favicon.svg` (simple bolt/leaf mark, deep blue + green).

- [ ] **Step 6: Verify** — Run `npm run build`. Expected: exit 0, `dist/index.html` exists.

- [ ] **Step 7: Commit** — `git add -A && git commit -m "feat: scaffold Astro project with FR/EN i18n config"`

---

### Task 2: i18n helper, design system, Base layout, Header/Footer

**Files:** Create `src/i18n/index.ts`, `src/i18n/fr.json`, `src/i18n/en.json`, `src/styles/global.css`, `src/layouts/Base.astro`, `src/components/Header.astro`, `src/components/Footer.astro`

- [ ] **Step 1: `src/i18n/index.ts`**

```ts
import fr from './fr.json';
import en from './en.json';

export const languages = { fr: 'FR', en: 'EN' } as const;
export type Lang = keyof typeof languages;

const dict: Record<Lang, unknown> = { fr, en };

export function useTranslations(lang: Lang) {
  return function t(key: string): string {
    const v = key.split('.').reduce<any>((o, k) => (o ?? {})[k], dict[lang]);
    return typeof v === 'string' ? v : key;
  };
}

// Route slugs differ per language; keys are stable identifiers.
export const routes = {
  home: { fr: '', en: '' },
  lab: { fr: 'laboratoire', en: 'laboratory' },
  team: { fr: 'equipe', en: 'team' },
  research: { fr: 'recherche', en: 'research' },
  projects: { fr: 'projets', en: 'projects' },
  publications: { fr: 'publications', en: 'publications' },
  contact: { fr: 'contact', en: 'contact' },
} as const;
export type RouteKey = keyof typeof routes;

const BASE = (import.meta.env.BASE_URL.replace(/\/$/, ''));
export function href(lang: Lang, key: RouteKey): string {
  const slug = routes[key][lang];
  const prefix = lang === 'fr' ? '' : '/en';
  return `${BASE}${prefix}/${slug}` || '/';
}
```

- [ ] **Step 2: dictionaries with shared keys** (nav, footer, site meta). `fr.json` starts:

```json
{
  "site": {
    "name": "LIREI",
    "fullname": "Laboratoire d'innovation et de recherche en énergie intelligente",
    "uqtr": "Université du Québec à Trois-Rivières"
  },
  "nav": {
    "home": "Accueil", "lab": "Laboratoire", "team": "Équipe",
    "research": "Recherche", "projects": "Projets",
    "publications": "Publications", "contact": "Nous joindre"
  },
  "footer": {
    "affiliation": "Membre de l'Institut de recherche sur l'hydrogène (IRH)",
    "address": "3351, boul. des Forges, C.P. 500, Trois-Rivières (Québec) G9A 5H7, Canada",
    "rights": "Tous droits réservés."
  }
}
```

`en.json` mirrors with: fullname "Laboratory of Innovation and Research in Intelligent Energy", nav Home/Laboratory/Team/Research/Projects/Publications/Contact, footer "Member of the Hydrogen Research Institute (IRH)", "All rights reserved."

- [ ] **Step 3: `src/styles/global.css`** — design tokens and base rules:

```css
:root {
  --blue-900: #0d2b45; --blue-700: #14456e; --blue-100: #e8f0f7;
  --green-600: #0e8a5f; --green-100: #e6f4ee;
  --ink: #1a2330; --muted: #5a6b7d; --line: #dde5ec; --bg: #ffffff; --bg-alt: #f5f8fa;
  --serif: Georgia, 'Times New Roman', serif;
  --sans: system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif;
  --maxw: 68rem;
}
```

Rules: body in `--sans`; h1–h3 in `--serif` color `--blue-900`; `.container {max-width: var(--maxw); margin-inline:auto; padding-inline:1.25rem}`; `.hero` (blue-900 background, white text, green accent rule); `.card` (border 1px `--line`, radius 8px, padding); `.grid-2/.grid-3` responsive grids; `.tag` pill (green-100/green-600); `.btn` solid green; skip-link, focus-visible outlines; `@media (max-width: 720px)` collapses nav to wrapped row.

- [ ] **Step 4: `src/layouts/Base.astro`**

Props: `lang: Lang`, `routeKey: RouteKey`, `title: string`, `description: string`. Renders `<html lang={lang}>`, meta description, `<link rel="alternate" hreflang>` pair using `href(otherLang, routeKey)`, favicon, global.css import, `<Header lang routeKey/>`, `<slot/>` in `<main id="main">`, `<Footer lang/>`.

- [ ] **Step 5: `Header.astro`** — site name block (LIREI + fullname small), `<nav>` looping over route keys with `t('nav.'+key)`, `aria-current="page"` on active key, and lang switch link to `href(otherLang, routeKey)` labelled `EN`/`FR`.

- [ ] **Step 6: `Footer.astro`** — three columns: identity + affiliation, address + email `lirei@irh.ca` + phone `819 376-5011, poste 4457`, copyright with current year.

- [ ] **Step 7: Wire index.astro** to use Base (temporary hero only). Run `npm run build` — exit 0. Open preview and check FR page renders with header/footer.

- [ ] **Step 8: Commit** — `feat: add i18n system, design tokens and base layout`

---

### Task 3: Content collections + team data

**Files:** Create `src/content.config.ts`, `src/content/team/*.json` (18 files)

- [ ] **Step 1: `src/content.config.ts`**

```ts
import { defineCollection, z } from 'astro:content';
import { glob } from 'astro/loaders';

const bilingual = z.object({ fr: z.string(), en: z.string() });

const team = defineCollection({
  loader: glob({ pattern: '**/*.json', base: './src/content/team' }),
  schema: z.object({
    name: z.string(),
    category: z.enum(['direction', 'professor', 'staff', 'postdoc', 'student']),
    role: bilingual,
    topic: bilingual.optional(),
    affiliation: z.string().optional(),
    email: z.string().optional(),
    phone: z.string().optional(),
    order: z.number().default(100),
  }),
});

const publications = defineCollection({
  loader: glob({ pattern: '**/*.json', base: './src/content/publications' }),
  schema: z.object({
    title: z.string(),
    authors: z.array(z.string()),
    venue: z.string().optional(),
    year: z.number(),
    type: z.enum(['journal', 'conference', 'other']),
    doi: z.string().optional(),
    url: z.string().url().optional(),
  }),
});

const projects = defineCollection({
  loader: glob({ pattern: '**/*.md', base: './src/content/projects' }),
  schema: z.object({
    title: bilingual,
    summary: bilingual,
    axis: z.enum(['smartgrids', 'residential', 'ml', 'hydrogen', 'flexibility', 'ev']),
    status: z.enum(['active', 'completed']),
    partners: z.array(z.string()).default([]),
  }),
});

export const collections = { team, publications, projects };
```

- [ ] **Step 2: team JSON files.** Example `src/content/team/kodjo-agbossou.json`:

```json
{
  "name": "Kodjo Agbossou",
  "category": "direction",
  "role": { "fr": "Professeur titulaire et directeur du LIREI", "en": "Full Professor and Director of LIREI" },
  "topic": { "fr": "Énergies renouvelables, hydrogène et réseaux intelligents", "en": "Renewable energy, hydrogen and smart grids" },
  "affiliation": "UQTR — Génie électrique et génie informatique",
  "order": 1
}
```

Full roster (name — category — role/topic notes):
1. Kodjo Agbossou — direction — as above, order 1
2. Sousso Kelouwani — professor — Prof. titulaire, mobilité intelligente / intelligent mobility, order 10
3. Roland Malhamé — professor — Prof. (Polytechnique Montréal), commande et jeux à champ moyen / control and mean-field games, order 11
4. Yves Dubé — professor — Prof. associé, systèmes thermiques / thermal systems, order 12
5. Juan Oviedo — professor — Prof. associé, optimisation des systèmes énergétiques / energy systems optimization, order 13
6. Nilson Henao — staff — Agent de recherche / Research Agent, gestion énergétique résidentielle et apprentissage automatique / residential energy management and machine learning, email Nilson.Henao@uqtr.ca, phone "819 376-5011, poste 4457", order 20
7. David Toquica — postdoc — Chercheur postdoctoral / Postdoctoral Researcher, marchés d'énergie transactive / transactive energy markets, order 30
8. Farshad Etedadi — postdoc — coordination de charges résidentielles / residential load coordination, order 31
9. Juan Dominguez — postdoc — systèmes multi-agents appliqués à l'énergie / multi-agent energy systems, order 32
10. Daniel Galeano — student — Étudiant au doctorat / PhD Student, équité dans les mécanismes d'énergie transactive / equity in transactive energy mechanisms, order 40
11. Juan Caballero — student — détection et modélisation de l'occupation / occupancy detection and modeling, order 41
12. Ibrahim Moussa — student — problèmes inverses pour la réponse à la demande / inverse problems for demand response, order 42
13. Naren Mantilla — student — regroupement (clustering) pour les marchés de flexibilité / clustering for flexibility markets, order 43
14. Rasmane Bande — student — véhicules électriques comme source de flexibilité / EVs as grid flexibility providers, order 44
15. Landry Adjanohun — student — gestion énergétique des serres / greenhouse energy management, order 45
16. Michael Arenas — student — coordination multi-agents / multi-agent coordination, order 46
17. Amen Bakpo — student — ontologies pour l'énergie transactive / ontologies for transactive energy, order 47
18. Rihab Hanfi — student — infonuagique pour la gestion énergétique / cloud computing for energy management, order 48
19. Zahra Farahzadi — student — efficacité énergétique des serres / greenhouse energy efficiency, order 49
20. Jersson Garcia — student — décarbonation du secteur minier / mining sector decarbonization, order 50
21. Mourad Belkassemi — student — modèles économiques de l'énergie / energy economic models, order 51

Students' generic role: `{"fr": "Étudiant(e) aux cycles supérieurs", "en": "Graduate Student"}` unless noted.

- [ ] **Step 3: Verify** — `npm run build` exit 0 (Zod validates all JSON).

- [ ] **Step 4: Commit** — `feat: add content collections and team roster`

---

### Task 4: Seed real publications (Semantic Scholar)

**Files:** Create `src/content/publications/*.json` (target 25–40 entries)

- [ ] **Step 1: Fetch.** Using the semantic-scholar MCP tools in-session (or `curl https://api.semanticscholar.org/graph/v1/author/search?query=...` at ~1 req/s): find author IDs for **Kodjo Agbossou**, **Sousso Kelouwani**, **Nilson Henao**, **David Toquica** (verify UQTR affiliation in results), then fetch each author's papers with fields `title,year,venue,externalIds,publicationTypes,authors`.

- [ ] **Step 2: Filter & dedupe.** Keep years ≥ 2021; drop entries with no venue and no DOI; dedupe by DOI/normalized title; keep papers relevant to LIREI axes (energy/EV/smart grid/hydrogen — exclude off-topic). Cap at ~40 most recent.

- [ ] **Step 3: Write JSON files** named `YYYY-first-word-slug.json`, e.g.:

```json
{
  "title": "Optimal residential demand response using deep reinforcement learning",
  "authors": ["N. Henao", "K. Agbossou", "S. Kelouwani"],
  "venue": "IEEE Transactions on Smart Grid",
  "year": 2023,
  "type": "journal",
  "doi": "10.1109/TSG.2023.XXXXXXX"
}
```

`type` mapping: JournalArticle→journal, Conference→conference, else other.

- [ ] **Step 4: Verify** — `npm run build` exit 0.

- [ ] **Step 5: Commit** — `feat: seed publications catalog from Semantic Scholar`

---

### Task 5: Seed projects

**Files:** Create `src/content/projects/*.md` (6 files)

- [ ] **Step 1:** Six projects derived from the lab's known research lines, one per axis. Example `src/content/projects/energie-transactive.md`:

```md
---
title:
  fr: "Mécanismes d'énergie transactive pour les communautés"
  en: "Transactive energy mechanisms for communities"
summary:
  fr: "Conception de marchés locaux d'électricité équitables permettant aux prosommateurs d'échanger leur flexibilité énergétique."
  en: "Design of fair local electricity markets enabling prosumers to trade their energy flexibility."
axis: flexibility
status: active
partners: ["Hydro-Québec"]
---
```

The six: (1) transactive energy / flexibility (above); (2) residential energy management systems — axis `residential`, partner Hydro-Québec; (3) ML for consumption monitoring (NILM/occupancy) — axis `ml`; (4) EVs as grid flexibility — axis `ev`; (5) hydrogen & fuel-cell integration — axis `hydrogen`, partner IRH; (6) greenhouse & building smart-grid coordination — axis `smartgrids`, partner RNCan. Each with bilingual title/summary in the same style.

- [ ] **Step 2: Verify** — `npm run build` exit 0.
- [ ] **Step 3: Commit** — `feat: seed project catalog`

---

### Task 6: Home page

**Files:** Create `src/components/pages/Home.astro`; modify `src/pages/index.astro`; create `src/pages/en/index.astro`; extend both dictionaries with a `home` section.

- [ ] **Step 1: copy in `fr.json` under `home`:** hero title = site fullname; tagline "Des solutions pour une gestion intelligente de l'énergie"; intro paragraph (multidisciplinary team, member of IRH since 1999, optimizing energy use in buildings and vehicles, interest in hydrogen and renewables); stats labels (fondé en 1999 · membres de l'équipe · publications · axes de recherche); sections "Axes de recherche", "Publications récentes", links "Voir tout". EN mirrors ("Solutions for intelligent energy management", etc.).

- [ ] **Step 2: `Home.astro`** — props `{lang}`; hero (`.hero`) with title/tagline/CTA buttons (Recherche, Publications); stats strip computed from collections (`getCollection('team').length`, publications count); axes grid of 6 cards linking to research page anchors (`#smartgrids` …); latest 5 publications via `PublicationItem`; partner names row (Hydro-Québec · Ressources naturelles Canada · Mitacs · Université Laval · IRH).

- [ ] **Step 3: route files.** `src/pages/index.astro`:

```astro
---
import Home from '../components/pages/Home.astro';
---
<Home lang="fr" />
```

`src/pages/en/index.astro` same with `lang="en"`.

- [ ] **Step 4: Verify** — build exit 0; preview `/` (FR) and `/en/` (EN) render, lang switch works both ways.
- [ ] **Step 5: Commit** — `feat: add bilingual home page`

---

### Task 7: Laboratory + Research pages

**Files:** Create `src/components/pages/Lab.astro`, `src/components/pages/Research.astro`; routes `laboratoire.astro`, `recherche.astro`, `en/laboratory.astro`, `en/research.astro`; extend dictionaries (`lab`, `research` sections).

- [ ] **Step 1: `lab` copy.** History paragraph (IRH inaugurated 1996; LIREI created 1999 as multidisciplinary team within IRH; located at UQTR, Trois-Rivières; mention of new Pavillon Pellerin-Marmen research facilities). Infrastructure section: state-of-the-art equipment, dedicated workspaces, simulation platforms, residential energy lab benches. Environment section: inclusive, interdisciplinary collaboration, mentorship and publication opportunities; industry-funded partner network. FR and EN versions of each paragraph.

- [ ] **Step 2: `research` copy.** Intro + 6 axes, each with heading, 2–3 sentence description (FR/EN), keyword tags. Axes content: smart grids & simulation platforms; residential energy management & demand response; machine learning & consumption monitoring; hydrogen & fuel cells; flexibility markets & transactive energy (incl. convex optimization, mean-field control); electric vehicles & smart mobility.

- [ ] **Step 3: components.** `Lab.astro`: three `.card` sections. `Research.astro`: axis sections with `id={axisKey}` anchors (targets of home-page links), tags rendered as `.tag`.

- [ ] **Step 4: Verify** build + preview both locales. **Step 5: Commit** — `feat: add laboratory and research pages`

---

### Task 8: Team page

**Files:** Create `src/components/MemberCard.astro`, `src/components/pages/Team.astro`; routes `equipe.astro`, `en/team.astro`; dictionary `team` section (group headings: Direction; Professeurs / Professors; Professionnels de recherche / Research Staff; Stagiaires postdoctoraux / Postdoctoral Fellows; Étudiantes et étudiants / Graduate Students).

- [ ] **Step 1: `MemberCard.astro`** — props `{member, lang}`; renders initials avatar (SVG circle, blue-100 bg, blue-700 initials computed from name), name, `role[lang]`, `topic[lang]` muted, affiliation if ≠ UQTR default, mailto link if email.

- [ ] **Step 2: `Team.astro`** — `getCollection('team')`, group by category in order direction→professor→staff→postdoc→student, sort by `order`, grid `.grid-3` of cards under each localized heading.

- [ ] **Step 3: Verify** build + preview; all 21 members visible in both locales. **Step 4: Commit** — `feat: add team page`

---

### Task 9: Publications + Projects pages (with filters)

**Files:** Create `src/components/PublicationItem.astro`, `src/components/ProjectCard.astro`, `src/components/pages/Publications.astro`, `src/components/pages/Projects.astro`; routes ×4; dictionary `pubs` and `projects` sections (filter labels: Tous/All, Revue/Journal, Conférence/Conference, Autre/Other; year label; status labels En cours/Active, Terminé/Completed; axis names reused from `research`).

- [ ] **Step 1: `PublicationItem.astro`** — `<li>` with authors (joined ", "), title (bold; wrapped in `<a>` when DOI/URL — `https://doi.org/{doi}`), *venue* italic, year, `.tag` for type.

- [ ] **Step 2: `Publications.astro`** — entries sorted year desc, grouped by year with `<h2>` headers. Filter bar: `<select>` for type and year, each `<li>` carries `data-type`/`data-year`; inline `<script>` (vanilla, ~20 lines) hides non-matching items and empty year groups. Page works fully with JS disabled (filters simply inert).

- [ ] **Step 3: `ProjectCard.astro` + `Projects.astro`** — card with `title[lang]`, `summary[lang]`, axis tag (localized), status tag, partner names. Filter buttons by axis using same data-attribute pattern.

- [ ] **Step 4: Verify** — build; preview: filters narrow lists in both locales. **Step 5: Commit** — `feat: add publications and projects catalogs with filters`

---

### Task 10: Contact page + SEO polish

**Files:** Create `src/components/pages/Contact.astro`; routes `contact.astro`, `en/contact.astro`; dictionary `contact` section. Modify `Base.astro` if any meta gaps.

- [ ] **Step 1: copy.** Contact block (email lirei@irh.ca as mailto, phone 819 376-5011 poste/ext. 4457, address); "Partenaires / Partners" list (Hydro-Québec, Ressources naturelles Canada, Mitacs, Université Laval, IRH); recruitment paragraph FR/EN ("Le LIREI est toujours à la recherche d'étudiantes et d'étudiants motivés… / LIREI is always looking for motivated graduate students…") with mailto CTA button.

- [ ] **Step 2: SEO check** — every page passes unique `title`/`description` to Base; hreflang alternates present; `<meta name="robots" content="index,follow">`; og:title/og:description in Base.

- [ ] **Step 3: Verify** build. **Step 4: Commit** — `feat: add contact page and SEO metadata`

---

### Task 11: README + final verification

**Files:** Create `README.md`

- [ ] **Step 1: README** (bilingual header, body FR): project description; requirements (Node ≥ 20); commands (`npm install`, `npm run dev`, `npm run build`, `npm run preview`); deployment ("copy `dist/` to the web server"; note `BASE_PATH=/subpath npm run build` for subpath hosting); content how-tos: add a team member (JSON template), add a publication (JSON template), add a project (MD template) — include the actual templates.

- [ ] **Step 2: Final verification** — `npm run build` exit 0; `npm run preview` and click through all 7 pages × 2 locales; verify lang switcher round-trips on every page; verify all publication DOI links are well-formed.

- [ ] **Step 3: Commit** — `docs: add README with deployment and content guides`

---

## Self-Review Notes

- Spec coverage: all 7 pages ×2 locales (Tasks 6–10), collections & schemas (Task 3), Semantic Scholar seed (Task 4), projects (Task 5), placeholder avatars (Task 8), deployment docs + configurable base (Tasks 1, 11). ✔
- Type consistency: axis enum keys (`smartgrids residential ml hydrogen flexibility ev`) shared between projects schema, research page anchors and home links; `bilingual` object reused. ✔
- Out of scope confirmed: no CMS, no news section, no form backend. ✔
