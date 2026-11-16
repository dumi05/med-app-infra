# med-app-infra

Infrastruttura per avviare con un solo comando l'intera applicazione **Studio Medico "La Mole"** (database, backend e frontend) con Docker Compose.

Questo repository non contiene codice applicativo: contiene il `docker-compose.yml`, il modello delle variabili d'ambiente (`.env.example`) e i diagrammi di progetto.

## I tre repository

L'applicazione è divisa in tre repository. Per usare il compose servono **tutti e tre**:

| Repository | Contenuto | Link |
|---|---|---|
| `med-app-infra` (questo) | Docker Compose, configurazione, diagrammi | https://github.com/dumi05/med-app-infra |
| `med-app-be` | Backend: API REST in Java 21 e Spring Boot 4.1 | https://github.com/dumi05/med-app-be |
| `med-app-fe` | Frontend: Single Page Application in Angular 20 | https://github.com/dumi05/med-app-fe |

## Prerequisiti

- **Docker** con Compose v2 (Docker Desktop su Windows e macOS). Verifica con `docker compose version`.
- **Git**.
- La porta **8081** libera sul computer.
- Connessione a Internet al primo avvio: le immagini vengono scaricate e i progetti compilati dentro i container.

Non servono JDK, Maven o Node.js installati: la compilazione avviene nei container.

## Struttura delle cartelle

Il compose compila le immagini dalle cartelle `../med-app-be` e `../med-app-fe`. I tre repository devono quindi essere **cartelle sorelle, con questi nomi esatti**:

```text
med-app/                  (il nome di questa cartella è libero)
├── med-app-infra/        <- qui si lancia il compose
├── med-app-be/
└── med-app-fe/
```

## Avvio

1. Scarica i tre repository nella stessa cartella:

```bash
mkdir med-app && cd med-app
git clone https://github.com/dumi05/med-app-infra.git
git clone https://github.com/dumi05/med-app-be.git
git clone https://github.com/dumi05/med-app-fe.git
```

2. Crea il file `.env` a partire dal modello e modifica i valori (vedi la tabella sotto):

```bash
cd med-app-infra
cp .env.example .env
```

Su PowerShell: `Copy-Item .env.example .env`.

3. Compila e avvia tutto:

```bash
docker compose up --build
```

Il primo avvio richiede alcuni minuti (scaricamento delle dipendenze e compilazione). Quando il backend ha terminato l'avvio, apri **http://localhost:8081**.

Per lasciarlo in background aggiungi `-d`: `docker compose up --build -d`.

## Configurazione (`.env`)

Il file `.env` non è nel repository (è escluso da `.gitignore`): contiene segreti e va creato da ogni persona che avvia il progetto.

| Variabile | Significato |
|---|---|
| `DB_NAME` | Nome del database PostgreSQL |
| `DB_USERNAME`, `DB_PASSWORD` | Credenziali del database |
| `JWT_SECRET` | Chiave con cui il backend firma i token di accesso: **almeno 32 caratteri** |
| `ADMIN_PSW` | Password iniziale dell'amministratore |
| `DEMO_DATA` | `true` per popolare l'applicazione con dati di prova (default `false`) |
| `DEMO_PASSWORD` | Password di tutti gli account di prova (serve solo con `DEMO_DATA=true`) |

Sostituisci tutti i valori `change-me...` con valori tuoi. Le password devono rispettare la politica dell'applicazione: almeno 12 caratteri e non presenti in un elenco di password comuni.

## Primo accesso

- **Amministratore**: e-mail `admin@studio-medico.it`, password uguale a `ADMIN_PSW`. Al primo accesso viene richiesto di cambiarla.
- **Dati di prova** (solo con `DEMO_DATA=true`): vengono creati cinque medici, vari pazienti e visite in tutti gli stati. Le e-mail hanno la forma `nome.cognome@demo.it` (ad esempio `elena.ferri@demo.it`, `luca.bianchi@demo.it`), l'amministratore di prova è `demo.admin@studio-medico.it`, e la password è `DEMO_PASSWORD`. L'inserimento dei dati di prova non viene ripetuto se esistono già.

