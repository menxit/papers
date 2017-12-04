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

