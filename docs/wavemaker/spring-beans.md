---
title: Spring Bean
parent: WaveMaker
nav_order: 7
---

# Spring Bean — Configurazione e Convenzioni

WaveMaker usa Spring come container IoC. La configurazione Spring è distribuita su più file XML con responsabilità distinte. Gli autori devono conoscere questa gerarchia per evitare di modificare file gestiti da WaveMaker Studio e per registrare correttamente i propri bean.

---

## 1. Gerarchia dei File di Configurazione Spring

```
src/main/webapp/WEB-INF/
├── project-springapp.xml          ← orchestratore principale (gestito da WM)
│   ├── import: springapp.xml
│   ├── import: project-rest-runtime-config.xml
│   ├── import: project-prefabs.xml
│   ├── import: project-services.xml        ← importa i service_*.spring.xml
│   ├── import: project-rest-service.xml
│   └── import: project-user-spring.xml     ← UNICO file modificabile dall'autore
│
services/<NomeServizio>/src/
└── service_<NomeServizio>.spring.xml       ← generato da WM Studio (NON modificare)
```

| File | Chi lo gestisce | Può essere modificato dall'autore |
| ---- | --------------- | --------------------------------- |
| `project-springapp.xml` | WaveMaker Studio | ❌ No |
| `service_<Nome>.spring.xml` | WaveMaker Studio | ❌ No |
| `project-user-spring.xml` | Autore | ✅ Sì — unico punto di estensione |

---

## 2. Regola Fondamentale

> Tutti i bean Spring custom prodotti dall'autore devono essere dichiarati in `project-user-spring.xml`.

Non modificare i file `service_*.spring.xml` generati da WaveMaker Studio: vengono sovrascritti ad ogni rigenerazione del servizio.

---

## 3. Registrazione di un Bean Custom

### Dichiarazione Esplicita (Preferita)

Per ogni classe custom in `src/main/java/` che deve essere gestita da Spring, aggiungere un `<bean>` in `project-user-spring.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- Servizio schedulato -->
    <bean id="cronService"
          class="com.oneclickapp.giglioli.CronService" />

    <!-- Exporter singleton con inizializzazione lazy -->
    <bean id="masterExcelExporter"
          class="com.oneclickapp.giglioli.MasterExcelExporter"
          scope="singleton"
          lazy-init="true" />

    <!-- Bean con dipendenza iniettata -->
    <bean id="reportService"
          class="com.oneclickapp.giglioli.GigliolicReportService">
        <property name="exporter" ref="masterExcelExporter" />
    </bean>

</beans>
```

### Component Scan (Solo se Necessario)

Se si ha un package custom con molte classi annotate (`@Component`, `@Service`, `@Repository`) in `src/main/java/`, è possibile usare il component scan **limitandolo al package specifico**:

```xml
<context:component-scan
    base-package="com.oneclickapp.giglioli.reporting" />
```

**Attenzione:** il `base-package` deve puntare al sotto-package specifico, **mai** al package radice dell'applicazione (`com.oneclickapp.giglioli`). Un scan troppo ampio entra in conflitto con i scan già dichiarati nei `service_*.spring.xml` dei servizi.

---

## 4. Package Prefix e Classpath — Regole

### Servizi DB (generati da WaveMaker)

Ogni servizio DB ha nel proprio `service_<Nome>.spring.xml` un `component-scan` che copre i suoi tre package:

```xml
<context:component-scan
    base-package="com.oneclickapp.giglioli.eventi.controller,
                  com.oneclickapp.giglioli.eventi.service,
                  com.oneclickapp.giglioli.eventi.dao"/>
```

E un `packagesToScan` nella SessionFactory per la scansione delle entity JPA:

```xml
<property name="packagesToScan"
          value="com.oneclickapp.giglioli.eventi"/>
```

Questi package prefix sono generati automaticamente e devono corrispondere esattamente alla struttura dei package Java del servizio (vedi [java-packages.md](../java-packages.md)).

**Se il package Java non corrisponde al prefix dichiarato nel service XML, Spring non trova i bean e l'applicazione non si avvia.**

### Classi Custom in `src/main/java/`

Le classi in `src/main/java/com/<azienda>/<app>/` **non** sono coperte da alcun component-scan automatico. Devono essere registrate esplicitamente in `project-user-spring.xml` come mostrato nella sezione 3.

---

## 5. Naming dei Bean

Il `id` del bean deve seguire la notazione **camelCase** e corrispondere al nome della classe con la prima lettera minuscola:

| Classe | Bean ID |
| ------ | ------- |
| `CronService` | `cronService` |
| `MasterExcelExporter` | `masterExcelExporter` |
| `GigliolicReportService` | `gigliolicReportService` |
| `FileService` | `fileService` |

Non usare nomi arbitrari non collegati alla classe (`bean1`, `myService`, `helper`).

---

## 6. Scope dei Bean

| Scope | Quando usarlo |
| ----- | ------------- |
| `singleton` (default) | Quasi sempre — un'unica istanza condivisa nell'applicazione |
| `prototype` | Quando ogni chiamata deve ricevere una nuova istanza separata |
| `request` | Bean legato al ciclo di vita di una singola richiesta HTTP |
| `session` | Bean legato alla sessione utente |

```xml
<!-- singleton esplicito con lazy init -->
<bean id="reportService"
      class="com.oneclickapp.giglioli.GigliolicReportService"
      scope="singleton"
      lazy-init="true" />
```

Usare `lazy-init="true"` per bean pesanti da inizializzare (connessioni, cache, exporter) che non devono rallentare l'avvio dell'applicazione.

---

## 7. Verifica del Caricamento dei Bean

Se un bean non viene trovato da Spring a runtime (`NoSuchBeanDefinitionException`), verificare nell'ordine:

1. Il bean è dichiarato in `project-user-spring.xml`?
2. Il `class` nel `<bean>` corrisponde esattamente al fully qualified name della classe Java?
3. La classe si trova in `src/main/java/` ed è compilata (presente in `target/classes/`)?
4. Il package della classe è registrato in `pom.xml` nella sezione `<sources>` del `build-helper-maven-plugin` (vedi [pom.md](../pom.md))?
5. In caso di component-scan, il `base-package` copre il package della classe?

---

## 8. Checklist Spring Bean

- [ ] Tutti i bean custom sono in `project-user-spring.xml`
- [ ] Nessuna modifica ai file `service_*.spring.xml` (gestiti da WM Studio)
- [ ] Il `id` del bean è in camelCase e corrisponde al nome della classe
- [ ] Il `class` del bean è il fully qualified name corretto
- [ ] Il `base-package` del component-scan è limitato al sotto-package specifico, non alla radice
- [ ] Le classi in `src/main/java/` sono registrate in `pom.xml` → `build-helper-maven-plugin` → `<sources>`
- [ ] Il package prefix dei servizi DB corrisponde alla struttura dei package Java generati
