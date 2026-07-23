# LIREI Website — Design Spec

**Date:** 2026-07-23
**Status:** Approved by user

## Goal

Academic/scientific bilingual (FR/EN) website for LIREI — Laboratoire d'innovation et de recherche en énergie intelligente, Université du Québec à Trois-Rivières (UQTR), part of the Institut de recherche sur l'hydrogène (IRH). Deployed as a static build to a self-hosted / UQTR server.

## Stack

- **Astro 5 + TypeScript**, fully static output (`dist/`), no UI framework.
- Custom CSS, sober academic design. Neutral scientific palette (deep blue / energy green accents), system-safe typography, responsive.
- **Astro built-in i18n**: French is the default locale served at `/`; English under `/en/`. Language switcher in the header. UI strings live in JSON dictionaries (`src/i18n/fr.json`, `src/i18n/en.json`); content entries carry per-locale fields (`title_fr` / `title_en`, etc.).
- No client-side JS except minimal enhancements (publication/project filters, mobile nav). Filters work via small vanilla JS; pages remain readable without JS.

## Pages (each in FR and EN)

| Route (FR / EN) | Content |
|---|---|
| `/` and `/en/` | Hero with mission, featured research axes, key figures (founded 1999, member counts), latest publications |
| `/laboratoire` / `/en/laboratory` | History (IRH 1996, LIREI 1999), infrastructure, facilities, work environment |
| `/equipe` / `/en/team` | Direction (Prof. Kodjo Agbossou), professors (Sousso Kelouwani, Roland Malhamé, Yves Dubé, Juan Oviedo), research staff (Nilson Henao), postdocs (David Toquica, Farshad Etedadi, Juan Dominguez), students with research topics |
| `/recherche` / `/en/research` | Research axes: smart grids, residential energy management, machine learning for energy, hydrogen & fuel cells, flexibility markets & transactive energy, electric vehicles |
| `/projets` / `/en/projects` | Project catalog, filterable by axis/status |
| `/publications` / `/en/publications` | Publication catalog seeded from Semantic Scholar, filterable by year/type, with DOI links |
| `/contact` / `/en/contact` | Contact info (lirei@irh.ca, address, phone), partners (Hydro-Québec, NRCan, Mitacs, Université Laval), recruitment note |

## Content collections

Adding content = adding a file; schemas typed with Zod via Astro content collections.

- `src/content/team/*.json` — one file per member: name, role (fr/en), category (direction/professor/staff/postdoc/student), research topic (fr/en), optional email/phone/links.
- `src/content/publications/*.json` — title, authors[], venue, year, type (journal/conference/other), DOI/URL.
- `src/content/projects/*.md` — frontmatter with bilingual title/summary, axis, status, partners; body optional.

## Data

- **Publications:** seeded with real publications of the team (Agbossou, Kelouwani, Henao, Toquica…) fetched via Semantic Scholar, last ~5 years, deduplicated; user curates afterwards.
- **Team:** full roster from the current UQTR LIREI site (site 7478).
- **Photos:** initial-based placeholder avatars; real photos added later by the lab.
- Logos of partners: text-only mentions (no unlicensed logo files).

## Deployment

- `npm run build` → static `dist/` copied to any web server. No server runtime required.
- `astro.config` keeps `base` configurable in case the UQTR server serves under a subpath.
- README documents: install, dev, build, deploy, and how to add team members/publications/projects.

## Verification

- `npm run build` must pass (also serves as type/schema check for content collections).
- Manual smoke check of both locales via `npm run preview`.

## Out of scope

- CMS/admin interface, search engine, news/blog section (can be added later), contact form backend (mailto link only).
