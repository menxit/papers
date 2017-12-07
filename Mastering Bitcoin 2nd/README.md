# Mastering Bitcoin 2nd

## Capitolo 1
Joe invia 0.1 btc ad Alice. Inizialmente la transazione viene propagata nella rete p2p e viene detta "Unconfirmed". Dopo in media dopo 10 minuti la transazione viene minata (aggiunta in un blocco) e la transazione diventa "Confirmed".

## Capitolo 2
Il sistema bitcoin a differenza delle banche e dei sistemi di pagamento come VISA è basata sulla decentralizzazione della fiducia.

Una transazione indica che il possessore di certi bitcoin ha autorizzato il trasferimento di tali bitcoin a un nuovo possessore, il quale a questo punto può spenderli nuovamente come preferisce.

In particolare nelle transazioni ci sono gli Input e gli Output.

Ogni transazione contiene uno o più input e output. Un input è una debito nei confronti di un altro account, mentre un output è un credito che acquisisce un account. In ere gli input e gli output non hanno lo stesso valore. Gli output generalmente hanno un valore leggermente più basso, questo perché ad essi viene sottratta una fee da dover pagare al miner di un certo blocco.

Facciamo conto che A debba spedire 0.8 bitcoin a B.

A possiede i seguenti input:
0.5 BTC
0.5 BTC

Per effettuare la transazione A -- 0.8 BTC --> B:
Input:
	- A, 0.5 BTC
	- A, 0.5 BTC

Output:
	- B, 0.5 BTC
	- A, 0.15 BTC (il resto che torna indietro) | B, 0.3 BTC [0.05 BTC fee]

Avendo A solamente due input di 0.5 BTC, per effettuare un pagamento di 0.8 BTC, dovrà inviare un input completo a B e un secondo input dovrò avere due indirizzi destinatari diversi; B e A stessa (per prendere il resto).

La "difficoltà" di capire quali input utilizzare per costruire una certa transazione è delegata al client bitcoin che stiamo utilizzando.

Se il client non è un full node, può fare una curl a un API come quella di blockchain.info per ottenere gli "Unspent" input:

```
curl https://blockchain.info/unspent?active=1Cdid9KFAaatwczBwBttQcwXYCpvK8h7FK
```

Come già detto inizialmente una transazione è "Unverified". Quando viene messa in un blocco e minata si dice "Verified".

Le transazioni "Unverified" vengono messe in un pool di transazioni non verificate gestite da ciascun full node. Un miner costruisce un nuovo blocco, verifica che tutte le transazioni selezionate siano valide e infine mina un blocco (Proof-Of-Work).

Le transazione sono aggiunte in un blocco sulla base di quelle che hanno associato la fee più alta e altri criteri. Ogni miner include in ciascun blocco una transazione particolare. Questa transazione è un ricompensa per se stesso, attualmente pari a 12.5 bitcoin, ai quali va aggiunto il ricavo delle fee delle transazioni.

Appena trova una transazione che rende il blocco valido, il miner vince la ricompensa e le relative fee di ciascuna transazione.


## Capitolo 4
I bitcoin hanno fortemente a che vedere con la crittografia. La crittografia può essere utilizzata in vari modi:
- criptare un segreto
- provare la conoscenza di un segreto (digital signature)
- provare l'autenticità di un dato (digital fingerprint)

La crittografia viene principalmente utilizzata per provare che un certo account possiede certi input e quindi che possiede certi bitcoin e che quindi può spenderli (un unica volta).

Ogni account bitcoin ha una chiave pubblica (l'indirizzo stesso dell'account bitcoin) e una chiave privata. Tramite la chiave privata un account può provare di voler spendere un certo input (digital signature). Questa prova della volontà di spendere un certo input viene salvata nei blocchi minati.

In bitcoin usa elliptic curve multiplication come base crittografica (ECC).

ECC si basa su una così detta trapdoor function (funzione a botola). Cos'è una trapdoor function?

