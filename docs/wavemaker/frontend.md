# Convenzioni Frontend — WaveMaker

Questa guida definisce le regole obbligatorie per la costruzione delle pagine e dei widget nelle applicazioni WaveMaker enterprise.

---

## 1. Naming delle Pagine

### Pagine Principali

Le pagine principali devono essere nominate interamente in **minuscolo**. Per nomi composti da più parole usare l'underscore come separatore.

| Corretto | Errato |
| -------- | ------ |
| `lista_materiali` | `listaMateriali` |
| `dettaglio_ordine` | `DettaglioOrdine` |
| `gestione_utenti` | `gestioneUtenti` |
| `dashboard` | `Dashboard` |
| `cambio_stato` | `cambioStato` |

**Regola:** il nome deve descrivere la funzione della pagina, non il tipo di widget che contiene. Evitare nomi generici come `pagina1`, `test`, `nuovo`.

### Pagine Partial

Le pagine partial si distinguono dalle pagine principali per la **prima lettera maiuscola** (PascalCase). Nessun prefisso aggiuntivo.

| Corretto | Errato |
| -------- | ------ |
| `FiltroRicerca` | `p_filtroRicerca` |
| `DettaglioArticolo` | `dettaglio_articolo` |
| `Header` | `header` |
| `Rightnav` | `rightnav_partial` |
| `ListinoMateriali` | `listino_materiali` |

La prima lettera maiuscola è il segnale visivo che distingue immediatamente una partial da una pagina navigabile, senza bisogno di prefissi.

---

## 2. Naming dei Widget

Il nome di ogni widget deve seguire il pattern:

```
<tipoWidget><NomeFunzionalità>
```

* Il tipo del widget è in minuscolo (corrisponde al tipo WaveMaker: `label`, `text`, `button`, `grid`, `select`, `form`, ecc.).
* Il nome funzionalità inizia con la maiuscola (PascalCase).

**Esempi:**

| Widget | Nome corretto | Nome errato |
| ------ | ------------- | ----------- |
| Label | `labelUltimaModifica` | `lbl_ultima_modifica` |
| Button | `buttonConfermaOrdine` | `btn1` |
| Text | `textNomeCliente` | `text_nome` |
| Select | `selectStatoArticolo` | `dropdown1` |
| Grid | `gridListaMateriali` | `datagrid1` |
| Form | `formInserimentoOrdine` | `form_ordine` |
| Container | `containerFiltriRicerca` | `div1` |

Non usare abbreviazioni (`lbl`, `btn`, `txt`) né numerazioni progressivi (`label1`, `label2`).

---

## 3. Variabili

Le variabili create all'interno dell'applicazione vanno in **camelCase**.

**Esempi:**

* ✔ `listaOrdini`
* ✔ `selectedArticolo`
* ✔ `isFormValid`
* ❌ `lista_ordini`
* ❌ `ListaOrdini`
* ❌ `var1`

---

## 4. Datagrid — Filtri di Colonna Obbligatori

Ogni widget **Datagrid** deve avere i filtri di colonna abilitati. Non è ammesso un datagrid senza la possibilità di filtrare i dati per colonna.

### Configurazione obbligatoria

Nel pannello proprietà del widget Datagrid:

* **Filter Mode** → `Multi Column` (filtro per singola colonna su ogni intestazione)

Ogni colonna esposta nella griglia deve avere il filtro attivo, a meno che la colonna non contenga dati non filtrabili per natura (es. colonna azioni, colonna con icone).

### Motivazione

Il filtro di colonna è il principale strumento di navigazione nei dataset enterprise. La sua assenza costringe l'utente a scorrere liste potenzialmente lunghe senza possibilità di affinare la ricerca, riducendo drasticamente l'usabilità.

### Esempio di configurazione accettabile

```
Datagrid: gridListaMateriali
├── Colonna: codiceArticolo    → filtro attivo
├── Colonna: descrizione       → filtro attivo
├── Colonna: quantita          → filtro attivo
├── Colonna: stato             → filtro attivo (select/dropdown)
└── Colonna: azioni            → filtro NON richiesto (colonna con bottoni Edit/Delete)
```

---

