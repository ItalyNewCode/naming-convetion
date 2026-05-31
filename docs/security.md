# Sicurezza — Gestione dei Segreti e Protezione delle Informazioni

La sicurezza non è una funzionalità aggiuntiva: è un requisito di base di ogni applicazione enterprise. Questo documento definisce le regole obbligatorie per la gestione di credenziali, segreti e informazioni sensibili.

Riferimenti correlati: [SSL/TLS](ssl.md) · [Logging](wavemaker/logging.md) · [Continuous Delivery](cicd.md)

---

## 1. Principio Fondamentale — Zero Segreti nel Repository

> Nessun segreto deve mai essere committato nel repository Git, in nessuna forma.

Sono segreti:
* Password di database
* Credenziali di accesso a servizi esterni (API key, token OAuth, client secret)
* Certificati TLS privati e chiavi private
* Credenziali AWS (access key, secret key)
* Stringhe di connessione con credenziali incorporate
* Chiavi di cifratura e salt

Questo vale anche per:
* File di profilo (`profiles/prod.properties`)
* File di configurazione (`application.properties`, `log4j2.xml`)
* Script di deploy e pipeline CI/CD
* Commenti nel codice
* File di test

**Un segreto nel repository Git — anche se rimosso in seguito — rimane nella history e deve essere considerato compromesso. Va ruotato immediatamente.**

---

## 2. `.gitignore` — File da Escludere

Il repository deve escludere dal tracking tutti i file che possono contenere segreti:

```gitignore
# Profili con credenziali reali
profiles/prod.properties
profiles/staging.properties

# File di ambiente locale
.env
.env.local
*.env

# Certificati e chiavi private
*.pem
*.key
*.p12
*.jks
*.keystore

# File di credenziali cloud
.aws/credentials
```

I file di profilo esclusi devono esistere localmente ma non devono mai essere committati. La pipeline CI/CD li riceve tramite secret manager o variabili d'ambiente (vedi sezione 4).

---

## 3. Profili WaveMaker — Gestione delle Credenziali

### Il Problema dell'Offuscamento WaveMaker

WaveMaker cifra le password nei file di profilo con un algoritmo proprietario. Il risultato è una stringa esadecimale come:

```properties
# Questo NON è sicuro nel repository
db.GIGLIOLI_PREVENTIVI.password=7b6a43524a093d0d6f6b6e7b1351777d483e313a6b124c1a
```

Questa stringa è **offuscamento**, non cifratura sicura. Chiunque abbia accesso al repository e alle librerie WaveMaker può decifrare il valore originale. Non costituisce protezione sufficiente per un ambiente di produzione.

### La Soluzione Corretta — Placeholder da Variabile d'Ambiente

Il file di profilo deve contenere **placeholder** invece dei valori reali:

```properties
# profiles/prod.properties — CORRETTO
db.GIGLIOLI_PREVENTIVI.url=jdbc:mariadb://${DB_HOST}:${DB_PORT}/GIGLIOLI_PREVENTIVI?useUnicode=yes&characterEncoding=UTF-8
db.GIGLIOLI_PREVENTIVI.username=${DB_USERNAME}
db.GIGLIOLI_PREVENTIVI.password=${DB_PASSWORD}

db.EVENTI.url=jdbc:mariadb://${DB_HOST}:${DB_PORT}/EVENTI_PRODUCTION?useUnicode=yes&characterEncoding=UTF-8
db.EVENTI.username=${DB_USERNAME}
db.EVENTI.password=${DB_PASSWORD}
```

I valori `${DB_HOST}`, `${DB_USERNAME}`, `${DB_PASSWORD}` vengono risolti a runtime dall'ambiente di esecuzione (vedi sezione 4).

---

## 4. Secret Manager — Dove Vivono i Segreti

I segreti devono essere memorizzati in un **secret manager dedicato**, non in file di testo o variabili d'ambiente del sistema operativo del server.

### AWS Secrets Manager (per deploy su Elastic Beanstalk)

Per applicazioni deployate su AWS Elastic Beanstalk, i segreti vengono letti da **AWS Secrets Manager** o **AWS Systems Manager Parameter Store** e iniettati come variabili d'ambiente nell'ambiente EB.

