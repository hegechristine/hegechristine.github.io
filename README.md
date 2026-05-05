# hegechristine.github.io

Personlig nettside for Hege Christine. Ren HTML/CSS/JS hostet på GitHub Pages.

**Live:** https://hegechristine.github.io/

## Sider

| URL | Hva |
|---|---|
| `/` | Forside (placeholder enn så lenge) |
| `/lenker/` | Linktree-erstatning. Profil + lenker til podcast, communities, kontakt + nyhetsbrev-skjema |

## Repo-struktur

```
.
├── index.html                    # Forside
├── favicon.svg                   # HC-monogram
├── lenker/
│   └── index.html                # Linktree-side (referanse-implementering av brand)
├── docs/
│   └── kajabi-form-pattern.md    # Hvordan skjemaer kobles til Kajabi-lister
├── design-system/                # Komplett brand-system fra claude.ai/design (referanse, delvis brukt)
│   ├── README.md
│   └── project/
│       ├── README.md
│       ├── assets/               # tokens.css, marks.css, components.css, patterns.css osv.
│       └── *.html                # Brand Guidelines, Visuelle Elementer, Deck osv.
└── README.md                     # Denne filen
```

## Brand-essensen

| | |
|---|---|
| **Logo** | HC-monogram (kvadrat, Archivo Black, rust-fyll eller ink-outline) |
| **Fonter** | Archivo · Newsreader Italic · JetBrains Mono |
| **Farger** | Ink `#2E3230` · Olive `#5E5F4C` · Sage `#AFBEA0` · Cream `#EFE6D4` · Sand `#F0D9A8` · Rust `#C5522C` |

Full referanse: `design-system/project/README.md` og `design-system/project/Brand Guidelines.html`.

## Når du bygger en ny side

Sjekkliste for konsistens:

1. `<link rel="icon" href="/favicon.svg" type="image/svg+xml">` i `<head>`
2. Fonter via Google Fonts: Archivo + Newsreader + JetBrains Mono (se `lenker/index.html` for eksakt link)
3. Bruk brand-fargene fra tabellen over som CSS-variabler
4. HC-monogram der profil/avatar/logo passer (kvadrat, ikke sirkel)
5. Italic-aksenter med Newsreader Italic Rust på 1–2 ord per overskrift (sparsomt)
6. Kicker-tekst i JetBrains Mono, uppercase, letter-spacing ~0.14em

`lenker/index.html` er den nåværende referanse-implementasjonen. Kopier og tilpass.

## Skjemaer som leverer til Kajabi

Bruk patternen i [docs/kajabi-form-pattern.md](docs/kajabi-form-pattern.md). Aktive form-IDer:

- `349307` — Nyhetsbrev (hovedliste)

## Workflow (flere maskiner)

```bash
git pull        # før du jobber
# ...endre filer...
git add -A && git commit -m "..." && git push
```

GitHub Pages bygger automatisk fra `main`. Endringer er live ~30-60 sek etter push.

## To GitHub-kontoer

Repoet eies av `hegechristine`-brukeren (egen konto for personlig brand). Hege bruker også `hegechris`-kontoen for aigree-prosjekter. Per-repo credential pinning er satt opp så pushes funker uavhengig av hvilken konto som er aktiv i `gh auth`.

## Custom domene

Planlagt: koble `hegechristine.no` til denne siden senere. Krever migrasjon fra Kajabi (som nå hoster rotdomenet) — se DNS-status i Proisp før det gjøres.
