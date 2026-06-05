---
title: Database
nav_order: 2
---

# Database Naming Conventions

Questa guida riassume le convenzioni di naming per tabelle, colonne, chiavi, indici, constraint, stored procedure, function, trigger, views e schemi, utile per mantenere coerenza e leggibilità tra diversi DB.

---

## 1. Formato Generale

* Minuscolo
* Separazione con underscore (snake_case)

**Esempi:**

* ✔ `user_account`
* ✔ `order_items`
* ❌ `UserAccount` (camelCase)
* ❌ `USERACCOUNT` (upper)

**Motivo:** Coerente, leggibile e portabile tra DB.

---

## 2. Prefisso di Progetto

Tabelle, colonne e schemi devono essere preceduti da un prefisso di tre caratteri composto da **lettere e numeri minuscoli**, seguito da underscore:

```
x0x_
```

Il prefisso identifica il progetto o il contesto applicativo e si applica a:

* Nomi di tabelle
* Nomi di colonne
* Nomi di schema

**Esempi:**

* ✔ `crm_users` (prefisso `crm`)
* ✔ `c3r_orders` (prefisso `c3r`)
* ✔ `c3r_order_id` (colonna con prefisso)
* ✔ Schema: `c3r` (schema prefissato)
* ❌ `users` (senza prefisso)
* ❌ `CRM_users` (maiuscolo)

**Motivo:** Evita collisioni di nomi in ambienti multi-tenant o multi-schema e rende immediatamente identificabile l'appartenenza di ogni oggetto.

---

## 3. Nomi di Tabelle

* Sempre al plurale perché le tabelle contengono insiemi.

**Esempi:**

* ✔ `c3r_users`
* ✔ `c3r_orders`
* ✔ `c3r_order_items`
* ❌ `user`
* ❌ `orderItem`

**Eccezioni:** Tabelle di lookup (es. `c3r_status_type`, `c3r_country`).

---

## 4. Colonne ID

* Chiave primaria = `id` (bigint autoincrementale per le specifiche base)
* Evitare generici `id` singoli, soprattutto in join complessi; in quel caso preferire `<table_name>_id`.

**Tipo del campo `id`:**

* **Specifiche base:** `id` è un `BIGINT` autoincrementale.
* **Progetti complessi:** valutare l'adozione di `UUID` al posto del bigint autoincrementale. Questa scelta va vagliata **progetto per progetto** in base a requisiti di scalabilità, distribuzione o interoperabilità.

**Esempi:**

* ✔ `id BIGINT AUTO_INCREMENT` in `c3r_users` (spec base)
* ✔ `id UUID DEFAULT gen_random_uuid()` in `c3r_events` (progetto complesso)
* ✔ `c3r_user_id` nella tabella `c3r_orders` (foreign key descrittiva)
* ✔ `c3r_product_id` nella tabella `c3r_order_items`

---

## 5. Foreign Key

* Usare esattamente il nome della primary key della tabella referenziata.

**Esempi:**

* ✔ `c3r_customer_id` → riferimento a `c3r_customers.id`
* ✔ `created_by_user_id` → se il riferimento non è banale

---

## 6. Constraints e Indici

**Primary Key:**

```
pk_<table>
```

Esempio: `pk_c3r_users`

**Foreign Key:**

```
fk_<table>_<column>
```

Esempio: `fk_c3r_orders_user_id`

**Unique Constraints:**

```
uq_<table>_<column>
```

**Check Constraints:**

```
ck_<table>_<column>_<rule>
```

**Indici:**

```
idx_<table>_<column1>[_column2]
```

---

## 7. Nomi delle Colonne

* Minuscolo
* Snake_case
* Descrittivi
* Evitare abbreviazioni eccessive
* Non usare `is_` per boolean

**Esempi:**

* ✔ `active` (boolean)
* ✔ `total_amount`
* ❌ `crt_at`, `upd`, `amt`, `tmp`

---

## 8. Timestamp Columns

* Standard comune:

  * `creation_timestamp`
  * `modification_timestamp`
  * `deletion_timestamp` (per soft delete)

attenzione ai contensti cloud in cui la timezone è UTC

---

## 9. Tabelle di relazione N–N

* Usare nome composto in ordine alfabetico.

**Esempi:**

* ✔ `c3r_product_category`
* ✔ `c3r_role_permission`
* ❌ `category_product` (se invertito senza criterio)

---

## 10. Stored Procedure, Function, Trigger

**Stored Procedure:**

```
sp_<module>_<action>
```

* ✔ `sp_user_create`
* ✔ `sp_order_recalculate_totals`

**Function:**

```
fn_<nome_logico>
```

**Trigger:**

```
trg_<table>_<timing>_<event>
```

* ✔ `trg_c3r_orders_before_insert`

---

## 11. Views

* Usare prefisso:

  * `v_<nome_view>` oppure `view_<nome_view>`
* ✔ `v_c3r_active_users`

---

## 12. Schema

* Minuscolo
* Nomi brevi e chiari, rispettando il formato prefisso `x0x`
* ✔ `c3r`
* ✔ `crm`
* ✔ `analytics`

---

## 13. Tipi di Campi

* Non usare campi `char`, preferire `varchar`.
