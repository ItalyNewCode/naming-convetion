---
title: Ambienti di Progetto
nav_order: 5
---

# Ambienti di Progetto — Stage, Database e Application Server

Ogni progetto gestito dalla community deve essere strutturato su **almeno tre ambienti distinti** (stage). Questa separazione è obbligatoria indipendentemente dal fatto che il progetto sia ospitato on-premise o su cloud.

---

## 1. I Tre Stage Obbligatori

| Stage | Sigla | Scopo |
| ----- | ----- | ----- |
| **Sviluppo** | `DEV` | Sviluppo attivo, sperimentazione, debug locale |
| **Test** | `TEST` | Collaudo funzionale, test di integrazione, UAT |
| **Produzione** | `PROD` | Sistema live, dati reali, utenti finali |

Ogni stage deve avere:
* Un **database dedicato** — mai condividere database tra stage diversi
* Un **profilo di configurazione** separato (vedi [pom.xml](pom.md) e sezione 3)
* Un **URL di accesso** distinto
* **Credenziali distinte** — mai usare le stesse credenziali DB tra DEV, TEST e PROD

---

## 2. Database per Stage — Regole di Accesso

### Sviluppo (DEV)

Il database di sviluppo può essere **accessibile pubblicamente** per facilitare il lavoro degli sviluppatori in ambienti distribuiti o remoti.

| Requisito | DEV |
| --------- | --- |
| Accesso pubblico | ✅ Consentito |
| SSL/TLS obbligatorio | ⚪ Raccomandato, non obbligatorio |
| Accesso privato (VPN/rete interna) | ⚪ Opzionale |
| Dati reali | ❌ Vietato — usare dati sintetici o anonimizzati |
| Backup | ⚪ Opzionale |

Il database DEV non deve mai contenere dati reali di clienti, dati personali (PII) o dati finanziari. Usare dati sintetici o anonimizzati.

### Test / Staging (TEST)

Il database di test può contenere dati sensibili o copie anonimizzate del database di produzione. Deve essere accessibile **solo dalla rete interna o tramite VPN**.

| Requisito | TEST |
| --------- | ---- |
| Accesso pubblico | ❌ Vietato |
| SSL/TLS obbligatorio | ✅ Sì |
| Accesso privato (VPN/rete interna) | ✅ Obbligatorio |
| Dati reali | ⚠️ Solo se anonimizzati |
| Backup | ✅ Raccomandato |

```properties
# profiles/staging.properties
db.APP.url=jdbc:mariadb://db-test.interno:3306/APP_TEST?useSSL=true&requireSSL=true&verifyServerCertificate=true
```

### Produzione (PROD)

Il database di produzione contiene dati reali. L'accesso deve essere **esclusivamente privato**, con SSL/TLS e autenticazione forte.

| Requisito | PROD |
| --------- | ---- |
| Accesso pubblico | ❌ Assolutamente vietato |
| SSL/TLS obbligatorio | ✅ Sì |
| Accesso privato (VPN/rete interna) | ✅ Obbligatorio |
| Dati reali | ✅ Sì — proteggere con accesso ristretto |
| Backup | ✅ Obbligatorio con retention minima 30 giorni |
| Monitoring | ✅ Obbligatorio |

```properties
# profiles/prod.properties
db.APP.url=jdbc:mariadb://${DB_HOST}:${DB_PORT}/APP_PROD?useSSL=true&requireSSL=true&verifyServerCertificate=true
db.APP.username=${DB_USERNAME}
db.APP.password=${DB_PASSWORD}
```

Le credenziali di produzione non devono mai essere nel repository — usare variabili d'ambiente o secret manager (vedi [Sicurezza](security.md)).

---

## 3. Profili di Configurazione per Stage

```
profiles/
├── development.properties   ← Stage DEV
├── deployment.properties    ← Stage TEST / Staging
└── prod.properties          ← Stage PROD
```

| Stage | File profilo | Parametro build |
| ----- | ------------ | --------------- |
| DEV | `development.properties` | `-Pdevelopment` |
| TEST | `deployment.properties` | `-Pdeployment` |
| PROD | `prod.properties` | `-Pprod` |

---

## 4. Mapping Stage → Branch → Pipeline

| Branch | Stage | Deploy |
| ------ | ----- | ------ |
| `develop` | DEV | Manuale o automatico su push |
| `staging` | TEST | Automatico su merge in `staging` |
| `main` / `master` | PROD | Automatico su merge in `main` con approvazione |

Il branch `main` deve essere protetto: il merge in produzione richiede almeno una Pull Request approvata.

---

## 5. Application Server — Tomcat

| Tipo applicazione | Tomcat da usare | Dove trovare la versione |
| ----------------- | --------------- | ------------------------ |
| **WaveMaker** | Versione specificata nelle Release Notes | [Release Notes](wavemaker/release-notes.md) |
| **Java standard** | Tomcat 10.1.x (Jakarta EE 10) | Documentare nel `README.md` del progetto |

Per WaveMaker 11.x la versione certificata è **Tomcat 10.1.39**.

### Specifiche Minime da Documentare

```
| Parametro         | DEV     | TEST    | PROD    |
|-------------------|---------|---------|---------|
| Tomcat version    | 10.1.39 | 10.1.39 | 10.1.39 |
| Java version      | 21      | 21      | 21      |
| JVM heap min      | 512m    | 1g      | 2g      |
| JVM heap max      | 1g      | 2g      | 4g      |
| Max threads       | 50      | 100     | 200     |
| Context path      | /App    | /App    | /App    |
```

### Configurazione JVM per Produzione

```bash
JAVA_OPTS="-server \
  -Xms2g -Xmx4g \
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=200 \
  -Djava.security.egd=file:/dev/./urandom \
  -Dfile.encoding=UTF-8"
```

### Reverse Proxy (Raccomandato in Produzione)

```
Internet → Nginx (443 HTTPS) → Tomcat (8080 HTTP interno)
```

```properties
# application.properties — necessario quando si usa un proxy
server.forward-headers-strategy=NATIVE
```

---

## 6. Checklist Ambienti

### Stage e Database
- [ ] Tre stage definiti: DEV, TEST, PROD
- [ ] Ogni stage ha un database dedicato con credenziali distinte
- [ ] Il database DEV non contiene dati reali o PII
- [ ] Il database TEST è accessibile solo da rete privata/VPN con SSL/TLS
- [ ] Il database PROD è accessibile solo da rete privata/VPN con SSL/TLS
- [ ] Backup del database PROD con retention ≥ 30 giorni
- [ ] Le credenziali PROD non sono nel repository

### Profili e Pipeline
- [ ] Tre file di profilo presenti: `development.properties`, `deployment.properties`, `prod.properties`
- [ ] Il branch `main` è protetto e richiede PR approvata
- [ ] Il deploy in PROD avviene solo tramite pipeline

### Application Server
- [ ] Versione Tomcat documentata e allineata alla matrice WaveMaker
- [ ] Specifiche JVM documentate per ogni stage
- [ ] HTTPS attivo su TEST e PROD
- [ ] `X-Forwarded-Proto` configurato se si usa reverse proxy
