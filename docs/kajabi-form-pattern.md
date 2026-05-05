# Kajabi Form Integration for hegechristine.no

Pattern for å bygge custom-stylede skjemaer som leverer påmeldinger til Kajabi-lister, uten Zapier, uten API-tilgang, uten tredjepartstjenester.

---

## Kort oppsummert

hegechristine.no bruker Kajabi som backend for e-postlister og automatikker, men nettsidene ligger utenfor Kajabi (Vercel/GitHub eller annet). Kajabi sin standard form-embed ser ikke bra ut og er vanskelig å style. API-en krever Pro-plan. Direct POST avvises pga CSRF-token (`authenticity_token`).

**Løsningen:** Last Kajabi sin embed-skript skjult, vis et custom skjema synlig, broadcast verdier mellom dem ved submit. Brukeren ser kun det vakre skjemaet. Kajabi får submission med riktig token og legger personen i listen.

---

## Når denne patternen brukes

Alle nettside-skjemaer som skal levere til Kajabi:

- Nyhetsbrev-påmelding
- Lead magnet (gratis PDF, guide, masterclass)
- Webinar-registrering
- Waitlist-påmelding
- Kontakt-skjema (hvis Kajabi-listen brukes til kontakter)

---

## Arkitektur

```
[Bruker] → fyller ut → [Synlig custom skjema]
                              ↓ submit
                       [JavaScript bro]
                              ↓ kopierer verdier
                       [Skjult Kajabi-embed med authenticity_token]
                              ↓ submit til Kajabi
                       [Skjult iframe fanger response]
                              ↓
                       [Kajabi-liste mottar person]
                              ↓
                  [Kajabi-automatikk kjører som vanlig]
```

**Hvorfor det fungerer:** Vi bruker Kajabi sin **ekte** form (med ekte authenticity_token), bare visuelt skjult. Vår JavaScript fyller ut Kajabi-skjemaet med brukerens data og submitter det. CSRF-validering passerer fordi tokenet er ekte.

---

## Setup: Legge til skjema på en ny side

### Steg 1: Finn Kajabi form embed-URL

Hver Kajabi form har en unik embed-URL. Pattern:
```
https://www.hegechristine.no/forms/[FORM_ID]/embed.js
```

For å finne FORM_ID: Logg inn i Kajabi → Marketing → Forms → klikk skjemaet → "Embed" → kopier URL.

Aktive skjemaer (per nov 2026):
- `349307` — Nyhetsbrev (hovedliste)
- _Legg til flere her etter hvert som de opprettes_

### Steg 2: Legg HTML-strukturen på sida

```html
<form id="newsletter-form" class="hc-kajabi-form"
      data-kajabi-form-id="349307">
  <input type="text" name="name" placeholder="Navn" required autocomplete="name">
  <input type="email" name="email" placeholder="E-post" required autocomplete="email">
  <button type="submit">Ja, takk! →</button>
  <div class="form-message" data-form-message></div>
</form>

<!-- Skjult Kajabi-embed (genererer authenticity_token) -->
<div class="kajabi-hidden-wrap" aria-hidden="true">
  <script src="https://www.hegechristine.no/forms/349307/embed.js"></script>
</div>

<!-- Skjult iframe for response -->
<iframe name="kajabi-response-frame" class="kajabi-response-frame" aria-hidden="true"></iframe>
```

### Steg 3: Konfigurer for use case

**Newsletter (inline success-melding):**
```html
<form id="newsletter-form" class="hc-kajabi-form">
  <!-- ingen ekstra attributter -->
</form>
```

**Lead magnet (redirect til takkeside):**
```html
<form id="newsletter-form" class="hc-kajabi-form"
      data-redirect-url="/takk-claude-guiden">
  <!-- ... -->
</form>
```

**Custom success-melding:**
```html
<form id="newsletter-form" class="hc-kajabi-form"
      data-success-message="✓ Sjekk innboksen din!">
  <!-- ... -->
</form>
```

---

## Attribute-referanse

| Attributt | Type | Default | Beskrivelse |
|-----------|------|---------|-------------|
| `data-kajabi-form-id` | string | — | Kajabi form ID (info, ikke brukt av script) |
| `data-redirect-url` | URL | — | Hvis satt, redirector hit etter submit i stedet for å vise melding |
| `data-success-message` | string | "✓ Takk! Du er på listen." | Inline melding ved success (overstyrer default) |

---

## Komponentkode

### CSS

Skjult Kajabi-embed og iframe må gjemmes uten å fjernes (de må fortsatt fungere):

