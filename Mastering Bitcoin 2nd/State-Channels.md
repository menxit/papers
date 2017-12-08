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


### Pay-to-Script-Hash (P2PSH)
