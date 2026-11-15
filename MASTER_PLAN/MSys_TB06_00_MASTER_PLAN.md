# MSys_TB06 – Master Plan del menu dei progetti

Mini-progetto **TB06_MENU** del super-progetto MSys, cartella `C:\CASA\LINGUAGGI\ACCESS\PROGETTI_MDB\MSYS\MSys_TB06_MENU\`.

Tre database back-end lavorano insieme per:

1. **creare il progetto**, con codice progetto, descrizione e note (TB06 – VOCI);
2. **dare a ogni progetto le sue funzioni**, cioè le voci di menu che il progetto mette a disposizione (TB07 – FUNZIONI);
3. **collegare a ogni progetto gli oggetti da utilizzare**: form, query, moduli, report, macro e tabelle (TB08 – OGGETTI).

Un database front-end, `GE_MSys_TB06_MENU.mdb`, contiene le maschere per gestire tutto questo e il codice che esegue le funzioni del progetto scelto.

Aggiornato il 15/11/2026, dopo la rinumerazione da TB01/02/03 a TB06/07/08 (capitolo 12).

## 1. I file in sintesi

| File (in `MDB\`) | Ruolo | Tabella principale | Righe | Chiave |
| --- | --- | --- | --- | --- |
| `MSys_TB06_MENU_VOCI.mdb` | back-end: **progetti** | `MSys_TB06_MENU_VOCI` | 7 | `CodMenu_s` |
| `MSys_TB07_MENU_FUNZIONI.mdb` | back-end: **funzioni di ogni progetto** | `MSys_TB07_MENU_FUNZIONI` | 23 | `CodFunzioni_s` |
| `MSys_TB08_MENU_OGGETTI.mdb` | back-end: **oggetti di ogni progetto** | `MSys_TB08_MENU_OGGETTI` | 0 | `ID_MENU` |
| `GE_MSys_TB06_MENU.mdb` | front-end: maschere, query e codice | (solo collegamenti) | – | – |

Ogni back-end ha anche una tabella `_TMP` gemella, che il front-end usa come area di lavoro per il progetto scelto (capitolo 6.3). Sono tutti file Access 2000 (`.mdb`).

## 2. Come si collegano

```
                      MSys_TB06_MENU_VOCI           (1 riga = 1 progetto)
                      CodMenu_s  (chiave)  es. LLPPGestMENU
                      COD_PROGETTO_s           LLPPGest
                      Menu_s                   GESTIONE_LAVORI_PUBBLICI
                      NoteMenu_s, COMANDO_s
                         |                         |
            CodMenu_s    | 1 : N          1 : N    |  CodMenu_s
                         v                         v
    MSys_TB07_MENU_FUNZIONI                 MSys_TB08_MENU_OGGETTI
    (1 riga = 1 funzione del progetto)      (1 riga = 1 oggetto del progetto)
    CodFunzioni_s (chiave)                  ID_MENU (contatore)
    Funzioni_s, ClassFunzioni_s,            CodMenu_s
    NoteFunzioni_s                          ... campi dell'oggetto: da costruire (cap. 5)

  GE_MSys_TB06_MENU.mdb  collega tutte le tabelle sopra (e le loro _TMP)
     MSys_TB06_Frm01_M01_GE_VOCI  -> sceglie il progetto, copia progetto e funzioni nelle _TMP,
                                     elenca le funzioni ed esegue quella scelta
