---
title: Continuous Delivery
nav_order: 7
---

# Continuous Delivery — GitHub Actions + AWS Elastic Beanstalk

Ogni applicazione WaveMaker enterprise deve adottare una pipeline di Continuous Delivery automatizzata. Il deploy su AWS Elastic Beanstalk avviene tramite una **GitHub Action riutilizzabile** mantenuta dalla community.

---

## 1. Pipeline Community — wm-elasticbeanstalk-publisher

La pipeline ufficiale è pubblicata e mantenuta dalla community al seguente indirizzo:

**[https://github.com/OneClickApp-by-LOGCONSULTING/wm-elasticbeanstalk-publisher](https://github.com/OneClickApp-by-LOGCONSULTING/wm-elasticbeanstalk-publisher)**

Gli autori **non devono replicare o mantenere** la logica di build e deploy nel proprio repository. Devono solo richiamare il workflow riutilizzabile da quel repository, che gestisce in automatico:

1. Checkout del codice sorgente
2. Validazione dei file di configurazione (evita deploy errati in produzione)
3. Setup dell'ambiente: Node.js, Maven, Java (con cache)
4. Report delle versioni degli strumenti di build
5. Compilazione Maven (`mvn clean install`)
6. Generazione del WAR
7. Autenticazione AWS
8. Upload del WAR su S3 (compresso)
9. Deploy su Elastic Beanstalk (nuova versione applicativa + deploy)

---

## 2. Struttura del Workflow nel Progetto

Nel repository dell'applicazione creare il file:

```
.github/
└── workflows/
    └── deploy-prod.yml
```

### Esempio Completo (Produzione)

Sostituire ogni occorrenza di `<nome-app>` con il nome effettivo dell'applicazione e adattare regione e versioni degli strumenti.

```yaml
name: WM-to-Beanstalk

on:
  push:
    branches:
      - main   # adattare se il branch si chiama "master"

jobs:
  call-build-and-deploy:
    permissions:
      id-token: write   # obbligatorio per autenticazione OIDC con AWS
      contents: read
    uses: OneClickApp-by-LOGCONSULTING/wm-elasticbeanstalk-publisher/.github/workflows/ebt-deployer.yml@main
    secrets:
      AWS_ROLE_TO_ASSUME_FOR_DEPLOY: ${{ secrets.AWS_ROLE_TO_ASSUME_FOR_DEPLOY }}
    with:
      node-version: '22.18.0'
      maven-version: '3.9.11'
      java-version: '21'
      aws-region: 'eu-central-1'
      environment-name: '<nome-app>-prod'
      wm-app-name: '<nome-app>'
      beanstalk-application-name: '<nome-app>'
      wm-profile: 'prod'
      aws-s3-bucket-name: '<nome-app>-ebt-deployment'
      aws-s3-bucket-region: 'eu-central-1'
      force-lowercase-wm-app-name: false
      log-group-prefix: '<nome-app>'
      force-root-wm-app-name: true
```

> Il file del workflow riutilizzabile da richiamare è `ebt-deployer.yml` — usare sempre questo nome.

---

## 3. Parametri del Workflow

### Input Obbligatori

| Parametro | Descrizione | Esempio |
| --------- | ----------- | ------- |
| `environment-name` | Nome dell'ambiente Elastic Beanstalk | `giglioli-prod` |
| `wm-profile` | Profilo WaveMaker da usare nella build | `prod` |
| `aws-s3-bucket-name` | Bucket S3 dove viene caricato il WAR | `my-app-deployments` |
| `aws-s3-bucket-region` | Regione AWS del bucket S3 | `eu-south-1` |

### Input Opzionali (con valori di default)

| Parametro | Default | Descrizione |
| --------- | ------- | ----------- |
| `node-version` | `18.16.1` | Versione Node.js per la build frontend |
| `maven-version` | `3.9.9` | Versione Maven |
| `java-version` | `21` | Versione Java (deve corrispondere a `<java.required.version>` nel `pom.xml`) |
| `aws-region` | `eu-south-1` | Regione AWS del deploy |
| `wm-app-name` | `OneClickApp` | Nome dell'applicazione WaveMaker |
| `beanstalk-application-name` | `OneClickApp` | Nome dell'applicazione in Elastic Beanstalk |
| `force-lowercase-wm-app-name` | `false` | Se `true`, forza il nome app in minuscolo nel path del WAR |
| `log-group-prefix` | *(vuoto)* | Prefisso del CloudWatch Log Group associato all'ambiente |
| `force-root-wm-app-name` | `false` | Se `true`, deploya il WAR come root context (`/`) invece di `/<nome-app>` |

> Sovrascrivere sempre le versioni di `node-version`, `maven-version` e `java-version` — non affidarsi ai default della pipeline community, che possono cambiare nel tempo.

### Secret Obbligatori

Il secret deve essere configurato nelle impostazioni GitHub del repository (`Settings → Secrets and variables → Actions`):

| Secret | Descrizione |
| ------ | ----------- |
| `AWS_ROLE_TO_ASSUME_FOR_DEPLOY` | ARN del ruolo IAM da assumere via OIDC per il deploy |

L'autenticazione avviene tramite **OIDC** (OpenID Connect): GitHub ottiene un token temporaneo e lo scambia con credenziali AWS a breve scadenza assumendo il ruolo configurato. Non si usano access key a lunga durata. Questo richiede `permissions: id-token: write` nel job (già incluso nell'esempio della sezione 2).