## 5. Filtri di Ricerca — Pulsante Reset Obbligatorio

Ogni area di ricerca o filtro presente in una pagina deve includere un pulsante **Reset** (o equivalente: *Pulisci*, *Cancella filtri*) che riporti tutti i campi di filtro al valore iniziale e riesegua la query senza filtri.

### Regola

> Dove c'è un filtro di ricerca, deve esserci un pulsante Reset.

Questo vale per:
* Pannelli di filtro sopra una griglia
* Form di ricerca avanzata
* Sidebar di filtro
* Filtri inline nella toolbar

### Comportamento atteso del pulsante Reset

1. Azzerare tutti i campi del filtro (svuotare i testi, deselezionare i select, rimuovere le date).
2. Rieseguire la query/variabile che popola la griglia o la lista, restituendo tutti i record.
3. Riportare la paginazione alla prima pagina.

### Esempio di layout accettabile

```
┌─────────────────────────────────────────────────┐
│  Codice: [__________]  Stato: [____▼]           │
│                                                 │
│  [Cerca]                        [Reset Filtri]  │
└─────────────────────────────────────────────────┘
│  Datagrid con risultati filtrati                │
└─────────────────────────────────────────────────┘
```

Il pulsante Reset deve essere visivamente distinto dal pulsante Cerca (es. stile secondario/outline) e posizionato in prossimità dei filtri, non in fondo alla pagina.

---

## 6. Feedback Utente e Gestione Errori

### appNotification

WaveMaker mette a disposizione la variabile `appNotification` per mostrare messaggi toast all'utente. Se presente nell'app, deve essere usata come canale unico per tutti i feedback: successo, avviso ed errore.

Se `appNotification` **non è presente**, l'autore deve comunque garantire un feedback visivo all'utente tramite un meccanismo alternativo (dialog, banner inline, label di errore). In ogni caso il messaggio deve essere nella lingua dell'utente.

### Intercettazione errori XHR in `app.js`

Tutti gli errori di chiamata HTTP (errori di rete, 4xx, 5xx) devono essere intercettati centralmente in `app.js` tramite il hook `onServiceError`. Questo evita che un errore silenzioso rimanga invisibile all'utente.

```javascript
// app.js
App.onServiceError = function(xhrObj, errorMsg, status) {
    var msg = App.appLocale.erroreGenerico || 'Si è verificato un errore. Riprovare.';

    if (App.Widgets.appNotification) {
        App.Widgets.appNotification.show({
            message: msg,
            type:    'error'
        });
    } else {
        // fallback se appNotification non è presente
        alert(msg);
    }
};
```

**Regole:**

* Il messaggio di errore deve provenire dal file i18n (`App.appLocale.<chiave>`) — mai testo hardcoded in inglese.
* Il codice HTTP (`status`) può essere usato per differenziare il messaggio (es. 401 → sessione scaduta, 403 → accesso negato, 500 → errore server).
* Gli errori di validazione lato server (422) vanno mostrati nel contesto del form, non come toast generici.

---

## 7. Operazioni Distruttive — Conferma Obbligatoria

Qualsiasi operazione irreversibile deve essere preceduta da un **dialog di conferma esplicito** prima di essere eseguita.

Sono operazioni distruttive:
* Eliminazione di un record (`DELETE`)
* Reset di dati o stati
* Sovrascrittura di un documento o file
* Qualsiasi azione che non può essere annullata

### Requisiti del dialog di conferma

Il testo del dialog deve identificare **cosa verrà eliminato**, non limitarsi a chiedere una conferma generica.

```
✔  "Eliminare il materiale 'Acciaio 316L' (cod. MAT-0042)?"
   [Annulla]  [Elimina]

✗  "Sei sicuro di voler procedere?"
   [No]  [Sì]
```

* Il bottone di conferma usa stile `danger` (rosso).
* Il bottone di annullamento è la scelta predefinita (focus iniziale).
* Il dialog deve essere in lingua (vedi sezione i18n).

---

## 8. Formattazione dei Dati

I dati devono essere sempre presentati tramite i **formatter di WaveMaker** o, dove questi non bastano, tramite formatter custom registrati nell'app. Non usare concatenazione di stringhe per formattare date, valute o timestamp.