**Flusso:**

```
AWS Secrets Manager
        │
        ▼
Elastic Beanstalk Environment Variables
        │
        ▼
JVM System Properties / Spring PropertyPlaceholderConfigurer
        │
        ▼
profiles/prod.properties  (${DB_PASSWORD} → valore reale)
```

**Configurazione nell'environment EB** (via console AWS o pipeline CI/CD):

```
DB_HOST     = srw-wm-prd.internal
DB_PORT     = 3306
DB_USERNAME = appuser
DB_PASSWORD = <valore letto da Secrets Manager>
```

Le variabili EB vengono esposte al processo JVM come proprietà di sistema, risolte da Spring al momento del caricamento del contesto.

### HashiCorp Vault (per deploy on-premise)

Per applicazioni su server on-premise, usare **HashiCorp Vault** come secret store centralizzato. I segreti vengono letti all'avvio dell'applicazione tramite l'agent Vault o un'integrazione Spring Cloud Vault.

```
HashiCorp Vault
        │
        ▼ (Vault Agent o Spring Cloud Vault)
Variabili d'ambiente del processo
        │
        ▼
profiles/prod.properties  (${DB_PASSWORD} → valore reale)
```

### Altre Opzioni Accettabili

| Piattaforma | Secret Manager |
| ----------- | -------------- |
| AWS | Secrets Manager, SSM Parameter Store |
| Azure | Azure Key Vault |
| GCP | Secret Manager |
| On-premise | HashiCorp Vault, CyberArk |
| Kubernetes | Kubernetes Secrets (cifrati at-rest con KMS) |

**Non accettabile:** file `.env` sul server, variabili d'ambiente impostate manualmente sul sistema operativo senza rotazione, file di testo con password in cartelle condivise.

---

## 5. Segreti nella Pipeline CI/CD

Le credenziali usate dalla pipeline (es. `AWS_ROLE_TO_ASSUME_FOR_DEPLOY`) devono essere memorizzate come **GitHub Secrets** nel repository (`Settings → Secrets and variables → Actions`).

Regole:
* Mai inserire credenziali direttamente nel file `.github/workflows/*.yml`
* Usare sempre `${{ secrets.NOME_SECRET }}` per referenziare i segreti
* Preferire autenticazione OIDC (ruolo IAM temporaneo) a credenziali statiche (vedi [cicd.md](cicd.md))
* Ruotare i segreti della pipeline almeno ogni 90 giorni

```yaml
# CORRETTO
secrets:
  AWS_ROLE_TO_ASSUME_FOR_DEPLOY: ${{ secrets.AWS_ROLE_TO_ASSUME_FOR_DEPLOY }}

# VIETATO
env:
  AWS_SECRET_ACCESS_KEY: "AKIAIOSFODNN7EXAMPLE..."
```

---

## 6. Protezione delle Informazioni nelle Risposte HTTP

L'applicazione non deve mai esporre informazioni interne nelle risposte di errore visibili al client.

### Stack Trace

Gli stack trace Java non devono mai essere restituiti al client in produzione. Configurare Spring per restituire solo un messaggio generico:

```properties
# application.properties
server.error.include-stacktrace=never
server.error.include-message=never
server.error.include-binding-errors=never
```

### Versioni e Tecnologie

Non esporre nei response header la versione del server, di WaveMaker o delle librerie:

```
# Header da rimuovere o oscurare
X-Powered-By: WaveMaker
Server: Apache-Coyote/1.1
X-WaveMaker-Version: 11.15.3
```

Configurare il server (Tomcat, Nginx, ALB) per sopprimere questi header.

### Messaggi di Errore

I messaggi di errore restituiti all'utente devono essere generici e in lingua. Non includere:
* Nomi di tabelle o colonne del database
* Query SQL
* Path interni del filesystem
* Indirizzi IP interni
* Nomi di host interni

```json
// VIETATO — espone dettagli interni
{
  "error": "Table 'giglioli.users' doesn't exist",
  "path": "/home/app/WEB-INF/classes/..."
}

// CORRETTO — messaggio generico
{
  "error": "Si è verificato un errore. Contattare l'assistenza."
}
```

