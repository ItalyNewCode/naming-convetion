---
title: ORM — Entity, DAO, DTO
parent: WaveMaker
nav_order: 4
---

# ORM — Configurazione Entity, DAO e DTO

WaveMaker genera automaticamente le classi Java (POJO/Entity, DAO, Service) a partire dallo schema del database. La generazione automatica non esonera l'autore dalla responsabilità di **verificare e configurare correttamente** ogni campo nell'editor ORM, in particolare il `Value Type`, il flag `Updatable` e i `Validators`.

Una configurazione errata produce oggetti Java non allineati al comportamento reale del database, con conseguenze a runtime difficili da diagnosticare.

---

## 1. I Tre Oggetti Generati

| Oggetto | Responsabilità | Generato da |
| ------- | -------------- | ----------- |
| **Entity (POJO)** | Mappa 1:1 una tabella del DB tramite annotazioni JPA/Hibernate | WaveMaker dal DB schema |
| **DAO** | Interfaccia di accesso ai dati (CRUD, query custom) | WaveMaker, estendibile |
| **DTO** | Oggetti di trasferimento per query e stored procedure custom | Autore, in `models/query/` e `models/procedure/` |

L'autore deve verificare la configurazione ORM di ogni Entity e, dove necessario, creare i DTO appropriati per le query che non restituiscono un'Entity completa.

---

## 2. Value Type — Regola per Ogni Tipo di Campo

Il `Value Type` dichiara **chi è responsabile di fornire il valore** per quel campo. È la configurazione più critica dell'ORM: un valore errato causa inserimenti o aggiornamenti con dati inconsistenti.

| Value Type | Significato | Campi tipici |
| ---------- | ----------- | ------------ |
| **Database Defined** | Il valore è generato o gestito interamente dal database (sequence, trigger, default) | Chiavi primarie auto-increment, `creation_timestamp`, `modification_timestamp` con trigger, `deletion_timestamp` |
| **Server Defined** | Il valore è impostato dal server applicativo prima della persistenza | Campi popolati da logica Java custom (es. utente corrente da sessione) |
| **App Environment Defined** | Il valore proviene da una variabile di ambiente dell'applicazione | Configurazioni dinamiche legate all'ambiente (es. tenant ID) |
| **User Defined** | Il valore è fornito esplicitamente dall'utente tramite l'interfaccia | Tutti i campi compilati dall'utente in un form |

### Regola Fondamentale

> Se il database gestisce il valore — tramite `DEFAULT`, `AUTO_INCREMENT`, sequenza o trigger — il campo **deve** essere impostato a `Database Defined`. Non lasciarlo a `User Defined`.

Lasciare un campo auto-generato a `User Defined` significa che Hibernate tenterà di inviare quel valore nell'`INSERT` o nell'`UPDATE`, sovrascrivendo o entrando in conflitto con il meccanismo del database.

---

## 3. Configurazione per Tipo di Campo

### Chiave Primaria Auto-Increment

```
Value Type  → Database Defined
Updatable   → OFF
Insertable  → OFF  (il DB assegna il valore all'INSERT)
```

L'Entity risultante avrà:
```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
@Column(name = "user_id", updatable = false, insertable = false)
private Integer userId;
```

### `creation_timestamp`

Il timestamp di creazione è impostato dal database al momento dell'INSERT (tramite `DEFAULT CURRENT_TIMESTAMP`) e non deve mai essere modificato successivamente.

```
Value Type  → Database Defined
Updatable   → OFF
Insertable  → OFF
```

```java
@Column(name = "creation_timestamp", updatable = false, insertable = false)
private LocalDateTime creationTimestamp;
```

### `modification_timestamp`

Il timestamp di ultima modifica è aggiornato dal database ad ogni UPDATE (tramite trigger `ON UPDATE CURRENT_TIMESTAMP`).

```
Value Type  → Database Defined
Updatable   → OFF  (il trigger lo aggiorna, non Hibernate)
Insertable  → OFF
```

> Se non esiste un trigger e l'aggiornamento è gestito dal layer applicativo (es. `@PreUpdate`), usare `Server Defined` e impostare `Updatable → ON`.

### `deletion_timestamp` (Soft Delete)

Campo nullable. Null = record attivo; valorizzato = record cancellato logicamente. Il valore è impostato dall'applicazione, non dal database.

```
Value Type  → User Defined
Updatable   → ON
Insertable  → OFF  (null all'inserimento)
```

### Campi Compilati dall'Utente

Tutti i campi che l'utente riempie tramite un form.

```
Value Type  → User Defined
Updatable   → ON
Insertable  → ON
```

---

## 4. Validators

I `Validators` nell'ORM WaveMaker aggiungono validazione a livello applicativo, **prima** che il dato raggiunga il database.

**Regola:** se il vincolo è già garantito dal database (NOT NULL, CHECK, UNIQUE con constraint dichiarato), non duplicare la validazione nell'ORM a meno che non si voglia un messaggio di errore più leggibile lato applicazione.

| Scenario | Configurazione Validators |
| -------- | ------------------------- |
| Campo obbligatorio gestito da DB NOT NULL | `No Validation` (il DB respinge il valore nullo) oppure `Required` per messaggio applicativo migliore |
| Campo con formato vincolato (email, codice) | Aggiungere validatore `Pattern` con REGEX |
| Campo numerico con range | Aggiungere validatore `Min` / `Max` |
| Campo auto-generato dal DB | `No Validation` — il DB non accetta input esterno |
| Timestamp gestiti da trigger | `No Validation` |

---

## 5. DTO per Query e Stored Procedure

Quando una query o una stored procedure restituisce un risultato che non corrisponde a nessuna Entity esistente, l'autore deve creare un **DTO** nella cartella appropriata del servizio.

```
services/<NomeServizio>/src/.../
└── models/
    ├── query/
    │   ├── Get<Dominio>Response.java     ← risposta query SELECT custom
    │   └── <Dominio>Response.java
    └── procedure/
        ├── <Azione>Request.java          ← parametri IN della stored procedure
        └── <Azione>Response.java         ← parametri OUT della stored procedure
```

Il DTO deve contenere solo i campi restituiti dalla query o dalla procedura — non copiare l'intera Entity se servono solo alcuni campi.

**Esempio:** una query che restituisce `codice_articolo`, `descrizione` e `quantita_disponibile` richiede un DTO dedicato `ArticoloDisponibileResponse.java`, non il riuso dell'Entity `Articoli.java`.

---

## 6. Riepilogo Checklist ORM

- [ ] Ogni campo con valore auto-generato dal DB ha `Value Type → Database Defined`
- [ ] `creation_timestamp`: `Database Defined`, `Updatable OFF`, `Insertable OFF`
- [ ] `modification_timestamp`: `Database Defined` (trigger) o `Server Defined` (app), `Updatable OFF`
- [ ] Chiavi primarie auto-increment: `Database Defined`, `Updatable OFF`, `Insertable OFF`
- [ ] Campi compilati dall'utente: `User Defined`, `Updatable ON`, `Insertable ON`
- [ ] `deletion_timestamp`: `User Defined`, `Updatable ON`, `Insertable OFF`
- [ ] Validators configurati per campi con formato vincolato (Pattern, Min, Max)
- [ ] Campi auto-generati hanno `No Validation`
- [ ] DTO creati per ogni query/procedura che non restituisce un'Entity completa
- [ ] I DTO si trovano in `models/query/` o `models/procedure/` del servizio corretto
