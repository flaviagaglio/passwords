# passwords

Generatore di password casuali, interamente client-side, con indicatore di forza basato sull'entropia reale.

**Demo:** https://flaviagaglio.github.io/projects/passwords

## Cosa fa

- Genera password casuali con lunghezza regolabile (8–64 caratteri).
- Permette di scegliere quali categorie di caratteri includere: maiuscole, minuscole, numeri, simboli.
- Mostra un indicatore di forza espresso in **bit di entropia**, non un punteggio arbitrario.
- Copia la password negli appunti con un click.

Nessun dato lascia il browser: non c'è backend, non ci sono richieste di rete, non ci sono script esterni, cookie o strumenti di tracciamento. La pagina è un singolo file HTML statico.

## Sicurezza

### Generatore di numeri casuali

La generazione usa la **Web Crypto API** (`crypto.getRandomValues`), il generatore di numeri pseudo-casuali crittograficamente sicuro (CSPRNG) fornito dal browser — lo stesso tipo di sorgente di casualità usata per operazioni crittografiche (es. generazione di chiavi). Non viene usato `Math.random()`, che è pensato per usi non di sicurezza (animazioni, simulazioni) e non offre garanzie di imprevedibilità.

Per evitare il **modulo bias** — la distorsione statistica che si introduce quando si mappa un intero casuale a 32 bit su un intervallo più piccolo con l'operatore `%` — la selezione di ogni carattere avviene tramite *rejection sampling*: i valori casuali che cadrebbero fuori da un intervallo multiplo esatto della dimensione dell'alfabeto vengono scartati e ripescati, così ogni carattere dell'alfabeto scelto ha esattamente la stessa probabilità di essere selezionato.

Va detto per chiarezza: conoscere l'algoritmo usato (CSPRNG di sistema + rejection sampling) non aiuta in alcun modo a prevedere l'output. È lo stesso principio per cui la sicurezza di un lucchetto non dipende dal tenere segreto come è fatto il lucchetto, ma dalla difficoltà di aprirlo senza la chiave (principio di Kerckhoffs). La sicurezza di questo strumento sta nell'uso di una sorgente di entropia crittografica, non nell'oscurità del codice.

### Nessuna persistenza

Le password generate non vengono salvate in `localStorage`/`sessionStorage`, non vengono loggate, non vengono inviate a nessun server. Vivono solo in memoria, nel DOM della pagina, finché non si genera una nuova password o si chiude la scheda.

## La matematica: entropia della password

La forza di una password generata casualmente si misura in **bit di entropia**:

```
H = L × log2(N)
```

dove:

- **L** = lunghezza della password (numero di caratteri)
- **N** = dimensione dell'alfabeto usato, cioè quanti simboli distinti sono disponibili a ogni posizione
- **H** = entropia in bit

Questa formula deriva direttamente dalla definizione di entropia di Shannon per una sorgente uniforme discreta: se ogni carattere è scelto in modo indipendente e uniforme tra N possibilità, ogni carattere contribuisce log2(N) bit di incertezza, e L caratteri indipendenti sommano la loro entropia.

Il numero totale di password possibili con quell'alfabeto e quella lunghezza è N^L, e H = log2(N^L) = L × log2(N) è esattamente il numero di bit necessari a rappresentare in binario quello spazio di combinazioni. In pratica, H è anche il logaritmo in base 2 del numero medio di tentativi che un attaccante deve fare per indovinare la password con un attacco a forza bruta (in assenza di altre informazioni).

### Dimensione dell'alfabeto (N) per categoria

| Set attivi | N |
|---|---|
| Solo minuscole | 26 |
| Maiuscole + minuscole | 52 |
| Maiuscole + minuscole + numeri | 62 |
| Tutti e 4 (+ simboli) | 86 |

### Entropia risultante per alcune combinazioni

| Alfabeto (N) | L=8 | L=16 | L=32 | L=64 |
|---|---|---|---|---|
| Minuscole (26) | 37.6 bit | 75.2 bit | 150.4 bit | 300.8 bit |
| Maiuscole+minuscole (52) | 45.6 bit | 91.2 bit | 182.4 bit | 364.8 bit |
| +numeri (62) | 47.6 bit | 95.3 bit | 190.5 bit | 381.1 bit |
| Tutti e 4 (86) | 51.4 bit | 102.8 bit | 205.6 bit | 411.3 bit |

Come riferimento generale (NIST SP 800-63B e prassi comune di settore): sotto i ~40 bit una password è considerata debole contro attacchi offline moderni, tra 40 e 80 bit è ragionevole per la maggior parte degli usi, oltre i 100–128 bit è considerata robusta anche a lungo termine e contro attaccanti con risorse di calcolo elevate. L'indicatore "FORZA" nell'interfaccia scala linearmente questi bit fino a un riferimento di 128 bit come soglia del 100%.

### Perché non si forza la presenza di ogni categoria selezionata

Alcuni generatori garantiscono che, se selezioni "maiuscole", almeno un carattere maiuscolo compaia sempre nel risultato. Questo strumento **non** lo fa di proposito: forzare una categoria in posizioni specifiche riduce lo spazio delle combinazioni possibili (e quindi l'entropia reale) rispetto a un campionamento uniforme sull'intero alfabeto combinato. Con lunghezze ragionevoli (8+) la probabilità che una categoria selezionata non compaia affatto è comunque molto bassa, e il calcolo dell'entropia mostrato nell'interfaccia resta quindi accurato al bit.

## Sviluppo locale

Il progetto è un singolo file statico, senza build step né dipendenze:

```bash
git clone https://github.com/flaviagaglio/passwords.git
cd passwords
open index.html   # oppure servilo con un qualsiasi server statico
```

## Autrice

[Flavia Gaglio](https://flaviagaglio.github.io)
