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

## 2. Nomi di Tabelle

* Sempre al plurale perché le tabelle contengono insiemi.

**Esempi:**

* ✔ `users`
* ✔ `orders`
* ✔ `order_items`
* ❌ `user`
* ❌ `orderItem`

**Eccezioni:** Tabelle di lookup (es. `status_type`, `country`).

---

## 3. Colonne ID

* Chiave primaria = `<table_name>_id`
* Evitare generici `id` singoli, soprattutto in join complessi.

**Esempi:**

* ✔ `user_id` in `users`
* ✔ `order_id` in `orders`
* ✔ `user_id` nella tabella `orders`
* ✔ `product_id` nella tabella `order_items`
* ❌ `id` da solo

---

## 4. Foreign Key

* Usare esattamente il nome della primary key della tabella referenziata.

**Esempi:**

* ✔ `customer_id` → riferimento a `customers.customer_id`
* ✔ `created_by_user_id` → se il riferimento non è banale

---

## 5. Constraints e Indici

**Primary Key:**

```
pk_<table>
```

Esempio: `pk_users`

**Foreign Key:**

```
fk_<table>_<column>
```

Esempio: `fk_orders_user_id`

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

## 6. Nomi delle Colonne

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

## 7. Timestamp Columns

* Standard comune:

  * `creation_timestamp`
  * `modification_timestamp`
  * `deletion_timestamp` (per soft delete)

attenzione ai contensti cloud in cui la timezone è UTC

---

## 8. Tabelle di relazione N–N

* Usare nome composto in ordine alfabetico.

**Esempi:**

* ✔ `product_category`
* ✔ `role_permission`
* ❌ `category_product` (se invertito senza criterio)

---

## 9. Stored Procedure, Function, Trigger

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

* ✔ `trg_orders_before_insert`

---

## 10. Views

* Usare prefisso:

  * `v_<nome_view>` oppure `view_<nome_view>`
* ✔ `v_active_users`

---

## 11. Schema

* Minuscolo
* Nomi brevi e chiari
* ✔ `public`
* ✔ `crm`
* ✔ `analytics`

---

## 12. Tipi di Campi

* Non usare campi `char`, preferire `varchar`.
