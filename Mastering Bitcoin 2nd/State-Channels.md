# State Channels

Cosa significa possedere dei Bitcoin? Possedere dei bitcoin significa possedere del UTXO, ovvero degli output ancora non spesi. Una transazione Bitcoin è costituita da un insieme di input (ovvero gli UTXO che si è deciso di utilizzare) e dei nuovi output. Come è possibile dimostrare che un certo UTXO ci appartiene? Gli output hanno vari campi, tra cui uno che prende il nome di locking script. Per sbloccare un output basta concatenare al locking script un unlocking script, tale che la sua esecuzione restituisca TRUE come valore. Questi locking e unlocking script sono scritti in un linguaggio denominato Script.

## Famiglie di locking script

### Pay-to-Public-Key-Hash (P2PKH)
La maggior parte delle transazioni processate dalla rete bitcoin, spende output bloccati con una tipologia di locking script denominata P2PKH. Un output bloccato da un P2PKH script può essere sbloccato (speso) presentando:
- una public key
- una firma digitale creata con la corrispondente private key

Facciamo un esempio. Alice deve spedire 0.015 BTC a Bob. Questa transazione crea un output che ha un locking script di questo tipo:
```
OP_DUP OP_HASH160 <BOB_PUBLIC_KEY_HASH160> OP_EQUALVERIFY OP_CHECKSIG
```

<BOB_PUBLIC_KEY>: equivalente all'indirizzo del conto di Bob, senza l'encoding Base58Check. Per capire meglio, un indirizzo encodato con la Base58Check è il seguente:
```
1F1tAaz5x1HUXrCNLbtMDqcw6o5GNn4xqX
```

Tale indirizzo decodificato è pari a questo numero in esadecimale:
```
0099BC78BA577A95A11F1A344D4D2AE55F2F857B989EA5E5E2
```

Che in decimale corrisponde a:
```
B = 3769601076313889390765835912693211188293256903077174568418
```

Ricordiamo che Bitcoin si basa sulla crittografia ellittica e che questo numero corrisponde a:
```
B = <BOB_PRIVATE_KEY> * G
```

Questo locking script è sbloccabile con un unlocking script di questo tipo:
```
<BOB_FIRMA_DIGITALE> <BOB_PUBLIC_KEY>
```

Quindi il risultato sarebbe:
```
<BOB_FIRMA_DIGITALE> <BOB_PUBLIC_KEY> OP_DUP OP_HASH160 <BOB_PUBLIC_KEY_HASH160> OP_EQUALVERIFY OP_CHECKSIG
```

Vediamo nel dettaglio il funzionamento dello script:

1. <BOB_FIRMA_DIGITALE>: questo comando mette nello stack <BOB_FIRMA_DIGITALE>
2. <BOB_PUBLIC_KEY>: questo comando mette nello staco <BOB_PUBLIC_KEY>
3. OP_DUP: mette una copia di <BOB_PUBLIC_KEY> nello stack
4. OP_HASH160: prende l'ultima copia nello stack di <BOB_PUBLIC_KEY> e ci effettua un hash. In particolare questo hash funziona in questo modo: RIPEMD160(SHA256(<BOB_PUBLIC_KEY>)). RIPEMD160 e SHA256 sono due algoritmi di hashing noti.
5. <BOB_PUBLIC_KEY_HASH160>: Viene messo nello stack <BOB_PUBLIC_KEY_HASH160>
6. EQUALVERIFY: A questo punto si verifica che la public key proveniente dall'unlocking script e hashato e quella presente nel locking script coincidano. Se questo è vero tutte e due le public key hashate vengono eliminate dallo stack, sopra il quale a questo punto rimane solamente la firma digitale e poi la public key (non hashata).
7. CHECKSIG: A questo punto si verifica che la firma digitale corrisponda alla public key fornita.


### Multisignature Script
Nel P2PKH c'era un unica chiave privata che poteva sbloccare un UTXO. Nei multisignature script invece c'è uno schema del tipo M-of-N, in cui sono necessarie almeno M chiavi private su N per sbloccare un output non speso. Questo significa che in uno script multisignature del tipo 2-of-2 ci saranno due chiavi private e che entrambe saranno necessarie per sbloccare un UTXO. Mentre invece in uno script multisignature del tipo 2-of-3, saranno necessarie almeno 2 chiavi private su 3 per sbloccare un UTXO.

