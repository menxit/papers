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