---

## 7. OWASP Top 10 — Protezioni Obbligatorie in WaveMaker

Gli autori devono attivare e configurare correttamente tutti gli strumenti di sicurezza forniti da WaveMaker e Spring. Non è richiesto costruire meccanismi custom: la piattaforma li fornisce già — vanno usati, non disabilitati.

La tabella seguente mappa ogni categoria OWASP Top 10 (2021) agli strumenti WaveMaker disponibili.

| OWASP | Categoria | Strumento WaveMaker / Spring |
| ----- | --------- | ---------------------------- |
| A01 | Broken Access Control | WaveMaker Security Service, Spring Security roles |
| A02 | Cryptographic Failures | TLS obbligatorio, no password in chiaro, secret manager |
| A03 | Injection | JPA parametrized, CRLF encoding Log4j2 |
| A04 | Insecure Design | Validazione input, design review |
| A05 | Security Misconfiguration | Spring Security defaults, CORS, header HTTP |
| A06 | Vulnerable Components | SBOM, pom.xml versioni fisse |
| A07 | Auth & Session Failures | Spring Security session, timeout, CSRF |
| A08 | Integrity Failures | SBOM, firma artefatti, CI/CD OIDC |
| A09 | Logging & Monitoring | Log4j2 configurato, no sensitive data nei log |
| A10 | SSRF | Validazione URL nelle chiamate a servizi esterni |

---

### A01 — Broken Access Control

WaveMaker fornisce il **Security Service** con gestione di ruoli e permessi integrata. Ogni endpoint REST generato da WaveMaker è protetto dalla configurazione di sicurezza del servizio.

**Regole:**

* Definire i ruoli applicativi nel Security Service di WaveMaker e assegnarli agli endpoint.
* Non esporre endpoint con `.permitAll()` senza una valutazione esplicita del rischio.
* Le API di amministrazione devono richiedere un ruolo dedicato (`ROLE_ADMIN`), non essere accessibili a tutti gli utenti autenticati.
* Verificare che ogni endpoint REST custom in `project-user-spring.xml` sia coperto dalla configurazione di sicurezza.
* Non affidare mai il controllo degli accessi solo al frontend — ogni regola di accesso deve essere applicata lato server.

```java
// VIETATO — espone tutto senza controllo
http.authorizeRequests().antMatchers("/**").permitAll();

// CORRETTO — solo gli endpoint pubblici sono aperti
http.authorizeRequests()
    .antMatchers("/login", "/public/**").permitAll()
    .anyRequest().authenticated();
```

---

### A02 — Cryptographic Failures

* **TLS obbligatorio** su tutti gli ambienti esposti — vedi [ssl.md](ssl.md).
* **Password mai in chiaro** nei profili, nel codice o nei log — vedi sezione 3.
* **Segreti da secret manager** — vedi sezione 4.
* Non usare algoritmi deprecati: no MD5, no SHA-1 per hash di password. Usare BCrypt (già disponibile in Spring Security) per l'hashing delle credenziali utente.
* Non cifrare dati sensibili con algoritmi simmetrici custom — usare le primitive fornite da Java Cryptography Architecture (JCA).

```java
// CORRETTO — BCrypt già integrato in Spring Security
PasswordEncoder encoder = new BCryptPasswordEncoder(12);
String hash = encoder.encode(rawPassword);
```

---

### A03 — Injection

#### SQL Injection

WaveMaker genera query JPA con parametri named che prevengono SQL injection per default. Per query custom usare **sempre** parametri named:

```java
// CORRETTO
String hql = "FROM Articoli WHERE stato = :stato AND attivo = :attivo";
Query q = session.createQuery(hql);
q.setParameter("stato", stato);
q.setParameter("attivo", true);

// VIETATO — SQL injection
String hql = "FROM Articoli WHERE stato = '" + stato + "'";
```

#### XSS — Cross-Site Scripting

Angular (usato da WaveMaker) applica automaticamente l'encoding dell'output nei template. Non aggirare questa protezione:

* **Non usare `[innerHTML]`** o equivalenti con dati non sanitizzati.
* **Non renderizzare HTML grezzo** da database o input utente in widget WaveMaker.
* Usare sempre binding su `label`, `text`, `span` — mai su elementi HTML raw.