Lo schema generale si un locking script del tipo M-of-N multisignature è il seguente:
```
M <PUBLIC_KEY_1> <PUBLIC_KEY_2> ... <PUBLIC_KEY_N> N CHECKMULTISIG
```

Dove N è il numero totale delle PUBLIC_KEY nella lista e M è il numero minimo di PRIVATE_KEY (SIGNATURE) necessarie.

Più nello specifico, in uno schema del tipo 2-of-3 multisignature:
```
# unlocking script
<SIGNATURE_2> <SIGNATURE_3>

# locking script
2 <PUBLIC_KEY_1> <PUBLIC_KEY_2> <PUBLIC_KEY_3> 3 CHECKMULTISIG
```

CHECKMULTISIG verificherà siano state fornite almeno 2 firme digitali valide. In realtà CHECKMULTISIG soffre di un bug che deve essere risolto con un workaround. Quando CHECKMULTISIGN viene eseguito in teoria dovrebbe consumare dallo stack M+N+2 elementi. Tuttavia, a causa del bug farà il pop di un ulteriore elemento.

Vediamo il funzionamento dello script passo passo:

1. <SIGNATURE_2>: fa il push sullo stack di <SIGNATURE_2>
2. <SIGNATURE_3>: fa il push sullo stack di <SIGNATURE_3>
3. 2: fa il push sullo stack di 2
4. <PUBLIC_KEY_1>: fa il push sullo stack di <PUBLIC_KEY_1>
5. <PUBLIC_KEY_2>: fa il push sullo stack di <PUBLIC_KEY_2>
6. <PUBLIC_KEY_3>: fa il push sullo stack di <PUBLIC_KEY_3>
7. 3: fa il push sullo stack di 3
8. CHECKMULTISIG
	
	- Effettua il pop dell'item che affiora sullo stack, ovvero N (in questo caso equivale a 3)
	- Dopo effettua il pop di N elementi, dunque le 3 public key
	- Dopo effettua il pop di M, ovvero il quorum necessario, che in questo caso è pari a 2
	- A questo punto occorre fare il pop degli ultimi 2 PUBLIC_KEY forniti, ma a causa di questo bug in realtà si effettuerà il pop di 3 elementi. Per questo deve essere presente un ulteriore elemento, altrimenti si fa il pop su uno stack vuoto e si restituire un errore. Questo ulteriore valore può essere qualunque cosa, generalmente si opta per lo 0.


Facciamo un esempio di caso d'uso della multisignature per capire quanto sia comoda. Mohammed è un importer di oggetti elettronici situato a Dubai. La compagnia di Mohammed usa la multisignature. Questo significa che accetta pagamenti dai clienti solo loccati tramite multisignature. In questo modo tutti gli UTXO ricevuti dai clienti, richiederà almeno due firme digitali per essere sbloccati, una di Mohammed e una dei sue due partner. Questo tutela maggiormente tutti i partner ed evita l'appropriazione indebita.

Sebbene gli script multisignature possano essere molto utili, sono alo stesso tempo molto "pesanti" da utilizzare. Infatti nel precedente caso, Mohammed avrebbe dovuto comunicare a tutti i suoi clienti di utilizzare il suo locking script particolare:
```
<PUBLIC_KEY1> <PUBLIC_KEY2> <PUBLIC_KEY3> 3 CHECKMULTISIG
```

Inoltre i customer avrebbero dovuto usare dei wallet particolari che accettano questo genere di pagamenti, il che risulta tutto molto macchinoso. Inoltre la transazione sarebbe anche particolarmente più grande, il che comporterebbe un maggior costo del fee. Questo genere di problemi viene però risolto con i P2SH.


### Pay-to-Script-Hash (P2PSH)
I Pay-To-Script-Hash vengono introdotti nel 2012 e rappresentano una terza tipologia di locking script che va a risolvere i problemi precedentemente sollevati con il multisign locking script.

