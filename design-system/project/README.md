# Hege Christine — Design System

Et produksjonsklart designsystem for hegechristine.no og alt brand-arbeid rundt.

## TL;DR for Claude Code

Når du bygger nettsiden, importer disse i denne rekkefølgen:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Archivo:wght@400;500;600;700;800;900&family=Newsreader:ital,wght@0,400;0,500;1,400;1,500;1,600&family=JetBrains+Mono:wght@400;500;600&display=swap" rel="stylesheet">

<link rel="stylesheet" href="/assets/tokens.css">       <!-- 1. variabler -->
<link rel="stylesheet" href="/assets/marks.css">        <!-- 2. logo + corner marks -->
<link rel="stylesheet" href="/assets/decorations.css">  <!-- 3. grid bg, linjer, steps -->
<link rel="stylesheet" href="/assets/components.css">   <!-- 4. knapper, kort, skjema -->
<link rel="stylesheet" href="/assets/patterns.css">     <!-- 5. nav, hero, footer -->
```

## Filer

| Fil | Hva |
|---|---|
| `Brand Guidelines.html` | **Forsidefil** — print-vennlig oversikt. Send til samarbeidspartnere. |
| `Design System.html` | Levende referanse. Alle komponenter, alle states, alle eksempler. |
| `Canva Brand Board.html` | 1-side brand-ark optimert for Canva-import. |
| `Visuelle Elementer.html` | Mark-bibliotek: corner marks, grid-bakgrunner, progress. |
| `Deck.html` | Slide-deck-template. |
| `assets/tokens.css` | Farger, type, spacing, skygger — alle CSS-variabler. |
| `assets/marks.css` | `.hc-mark`, `.wordmark`, `.corner-mark`. |
| `assets/decorations.css` | `.bg-grid`, `.line`, `.steps`. |
| `assets/components.css` | `.btn`, `.card`, `.badge`, `.input`, `.accordion`. |
| `assets/patterns.css` | `.ds-nav`, `.hero-1/2/3`, `.cta-banner`, `.ds-footer`. |

## Brand-essensen

| | |
|---|---|
| **Logo** | HC-monogram (kvadrat, 1.5px outline, Archivo Black). Wordmark «Hege *Christine*» som sekundær. |
| **Fonter** | Archivo (400/500/700/800/900) · Newsreader Italic (500) · JetBrains Mono (600) |
| **Farger** | Ink `#2E3230` · Olive `#5E5F4C` · Sage `#AFBEA0` · Cream `#EFE6D4` · Sand `#F0D9A8` · Rust `#C5522C` |
| **Stemme** | Edit, don't add. Strategisk, varm, direkte. |

## Regler

1. **Ink, aldri sort.** Tekst er `var(--ink-600)`.
2. **Rust er retning** — kun primær handling, aksent-ord (1–2 i overskrifter), og hover på primary buttons.
3. **Newsreader Italic Rust** for aksenter i H1/H2 (sparsomt). **Newsreader Italic Ink** for sitater.
4. **Grid alltid synlig** — `.bg-grid` på seksjoner, lavt nivå (10% opacity).
5. **Skarpe hjørner** — `--radius-0` til `--radius-3` (0–8px). Pill kun for tags/badges.
6. **Mindre, mer presist** — hvitrom er innhold.

## Logo-bruk

```html
<!-- Primær -->
<div class="hc-mark hc-mark--lg hc-mark--ink"><span>HC</span></div>

<!-- På mørk flate -->
<div class="hc-mark hc-mark--lg hc-mark--cream"><span>HC</span></div>

<!-- Wordmark -->
<span class="wordmark">Hege<em>Christine</em></span>

<!-- Lockup (mark + wordmark) -->
<div class="lockup">
  <div class="hc-mark hc-mark--md hc-mark--ink"><span>HC</span></div>
  <span class="wordmark">Hege<em>Christine</em></span>
</div>
```

## Type-bruk

```html
<h1>Bygg en business du <em>faktisk</em> vil leve med.</h1>
<!-- Archivo Black 900 · em rendres som Newsreader Italic Rust automatisk -->

<p class="lede">Det handler ikke om mer. Det handler om mindre, mer presist.</p>
<!-- Newsreader Italic 500 -->
```

## Bakgrunner

```html
<section class="bg-grid">…</section>              <!-- standard -->
<section class="bg-grid bg-grid--fine">…</section> <!-- tett -->
<section class="bg-grid bg-grid--accent">…</section> <!-- med rust-linje -->
<section class="bg-dots">…</section>
<section class="on-dark bg-grid">…</section>      <!-- på mørk flate -->
```

## Knapper

```html
<a class="btn btn--primary">Book samtale</a>      <!-- Ink → Rust hover -->
<a class="btn btn--accent">Meld deg på</a>        <!-- Rust + 3px shadow hover -->
<a class="btn btn--secondary">Les mer</a>         <!-- Outline → Ink hover -->
<a class="btn btn--ghost">Til arkivet <span class="arrow">→</span></a>
```

## Versjonering

`v1.0` — Mai 2026. Live system. Endringer i `tokens.css` påvirker alt — endre forsiktig.
