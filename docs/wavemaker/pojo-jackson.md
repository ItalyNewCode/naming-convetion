# POJO e Annotazioni Jackson — WaveMaker

Quando gli autori scrivono classi Java custom (DTO, modelli di integrazione, modelli condivisi), queste devono rispettare le convenzioni Jackson usate da WaveMaker e le specifiche JavaBeans. Una classe mal annotata produce JSON inconsistente, errori di deserializzazione o comportamenti imprevisti con Spring MVC e Hibernate.

---

## 1. Struttura Base Obbligatoria

Ogni POJO custom deve avere:

```java
import java.io.Serializable;
import java.util.Objects;
import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.annotation.JsonPropertyOrder;

@JsonInclude(JsonInclude.Include.NON_NULL)
@JsonPropertyOrder({ "campo1", "campo2", "campo3" })
public class NomeClasse implements Serializable {

    @JsonProperty("campo1")
    private String campo1;

    @JsonProperty("campo2")
    private Integer campo2;

    public String getCampo1() { return campo1; }
    public void setCampo1(String campo1) { this.campo1 = campo1; }

    public Integer getCampo2() { return campo2; }
    public void setCampo2(Integer campo2) { this.campo2 = campo2; }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof NomeClasse)) return false;
        NomeClasse that = (NomeClasse) o;
        return Objects.equals(campo1, that.campo1) &&
               Objects.equals(campo2, that.campo2);
    }

    @Override
    public int hashCode() {
        return Objects.hash(campo1, campo2);
    }
}
```

**Elementi obbligatori:**

| Elemento | Motivazione |
| -------- | ----------- |
| `implements Serializable` | Richiesto da WaveMaker per la serializzazione HTTP e la cache di sessione |
| `@JsonInclude(NON_NULL)` | Esclude i campi null dalla risposta JSON — produce output più pulito e riduce il payload |
| `@JsonPropertyOrder` | Garantisce un ordine stabile dei campi nel JSON — fondamentale per la leggibilità e i test |
| `@JsonProperty` sui campi | Mappa esplicitamente il nome Java al nome JSON |
| `equals()` e `hashCode()` | Richiesti per il corretto funzionamento in collection, cache e confronti WaveMaker |
| Getter e setter standard | Richiesti da Jackson e dal container Spring per la serializzazione/deserializzazione |

---

## 2. Annotazione `@JsonProperty` — Posizionamento Corretto

`@JsonProperty` deve essere annotata **una sola volta**, sul campo. Non duplicarla su getter e setter.

```java
// CORRETTO — annotazione solo sul campo
@JsonProperty("nome_cliente")
private String nomeCliente;

public String getNomeCliente() { return nomeCliente; }
public void setNomeCliente(String nomeCliente) { this.nomeCliente = nomeCliente; }
```

```java
// VIETATO — annotazione duplicata su campo, getter e setter
@JsonProperty("nome_cliente")
private String nomeCliente;

@JsonProperty("nome_cliente")      // ← ridondante, causa ambiguità
public String getNomeCliente() { return nomeCliente; }

@JsonProperty("nome_cliente")      // ← ridondante
public void setNomeCliente(String v) { this.nomeCliente = v; }
```

---

## 3. Mapping JSON ↔ Java — Convenzioni di Naming

WaveMaker usa **camelCase** nel layer Java e nella REST API. Se il sorgente esterno usa snake_case (es. risposte di API esterne), `@JsonProperty` risolve la mappatura:

| JSON esterno | Campo Java | `@JsonProperty` |
| ------------ | ---------- | --------------- |
| `"nome_cliente"` | `nomeCliente` | `@JsonProperty("nome_cliente")` |
| `"timestamp_richiesta"` | `timestampRichiesta` | `@JsonProperty("timestamp_richiesta")` |
| `"id_articolo"` | `idArticolo` | `@JsonProperty("id_articolo")` |

Quando il nome JSON coincide con il nome del campo Java (già camelCase), `@JsonProperty` è facoltativo ma **raccomandato** per rendere esplicito il contratto:

```java
@JsonProperty("statoPreventivo")   // esplicito — preferito
private String statoPreventivo;
```

---

## 4. Campi Boolean — Specifica JavaBeans

Per i campi booleani, il getter deve usare il prefisso `is` come da specifica JavaBeans. Jackson serializza il campo nel JSON **senza** il prefisso `is`.

```java
@JsonProperty("attivo")
private Boolean attivo;

public Boolean isAttivo() { return attivo; }       // ← getter con "is"
public void setAttivo(Boolean attivo) { this.attivo = attivo; }
```

JSON prodotto:
```json
{ "attivo": true }
```

Non usare `getAttivo()` per i boolean — Jackson non lo riconoscerà come getter booleano standard.

---

## 5. Annotazioni WaveMaker Specifiche

### `@ColumnAlias` — Risposta da Query e Stored Procedure

