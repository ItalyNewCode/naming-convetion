# REST Naming Conventions e Mapping WaveMaker

Questa guida descrive come WaveMaker costruisce nomi di metodi Java e API REST, con focus su booleani, mapping DB → Java → REST, e convenzioni CRUD.

---

## Pipeline di generazione WM

```
DB schema
 ↓
Entity JPA (Java)
 ↓
Repository (Spring Data)
 ↓
Service
 ↓
REST Controller
```

Le convenzioni derivano principalmente da:

* JavaBeans naming spec
* Spring Data JPA
* Spring MVC REST

---

## DB → Entity (Java)

### Campi non booleani

| SQL column | Java field | Getter         |
| ---------- | ---------- | -------------- |
| first_name | firstName  | getFirstName() |
| order_id   | orderId    | getOrderId()   |

**Regole:**

* snake_case → camelCase
* getter sempre getXxx()

### Campi BOOLEAN

| SQL column | Java field | Getter      |
| ---------- | ---------- | ----------- |
| active     | active     | isActive()  |
| enabled    | enabled    | isEnabled() |
| is_deleted | isDeleted  | isDeleted() |

**Nota:**

* Segue JavaBeans spec: boolean/Boolean → getter `isXxx()`
* NON usare `getIsXxx()`
* Fondamentale per compatibilità con Jackson, Spring, Hibernate

### JSON generato

```java
@Column(name = "active")
private Boolean active;

public Boolean isActive() {
   return active;
}
```

JSON:

```json
{
 "active": true
}
```

* Il prefisso `is` NON appare nel JSON

---

## Convenzioni Controller REST

WaveMaker mappa HTTP → Java senza esporre i nomi dei metodi Java nel path REST.

### CRUD standard per `Customer`

| HTTP   | Path           | Metodo Java       |
| ------ | -------------- | ----------------- |
| GET    | /Customer      | getCustomers()    |
| GET    | /Customer/{id} | getCustomerById() |
| POST   | /Customer      | createCustomer()  |
| PUT    | /Customer/{id} | editCustomer()    |
| DELETE | /Customer/{id} | deleteCustomer()  |

**Convenzione WM:**

* verbo HTTP → verbo semantico (`getXxx`, `createXxx`, `editXxx`, `deleteXxx`)

---

## Query automatiche (Spring Data style)

* Esempio DB: `active BOOLEAN`, `created_at TIMESTAMP`
* Repository generato:

```java
List<Customer> findByActive(Boolean active);
List<Customer> findByActiveAndCreatedAtAfter(Boolean active, LocalDateTime date);
```

* REST endpoint:

```
GET /Customer/search/findByActive?active=true
```

* Convenzioni: `findBy`, `And`, `Or`, `Before`, `After`, `Between`
* Boolean → nome del campo senza `is`

### Campi boolean nei filtri REST

```
GET /Customer?active=true
@QueryParam("active") Boolean active
```

* Mai `isActive` nel REST, solo `active`

---

## Motivo per isXxx() nei getter ma non nel REST

| Layer       | Standard       |
| ----------- | -------------- |
| Java Entity | JavaBeans      |
| REST        | JSON / OpenAPI |
| Repository  | Spring Data    |

* JavaBeans richiede `isXxx()` per boolean
* REST richiede nomi puliti senza prefissi

---

## Riassunto regole chiave

* Boolean:

  * SQL: `active`
  * Java field: `active`
  * Getter: `isActive()`
  * JSON / REST: `"active": true`
  * Query param: `active=true`
* Metodi Controller: GET → getXxx, POST → createXxx, PUT → editXxx, DELETE → deleteXxx
* Repository: `findByActive`, `findByActiveAndStatus`

### Caso colonne SQL con prefisso is_

* SQL: `is_active` → Java: `isActive` → Getter: `isActive()` → JSON: `isActive` ❗
* Ambiguità: WM consiglia colonne boolean senza `is_`

---

## Relazioni One-to-Many / Many-to-Many

* DB: `customer` / `address` → `address.customer_id`
* Java: `List<Address> addresses; getAddresses();`
* REST: `GET /Customer/{id}/addresses`

Repository example:

```java
List<Address> findByCity(String city);
```

---

## Principio architetturale WM

* SQL (snake_case) → Entity Java (camelCase) → REST API (camelCase)
* Esempio:

```
GET /Import?importFlowId=7
```

* Non usare snake_case nei query param REST

**Regola ufficiale:** nel layer REST di WM sempre camelCase, senza eccezioni.

### Dove vale camelCase

| Contesto API     | snake_case | camelCase |
| ---------------- | ---------- | --------- |
| Query param      | ❌          | ✅         |
| Body JSON        | ❌          | ✅         |
| Sort             | ❌          | ✅         |
| Search (findBy…) | ❌          | ✅         |
| Path param       | ❌          | ✅         |

### Dove vale snake_case

| Contesto  | snake_case |
| --------- | ---------- |
| SQL       | ✅          |
| DDL       | ✅          |
| Script DB | ✅          |
| View      | ✅          |
| Trigger   | ✅          |

---

## Gestione doppio stile senza confusione

1. Layer e stile:

   * DB → snake_case
   * Java → camelCase
   * REST → camelCase
2. Usa sempre lo stesso nome concettuale:

   * DB: `import_flow_id`
   * Java: `importFlowId`
   * REST: `?importFlowId=7`
3. Documenta l’API in camelCase (Swagger/OpenAPI, README, esempi curl)
4. Regola pratica:

> Se scrivi SQL, usa snake_case. Se chiami un’API, usa camelCase.
