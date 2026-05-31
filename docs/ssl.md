# SSL/TLS Conventions

Ogni applicazione enterprise deve essere sempre raggiungibile esclusivamente via HTTPS. Il traffico HTTP deve essere rediretto automaticamente a HTTPS.

---

## 1. Requisito SSL/TLS Obbligatorio

* Tutte le applicazioni devono esporre solo endpoint HTTPS.
* Il redirect da HTTP a HTTPS deve essere abilitato a livello di server o di applicazione.
* Non è accettabile esporre dati o autenticazioni su connessioni non cifrate.

---

## 2. Configurazione Redirect SSL in WaveMaker

In WaveMaker il redirect SSL si abilita nelle proprietà Spring Boot dell'applicazione generata.

**`application.properties`**

```properties
server.forward-headers-strategy=NATIVE
security.require-ssl=true
```

Se l'app è dietro un reverse proxy (Nginx, Apache, AWS ALB):

```properties
server.forward-headers-strategy=FRAMEWORK
```

Il proxy deve aggiungere l'header `X-Forwarded-Proto: https` e redirigere la porta 80 alla 443.

**Esempio Nginx:**

```nginx
server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;
}
```

---

## 3. Tipi di Certificato

### Certificato Autofirmato (Self-Signed)

Accettabile **solo in ambienti di sviluppo o test locali**. Mai in produzione.

* Generazione rapida con OpenSSL:

```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
```

* I browser mostrano un avviso di sicurezza: normale in sviluppo, inaccettabile in produzione.

### Certificato Emesso da CA (Production)

In produzione si usa un certificato firmato da una Certificate Authority riconosciuta.

| Tipo | Esempi CA | Uso tipico |
| ---- | --------- | ---------- |
| DV (Domain Validation) | Let's Encrypt, ZeroSSL | App interne, staging |
| OV (Organization Validation) | DigiCert, Sectigo | Applicazioni aziendali |
| EV (Extended Validation) | DigiCert EV, GlobalSign EV | Portali pubblici, e-commerce |
| Wildcard | Qualsiasi CA | Più sottodomini (`*.example.com`) |

**Raccomandazione:** usare almeno OV per ambienti enterprise esposti all'esterno.

---

## 4. Standard di Configurazione SSL/TLS

Per ottenere la massima sicurezza e compatibilità, rispettare i seguenti standard.

### Versioni di Protocollo

| Protocollo | Stato |
| ---------- | ----- |
| SSLv2 | ❌ Disabilitato |
| SSLv3 | ❌ Disabilitato |
| TLS 1.0 | ❌ Disabilitato |
| TLS 1.1 | ❌ Disabilitato |
| TLS 1.2 | ✅ Minimo accettabile |
| TLS 1.3 | ✅ Preferito |

### Cipher Suite Raccomandate

Usare solo cipher suite con **Perfect Forward Secrecy (PFS)** e **AEAD**:

```
TLS_AES_256_GCM_SHA384          (TLS 1.3)
TLS_CHACHA20_POLY1305_SHA256    (TLS 1.3)
TLS_AES_128_GCM_SHA256          (TLS 1.3)
ECDHE-RSA-AES256-GCM-SHA384     (TLS 1.2)
ECDHE-RSA-AES128-GCM-SHA256     (TLS 1.2)
ECDHE-RSA-CHACHA20-POLY1305     (TLS 1.2)
```

### Chiave del Certificato

* RSA: minimo **2048 bit**, raccomandato **4096 bit**
* ECC (Elliptic Curve): minimo **256 bit** (P-256 o P-384) — preferito per performance

### Header di Sicurezza Obbligatori

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

* `max-age` di almeno 1 anno (31536000 secondi)
* `includeSubDomains` obbligatorio se si usano sottodomini
* `preload` consigliato per registrazione nella HSTS preload list

### Funzionalità Aggiuntive Raccomandate

| Feature | Descrizione |
| ------- | ----------- |
| OCSP Stapling | Riduce latenza nella verifica del certificato |
| Certificate Transparency (CT) | Il certificato deve essere nei log CT pubblici |
| DNS CAA Record | Limita le CA autorizzate a emettere certificati per il dominio |

---

## 5. Test SSL Labs — Requisito Obbligatorio

Prima del rilascio in produzione, ogni autore di applicazione deve verificare la configurazione SSL con **Qualys SSL Labs**.

**URL:** [https://www.ssllabs.com/ssltest/](https://www.ssllabs.com/ssltest/)

### Grading

| Voto | Significato | Accettabilità |
| ---- | ----------- | ------------- |
| A+ | Configurazione eccellente con HSTS preload | ✅ Obiettivo ideale |
| A | Configurazione sicura e corretta | ✅ Minimo accettabile in produzione |
| A- | Configurazione buona con piccole lacune | ⚠️ Accettabile solo con giustificazione |
| B o inferiore | Vulnerabilità o protocolli deboli | ❌ Non accettabile in produzione |

### Procedura

1. Eseguire il test su `https://www.ssllabs.com/ssltest/` prima di ogni deploy in produzione.
2. Allegare uno screenshot del risultato (o il link al report) alla documentazione del rilascio.
3. Il voto minimo accettabile è **A**.
4. In caso di voto inferiore ad A, correggere la configurazione prima del go-live.

### Punti comuni che abbassano il voto

* TLS 1.0 o 1.1 abilitati → disabilitare
* Cipher suite deboli (RC4, DES, 3DES) → rimuovere
* Certificato scaduto o autofirmato in produzione → sostituire
* HSTS assente o `max-age` troppo basso → correggere
* Vulnerabilità note (POODLE, BEAST, Heartbleed) → applicare patch

---

## Riepilogo Checklist

- [ ] HTTPS abilitato, redirect HTTP → HTTPS attivo
- [ ] TLS 1.2 come minimo, TLS 1.3 abilitato
- [ ] SSLv2, SSLv3, TLS 1.0, TLS 1.1 disabilitati
- [ ] Cipher suite con PFS e AEAD
- [ ] Certificato valido (non self-signed in produzione)
- [ ] Chiave RSA ≥ 2048 bit o ECC ≥ 256 bit
- [ ] Header HSTS configurato
- [ ] OCSP Stapling abilitato
- [ ] Test SSL Labs superato con voto **A** o superiore
- [ ] Screenshot/report SSL Labs allegato al rilascio