Le credenziali AWS devono appartenere a un ruolo IAM con i permessi minimi necessari (vedi sezione 5).

---

## 4. Profili di Configurazione

Il parametro `wm-profile` corrisponde a un file nella cartella `profiles/` del progetto:

```
profiles/
├── development.properties   ← usato in locale
├── deployment.properties    ← usato nella pipeline CI
└── prod.properties          ← usato nel deploy in produzione
```

Il profilo selezionato dalla pipeline sovrascrive le variabili di connessione al database, URL degli endpoint e altre proprietà ambiente-specifiche. Non inserire mai credenziali in chiaro nei file di profilo: usare variabili d'ambiente risolte a runtime da Elastic Beanstalk.

---

## 5. Permessi IAM Minimi per il Deploy

Il **ruolo IAM** assunto dalla pipeline deve avere solo i permessi strettamente necessari. Non usare `AdministratorAccess`.

Il ruolo deve avere una trust policy che autorizza GitHub Actions ad assumerlo via OIDC:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<ACCOUNT_ID>:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:<org>/<repo>:ref:refs/heads/main"
        }
      }
    }
  ]
}
```

Sostituire `<ACCOUNT_ID>`, `<org>` e `<repo>` con i valori reali. Il campo `sub` limita l'assunzione del ruolo al solo branch `main` del solo repository specificato.

Policy dei permessi minimi raccomandata:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::nome-bucket-s3-artefatti",
        "arn:aws:s3:::nome-bucket-s3-artefatti/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "elasticbeanstalk:CreateApplicationVersion",
        "elasticbeanstalk:UpdateEnvironment",
        "elasticbeanstalk:DescribeEnvironments",
        "elasticbeanstalk:DescribeApplicationVersions"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "autoscaling:DescribeAutoScalingGroups",
        "cloudformation:DescribeStackResource",
        "cloudformation:DescribeStackResources"
      ],
      "Resource": "*"
    }
  ]
}
```

---

## 6. Strategia di Branch e Ambienti

Ogni branch può puntare a un ambiente Elastic Beanstalk diverso. Configurare un workflow separato per ambiente oppure usare condizioni nel medesimo file.

Struttura raccomandata:

| Branch | Ambiente EB | Profilo WM |
| ------ | ----------- | ---------- |
| `main` | `nomeapp-prod` | `prod` |
| `staging` | `nomeapp-staging` | `staging` |
| `develop` | *(deploy manuale)* | `development` |

Esempio con due ambienti nello stesso file:

