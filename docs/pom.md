# Convenzioni pom.xml — WaveMaker

Il file `pom.xml` è il cuore della build Maven dell'applicazione. Una gestione errata delle versioni delle dipendenze può causare build non riproducibili, conflitti a runtime e vulnerabilità di sicurezza. Tutti gli autori devono rispettare le regole seguenti.

---

## 1. Regola Fondamentale — Versioni Sempre Esplicite

Ogni dipendenza aggiunta manualmente **deve** dichiarare una versione esatta e immutabile.

```xml
<!-- CORRETTO -->
<dependency>
    <groupId>net.coobird</groupId>
    <artifactId>thumbnailator</artifactId>
    <version>0.4.20</version>
</dependency>

<!-- CORRETTO -->
<dependency>
    <groupId>net.sf.jasperreports</groupId>
    <artifactId>jasperreports</artifactId>
    <version>6.21.5</version>
</dependency>
```

---

## 2. Versioni Vietate

Non usare mai le seguenti forme di versione:

| Forma | Esempio | Problema |
| ----- | ------- | -------- |
| `LATEST` | `<version>LATEST</version>` | Maven risolve la versione al momento della build — risultato non riproducibile |
| `RELEASE` | `<version>RELEASE</version>` | Come LATEST, non deterministico |
| SNAPSHOT | `<version>2.1.0-SNAPSHOT</version>` | Versione instabile e mutabile nel tempo |
| Range aperto | `<version>[1.0,)</version>` | Può prendere versioni future con breaking change |
| Range chiuso | `<version>[1.0,2.0]</version>` | Comportamento dipendente dall'environment |
| Assente | *(nessun tag `<version>`)* | Solo ammesso se la dipendenza è gestita dal BOM del parent |

```xml
<!-- VIETATO -->
<dependency>
    <groupId>com.example</groupId>
    <artifactId>my-lib</artifactId>
    <version>LATEST</version>
</dependency>

<!-- VIETATO -->
<dependency>
    <groupId>com.example</groupId>
    <artifactId>my-lib</artifactId>
    <version>2.0.0-SNAPSHOT</version>
</dependency>

<!-- VIETATO -->
<dependency>
    <groupId>com.example</groupId>
    <artifactId>my-lib</artifactId>
    <version>[1.0,)</version>
</dependency>
```

---

## 3. Dipendenze Gestite dal Parent WaveMaker

Il parent WaveMaker include un BOM (Bill of Materials) che gestisce già le versioni di molte librerie (es. driver DB, Jackson, Spring). Per queste dipendenze è **corretto** omettere `<version>` — la versione viene ereditata automaticamente e resta allineata con la versione del runtime WaveMaker.

```xml
<!-- CORRETTO — versione gestita dal parent WM -->
<dependency>
    <groupId>org.mariadb.jdbc</groupId>
    <artifactId>mariadb-java-client</artifactId>
</dependency>
```

Non ridefinire la versione per dipendenze già nel BOM del parent, a meno che non ci sia una ragione tecnica esplicita documentata nel pom.

---

## 4. Parent WaveMaker

Il blocco `<parent>` identifica la versione del runtime WaveMaker. Deve sempre corrispondere alla versione ufficiale approvata per il progetto.

```xml
<parent>
    <groupId>com.wavemaker.app</groupId>
    <artifactId>wavemaker-app-parent</artifactId>
    <version>11.15.2</version>
</parent>
```

* Non modificare il `<version>` del parent senza una decisione esplicita del team.
* L'aggiornamento del parent deve essere testato in un ambiente di staging prima del deploy in produzione.

---

## 5. Esclusioni di Dipendenze Transitive

Quando si aggiunge una libreria che porta versioni di dipendenze in conflitto con il runtime WaveMaker (es. Jackson, Spring), escludere esplicitamente le dipendenze in conflitto.

```xml
<dependency>
    <groupId>net.sf.jasperreports</groupId>
    <artifactId>jasperreports</artifactId>
    <version>6.21.5</version>
    <exclusions>
        <exclusion>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
        </exclusion>
        <exclusion>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

Poi ridichiarare la versione corretta della libreria esclusa come dipendenza diretta:

```xml
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
    <version>2.21.2</version>
</dependency>
```

---

## 6. Registrazione dei Servizi nel Build

Ogni servizio presente in `services/` deve essere registrato nel plugin `build-helper-maven-plugin` affinché il suo codice sorgente sia compilato.

```xml
<build>
    <finalName>NomeApp</finalName>
    <plugins>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>build-helper-maven-plugin</artifactId>
            <executions>
                <execution>
                    <id>add-source</id>
                    <phase>generate-sources</phase>
                    <goals>
                        <goal>add-source</goal>
                    </goals>
                    <configuration>
                        <sources>
                            <source>services/NomeServizio1/src</source>
                            <source>services/NomeServizio2/src</source>
                        </sources>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

* `<finalName>` deve corrispondere esattamente al nome dell'applicazione (es. `GIGLIOLI`).
* Se si aggiunge un nuovo servizio in `services/`, registrarlo subito in `<sources>`.
* Se si rimuove un servizio, rimuoverlo dalla lista — build failure altrimenti.

---

## 7. Aggiornamento di una Dipendenza

Prima di aggiornare una versione nel pom.xml:

1. Verificare il changelog della libreria per breaking change.
2. Aggiornare la versione nel pom con il numero esatto (`X.Y.Z`).
3. Eseguire `mvn dependency:tree` per rilevare conflitti con le librerie WaveMaker.
4. Testare l'applicazione in ambiente di sviluppo.
5. Documentare la motivazione dell'aggiornamento in commit message o PR.

---

## 8. SBOM

Ogni autore deve produrre una SBOM (Software Bill of Materials) per ogni release. La SBOM è generata automaticamente tramite un plugin Maven e deve essere allegata alla documentazione di rilascio.

Vedi la guida completa: [sbom.md](sbom.md)

---

## 9. Checklist pom.xml

- [ ] Tutte le dipendenze aggiunte manualmente hanno `<version>` esplicita
- [ ] Nessuna versione `LATEST`, `RELEASE` o `-SNAPSHOT`
- [ ] Nessun range di versione (`[`, `(`)
- [ ] Il `<parent>` WaveMaker è alla versione approvata dal team
- [ ] Le dipendenze in conflitto con WM sono escluse e ridichiarate
- [ ] Tutti i servizi in `services/` sono registrati in `<sources>`
- [ ] `<finalName>` corrisponde al nome dell'applicazione
- [ ] Plugin `cyclonedx-maven-plugin` presente con versione esplicita
- [ ] SBOM generata e consegnata (vedi [sbom.md](sbom.md))