### Date

| Contesto | Formato | Esempio |
| -------- | ------- | ------- |
| Visualizzazione in griglia o label | `dd/MM/yyyy` | `31/05/2026` |
| Input utente (date picker) | `dd/MM/yyyy` | `31/05/2026` |
| Esportazione/API | `yyyy-MM-dd` (ISO 8601) | `2026-05-31` |

Usare il formatter WaveMaker `toDate` con il pattern appropriato. Non concatenare giorno, mese e anno a mano.

### Valuta e Numeri

| Tipo | Formato | Esempio |
| ---- | ------- | ------- |
| Valuta Euro | `€ #.##0,00` | `€ 1.234,56` |
| Valuta generica | simbolo + spazio + valore | `$ 1,234.56` |
| Numero decimale | separatore decimale `,` (IT) | `3.141,59` |
| Numero intero con migliaia | separatore `.` (IT) | `1.000.000` |
| Percentuale | valore + `%` | `12,5%` |

Usare il formatter WaveMaker `toCurrency` o `toNumber` con il locale corretto. Se l'app è internazionale, il locale viene letto dalla lingua attiva (vedi sezione i18n).

### Timestamp e Timezone

I timestamp devono essere convertiti nel **fuso orario locale dell'utente** prima della visualizzazione. Non mostrare mai un timestamp UTC grezzo all'utente finale.

```
✔  "31/05/2026 14:32:05 (CEST)"
✗  "2026-05-31T12:32:05Z"
```

In ambienti cloud il backend opera in UTC. La conversione timezone avviene nel layer di presentazione, usando il formatter `toDate` con il pattern `dd/MM/yyyy HH:mm:ss` e il locale del browser.

---

## 9. Internazionalizzazione (i18n)

Se l'applicazione deve essere multilingua, **ogni testo visibile all'utente** deve essere gestito tramite i file i18n. Non è ammesso testo hardcoded nelle pagine o nel codice JavaScript.

### File i18n

I file di lingua si trovano nella cartella `i18n/` alla radice del progetto:

```
i18n/
├── en.json   ← inglese (lingua di riferimento)
├── it.json   ← italiano
└── de.json   ← altre lingue se necessario
```

Ogni file contiene coppie chiave-valore. La chiave è identica in tutti i file; il valore è tradotto.

```json
// i18n/it.json
{
  "labelNomeCliente":     "Nome cliente",
  "buttonSalva":          "Salva",
  "erroreGenerico":       "Si è verificato un errore. Riprovare.",
  "confermaCancellazione": "Eliminare il record selezionato?",
  "validazioneCampoObbligatorio": "Campo obbligatorio",
  "validazioneEmailNonValida":    "Inserire un indirizzo email valido (es. nome@dominio.it)"
}
```

```json
// i18n/en.json
{
  "labelNomeCliente":     "Customer name",
  "buttonSalva":          "Save",
  "erroreGenerico":       "An error occurred. Please try again.",
  "confermaCancellazione": "Delete the selected record?",
  "validazioneCampoObbligatorio": "Required field",
  "validazioneEmailNonValida":    "Enter a valid email address (e.g. name@domain.com)"
}
```

### Cosa deve essere internazionalizzato

| Elemento | i18n obbligatorio |
| -------- | ----------------- |
| Etichette (label, intestazioni colonne) | ✅ |
| Testo dei bottoni | ✅ |
| Messaggi di errore (validazione, XHR) | ✅ |
| Messaggi informativi e toast | ✅ |
| Testo dei dialog di conferma | ✅ |
| Placeholder dei campi input | ✅ |
| Messaggi stato vuoto ("Nessun risultato") | ✅ |
| Testo interno agli script `app.js` | ✅ |
| Dati provenienti dal database | ❌ (responsabilità del backend) |

### Formatter localizzati

Se l'app è multilingua e usa formati diversi per nazioni diverse (es. data `MM/dd/yyyy` per gli USA, `dd/MM/yyyy` per l'Italia), i formatter devono essere configurati in base al locale attivo, non hardcodati.

---

## 10. Validazione dei Form

### Campi Obbligatori

