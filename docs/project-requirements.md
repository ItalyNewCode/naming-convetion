---
title: Survey Requisiti di Progetto
nav_order: 51
---

# Definizione dei Requisiti di Progetto

Survey per la raccolta dei requisiti iniziali da sottoporre al cliente finale in fase di prevendita. Le risposte alimentano la costruzione del blueprint di progetto, poi portato alla community e agli autori/PM.

---

## 1. Requisiti Generali

**AVETE UN "NDA" DA SOTTOSCRIVERE PER LA RACCOLTA DELLE INFORMAZIONI?**
- [ ] SI
- [ ] NO
- [ ] NON NECESSARIO
- [ ] UTILIZZIAMO QUELLO FORNITO DA VOI

---

**AVETE IDENTIFICATO UN REFERENTE AZIENDALE (P.M.) CHE SEGUIRÀ IL PROGETTO?**
- [ ] SI
- [ ] NON ANCORA
- [ ] CI SERVE SUPPORTO PER LA SCELTA DI UN P.M. ESTERNO

---

**AVETE IDENTIFICATO UN REFERENTE AZIENDALE CHE SI DEDICHERÀ AL FEEDBACK LOOP E/O TESTING APPLICATIVO?**
- [ ] SI — SARÀ UN UTENTE DIVERSO DAL P.M.
- [ ] SI — SARÀ IL P.M.
- [ ] NON ANCORA

---

**AVETE UNA TIME LINE DA RISPETTARE PER LA REALIZZAZIONE?**
- [ ] NO
- [ ] SI
  - [ ] ENTRO 3 MESI
  - [ ] ENTRO 6 MESI
  - [ ] ALTRO: `_______________________________________`

---

**AVETE UNA STRUTTURA DI SVILUPPO SOFTWARE / SISTEMISTICA INTERNA?**
- [ ] NO
- [ ] SI — da quali figure è composta?

  `_______________________________________________`

---

**INSERISCI UNA DESCRIZIONE GENERALE DEL PROGETTO:**

`___________________________________________________________________________________`

`___________________________________________________________________________________`

`___________________________________________________________________________________`

`___________________________________________________________________________________`

---

## 2. Requisiti Legali

**AVETE ESEGUITO UNA VALUTAZIONE DI COMPLIANCE AL GDPR E/O NIS2 PER QUANTO CONCERNE LE INFORMAZIONI E/O I DATI TRATTATI DAL SOFTWARE?**
- [ ] SI
  - [ ] È COMPLIANCE *(non trattiamo dati sensibili)*
  - [ ] L'APPLICAZIONE NECESSITA DI SVILUPPO "PRIVACY BY DESIGN"
  - [ ] A QUALE COMPLIANCE NORMATIVA DEVE SOTTOSTARE IL SISTEMA? `________________`
- [ ] NO
  - [ ] NECESSITIAMO DI SUPPORTO PER QUESTA VALUTAZIONE
  - [ ] NON LO RITENIAMO NECESSARIO
  - [ ] NON SO
- [ ] ABBIAMO UN DPO INTERNO CON CUI POTETE PARLARE
- [ ] VI CHIEDIAMO DI VERIFICARE CON IL DPO ESTERNO CHE SEGUE L'AZIENDA

---

**AVETE UNA PRIVACY POLICY AZIENDALE?**
- [ ] SI
  - [ ] POSSIAMO FORNIRE DOCUMENTAZIONE ADEGUATA
  - [ ] ABBIAMO UN DPO INTERNO CON CUI POTETE PARLARE
  - [ ] VI CHIEDIAMO DI INTERFACCIARVI CON IL DPO ESTERNO CHE SEGUE L'AZIENDA
- [ ] NO
  - [ ] NECESSITIAMO DI SUPPORTO IN TAL SENSO
  - [ ] NON CI INTERESSA

> ⚠️ In assenza di Privacy Policy non sarà possibile pubblicare l'app negli store.

---

**È NECESSARIO SOTTOSCRIVERE UN CONTRATTO CHE REGOLAMENTI I SERVICE LEVEL AGREEMENT RELATIVI AI SERVIZI EROGATI?**
- [ ] SI
- [ ] NO

---

## 3. Requisiti Tecnici

### Distribuzione del software

**DOVE SARÀ INSTALLATO O COME SARÀ DISTRIBUITO IL SOFTWARE?**

