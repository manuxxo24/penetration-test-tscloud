# Security Testing Report – TestWatchApp

## 1. Introduzione
Il report descrive i test di sicurezza effettuati sull’applicazione Next.js/TypeScript ospitata su Vercel.  
**Obiettivo:** identificare vulnerabilità lato login, API, input utente, configurazioni server e logica applicativa.

**Ambiente di test:**
- Sistema operativo: Kali Linux  
- Strumenti: Burp Suite Community Edition, sqlmap, Nikto, Gobuster, Dirb, Nmap  
- Applicazione target: Next.js / TypeScript  
- URL: `https://testwatchapp.vercel.app`  

---

## 2. Scansione iniziale e porte

**Obiettivo:** identificare le porte aperte e i servizi esposti dal server.  

| Porta | Stato     | Servizio   | Note                                 |
|-------|-----------|------------|--------------------------------------|
| 443   | open      | HTTPS      | Accessibile, login e API funzionanti |
| 80    | open      | HTTP       | Reindirizza automaticamente a HTTPS  |
| 8080  | filtered  | HTTP-proxy | Non raggiungibile dall’esterno       |

**Screenshot:**  
- IP del sito  
- Scan Nmap porte  

*Risultato:* solo le porte 80 e 443 risultano testabili, la 8080 è filtrata.

---

## 3. Enumerazione directory

**Strumenti:** Gobuster, Nikto, Dirb  
**Obiettivo:** scoprire directory e file sensibili esposti sul server.  

| Strumento | Risultato | Note                          |
|-----------|-----------|-------------------------------|
| Gobuster  | 403       | Tutti i tentativi bloccati    |
| Dirb      | 403       | Tutti i tentativi bloccati    |
| Nikto     | 403       | Nessuna directory accessibile |

**Screenshot:**  
- Gobuster/Dirb scansioni  

*Conclusione:* l’enumerazione di directory non ha dato informazioni utili.

---

## 4. Analisi dei tool web

| Tool     | Risultato                   | Note                                      |
|----------|-----------------------------|-------------------------------------------|
| curl     | Nessuna info utile          | Scan endpoint base                        |
| WhatWeb  | Individua stack tecnologico | Nessuna vulnerabilità rilevata            |
| Wafw00f  | Nessun WAF rilevato         | Applicazione non protetta da WAF visibile |

---

## 5. Login e SQL Injection

**Obiettivo:** testare endpoint `/api/login` per SQL injection e enumerazione NID.  

**Passaggi effettuati:**
1. Intercettata richiesta POST con Burp Suite Repeater.  
2. Tentativo SQL injection sul campo `nid` (accetta solo 4 cifre).  
3. Test sqlmap con file di richiesta.  

| Test                        | Input / Payload                 | Risultato / Note                |
|-----------------------------|---------------------------------|---------------------------------|
| NID valido, password errata | NID reale + pwd sbagliata       | `"Password non corretta"`       |
| NID inesistente             | NID casuale                     | `"Codice di accesso non valido"`|
| SQL Injection               | ' OR 1=1--                      | Nessun risultato                |
| Sqlmap file request         | Intercettata login POST         | 0 risultati                     |

*Conclusione:* nessuna vulnerabilità SQL injection, logica NID distinguibile.

---

## 6. Test XSS

**Obiettivo:** verificare Cross-Site Scripting sui campi input del sito.  

| Campo / Endpoint          | Payload                                       | Risultato / Note |
|---------------------------|-----------------------------------------------|----------------------------------------|
| Motivo ferie (/api/ferie) | `<script>alert('Test')</script>`              | Input sanitizzato, alert non eseguito  |
| Bio / Descrizione         | `<b>bold</b>`, `<img src=x onerror=alert(1)>` | Output escapato, nessuna vulnerabilità |
| Vari input frontend       | Payload HTML/JS                               | Sanitizzazione corretta                |

*Conclusione:* nessuna vulnerabilità XSS rilevata.

---

## 7. Test API

**Obiettivo:** intercettare richieste GET/POST con token, testare accesso a risorse protette.  

| Risorsa API            | Test                  | Risultato / Note        |
|------------------------|-----------------------|-------------------------|
| `/api/orders`          | Token intercettati    | 401 Token non valido    |
| `/api/watches`         | Token intercettati    | 500 Errore fetch        |
| `/api/documents`       | Token intercettati    | 500 Errore fetch        |
| `/api/users`           | Token intercettati    | 404 Risorsa non trovata |

*Conclusione:* i token validi non permettono accesso non autorizzato, alcune API restituiscono errori server.

---

## 8. Conclusioni e raccomandazioni

1. **SQL Injection:** non presente su login o API.  
2. **XSS:** input sanitizzato correttamente → protezione efficace.  
3. **CSRF e sessione:** protezione attiva verificata tramite richieste POST senza token/sessione.  
4. **Enumerazione NID:** possibile distinguere NID esistenti e non → considerare messaggi più generici.  
5. **API:** alcune restituiscono 500 o 404 → controllare gestione errori e validazione token.  
6. **Porte:** 8080 filtrata, nessuna azione possibile; 443 e 80 testabili senza vulnerabilità rilevate.

---

**Note finali:** il sito gestisce correttamente sanitizzazione degli input, controllo sessioni e accesso alle API. Nessuna vulnerabilità critica rilevata durante i test.
