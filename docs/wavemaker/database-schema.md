# Schema del Database nel Repository

Ogni progetto WaveMaker deve includere nel repository lo schema SQL del database collegato. Lo schema è la fonte di verità della struttura dati: permette di ricreare il database da zero, documentare l'evoluzione del modello e garantire la riproducibilità dell'ambiente.

---

## 1. Regola Fondamentale

> Lo schema SQL del database **deve essere presente nel repository** del progetto. Un repository senza schema non è considerato completo.

Lo schema deve riflettere lo stato attuale del database di produzione. Deve essere aggiornato ogni volta che la struttura del database cambia.

---

## 2. Posizione dei File

La community mette a disposizione la struttura di cartelle standard sotto il path:

```
src/main/webapp/resources/db/
```

Dentro questa cartella, ogni database ha la propria sottocartella identificata dal tipo di database:

```
src/main/webapp/resources/db/
├── mariadb/
│   └── schema.sql
├── mysql/
│   └── schema.sql
├── postgresql/
│   └── schema.sql
├── oracle/
│   └── schema.sql
└── sqlserver/
    └── schema.sql
```

L'autore crea **solo la sottocartella del database effettivamente usato** dal progetto. Le altre rimangono assenti.

### Identificatori di Database Supportati

| Identificatore cartella | Database |
| ----------------------- | -------- |
| `mariadb` | MariaDB (qualsiasi versione) |
| `mysql` | MySQL (qualsiasi versione) |
| `postgresql` | PostgreSQL |
| `oracle` | Oracle Database |
| `sqlserver` | Microsoft SQL Server |
| `hsqldb` | HSQLDB (solo per sviluppo/test embedded) |

---

## 3. Contenuto del File `schema.sql`

Il file deve contenere, nell'ordine:

1. **DROP / CREATE dello schema** (opzionale ma consigliato per ambienti di sviluppo)
2. **CREATE TABLE** per ogni tabella, con tutti i campi, tipi e vincoli
3. **PRIMARY KEY** e **AUTO_INCREMENT** / **SEQUENCE**
4. **FOREIGN KEY** con le relative regole di cascade
5. **UNIQUE constraint**
6. **CHECK constraint**
7. **INDEX** aggiuntivi (non già coperti da PK/FK)
8. **VIEW** (se presenti)
9. **STORED PROCEDURE e FUNCTION** (se presenti)
10. **TRIGGER** (se presenti)
11. **Dati iniziali obbligatori** (lookup table, valori di configurazione fissi)

Il naming di tutti gli oggetti SQL deve rispettare le convenzioni definite in [database.md](../database.md).

### Esempio — MariaDB

```sql
-- ============================================================
-- Schema: giglioli
-- Database: MariaDB 10.11+
-- Aggiornato: 2026-05-31
-- ============================================================

CREATE DATABASE IF NOT EXISTS giglioli
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;

USE giglioli;

-- ------------------------------------------------------------
-- Tabella: users
-- ------------------------------------------------------------
CREATE TABLE IF NOT EXISTS users (
    user_id            INT UNSIGNED    NOT NULL AUTO_INCREMENT,
    username           VARCHAR(100)    NOT NULL,
    email              VARCHAR(255)    NOT NULL,
    active             BOOLEAN         NOT NULL DEFAULT TRUE,
    creation_timestamp DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP,
    modification_timestamp DATETIME    NOT NULL DEFAULT CURRENT_TIMESTAMP
                                       ON UPDATE CURRENT_TIMESTAMP,
    CONSTRAINT pk_users        PRIMARY KEY (user_id),
    CONSTRAINT uq_users_email  UNIQUE (email)
);

-- ------------------------------------------------------------
-- Tabella: orders
-- ------------------------------------------------------------
CREATE TABLE IF NOT EXISTS orders (
    order_id           INT UNSIGNED    NOT NULL AUTO_INCREMENT,
    user_id            INT UNSIGNED    NOT NULL,
    total_amount       DECIMAL(12,2)   NOT NULL DEFAULT 0.00,
    creation_timestamp DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP,
    modification_timestamp DATETIME    NOT NULL DEFAULT CURRENT_TIMESTAMP
                                       ON UPDATE CURRENT_TIMESTAMP,
    CONSTRAINT pk_orders           PRIMARY KEY (order_id),
    CONSTRAINT fk_orders_user_id   FOREIGN KEY (user_id)
        REFERENCES users (user_id) ON DELETE RESTRICT ON UPDATE CASCADE
);

CREATE INDEX idx_orders_user_id ON orders (user_id);
```

---

## 4. Aggiornamento dello Schema

Lo schema SQL nel repository deve essere aggiornato **contestualmente** a ogni modifica della struttura del database, nella stessa Pull Request che introduce la modifica.

Non è accettabile un repository in cui lo schema SQL è disallineato rispetto al database di produzione.

### Modifiche Additive (retrocompatibili)

Aggiungere in fondo al file `schema.sql` le istruzioni `ALTER TABLE`, `CREATE INDEX`, ecc., con un commento che indica la versione o la data:

```sql
-- ------------------------------------------------------------
-- Modifica 2026-06-15 — aggiunta colonna note a orders
-- ------------------------------------------------------------
ALTER TABLE orders
    ADD COLUMN note TEXT NULL
    AFTER total_amount;
```

### Modifiche Distruttive

Per `DROP TABLE`, `DROP COLUMN`, rinominare tabelle o colonne, documentare la modifica con un commento esplicito e aggiornare le istruzioni `CREATE TABLE` corrispondenti in cima al file.

---

## 5. Schema e ORM — Allineamento Obbligatorio

Lo schema SQL nel repository e la configurazione ORM in WaveMaker devono essere allineati. In particolare:

* I campi dichiarati `DEFAULT CURRENT_TIMESTAMP` nel SQL devono avere `Value Type → Database Defined` nell'ORM (vedi [orm.md](orm.md)).
* I campi `AUTO_INCREMENT` o `SEQUENCE` devono avere `Value Type → Database Defined`, `Updatable OFF`, `Insertable OFF`.
* I campi `NOT NULL` senza `DEFAULT` devono avere il validatore `Required` nell'ORM.

---

## 6. Struttura di Cartelle da Creare

La community fornisce questa struttura base da copiare in ogni nuovo progetto WaveMaker. Creare manualmente la cartella del proprio database e aggiungere il file `schema.sql`.

```
src/main/webapp/resources/db/
└── <tipo-database>/
    └── schema.sql
```

Il file `schema.sql` deve avere in testa un blocco di commento con:

```sql
-- Progetto:  <NomeApp>
-- Database:  <TipoDatabase> <Versione>
-- Schema:    <NomeSchema>
-- Creato:    <YYYY-MM-DD>
-- Aggiornato: <YYYY-MM-DD>
```

---

## 7. Checklist Schema Database

- [ ] Cartella `src/main/webapp/resources/db/<tipo-database>/` presente nel repository
- [ ] File `schema.sql` presente e aggiornato allo stato attuale del DB di produzione
- [ ] Il file include: tabelle, PK, FK, constraint, indici, view, procedure, trigger
- [ ] Ogni oggetto SQL rispetta le convenzioni di naming in [database.md](../database.md)
- [ ] Il file ha il blocco di commento di intestazione con progetto, database e date
- [ ] Lo schema è aggiornato nella stessa PR che modifica il database
- [ ] I campi auto-generati nel SQL corrispondono a `Database Defined` nell'ORM
