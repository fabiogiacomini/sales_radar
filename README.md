# sales_radar
Sistema di monitoraggio che ogni giorno raccoglie da tre fonti pubbliche i segnali di aziende ed enti che si stanno muovendo sulla comunicazione, e li recapita alla forza vendita in un'unica mail leggibile in due minuti. Questo repository è una scheda di progetto: documenta l'architettura e le scelte, **non contiene il workflow eseguibile**.
## Il problema

Un commerciale che vende servizi di comunicazione ha bisogno di sapere tre cose: chi ha
appena cambiato responsabile marketing, chi ha una gara aperta, chi ha appena affidato un
incarico a qualcun altro. Le tre informazioni vivono in tre posti diversi, nessuno dei
quali è pensato per essere letto tutti i giorni.

Il rischio, automatizzando, è il rumore: un sistema che segnala troppo viene ignorato dopo
una settimana. Il progetto è nato con un vincolo esplicito — **una mail al giorno, mai due**
— e tutto il resto è disceso da lì.

---

## Le tre fonti

| Sezione | Fonte | Cosa intercetta |
|---|---|---|
| **News** | Feed RSS di Google News su un paniere di aziende sorvegliate | Cambi di responsabile marketing, nuove campagne, lanci di prodotto |
| **Gare** | API TED — Tenders Electronic Daily, il portale europeo degli appalti | Bandi sopra soglia, filtrati per codici CPV di comunicazione e marketing, committente italiano |
| **Intel** | Piattaforma Pubblicità Legale di ANAC | Affidamenti sotto soglia: non sono gare a cui partecipare, sono **la lista degli enti che stanno comprando comunicazione adesso** |

La terza fonte è la più interessante delle tre, e non era nel piano iniziale. Gli
affidamenti diretti non offrono un'occasione immediata — sono già assegnati — ma dicono
quali enti hanno un budget attivo, a chi si sono rivolti e con che importo. È materiale da
business intelligence, non da risposta a bando, e per questo nella mail ha una sezione
propria invece di essere mescolato alle gare.

---

## Architettura

```
Trigger giornaliero
      │
      ├── RSS Google News ──── filtro per parole chiave ──┐
      │                                                   │
      ├── API TED ──────────── filtro CPV + paese ────────┤── Merge (3 ingressi)
      │                                                   │
      └── API ANAC PVL ─────── filtro full-text ──────────┘
                                                          │
                                            Composizione della mail
                                          (sezioni: News · Gare · Intel)
                                                          │
                                                   Invio giornaliero
```

---

## Scelte progettuali

**Una sola mail, sempre, anche quando non c'è nulla.** Se non emerge niente di rilevante
parte comunque un messaggio che lo dice. Sembra un dettaglio, ed è invece la funzione più
importante: distingue *«oggi non è successo nulla»* da *«il sistema è morto tre settimane
fa e nessuno se n'è accorto»*. Un monitoraggio silenzioso in caso di guasto è un
monitoraggio che non esiste.

**Aggregazione, non notifica per evento.** La prima versione mandava una mail per notizia.
Funzionava per un'azienda sorvegliata; con dieci sarebbe diventata spam interno. Un unico
nodo di composizione raccoglie tutto e costruisce un solo messaggio, con un ordine di
sezioni fisso — News, Gare, Intel — perché la ripetizione dello stesso formato è ciò che
rende la lettura veloce.

**Filtro per parole chiave, non classificazione con modello.** Il ramo news scarta a monte
tutto ciò che non contiene i termini rilevanti. Meno elegante di una classificazione
semantica, molto più economico e soprattutto ispezionabile: quando un risultato non
convince, si legge la regola, non si interroga un modello.

**Fonti pubbliche e senza chiave dove possibile.** TED non richiede autenticazione;
l'endpoint ANAC utilizzato non è documentato ma è di pubblico dominio. Il costo di
esercizio del monitoraggio è di fatto nullo.

---

## Stato

In produzione, in uso quotidiano dalla forza vendita. È il sistema da cui è nato il
progetto successivo, l'[Agente di Analisi Brand](../agente-analisi-brand): il radar segnala
*chi* vale la pena avvicinare, l'agente prepara *con cosa* presentarsi.

---

## Stack

Orchestrazione n8n autoinstallata su VPS · API TED (Unione Europea) · API ANAC ·
feed RSS Google News · JavaScript nei nodi di elaborazione · invio SMTP.

---

## Licenza e utilizzo

**Tutti i diritti riservati.** Vedi License (https://github.com/fabiogiacomini/sales_radar/blob/main/license).

Questo repository ha finalità dimostrative. Il workflow eseguibile non è incluso e non è
concesso in uso, riproduzione o opera derivata. Per collaborazioni: contattami.
