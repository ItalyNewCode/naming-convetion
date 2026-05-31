# Struttura dei Package Java — WaveMaker

Questa guida definisce come organizzare cartelle e package Java all'interno di un progetto WaveMaker enterprise. La struttura di riferimento è ricavata da un progetto reale e deve essere rispettata da tutti gli autori.

---

## 1. Struttura Generale del Progetto

```
<NomeApp>/
├── services/                  ← servizi backend (uno per dominio o integrazione)
├── src/
│   ├── main/
│   │   ├── java/              ← codice Java condiviso tra servizi
│   │   ├── resources/
│   │   └── webapp/
│   │       ├── pages/         ← pagine e partial WaveMaker
│   │       ├── resources/     ← immagini, file, audio, video
│   │       └── themes/
│   └── test/
├── profiles/                  ← profili di configurazione (dev, prod)
├── i18n/                      ← file di traduzione
├── lib/                       ← librerie locali
└── pom.xml
```

---

## 2. Struttura di un Servizio

Ogni servizio vive in `services/<NomeServizio>/` e segue questa struttura:

```
services/<NomeServizio>/
├── designtime/                ← metadati WaveMaker — NON modificare manualmente
└── src/
    └── com/<azienda>/<app>/<nomeservizio>/
        ├── controller/
        ├── dao/
        ├── service/
        └── models/
            ├── procedure/
            └── query/
```

---

## 3. Naming della Cartella Servizio

| Tipo di servizio | Convenzione cartella | Esempi |
| ---------------- | -------------------- | ------ |
| Servizio DB (WM genera da schema) | MAIUSCOLO o MAIUSCOLO_CON_UNDERSCORE | `EVENTI`, `GIGLIOLI_PREVENTIVI` |
| Servizio custom (logica applicativa) | PascalCase | `FileService`, `ManagerArticoli`, `Note` |
| Integrazione REST esterna | camelCase o nome tecnico | `restListinoMateriali`, `ana01`, `ste01` |

**Regola generale:** usare **PascalCase** per tutti i nuovi servizi custom. I servizi generati da WaveMaker da schema DB ereditano il nome dello schema.

---

## 4. Naming del Package Java

Il package Java segue sempre questo schema:

```
com.<azienda>.<nomeapp>.<nomeservizio_lowercase>
```

| Variabile | Regola | Esempio |
| --------- | ------ | ------- |
| `<azienda>` | dominio aziendale in reverse, tutto minuscolo | `com.oneclickapp` |
| `<nomeapp>` | nome dell'applicazione, tutto minuscolo | `giglioli` |
| `<nomeservizio_lowercase>` | nome del servizio in minuscolo (snake_case se necessario) | `eventi`, `giglioli_preventivi`, `fileservice` |

**Esempi completi:**

```
com.oneclickapp.giglioli.eventi
com.oneclickapp.giglioli.giglioli_preventivi
com.oneclickapp.giglioli.fileservice
com.oneclickapp.giglioli.managerarticoli
com.oneclickapp.giglioli.note
```

> I package Java sono sempre in minuscolo — standard Java ufficiale. Il nome della cartella servizio può usare PascalCase, ma il package corrispondente è sempre lowercase.

---

## 5. Sub-package per Tipo di Servizio

### Servizio DB (generato da WaveMaker)

```
<package_servizio>/
├── <Entity>.java              ← entity JPA (una per tabella)
├── <Entity>Id.java            ← chiave composita (se necessario)
├── controller/
│   ├── <Entity>Controller.java
│   ├── ProcedureExecutionController.java
│   └── QueryExecutionController.java
├── dao/
│   └── <Entity>Dao.java
├── service/
│   ├── <Entity>Service.java
│   ├── <Entity>ServiceImpl.java
│   ├── <SERVIZIO>ProcedureExecutorService.java
│   ├── <SERVIZIO>ProcedureExecutorServiceImpl.java
│   ├── <SERVIZIO>QueryExecutorService.java
│   └── <SERVIZIO>QueryExecutorServiceImpl.java
└── models/
    ├── procedure/             ← request/response stored procedure
    │   ├── <Azione>Request.java
    │   └── <Azione>Response.java
    └── query/                 ← request/response query custom
        ├── Get<Dominio>Response.java
        └── <Dominio>Response.java
```

### Servizio Custom (logica applicativa)

```
<package_servizio>/
├── controller/
│   └── <NomeServizio>Controller.java
├── model/
│   └── <Modello>.java
└── <NomeServizio>.java        ← entry point del servizio (opzionale)
```

### Integrazione REST Esterna