In particolare P2SH è stato sviluppato per rendere difficile usare complessi script quanto effettuare un pagamento a un address bitcoin.

Con P2SH la complessità del locking script viene rimpiazzata con la sua firma digitale, un hash criptografico.

In particolare questo significa che quando qualcuno cercherà di spendere un UTXO, esso dovrò presentare un unlocking script, che oltre all'unlocking script vero e proprio dovrà contenere lo script associato. Questo script associato è riferito al redeem script.

Quindi la multisig senza P2SH funziona così:
Locking script: 2 <P_KEY1> <P_KEY2> <P_KEY3> 3 CHECKMULTISIG
Unlocking Script: <SIG1> <SIG2>


La multisign con P2SH invece:
Redeem Script: 2 <P_KEY1> <P_KEY2> <P_KEY3> 3 CHECKMULTISIG
Locking script: HASH160 <20-byte hash of redeem script> EQUAL
Unlocking Script: Sig1 Sig2 <REDEEM SCRIPT>

In questo modo il locking script è conciso (meno costo fee), e tutta la complessità viene invece spostata nell'unlocking script. Quindi la complessità e le spese vengono spostate dal cliente al venditore.

Gli indirizzi P2PSH invece di iniziare con il numero 1, iniziano con il numero 3.


## Timelocks
Timelocks sono delle restrizioni sulle transazioni o sugli output che permettono di spendere un certo output solo dopo un certo tempo.

### nLocktime (restrizione assoluta sulla transazione)
Questa è una restrizione temporale a livello di transazione, rappresentato da un vero e proprio campo della transazione. Può assumere vari valori:

1. 0: significa che la transazione deve essere propagata immediatamente
2. <500M: significa che non deve essere spesa prima che un certo blocco venga minato.
3. >500M: viene interpretato come un UNIX timestamp.

Immagina che Alice deve inviare a Bob 1 bitcoin. Allora crea una transazione, e imposta il campo nlocktime in maniera tale che la transazione sarà valida tra 30 giorni.

Nessuno però vieta ad Alice di prendere l'output speso nella transazione per Bob e riutilizzarlo. Questo significa che la transazione precedentemente creata, scattato l'nLocktime non sarà comunque più valida. Per avere questo genere di garanzia, la restrizione non deve essere a livello di transazione, ma a livello di output.

### Check Lock Time Verify, CLVT (restrizione assoluta sull'output)
In Script c'è un operatore che prende il nome di CLTV. CLTV è un time lock per output. Aggiungendo CLTV opcode nel redeem script di un output, esso impedisce di spendere l'output prima del tempo definito.

Per esempio, se Alice deve pagare Bob, facendo in modo di bloccare l'output per 3 mesi, la transazione dovrà essere una P2SH transaction con un redeem script del tipo:
```
<now + 3 months> CHECKLOCKTIMEVERIFY DROP DUP HASH160 <BOB_PUBLICK_KEY_HASH> EQUALVERIFY CHECKSIG
```

### nSequence (restrizione relativa sulla transazione)
nSequence è un campo che può essere impostato su ciascun input di una transazione. In particolare una transazione con un input che ha una nSequence pari a 30 ha questo significato. Questa transazione sarà valida, solo quando dopo il blocco minato per l'input utilizzato, sono stati minati altri 30 blocchi.

### Check Sequence Verify CSV (restrizione relativa sull'output)
Il corrispettivo (ma relativo) di CLVT è CSV. L'opcode CHECKSEQUENCEVERIFY viene inserito nel redeem script. Esso impedisce di spendere un UTXO fino a che non sono passati un certo numero di blocchi (o secondi), relativamente al momento in cui la UTXO è stata minata.

## Payment Channels e State Channels
I Payment Channels sono un meccanismo trustless che permette di scambiare bitcoin "fuori" dalla blockchain e rappresentano una sorta di cambiale. Visto che queste transazioni sono off chain, non è necessario nè attendere che la transazione venga minata in un blocco, nè pagare le fee.

Attualmente il termine "channel" è una metafora. Gli state channel in realtà sono una costruzione virtuale, che rappresentano lo scambio di stati tra due parti fuori dalla blockchain. Non c'è alcun canale fisico a livello di trasporto dati. Il termine channel viene usato solo per identificare lo scambio di stato tra due parti.

