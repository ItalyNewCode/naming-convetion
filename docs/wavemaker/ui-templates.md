# UI Templates e Standard Frontend — WaveMaker

Ogni applicazione WaveMaker deve adottare un'interfaccia utente coerente e di qualità professionale. Quando i template Material inclusi nella piattaforma non sono utilizzati, l'autore deve conformarsi agli esempi ufficiali pubblicati nello **WaveMaker Showcase**.

---

## 1. Regola Fondamentale

> Il frontend di un'applicazione deve essere quanto più simile possibile alle applicazioni di riferimento dello WaveMaker Showcase. Non è accettabile consegnare un'interfaccia che si discosta significativamente dallo stile, dalla struttura e dall'esperienza utente dimostrata in questi esempi.

Due percorsi ammessi:

| Percorso | Quando usarlo |
| -------- | ------------- |
| **Template Material WaveMaker** | Prima scelta — disponibili direttamente nell'IDE al momento della creazione del progetto |
| **Showcase app come riferimento** | Quando il template Material non è adatto al tipo di applicazione o al contesto di business |

In nessun caso è ammesso un frontend creato da zero senza un riferimento esplicito a uno dei due percorsi sopra.

---

## 2. WaveMaker Showcase — Riferimento Ufficiale

Lo showcase ufficiale è disponibile all'indirizzo:

**[https://showcase.onwavemaker.com/](https://showcase.onwavemaker.com/)**

Le applicazioni presenti coprono diversi settori e tipologie (web e mobile). Ogni app è scaricabile dal repository GitHub dell'organizzazione WaveMaker:

**[https://github.com/wavemakerapps](https://github.com/wavemakerapps)**

L'autore deve individuare l'applicazione showcase più vicina al dominio del progetto, scaricarla, studiarla e usarla come modello per struttura, navigazione, componenti e stile visivo.

---

## 3. Applicazioni Disponibili nello Showcase

### Applicazioni Web

| App | Settore | Descrizione |
| --- | ------- | ----------- |
| **Sales Vision** | Sales, Marketing | Gestione vendite, pipeline commerciale, relazioni con i clienti |
| **Pixi Service** | Service Center | Gestione assistenza tecnica e ticketing per centri servizi |
| **Loancorp** | Banking, Fintech | Richiesta e gestione prestiti con processo a step minimali |
| **Teller** | Fintech, Banking | Gestione depositi bancari con autenticazione e integrazione DB |
| **DataWiz** | Dashboard, Data Management | Rendering dinamico di tabelle, grafici e liste da configurazione DB |

### Applicazioni Mobile

| App | Settore | Descrizione |
| --- | ------- | ----------- |
| **Turbo Mobile** | Sales, Telco | App mobile per la forza vendita con esperienza personalizzabile |
| **Blink Wireless** | E-commerce, Telco | Catalogo prodotti e piani tariffari con UX ottimizzata per mobile |

---

## 4. Come Usare lo Showcase come Riferimento

### Passo 1 — Scegliere l'app di riferimento

Selezionare l'applicazione showcase il cui dominio funzionale è più vicino al progetto da sviluppare:

* App finanziarie o bancarie → **Loancorp** o **Teller**
* Dashboard e reportistica → **DataWiz**
* CRM o gestione commerciale → **Sales Vision**
* Assistenza e ticketing → **Pixi Service**
* App mobile commerciale → **Turbo Mobile** o **Blink Wireless**
* E-commerce mobile → **Blink Wireless**

### Passo 2 — Scaricare il progetto

Trovare il repository corrispondente in [github.com/wavemakerapps](https://github.com/wavemakerapps) e scaricare il codice sorgente. Il progetto è un'applicazione WaveMaker standard apribile nell'IDE.

### Passo 3 — Analizzare la struttura

Prima di iniziare lo sviluppo, analizzare nell'app di riferimento:

* Struttura di navigazione (menu, sidebar, topbar)
* Layout delle pagine principali (lista, dettaglio, form)
* Componenti utilizzati (tabelle, card, grafici, form)
* Gestione della responsività
* Palette colori e tipografia (tema applicato)

### Passo 4 — Replicare lo stile nel proprio progetto

Sviluppare le proprie pagine rispettando:

* La stessa gerarchia di navigazione
* Gli stessi pattern di layout per pagine dello stesso tipo (es. lista → sempre tabella con filtri in alto)
* I componenti WaveMaker equivalenti a quelli usati nell'app di riferimento
* Il tema o una sua variante coerente

---

## 5. Cosa Non è Accettabile

* Interfacce senza tema applicato (stile di default non personalizzato)
* Layout difformi dallo showcase senza motivazione documentata
* Componenti personalizzati in HTML/CSS grezzo quando esiste un widget WaveMaker equivalente
* Navigazione incoerente tra pagine dello stesso applicativo
* Mancanza di responsività su dispositivi mobile (per app web) o su diverse dimensioni di schermo (per app mobile)

---

## 6. Documentazione di Riferimento per ogni App Showcase

Le singole app dello showcase hanno documentazione tecnica dedicata su:

**[https://showcase.wavemaker.com/showcase/docs/](https://showcase.wavemaker.com/showcase/docs/)**

La documentazione descrive architettura, componenti usati, integrazione con i servizi backend e pattern implementati. È la fonte primaria da consultare prima di iniziare lo sviluppo.
