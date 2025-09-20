# penetration-test-tscloud
Questo report documenta i test di sicurezza effettuati sul sito di test.  Obiettivo: verificare sicurezza e logica dell’applicazione.
## 1. Introduzione
Questo report documenta i test di sicurezza effettuati sul sito di test ospitato su Vercel.  
**Obiettivo:** verificare sicurezza e logica dell’applicazione Next.js/TypeScript.

---

## 2. Ambiente
- **Sistema operativo:** Kali Linux
- **Strumenti:** Burp Suite Community Edition, Browser con DevTools
- **Applicazione:** Next.js / TypeScript
- **Porte rilevate:** 443 (HTTPS), 8080 (filtrata)

---

## 3. Test Effettuati

| # | Campo / Endpoint               | Tipo di test          | Input / Payload                               | Risultato / Note |
|---|--------------------------------|----------------------|------------------------------------------------|--------------------------------------|
| 1 | Login (`/api/login`)           | Enumerazione nid     | Nid validi e sconosciuti, password `admin`     | `"Password non corretta"` → nid esiste; `Codice non valido`→ nid inesistente |
|---|--------------------------------|----------------------|------------------------------------------------|--------------------------------------|
| 2 | Motivo ferie (`/api/ferie`)    | XSS / Caratteri speciali | `<script>alert('Test')</script>`             | Sanitizzazione attiva, alert non eseguito                             |
|---|--------------------------------|----------------------|------------------------------------------------|--------------------------------------|
| 3 | Nome utente / Bio / Descrizione| XSS / Input speciale | `<b>bold</b>`, `<img src=x onerror=alert(1)>` | Testato, output escapato → sicuro |
|---|--------------------------------|----------------------|------------------------------------------------|--------------------------------------|
| 4 | Modifica ferie                 | CSRF / Session test  | Richiesta POST senza cookie/sessione          | Fallisce senza sessione valida → protezione attiva                    |
|---|--------------------------------|----------------------|------------------------------------------------|--------------------------------------|
| 5 | Visualizzazione ferie / profilo| Access control       | Modifica ID risorsa `/api/ferie/ID`          | Status code corretto (403/401) per utenti non autorizzati                      |
|---|--------------------------------|----------------------|------------------------------------------------|--------------------------------------|
| 6 | Cookie / Session               | Manipolazione cookie | Modifica cookie di sessione                   | Azioni bloccate se cookie non valido |

---

## 4. Screenshot

![Login Test]()  
![Burp Intercept]()  
![NID Enumeration]()  

---

## 5. Risultati

- **XSS:** tutti i payload testati vengono sanitizzati → nessuna vulnerabilità XSS Stored riflessa.  
- **Enumerazione NID:** possibile distinguere NID esistenti e non → vulnerabilità logica da considerare in ambiente interno/test.  
- **CSRF:** protezione corretta verificata tramite richiesta POST senza token/sessione.  
- **Access control:** controlli corretti sulle risorse degli utenti.

---

## 6. Conclusioni e Raccomandazioni

1. L’applicazione Next.js / React gestisce correttamente la sanitizzazione dell’input → protezione XSS efficace.  
3. Nessuna vulnerabilità critica lato rete o porte aperte.  