Per capire meglio questo punto, si può prendere come paragone uno stream TCP. Ad alto livello esso non è altro che una socket che connette due punti mediante internet. Più nel dettaglio, uno stream TCP è un canale virtuale over IP. Ciascuno dei due punti che usa il protocollo tcp non fa altro che scambiarsi dei pacchetti, che successivamente il protocollo si impegna a sequenziare e ordinare per dare l'idea di avere un vero e proprio stream di byte, ma sotto non sono altro che pacchetti disordinati e sconnessi che viaggiano su IP.

Allo stesso modo un payment channel è solo una serie di transazioni disordinate (pacchetti disordinati IP). Ma se queste transazioni vengono sequenziate e connesse, allora creano dei veri e propri obblighi riscattabili, anche se non puoi fidarti di chi sta dall'altro capo del canale.

La prima cosa da chiarire è che i Payment Channel sono una parte di un concetto più ampio che è quello di State Channel. Gli state channel rappresentano un'alterazione dello stato off-chain, che successivamente viene reso sicuro "saldandolo" sulla blockchain.

### State Channel
Uno state channel è stabilito tra due parti e si basa su una transazione che locca uno stato condiviso sulla blockchain. Questa prima transazione è detta funding transaction o anche anchor transaction e questa singola transazione deve effettivamente essere minata e occorre pagare le fee.

Successivamente le due parti si scambiano delle transazioni firmate, chiamate commitment transactions, che alterano lo stato iniziale.

Queste transazioni come già detto non devono essere minate, quindi il numero di transazioni che è possibile creare è basato sul numero di transazione che il calcolatore che stiamo utilizzando riesce a creare e trasmettere all'altra parte. In pratica questo ci abilita a creare migliaia di transazione al secondo.

Ogni volta che una nuova commitment transaction avviene, la precedente viene invalidata. Quindi l'ultima transazione è l'ultima che può essere riscattata. 

Infine il canale viene chiuso con una transazione, detta settlement transaction, che deve essere minata e inserita nella blockchain. QUesta transazione può essere create in maniera cooperativa (multisig) o in maniera unilaterale.


### Payment channel (unidirezionale)
Per capire meglio gli state channel, proponiamo un esempio base di payment channel (unidirezionale), in cui inizialmente facciamo l'assunzione che nessuno voglia truffare nessuno.

Immaginiamo due partecipanti Emma e Fabian. Fabian offre un servizio di video streaming  che costa 0.01 millibit al secondo, quindi 36 millibit all'ora.

Emma invece è un utente del servizio di streaming di Fabian.

Sia Emma che Fabian usano un software speciale, in particolare Emma nel browser e Fabian sul server. Questo software include un wallet per bitcoin basilare e può creare e firmare transazioni.

Per mettere in piedi il payment channel, Emma e Fabian stabiliscono un 2-of-2 indirizzo multisignature, del quale ciascuno dei due possiede una chiave privata.

Dal punto di vista di Emma, il software mostra un qrcode con un indirizzo P2SH (quello che inizia con 3), il quale gli richiede di inviare il deposito necessario per 1 ora di video, questa rappresenta la funding transaction, quella che viene minata sulla blockchain.

A questo per punto per spendere gli output inviati all'indirizzo P2SH occorrerà il consenso sia di Emma che di Fabian.

In questo esempio quindi Emma invierà 36 millibit, il che consentirà ad Emma di consumare fino a un ora di servizio, in altre parole questa transazione fissa il quantitativo massimo che si è disposti a far passare su questo channel.

Una volta che la funding transaction viene confermata, Emma inizia a usufruire del video e creerà una commitment transaction.

A questo punto le commitment transaction vengono fatte sulla base degli output contenuti dal P2SH address.

La prima commitment transaction sarà una transazione che invierà a Emma un rimborso di 35.99 millibits e a Fabian un pagamento di 0.01 millibit.

A questo punto avviene una microtransazione (di quelle non minate) per ogni secondo di visione del video in streaming.