RSA ad esempio si basa sulla fattorizzazione del prodotto di due numeri primi (altro esempio di trapdoor function).

### ECC vs RSA

Per ottenere lo stesso livello di sicurezza che si ha con una chiave privata da 256 bit usando la trapdoor function di ECC in RSA, occorre utilizzare una chiave segreta di 3072 bit.

384 bit ECC == 7680 bit in RSA (cresce più che linearmente la dimensione della chiave RSA).

Quindi con ECC stesso livello di sicurezza di RSA, con una chiave privata più piccola.

https://www.youtube.com/watch?v=F3zzNa42-tQ

In bitcoin quindi c'è:
- chiave pubblica (usata per ricevere fondi)
- chiave privata: usata per firmare le transazioni in cui si spendono fondi

Ogni volta che si effettua una transazione, l'account owner presenta la sua public key e una signature che è diversa ogni volta, ma che viene sempre generata dalla stessa chiave privata.

Un wallet bitcoin contiene un insieme di coppie di chiavi, ognuna costituita da una chiave privata e una pubblica. La chiave privata è un numero, generalmente ottenuto in maniera randomica. Dalla chiave privata, ovvero un numero, eseguiamo la elliptic curve multiplication. Una trapdoor function. 


Riassumendo:

1. beta: Genero un numero random di 256 bit, ovvero un numero tra 1 e 2^256-1 (Chiave privata)
2. Lo standard ha un punto generatore detto G
3. Applico la moltiplicazione scalare e ottengo la chiave pubblica:
B = beta * G

L'operazione 3 è facilmente calcolabile, ma difficilmente invertibile. Ovvero avendo B e G (noti a un eventuale utente malevolo), risulta molto difficile ricavare beta.

Come utilizzano ECC Alice e Bob?

Alice
1. Genera alpha
2. Calcola A = alpha * G
3. Invia in chiaro A a Bob

Bob
1. Genera beta
2. Calcola B = beta * G
3. Invia in chiaro B ad Alice

Alice
4. Riceve B da Bob
5. Si calcola il prodotto scalare di P = alpha*(beta*G)
Chiaramente Alice non ha beta, ma tanto a lui serve beta*G, ovvero B e questo è un valore che effettivamente possiede. Quindi di fatto cioò che fa è:
P = alpha*B

Bob
4. Riceve A da Alice
5. Si calcola il prodotto scalare di P = beta*(alpha*G) = beta*B


P è un segreto condiviso da Alice e Bob

Un possibile utente malevolo possiede:
G
A -> non può ricavare facilmente alpha
B -> non può ricavare facilmente beta

Ma non potrà mai calcolare P, perché non possiede e non può calcolare facilmente alpha o beta.


L'hash di A e B corrispondono agli indirizzi pubblici di Alice e Bob.


## Capitolo 5
Esistono due tipi di wallet:

Non deterministici: ogni wallet ha una chiave privata generata in maniera randomica
Deterministici: c'è un seed e da questo seed si generano le chiavi di tutti i wallet in maniera deterministica

Questa seconda scelta è migliore, perché mentre nel primo tipo di wallet il backup è più complesso (più chiavi di cui effettuare il backup), nella seconda tipologia basta effettuare il backup del seed.

Dei wallet deterministici ne esistono varie tipologie, una delle più famose è la cosidetta HD (Hierarchical Deterministic) Wallet (BIP-32/BIP-44).

Negli HD Wallet c'è un seed e poi un insieme di key derivate da una struttura ad albero. In questa struttura ad albero una parent key può usare i wallet figli. Questo garantisce una serie di vantaggi, in particolar modo per quanto riguarda la gestione e la privacy.

Perché la privacy migliora? Perché in questo modo per ogni transazione è possibile generare una nuova chiave pubblica

## Capitolo 6

set UTXO: tutti gli output ancora non spesi

Ogni transazione rappresenta un cambiamento del set UTXO

