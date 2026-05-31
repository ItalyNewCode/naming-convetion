# WaveMaker — Release Notes e Matrici di Compatibilità

Gli autori di applicazioni WaveMaker **devono attenersi alle matrici di versionamento e compatibilità** pubblicate ufficialmente dalla documentazione WaveMaker. Non è ammesso usare versioni di runtime, strumenti o dipendenze non allineate alla versione WaveMaker adottata dal progetto.

Riferimento ufficiale: [https://docs.wavemaker.com/learn/wavemaker-release-notes/](https://docs.wavemaker.com/learn/wavemaker-release-notes/)

---

## 1. Versioning di WaveMaker

WaveMaker usa lo schema **MAJOR.MINOR.PATCH** (es. `11.15.3`).

| Componente | Significato |
| ---------- | ----------- |
| MAJOR | Cambiamento architetturale significativo (es. da 10.x a 11.x) |
| MINOR | Nuove funzionalità, aggiornamento delle dipendenze di sistema |
| PATCH | Bugfix e aggiornamenti di sicurezza |

**Regola:** un progetto deve sempre dichiarare esplicitamente la versione WaveMaker nel `pom.xml` (tag `<parent>`) e aggiornarla solo dopo aver verificato la matrice di compatibilità della versione target.

---

## 2. Obbligo di Allineamento alla Matrice di Compatibilità

Ogni versione WaveMaker definisce le versioni esatte dei componenti con cui è stata testata e certificata. Gli autori **non possono scegliere liberamente** le versioni di Java, Node.js, Maven o delle librerie backend: devono usare quelle indicate dalla matrice.

Conseguenze del mancato allineamento:
* Build non riproducibile
* Incompatibilità runtime tra librerie WaveMaker e dipendenze del progetto
* Comportamento non previsto di componenti generati automaticamente (Entity, Controller, Service)
* Supporto negato dalla community in caso di anomalie

---

## 3. Matrice di Compatibilità — WaveMaker 11.15.3 *(corrente)*

> Aggiornare questa tabella ad ogni aggiornamento della versione WaveMaker del progetto, allineandola ai dati presenti nella [pagina ufficiale delle release notes](https://docs.wavemaker.com/learn/wavemaker-release-notes/).

### Ambiente di Build

| Strumento | Versione certificata |
| --------- | -------------------- |
| Java (JDK) | **21** (21.0.6) |
| Node.js | **22.18.0** |
| Maven | **3.9.12** |
| npm | 10.9.3 |
| Apache Ant | 1.10.11 |

### Frontend

| Libreria | Versione |
| -------- | -------- |
| Angular | 18.2.13 |
| Bootstrap | 3.3.7 |
| jQuery | 3.7.1 |
| ngx-bootstrap | 9.0.0 |
| D3.js | 7.8.5 |

### Backend (librerie generate/gestite da WaveMaker)

| Libreria | Versione |
| -------- | -------- |
| Spring Framework | 6.2.18 |
| Spring Boot | 3.5.13 |
| Spring Security | 6.5.10 |
| Hibernate (Jakarta) | 5.6.15 Final |
| Jackson | 2.21.3 |
| Log4j2 | 2.25.4 |

> Le versioni delle librerie backend sono gestite dal BOM del parent WaveMaker. Non ridefinirle nel `pom.xml` del progetto (vedi [pom.md](../pom.md)).

### Server di Deployment Supportati

| Server | Versione minima |
| ------ | --------------- |
| Tomcat (default WM) | 10.1.39 |
| WebSphere Liberty | 23.0.0.9+ |
| JBoss WildFly | 27+ |

### React Native (Mobile)

| Strumento | Versione |
| --------- | -------- |
| Java | 17 |
| Node.js | 22.11.0 |
| Maven | 3.9.9 |
| Expo | 54.0.12 |
| React Native | 0.81.4 |
| Android Gradle Plugin | 8.14.3 |

---

## 4. Come Verificare la Versione del Progetto

La versione WaveMaker in uso è dichiarata nel `pom.xml` del progetto:

```xml
<parent>
    <groupId>com.wavemaker.app</groupId>
    <artifactId>wavemaker-app-parent</artifactId>
    <version>11.15.3</version>   ← versione WaveMaker
</parent>
```

La versione nel `pom.xml` **deve corrispondere** alla versione per cui si legge la matrice di compatibilità. Se il progetto è su `11.14.x`, la matrice di riferimento è quella della `11.14.x`, non della più recente.

---

## 5. Aggiornamento della Versione WaveMaker

L'aggiornamento di WaveMaker (es. da `11.14.x` a `11.15.x`) non è un cambio di dipendenza ordinario: può comportare aggiornamenti di Angular, Spring Boot, Java e del runtime.

**Procedura obbligatoria:**

1. Leggere le release notes della versione target su [docs.wavemaker.com](https://docs.wavemaker.com/learn/wavemaker-release-notes/).
2. Confrontare le matrici di compatibilità tra la versione attuale e quella target.
3. Aggiornare il tag `<version>` nel `<parent>` del `pom.xml`.
4. Aggiornare le versioni di Node.js, Maven e Java nella pipeline CI/CD (vedi [cicd.md](../cicd.md)) per allinearle alla nuova matrice.
5. Rieseguire `mvn clean install` in locale.
6. Testare l'applicazione in ambiente di sviluppo prima di promuovere la build.
7. Rigenerare la SBOM (vedi [sbom.md](../sbom.md)) dopo l'aggiornamento.

---

## 6. Fine del Supporto

| Versione | Stato | Fine supporto |
| -------- | ----- | ------------- |
| WaveMaker 11.x | ✅ Attiva | Non annunciato |
| WaveMaker 10.x | ❌ End of Life | 25 luglio 2022 |
| WaveMaker 9.x e precedenti | ❌ End of Life | — |

Non sviluppare nuove applicazioni su versioni End of Life. Le applicazioni esistenti su versioni EOL devono essere migrate alla versione attiva nel più breve tempo possibile.