```
<package_servizio>/
├── model/
│   ├── RootResponse.java
│   ├── ResponseDati.java
│   └── <Entità>EntryItem.java
└── service/
    └── <NomeServizio>Service.java
```

---

## 6. Naming dei File Java

### Classi principali

| Tipo | Pattern | Esempio |
| ---- | ------- | ------- |
| Entity JPA | `<Entità>.java` | `Articoli.java`, `Bom.java` |
| Entity con chiave composita | `<Entità>Id.java` | `NoteId.java` |
| Controller REST | `<Entità>Controller.java` | `ArticoliController.java` |
| DAO | `<Entità>Dao.java` | `ArticoliDao.java` |
| Service (interfaccia) | `<Entità>Service.java` | `ArticoliService.java` |
| Service (implementazione) | `<Entità>ServiceImpl.java` | `ArticoliServiceImpl.java` |

### Modelli Query e Procedure

| Tipo | Pattern | Esempio |
| ---- | ------- | ------- |
| Response query | `Get<Dominio>Response.java` | `GetArticoloByIdResponse.java` |
| Response generica | `<Dominio>Response.java` | `ArticoliResponse.java` |
| Request | `<Azione>Request.java` | `SetStatoRequest.java` |
| Response procedura | `<Azione>Response.java` | `SetStatoResponse.java` |
| Item lista | `<Dominio>EntryItem.java` | `ResponseDatiEntryItem.java` |

---

## 7. Classi Java Custom — `src/main/java`

Tutte le classi Java prodotte dall'autore a corredo del progetto (utility, exporter, service applicativi, modelli condivisi, cron job) devono essere collocate sotto:

```
src/main/java/com/<azienda>/<app>/
```

Non è ammesso mettere classi Java custom all'interno dei package generati da WaveMaker nei servizi DB, né in cartelle arbitrarie fuori da questo path.

```
src/main/java/com/<azienda>/<app>/
├── <NomeClasse>.java          ← utility o service applicativo globale
└── models/
    └── <dominio>/             ← modelli condivisi organizzati per dominio
        └── <Modello>.java
```

**Esempi:**

```
src/main/java/com/oneclickapp/giglioli/
├── CronService.java
├── GigliolicReportService.java
├── MasterExcelExporter.java
└── models/
    ├── Articolo.java
    ├── bom/
    │   ├── BOMArticolo.java
    │   ├── LavorazioniEsterne.java
    │   └── Materiali.java
    └── master/
        ├── ArticoloMaster.java
        └── Scheda.java
```

**Regola:** se una classe è usata solo da un servizio, sta nel package di quel servizio (`services/<NomeServizio>/src/...`). Se è condivisa tra due o più servizi, va obbligatoriamente in `src/main/java`.

---

## 8. Cartelle Custom in `src/main/webapp/resources/`

Ogni nuova cartella aggiunta dall'autore sotto `src/main/webapp/resources/` deve contenere un file `readme.md` che ne descrive il contenuto e lo scopo.

```
src/main/webapp/resources/
├── audios/
│   └── readme.md
├── files/
│   └── readme.md
├── images/
│   └── readme.md
├── videos/
│   └── readme.md
└── db/
    └── <tipo-database>/
        └── schema.sql     ← vedi database-schema.md
```

### Contenuto minimo del `readme.md`

```markdown
# <Nome Cartella>

Descrizione del contenuto di questa cartella.

## Contenuto

- Tipo di file atteso
- Naming convention applicata (se presente)
- Note per i contributori
```

**Regole:**

* Il `readme.md` è obbligatorio anche se la cartella è inizialmente vuota — serve a dichiarare l'intenzione e le regole d'uso.
* Non usare `Readme.txt` o altri formati: solo `readme.md` in minuscolo.
* Il file va committato insieme alla creazione della cartella nella stessa PR.

---

## 9. Riepilogo Regole Chiave

| Elemento | Regola |
| -------- | ------ |
| Cartella servizio | PascalCase (custom), MAIUSCOLO (DB da schema WM) |
| Package Java | sempre minuscolo, mai PascalCase |
| Entity JPA | PascalCase senza suffisso |
| Controller | `<Entity>Controller` |
| DAO | `<Entity>Dao` |
| Service interfaccia | `<Entity>Service` |
| Service impl | `<Entity>ServiceImpl` |
| Response query | `Get<X>Response` o `<X>Response` |
| Request | `<X>Request` |
| Classi Java custom | in `src/main/java/com/<azienda>/<app>/` |
| Modelli condivisi | in `src/main/java/.../models/<dominio>/` |
| Codice solo di un servizio | nel package del servizio, mai in `src/main/java` |
| Nuova cartella in `resources/` | deve contenere un `readme.md` in minuscolo |