Ogni campo obbligatorio deve avere:
* L'attributo `required` abilitato nel widget.
* Un'etichetta con asterisco visibile: `Nome *`.
* Un messaggio di errore in lingua quando il campo è vuoto al submit: `"Campo obbligatorio"`.

Non è sufficiente impedire il submit — l'utente deve sapere quale campo è mancante e perché.

### Validazione con Pattern (REGEX)

Quando il formato di un campo è vincolato, usare una REGEX o un validatore custom che mostri all'utente **come inserire correttamente il dato**, non solo che il dato è sbagliato.

Il messaggio di errore deve contenere un esempio concreto del formato atteso.

| Campo | REGEX esempio | Messaggio errore |
| ----- | ------------- | ---------------- |
| Email | `^[\w.+-]+@[\w-]+\.[a-z]{2,}$` | `Inserire un'email valida (es. nome@azienda.it)` |
| Codice fiscale | `^[A-Z]{6}[0-9]{2}[A-Z][0-9]{2}[A-Z][0-9]{3}[A-Z]$` | `Formato: RSSMRA80A01H501U` |
| Telefono IT | `^(\+39)?[\s-]?3\d{2}[\s-]?\d{6,7}$` | `Inserire un numero mobile italiano (es. +39 333 1234567)` |
| CAP italiano | `^\d{5}$` | `Il CAP deve essere composto da 5 cifre (es. 40121)` |
| Partita IVA | `^\d{11}$` | `La partita IVA deve essere composta da 11 cifre` |

### Quando scatta la validazione

| Evento | Comportamento |
| ------ | ------------- |
| `on blur` (uscita dal campo) | Validare il singolo campo — errore immediato e localizzato |
| `on submit` (click Salva) | Validare tutti i campi obbligatori non compilati |
| `on change` (mentre si digita) | Solo per feedback positivo (es. indicatore di forza password) — non mostrare errori mentre l'utente scrive |

---

## 11. Classi Visive e Gerarchia dei Bottoni

### Classi Semantiche Bootstrap

Tutta la comunicazione visiva verso l'utente — bottoni, badge, alert, panel, toast, label di stato — deve usare le classi semantiche Bootstrap. Non usare colori arbitrari o classi CSS custom per veicolare un significato che Bootstrap già copre.

| Classe | Significato | Quando usarla |
| ------ | ----------- | ------------- |
| `primary` | Azione principale | Il bottone più importante dell'area (es. Salva, Conferma, Cerca) |
| `secondary` / `outline` | Azione secondaria | Azioni accessorie (es. Indietro, Annulla, Esporta) |
| `success` | Esito positivo | Operazione completata, record salvato, stato attivo |
| `info` | Informazione neutra | Messaggi informativi, hint, stati informativi |
| `warning` | Attenzione richiesta | Dati incompleti, scadenze imminenti, azioni con conseguenze |
| `danger` | Operazione distruttiva o errore | Eliminazione, errore bloccante, stato critico |
| `light` / `dark` | Neutro | Elementi decorativi o di sfondo senza significato semantico |

Questi valori si applicano a qualsiasi componente WaveMaker che accetta una classe di stile: `Button`, `Label`, `Badge`, `Alert`, `Panel`, `Chips`.

### Gerarchia dei Bottoni

**Regola fondamentale: al massimo un bottone `primary` per area visiva.**

Un'area visiva è qualsiasi sezione autonoma della pagina: un form, una toolbar, un dialog, un pannello laterale. Avere due bottoni `primary` nella stessa area comunica che entrambe le azioni sono ugualmente importanti, annullando la gerarchia.

```
✔  Area form:
   [Salva]primary   [Annulla]secondary   [Elimina]danger

✗  Area form:
   [Salva]primary   [Esporta]primary   [Annulla]secondary
```

| Tipo di azione | Classe | Quantità per area |
| -------------- | ------ | ----------------- |
| Azione principale (submit, conferma) | `primary` | Massimo 1 |
| Azioni secondarie (annulla, indietro, esporta) | `secondary` o `outline` | Illimitato |
| Azioni distruttive (elimina, reset totale) | `danger` | Massimo 1 |
| Azioni informative (visualizza dettaglio, help) | `info` o `light` | Illimitato |