```html
<!-- CORRETTO — Angular escapa automaticamente -->
<wm-label caption="{{variabile}}"></wm-label>

<!-- VIETATO — bypassa l'encoding -->
<div [innerHTML]="variabile"></div>
```

#### Log Injection

Il pattern Log4j2 configurato da WaveMaker include `%encode{%m}{CRLF}` che previene la manipolazione dei log tramite sequenze di a capo. Non rimuovere questo encoding dal pattern in `log4j2.xml`.

Non loggare mai input utente non sanitizzato a livello `INFO` o superiore — vedi [logging.md](wavemaker/logging.md).

---

### A04 — Insecure Design

* Ogni form deve validare l'input sia lato frontend (UX) sia lato backend (sicurezza) — vedi [frontend.md](wavemaker/frontend.md).
* Le operazioni distruttive richiedono conferma esplicita e autorizzazione server-side.
* I dati sensibili (PII, dati finanziari) devono essere identificati nel design e trattati con protezioni aggiuntive (cifratura at-rest, accesso ristretto per ruolo).
* Effettuare una revisione di sicurezza prima di ogni release in produzione (vedi checklist in fondo).

---

### A05 — Security Misconfiguration

#### CORS — Cross-Origin Resource Sharing

Configurare il CORS in modo restrittivo in `project-user-spring.xml`. **Mai usare `*` come `allowedOrigins` in produzione.**

```xml
<bean id="corsFilter" class="org.springframework.web.filter.CorsFilter">
    <constructor-arg>
        <bean class="org.springframework.web.cors.UrlBasedCorsConfigurationSource">
            <property name="corsConfigurations">
                <map>
                    <entry key="/**">
                        <bean class="org.springframework.web.cors.CorsConfiguration">
                            <property name="allowedOrigins">
                                <list>
                                    <!-- SOLO origini autorizzate, mai "*" -->
                                    <value>https://app.esempio.it</value>
                                </list>
                            </property>
                            <property name="allowedMethods">
                                <list>
                                    <value>GET</value>
                                    <value>POST</value>
                                    <value>PUT</value>
                                    <value>DELETE</value>
                                    <value>OPTIONS</value>
                                </list>
                            </property>
                            <property name="allowedHeaders">
                                <list>
                                    <value>Authorization</value>
                                    <value>Content-Type</value>
                                    <value>X-WM-Request-Track-Id</value>
                                </list>
                            </property>
                            <property name="allowCredentials" value="true"/>
                            <property name="maxAge" value="3600"/>
                        </bean>
                    </entry>
                </map>
            </property>
        </bean>
    </constructor-arg>
</bean>
```

| Parametro CORS | ✅ Produzione | ❌ Vietato |
| -------------- | ------------- | ---------- |
| `allowedOrigins` | Lista esplicita di domini HTTPS | `*` |
| `allowedMethods` | Solo i metodi usati (`GET`, `POST`, ...) | `*` |
| `allowCredentials: true` | Solo con origini esplicite | Con `allowedOrigins: *` |

#### Header di Sicurezza HTTP

Spring Security aggiunge automaticamente questi header. Verificare che siano presenti nelle risposte in produzione e che il reverse proxy (Nginx, ALB) non li rimuova:

| Header | Valore | Protezione |
| ------ | ------ | ---------- |
| `X-Content-Type-Options` | `nosniff` | Blocca MIME sniffing |
| `X-Frame-Options` | `DENY` | Blocca clickjacking |
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` | Forza HTTPS |
| `Content-Security-Policy` | `default-src 'self'; script-src 'self'` | Riduce superficie XSS |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Limita leakage URL |

Per aggiungere o personalizzare header in Spring Security:

```xml
<!-- project-user-spring.xml -->
<bean id="securityHeadersFilter"
      class="org.springframework.security.web.header.HeaderWriterFilter">
    <constructor-arg>
        <list>
            <bean class="org.springframework.security.web.header.writers.ContentSecurityPolicyHeaderWriter">
                <constructor-arg value="default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'"/>
            </bean>
            <bean class="org.springframework.security.web.header.writers.ReferrerPolicyHeaderWriter">
                <constructor-arg>
                    <value type="org.springframework.security.web.header.writers.ReferrerPolicyHeaderWriter$ReferrerPolicy">
                        STRICT_ORIGIN_WHEN_CROSS_ORIGIN
                    </value>
                </constructor-arg>
            </bean>
        </list>
    </constructor-arg>