Dopo 5 secondi la situazione è questa. Sono state fatte 5 microtransazioni, ma l'unica valida è l'ultima e quest'ultima transazione dice:
invia a Emma un rimborso di 35.95 millibits e a Fabian un pagamento di 0.05 millibits.

A questo punto Emma clicca sul pulsante "Stop" per fermare la trasmissione del video in streaming. 

A questo punto, l'ultima commitment transaction viene effettivamente trasmessa broadcast alla rete, per far si che venga minata. Questa è detta la settlment transaction e fa si che Emma ottenga il suo rimborso e che Fabian ottenga il suo pagamento.

Ora però tutto questo discorso regge solamente se Emma e Fabian sono onesti e se il sistema non fallisce mai. Come facciamo a rendere il payment channel unidirezionale trustless?

Per prima cosa occorre capire come si possa attuare una truffa o cosa comporterebbe un fallimento:

1. Emma effettua il pagamento all'indirizzo P2SH, poi per qualche motivo la connessione con Fabian si interrompe prima che venga effettuata la settlment transaction e a questo punto l'output di Emma rimane incastrato in un P2SH address e nè Emma ne Fabian potranno utilizzarlo.

2. Altro problema è questo. Emma può prendere qualunque commitement transaction firmata da Fabian e trasmetterla alla blockchain. Perché pagare per 600 secondi, quando Emma può inviare alla blockchain la prima commitment transaction, quella in cui pagava solo un secondo e riceveva il rimborso di 35.99 millibits?

Entrambi i problemi possono essere risolti con l'uso dei timelocks assoluti a livello di transazione (quindi gli nLocktime).

1. Il primo problema può essere sintetizzato in questo modo. Emma non può inviare del denaro a un P2PSH address, senza avere la garanzia di ricevere un rimborso. Per risolvere questo problema Emma costruisce due transazioni. La transazione di fund (quella da inviare al P2PSH) e una transazione di refund. A questo punto firma la transazione di funding, ma non la trasmette a nessuno. Dopo Emma trasmette solo la transazione di refund a Fabian e ottiene dunque la sua firma. A questo punto la refund transaction funge da prima commitment transaction e avrà un timelock nLocktime pari a 30 giorni. Che significa questo? Significa che quella transazione sarà valida solo tra 30 giorni. Tutte le successive commitment transaction dovranno avere un nLocktime minore, in modo tale che possano essere riscattate prima della refund transaction. Nel momento in cui Emma ha una commitment transaction di refund, valida tra 30 giorni, inviare tranquillamente la transazione di funding, sapendo che se qualche cosa dovesse andare male, potrà utilizzare una transazione di refund totale tra 30 giorni.

2. Il secondo problema viene risolto sempre con l'uso degli nLocktime. Ovvero, se ho 3 commitment transaction, come faccio a fare in modo che non venga minata la prima commitment transaction invece della terza? Ogni commitment transaction viene loccata nel futuro. Ad esempio:
- TX1: loccata al blocco 4320
- TX2: loccata al blocco 4319
..
- TXN: loccata al blocco 4320-N

Questo fa si che diventino valide prima le ultime transazioni e dopo le successive. 

Chiaramente se tutto va bene, la settlment transaction sarà firmata in maniera cooperativa e avrà un nTimelock pari a 0.

Quindi per risolvere questo secondo problema l'idea è quella di far si che le ultime commitment transaction diventino valide prima delle precedenti.

Chiaramente queste soluzioni hanno comunque dei problemi:
1. Il refund deve avvenire non è immediato, ma occorre aspettare un tempo prefissato.
2. Decrementare di un blocco il tempo di attesa, significa che c'è un numero massimo di microtransazione che può essere effettuato, ovvero quello tale che si arriva al blocco attuale.


### Asymmetric Revocable Commitments
Una soluzione alternativa al secondo problema precedentemente, è proposta di seguito e prende il nome di Asymmetric Revocable Commitments. Questa idea non si basa sull'uso dei locktime, ma su un modo che fa si che la precedente commitments transaction venga invalidata. Il problema è che una transazione in bitcoin una volta emessa non può essere invalidata. L'unico modo per invalidare una transazione è mediante il double-spending che a pensarci bene è l'approccio utilizzato nei locktime. Infatti li, fai si che la transazione precedente non possa essere valida, perché viene validata prima la successiva (che spende lo stesso output).

