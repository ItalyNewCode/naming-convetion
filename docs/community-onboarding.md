---
title: Onboarding Community
nav_order: 50
---

# Discovery Questions — Software Community Onboarding

Checklist di domande da porre a un'azienda con sviluppo software interno che vuole entrare in una community focalizzata sul miglioramento dello stack e dei processi.

> **Come usarle:** segui l'ordine delle sezioni. Le prime domande sono fattuali e rompono il ghiaccio; le ultime sono strategiche e richiedono che l'interlocutore si sia già aperto.

---

## 1. Infrastruttura & Cloud

- Usate cloud pubblico, privato, on-premise o una combinazione ibrida?
- Su quali provider siete? (AWS, Azure, GCP, OVH, Hetzner, altro)
- Avete una strategia multi-cloud o siete single-vendor?
- Usate Kubernetes? Self-managed o managed (EKS, AKS, GKE…)?
- Come gestite provisioning e Infrastructure as Code? (Terraform, Pulumi, Ansible, CloudFormation…)
- Avete ambienti separati e isolati per dev / staging / prod?
- Usate container? (Docker, Podman) e un registry privato?

---

## 2. Backend — Linguaggi & Framework

### Linguaggi principali
- Quale linguaggio usate principalmente lato backend?
  - [ ] Java / Kotlin
  - [ ] PHP
  - [ ] Python
  - [ ] Node.js / TypeScript
  - [ ] Go
  - [ ] .NET (C#)
  - [ ] Ruby
  - [ ] Altro: ___________

### Framework backend
- Quale framework backend usate?

| Linguaggio | Framework comuni da chiedere |
|---|---|
| Java / Kotlin | Spring Boot, Quarkus, Micronaut |
| PHP | Laravel, Symfony, WordPress (headless) |
| Python | Django, FastAPI, Flask |
| Node.js | Express, NestJS, Fastify |
| Go | Gin, Echo, Fiber |
| .NET | ASP.NET Core, Minimal API |
| Ruby | Rails, Sinatra |

- Usate un'architettura a microservizi, monolite modulare o monolite classico?
- Avete un API layer REST, GraphQL o gRPC?
- Come gestite l'autenticazione/autorizzazione? (JWT, OAuth2, Keycloak, Auth0…)

---

## 3. Frontend — Linguaggi & Framework

### Framework / librerie UI
- Quale framework usate lato frontend?
  - [ ] React
  - [ ] Angular
  - [ ] Vue.js
  - [ ] Svelte / SvelteKit
  - [ ] Next.js (React SSR)
  - [ ] Nuxt.js (Vue SSR)
  - [ ] Nessun framework (vanilla JS / jQuery)
  - [ ] Altro: ___________

### Domande di approfondimento frontend
- Usate un design system o component library interno? (MUI, Ant Design, PrimeNG, Tailwind…)
- Come gestite lo state management? (Redux, Zustand, Pinia, NgRx…)
- Avete applicazioni mobile native, ibride o PWA? (Swift, Kotlin, React Native, Flutter…)
- Usate strumenti low-code / no-code per alcune applicazioni interne? (Retool, Power Apps, Bubble, Appsmith…)

---

## 4. Database & Storage

### Database relazionali
- Usate database relazionali? Quali?
  - [ ] PostgreSQL
  - [ ] MySQL / MariaDB
  - [ ] Microsoft SQL Server
  - [ ] Oracle
  - [ ] SQLite (per ambienti embedded/test)

### Database NoSQL
- Usate database NoSQL? Quali?
  - [ ] MongoDB (documentale)
  - [ ] Redis (key-value / cache)
  - [ ] Elasticsearch / OpenSearch (full-text search)
  - [ ] Cassandra / ScyllaDB (columnar, wide-column)
  - [ ] InfluxDB / TimescaleDB (time-series)
  - [ ] Neo4j (a grafo)
  - [ ] Pinecone / pgvector / Weaviate (vettoriale, AI/RAG)

### Domande di approfondimento dati
- Come gestite le migration del database? (Flyway, Liquibase, Alembic, custom…)
- Avete un data warehouse o un layer di analytics separato? (BigQuery, Redshift, Snowflake, dbt…)
- Come gestite backup, point-in-time recovery e disaster recovery?
- Avete un sistema di caching strutturato? (Redis, Memcached, cache a livello applicativo)

---

## 5. Messaggistica & Integrazione

- Come gestite la comunicazione asincrona tra servizi?
  - [ ] Apache Kafka
  - [ ] RabbitMQ
  - [ ] AWS SQS / SNS
  - [ ] Azure Service Bus
  - [ ] Google Pub/Sub
  - [ ] NATS
  - [ ] Nessun message broker (tutto sincrono)
- Avete un API gateway centralizzato? (Kong, AWS API Gateway, NGINX, Traefik…)
- Usate un service mesh? (Istio, Linkerd, Consul Connect)
- Come gestite le integrazioni con sistemi esterni o terze parti? (ESB, iPaaS come MuleSoft, Zapier, Make…)

---

## 6. Processi & DevOps

- Che metodologia di sviluppo usate? (Scrum, Kanban, SAFe, Shape Up…)
- Com'è strutturata la pipeline CI/CD? (GitHub Actions, GitLab CI, Jenkins, CircleCI, ArgoCD…)
- Quanti deploy fate in produzione a settimana o al mese?
- Usate trunk-based development o branch di lunga vita (gitflow, feature branch)?
- Come gestite code review e quality gate? (linting, SAST, test coverage, PR policy…)
- Avete un processo strutturato di on-call e incident management? (PagerDuty, OpsGenie…)
- Come gestite il versionamento delle API?

---

## 7. Osservabilità & Sicurezza

- Come fate monitoring, logging e distributed tracing? (Datadog, Grafana + Prometheus, OpenTelemetry, ELK…)
- Avete SLO / SLA definiti per i servizi critici?
- Come gestite secrets e credenziali? (HashiCorp Vault, AWS Secrets Manager, Azure Key Vault…)
- Avete vulnerability scanning e dependency management automatizzati? (Snyk, Dependabot, Trivy…)
- Come è gestita la compliance? (SOC 2, ISO 27001, GDPR, PCI DSS…)

---

## 8. Team & Organizzazione

- Quanti sviluppatori ha il team? Quanti team / squad esistono?
- Come sono strutturate le squad? (full-stack, frontend/backend separati, platform team…)
- Chi possiede le decisioni tecniche? (CTO, principal engineer, architecture committee…)
- Avete un team Platform / SRE o il delivery team gestisce anche l'infrastruttura?
- Come onboardate nuovi sviluppatori? Esiste un internal developer portal?

---

## 9. Pain Point & Obiettivi

- Qual è il principale collo di bottiglia nel vostro ciclo di sviluppo oggi?
- Dove sentite il debito tecnico più pesante? (stack obsoleto, test mancanti, monolite da spaccare…)
- Cosa bloccherebbe concretamente l'adozione di una nuova pratica nel team?
- Avete già tentato miglioramenti simili? Com'è andata e perché si è fermato?
- Quali metriche usate per misurare la salute del delivery? (DORA, cycle time, lead time, MTTR…)
- Che tipo di supporto o governance vi aspettate dalla community?

---

## Riepilogo — quando approfondire ogni area

| Area | Focus se… |
|---|---|
| Infrastruttura & Cloud | Sempre — è il punto di partenza |
| Backend | Sempre — capire il core tecnologico |
| Frontend | Il prodotto ha una UI significativa |
| Database & Storage | Ci sono problemi di performance, scalabilità o dati complessi |
| Messaggistica | Architettura distribuita o microservizi |
| Processi & DevOps | Il problema dichiarato è velocità o qualità del rilascio |
| Osservabilità & Sicurezza | Problemi di stabilità o requisiti di compliance |
| Team & Organizzazione | Fondamentale per capire chi prende le decisioni |
| Pain Point & Obiettivi | Chiudi sempre con questa sezione |

---

*Documento per onboarding community — personalizzare con domande specifiche al dominio verticale dell'azienda.*