</bean>
```

#### Informazioni di Versione

Non esporre versioni nei response header. Configurare in `application.properties`:

```properties
server.error.include-stacktrace=never
server.error.include-message=never
server.error.include-binding-errors=never
```

---

### A06 — Vulnerable and Outdated Components

* Usare **sempre versioni esatte** nel `pom.xml` — vedi [pom.md](pom.md).
* Generare la **SBOM** ad ogni release e scansionarla con Grype o OWASP Dependency-Check — vedi [sbom.md](sbom.md).
* Aggiornare WaveMaker alla versione più recente seguendo la matrice di compatibilità — vedi [release-notes.md](wavemaker/release-notes.md).
* Nessuna dipendenza con versione `LATEST`, `SNAPSHOT` o range aperto.

---

### A07 — Identification and Authentication Failures

#### CSRF — Cross-Site Request Forgery

WaveMaker abilita la protezione CSRF di Spring Security per default. Non disabilitarla nelle app con sessione browser.

```java
// VIETATO nelle app web con sessione
http.csrf().disable();
```

Se l'app espone API stateless con JWT per client mobile o SPA esterni, la disabilitazione CSRF è tecnicamente giustificata ma deve essere documentata con un commento nel file di configurazione.

#### Sessione e Timeout

Configurare un timeout di sessione appropriato in `application.properties`:

```properties
# Timeout sessione — 30 minuti di inattività
server.servlet.session.timeout=30m
```

Non usare timeout eccessivamente lunghi (oltre 8 ore) per applicazioni che gestiscono dati sensibili.

#### Password Policy

Se l'applicazione gestisce utenti propri (non delegando a un IdP esterno), le password devono:

* Avere lunghezza minima di 12 caratteri.
* Essere hashate con BCrypt (già in Spring Security) — mai MD5 o SHA-1.
* Non essere mai loggabili in nessun livello di log.

---

### A08 — Software and Data Integrity Failures

* **SBOM obbligatoria** ad ogni release — verifica che nessun componente sia stato alterato rispetto al `pom.xml` dichiarato (vedi [sbom.md](sbom.md)).
* **Pipeline CI/CD con OIDC** — evitare credenziali statiche; usare ruoli IAM temporanei (vedi [cicd.md](cicd.md)).
* **Firmare i tag di release** con GPG se il progetto lo richiede.
* Non scaricare dipendenze da repository non verificati — usare solo Maven Central o un repository Nexus/Artifactory aziendale.

---

### A09 — Security Logging and Monitoring Failures

* Ogni errore di autenticazione fallita deve essere loggato a livello `WARN` con IP e username (senza password).
* Ogni accesso a risorse protette negato (403) deve essere loggato.
* I log devono essere centralizzati e monitorati — non solo scritti su file locale.
* Non loggare mai dati sensibili: password, token, PII, numeri di carta — vedi [logging.md](wavemaker/logging.md).

```java
// CORRETTO — log di autenticazione fallita
logger.warn("Tentativo di accesso negato per utente '{}' da IP {}", username, remoteIp);

// VIETATO — include la password
logger.warn("Login fallito: utente='{}' password='{}'", username, password);
```

---

### A10 — Server-Side Request Forgery (SSRF)

Se i servizi Java custom effettuano chiamate HTTP verso URL forniti dall'utente o da configurazione dinamica, validare sempre l'URL prima di eseguire la chiamata:

* Consentire solo schemi `https://` — mai `file://`, `ftp://`, `gopher://`.
* Mantenere una lista esplicita di host/domini autorizzati (`allowlist`).
* Non seguire redirect automaticamente verso host non nella allowlist.
* Non esporre il body delle risposte di errore di chiamate interne verso il client.