```css
.kajabi-hidden-wrap {
  position: absolute;
  left: -9999px;
  top: -9999px;
  visibility: hidden;
  pointer-events: none;
}

.kajabi-response-frame {
  display: none;
}
```

Selve skjema-stylingen er opp til prosjektet — patternen krever bare at strukturen og oppførselen fungerer. Strukturell CSS du må/bør ha:

```css
.hc-kajabi-form {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 9px;
}

.hc-kajabi-form input[type="text"],
.hc-kajabi-form input[type="email"] {
  width: 100%;
  box-sizing: border-box;
}

.hc-kajabi-form button {
  width: 100%;
  cursor: pointer;
  grid-column: 1 / -1;
}

.hc-kajabi-form button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.hc-kajabi-form [data-form-message] {
  grid-column: 1 / -1;
  min-height: 1em;
}

@media (max-width: 480px) {
  .hc-kajabi-form {
    grid-template-columns: 1fr;
  }
}
```

Farger, fonter, padding, border-radius, hover-states og lignende styles legges på etter prosjektets eget designsystem.

### JavaScript

Generisk skript som finner alle skjemaer med `class="hc-kajabi-form"` og wirer dem opp:

```javascript
(function() {
  const forms = document.querySelectorAll('.hc-kajabi-form');
  forms.forEach(initKajabiForm);

  function initKajabiForm(customForm) {
    const button = customForm.querySelector('button[type="submit"]');
    const message = customForm.querySelector('[data-form-message]');
    const originalButtonText = button.textContent;
    const redirectUrl = customForm.getAttribute('data-redirect-url');
    const successMessage = customForm.getAttribute('data-success-message')
      || '✓ Takk! Du er på listen.';

    function setMessage(text, state) {
      if (!message) return;
      message.textContent = text;
      // state: 'success' | 'error' | '' (default)
      message.setAttribute('data-state', state || '');
    }

    function findKajabiForm() {
      return document.querySelector('#kajabi-form form, .kajabi-hidden-wrap form');
    }

    customForm.addEventListener('submit', function(e) {
      e.preventDefault();

      const kajabiForm = findKajabiForm();

      if (!kajabiForm) {
        setMessage('Skjemaet laster fortsatt — prøv igjen om et øyeblikk.', 'error');
        return;
      }

      button.disabled = true;
      button.textContent = 'Sender...';
      setMessage('');

      const userName = customForm.querySelector('input[name="name"]').value.trim();
      const userEmail = customForm.querySelector('input[name="email"]').value.trim();

      // Fyll Kajabi-skjemaet
      const emailInput = kajabiForm.querySelector('input[type="email"]');
      if (emailInput) emailInput.value = userEmail;

      const textInputs = kajabiForm.querySelectorAll('input[type="text"]');
      for (let i = 0; i < textInputs.length; i++) {
        const inp = textInputs[i];
        if (inp.name && inp.name.toLowerCase().indexOf('name') !== -1) {
          inp.value = userName;
          break;
        }
      }

      kajabiForm.target = 'kajabi-response-frame';

      try {
        kajabiForm.submit();

        setTimeout(function() {
          if (redirectUrl) {
            window.location.href = redirectUrl;
          } else {
            setMessage(successMessage, 'success');
            customForm.reset();
            button.disabled = false;
            button.textContent = originalButtonText;
          }
        }, 800);
      } catch (err) {
        setMessage('Noe gikk galt. Prøv igjen om litt.', 'error');
        button.disabled = false;
        button.textContent = originalButtonText;
      }
    });
  }
})();
```

Meldingens utseende styres via CSS basert på `data-state`:

```css
.hc-kajabi-form [data-form-message][data-state="success"] { /* prosjektets success-styling */ }
.hc-kajabi-form [data-form-message][data-state="error"] { /* prosjektets error-styling */ }
```

Dette kan ligge inline på sida, eller (bedre) i en separat fil `/js/kajabi-form.js` som lastes på alle sider som har skjema.

---

## Testing

**Lokal testing fungerer ikke** — JavaScript-en POSTer til `www.hegechristine.no` som blokkeres fra `localhost`. Test på live URL.

### Sjekkliste

1. Last opp sida til hosting (Vercel preview deploy fungerer fint)
2. Åpne sida → fyll ut skjema → klikk submit
3. Bekreft visuell feedback:
   - Knappen viser "Sender..."
   - Etter ~0.8s: success-melding **eller** redirect til takkeside
4. Logg inn i Kajabi → Marketing → Forms → [valgt form] → Submissions
5. Bekreft at testpåmeldingen dukker opp
6. Sjekk at e-postautomatikk kjører (test-e-post bør komme i innboksen)

### Hvis det ikke fungerer

