# Penetration Test – tscloudapp.vercel.app

Questo repository documenta i test di sicurezza effettuati sul sito di test ospitato su Vercel.  
**Obiettivo:** verificare la sicurezza e la logica dell’applicazione Next.js/TypeScript.

---

## 1. Introduzione
Il report descrive i test di sicurezza effettuati sui principali endpoint e input dell’applicazione.  
Focus su:
- Login e autenticazione
- XSS e input sanitization
- CSRF e controllo sessione
- Enumerazione ID utenti (nid)
- Controllo accesso e sicurezza API

---

## 2. Ambiente
- **Sistema operativo:** Kali Linux  
- **Strumenti:** Burp Suite Community Edition, Browser con DevTools, sqlmap, Nikto, Gobuster, Dirb, Nmap  
- **Applicazione:** Next.js / TypeScript  
- **Porte rilevate:** 443 (HTTPS), 8080 (filtrata)

---

## 3. Test effettuati

| # | Campo / Endpoint                  | Tipo di test                  | Input / Payload                              | Risultato / Note |
|---|----------------------------------|-------------------------------|---------------------------------------------|-----------------|
| 1 | Login (/api/login)                | Enumerazione NID              | NID validi e sconosciuti, password admin    | "Password non corretta" → NID esiste; "Codice non valido" → NID inesistente |
| 2 | Motivo ferie (/api/ferie)         | XSS / Caratteri speciali      | `<script>alert('Test')</script>`            | Sanitizzazione attiva, alert non eseguito |
| 3 | Nome utente / Bio / Descrizione   | XSS / Input speciale          | `<b>bold</b>`, `<img src=x onerror=alert(1)>` | Output escapato → sicuro |
| 4 | Modifica ferie                    | CSRF / Session test           | POST senza cookie/sessione                  | Fallisce senza sessione valida → protezione attiva |
| 5 | Visualizzazione ferie / profilo   | Access control                | Modifica ID risorsa `/api/ferie/ID`        | Status code corretto (403/401) per utenti non autorizzati |
| 6 | Cookie / Session                  | Manipolazione cookie          | Modifica cookie di sessione                | Azioni bloccate se cookie non valido |

---

## 4. Screenshot

| Test | Screenshot |
|------|------------|
| Login Test | ![Login page]( https://i.imgur.com/5I12Dsj ) |
| Burp Intercept | ![Burp login intercept]( https://i.imgur.com/UwTSsbV ) |
| NID Enumeration | ![Tentativo id reale]( https://i.imgur.com/tEaqX2s ) |

*(Per tutti gli screenshot completi, vedere la cartella `screenshots/` o i link Imgur.)*

---

## 5. Risultati
- **XSS:** tutti i payload testati vengono sanitizzati → nessuna vulnerabilità XSS stored o riflessa.  
- **Enumerazione NID:** possibile distinguere NID esistenti e non → vulnerabilità logica in ambiente di test.  
- **CSRF:** protezione corretta verificata tramite richiesta POST senza token/sessione.  
- **Access control:** controlli corretti sulle risorse degli utenti.  
- **Porte e rete:** nessuna vulnerabilità critica lato rete o porte aperte.

---

## 6. Conclusioni e raccomandazioni
1. L’applicazione Next.js / React gestisce correttamente la sanitizzazione dell’input → protezione XSS efficace.  
2. Nessuna vulnerabilità critica lato rete o porte aperte.  
3. Enumerazione NID potrebbe rappresentare una vulnerabilità logica → considerare messaggi più generici su login.  
4. CSRF e controllo sessione attivi e funzionanti.  