- [ ] **PUBLIC CLOUD** — avete già un provider cloud?
  - [ ] NO
  - [ ] SI
    - [ ] AWS
    - [ ] AZURE
    - [ ] GOOGLE
    - [ ] DIGITAL OCEAN
    - [ ] OVH
    - [ ] ALTRO: `________________________________`

- [ ] **ON PREMISES** — indicare lo stack tecnologico utilizzato in azienda:

  `_______________________________________________`

---

**AVETE LE COMPETENZE SISTEMISTICHE NECESSARIE AL MONITORAGGIO E AL TUNING DEI SISTEMI?**
- [ ] SI
- [ ] NO
  - [ ] DESIDERIAMO SUPPORTO PER ACQUISIRLE
  - [ ] DESIDERIAMO DELEGARE A VOI QUESTO ASPETTO

---

**SIETE ORGANIZZATI PER FORNIRE UN AMBIENTE DI QUALITY PER IL TEST DI COLLAUDO DELL'APPLICAZIONE?**
- [ ] SI *(forniteci le specifiche)*
- [ ] NO *(il collaudo sarà eseguito nell'ambiente di Quality di Innovation Code)*

---

**SIETE ORGANIZZATI PER L'EROGAZIONE DELL'AMBIENTE DI PRODUZIONE DEL SOFTWARE CHE VERRÀ REALIZZATO?**
- [ ] SI
- [ ] NO
  - [ ] DESIDERIAMO SUPPORTO PER ACQUISIRE LE COMPETENZE
  - [ ] DESIDERIAMO DELEGARE A VOI QUESTO ASPETTO

---

### Tipologia di progetto

**IL PROGETTO PREVEDE LO SVILUPPO DI:**

- [ ] **UNA PIATTAFORMA SAAS PUBBLICA** — agli utenti sarà applicato un canone?
  - [ ] NO — L'APPLICAZIONE NON GESTIRÀ CANONI DI UTILIZZO
  - [ ] SI — deve essere prevista la gestione degli incassi?
    - [ ] NO *(pagamento con bonifico o altre forme indirette)*
    - [ ] SI — avete già scelto un gateway?
      - [ ] NO — LASCIAMO DECIDERE A VOI
      - [ ] NO — VALUTEREMO LA VOSTRA PROPOSTA
      - [ ] SI
        - [ ] PAYPAL
        - [ ] STRIPE
        - [ ] XPAY
        - [ ] ALTRO: `________________`

- [ ] **UNA PIATTAFORMA PER USO AZIENDALE**

---

### Sicurezza

**DOBBIAMO CONSIDERARE SPECIFICHE POLITICHE DI SICUREZZA?**
- [ ] NO
- [ ] SI
  - [ ] AUTENTICAZIONE A DUE FATTORI
  - [ ] CRITTOGRAFIA DEI DATI
  - [ ] BACKUP DEI DATI — sono previste specifiche politiche di retention?
    - [ ] NO
    - [ ] SI — indicare quali: `__________________________`
  - [ ] DISASTER RECOVERY — sono previsti specifici obiettivi di R.T.O. e R.P.O.?
    - [ ] NO
    - [ ] SI
      - R.T.O.: `________________________________`
      - R.P.O.: `________________________________`
  - [ ] ALTRO: `__________________________________________________________`

---

### Architettura

**A LIVELLO ARCHITETTURALE, QUALE SOLUZIONE DOBBIAMO CONSIDERARE?**

- [ ] **PIATTAFORMA WEB**

- [ ] **MOBILE APPLICATION**
  - [ ] APP NATIVA — deve essere pubblicata sugli store?
    - [ ] NO
    - [ ] SI
      - [ ] GOOGLE PLAY
      - [ ] APPLE APP STORE
  - [ ] WEB APPLICATION (PWA)

---

**È NECESSARIO PREVEDERE SPECIFICI REQUISITI DI COMPATIBILITÀ?**
- [ ] NO
- [ ] SI
  - Specifici browser: `____________________________________________`
  - Sistemi operativi: `____________________________________________`
  - Apparati hardware: `____________________________________________`

---

### Utenti

**QUANTI UTENTI È PREVISTO CHE POSSANO UTILIZZARE IL SOFTWARE?**
- [ ] NON LO SAPPIAMO
- [ ] ABBIAMO UNA STIMA
  - N. utenti interni (attesi): `__________`
  - N. utenti esterni (attesi): `__________`
  - N. max utenti interni: `__________`
  - N. max utenti esterni: `__________`

---

**DOBBIAMO PREVEDERE L'AUTENTICAZIONE DEGLI UTENTI?**
- [ ] NO — non prevediamo autenticazione degli utenti
- [ ] SI
  - [ ] AUTENTICAZIONE BASATA SU PASSWORD — regole aziendali: `__________________________`
  - [ ] AUTENTICAZIONE BASATA SU CERTIFICATO
  - [ ] AUTENTICAZIONE BIOMETRICA
  - [ ] AUTENTICAZIONE BASATA SU TOKEN
  - [ ] ONE-TIME PASSWORD (OTP)
  - [ ] AUTENTICAZIONE A DUE FATTORI (2FA)
  - [ ] AUTENTICAZIONE A PIÙ FATTORI (MFA)
  - [ ] AUTENTICAZIONE TRAMITE DOMINIO INTERNO — LDAP o indicare quale: `____________________`
  - [ ] AUTENTICAZIONE TRAMITE PROVIDER PUBBLICI
    - [ ] GOOGLE
    - [ ] MICROSOFT
    - [ ] FACEBOOK
    - [ ] INSTAGRAM
    - [ ] ALTRO: `____________________________________`
  - [ ] SPID

---

**L'ELENCO DEGLI UTENTI È GIÀ PRESENTE SU SISTEMI IN PRODUZIONE?**
- [ ] NO
- [ ] SI
  - [ ] IL SISTEMA ESPONE API DOCUMENTATE
  - [ ] IL SISTEMA CONSENTE L'ACCESSO ALLE TABELLE UTENTI

---

**PREVEDIAMO IL SINGLE SIGN ON (SSO)?**
- [ ] NO
- [ ] SI
  - [ ] ACTIVE DIRECTORY
  - [ ] OFFICE 365
  - [ ] KERBEROS
  - [ ] ALTRO: `________________________________`

---

### Integrazioni

**IL PROGETTO SOFTWARE PREVEDE L'INTEGRAZIONE CON DEVICE ESTERNI? (IoT, PLC, ecc.)**
- [ ] NO
- [ ] SI — indicare quali: `________________________________________`

---

**IL PROGETTO SOFTWARE PREVEDE LA CONNESSIONE A SERVIZI, API E/O BANCHE DATI ESISTENTI?**
- [ ] NO
- [ ] SI
  - [ ] **ERP AZIENDALE**
    - [ ] SAP
    - [ ] ORACLE
    - [ ] MICROSOFT
    - [ ] BASATO SU AS-400
    - [ ] ALTRO: `_______________________________________________`
  - [ ] **CRM**
    - [ ] SALESFORCE
    - [ ] DYNAMICS
    - [ ] HUBSPOT
    - [ ] SAP CEC SUITE
    - [ ] ORACLE CX CLOUD
    - [ ] MAILCHIMP
    - [ ] ZOHO
    - [ ] CLICKUP
    - [ ] SUGARCRM
    - [ ] ZENDESK
    - [ ] MARKETO
    - [ ] ALTRO: `_______________________________________________`
  - [ ] **PDM** — Nome: `________________________________________`
  - [ ] **PLM** — Nome: `________________________________________`
  - [ ] **ELM** — Nome: `________________________________________`
  - [ ] **BPM** — Nome: `________________________________________`
  - [ ] **MES** — Nome: `________________________________________`
  - [ ] **WMS** — Nome: `________________________________________`
  - [ ] **DMS** — Nome: `________________________________________`
  - [ ] **BUSINESS INTELLIGENCE**
    - [ ] SAP BUSINESS OBJECT
    - [ ] MICROSTRATEGY
    - [ ] QLIK SENSE
    - [ ] ZOHO ANALYTICS
    - [ ] SISENSE
    - [ ] MICROSOFT POWER BI
    - [ ] GOOGLE DATA STUDIO
    - [ ] CLEAR ANALYTICS
    - [ ] TABLEAU
    - [ ] ORACLE BI
    - [ ] IBM COGNOS ANALYTICS
    - [ ] ALTRO: `_________________________________________`

---

**IL PROGETTO SOFTWARE PREVEDE LA FORNITURA DI SERVIZI API A SISTEMI TERZI?**
- [ ] NO
- [ ] SI — indicare quali: `________________________________________`

---

## 4. Requisiti Funzionali

**SI TRATTA DI UNO SVILUPPO EX NOVO O DI UN REFACTORING DI UN'APPLICAZIONE ESISTENTE?**
- [ ] NUOVO SVILUPPO
- [ ] REFACTORING
  - [ ] RICHIESTO PER GLI ASPETTI TECNOLOGICI
  - [ ] RICHIESTO PER GLI ASPETTI FUNZIONALI

---

**ESISTE UN'ANALISI CHE DOCUMENTA LE FUNZIONALITÀ DEL SOFTWARE?**
- [ ] NO
- [ ] SI — sono suddivise per tipologia (trasversali e verticali)?
  - [ ] NO
  - [ ] SI

---

**È DISPONIBILE UN LAYOUT GRAFICO ATTRAVERSO UN MOCKUP?**
- [ ] NO
  - [ ] CHIEDIAMO A VOI DI REALIZZARLO
  - [ ] ABBIAMO LA POSSIBILITÀ DI REALIZZARLO
- [ ] SI

---

**L'INTERFACCIA GRAFICA DEVE RISPONDERE A REQUISITI DI UNIFORMITÀ CON LA BRAND IDENTITY AZIENDALE?**
- [ ] NO
- [ ] SI
  - [ ] È PREVISTA LA FORNITURA DI TEMPLATE BASATI SU CSS E FRAMEWORK DI COMUNE UTILIZZO — indicare quali: `___________________`
  - [ ] È NECESSARIO PREVEDERE LO STUDIO DI UX E UI PERSONALIZZATI

---

**L'APPLICAZIONE DEVE PREVEDERE UN TAGLIO INTERNAZIONALE?**
- [ ] NO
- [ ] SI
  - Numero di lingue: `___________________________`
  - Chi fornirà il supporto di traduzione?
    - [ ] DISPONIAMO DI RISORSE INTERNE
    - [ ] SARÀ FORNITO ESTERNAMENTE
      - [ ] IL NOSTRO P.M. GESTIRÀ IL RAPPORTO CON IL FORNITORE
      - [ ] CHIEDIAMO CHE SIA IL VOSTRO P.M. A GESTIRE IL RAPPORTO CON IL FORNITORE
    - [ ] CHIEDIAMO A VOI DI FORNIRE IL SERVIZIO

---

**SI DEVONO PREVEDERE LOGICHE DI PROFILAZIONE APPLICATIVA DEGLI UTENTI?**
- [ ] NO
- [ ] SI
  - [ ] ABBIAMO GIÀ IDENTIFICATO LA TASSONOMIA DELLA STRUTTURA GERARCHICA E LE LOGICHE CHE NE SOTTENDONO
  - [ ] DOBBIAMO ANCORA RAGIONARE SU QUESTI ASPETTI

---

**QUALI DI QUESTE FUNZIONALITÀ TRASVERSALI FARANNO PARTE DEL PRODOTTO?**
- [ ] UPLOAD E/O DOWNLOAD FILE
- [ ] OCR
- [ ] PREVIEW FILE SENZA DOWNLOAD
- [ ] POSSIBILITÀ PER I NUOVI UTENTI DI REGISTRARSI AUTONOMAMENTE
- [ ] RICERCA DI DOCUMENTI INDICIZZABILI
- [ ] VISUALIZZAZIONE GALLERIE DI FOTO
- [ ] VISUALIZZAZIONE VIDEO
- [ ] MESSAGING E NOTIFICATION
- [ ] FUNZIONALITÀ E-COMMERCE

---

**QUALI FUNZIONALITÀ AI SONO ATTESE NEL PRODOTTO?**
- [ ] RICERCA SU REPO DOCUMENTALI
- [ ] RICERCA PER CONTENUTI SEMANTICI
- [ ] SVILUPPO DI AGENTI SPECIFICI
- [ ] ALTRO: `_______________________________________________`

---

## 5. Requisiti Economici

**AVETE DEFINITO UN BUDGET GENERALE DI PROGETTO?**
- [ ] NO
- [ ] SI — importo: `_______________________________`

---

**AVETE DEFINITO UN BUDGET DEDICATO AI COSTI INFRASTRUTTURALI?**
- [ ] NO
- [ ] SI — importo: `_______________________________`
  - [ ] PUBLIC CLOUD
  - [ ] INFRASTRUTTURA IN PREMISES
  - [ ] EVENTUALI LICENZE ACCESSORIE *(server, DB, client, CAL, ecc.)*

---

**AVETE DEFINITO UN BUDGET DEDICATO ALLA MANUTENZIONE EVOLUTIVA?**
- [ ] NO
- [ ] SI — importo: `_______________________________`

---

*Documento per la raccolta dei requisiti in fase di prevendita — da portare alla community e agli autori/PM per la costruzione del blueprint di progetto.*