## Cosa viene avviato

| Servizio | Immagine | Ruolo |
|---|---|---|
| `db` | `postgres:16` | Database. I dati sono nel volume `db-data` |
| `backend` | compilato da `../med-app-be` | API REST sulla porta 8080 interna. Gli allegati dei referti sono nel volume `attachments` |
| `frontend` | compilato da `../med-app-fe` | Interfaccia web servita da nginx. È l'unico servizio raggiungibile dall'esterno, su **8081**; le richieste a `/api/` vengono inoltrate al backend |

Il backend parte solo quando il database risponde (controllo `pg_isready`).

## Comandi utili

```bash
docker compose logs -f backend      # segue i log del backend
docker compose ps                   # stato dei servizi
docker compose down                 # ferma e rimuove i container (i dati restano)
docker compose down -v              # ferma e CANCELLA anche database e allegati
docker compose up --build -d        # ricompila dopo una modifica al codice
```

## Problemi comuni

- **Errore sul percorso di build (cartella `med-app-be` o `med-app-fe` non trovata)**: i repository non sono cartelle sorelle con i nomi corretti. Controlla la struttura.
- **Porta 8081 occupata**: cambia la mappatura `"8081:80"` nel compose.
- **Ho cambiato `DB_PASSWORD` o `DB_NAME` e il backend non si collega**: PostgreSQL applica queste variabili solo alla creazione del volume. Esegui `docker compose down -v` e riavvia (cancella i dati).
- **Ho acceso `DEMO_DATA` dopo il primo avvio**: modifica `.env` e rilancia `docker compose up -d`, il backend viene ricreato ed esegue il popolamento.
- **Swagger UI non raggiungibile**: nel compose la porta 8080 del backend non è pubblicata. Per usarla avvia il backend in locale (vedi il README di `med-app-be`) oppure aggiungi temporaneamente `ports: ["8080:8080"]` al servizio `backend`.
- **Ispezionare il database con un client (ad esempio pgAdmin 4)**: aggiungi temporaneamente `ports: ["5433:5432"]` al servizio `db`.

## Note

Questa configurazione serve per provare e dimostrare l'applicazione in locale: **non configura HTTPS** e non è pensata per essere esposta su Internet così com'è.

## Documentazione

La cartella `diagrams/` contiene i diagrammi di progetto. Ogni diagramma è presente in due file con lo stesso nome: il sorgente `.drawio`, modificabile con [draw.io](https://app.diagrams.net), e l'immagine `.png`.

| File | Diagramma | Contenuto |
|---|---|---|
| `casi-d-uso` | Casi d'uso (UML) | I quattro attori (paziente, medico, amministratore, delegato) e le funzionalità principali |
| `er` | Entità-relazione (schema concettuale) | Le 15 entità del dominio, le relazioni e le cardinalità (min,max) |
| `classi-dominio` | Classi (UML) | Le entità del backend con attributi e associazioni |
| `classi-servizi` | Classi (UML) | I servizi del backend e le loro dipendenze |
| `stati-appuntamento` | Macchina a stati (UML) | Ciclo di vita dell'appuntamento: prenotato, completato, fatturato, refertato, annullato |
| `stati-delega-medici` | Macchina a stati (UML) | Ciclo di vita della delega tra medici |
| `seq-prenotazione` | Sequenza (UML) | Prenotazione di una visita, con i casi di errore |
| `seq-referto-fatturazione` | Sequenza (UML) | Registrazione del pagamento, emissione della fattura e scrittura del referto |
| `architettura-container` | Architettura | I tre container (frontend, backend, database) e i volumi |

I diagrammi dei casi d'uso, ER e di sequenza della prenotazione sono quelli riportati nell'elaborato del Project Work. La documentazione delle API non è in questo repository: è generata dal codice del backend ed è consultabile dalla Swagger UI (vedi il README di `med-app-be`).
