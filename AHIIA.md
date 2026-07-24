# AHIA

**Archivio e lettura dei referti medici, interamente in locale.**

AHIA prende i PDF dei tuoi referti, ne estrae i valori, li mette in serie
storica e ti lascia ragionarci sopra con un modello linguistico che gira sulla
tua macchina. Nessun dato lascia il computer: niente servizi esterni, niente
account, niente telemetria.

Nasce come banco di prova per capire cosa un LLM locale sappia realmente fare
su documenti sanitari. È scritto per referti italiani: virgola decimale,
prefissi di matrice (`S-`, `P-`, `U-`), diciture che cambiano da laboratorio a
laboratorio, elettroforesi proteica.

---

## Cosa fa

**Archivia i referti.** Carichi i PDF — nativi o scansionati — e l'app
riconosce di che documento si tratta: analisi del sangue, urine, ecografia,
radiografia, TAC, visita specialistica, ricovero. Dai referti con valori
numerici estrae gli analiti; da quelli descrittivi conserva il testo e una
sintesi con le conclusioni.

**Normalizza le diciture.** Ogni laboratorio scrive gli esami a modo suo:
`Glicemia`, `S-Glucosio`, `GLUCOSIO SIERICO` sono lo stesso esame. Un dizionario
li riconduce a un nome unico, altrimenti le serie storiche si spezzano. Le
diciture nuove si mappano a mano, con proposte suggerite dal modello che
confermi tu.

**Mostra gli andamenti.** Grafici temporali con l'intervallo di riferimento in
trasparenza, i punti colorati per stato, un confronto normalizzato che mette
sullo stesso asse esami con unità diverse, e una tabella di variazioni
esportabile.

**Legge i referti con un LLM locale.** Analisi complessiva o su un singolo
referto, e una chat che risponde sui tuoi dati. I calcoli non li fa il modello:
differenze, percentuali e conteggi arrivano già pronti, oppure il modello li
chiede a funzioni che interrogano l'archivio.

**Cerca nell'archivio.** Ricerca esatta e ricerca per significato, perché in
italiano medico cercare "fegato grasso" deve trovare "steatosi epatica".

**Prepara un secondo parere anonimizzato.** Un modello locale da 14 miliardi di
parametri non regge il confronto con uno di frontiera sul ragionamento clinico.
L'app compone un quesito da sottoporre a un modello esterno passando il minimo:
niente nome, niente laboratorio, date sostituite da intervalli relativi, età
ridotta a fascia. Il testo lo leggi, lo modifichi e lo copi tu — non parte
nulla in automatico.

**Gestisce più persone.** Ogni utente ha credenziali proprie e un archivio
fisicamente separato. Nemmeno l'amministratore accede ai dati altrui.

---

## Cosa non fa

Vale la pena essere espliciti, perché è la parte che di solito manca.

**Non è un dispositivo medico.** Non è certificato né validato clinicamente.
Non fornisce diagnosi, prognosi o indicazioni terapeutiche, e nessuna sua
risposta va intesa come parere sanitario.

**Non sostituisce il medico e non deve ritardarne il consulto.** Interpretare
un esame richiede la storia clinica, il motivo della prescrizione, le terapie
in corso e l'esame obiettivo: cose che l'app non ha.

**Non garantisce che l'estrazione sia corretta.** Un modello può spostare una
virgola decimale o attribuire a una riga l'intervallo di riferimento di
un'altra. I valori estratti vanno confrontati con il referto originale, almeno
la prima volta per ogni laboratorio nuovo.

**Non sincronizza niente.** Nessun cloud, nessun backup automatico. La cartella
dei dati è tua da copiare.

**Non è un gestionale sanitario.** Non dialoga con il Fascicolo Sanitario
Elettronico, non importa da provider FHIR, non gestisce prescrizioni o
appuntamenti.

---

## Cosa serve

L'app si appoggia a [Ollama](https://ollama.com) per eseguire i modelli in
locale, e funziona su macOS, Linux e Windows.

### Modelli

| Modello | A cosa serve | Peso |
|---|---|---|
| `qwen3:14b` | estrazione dai PDF nativi, analisi, chat | ~9 GB |
| `qwen2.5vl:7b` | lettura delle scansioni (multimodale) | ~6 GB |
| `bge-m3` | ricerca per significato, facoltativa | ~1,2 GB |
| `qwen3:32b` | analisi più accurata, se la macchina lo regge | ~19 GB |

Tutti sono selezionabili singolarmente: si può usare un modello diverso per
ogni funzione, ed è sensato farlo — l'estrazione deve solo trascrivere una
tabella, l'analisi deve ragionare.

### Hardware

