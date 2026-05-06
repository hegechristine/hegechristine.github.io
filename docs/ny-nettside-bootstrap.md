# Bootstrap: ny nettside

Sjekkliste og paste-ready prompt for å sette opp en ny statisk nettside (GitHub Pages) i en ny Claude-session, basert på erfaringen fra `hegechristine.github.io`.

---

## Bestem deg for før du starter

| | |
|---|---|
| **GitHub-konto** | Skal siden hostes under `hegechristine`, `hegechris`, eller en helt ny konto? |
| **Repo-navn** | For GitHub Pages bør repo-navnet være `<username>.github.io` for å bli hovedside. Ellers blir det `<username>.github.io/<reponavn>` (project page) |
| **Domene** | Eget domene (f.eks. kursnavn.no), eller bare github.io-URL? |
| **Type side** | Linktree-stil, landing-page, blog, lead-magnet, kurs-presentasjon? |
| **Skjemaer** | Skal noe levere til Kajabi? (Kontakt, nyhetsbrev, lead magnet) |
| **Brand** | Skal siden bruke samme brand som hegechristine.no (HC-monogram, designsystem), eller egen identitet? |

---

## Det du kan gjenbruke fra dette repoet

| Ressurs | Hvor | Når du trenger det |
|---|---|---|
| **HC-monogram favicon** | `/favicon.svg` | Hvis siden er HC-brand |
| **Designsystem (tokens, marks, components, patterns)** | `/design-system/project/` | Hvis siden er HC-brand |
| **Kajabi form-pattern** | `/docs/kajabi-form-pattern.md` | Alltid hvis du vil ha skjema → Kajabi |
| **Linktree-side referanse** | `/lenker/index.html` | Layout, kort-stiler, flip-animasjon, kontakt-modal |
| **Repo-struktur og README-mal** | `/README.md` (denne repo) | Som mal for ny repo |

---

## Steg-for-steg setup

### 1. Avgjør konto og repo-navn

```bash
# Sjekk hvilken gh-konto som er aktiv
gh auth status

# Bytt om nødvendig
gh auth switch
```

### 2. Opprett lokalt og remote

```bash
mkdir -p ~/Documents/GitHub/<reponavn>
cd ~/Documents/GitHub/<reponavn>
git init -b main
git config user.name "Hege Christine"
git config user.email "<riktig epost for kontoen>"
git config credential.https://github.com.username <gh-brukernavn>

gh repo create <reponavn> --public --source=. --remote=origin
```

### 3. Initial fil-struktur

```
.
├── index.html
├── favicon.svg          # Kopier fra hegechristine.github.io hvis HC-brand
├── README.md
└── docs/                # Hvis prosjektet trenger dokumentasjon
```

### 4. Aktiver GitHub Pages

For `<username>.github.io`-repoer er det automatisk. For project-pages:
```bash
gh api -X POST repos/<owner>/<repo>/pages -f "source[branch]=main" -f "source[path]=/"
```

### 5. (Valgfritt) Custom domene

Etter Pages er aktivert:
```bash
gh api -X PUT repos/<owner>/<repo>/pages -f cname=<domene.no>
```
+ DNS-oppsett hos registrar (CNAME `<sub>` → `<username>.github.io` for subdomene, eller A-records til GitHub Pages IPs for rotdomene).

---

## Paste-ready kickstart-prompt for ny Claude-session

Lim inn dette i en ny Claude-session (i en ny mappe, før du har laget noe):

```
Jeg skal sette opp en ny statisk nettside hostet på GitHub Pages.

KONTEKST OM MEG (samme som hegechristine.github.io-prosjektet):
- Jobber fra ~/Documents/GitHub/ (ikke i Drive — gh repos må aldri ligge i Drive pga sync-konflikter med .git/)
- To GitHub-kontoer: hegechris (aigree) og hegechristine (personlig brand). Jeg vil bruke [VELG: hegechristine / hegechris / NY]
- Bruker Kajabi som backend for e-postlister og kurs
- Bruker Proisp som domene-registrar (DNS administreres der)
- E-post via Google Workspace (kritisk: må ikke knekke @hegechristine.no-sending via Kajabi/Mailgun)

OM DEN NYE SIDEN:
- Repo-navn: [SETT INN]
- Hva siden er for: [BESKRIV — landing page, blog, lead magnet, kurs-side osv.]
- Domene: [github.io-URL ELLER eget domene]
- Skal bruke HC-brand (designsystem fra hegechristine.github.io)? [JA/NEI]
- Skjemaer som skal kobles til Kajabi? [LIST OPP, eller NEI]
- Spesielle krav: [BESKRIV om noe]

REFERANSE:
Hvis du trenger å se hvordan jeg setter opp ting, har jeg `~/Documents/GitHub/hegechristine.github.io/` med:
- README.md (oversikt + sjekkliste for ny side)
- docs/kajabi-form-pattern.md (form → Kajabi-pattern)
- docs/ny-nettside-bootstrap.md (denne guiden)
- design-system/ (komplett brand-system)
- favicon.svg (HC-monogram)
- lenker/index.html (referanse-implementering: kort, flip-animasjon, kontakt-modal)

Du kan gjerne lese disse filene før du foreslår hvordan vi gjør det.

Foreslå et oppsett, så går vi gjennom det sammen før vi begynner å bygge.
```

---

## Vanlige fallgruver

- ❌ Aldri opprett `.git`-repo i en Drive-mappe (`~/Library/CloudStorage/...`)
- ❌ Ikke endre DNS-records på rotdomenet hegechristine.no uten plan — kan knekke e-post via Kajabi/Mailgun
- ❌ Ikke kopier favicon.svg eller designsystemet hvis det er en *ikke*-HC-brand-side
- ✅ Sjekk alltid `gh auth status` før push første gang i ny repo — ellers ender du med å pushe under feil bruker
- ✅ Etter `gh repo create`, sett `git config credential.https://github.com.username <bruker>` så credential helper plukker riktig token
