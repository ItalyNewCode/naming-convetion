# db/oracle

Schema SQL del database Oracle collegato al progetto.

## Contenuto

- `schema.sql` — script completo di creazione dello schema (tabelle, sequenze, indici, constraint, view, procedure, function, trigger)

## Note

- Aggiornare `schema.sql` in ogni PR che modifica la struttura del database.
- In Oracle lo schema coincide con l'utente DB — indicare l'utente nel blocco di intestazione del file.
- Il naming degli oggetti deve rispettare [docs/database.md](../../../../docs/database.md).
- Per la configurazione ORM corrispondente vedi [docs/wavemaker/orm.md](../../../../docs/wavemaker/orm.md).