Il fattore che conta è la memoria disponibile per il modello: se ci sta
interamente, la velocità è accettabile; se deve essere ripartito sulla CPU,
crolla.

| | Memoria | Cosa aspettarsi |
|---|---|---|
| **Minimo** | 16 GB di RAM, solo CPU | Funziona, ma l'estrazione di un referto richiede diversi minuti e la chat è impraticabile. Utile per provare l'app, non per usarla. |
| **Consigliato** | 12 GB di VRAM (es. RTX 3060) oppure 16 GB di memoria unificata su Apple Silicon | I modelli consigliati stanno interamente in memoria: 20-30 token al secondo, estrazione di un referto in 30-60 secondi. |
| **Comodo** | 24 GB di VRAM o 32 GB di memoria unificata | Permette il modello da 32 miliardi di parametri per l'analisi, tenendo quello più piccolo e veloce per estrazione e chat. |

**Spazio su disco**: circa 16 GB per i modelli, più pochi megabyte per
l'archivio — un database con anni di referti resta sotto i 50 MB, i PDF
originali a parte.

Su GPU NVIDIA i modelli girano in CUDA senza configurazione; su Apple Silicon
in Metal, con una resa più bassa nella lettura delle scansioni.

---

## Come funziona

L'idea di fondo è usare lo strumento giusto per ogni tipo di dato.

**I numeri stanno in un database relazionale.** Le domande che si fanno a una
serie di valori sono ordinamenti, confronti e differenze: un database le esegue
in modo esatto e completo. Una ricerca vettoriale restituirebbe i risultati più
simili senza garantire di averli trovati tutti — su una serie storica è
esattamente il difetto da evitare.

**Il testo passa dalla ricerca.** Per i referti descrittivi servono ricerca
testuale e similarità semantica, ed è lì che gli embedding hanno senso.

**L'LLM sceglie le domande, non fa i conti.** Il contesto contiene numeri già
calcolati, e le funzioni che il modello può invocare eseguono interrogazioni
predefinite, con i nomi degli esami validati contro l'archivio.

---

## Privacy e sicurezza

I dati restano sul computer, in un archivio **non cifrato**: se la macchina è
condivisa, conviene tenerlo su un filesystem cifrato.

Le password non vengono mai salvate in chiaro, e ripetuti tentativi falliti
sospendono temporaneamente l'accesso. L'app risponde solo in locale: esporla in
rete richiederebbe almeno una connessione cifrata davanti.

I PDF sono dati non fidati, e c'è un limite che nessun accorgimento tecnico
risolve del tutto: **un PDF costruito ad arte può influenzare il modello** con
istruzioni nascoste nel testo. Carica referti che vengono dal tuo laboratorio.

---

## Stato del progetto

Software sperimentale, in evoluzione, nato per un uso personale. Funziona ed è
collaudato sui casi che ho incontrato, ma incontrerà sicuramente layout di
referti che non gestisce bene: i formati dei laboratori italiani sono molti e
tutti diversi.

Segnalazioni e contributi sono benvenuti, soprattutto se accompagnati dalla
descrizione del referto che ha creato problemi — non dal referto stesso.

## Ringraziamenti

Le descrizioni degli esami sono collegate a
[labtestsonline.it](https://labtestsonline.it), il portale divulgativo di
SIBioC — Società Italiana di Biochimica Clinica e Biologia Molecolare Clinica.
L'app conserva solo i collegamenti: i contenuti restano sul loro sito.

## Licenza

[GNU Affero General Public License v3.0](LICENSE).

In breve: puoi usare, studiare, modificare e ridistribuire il programma
liberamente, a condizione che le versioni modificate restino sotto la stessa
licenza. La particolarità dell'AGPL rispetto alla GPL è che l'obbligo vale
anche per chi non distribuisce alcun file ma offre il programma come servizio
attraverso una rete: anche in quel caso gli utenti hanno diritto al sorgente.

Per un'app che tratta referti medici mi sembra la scelta giusta: chi affida i
propri dati sanitari a un programma dovrebbe poter vedere cosa quel programma
ne fa.

Il progetto usa PyMuPDF, a sua volta distribuita sotto AGPL-3.0.

---

### In English

AHIA is a fully local medical records tool: it extracts values from lab report
PDFs, tracks them over time, and lets you discuss them with a language model
running on your own machine. Nothing leaves the computer.

It is built specifically for **Italian** lab reports — decimal commas, sample
matrix prefixes, laboratory-specific naming — so its usefulness outside that
context is limited. The interface is in Italian; the usage disclaimer is
available in both Italian and English.

It is experimental software, not a medical device, and does not replace a
physician.
