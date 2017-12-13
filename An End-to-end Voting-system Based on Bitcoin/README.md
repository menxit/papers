# An End-to-end Voting-system Based on Bitcoin

## Abstract
In questo progetto abbiamo riadattato il sistema di pagamento Bitcoin e proposto una piattaforma di elezione decentralizzata (dagli elettori ai candidati). Abbiamo descritto le principali scelte architetturali dietro l'implementazione, le quali consistono nelle fasi di pre-voting, voting e post-voting. Il risultato è quello di una piattaforma completamente decentralizzata, in cui è possibile votare direttamente nella block.chain, senza alcun livello intermedio. Tutti i voti possono essere verificati da chiunque, semplicemente consultando il public ledger. Inoltre abbiamo sfruttato un digital asset coin per tenere traccia in maniera diretta dei voti, mostrando il costo di elezione per n elettori.

## 1. Introduzione e motivazioni
In breve, la votazione elettronica, anche nota come e-voting permette di votare usando un sistema elettronico che permette di facilitare la votazione e il relativo conteggio dei voti. In questo lavoro abbiamo proposto un sistema end-to-end verificabile, basato sulla ben nota cryptomoneta bitcoin. In un sistema E2E:
1. Tutti gli elettori possono controllare che il loro voto sia stato correttamente conteggiato.
2. Tutti possono determinare che tutti gli scrutini siano stati conteggia correttamente.
In questo lavoro ci siamo ispirati a un a un campo storicamente correlato, quello dei sistemi di pagamento digitale, nello specifico la piattaforma Bitcoin. Gli elettori si autenticano in maniera anonima e di conseguenza ricevono un token per votare (una frazione di bitcoin). Successivamente l'elettore può "spendere" questo token per trasferirlo all'indirizzo del candidato desiderato. Questo è registrato un unica volta e per sempre nella block-chain di Bitcoin, che consiste in un ledger pubblico e distribuito, facendo si che non sia necessario avere un database centralizzato gestito una una terza parte trusted. Per come è progettata la block-chain, i costi computazionali per sovrascriverla o alterarla sono proibitivi. Alla fine, il risultato può essere verificato contando nella block-chain i token per ciascun candidato.
La soluzione proposta è distribuita e non è necessaria alcuna autorità centralizzata.

## Votare con Bitcoin
In questa sezione descriviamo come usare la piattaforma Bitcoin per implementare un E2E sistema di votazione. Come già mostrato da molti altri sistemi di elezione e altre proposte presenti in letteratura, l'e-voting si suddivide in tre fasi:

1. Pre-voting Phase
- a. I candidati vengono nominati e registrati nel processo.
- b. L'elettore registra il processo

2. Voting Phase
- a. Autenticazione dell'elettore
- b. Votazione
- c. Trasmissione e conferma del voto

3. Post-voting Phase
- a. Conteggio
- b. Risultato
- c. Controllo e verifica dei risultati

### Pre-voting Phase

#### Step 1 (a)
In questo step avviene il processo di approvazione dei candidati. Un candidato in questo contesto può essere un individuo con nome e cognome o un'entità differente. L'idea è quella di ottenere alla fine una lista di candidati, ognuno dei quali possiede una chiave asimmetrica. La chiave pubblica è associata alla sua indentità (l'indirizzo) e deve essere liberamente ottenibile da tutti gli elettori, mentre la relativa chiave privata deve essere mantenuta segreta da ciascun candidato. Inoltre facciamo in modo tale da far ricevere a ciascun elettore la lista delle public-keys verificabili dei candidati nello stesso momento in cui ottengono il token. In questo setting, supponiamo che i candidati durante la fase di registrazione mostrino un ID e comunicano la loro chiave pubblica direttamente all'organizzatore dell'elezione. Per evitare questo, una registrazione digitale può essere implementata per i candidati, similarmente a quella proposta in per gli elettori nella fase successiva.

#### Step 2 (b)
Per quanto riguarda il processo di approvazione degli elettori, a causa della sua natura (gran numero di elettori), questo step deve essere completamente digitale. La public key di un elettore verrà ricambiata con un certo quantitativo di bitcoin, che rappresenta il token per effettuare la votazione. La private key di un elettore dovrà essare mantenuta segreta e l'elettore potrà utilizzarla per effettuare la votazione. Quindi ciascun elettore avrà la propria public key e private key.

Tuttavia, una public key non può essere associata in maniera diretta all'identità di un elettore, altrimenti non può essere garantito l'anonimato. Per garantire l'anonimato, proponiamo una soluzione basata su un protocollo di autentication Anonymoys Kerberos. Notiamo che questo è solo uno dei tanti approcci a disposizione per garantire un'autenticazione anonima. Un'alternativa potrebbe essere quella di uno schema di [Blind Signature](https://en.wikipedia.org/wiki/Blind_signature). Nel nostro schema abbiamo optato per una variante del protocollo Anonymous Kerberos. Supponiamo che il processo abbia inizio con Alice (una degli elettori), che si logga nel KerberosClient KC, con una username e una password. Gli altri due partecipanti del protocollo sono un Autentication Server (AS) e un (voting-)Token Distribution Server (TDS). AS ha la responsabilità di autenticare Alice, mentre TDS deve trasferire (tramite una transazione Bitcoin) il token per effettuare la votazione ad Alice, usando la sua public key. In figura mostriamo il diagrama di sequenza dei messaggi scambiati tra le tre entità. K_Alice, K_AS e K_TDS sono rispettivamente le chaivi segrete di Alice, AS e TDS. Per far si che l'autenticazione venga separata dall'assegnazione del token, è importante che l'AS rilasci una credenziale anonima (AnonymousID) assieme a una sessione key K_session2 ad Alice, che potrà utilizzarla per richiedere al TDS un token in maniera anonima. L'AS e il TDS devono essere implementati da entità distinte. E hanno il compito di verificare che solo chi è autorizzato possa richiedere un token e che esso venga richiesto un unica volta.

### Voting Phase
Dopo aver completato la pre-registration phase, un elettore ottiene un token per votare. A questo punto per votare, l'elettore dovrà trasferire il token al bitcoin address del candidato che preferisce. 