Sebbene una transazione non possa essere invalidata, può essere costruita in modo tale da rendere non desiderabile utilizzarla. In particolare quel che si fa è consegnare a ciascuna parte una revocation key, che può essere utilizzata per punire l'altra parte se tenta di truffare.

Questo meccanismo è stato proposto per la prima volta come parte del Lightning Network.

Introduciamo quindi un nuovo esempio. Hitesh e Irene sono due agenti di cambio, uno in India e l'altro negli USA. I clienti in India di Hitesh spesso inviano pagamenti ai clienti negli USA di irene e viceversa. Attualmente queste transazioni avvengono sulla blockchain, ma questo comporta pagare le fee e avere lunghi tempi di attesa.

Si vuole invece creare un payment channel tra Hitesh e Irene, in modo tale da far viaggiare all'interno di questo canale le transazioni dei clienti.

Hitesh e Irene mettono in piedi un payment channel con una funding transaction nella quale ciascuna parte mette 5 bitcoin. Quindi il bilancio iniziale delle due parti è di 5 bitcoin ciascuno. La funding transaction locca il canale in un 2-of-2 multisig P2SH address.

Questa transazione iniziale è realizzata in questo modo: ci saranno più input di irene che vanno a formare i suoi 5 bitcoin, poi ci saranno più input di hitesh che formeranno i suoi 5 bitcoin e poi ci sarà un unico output multisig da 10 bitcoin (più eventuale output per i resti di irene e hitesh).

A questo punto invece di creare un unica commitment transaction in cui ciascuna parte firma, hitesh e irene creano due differenti commitment transaction, che sono asimmetrici.

TX HITESH	
1. OUTPUT -> 5BTC IRENE (IMMEDIATAMENTE)
2. OUTPUT -> 5BTC HITESH (DOPO 1000 BLOCCHI)

TX IRENE
1. OUTPUT -> 5BTC HITESH (IMMEDIATAMENTE)
2. OUTPUT -> 5BTC IRENE (DOPO 1000 BLOCCHI)

In qualunque momento sia irene che hitesh possono firmare la transazione e propagarla nella rente. Chiaramente se usano questa transazione, ciascuno pagherà la controparte immediatamente, mentre otterrà i propri bitcoin dopo un certo quantitativo di tempo.

In questo modo se una delle due parti decide di utilizzare una commitment transaction, si troveranno in svantaggio, in quanto dovranno attendere questo lasso di tempo.

In questo modo mettiamo in svantaggio il truffatore, ma ad ogni modo gli basterà attendere il tempo necessario per vedersi sbloccato il suo output.

Per questo motivo entra in gioco un nuovo elemento, ovvero la revocation key che permette a una delle due parti di punire l'altro, prendendogli l'intero bilancio che ha messo sul canale.

Come detto ciascuna commitment transaction ha un output ritardato, che in particolare gli permette di riottenere l'output solo dopo che siano stati minati 1000 blocchi (circa 167 giorni).

A questo punto, quando hitesh crea la commitment transaction per irene, la firma e rende il secondo output revocabile dopo 1000 blocchi o da chiunque possieda una revocation key. A questo punto hitesh crea la transazione e si tiene la revocation key segreta.

Rivelerà questa chiave segreta a irene solo quando lui sarà pronto a muoversi in un nuovo canale e vuole revocare questo commitment.


### Hash Time Lock Contracts (HTCL)
Gli HTCL sono uno strumento utilizzato sia nei pagamenti bidirezionali che nel ligthining network. L'idea è quella di creare un fondo che è possibile utilizzare con un segreto e che è revocato dopo un certo tempo.

L'idea è questa. Chi dovrà poter sbloccare l'output, crea una chiave segreta R, ne effettua l'hash e lo mette nel locking script e firma. La controparte firma. A questo punto se si conosce la chiave R si può sbloccare l'output, oppure se è passato un certo tempo, l'utente originale può sbloccare l'output.


## Lightning Network
