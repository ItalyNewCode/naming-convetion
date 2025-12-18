Mappatura dei campi e dei metodi creati da WM  

WM costruisce i nomi dei metodi Java e delle API REST, con focus su boolean, GET/POST, e mapping DB → Java → REST. 

 

Pipeline di generazione WM 

Quando WaveMaker crea le API: 

DB schema 
 ↓ 
Entity JPA (Java) 
 ↓ 
Repository (Spring Data) 
 ↓ 
Service 
 ↓ 
REST Controller 
 

Le convenzioni dei nomi derivano principalmente da: 

JavaBeans naming spec 

Spring Data JPA 

Spring MVC REST 

 

Convenzioni DB → Entity (Java) 

🔹 Campi non booleani 

SQL column 

Java field 

Getter 

first_name 

firstName 

getFirstName() 

order_id 

orderId 

getOrderId() 

Regole: 

snake_case → camelCase 

getter sempre getXxx() 

 

🔹 Campi BOOLEAN 

SQL column 

Java field 

Getter 

active 

active 

isActive() 

enabled 

enabled 

isEnabled() 

is_deleted 

isDeleted 

isDeleted() 

WaveMaker segue JavaBeans spec: 

se il tipo è boolean / Boolean 

il getter è isXxx() 

NON getIsXxx() 

📌 Questo è fondamentale perché: 

Jackson (JSON) 

Spring 

Hibernate 
usano JavaBeans introspection 

 

Entity → REST JSON 

Esempio: 

@Column(name = "active") 
private Boolean active; 
 
public Boolean isActive() { 
   return active; 
} 
 

JSON prodotto: 

{ 
 "active": true 
} 
 

NOTE: Il prefisso is NON appare nel JSON, solo nel metodo Java. 

 

Convenzioni Controller REST (nomi metodi) 

WaveMaker non espone i nomi dei metodi Java nel path REST, ma usa mapping Spring. 

🔹 CRUD standard 

Per una entity Customer: 

HTTP 

Path 

Metodo Java 

GET 

/Customer 

getCustomers() 

GET 

/Customer/{id} 

getCustomerById() 

POST 

/Customer 

createCustomer() 

PUT 

/Customer/{id} 

editCustomer() 

DELETE 

/Customer/{id} 

deleteCustomer() 

📌 Convenzione WM: 

verbo HTTP → verbo semantico 

getXxx 

createXxx 

editXxx 

deleteXxx 

 

5️⃣ Query automatiche (Spring Data style) 

Se in DB hai: 

active BOOLEAN 
created_at TIMESTAMP 
 

WM genera repository tipo: 

List<Customer> findByActive(Boolean active); 
List<Customer> findByActiveAndCreatedAtAfter(Boolean active, LocalDateTime date); 
 

E REST endpoint: 

GET /Customer/search/findByActive?active=true 
 

📌 Convenzioni: 

findBy 

And, Or 

Before, After, Between 

per boolean usa il nome del campo senza is 

 

Campi boolean nei filtri REST 

Esempio: 

GET /Customer?active=true 
 

WM mappa: 

@QueryParam("active") Boolean active 
 

Mai isActive nel REST, solo active 

 

7️⃣ Perché WM aggiunge is nei getter ma non nei REST? 

Perché segue 3 standard diversi: 

Layer 

Standard 

Java Entity 

JavaBeans 

REST 

JSON / OpenAPI 

Repository 

Spring Data 

👉 JavaBeans richiede: 

isXxx() per boolean 

👉 REST richiede: 

nomi campi puliti, senza prefissi 

 

8️⃣ Riassunto regole chiave 

Boolean 

SQL: active 

Java field: active 

Getter: isActive() 

JSON / REST: "active": true 

Query param: active=true 

Metodi Controller 

GET → getXxx 

POST → createXxx 

PUT → editXxx 

DELETE → deleteXxx 

Repository 

findByActive 

findByActiveAndStatus 

 

9️⃣ Caso particolare: colonne SQL che iniziano con is_ 

SQL 

Java 

is_active 

isActive 

Getter 

isActive() 

JSON 

isActive ❗ 

⚠️ Questo può creare ambiguità 
➡️ WM consiglia colonne boolean senza is_ nel nome SQL 

1️⃣ Contesti in cui WM può aggiungere es 

🔹 A) Relazioni One-to-Many / Many-to-Many 

DB: 

customer 
address 
 

Relazione: 

address.customer_id 
 

Java generato: 

List<Address> addresses; 
getAddresses(); 
 

➡️ address → addresses (es perché termina in ss) 

 

🔹 B) Metodi REST che restituiscono collezioni 

GET /Customer 
GET /Customer/{id}/addresses 
 

Java: 

List<Customer> getCustomers(); 
List<Address> getAddresses(); 
 

 

🔹 C) Repository / Search methods 

List<Address> findByCity(String city); 

 

 

WaveMaker ti obbliga consapevolmente a usare due stili diversi, ed è importante accettarlo e governarlo, non cercare di “uniformarlo”. 

Qui sotto trovi la spiegazione ufficiale + la regola di progetto, così non resta una conoscenza “tribale”. 

 

📌 Perché succede (motivo architetturale) 

WaveMaker non espone il database ma il modello Java. 

La catena è rigida: 

SQL (snake_case) 
  ↓ 
Entity Java (camelCase) 
  ↓ 
REST API (camelCase) 
 

Quando chiami un’API con filtri: 

GET /Import?importFlowId=7 
 

WaveMaker: 

legge importFlowId 

lo risolve come property Java 

lo traduce internamente in colonna SQL import_flow_id 

Se scrivi: 

GET /Import?import_flow_id=7 
 

❌ fallisce perché nel modello Java non esiste import_flow_id. 

 

✅ Regola UFFICIALE di progetto (da mettere per iscritto) 

Nel layer REST di WaveMaker si usano SEMPRE i nomi Java (camelCase), 
anche se il database usa snake_case. 

Questa regola non ha eccezioni. 

 

📍 Dove vale SEMPRE camelCase 

Contesto API 

snake_case 

camelCase 

Query param 

❌ 

✅ 

Body JSON 

❌ 

✅ 

Sort 

❌ 

✅ 

Search (findBy…) 

❌ 

✅ 

Path param (campi) 

❌ 

✅ 

 

📍 Dove vale snake_case 

Contesto 

snake_case 

SQL 

 

DDL 

 

Script DB 

 

View 

 

Trigger 

 

 

🧠 Perché WM non supporta entrambi 

Perché: 

Spring MVC lavora su property Java 

Jackson serializza property Java 

Spring Data risolve query su property Java 

Supportare entrambi significherebbe: 

duplicare mapping 

rompere introspezione JavaBeans 

aumentare ambiguità 

WaveMaker sceglie coerenza tecnica, non comodità apparente. 

 

🧩 Come gestire il “doppio stile” senza impazzire 

1️⃣ Accetta che lo stile è legato al layer 

Layer 

Stile 

DB 

snake_case 

Java 

camelCase 

REST 

camelCase 

👉 Non mischiarli mai. 

 

2️⃣ Usa sempre lo stesso nome concettuale 

import_flow_id 
 

importFlowId 
 

?importFlowId=7 
 

La radice semantica è la stessa, cambia solo lo stile. 

 

3️⃣ Documenta l’API in camelCase (non il DB) 

Swagger / OpenAPI 

README 

esempi curl 

Devono riflettere il REST, non il DB. 

 

4️⃣ Regola pratica per il team 

“Se stai scrivendo SQL, usa snake_case. 
Se stai chiamando un’API, usa camelCase.” 

 
 

 

 

 
