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