```

- Il campo che lega tutto è **`CodMenu_s`**, il codice menu del progetto. Per convenzione è il codice progetto seguito da `MENU` (per esempio `GeCondomMENU`).
- Nei database non ci sono **relazioni** definite (né integrità referenziale): il legame esiste solo nelle query, nelle sottomaschere (`CodMenu_s` → `CodMenu_s`) e nel codice.

## 3. TB06 – Progetti (MSys_TB06_MENU_VOCI)

### 3.1 Campi

| Campo | Tipo | Significato |
| --- | --- | --- |
| `CodMenu_s` | Testo 25 | **codice menu del progetto**, chiave primaria, univoco |
| `COD_PROGETTO_s` | Testo 45 | codice del progetto (per esempio `MSys`, `LLPPGest`) |
| `Menu_s` | Testo 50 | **descrizione** del progetto, mostrata nel menu |
| `PROGETTO_s` | Testo 50 | nome esteso del progetto (oggi sempre vuoto) |
| `NoteMenu_s` | Testo 255 | **note esplicative** del progetto (oggi sempre vuoto) |
| `COMANDO_s` | Testo 255 | comando collegato al progetto (oggi sempre vuoto) |
| `IdOrd` | Lungo | ordine di visualizzazione nel menu |
| `bCanc`, `bAgg`, `bScel` | Sì/No | flag di servizio: da cancellare, aggiornato, scelto |
| `IDKeyArch`, `Msg01` | Lungo, Testo | chiave di archivio e messaggio di lavoro |
| `ID` | Contatore | identificativo interno |
| `DATA_INS`, `DATA_AGG`, `TIMEOPER` | Data | date di inserimento, aggiornamento e operazione |

I campi `00___`, `pk___`, `01___`, `02___`, `03___`, `ag___` sono **separatori visivi** (Testo 1): servono solo a dividere i gruppi di campi in struttura.

### 3.2 Progetti presenti

| IdOrd | CodMenu_s | COD_PROGETTO_s | Menu_s (descrizione) |
| --- | --- | --- | --- |
| 1 | `GeCasaMENU` | `GeCasa` | GESTIONE CASA |
| 2 | `GeCondomMENU` | `GeCondom` | Gestione Condomini |
| 3 | `LLPPGestMENU` | `LLPPGest` | GESTIONE_LAVORI_PUBBLICI |
| 4 | `LTT_MENU` | `LTT` | LTT ESTRAZIONI |
| 5 | `MSysMENU` | `MSys` | PROGETTO MSys |
| 6 | `Pres3000_Menu` | `Pres3000` | GESTIONE_PRESENZE |
| 7 | `RUST_Menu` | `RUST` | GESTIONE RUST |

Le note (`NoteMenu_s`) e il nome esteso (`PROGETTO_s`) sono vuoti per tutti i progetti: vanno compilati (capitolo 11, Fase 2).

## 4. TB07 – Funzioni dei progetti (MSys_TB07_MENU_FUNZIONI)

### 4.1 Campi

| Campo | Tipo | Significato |
| --- | --- | --- |
| `CodFunzioni_s` | Testo 25 | **codice funzione**, chiave primaria; è il valore che il codice usa per eseguire la funzione |
| `CodMenu_s` | Testo 25 | progetto a cui appartiene la funzione (legame con TB06) |
| `Funzioni_s` | Testo 255 | descrizione della funzione, mostrata nel menu (**indice univoco**) |
| `ClassFunzioni_s` | Testo 50 | classificazione, per esempio `Funzioni Cdm`, `Stampe Cdm`, `Visualizzazioni` |
| `NoteFunzioni_s` | Testo 255 | note esplicative della funzione |
| `bCanc`, `bAgg`, `bScel`, `IDKeyArch`, `IdOrd`, `Msg01`, `ID`, `DATA_INS`, `DATA_AGG`, `TIMEOPER` | | campi di servizio, come in TB06 |

### 4.2 Funzioni presenti per progetto

| CodMenu_s | Funzioni | Esempi |
| --- | --- | --- |
| `GeCasaMENU` | 1 | `GeSpeseCasa` |
| `GeCondomMENU` | 10 (1 separatore) | `AnagraficaCdm`, `Bollettini`, `PreventivoCdm`, `RipartoPrevCdm`, `StampAnagrafCdm` |
| `LLPPGestMENU` | 7 | `LLPP_ATTI`, `LLPPCUPCIG`, `LLPPDETERM`, `LLPPTRASP`, `PROC_LLPP` |
| `LTT_MENU` | 1 | `AMB` |
| `RUST_Menu` | 1 | `RustCmd` |
| `MSysMENU` | **0** | le funzioni di MSys sono in `MSys_TB07_MENU_FUNZIONI_NW` (4.3) |
| `ParamCond` | 1 | `GestioneCdm` – **progetto inesistente in TB06** |
| `Pres3000MENU` | 1 | `CalendarioGiorn` – **in TB06 il codice è `Pres3000_Menu`** |
| (vuoto) | 1 | riga senza progetto (`1`, `2`) |

### 4.3 Tabella MSys_TB07_MENU_FUNZIONI_NW

Contiene 49 funzioni, in gran parte del progetto MSys (`CodMenu_s = "MSys"`), con un campo in più: `TIPO_FUNZIONE_s`, che raggruppa le funzioni per tipo (`SALVA`, `CANCELLA`, `COLLEGA`, `ESPORTA`, `IMPORTA`, `ACCODA`). Per esempio: `01_SALVA_FORM`, `02_CANCELLA_QUERY`. È una versione più nuova della tabella funzioni, non ancora usata dal front-end e non collegata in `GE_MSys_TB06_MENU.mdb`.

## 5. TB08 – Oggetti dei progetti (MSys_TB08_MENU_OGGETTI)

### 5.1 Stato attuale

La tabella è **vuota** e ha solo il campo di legame `CodMenu_s` (Testo **10**, troppo corto per codici come `LLPPGestMENU`), il contatore `ID_MENU` e i campi di servizio. Nel front-end non c'è nessuna maschera né codice che la usi. Il terzo scopo del sistema, collegare a ogni progetto gli oggetti da utilizzare, è quindi **ancora da costruire**.

### 5.2 Struttura proposta

| Campo | Tipo | Significato |
| --- | --- | --- |
| `ID_MENU` | Contatore | chiave primaria (esistente) |
| `CodMenu_s` | Testo 25 | progetto (legame con TB06), da allargare da 10 a 25 |
| `CodFunzioni_s` | Testo 25 | facoltativo: funzione di TB07 che usa l'oggetto |
| `TIPO_OGGETTO_s` | Testo 20 | `FORM`, `QUERY`, `MODULO`, `REPORT`, `MACRO`, `TABELLA` |
| `NOME_OGGETTO_s` | Testo 255 | nome dell'oggetto in Access |
| `DB_ORIGINE_s` | Testo 255 | database dentro `PROGETTI_MDB\MSYS\` da cui prendere l'oggetto |
| `NoteOggetto_s` | Testo 255 | a cosa serve l'oggetto nel progetto |
| `IdOrd`, `bScel`, `DATA_INS`, `DATA_AGG` | | servizio (esistenti) |

Con questa struttura il catalogo oggetti di TB99 (`Msys_TB99_LINK_Object`) e il catalogo collegamenti (`Msys_TB99_LINK`) possono fornire l'elenco degli oggetti da scegliere. Le tabelle collegate di un progetto si possono poi inviare al suo front-end con il modulo `Mdl05_INVIA_LINK_FRONTEND` di TB99.

## 6. Il front-end GE_MSys_TB06_MENU.mdb

### 6.1 Tabelle collegate

| Link | Database |
| --- | --- |
| `MSys_TB06_MENU_VOCI`, `MSys_TB06_MENU_VOCI_TMP`, `MSys_TB06_{@===@}`, `MSys_TB06_}---@` | `MSys_TB06_MENU_VOCI.mdb` |
| `MSys_TB07_MENU_FUNZIONI`, `MSys_TB07_MENU_FUNZIONI_TMP`, `MSys_TB07_}---@` | `MSys_TB07_MENU_FUNZIONI.mdb` |
| `MSys_TB08_MENU_OGGETTI`, `MSys_TB08_}---@` | `MSys_TB08_MENU_OGGETTI.mdb` |

### 6.2 Maschere

| Maschera | Origine dati | Contenuto |
| --- | --- | --- |
| `MSys_TB06_Frm01_M01_GE_VOCI` | – | **maschera principale**. In testata 4 combo: `GESTIONI >>` (progetti), `FUNZIONI`, `ATTIVITA`, `STAMPE`. Nel corpo 2 schede: `VOCI MENU` e `VOCI -MENU TMP` |
| `MSys_TB06_Frm01_S01_VOCI` | `MSys_TB06_Qry01_01_VOCI` | elenco dei progetti; contiene la sottomaschera delle funzioni collegata su `CodMenu_s` |
| `MSys_TB06_Frm01_S01_VOCI_TMP` | `MSys_TB06_Qry01_01_VOCI_TMP` | il progetto scelto, copiato nella tabella `_TMP` |
| `MSys_TB07_Frm01_M01_GE_FUNZIONI` | – | maschera di gestione delle funzioni, con filtro |
| `MSys_TB07_Frm01_S01_FUNZIONI` | `MSys_TB07_Qry01_01_FUNZIONI` | elenco delle funzioni |
| `MSys_TB07_Frm01_S01_FUNZIONI_TMP` | `MSys_TB07_Qry01_01_FUNZIONI_TMP` | funzioni del progetto scelto |
| `MSys_TB06_Frm00_{@===@}`, `MSys_TB07_Frm00_{@===@}`, `MSys_TB07_Frm01_}---@` | – | separatori nell'elenco degli oggetti |

### 6.3 Come funziona: dalla scelta del progetto all'esecuzione

1. **Scelta del progetto.** Nella combo `GESTIONI >>` (`Cmb_01_TXT`, origine `MSys_TB06_Qry01_01_VOCI`) si sceglie il progetto.
2. **Copia nell'area di lavoro.** L'evento `Cmb_01_TXT_Change` chiama `SALVA_MENU_pFunct(CodMenu_s, COD_PROGETTO_s, Menu_s)`, del modulo `MSys_TB06_Mdl01_VOCI`. I tre valori non devono essere vuoti. La funzione:
   - svuota le tabelle di lavoro (`MSys_TB06_Qry01_01_VOCI_TMP_Delete`, `MSys_TB07_Qry01_01_FUNZIONI_TMP_Delete`);
   - controlla che il progetto abbia funzioni in TB07; se non ne ha, avvisa "NON ESISTE IL MENU ... NELLE FUNZIONI";
   - copia il progetto in `MSys_TB06_MENU_VOCI_TMP`;
   - copia le sue funzioni in `MSys_TB07_MENU_FUNZIONI_TMP` (query `MSys_TB07_Qry01_02_INSERT_FUNZIONI_To_TMP`).
   Lo stesso avviene con il **doppio clic** su `CodMenu_s` nella sottomaschera `MSys_TB06_Frm01_S01_VOCI`.
3. **Elenco delle funzioni.** La combo `FUNZIONI` (`Cmb_03_TXT`, origine `MSys_TB07_Qry01_01_FUNZIONI_TMP`) viene aggiornata e mostra solo le funzioni del progetto scelto.
4. **Esecuzione.** Scegliendo una funzione (o con il doppio clic), `Cmb_03_TXT_AfterUpdate` guarda il progetto scelto e chiama la sua funzione di esecuzione, passando il codice funzione: `ESEGUI_MENU_<CodMenu>_TMP_pFunct("", "", "", CodFunzioni_s)`.
5. **Smistamento.** Ogni `ESEGUI_MENU_<CodMenu>_TMP_pFunct`, nel modulo `MSys_TB06_Mdl02_VOCI_TMP`, ha un `Select Case` sul codice funzione. Ogni `Case` contiene l'azione da eseguire: aprire una maschera, lanciare una query, chiamare una procedura. Oggi molti `Case` mostrano solo un messaggio ("MENU CASA NON ATTIVO").

### 6.4 Query

| Query | Tipo | Uso |
| --- | --- | --- |
| `MSys_TB06_Qry01_01_VOCI` | selezione | progetti in ordine di `IdOrd`: combo e sottomaschera |
| `MSys_TB06_Qry01_01_VOCI_TMP` | selezione | progetto scelto |
| `MSys_TB06_Qry01_01_VOCI_TMP_Delete` | eliminazione | svuota la tabella di lavoro dei progetti |
| `MSys_TB07_Qry01_01_FUNZIONI` | selezione | funzioni, con il codice menu come chiave di collegamento |
| `MSys_TB07_Qry01_01_FUNZIONI_TMP` | selezione | funzioni del progetto scelto: combo `FUNZIONI` |
| `MSys_TB07_Qry01_01_FUNZIONI_TMP_Delete` | eliminazione | svuota la tabella di lavoro delle funzioni |
| `MSys_TB07_Qry01_02_INSERT_FUNZIONI_To_TMP` | accodamento | copia le funzioni del progetto scelto nella `_TMP`, senza doppioni |
| `MSys_TB06_Qry00_{===}_Qry`, `MSys_TB06_Qry01_00_}---`, `MSys_TB07_Qry01_}---@` | – | separatori |

### 6.5 Moduli e macro

| Oggetto | Contenuto |
| --- | --- |
| `MSys_TB06_Mdl01_VOCI` | `SALVA_MENU_pFunct`: copia progetto e funzioni nelle tabelle `_TMP` |
| `MSys_TB06_Mdl02_VOCI_TMP` | per ogni progetto: `ATTIVA_ESEGUI_MENU_<CodMenu>_TMP_pFunct` (prova) e `ESEGUI_MENU_<CodMenu>_TMP_pFunct` (smistamento delle funzioni) |
| `COLLEGA_Mdl01_MSys_TB06_MENU_VOCI` | ricollega le tabelle: `GetTabellaPaths_MENU` (elenco tabelle e percorsi), `CollegaTabelle_MENU_PFunct`, `CancellaTabelleCollegate_MENU_PFunct`, `CancellaTutteTabelleCollegate_MENU_PFunct` |
| `CLASSE_GESTIONE_OGGETTI_DATABASE` | classe generica di gestione degli oggetti (8.244 righe) |
| macro `COLLEGA_Mcr_MSys_TB06_MENU_VOCI` | lancia `CollegaTabelle_MENU_PFunct()` |
| macro `CANCELLA_Mcr_MSys_TB06_MENU_VOCI` | lancia `CancellaTutteTabelleCollegate_MENU_PFunct()` |

## 7. Gli altri oggetti nei database back-end

I tre back-end contengono anche copie delle maschere e delle query del front-end, oltre a oggetti modello (`MODELLO_Frm01_*`, `MODELLI_DEFINIZIONE_Mdl*`, `Dbase_*`, `Report1`, `Macro1`, `Modulo1`). I back-end devono contenere solo i dati: le maschere e le query duplicate e gli oggetti modello vanno tolti (capitolo 11, Fase 4).

## 8. Come si usa

### 8.1 Creare un nuovo progetto

1. In `MSys_TB06_MENU_VOCI` inserire una riga:
   - `CodMenu_s` = codice progetto + `MENU`;
   - `COD_PROGETTO_s` = codice progetto;
   - `Menu_s` = descrizione;
   - `NoteMenu_s` = note esplicative;
   - `IdOrd` = posizione nel menu.
2. Inserire le sue funzioni in `MSys_TB07_MENU_FUNZIONI` (8.2): senza funzioni il progetto non si può aprire dal menu.
3. In `MSys_TB06_Mdl02_VOCI_TMP` aggiungere `ESEGUI_MENU_<CodMenu>_TMP_pFunct`, copiando uno dei blocchi esistenti.
4. In `Cmb_03_TXT_AfterUpdate` della maschera principale aggiungere il `Case "<CodMenu>"` che chiama la nuova funzione.

### 8.2 Aggiungere una funzione a un progetto

1. In `MSys_TB07_MENU_FUNZIONI` inserire:
   - `CodFunzioni_s` = codice univoco;
   - `CodMenu_s` = codice menu del progetto, **identico** a quello di TB06;
   - `Funzioni_s` = descrizione, univoca in tutta la tabella;
   - `ClassFunzioni_s` e `NoteFunzioni_s`.
2. In `ESEGUI_MENU_<CodMenu>_TMP_pFunct` aggiungere `Case "<CodFunzioni_s>"` con l'azione da eseguire.

### 8.3 Collegare gli oggetti a un progetto

Non ancora possibile: richiede la struttura di TB08 proposta al capitolo 5.2 e una maschera di gestione (Fase 3).

## 9. Collegamenti con il resto di MSys

- Le 11 tabelle dei tre back-end sono censite nel catalogo `MSys_TB99_LINK.mdb`, codici `MSys_TB06`, `MSys_TB07`, `MSys_TB08`.
- `MSYS_ATTIVA_GEST_OGGETTI.mdb` e `GE_MSys_DF01_COMANDI.mdb` hanno ancora link con i **vecchi nomi** (`GEST_MENU_Tb01_VOCI`, ...) verso i vecchi file in `PROGETTI_MDB\MSYS_OGGETTI\MSYS\MDB\MSYS_MENU\MDB\`. Vanno ricollegati ai nuovi nomi, e le loro maschere e il loro codice aggiornati.

## 10. Anomalie trovate il 15/11/2026

| # | Dove | Problema | Effetto |
| --- | --- | --- | --- |
| A1 | `Cmb_03_TXT_AfterUpdate` (maschera principale) | per `LLPPGestMENU`, `LTT_MENU`, `MSysMENU`, `Pres3000_Menu`, `RUST_Menu` chiama `ESEGUI_MENU_GeCondomMENU_TMP_pFunct` invece della funzione del proprio progetto | le funzioni di questi progetti finiscono nel `Select Case` dei Condomini e non vengono eseguite |
| A2 | TB07 | 1 funzione con `CodMenu_s = "ParamCond"`, progetto che non esiste in TB06 | funzione orfana, mai visibile |
| A3 | TB07 | `CodMenu_s = "Pres3000MENU"`, mentre in TB06 è `Pres3000_Menu` | il progetto Presenze risulta senza funzioni |
| A4 | TB07 | `MSysMENU` non ha funzioni; le sue 49 sono in `_NW` con `CodMenu_s = "MSys"` | il progetto MSys non si apre dal menu |
| A5 | TB07 | una riga senza progetto (`CodFunzioni_s = "1"`) e un separatore `01_____` tra i dati di GeCondom | righe spurie nelle combo |
| A6 | TB07 | l'indice univoco su `Funzioni_s` si chiama `CodMenu_s` | due progetti non possono avere una funzione con la stessa descrizione |
| A7 | TB06 | `NoteMenu_s`, `PROGETTO_s`, `COMANDO_s` vuoti in tutti i progetti | mancano le note esplicative |
| A8 | TB08 | tabella vuota, senza campi per l'oggetto; `CodMenu_s` lungo solo 10 caratteri | il collegamento progetto-oggetti non esiste ancora |
| A9 | combo `ATTIVITA` e `STAMPE` | leggono la tabella `GEST_MENU`, che non esiste in nessun database | combo vuote o in errore |
| A10 | `GE_MSys_TB06_MENU.mdb` | il codice VBA non compila; non compilava già prima della rinumerazione | da controllare con Debug > Compila |
| A11 | `MSys_TB06_MENU_VOCI.mdb` | 4 link verso `MSYS_N01_OGGETTI.mdb`, file che non esiste (`MSys_TB06_MENU_}---@`, `Msys_DF12_GE_ObjProgetti`, `Msys_DF12_TIPO_OGGETTI`, `MSys_ELENCO_ObjProgetti`) | link morti |
| A12 | back-end | copie di maschere e query del front-end, e oggetti modello | confusione su quale copia sia quella buona |

## 11. Piano dei prossimi passi

### 11.1 Fase 1 – Correggere il funzionamento attuale

1. A1: in `Cmb_03_TXT_AfterUpdate` far chiamare a ogni progetto la propria `ESEGUI_MENU_<CodMenu>_TMP_pFunct`.
2. A3 e A2: allineare i codici menu in TB07 (`Pres3000MENU` → `Pres3000_Menu`; decidere a quale progetto assegnare `ParamCond`).
3. A5: togliere la riga senza progetto e il separatore dai dati.
4. A10: aprire l'editor VBA del front-end, Debug > Compila, correggere la riga segnalata.
5. A9: togliere le combo `ATTIVITA` e `STAMPE` o collegarle a TB07 filtrando su `ClassFunzioni_s`.

### 11.2 Fase 2 – Completare i progetti

1. A7: compilare `PROGETTO_s` e `NoteMenu_s` per ogni progetto.
2. A4: portare le funzioni di MSys da `_NW` alla tabella principale con `CodMenu_s = "MSysMENU"`, oppure sostituire la tabella principale con `_NW`, il cui campo `TIPO_FUNZIONE_s` è utile per classificare.
3. A6: rinominare l'indice e valutare se `Funzioni_s` deve essere univoco per progetto (`CodMenu_s` + `Funzioni_s`) invece che in tutta la tabella.

### 11.3 Fase 3 – Collegare gli oggetti ai progetti (TB08)

1. Allargare `CodMenu_s` a 25 caratteri e aggiungere i campi proposti al capitolo 5.2.
2. Creare nel front-end la maschera `MSys_TB08_Frm01_S01_OGGETTI`, come sottomaschera di `MSys_TB06_Frm01_S01_VOCI` collegata su `CodMenu_s`, con una combo che propone gli oggetti dal catalogo TB99.
3. Aggiungere a `SALVA_MENU_pFunct` la copia degli oggetti del progetto in `MSys_TB08_MENU_OGGETTI_TMP` (da creare).
4. Usare l'elenco degli oggetti per importare o collegare nel database del progetto solo quelli previsti (in collaborazione con `MSys_TB99_LINK` e il modulo `Mdl05_INVIA_LINK_FRONTEND`).

### 11.4 Fase 4 – Pulizia

1. A12: togliere dai back-end le copie di maschere e query e gli oggetti modello; nei back-end restano solo le tabelle.
2. A11: togliere i 4 link morti da `MSys_TB06_MENU_VOCI.mdb`.
3. Ricollegare `MSYS_ATTIVA_GEST_OGGETTI.mdb` e `GE_MSys_DF01_COMANDI.mdb` ai nuovi nomi (capitolo 9).

## 12. Storico

### 12.1 Rinumerazione del 15/11/2026

Il mini-progetto usava i numeri TB01, TB02, TB03, già assegnati ad altri mini-progetti MSys. È stato rinumerato con lo standard MSys:

| Prima | Dopo |
| --- | --- |
| `MENU_TB01_VOCI.mdb` | `MSys_TB06_MENU_VOCI.mdb` |
| `MENU_TB02_FUNZIONI.mdb` | `MSys_TB07_MENU_FUNZIONI.mdb` |
| `MENU_TB03_OGGETTI.mdb` | `MSys_TB08_MENU_OGGETTI.mdb` |
| `MENU_GESTIONE.mdb` | `GE_MSys_TB06_MENU.mdb` |
| `GEST_MENU_Tb01_VOCI` | `MSys_TB06_MENU_VOCI` |
| `GEST_MENU_TB02_FUNZIONI` | `MSys_TB07_MENU_FUNZIONI` |
| `GEST_MENU_TB03_OBJECT` | `MSys_TB08_MENU_OGGETTI` |
| `GEST_MENU_Tb01Qry…`, `GEST_MENU_Tb01Frm…` | `MSys_TB06_Qry…`, `MSys_TB06_Frm…` |
| `GEST_MENU_TB02Qry…`, `GEST_MENU_Tb02Frm…` | `MSys_TB07_Qry…`, `MSys_TB07_Frm…` |
| `GEST_MENU_Tb01_Mdl01_VOCI` | `MSys_TB06_Mdl01_VOCI` |
| `GEST_MENU_Tb01_Md01_VOCI_TMP` | `MSys_TB06_Mdl02_VOCI_TMP` |

- Sono stati aggiornati tutti i riferimenti in query, maschere, sottomaschere, combo, codice VBA e macro, insieme ai percorsi dei file.
- Sono stati tolti 15 link temporanei `~TMPCLP` dal front-end, ed è stata disattivata la correzione automatica dei nomi di Access.
- Backup dei file originali: `MDB\BACKUP_CLAUDE_2026_11_15_19_36_13_PRIMA_RINOMINA_TB06\`.