```yaml
name: Deploy

on:
  push:
    branches:
      - main
      - staging

jobs:
  deploy-prod:
    if: github.ref == 'refs/heads/main'
    uses: OneClickApp-by-LOGCONSULTING/wm-elasticbeanstalk-publisher/.github/workflows/deploy.yml@main
    with:
      environment-name: "giglioli-prod"
      wm-profile: "prod"
      aws-s3-bucket-name: "giglioli-deployments"
      aws-s3-bucket-region: "eu-south-1"
    secrets:
      aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
      aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

  deploy-staging:
    if: github.ref == 'refs/heads/staging'
    uses: OneClickApp-by-LOGCONSULTING/wm-elasticbeanstalk-publisher/.github/workflows/deploy.yml@main
    with:
      environment-name: "giglioli-staging"
      wm-profile: "staging"
      aws-s3-bucket-name: "giglioli-deployments"
      aws-s3-bucket-region: "eu-south-1"
    secrets:
      aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
      aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

---

## 7. Versione della Pipeline Community

Il workflow viene richiamato con il riferimento `@main`. Questo significa che si usa sempre l'ultima versione stabile mantenuta dalla community.

Se un progetto ha esigenze di stabilità assoluta (es. freeze di release), è possibile puntare a un tag specifico:

```yaml
uses: OneClickApp-by-LOGCONSULTING/wm-elasticbeanstalk-publisher/.github/workflows/deploy.yml@v1.2.0
```

In questo caso monitorare il repository community per aggiornamenti di sicurezza e applicarli tempestivamente.

---

## 8. Deploy On-Premise — GitHub Release

Quando il deploy **non avviene su cloud** (niente AWS, niente Elastic Beanstalk), l'artefatto WAR viene distribuito tramite una **GitHub Release**. Il server di destinazione scarica il WAR dalla release e lo installa.

### 8.1 Workflow

Creare il file `.github/workflows/release.yml`:

```yaml
name: Build and Release WAR

on:
  push:
    branches:
      - main   # adattare se il branch si chiama "master"

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          distribution: 'corretto'
          java-version: '21'

      - name: Cache Maven packages
        uses: actions/cache@v4
        with:
          path: ~/.m2
          key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
          restore-keys: |
            ${{ runner.os }}-maven-

      - name: Build with Maven
        run: mvn clean install -Pprod

      - name: Locate WAR
        id: war
        run: |
          WAR_FILE=$(find target -name "*.war" | head -n 1)
          echo "WAR=$WAR_FILE" >> $GITHUB_OUTPUT

      - name: Generate release tag
        id: tag
        run: |
          DATE=$(date +'%Y%m%d-%H%M%S')
          TAG_NAME="release-${DATE}"
          echo "TAG_NAME=$TAG_NAME" >> $GITHUB_OUTPUT

      - name: Create GitHub Release
        uses: softprops/action-gh-release@v2
        with:
          tag_name: ${{ steps.tag.outputs.TAG_NAME }}
          name: Release ${{ steps.tag.outputs.TAG_NAME }}
          body: |
            Build automatica dal branch ${{ github.ref_name }}
            Commit: ${{ github.sha }}
          files: ${{ steps.war.outputs.WAR }}
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Il `GITHUB_TOKEN` è fornito automaticamente da GitHub — non richiede secret aggiuntivi.

---

### 8.2 Convenzione di Versionamento — Regola Fondamentale

> **Ogni push su `main` che va in produzione deve produrre esattamente una release con un tag univoco e non riutilizzabile.**

Il tag è la versione. Senza un tag leggibile e tracciabile non è possibile fare rollback, confrontare artefatti o rispondere a un audit.

#### Formato del tag — `release-YYYYMMDD-HHMMSS`

Il formato generato automaticamente dal workflow è:

```
release-20260531-143022
```

| Parte | Significato |
| ----- | ----------- |
| `release-` | prefisso fisso, identifica i tag di release |
| `YYYYMMDD` | data della build in UTC |
| `HHMMSS` | ora della build in UTC |

