---
title: Processo di Fornitura
nav_order: 52
---

# Processo di Fornitura — Innovation Code Community

Questo documento descrive il ciclo di vita di un progetto software affidato alla community Innovation Code, dalla fase di ingaggio iniziale con il cliente fino alla validazione finale del collaudo.

---

## Panoramica del Processo

```
Cliente ingaggia IC
        │
        ▼
Colloquio + Survey iniziale
        │
        ▼
Offerta commerciale
  ├── Allegato Tecnico alla Fornitura
  └── Mappa RACI
        │
        ▼
Firma del Contratto di Fornitura
        │
        ▼
Analisi di Dettaglio  ◄── PM + Autore
        │
        ▼
Sviluppo  ◄── Supervisione Comitato Tecnico + PM
  └── Documentazione Tecnica in formato condiviso
        │
        ▼
Documento di Consegna  ◄── Autore → PM
        │
        ▼
Foglio di Collaudo  ◄── Cliente valida
```

---

## Fase 1 — Ingaggio

Il cliente contatta la community **Innovation Code** per l'affidamento di un progetto software.

In questa fase vengono raccolte le prime informazioni sul contesto aziendale, sullo stack tecnologico esistente e sulle esigenze generali. L'obiettivo non è ancora la raccolta dei requisiti funzionali, ma la qualificazione dell'opportunità.

Strumenti di supporto a questa fase:
- [Survey Onboarding Community](community-onboarding.md) — per le aziende che intendono entrare nella community
- [Survey Requisiti di Progetto](project-requirements.md) — per la raccolta dei requisiti tecnici ed economici iniziali

---

## Fase 2 — Colloquio e Survey Iniziale

A seguito dell'ingaggio, viene condotto un **colloquio con il cliente** per approfondire il contesto e le aspettative. Il colloquio è guidato dalla Survey Requisiti di Progetto.

L'esito del colloquio produce:
- Un quadro chiaro del perimetro funzionale atteso
- L'identificazione delle principali dipendenze tecnologiche e architetturali
- Una prima stima dell'impegno necessario
- L'identificazione del PM e dell'autore (o degli autori) da coinvolgere

Il colloquio è condotto congiuntamente da un membro del comitato tecnico e dal PM designato.

---

## Fase 3 — Offerta Commerciale

Sulla base del colloquio e della survey, la community predispone l'**offerta commerciale**. L'offerta è composta da due documenti obbligatori:

### 3.1 Allegato Tecnico alla Fornitura

L'allegato tecnico descrive il **perimetro della fornitura** a un livello sufficiente a definire cosa è incluso e cosa è escluso, senza costituire un'analisi funzionale di dettaglio.

Deve contenere:
- Descrizione ad alto livello delle macro-funzionalità incluse nella fornitura
- Stack tecnologico previsto
- Ambienti di deploy (sviluppo, quality, produzione)
- Esclusioni esplicite dal perimetro
- Modalità di consegna (repository, documentazione, collaudo)
- Eventuali vincoli o dipendenze da sistemi del cliente

L'allegato tecnico non è un'analisi funzionale di dettaglio: i requisiti di dettaglio sono oggetto della **Fase 5**.

### 3.2 Mappa RACI

La mappa RACI definisce chiaramente le responsabilità di ciascun attore sul progetto.

| Attività | Cliente | PM | Autore/i | Comitato Tecnico |
| -------- | ------- | -- | -------- | ---------------- |
| Definizione requisiti di business | **R/A** | C | I | I |
| Analisi di dettaglio | I | **A** | **R** | C |
| Sviluppo e test | I | **A** | **R** | C |
| Documentazione tecnica | I | **A** | **R** | **C** |
| Validazione documentazione | I | C | I | **R/A** |
| Consegna dei lavori | I | **A** | **R** | I |
| Collaudo funzionale | **R/A** | C | I | I |

> **R** = Responsible (chi esegue) · **A** = Accountable (chi risponde del risultato) · **C** = Consulted · **I** = Informed