```java
// CORRETTO — validazione prima della chiamata
private static final Set<String> ALLOWED_HOSTS = Set.of("api.esempio.it", "ssl.gigliolispa.it");

public void callExternalService(String url) {
    URI uri = URI.create(url);
    if (!ALLOWED_HOSTS.contains(uri.getHost())) {
        throw new SecurityException("Host non autorizzato: " + uri.getHost());
    }
    // esegui la chiamata
}
```

---

## 8. Spring Security — Non Indebolire la Configurazione WaveMaker

WaveMaker configura Spring Security con impostazioni sicure per default. Le modifiche alla configurazione di sicurezza devono essere documentate e giustificate.

In particolare è vietato:
* Disabilitare CSRF senza documentazione (app web con sessione)
* Aprire endpoint con `.permitAll()` senza valutazione del rischio
* Abbassare `org.springframework.security` a `DEBUG` in produzione
* Disabilitare HTTPS redirect
* Rimuovere header di sicurezza HTTP aggiunti da Spring Security

---

## 9. Rotazione dei Segreti

I segreti non sono statici. Devono essere ruotati periodicamente e immediatamente in caso di compromissione.

| Tipo di segreto | Frequenza di rotazione |
| --------------- | ---------------------- |
| Password database | Ogni 90 giorni |
| API key servizi esterni | Ogni 90 giorni |
| Credenziali CI/CD | Ogni 90 giorni |
| Certificati TLS | Prima della scadenza (rinnovo automatico con Let's Encrypt) |
| Chiavi di cifratura | Ogni 12 mesi o in caso di compromissione |

In caso di compromissione accertata o sospetta, la rotazione è immediata — non si aspetta la scadenza pianificata.

---

## 9. Checklist Sicurezza

### Segreti e Credenziali
- [ ] Nessuna password, token o chiave in chiaro nel repository (inclusa la git history)
- [ ] File `profiles/prod.properties` e `profiles/staging.properties` in `.gitignore`
- [ ] I placeholder `${VAR}` nei profili sono risolti da variabili d'ambiente o secret manager
- [ ] I segreti della pipeline sono in GitHub Secrets, mai hardcoded nei workflow YAML
- [ ] Preferita autenticazione OIDC a credenziali statiche AWS

### Esposizione delle Informazioni
- [ ] `server.error.include-stacktrace=never` in produzione
- [ ] Stack trace non restituiti al client nelle risposte HTTP
- [ ] Messaggi di errore generici, senza dettagli di tabelle, query o path interni
- [ ] Header `Server`, `X-Powered-By` soppressi o oscurati
- [ ] Dati sensibili (PII, password) mai presenti nei log (vedi [logging.md](wavemaker/logging.md))

### OWASP — Copertura Completa
- [ ] **A01** Endpoint protetti da ruoli WaveMaker Security Service — nessun `.permitAll()` non valutato
- [ ] **A02** TLS abilitato, BCrypt per password utente, nessun MD5/SHA-1
- [ ] **A03** Query HQL/JPA con parametri named — nessuna concatenazione di stringa; no `[innerHTML]` con dati non sanitizzati; `%encode{%m}{CRLF}` mantenuto in log4j2.xml
- [ ] **A04** Validazione input lato server per ogni form e API custom
- [ ] **A05** CORS con `allowedOrigins` espliciti (mai `*`); header HTTP di sicurezza presenti e verificati; stack trace non esposti
- [ ] **A06** SBOM generata e scansionata; dipendenze con versione esatta nel pom.xml
- [ ] **A07** CSRF attivo nelle app web con sessione; session timeout ≤ 30 minuti; password BCrypt
- [ ] **A08** Pipeline CI/CD con OIDC; dipendenze solo da repository verificati
- [ ] **A09** Errori 401/403 loggati a WARN; nessun dato sensibile nei log
- [ ] **A10** Chiamate a servizi esterni validate contro allowlist di host autorizzati

### Infrastruttura
- [ ] SSL/TLS abilitato e test SSL Labs superato con grado A (vedi [ssl.md](ssl.md))
- [ ] Rotazione dei segreti pianificata (90 giorni per credenziali, prima della scadenza per certificati)
- [ ] Spring Security non modificata o indebolita rispetto alla configurazione WaveMaker
