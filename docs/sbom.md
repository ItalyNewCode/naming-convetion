# SBOM — Software Bill of Materials

Una SBOM è l'elenco completo e formale di tutti i componenti software inclusi in un'applicazione: dipendenze dirette, dipendenze transitive, versioni, licenze e hash. È l'equivalente di una "lista ingredienti" del software.

Ogni autore deve produrre e consegnare una SBOM per ogni release in produzione.

---

## 1. Perché è Obbligatoria

* **Sicurezza:** permette di sapere immediatamente se un componente vulnerabile (es. Log4Shell) è presente nell'applicazione.
* **Compliance:** richiesta da normative internazionali (EU Cyber Resilience Act, US Executive Order 14028, DORA).
* **Supply chain:** consente di verificare che nessuna dipendenza non autorizzata o malevola sia entrata nella build.
* **Audit:** fornisce evidenza tracciabile di cosa gira in produzione in un dato momento.

---

## 2. Standard di Riferimento

Esistono tre standard principali. Per progetti Maven/WaveMaker lo standard raccomandato è **CycloneDX**.

### CycloneDX (OWASP) — Raccomandato

* Mantenuto da OWASP.
* Progettato specificamente per la sicurezza del software e la supply chain.
* Ha un plugin Maven nativo e maturo.
* Supporta analisi di vulnerabilità (CVE), licenze, dipendenze transitive.
* Formati: XML, JSON.
* Versione schema corrente: **1.6**
* Sito: [cyclonedx.org](https://cyclonedx.org)

### SPDX (Linux Foundation) — Standard ISO

* Standard ISO/IEC 5962:2021.
* Adottato prevalentemente in ambito open source e governo.
* Più orientato alla gestione delle licenze che alla sicurezza.
* Formati: `.spdx`, `.json`, `.xml`, `.yaml`, `.rdf`.
* Plugin Maven meno maturo rispetto a CycloneDX.
* Sito: [spdx.dev](https://spdx.dev)

### NTIA Minimum Elements

* Non è un formato, ma un insieme di **campi minimi obbligatori** definiti dalla NTIA (US).
* Qualsiasi SBOM (CycloneDX o SPDX) che include questi campi è conforme:
  * Nome del componente
  * Versione del componente
  * Identificatore univoco (es. Package URL — purl)
  * Relazione tra componenti (dipende da)
  * Autore della SBOM
  * Timestamp di generazione

CycloneDX 1.4+ soddisfa automaticamente tutti i campi NTIA.

---

## 3. Generazione con Maven — Plugin CycloneDX

Aggiungere il plugin al `pom.xml` del progetto:

```xml
<build>
    <plugins>
        <!-- altri plugin -->

        <plugin>
            <groupId>org.cyclonedx</groupId>
            <artifactId>cyclonedx-maven-plugin</artifactId>
            <version>2.9.1</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>makeAggregateBom</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <projectType>application</projectType>
                <schemaVersion>1.6</schemaVersion>
                <includeBomSerialNumber>true</includeBomSerialNumber>
                <includeCompileScope>true</includeCompileScope>
                <includeProvidedScope>true</includeProvidedScope>
                <includeRuntimeScope>true</includeRuntimeScope>
                <includeSystemScope>true</includeSystemScope>
                <includeTestScope>false</includeTestScope>
                <outputFormat>all</outputFormat>
                <outputName>bom</outputName>
            </configuration>
        </plugin>
    </plugins>
</build>
```

**Dopo `mvn package` vengono generati:**

```
target/
├── bom.xml     ← SBOM in formato CycloneDX XML
└── bom.json    ← SBOM in formato CycloneDX JSON
```

La versione del plugin deve essere sempre esplicita (vedi [pom.md](pom.md)) — no `LATEST`, no `SNAPSHOT`.

---

## 4. Contenuto della SBOM Generata

Il file `bom.xml` / `bom.json` include per ogni componente:

| Campo | Esempio |
| ----- | ------- |
| Nome | `thumbnailator` |
| Gruppo | `net.coobird` |
| Versione | `0.4.20` |
| Package URL (purl) | `pkg:maven/net.coobird/thumbnailator@0.4.20` |
| Hash SHA-256 | `e3b0c44298fc1c149...` |
| Licenza | `MIT` |
| Tipo | `library` |

Include anche le dipendenze **transitive** — quelle che non compaiono nel pom.xml ma vengono portate dalle dipendenze dirette.

---

## 5. Quando Generare la SBOM

| Evento | Obbligo |
| ------ | ------- |
| Ogni build di release | ✅ Obbligatorio |
| Ogni modifica al `pom.xml` (nuova dipendenza, aggiornamento versione) | ✅ Obbligatorio |
| Build di sviluppo quotidiana | ⚪ Opzionale |
| Hotfix in produzione | ✅ Obbligatorio |

La SBOM deve essere rigenerata ogni volta che cambiano le dipendenze. Una SBOM datata non è affidabile.

---

## 6. Dove Consegnare la SBOM

* **Repository del progetto:** committare `bom.xml` e `bom.json` nella cartella `sbom/` alla radice del progetto per ogni release taggata.
* **Artifact repository:** allegare i file `bom.xml` / `bom.json` all'artifact WAR nel repository Maven (Nexus, Artifactory).
* **Documentazione di rilascio:** riferimento ai file SBOM nella PR o nel ticket di release.

```
<root>/
└── sbom/
    ├── bom-1.0.xml
    ├── bom-1.0.json
    ├── bom-1.1.xml
    └── bom-1.1.json
```

## 8. Checklist SBOM per ogni Release

- [ ] Plugin `cyclonedx-maven-plugin` presente nel `pom.xml` con versione esplicita
- [ ] `mvn package` eseguito con successo — `target/bom.xml` e `target/bom.json` generati
- [ ] File SBOM committati in `sbom/` con nome versionato (`bom-<versione>.xml`)
- [ ] Vulnerability scan eseguito su `bom.json` (Grype o OWASP Dependency-Check)
- [ ] Nessuna vulnerabilità **Critical** o **High** non gestita presente
- [ ] SBOM allegata alla documentazione di release