---

## Fase 4 — Firma del Contratto di Fornitura

Il cliente riceve e firma il **contratto di fornitura**, comprensivo dell'allegato tecnico (Fase 3.1) e della mappa RACI (Fase 3.2).

Nessuna attività di analisi o sviluppo inizia prima della firma del contratto. La firma sancisce l'accordo sul perimetro e sulle responsabilità.

---

## Fase 5 — Analisi di Dettaglio

Dopo la firma del contratto, il PM e l'autore assegnato avviano l'**analisi di dettaglio** seguendo il modello definito dalla community.

L'analisi di dettaglio produce:
- Specifiche funzionali delle singole funzionalità incluse nel perimetro
- Schema dati / modello ER (se applicabile)
- Diagrammi di flusso per i processi principali
- Identificazione delle API e delle integrazioni
- Piano di sviluppo con milestone

L'analisi di dettaglio è condivisa con il cliente per conferma prima dell'avvio dello sviluppo. Eventuali scostamenti rispetto all'allegato tecnico devono essere gestiti con una variazione contrattuale concordata.

---

## Fase 6 — Sviluppo e Documentazione Tecnica

Durante la fase di sviluppo, il **comitato tecnico** verifica che gli autori producano e mantengano la documentazione tecnica nel **formato condiviso dalla community**, sotto la supervisione del PM.

### Documentazione obbligatoria durante lo sviluppo

La documentazione deve essere prodotta in parallelo allo sviluppo, non a posteriori. Il comitato tecnico valida che ogni contributo rispetti le convenzioni di questo repository:

- Naming conventions (database, package Java, API REST)
- Configurazione di sicurezza (ruoli, segreti, OWASP)
- Configurazione logging (`log4j2.xml`)
- Schema database versionato nel repository
- Dipendenze dichiarate con versione esatta nel `pom.xml`
- SBOM generata ad ogni release

Il PM è responsabile del rispetto delle scadenze e della qualità complessiva. Il comitato tecnico è responsabile della conformità alle convenzioni tecniche.

---

## Fase 7 — Documento di Consegna

Al termine dei lavori, l'autore assegnato fornisce al PM un **documento di consegna** che attesta il completamento delle attività previste.

Il documento di consegna include:
- Riepilogo delle funzionalità realizzate, riferite alle specifiche dell'analisi di dettaglio
- Elenco degli eventuali scostamenti rispetto all'analisi di dettaglio, con motivazione
- Istruzioni per il deploy in produzione
- Credenziali e accessi da consegnare al cliente (gestiti tramite secret manager — vedi [security.md](security.md))
- Riferimenti alla documentazione tecnica prodotta
- Checklist di verifica pre-collaudo

Il documento di consegna è revisore dal PM prima di essere inoltrato al comitato tecnico per la validazione finale.

---

## Fase 8 — Foglio di Collaudo

Se il documento di consegna è validato e i lavori corrispondono alle specifiche, viene predisposto il **foglio di collaudo** da consegnare al cliente.

Il foglio di collaudo contiene:
- Elenco delle funzionalità da verificare, derivato dall'analisi di dettaglio
- Casi di test da eseguire per ciascuna funzionalità
- Spazio per l'esito del test (superato / non superato) e per le note del cliente
- Sezione di firma del cliente a conferma dell'accettazione

Il cliente esegue il collaudo e restituisce il foglio firmato. Il collaudo firmato costituisce l'**accettazione formale** della fornitura.

In caso di non conformità rilevate durante il collaudo, viene aperto un ciclo di correzione con priorità concordata tra PM e cliente, prima di una nuova sessione di collaudo.

---

## Riferimenti

- [Survey Onboarding Community](community-onboarding.md)
- [Survey Requisiti di Progetto](project-requirements.md)
- [Sicurezza — Gestione dei Segreti](security.md)
- [Continuous Delivery](cicd.md)