### Alert e Messaggi Inline

I messaggi contestuali all'interno delle pagine (non toast) devono usare il componente Alert di WaveMaker con la classe appropriata:

```
[info]    ℹ  Nessuna modifica rilevata dall'ultima sincronizzazione.
[success] ✔  Ordine salvato correttamente.
[warning] ⚠  La data di scadenza è tra 3 giorni.
[danger]  ✖  Errore durante il salvataggio. Verificare la connessione.
```

Non usare label con colori CSS custom (`color: red`, `background: yellow`) per comunicare stati semantici.

### Badge e Indicatori di Stato

Le colonne di stato nelle griglie e i badge devono usare le classi semantiche in modo coerente in tutta l'applicazione. Definire una mappatura fissa tra i valori di stato del dominio e le classi Bootstrap:

```
Stato "Attivo"     → badge success
Stato "In attesa"  → badge warning
Stato "Scaduto"    → badge danger
Stato "Bozza"      → badge secondary
Stato "Archiviato" → badge light / dark
```

La mappatura deve essere documentata e applicata uniformemente in tutte le pagine dell'app.

---

## 12. Riepilogo Checklist Frontend

### Naming
- [ ] Ogni pagina principale è tutta minuscola con underscore (`lista_materiali`)
- [ ] Ogni pagina partial inizia con la lettera maiuscola (`FiltroRicerca`)
- [ ] Ogni widget ha nome `<tipoWidget><NomeFunzionalità>` (`buttonConfermaOrdine`)
- [ ] Nessun widget con nome di default (`label1`, `button2`, `datagrid1`)
- [ ] Tutte le variabili sono in camelCase

### Datagrid
- [ ] Ogni datagrid ha il filtro di colonna abilitato (`Multi Column`)
- [ ] Le colonne di tipo "azioni" sono le uniche esentate dal filtro

### Filtri e Reset
- [ ] Ogni area filtro ha un pulsante Reset visibile
- [ ] Il Reset azzera tutti i campi e ricarica i dati senza filtri
- [ ] Il Reset riporta la paginazione alla prima pagina
- [ ] Il pulsante Reset è visivamente distinguibile dal pulsante Cerca

### Feedback e Errori
- [ ] `onServiceError` in `app.js` gestisce tutti gli errori XHR
- [ ] I messaggi di errore provengono dai file i18n, mai testo hardcoded
- [ ] `appNotification` presente, oppure meccanismo alternativo documentato

### Operazioni Distruttive
- [ ] Ogni Delete o azione irreversibile è preceduta da un dialog di conferma
- [ ] Il dialog identifica esplicitamente l'oggetto da eliminare
- [ ] Il bottone di conferma usa stile `danger`

### Formattazione
- [ ] Date visualizzate con formatter WaveMaker (`dd/MM/yyyy`)
- [ ] Timestamp convertiti nel fuso orario locale dell'utente
- [ ] Valute e numeri formattati con il locale corretto (`,` decimale, `.` migliaia per IT)

### i18n
- [ ] File `i18n/<lingua>.json` presente per ogni lingua supportata
- [ ] Tutte le label, messaggi di errore e testi UI provengono dai file i18n
- [ ] Formatter configurati per il locale attivo (non hardcodati)

### Validazione Form
- [ ] Campi obbligatori marcati con `required` e asterisco visibile
- [ ] Campi con formato vincolato hanno REGEX con messaggio che mostra un esempio
- [ ] Validazione on-blur per il singolo campo, on-submit per il form completo
- [ ] Messaggi di validazione in lingua (da file i18n)

### Classi Visive e Bottoni
- [ ] Al massimo un bottone `primary` per area visiva
- [ ] Azioni secondarie usano `secondary` o `outline`
- [ ] Azioni distruttive usano `danger`
- [ ] Alert e messaggi inline usano le classi Bootstrap (`success`, `info`, `warning`, `danger`)
- [ ] Badge di stato hanno una mappatura fissa e coerente tra valore dominio e classe Bootstrap
- [ ] Nessun colore CSS arbitrario usato per comunicare un significato semantico
