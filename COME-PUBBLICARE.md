# Pubblicare le pagine legali — tutorial da terminale

Obiettivo: ottenere un indirizzo web pubblico per la privacy policy, da
incollare nella scheda App Store e in quella di Google Play. Senza, non puoi
sottomettere su nessuno dei due.

Tempo richiesto: 10-15 minuti.

---

## Prima di tutto: perché conviene usare un sottodominio di vibeschool.it

Se dai agli store un indirizzo `github.io`, quell'indirizzo resta legato al tuo
account GitHub personale. Il giorno in cui trasferirai il progetto a Simone
dovrai cambiare l'URL in entrambe le schede, e ogni modifica alla scheda
comporta un nuovo passaggio di revisione.

Se invece dai `legal.vibeschool.it`, l'indirizzo è tuo per sempre e puoi
spostare l'hosting sotto qualunque account, in qualunque momento, senza toccare
nulla sugli store. Il passaggio in più è un solo record DNS.

Il PASSO 6 fa esattamente questo. È facoltativo ma consigliato.

---

## PASSO 0 — Compilare il nome del titolare

Apri `privacy.html` e sostituisci `[NOME E COGNOME]` con il nome concordato
con Simone. Poi cancella il blocco `<div class="todo"> ... </div>`, che è solo
un promemoria e non deve finire online.

```bash
open -e /Users/diegomarini/vibe-school/legal-site/privacy.html
```

---

## PASSO 1 — Installare e autenticare GitHub CLI

Serve a creare il repository e attivare le pagine senza aprire il browser.

```bash
gh --version
```

Se risponde "command not found":

```bash
brew install gh
```

Poi autenticati (una volta sola, in assoluto):

```bash
gh auth login
```

Rispondi: `GitHub.com` → `HTTPS` → `Y` per autenticare Git → `Login with a web
browser`. Copia il codice che compare, premi Invio, incollalo nella pagina che
si apre.

---

## PASSO 2 — Preparare il repository locale

```bash
cd /Users/diegomarini/vibe-school/legal-site
git init
git add .
git commit -m "Pagine legali pubbliche di Vibe School"
git branch -M main
```

---

## PASSO 3 — Creare il repository su GitHub e caricarlo

Un comando solo: crea il repository pubblico, collega il remote e carica tutto.

```bash
gh repo create vibe-school-legal --public --source=. --remote=origin --push
```

> Deve essere pubblico: GitHub Pages sui piani gratuiti non pubblica da
> repository privati. Qui dentro c'è solo HTML, nessuna chiave e nessun codice
> dell'app, quindi non è un problema. Il repository dell'app resta privato.

---

## PASSO 4 — Attivare GitHub Pages

```bash
gh api -X POST repos/{owner}/vibe-school-legal/pages \
  --input - <<'JSON'
{"source":{"branch":"main","path":"/"}}
JSON
```

`{owner}` viene risolto automaticamente da gh. Se il comando dovesse dare
errore, l'equivalente via browser è: repository → **Settings** → **Pages** →
Source `Deploy from a branch`, branch `main`, cartella `/ (root)` → **Save**.

---

## PASSO 5 — Verificare che sia online

Attendi un paio di minuti, poi:

```bash
curl -sI https://$(gh api user --jq .login).github.io/vibe-school-legal/privacy.html | head -1
```

Deve rispondere `HTTP/2 200`. Se risponde `404`, aspetta ancora un minuto e
riprova: la prima pubblicazione richiede qualche istante.

Poi aprila davvero, per vedere che sia leggibile:

```bash
open https://$(gh api user --jq .login).github.io/vibe-school-legal/privacy.html
```

A questo punto **hai già un URL valido** da dare agli store. I passi successivi
servono solo a renderlo permanente.

---

## PASSO 6 — Sottodominio legal.vibeschool.it (consigliato)

### 6a. Dichiara il dominio nel repository

```bash
cd /Users/diegomarini/vibe-school/legal-site
echo "legal.vibeschool.it" > CNAME
git add CNAME
git commit -m "Dominio personalizzato"
git push
```

### 6b. Aggiungi il record DNS

Nel pannello DNS del dominio `vibeschool.it` — lo stesso dove hai messo i
record MX di Zoho — crea un record:

| Campo | Valore |
|---|---|
| Tipo | `CNAME` |
| Nome / Host | `legal` |
| Valore / Punta a | `<tuo-username-github>.github.io` |
| TTL | automatico |

Per sapere il nome esatto da inserire:

```bash
echo "$(gh api user --jq .login).github.io"
```

### 6c. Verifica la propagazione

Il DNS può metterci da pochi minuti a qualche ora.

```bash
dig +short legal.vibeschool.it
```

Quando risponde con l'indirizzo github.io, controlla che la pagina risponda:

```bash
curl -sI https://legal.vibeschool.it/privacy.html | head -1
```

### 6d. Attiva HTTPS

Nel repository su GitHub: **Settings** → **Pages** → spunta **Enforce HTTPS**.
Compare solo dopo che il certificato è stato emesso, di solito entro un'ora dal
momento in cui il DNS è corretto.

---

## Gli URL finali da usare negli store

```
https://legal.vibeschool.it/privacy.html      → privacy policy (Apple e Google)
https://legal.vibeschool.it/termini.html      → termini di servizio
https://legal.vibeschool.it/                  → pagina di supporto
```

Senza sottodominio, sostituisci `legal.vibeschool.it` con
`<tuo-username>.github.io/vibe-school-legal`.

**Prova ogni URL in una finestra di navigazione privata prima di incollarlo
negli store.** Se non è raggiungibile pubblicamente, la submission viene
respinta.

---

## Aggiornare i testi in futuro

```bash
cd /Users/diegomarini/vibe-school/legal-site
# modifica i file, poi:
git add .
git commit -m "Aggiornamento informativa"
git push
```

Online entro un minuto. Ricordati di aggiornare la data "Ultimo aggiornamento"
in cima alla pagina modificata.

---

## Quando passerete il progetto a Simone

Il repository si trasferisce dalle impostazioni, in tre clic: **Settings** →
**Danger Zone** → **Transfer ownership**. Nessuno perde nulla e, se hai
configurato il sottodominio, gli URL negli store non cambiano affatto.

---

## Da non dimenticare

Le schermate legali dentro l'app (`app/legal/privacy.tsx` e
`app/legal/terms.tsx`) parlano ancora di "beta chiusa" e vanno allineate a
questi testi. Due versioni diverse dello stesso documento sono il genere di
incoerenza che un reviewer nota.