**Vantaggi:** univocità garantita, ordine cronologico leggibile, nessuna gestione manuale.

**Vincoli:**
* Il tag **non si modifica mai** dopo la creazione — una release è immutabile.
* **Non cancellare mai** una release passata, nemmeno se errata: creare una nuova release correttiva.
* Non riutilizzare lo stesso tag su commit diversi.

#### Quando usare Semantic Versioning (`vMAJOR.MINOR.PATCH`)

Se il progetto espone API consumate da altri sistemi, affiancare al tag timestamp un tag semantico manuale prima del deploy:

```
v1.4.2
```

In questo caso il workflow deve essere adattato per leggere la versione da un file (es. `VERSION`) o da una variabile d'ambiente impostata manualmente prima del push.

---

### 8.3 Contenuto della Release

Ogni GitHub Release deve contenere almeno:

| File | Obbligatorio | Descrizione |
| ---- | ------------ | ----------- |
| `NomeApp.war` | ✅ | Artefatto deployabile |
| `bom.xml` | ✅ | SBOM CycloneDX XML (vedi [sbom.md](sbom.md)) |
| `bom.json` | ✅ | SBOM CycloneDX JSON |

Per allegare automaticamente anche la SBOM, aggiungere i file nel parametro `files`:

```yaml
- name: Create GitHub Release
  uses: softprops/action-gh-release@v2
  with:
    tag_name: ${{ steps.tag.outputs.TAG_NAME }}
    name: Release ${{ steps.tag.outputs.TAG_NAME }}
    body: |
      Build automatica dal branch ${{ github.ref_name }}
      Commit: ${{ github.sha }}
    files: |
      ${{ steps.war.outputs.WAR }}
      target/bom.xml
      target/bom.json
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

### 8.4 Rollback

In caso di regressione in produzione, il rollback consiste nel scaricare il WAR della release precedente e ridistribuirlo.

1. Aprire la tab **Releases** del repository GitHub.
2. Individuare la release precedente all'ultima (ordinate cronologicamente per tag).
3. Scaricare il file `.war`.
4. Installarlo sul server di destinazione.

Questo è possibile solo se le release non vengono cancellate e i tag non vengono riusati.

---

## 9. Riepilogo — Quale Strategia Usare

| Scenario | Strategia |
| -------- | --------- |
| Deploy su AWS Elastic Beanstalk | Pipeline community `wm-elasticbeanstalk-publisher` (sezione 1–7) |
| Deploy on-premise o server dedicato | GitHub Release con WAR allegato (sezione 8) |
| Entrambi (staging cloud, prod on-premise) | Workflow separati nello stesso repository |

---

## 10. Checklist Continuous Delivery

### Deploy AWS Elastic Beanstalk
- [ ] File `.github/workflows/deploy.yml` presente nel repository
- [ ] Parametri `environment-name`, `wm-profile`, `aws-s3-bucket-name`, `aws-s3-bucket-region` configurati
- [ ] Secret `AWS_ACCESS_KEY_ID` e `AWS_SECRET_ACCESS_KEY` configurati in GitHub
- [ ] Utente IAM con policy minima (no `AdministratorAccess`)
- [ ] Profili `profiles/prod.properties` e `profiles/staging.properties` presenti e corretti
- [ ] Nessuna credenziale in chiaro nei file di profilo
- [ ] Branch `main` protetto (richiede Pull Request approvata prima del merge)
- [ ] Pipeline testata su staging prima del primo deploy in produzione

### Deploy On-Premise via GitHub Release
- [ ] File `.github/workflows/release.yml` presente nel repository
- [ ] Tag generato automaticamente con formato `release-YYYYMMDD-HHMMSS`
- [ ] WAR allegato alla release
- [ ] SBOM (`bom.xml`, `bom.json`) allegata alla release
- [ ] Release mai cancellate — storico completo mantenuto
- [ ] Tag mai riutilizzati su commit diversi
- [ ] Procedura di rollback documentata e testata
