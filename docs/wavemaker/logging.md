---
title: Logging
parent: WaveMaker
nav_order: 6
---

# Logging — Convenzioni Log4j2 WaveMaker

WaveMaker configura il logging tramite Log4j2. Il file di configurazione si trova in:

```
src/main/resources/log4j2.xml
```

Il file è già presente nel progetto generato da WaveMaker con una configurazione base. Gli autori devono **estendere** questa configurazione per il codice custom, senza modificare i livelli dei logger di framework già definiti.

---

## 1. Configurazione Base — Non Modificare

La configurazione base fornita da WaveMaker imposta:

| Logger | Livello | Motivazione |
| ------ | ------- | ----------- |
| Root (tutto il codice) | `INFO` | Livello predefinito |
| `org.hibernate` | `WARN` | Evita il rumore generato da Hibernate (SQL, cache, transaction) |
| `org.springframework` | `WARN` | Evita il rumore del container Spring |
| `org.springframework.security` | `WARN` | Evita il dettaglio dei filtri di sicurezza |

**Non abbassare** i livelli di Hibernate, Spring o Spring Security a `DEBUG` o `TRACE` in produzione. Generano un volume di log che rende impossibile individuare i messaggi applicativi rilevanti.

Il pattern di output è:

```
%d{dd MMM yyyy HH:mm:ss,SSS} -%X{X-WM-Request-Track-Id} %t %p [%c] - %m%n
```

Produce righe del tipo:

```
31 May 2026 14:32:05,123 -abc-123 http-nio-8080-exec-1 INFO [com.oneclickapp.giglioli.ArticoliService] - Articolo MAT-0042 aggiornato correttamente
```

Non modificare il pattern senza una necessità documentata.

---

## 2. Aggiungere un Logger per Codice Custom

Ogni classe Java custom che produce log deve dichiarare il proprio logger con il nome della classe stessa:

```java
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

public class ArticoliServiceImpl implements ArticoliService {

    private static final Logger logger = LogManager.getLogger(ArticoliServiceImpl.class);

    public Articoli updateArticolo(Integer id, Articoli articolo) {
        logger.info("Aggiornamento articolo id={}", id);
        // ...
        logger.debug("Payload ricevuto: {}", articolo);
        // ...
    }
}
```

Non usare `System.out.println` o `e.printStackTrace()` nel codice di produzione.

---

## 3. Quando Usare Ogni Livello

| Livello | Quando usarlo | Esempi |
| ------- | ------------- | ------ |
| `ERROR` | Errore che impedisce il completamento dell'operazione e richiede intervento | Eccezione non gestita, connessione DB persa, file non trovato |
| `WARN` | Situazione anomala ma recuperabile, o configurazione subottimale | Retry in corso, valore fuori range ma accettato, deprecazione |
| `INFO` | Evento di business rilevante, inizio/fine operazione significativa | Record creato/aggiornato/eliminato, job avviato/completato, autenticazione riuscita |
| `DEBUG` | Dettaglio utile solo in fase di sviluppo e diagnostica | Payload di richiesta, valori intermedi, branch logici eseguiti |
| `TRACE` | Dettaglio estremo, mai in produzione | Ogni iterazione di un loop, ogni chiamata a metodo |

### Regola Anti-Rumore

> Un log `INFO` deve essere significativo per un operatore che monitora il sistema in produzione. Se un messaggio viene scritto decine di volte al secondo o non aggiunge informazioni utili, abbassarlo a `DEBUG`.

**Esempi di log corretti:**

```java
// INFO — evento di business rilevante
logger.info("Ordine {} creato per utente {}", orderId, userId);

// WARN — situazione anomala recuperabile
logger.warn("Tentativo di accesso a risorsa inesistente: {}", resourceId);

// ERROR — errore con contesto completo
logger.error("Errore durante il salvataggio dell'ordine id={}", orderId, e);

// DEBUG — solo per sviluppo/diagnostica
logger.debug("Parametri query: stato={}, dataInizio={}", stato, dataInizio);
```

**Esempi di log da evitare:**

```java
// Troppo generico — nessuna informazione utile
logger.info("Inizio metodo");
logger.info("Fine metodo");

// Rumore in loop
for (Articolo a : articoli) {
    logger.info("Processo articolo: {}", a.getCodice()); // ❌ se la lista è lunga
}

// Informazioni sensibili
logger.info("Login utente: {} password: {}", username, password); // ❌ MAI
```

---

## 4. Logger Custom in `log4j2.xml`

Se una classe custom richiede un livello di log diverso dal root `INFO`, aggiungere un logger specifico nel `log4j2.xml` con il package o la classe esatta:

```xml
<Loggers>
    <Root level="info">
        <AppenderRef ref="stdout"/>
    </Root>

    <!-- Framework — NON modificare -->
    <logger name="org.hibernate" level="WARN"/>
    <logger name="org.springframework" level="WARN"/>
    <logger name="org.springframework.security" level="WARN"/>

    <!-- Logger custom del progetto -->
    <logger name="com.oneclickapp.giglioli.CronService" level="DEBUG"/>
</Loggers>
```

I logger custom del progetto vanno aggiunti **dopo** i logger di framework, con un commento che indica il nome del progetto. Non mescolarli con i logger WaveMaker/Hibernate/Spring.

---

## 5. Logger di Debug per Sviluppo

I logger di framework utili durante lo sviluppo (SQL Hibernate, chiamate REST, pool di connessioni) sono già presenti nel `log4j2.xml` come commenti. Abilitarli temporaneamente in locale decommentando la riga, ma **non committarli abilitati**:

```xml
<!-- Abilitare solo in locale per diagnostica SQL — NON committare abilitato -->
<!--<logger name="org.hibernate.SQL" level="DEBUG"/>-->
<!--<logger name="com.wavemaker.runtime.rest" level="DEBUG"/>-->
<!--<logger name="com.zaxxer.hikari" level="DEBUG"/>-->
```

Se un logger di debug viene committato abilitato, la pipeline CI/CD deve rilevarlo come anomalia durante la code review.

---

## 6. Cosa Non Loggare Mai

* **Password, token, API key, secret** — in nessun livello di log
* **Dati personali (PII)** — nome completo, codice fiscale, email, numero di carta — mai in chiaro nei log
* **Payload completi di richieste HTTP** a livello `INFO` — usare `DEBUG` al massimo
* **Stack trace completi a livello `INFO`** — gli stack trace vanno a `ERROR` con l'eccezione passata come secondo argomento: `logger.error("msg", e)`

---

## 7. Checklist Logging

- [ ] Ogni classe custom ha il logger dichiarato come `private static final Logger logger = LogManager.getLogger(<Classe>.class)`
- [ ] Nessun `System.out.println` o `e.printStackTrace()` nel codice
- [ ] I livelli `org.hibernate`, `org.springframework`, `org.springframework.security` rimangono a `WARN`
- [ ] I log `INFO` sono eventi di business rilevanti, non tracce di ogni metodo chiamato
- [ ] Dati sensibili (password, token, PII) mai presenti nei log
- [ ] Logger di debug WaveMaker/Hibernate commentati nel `log4j2.xml` committato
- [ ] Logger custom del progetto aggiunti dopo i logger di framework, con commento identificativo