Dire che un wallet ha ricevuto dei bitcoin non ha senso. Ha più senso dire che un wallet ha rilevato nella block chain un output non speso che è utilizzabile con una delle chiavi private gestite dal wallet.

In questo senso il totale di un wallet non è altro che la somma di tutte le UTXO utilizzabili mediante le chiavi private del wallet.

Una caratteristica essenziale degli output è che sono indivisibili. Ovvero se un wallet ha due output spendibili:
1.2 BTC
0.8 BTC
Allora questi due output non possono essere divisi.

Se A vuole pagare 0.2 BTC a B, dovrà spedire 0.2 BTC a B e 1 BTC a se stesso. Questo meccanismo è nascosto dal wallet, non dal protocollo.

L'unica eccezione alla chain di input e output è rappresentata dalla coinbase, un output particolare, che sta in cima a ogni blocco minato e che rappresenta il reward del miner. 

Gli output delle transazioni quando devono essere trasmesse sul network vengono serializzate. Il formato di serializzazione è il seguente:

1. Amount in satoshi (10^-8) 8 bytes (little-endian)
![alt text](https://cdn-images-1.medium.com/max/1500/1*XzLlyZSuMyVCEOGaUh2hFg.png "Little-Endian")

2. Locking-Script size 1-9 bytes (VarInt): la dimensione del Locking-Script in bytes

3. Locking-Script (dimensione variabile): uno script che viene eseguito per verificare che un certo output sia spendibile.


Gli input identificato quale UTXO si sta consumando o forniscono una prova della proprietà di quell'output fornendo un unlocking script.


Ora vediamo come è strutturata un input nel dettaglio:

- (txid, vout) questo è un pointer all'UTXO di riferimento, quindi all'output che si desidera spendere. In particolare ci sono due campi: txid è l'id della transazione che contiene l'UTXO speso, mentre il vout è un indice che indica a quale output ci si riferisce. Se vout = 0 significa che ci si riferisce al primo
- scriptSig: questo è l'unlocking script, che spesso rappresenta una firma digitale che prova la proprietà dell'output. In realtà può avere anche un contenuto diverso.
- numero di sequenza (visto più in la)


Guardando all'input è possibile notare che c'è solo un puntatore a un UTXO, ma non la quantità di satoshi effettivamente legata ad essa. Questo significa che ogni nodo che riceverà questa transazione, dovrà andarsi a cercare la transazione di riferimento, per poter validare la transazione. Validare la transazione significa verificare che l'output sia minore dell'input e che il locking script sia stato risolto correttamente.

Le transazioni includono delle fee. Le fee esistono per due motivi:
- ricompensare i miners di rendere sicuro il network
- evitare attacchi in cui si innona il network di transazioni

Generalmente le fee sono calcolate automaticamente dal wallet e sono aggiunte automaticamente. Se invece stai costruendo una transazione "a mano", allora dovrai includere autonomamente le fees. Le transaction fee sono calcolate sulla base della grandezza in kilobytes, non in base al valore che trasportano in bitcoin.

Quanto si pagano le fees? In realtà il costo delle fee varia e dipende dal mercato (domanda (transazioni richieste), offerta (miners disponibili a minare)).

I miner a loro volta danno una maggiore priorità alle transazioni che hanno un maggior quantitativo di fee. Si capisce quindi che in realtà le fee non sono obbligatorie, in teoria una transazione può essere minata anche con 0 fee. Il problema è che sono poche le speranze per cui una transazione con 0 fee venga propagata nel network e poi effettivamente minata.

In bitcoin core il minimo di fee richiesta è 0.00001 (1 millibitcoin) per kilobyte.

Questo significa che le transazioni con fee minore a 1millibitcoin vengono generalmente scartate, o al limite aggiunte se c'è spazion nel mempool.

Detto questo, ciò che fanno i wallet è appoggiarsi a servizi di terzi, che gli dicono qual è il prezzo medio delle fee per kilobyte e a quel punto propongono una fee adeguata alla transazione da voler effettuare in quel momento.