Åpne DevTools → Console og se etter feilmeldinger. Vanlige problemer:

- **"Skjemaet laster fortsatt..."** → Kajabi embed-script har ikke lastet enda. Sjekk at `<script src="...embed.js">` er på sida og ikke blokkert.
- **Submission går igjennom men ingen havner i Kajabi** → Sjekk at navne-feltet i Kajabi-skjemaet matcher antakelsen i scriptet (text-input med `name` som inneholder "name"). Hvis Kajabi-skjemaet bruker `first_name`, juster scriptets matching.
- **CORS-feil i console** → Skjemaet sendes men kunne ikke vise response. Det er ofte OK siden vi bruker iframe-target. Men bekreft i Kajabi at submission havnet riktig.

---

## Use cases (eksempler)

### Use case 1: Nyhetsbrev (lenker.html, footer på alle sider)

```html
<form id="newsletter-form" class="hc-kajabi-form">
  <input type="text" name="name" placeholder="Navn" required>
  <input type="email" name="email" placeholder="E-post" required>
  <button type="submit">Ja, takk! →</button>
  <div class="form-message" data-form-message></div>
</form>
```

Bruker Kajabi form 349307. Inline success-melding etter submit. Ingen redirect.

### Use case 2: Lead magnet (Claude-guiden)

På `hegechristine.no/claude-guiden`:

```html
<form id="freebie-form" class="hc-kajabi-form"
      data-redirect-url="/takk-claude-guiden">
  <input type="text" name="name" placeholder="Navn" required>
  <input type="email" name="email" placeholder="E-post" required>
  <button type="submit">Send meg guiden →</button>
  <div class="form-message" data-form-message></div>
</form>
```

Bruker dedikert Kajabi form for lead magnet (eget form-ID). Redirector til `/takk-claude-guiden` ved success. Kajabi-automatikk sender PDF på e-post automatisk.

### Use case 3: Flere skjemaer på samme side (waitlist + nyhetsbrev)

Pattern er ikke designet for **flere skjemaer fra ulike Kajabi-lister på samme side**. Hvis det trengs senere, må scriptet utvides til å håndtere flere `kajabi-hidden-wrap`-instances per skjema. Foreløpig: én Kajabi-liste per side.

---

## Hosting og deployment

Skjema-patternen fungerer både same-origin (hostet på hegechristine.no) og cross-origin (hostet på Vercel/Netlify med custom domene). Anbefalt setup:

- `hegechristine.no` → Vercel/GitHub Pages (statisk site, dette repo)
- `kurs.hegechristine.no` → Kajabi (kurs, login, checkout)
- `hegechristine.mykajabi.com` → Kajabi backup + filserver (fonter, bilder, evt. legacy-sider)

Skjema-submissions går alltid til `www.hegechristine.no/forms/.../form_submissions` uavhengig av hvor sida er hostet.

---

## Vedlikehold

**Hva kan brekke:**
- Hvis Kajabi endrer form-strukturen sin (sjelden, men mulig). Da må scriptets feltsøk justeres.
- Hvis Kajabi flytter authenticity_token til cookies istedenfor hidden input. Da må vi tenke nytt.

**Hva som er stabilt:**
- Action URL-formatet (`/forms/[ID]/form_submissions`) har vært stabilt i mange år
- HTML form med authenticity_token er Rails-standard, endres ikke uten varsel
- iframe-target for å fange response er rent HTML-pattern, fungerer alltid

Hvis Kajabi-strukturen endres en gang i fremtiden, ta opp DevTools → Network → submit en testform → sjekk hva som har endret seg → juster scriptet. Bør ta 15-30 minutter.

---

## Notater og fremtidige forbedringer

- **Loading state:** Knappen viser "Sender..." i 800ms før success vises. Kan justeres.
- **Validering:** Kun HTML5-validering p.t. Kan utvides med Parsley eller egen validering hvis det blir behov.
- **Analytics:** Submit-event kan hookes opp til Plausible/Google Analytics for konverteringssporing. Eksempel:
  ```javascript
  if (window.plausible) {
    window.plausible('Newsletter signup');
  }
  ```
- **Honeypot/anti-spam:** Kajabi har innebygd spam-beskyttelse. Hvis spam blir et problem, legg til honeypot-felt eller reCAPTCHA.
- **Multi-form på samme side:** Krever utvidelse av scriptet hvis det blir behov.

---

## Referanse

- Pattern utviklet og bevist mai 2026 i samtaler med Hege Christine
- Originalkode i `lenker/index.html`
- Tested mot Kajabi form 349307 (nyhetsbrev-hovedliste)