Usata nei DTO generati da WaveMaker per mappare il nome della colonna SQL al campo Java. L'autore deve usarla nei DTO custom di query quando il nome SQL differisce dal nome Java:

```java
import com.wavemaker.runtime.data.annotations.ColumnAlias;

public class ArticoloDisponibileResponse implements Serializable {

    @ColumnAlias("CODICE_ARTICOLO")
    private String codiceArticolo;

    @ColumnAlias("QTA_DISPONIBILE")
    private Integer qtaDisponibile;

    // getter e setter
}
```

`@ColumnAlias` è alternativa a `@JsonProperty` nei DTO di risposta query: gestisce il mapping dalla colonna SQL (MAIUSCOLO o snake_case) al campo Java camelCase. Non usare entrambe sulla stessa classe senza necessità.

### `@NotNull` — Validazione dei Campi Obbligatori

Per i DTO di richiesta (stored procedure, endpoint custom), marcare i campi obbligatori con `@NotNull` di Jakarta Validation:

```java
import jakarta.validation.constraints.NotNull;
import com.fasterxml.jackson.annotation.JsonProperty;

public class SetStatoRequest implements Serializable {

    @JsonProperty("stagione")
    @NotNull
    private String stagione;

    @JsonProperty("nuovo_stato")
    @NotNull
    private String nuovoStato;

    // getter e setter
}
```

---

## 6. Integrazione con API REST Esterne

I DTO che modellano risposte di API esterne devono usare `@JsonIgnoreProperties(ignoreUnknown = true)` per resistere a campi aggiuntivi non previsti nel contratto:

```java
import com.fasterxml.jackson.annotation.JsonIgnoreProperties;
import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.annotation.JsonProperty;

@JsonInclude(JsonInclude.Include.NON_NULL)
@JsonIgnoreProperties(ignoreUnknown = true)
public class RootResponse implements Serializable {

    @JsonProperty("timestamp_richiesta")
    private String timestampRichiesta;

    @JsonProperty("dati")
    private List<ResponseDatiEntryItem> dati;

    @JsonProperty("dato_trovato")
    private Boolean datoTrovato;

    // getter e setter, equals, hashCode
}
```

Senza `@JsonIgnoreProperties(ignoreUnknown = true)`, se l'API esterna aggiunge un campo non mappato Jackson lancia `UnrecognizedPropertyException` e la chiamata fallisce.

---

## 7. Campi da Nascondere — `@JsonIgnore`

I campi che non devono apparire nel JSON (password, token interni, dati tecnici) vanno annotati con `@JsonIgnore`:

```java
@JsonIgnore
private String passwordHash;

@JsonIgnore
private transient String cacheKey;
```

Non usare `@JsonIgnore` su campi che dovrebbero semplicemente essere null: per quelli usa `@JsonInclude(NON_NULL)` a livello di classe.

---

## 8. Tipi di Campo — Mapping Consigliati

| Tipo SQL / Java | Tipo Java nel POJO | Note |
| --------------- | ------------------ | ---- |
| `VARCHAR`, `TEXT` | `String` | |
| `INT`, `SMALLINT` | `Integer` | Preferire wrapper a primitivi — supporta null |
| `BIGINT` | `Long` | |
| `DECIMAL`, `NUMERIC` | `BigDecimal` | Mai `double` per valori monetari |
| `BOOLEAN` | `Boolean` | Getter con `is` prefix |
| `DATE` | `java.time.LocalDate` | |
| `DATETIME`, `TIMESTAMP` | `java.time.LocalDateTime` | |
| `TIMESTAMP WITH TIME ZONE` | `java.time.OffsetDateTime` | |
| Array / lista | `List<T>` | Inizializzare a `new ArrayList<>()` |
| Oggetto annidato | Classe separata | Non usare `Map<String, Object>` se il contratto è noto |

---

## 9. Checklist POJO Custom

- [ ] La classe implementa `Serializable`
- [ ] `@JsonInclude(JsonInclude.Include.NON_NULL)` presente a livello di classe
- [ ] `@JsonPropertyOrder` presente con l'elenco dei campi in ordine logico
- [ ] `@JsonProperty` su ogni campo, posizionata **solo sul campo** (non su getter e setter)
- [ ] Getter e setter presenti per tutti i campi
- [ ] Campi boolean con getter `isXxx()` (non `getXxx()`)
- [ ] `equals()` e `hashCode()` implementati su tutti i campi significativi
- [ ] `@JsonIgnoreProperties(ignoreUnknown = true)` su DTO che deserializzano API esterne
- [ ] `@JsonIgnore` sui campi da non serializzare
- [ ] `@NotNull` sui campi obbligatori nei DTO di richiesta
- [ ] `@ColumnAlias` usato nei DTO di risposta query al posto di `@JsonProperty`
- [ ] Nessun tipo primitivo (`int`, `boolean`) — usare i wrapper (`Integer`, `Boolean`)
- [ ] Nessun `double` per valori monetari — usare `BigDecimal`
