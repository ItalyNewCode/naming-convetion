# db/sqlserver

Schema SQL del database Microsoft SQL Server collegato al progetto.

## Contenuto

- `schema.sql` — script completo di creazione dello schema (tabelle, indici, constraint, view, stored procedure, function, trigger)

## Note

- Aggiornare `schema.sql` in ogni PR che modifica la struttura del database.
- Separare i blocchi con `GO` come richiesto da SQL Server.
- Il naming degli oggetti deve rispettare [docs/database.md](../../../../docs/database.md).
- Per la configurazione ORM corrispondente vedi [docs/wavemaker/orm.md](../../../../docs/wavemaker/orm.md).
