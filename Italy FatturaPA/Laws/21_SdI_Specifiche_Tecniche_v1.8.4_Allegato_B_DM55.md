# Specifiche delle regole tecniche di cui all'Allegato B del DM 55 del 3 aprile 2013 per la trasmissione delle fatture elettroniche tramite Sistema di Interscambio — versione 1.8.4 (31/03/2026)

**Publication:** fatturapa.gov.it, "Documentazione Sistema d'Interscambio": **valid from 2026-05-15** (v1.8.3 valid until 2026-05-14)  
**Source:** https://www.fatturapa.gov.it/export/documenti/Specifiche-tecniche-relative-al-Sistema-di-Interscambio-versione-1.8.4.pdf  
**Page:** https://www.fatturapa.gov.it/it/norme-e-regole/DocumentazioneSDI/  
**Retrieved:** 2026-10-29  
**Text:** Text layer of the published PDF (56 pages) via `pdftotext -layout`, one fenced block per page; the running header (title, "Pag. N di 56", "[versione 1.8.4] 31/03/2026") and bare page numbers removed.  
**Note:** The **SdI technical rules of DM 55/2013 Allegato B** (B2G lineage, also covering invoices under D.Lgs. 127/2015 art. 1 c. 2): channels (PEC, SdICoop, SPCoop, SdIFtp, web), receipts and notifications incl. esito committente, controls. A **third** document beside the FatturaPA format spec V1.4 and the AdE Allegato A v1.9.1 (file 17) — different publisher, different numbering.

<!-- PDF page 1 -->

```text
 SPECIFICHE DELLE REGOLE TECNICHE DI CUI
ALL’ALLEGATO B DEL DM 55 DEL 3 APRILE 2013
   PER LA TRASMISSIONE DELLE FATTURE
     ELETTRONICHE TRAMITE SISTEMA DI
              INTERSCAMBIO

                                              Versione 1.8.4
```

<!-- PDF page 2 -->

```text
                                                         INDICE

STATO DEL DOCUMENTO                                                                                          5

1. INTRODUZIONE                                                                                              6

   1.1           DEFINIZIONI                                                                                 6

2. MODALITÀ DI EMISSIONE DELLE FATTURE ELETTRONICHE                                                          8

   2.1           FORMATI DI FIRMA ELETTRONICA E RIFERIMENTO TEMPORALE                                        8

   2.2           NOMENCLATURA DEI FILE DA TRASMETTERE                                                        9

3. MODALITÀ DI TRASMISSIONE DELLE FATTURE, RICEVUTE E NOTIFICHE                                              11

   3.1           TRASMISSIONE DEL FILE AL SDI                                                                11
   3.1.1         Posta elettronica certificata (servizio PEC)                                                11
   3.1.2         Cooperazione applicativa su rete Internet (servizio SdICoop - Trasmissione)
                                                                                         13
   3.1.3         Porte di dominio in ambito spcoop (servizio SPCoop - Trasmissione)                          14
   3.1.4         Sistema di trasmissione dati tra terminali remoti basato su protocollo FTP
                 (servizioSdIFtp)                                                          15
   3.1.5         Invio tramite web                                                                           16

   3.2           TRASMISSIONE DEL FILE AL SOGGETTO RICEVENTE                                                 16
   3.2.1         Posta elettronica certificata (servizio PEC)                                                17
   3.2.2         Cooperazione applicativa su rete Internet (servizio SdICoop - Ricezione) 17
   3.2.3         Porte di dominio in ambito SPCoop (servizio SPCoop - Ricezione)                             18
   3.2.4         Sistema di trasmissione dati tra terminali remoti basato su protocollo FTP
                 (servizioSdIFtp)                                                          19

   3.3           MODALITÀ DI INOLTRO DELLE COMUNICAZIONI DA PARTE DEL SDI                                    20

   3.4           MODALITÀ DI INOLTRO DELLE COMUNICAZIONI DAL SOGGETTO RICEVENTE AL SDI
                                                                                   21

4. MODALITÀ DI INTERAZIONE CON IL SISTEMA DI INTERSCAMBIO                                                    22
```

<!-- PDF page 3 -->

```text
   4.1           PROCEDURA PER L’ACCREDITAMENTO DEL SOGGETTO RICEVENTE AI FINI
                 DELL’INTERAZIONE CON IL SDI                                                                 22

   4.2           PROCEDURA PER L’ACCREDITAMENTO DEL SOGGETTO TRASMITTENTE AI FINI
                 DELL’INTERAZIONE CON IL SDI                                      23

   4.3           PROCEDURA DI INVIO FATTURA AL SDI                                                           23

   4.4           PROCEDURA DI INOLTRO DELLA FATTURA AL SOGGETTO RICEVENTE                                    24

   4.5           PROCEDURA DI GESTIONE DELLE RICEVUTE E DELLE NOTIFICHE                                      24

5. CONTROLLI EFFETTUATI DAL SDI                                                                              27

   5.1           TIPOLOGIE E MODALITÀ DI VERIFICA                                                            27
   5.1.1         Verifiche effettuate sui file fattura                                                       27
   5.1.2         Tempi di elaborazione da parte del SDI                                                      34

ALLEGATO B-1 STRUTTURA DEI MESSAGGI DI COMUNICAZIONE
DEL SISTEMA DI INTERSCAMBIO GUIDA ALL’UTILIZZO                                                               36

PREMESSA                                                                                                     37

1. DESCRIZIONE E REGOLE DI COMPILAZIONE                                                                      38

   1.1           NOMENCLATURA DEI FILE PER LA TRASMISSIONE DI RICEVUTE/ NOTIFICHE                            38

   1.2           RICEVUTA DI CONSEGNA DEL FILE AL DESTINATARIO                                               39

   1.3           NOTIFICA DI SCARTO                                                                          42

   1.4           NOTIFICA DI MANCATA CONSEGNA                                                                43

   1.5           NOTIFICA DI ESITO COMMITTENTE                                                               45

   1.6           NOTIFICA DI SCARTO ESITO COMMITTENTE                                                        47

   1.7           NOTIFICA DI ESITO (CEDENTE)                                                                 49

   1.8           NOTIFICA DI DECORRENZA TERMINI                                                              51

   1.9           NOTIFICA METADATI DEL FILE FATTURA AL DESTINATARIO                                          53

   1.10          ATTESTAZIONE DI AVVENUTA TRASMISSIONE DELLA FATTURA CON IMPOSSIBILITÀ DI
                 RECAPITO                                                           55
```

<!-- PDF page 4 -->

```text
STATO DEL DOCUMENTO
revisione data                                 note
1.3       01 maggio 2016                       Vengono modificati controlli esistenti ed
                                               introdotti nuovi controlli sui file FatturaPA
1.4              10 novembre                   Il Sistema viene adattato alla fatturazione tra
                 2016                          privati (B2B)

1.5              15 marzo 2017                 Viene modificato il controllo con codice 00399

1.6              23 aprile 2018                Aggiornata modalità di integrazione su porta
                                               di dominio

1.7              30 luglio 2019                Vengono introdotti nuovi controlli legati ai
                                               gruppi IVA

1.8              04 maggio 2020                Vengono introdotti nuovi controlli

1.8.1            01 ottobre 2020               Modificata data decorrenza controllo 00445

1.8.2            01 ottobre 2022               Aggiunti controlli con codici 00475 e 00476
                                               Introduzione del messaggio di cortesia per
                                               PEC inviata senza allegato
1.8.3            12 dicembre                   Aggiunto controllo con codice 00477
                 2023

1.8.4            31 marzo 2026                 Aggiornamento procedure di accreditamento
                                               dei canali WS e SFTP
```

<!-- PDF page 5 -->

```text
1.          INTRODUZIONE
           Il presente documento riporta le specifiche tecniche di cui all’allegato B al DM
           55 del 3 aprile 2013, relative alle soluzioni informatiche da utilizzare per
           l’emissione e la trasmissione delle fatture di cui all’articolo 1, comma 213,
           lettera b), della legge 24 dicembre 2007, n. 244, e delle fatture di cui
           all’articolo 1, comma 2, del decreto legislativo 127/2015, nonché quelle
           idonee a garantire l’attestazione della data, l’autenticità dell’origine e
           l’integrità del contenuto della fattura elettronica.

     1.1    DEFINIZIONI

           Ai fini del presente documento si intende:
           - per AgID (già DigitPA, già CNIPA), l’Agenzia per l’Italia Digitale;
           - per cedente/prestatore, il soggetto fornitore dei beni/servizi;
           - per Certificatore, il soggetto pubblico o privato che emette certificati
                qualificati di firma conformi alla Direttiva europea 1999/93/CE e nazionale
                in materia;
           - per cessionario/committente, il soggetto cliente destinatario dei
                beni/servizi;
           - per fattura elettronica, il documento informatico, non contenente codice
                eseguibile né macroistruzioni, in formato strutturato, trasmesso per via
                telematica al Sistema di Interscambio e da questo recapitato al Soggetto
                ricevente; può riferirsi ad una fattura singola ovvero ad un lotto di fatture;
           - per Firma elettronica qualificata, la firma elettronica che garantisce la
                connessione univoca al firmatario e la sua univoca identificazione, basata
                su un certificato qualificato e realizzata mediante un dispositivo sicuro
                per la creazione della firma;
           - per FTP (File Transfer Protocol), il protocollo di trasferimento dati tra
                sistemi remoti;
           - per HTTPS (HyperText Transfer Protocol Secured), il protocollo di
                trasmissione dati su web con ulteriore livello di crittografia ed
                autenticazione dei dati trasmessi (SSL - Secure Sockets Layer);
           - per Intermediario, il soggetto di cui si avvale il Cedente/prestatore per
                l’emissione e/o la trasmissione delle fatture elettroniche al Sistema di
                Interscambio, o di cui si avvale l’Amministrazione per la ricezione delle
                stesse dal Sistema di Interscambio;
           - per Messaggio SOAP, messaggio XML, strutturato in un header e in un
                body, utilizzato nel colloquio tra web services;
           - per Riferimento temporale, l’informazione contenente la data e l’ora che
                viene associata ad uno o più documenti informatici; insieme alla firma
                elettronica qualificata, caratterizza la fattura elettronica;
```

<!-- PDF page 6 -->

```text
-    per SdI, il Sistema di Interscambio, vale a dire la struttura istituita dal
     Ministero dell’Economia e delle Finanze attraverso la quale avviene la
     trasmissione delle fatture elettroniche verso l’Amministrazione (art.1,
     comma 211, legge 24 dicembre 2007 n. 244) o verso privati (art. 1,
     comma 2, decreto legislativo 127/2015);
-    per Soggetto emittente, il Cedente/prestatore o l’Intermediario delegato
     all’emissione della Fattura elettronica;
-    per Soggetto ricevente, il cessionario/committente o l’intermediario
     delegato alla ricezione della Fattura elettronica dal SdI;
-    per Soggetto trasmittente, il cedente/prestatore o l’intermediario delegato
     alla trasmissione della Fattura elettronica;
-    per SPC, il Sistema Pubblico di Connettività di cui agli articoli 73 e
     seguenti del Codice dell’Amministrazione Digitale;
-    per SPCoop, la parte del SPC finalizzata all’interazione tra i sistemi
     informatici delle Pubbliche Amministrazioni e tra queste e i cittadini;
-    per WSDL (Web Service Definition Language), il linguaggio basato su
     XML per definire un web service e descriverne le modalità di accesso;
-    per XML (Extensible Markup Language), l’insieme di regole per
     strutturare in formato testo i dati oggetto di elaborazione;
-    per Sistema di accreditamento: il sistema che permette agli utenti di
     effettuare l’accreditamento al SDI tramite una procedura dedicata;
-    per Accordo di servizio: il documento, generato sul Sistema di
     Accreditamento, che contiene le regole relative al flusso telematico fra il
     soggetto trasmittente e/o ricevente ed il SdI.
```

<!-- PDF page 7 -->

```text
2.          MODALITÀ DI EMISSIONE DELLE FATTURE ELETTRONICHE
           Il presente capitolo descrive le specifiche tecniche di emissione della fattura
           elettronica, con riferimento al formato di rappresentazione e alle
           caratteristiche del documento da trasmettere.

           I dati della fattura elettronica da trasmettere attraverso il SdI devono essere
           rappresentati in formato XML (eXtensible Markup Language), secondo lo
           schema e le regole riportate nelle Specifiche tecniche del formato della
           fattura del Sistema di Interscambio pubblicate sul sito del Sistema di
           Interscambio www.fatturapa.gov.it nella sezione Documentazione Fattura
           elettronica.

     2.1    FORMATI DI FIRMA ELETTRONICA E RIFERIMENTO TEMPORALE
           Il SdI accetta come fattura elettronica un documento informatico che:
           - se destinato ad una pubblica amministrazione (art.1, comma 211, legge
                24 dicembre 2007 n. 244), sia provvisto di un riferimento temporale e
                firmato elettronicamente tramite un certificato di firma elettronica
                qualificata, non contenente macroistruzioni o codici eseguibili tali da
                attivare funzionalità che possano modificare gli atti, i fatti o i dati nello
                stesso rappresentati;
           - se destinato ad un soggetto diverso da pubblica amministrazione (art. 1,
                comma 2, decreto legislativo 127/2015), sia firmato o secondo la modalità
                precedente, oppure in formato XAdES con certificato di firma CA Agenzia
                delle Entrate.
               Nel primo caso, il certificato di firma elettronica qualificata deve essere
               rilasciato da un certificatore accreditato, presente nell’elenco pubblico dei
               certificatori gestito dall’Agenzia per l’Italia Digitale così come disciplinato
               dall’art. 29, comma 1, del DLGS 7 marzo 2005 n. 82 e successive
               modifiche. I formati ammessi per firmare elettronicamente la fattura sono i
               seguenti:
            -    CAdES Baseline B (CMS Advanced Electronic Signatures), di seguito
                 CAdES, come previsto dalla normativa europea vigente in materia di
                 firma elettronica avanzata;
           XAdES Baseline B (XML Advanced Electronic Signatures), di seguito
           XAdES, come previsto dalla normativa europea vigente in materia di firma
           elettronica avanzata. Nell’ambito del formato di firma XML l’unica modalità
           accettata è quella “enveloped”.
           Come riferimento temporale il SdI intende la valorizzazione dell’attributo
           “signing time” che deve essere presente nella firma elettronica apposta sul
           documento.
```

<!-- PDF page 8 -->

```text
2.2    NOMENCLATURA DEI FILE DA TRASMETTERE
      Le fatture elettroniche devono essere trasmesse al SdI sotto forma di file
      secondo una delle modalità di seguito descritte:

       a) un file in formato xml contenente una singola fattura;

       b) un file in formato xml contenente un singolo lotto di fatture (dove il “lotto” è
          inteso nell’accezione dell’art. 21, comma 3, Decreto del Presidente della
          Repubblica del 26 ottobre 1972 n. 633); il lotto è rappresentabile all’interno della
          struttura xml, grazie alla divisione della struttura tra header, comune all’intero
          lotto e body, specifico per ogni singola fattura;

       c) un file in formato compresso contenente uno o più file di tipo a) e/o uno o più
          file di tipo b); il formato di compressione accettato è il formato ZIP.

      Nei casi a) e b) il nome del file deve rispettare la seguente nomenclatura:

                             codice           identificativo univoco del                    progressivo
                             paese              soggetto trasmittente                      univoco del file
      dove:

       -    il codice paese va espresso secondo lo standard ISO 3166-1 alpha-2
            code;
       - l’identificativo univoco del soggetto trasmittente, sia esso persona fisica o
            persona giuridica, è rappresentato dal suo identificativo fiscale (codice
            fiscale nel caso di soggetto trasmittente residente in Italia, identificativo
            proprio del paese di appartenenza nel caso di soggetto trasmittente
            residente all’estero); la lunghezza di questo identificativo è di:
                o 11 caratteri (minimo) e 16 caratteri (massimo) nel caso di codice
                    paese IT;
                o 2 caratteri (minimo) e 28 caratteri (massimo) altrimenti;
       - il progressivo univoco del file è rappresentato da una stringa
            alfanumerica di lunghezza massima di 5 caratteri e con valori ammessi
            [a-z], [A-Z], [0-9].
        Il progressivo univoco del file ha il solo scopo di differenziare il nome dei file
        trasmessi al Sistema di Interscambio da parte del medesimo soggetto; non
        deve necessariamente seguire una stretta progressività e può presentare
        anche stili di numerazione differenti.
        Il file deve essere firmato elettronicamente (come indicato al precedente
        paragrafo 2.1); in base al formato di firma elettronica adottato, l’estensione
        del file assume il valore “.xml” (per la firma XAdES) oppure “.xml.p7m” (per
        la firma CAdES).
```

<!-- PDF page 9 -->

```text
Il separatore tra il secondo ed il terzo elemento del nome file è il carattere
underscore (“_”), codice ASCII 95.
Es.: ITAAABBB99T99X999W_00001.xml
      IT99999999999_00002.xml.p7m
Nel caso c) il nome del file deve rispettare la stessa nomenclatura e
l’estensione del file può essere solo .zip.
In questo caso non è il file compresso (.zip) che deve essere firmato
digitalmente, ma ogni singolo file in esso contenuto.
Es.: ITAAABBB99T99X999W_00001. zip
      che al suo interno contiene, a titolo di esempio
      ITAAABBB99T99X999W_00002.xml
               ITAAABBB99T99X999W_00003.xml
               ITAAABBB99T99X999W_00004.xml.p7m
La nomenclatura dei file, così come descritta, viene mantenuta nella fase di
inoltro al soggetto ricevente. Qualora al SdI sia inviato un file del tipo
indicato nel caso c) (file compresso), il SdI trasmette al soggetto ricevente il
file o i file in esso contenuti e non il file compresso.
Ogni file inviato al Sistema di Interscambio deve avere un nome diverso da
qualsiasi altro file inviato in precedenza.
```

<!-- PDF page 10 -->

```text
3.          MODALITÀ DI TRASMISSIONE DELLE FATTURE, RICEVUTE E NOTIFICHE
               Il presente capitolo descrive le specifiche tecniche per l’utilizzo dei canali
               di trasmissione dei file contenenti le fatture, così come descritto nel
               precedente paragrafo 2.2, e dei messaggi di ricevuta e di notifica.

     3.1    TRASMISSIONE DEL FILE AL SDI
               La trasmissione dei file verso il SDI può essere effettuata utilizzando le
               seguenti modalità:
               - un sistema di posta elettronica certificata, o di analogo sistema di posta
                elettronica basato su tecnologie che certifichino data e ora dell'invio e
                della ricezione delle comunicazioni e l'integrità del contenuto delle
                stesse, di seguito “servizio PEC”,
               - un sistema di cooperazione applicativa, su rete Internet, con servizio
                esposto tramite modello “web service” fruibile attraverso protocollo
                HTTPS, di seguito “servizio SdICoop”;
               - un sistema di cooperazione applicativa tramite porte di dominio in
                ambito Sistema Pubblico di Cooperazione (SPCoop), di seguito “servizio
                SPCoop”;
               - un sistema di trasmissione dati tra terminali remoti basato su protocollo
                FTP, di seguito “servizio SdIFtp”;
               - un sistema di trasmissione per via telematica attraverso il sito del
                Sistema d’Interscambio www.fatturapa.gov.it o l’interfaccia web di
                fruizione dei servizi “Fatture e corrispettivi”.

           3.1.1 POSTA ELETTRONICA CERTIFICATA (SERVIZIO PEC)
               Il soggetto trasmittente che intende utilizzare la posta elettronica
               certificata, deve avvalersi di un gestore con il quale mantenere un rapporto
               finalizzato alla disponibilità del servizio di PEC. Tale gestore deve essere
               tra quelli inclusi in apposito elenco pubblico gestito dall’AgID, così come
               disciplinato dagli artt. 14 e 15 del DPR 11 febbraio 2005, n.
               68 (“Regolamento recante disposizioni per l’utilizzo della posta elettronica
               certificata, a norma dell’art. 27 della legge 16 gennaio 2003, n. 3” - G.U.
               28 aprile 2005, n. 97).
               Il file da trasmettere costituisce l’allegato del messaggio di posta.
               Il messaggio con relativi allegati non deve superare la dimensione di 30
               megabyte, valore che costituisce il limite massimo entro il quale il gestore
               è tenuto a garantire il suo invio, come previsto dall’art. 12 del DM 2
               novembre 2005 (“Regole tecniche per la formazione, la trasmissione e la
               validazione, anche temporale, della posta elettronica certificata” - G.U. 15
```

<!-- PDF page 11 -->

```text
novembre 2005, n. 266); nel rispetto di tale limite dimensionale è possibile
inviare, con lo stesso messaggio, uno o più file allegati.
L’utilizzo della PEC garantisce di per sé l’identificazione del soggetto
trasmittente; ciò consente di non ricorrere a procedure di identificazione
del soggetto come attività propedeutiche alla trasmissione e ne deriva che
il SdI entra in contatto per la prima volta con il soggetto trasmittente nel
momento in cui si verifica la ricezione del primo messaggio di posta.
Per garantire una gestione efficiente del processo di trasmissione il SdI
utilizza più indirizzi di PEC tramite i quali ricevere i file; la procedura
tramite la quale vengono gestiti gli indirizzi è descritta di seguito.
La prima volta che il soggetto trasmittente intende utilizzare la PEC, deve
inviare il messaggio e i relativi file allegati all’indirizzo di posta elettronica
certificata del SdI, sdi01@pec.fatturapa.it, pubblicato sul sito web
www.fatturapa.gov.it; il SdI, con il primo messaggio di risposta (notifica di
errore, ricevuta di consegna, ricevuta di mancata consegna o attestazione
di avvenuta trasmissione della fattura con impossibilità di recapito),
comunica al soggetto trasmittente l’indirizzo di PEC che dovrà utilizzare
per le successive eventuali trasmissioni e che verrà utilizzato anche dal
SdI per i messaggi in risposta; in questo modo il SdI individua,
flessibilmente, un indirizzo di PEC dedicato, ma non esclusivo, per il
colloquio con ogni soggetto trasmittente.
L’utilizzo di un indirizzo di PEC diverso da quello assegnato dal SdI non
garantisce il buon fine della ricezione del messaggio di posta da parte del
SdI stesso.
Il normale flusso di trasmissione tramite PEC prevede, se il processo di
invio e ricezione va a buon fine, che al mittente vengano recapitate due
ricevute: una di accettazione da parte del proprio gestore di posta, e una
di avvenuta consegna da parte del gestore di posta del destinatario;
quest’ultima non garantisce di per sé la lettura del messaggio e del relativo
allegato, ma ne attesta il solo “deposito” nella casella del SdI.
L’avvenuta lettura del messaggio da parte del SdI, ed il corretto recapito
della fattura allegata al soggetto ricevente, sono certificati dal SdI stesso
attraverso la predisposizione e l’invio al mittente di ricevute e notifiche ad
hoc, secondo il sistema di comunicazione descritto al successivo
paragrafo 4.5.
Inoltre, a fronte dell’invio di una PEC priva di allegato da parte del
soggetto trasmittente, il SdI invia un messaggio di cortesia allo stesso
informandolo dell’assenza dell’allegato.
```

<!-- PDF page 12 -->

```text
3.1.2 COOPERAZIONE APPLICATIVA SU RETE INTERNET (SERVIZIO SDICOOP -
        TRASMISSIONE)
            Il SdI mette a disposizione su rete Internet un servizio web,
            richiamabile da un sistema informatico o da una applicazione, che
            consente di trasmettere i file come allegati di un messaggio SOAP.
            La dimensione massima del file allegato al messaggio deve essere di
            5 megabyte.
            Diversamente dalla PEC che consente, nel limite dimensionale di 30
            megabyte, di inviare con un solo messaggio più file allegati, questa
            modalità permette la trasmissione di un solo file (fattura singola
            piuttosto che lotto di fatture piuttosto che archivio di fatture) alla volta.
            Il servizio esposto ha le caratteristiche seguenti:
            protocollo HTTPS come trasporto su canale cifrato TLS 1.2;
            -    SOAP (with attachments) come standard per i messaggi;
            -    MTOM (Message Transmission Optimization Mechanism);
            -    WSDL (Web Services Description Language) per descrivere
      l'interfaccia pubblica del web service;
            -    autenticazione e autorizzazione basata sull’utilizzo di certificati.

    Questa modalità di trasmissione prevede:
         - l’accreditamento al SdI del canale trasmissivo tramite il Sistema
            di Accreditamento;
         - la richiesta di certificati (validi anche per la produzione) sul
            Sistema di Accreditamento al fine di rendere possibile il colloquio
            con il SdI;
         - il completamento dei test di interoperabilità previsti dal canale;
         - la presa visione dell’Accordo di servizio reso disponibile sul
            Sistema di Accreditamento al temine dei test di interoperabilità
            conclusi positivamente;
         - il passaggio in produzione, operazione attraverso la quale il
            canale viene ufficialmente riconosciuto come canale accreditato;
         - la capacità di gestione di certificati digitali.

    Il file, inviato come allegato SOAP, deve essere identificato secondo le
    regole di nomenclatura previste al precedente paragrafo 2.2.

    Tutte le informazioni relative alle procedure per l’utilizzo del servizio, così
    come la descrizione formale attraverso WSDL dei servizi, sono riportate
    nel documento Istruzioni per il servizio SDICoop - Trasmissione pubblicato
    sul sito web www.fatturapa.gov.it nella sezione Documentazione Sistema
    di Interscambio.
```

<!-- PDF page 13 -->

```text
3.1.3    PORTE DI DOMINIO IN AMBITO SPCOOP (SERVIZIO SPCOOP - TRASMISSIONE)

         Il SdI dispone di una porta di dominio qualificata su rete SPC secondo le
         modalità e le caratteristiche previste dalla normativa di riferimento (fino allo
         scorso luglio 2017) per i servizi di cooperazione tramite porta applicativa.

         Il servizio consente di trasmettere le fatture come file allegati ad una busta di
         e-gov; questa modalità permette la trasmissione di un solo file (fattura
         singola piuttosto che lotto di fatture piuttosto che archivio di fatture) alla
         volta. La dimensione massima del file allegato al messaggio deve essere di
         5 megabyte. Tutte le informazioni relative alle procedure per l’utilizzo del
         servizio, così come la descrizione formale attraverso WSDL dei servizi, sono
         riportate nel documento Istruzioni per il servizio SPCoop - Trasmittente
         pubblicato sul sito web www.fatturapa.gov.it nella sezione Documentazione
         Sistema di interscambio.

         L’Agenzia per l’Italia Digitale con “Le Linee guida per transitare al nuovo
         Modello di interoperabilità” - approvate con Determinazione del Direttore
         Generale n. 219/2017 – ha previsto che le Porte di dominio non
         costituiscano più componenti architetturali del Sistema di Cooperazione
         applicativa (SPC). Nello stesso documento ha specificato che dal 31
         dicembre 2017 non rinnoverà i certificati di qualificazione delle porte di
         dominio e che fino all’emanazione delle nuove regole tecniche per
         l’interoperabilità tra le Pubbliche Amministrazioni è possibile continuare ad
         utilizzare la medesima infrastruttura, a patto di dotarsi autonomamente dei
         certificati digitali.
         In sintesi, le indicazioni dell’AgID alle Amministrazioni che fanno attualmente
         uso del Sistema di Cooperazione applicativa (SPC) sono di predisporsi ad
         erogare i propri servizi senza l’intermediazione delle Porte; nel frattempo le
         Porte di dominio già qualificate e attualmente in produzione potranno essere
         temporaneamente mantenute come componenti architetturali interni ai
         sistemi delle amministrazioni, ed in questa logica esse non saranno più
         censite nel dominio spcoop.gov.it ma dovranno essere registrate nei domini
         istituzionali delle amministrazioni stesse. Alla scadenza dei certificati server
         attualmente installati sulle Porte di dominio, e fino allo scorso dicembre
         fornite da AgID, le amministrazioni dovranno provvedere direttamente a
         dotarsi di nuovi certificati attraverso propri processi di approvvigionamento.

         A seguito di tale previsione:

         -    a partire dal 23 aprile 2018 non è più possibile accreditare una Porta di
              dominio come canale di comunicazione con il Sistema di interscambio
              per la fatturazione elettronica;
```

<!-- PDF page 14 -->

```text
         -     la ricezione di fatture elettroniche da porte di dominio già accreditate sarà
               supportata fino alla migrazione del sistema verso il nuovo modello di
               interoperabilità, in ogni caso entro e non oltre il 20 gennaio 2020;

         -     le PA che intendono temporaneamente, fino all’adozione del nuovo
               modello di interoperabilità, mantenere la comunicazione con il Sistema di
               Interscambio attraverso Porta di dominio dovranno, almeno 1 mese
               prima della scadenza del certificato, inviare una richiesta di modifica delle
               condizioni definite nell’accordo di servizio registrato in fase di
               accreditamento. Le modalità con le quali trasmettere tale richiesta
               saranno comunicate direttamente alle PA interessate

         Sarà cura dell’Amministrazione aggiornare di conseguenza anche le
         informazioni di riferimento all’interno dell’indice delle PA (IPA).

3.1.4    SISTEMA DI TRASMISSIONE DATI TRA TERMINALI REMOTI BASATO SU PROTOCOLLO FTP
         (SERVIZIOSDIFTP)
             L’invio dei file, preventivamente crittografati, al SdI, previo accordo con i
             soggetti interessati per disciplinare aspetti particolari di trasmissione, è
             possibile attraverso protocolli di interconnessione e canali trasmissivi,
             anche eventualmente già in uso (seppure per altre finalità) nei rapporti con
             l’Amministrazione finanziaria, in ogni caso all'interno di circuiti chiusi che
             identificano in modo certo i partecipanti assicurando la qualificazione del
             canale.
             La dimensione massima del supporto contenente i file deve essere di 150
             megabyte.

         Per utilizzare il servizio è necessario:
         • l’accreditamento al SdI del canale trasmissivo tramite il Sistema di
            Accreditamento;
         • la dotazione di un server SFTP esposto su Internet;
         • il completamento dei test di interoperabilità previsti dal canale;
         • la presa visione dell’Accordo di servizio reso disponibile sul Sistema di
            Accreditamento al temine dei test di interoperabilità conclusi
            positivamente;
         • il passaggio in produzione, operazione attraverso la quale il canale viene
            ufficialmente riconosciuto come canale accreditato.

         Tutte le informazioni relative alle procedure per l’utilizzo del servizio, così
         come la descrizione formale attraverso WSDL dei servizi, sono riportate nel
         documento Istruzioni per il servizio SDIFTP pubblicato sul sito web
```

<!-- PDF page 15 -->

```text
       www.fatturapa.gov.it                 nella       sezione           Documentazione   Sistema         di
       interscambio.

          3.1.5          INVIO TRAMITE WEB
      Tale modalità prevede l’utilizzo di una funzionalità di trasmissione telematica
      via internet con protocollo HTTPS come trasporto, su canale cifrato TLS 1.2 ,
      disponibile nella sezione Inviare la FatturaPA sul sito www.fatturapa.gov.it o
      su interfaccia web di fruizione dei servizi “Fatture e corrispettivi”; tramite
      questa funzionalità il soggetto trasmittente potrà inviare la fattura o l’archivio
      di fatture (la dimensione del file da trasmettere non può eccedere il limite di 5
      megabyte); successivamente all’invio l’utente potrà aspettare direttamente
      online l’esito dei controlli svolti sulla/fattura/e e ricevere direttamente la
      ricevuta di consegna o l’eventuale notifica di scarto, ovvero potrà visualizzare
      l’esito dell’invio in un secondo momento accedendo alle funzionalità di
      monitoraggio a disposizione sul sito del SdI o sull’interfaccia web sopra
      indicati .

3.2    TRASMISSIONE DEL FILE AL SOGGETTO RICEVENTE
      Il SdI trasmette al soggetto ricevente la fattura tramite l’inoltro del file ricevuto
      in ingresso utilizzando canali di trasmissione analoghi a quelli utilizzati per la
      ricezione.
      Le modalità previste per l’inoltro sono:
       - un sistema di posta elettronica certificata, o di analogo sistema di posta
           elettronica basato su tecnologie che certifichino data e ora dell'invio e
           della ricezione delle comunicazioni e l'integrità del contenuto delle
           stesse, di seguito “Servizio PEC”;
       - un sistema di porte di dominio in ambito Sistema Pubblico di
           Cooperazione (SPCoop), di seguito “Servizio SPCoop - Ricezione”;
       - un sistema di cooperazione applicativa, su rete Internet, con servizio
           esposto tramite modello “web service” fruibile attraverso protocollo
           HTTPS, di seguito “servizio SdICoop - Ricezione”;
       - un sistema di trasmissione dati tra terminali remoti basato su protocollo
           FTP, di seguito “Servizio SdIFtp”.
      Oltre a trasmettere il file ricevuto in ingresso il SdI invia i dati utili per
      agevolare l’elaborazione del file stesso da parte del soggetto ricevente; i dati
      sono riportati nel messaggio “notifica dei metadati del file fattura” (rif.
      paragrafo 4.5) che viene trasmesso come file XML.
```

<!-- PDF page 16 -->

```text
3.2.1     POSTA ELETTRONICA CERTIFICATA (SERVIZIO PEC)
          Il soggetto che per ricevere le fatture elettroniche dal SdI intende utilizzare la
          posta elettronica certificata, deve avvalersi di un gestore con il quale
          mantenere un rapporto finalizzato alla disponibilità del servizio di PEC. Tale
          gestore deve essere tra quelli inclusi in apposito elenco pubblico gestito
          dall’Agenzia per l’Italia Digitale, così come disciplinato dagli artt. 14 e 15 del
          DPR 11 febbraio 2005, n. 68 (“Regolamento recante disposizioni per l’utilizzo
          della posta elettronica certificata, a norma dell’art. 27 della legge 16 gennaio
          2003, n. 3” - G.U. 28 aprile 2005, n. 97).
          Il SdI, quale indirizzo di PEC di destinazione, utilizzerà:
        -       nel caso di fatture indirizzate a pubbliche amministrazioni,, quello
                indicato in corrispondenza del codice ufficio riportato all’interno
                dell’anagrafica di riferimento, secondo le regole e le modalità descritte
                nell’allegato D al DM n. 55 del 3 aprile 2013 e nelle specifiche operative
                pubblicate sul sito www.indicepa.gov.it;
        -       nel caso di fatture indirizzate a soggetti diversi da pubbliche
                amministrazioni, quello indicato all’interno della fattura nell’elemento
                appositamente previsto (<PECDestinatario>).
            Il file inoltrato costituisce l’allegato del messaggio di posta; sempre allegato
            al medesimo messaggio di posta il SdI trasmetterà il file di “notifica dei
            metadati del file fattura” (rif. paragrafo 4.5): ogni messaggio di posta
            riporterà in allegato un solo “file fattura” ed un solo “file di metadati”.
            Il normale flusso di trasmissione tramite posta elettronica certificata
            prevede, se il processo di invio e ricezione va a buon fine, che al SdI
            vengano recapitate nella propria casella di PEC due ricevute: una di
            accettazione da parte del proprio gestore di posta, e una di avvenuta
            consegna da parte del gestore di posta del destinatario; quest’ultima attesta
            il ‘deposito’, nella casella di PEC del soggetto ricevente, del messaggio e
            dei relativi allegati ed ha valore, per il SdI, di “messa a disposizione della
            fattura al destinatario” e pertanto dà luogo all’invio al soggetto trasmittente
            della “ricevuta di consegna” (rif. paragrafo 4.5).

3.2.2     COOPERAZIONE APPLICATIVA SU RETE INTERNET (SERVIZIO SDICOOP - RICEZIONE)
           L’utilizzo di tale modalità è possibile per i soggetti che mettono a
           disposizione su rete Internet un servizio web che consente al SdI,
           richiamando tale servizio, di trasmettere il file fattura ed il file di “notifica dei
           metadati del file fattura” come allegato di un messaggio SOAP.
           Il servizio esposto ha le caratteristiche seguenti:
           - protocollo HTTPS come trasporto su canale cifrato TLS 1.2;
           - SOAP (with attachments) come standard per i messaggi;
           - MTOM (Message Transmission Optimization Mechanism);
```

<!-- PDF page 17 -->

```text
          - WSDL (Web Services Description Language) per descrivere l'interfaccia
              pubblica del web service;
          - autenticazione e autorizzazione basata sull’utilizzo di certificati.

          Questa modalità di ricezione prevede:
            - l’accreditamento al SdI del canale trasmissivo tramite il Sistema di
                 Accreditamento;
            - la richiesta di certificati (validi anche per la produzione) sul Sistema di
                 Accreditamento al fine di rendere possibile il colloquio con il SDI;
            - il completamento dei test di interoperabilità previsti dal canale;
            - la presa visione dell’Accordo di servizio reso disponibile sul Sistema
                 di Accreditamento al temine dei test di interoperabilità conclusi
                 positivamente;
            - il passaggio in produzione, operazione attraverso la quale il canale
                 viene ufficialmente riconosciuto come canale accreditato;
         - la capacità di gestione di certificati digitali.

          Tutte le informazioni relative alle procedure per l’utilizzo del servizio, così
          come la descrizione formale attraverso WSDL dei servizi, sono riportate nel
          documento Istruzioni per il servizio SDICoop - Ricezione pubblicato sul sito
          web www.fatturapa.gov.it nella sezione Documentazione Sistema di
          interscambio.

3.2.3    PORTE DI DOMINIO IN AMBITO SPCOOP (SERVIZIO SPCOOP - RICEZIONE)
         L’utilizzo di tale modalità è possibile, fino alla transizione verso il nuovo
         modello di interoperabilità definito da AgID, e comunque non oltre il 20
         gennaio 2020 per i soggetti attestati su rete SPC e che dispongono di una
         porta di dominio qualificata secondo quanto previsto dalle regole tecniche
         della SPC-Coop Dpcm 1 aprile 2008 pubblicate nella G.U. n. 144 del 21
         giugno 2008.
         Il servizio, esposto da una porta di dominio qualificata, consente al SdI di
         trasmettere il file fattura ed il file di “notifica dei metadati del file fattura” (rif.
         paragrafo 4.5) come allegato ad una busta di e-gov.

         L’Agenzia per l’Italia Digitale con “Le Linee guida per transitare al nuovo
         Modello di interoperabilità” - approvate con Determinazione del Direttore
         Generale n. 219/2017 – ha previsto che le Porte di dominio non
         costituiscano più componenti architetturali del Sistema di Cooperazione
         applicativa (SPC). Nello stesso documento ha specificato che dal 31
         dicembre 2017 non rinnoverà i certificati di qualificazione delle porte di
         dominio e che fino all’emanazione delle nuove regole tecniche per
```

<!-- PDF page 18 -->

```text
         l’interoperabilità tra le Pubbliche Amministrazioni è possibile continuare ad
         utilizzare la medesima infrastruttura, a patto di dotarsi autonomamente dei
         certificati digitali.
         In sintesi, le indicazioni dell’AgID alle Amministrazioni che fanno attualmente
         uso del Sistema di Cooperazione applicativa (SPC) sono di predisporsi ad
         erogare i propri servizi senza l’intermediazione delle Porte; nel frattempo le
         Porte di dominio già qualificate e attualmente in produzione potranno essere
         temporaneamente mantenute come componenti architetturali interni ai
         sistemi delle amministrazioni, ed in questa logica esse non saranno più
         censite nel dominio spcoop.gov.it ma dovranno essere registrate nei domini
         istituzionali delle amministrazioni stesse.
         Alla scadenza dei certificati server attualmente installati sulle Porte di
         dominio, e fino allo scorso dicembre fornite da AgID, le amministrazioni
         dovranno provvedere direttamente a dotarsi di nuovi certificati attraverso
         propri processi di approvvigionamento.

         A seguito di tale previsione:

         -    a partire dal 23 aprile 2018 non è più possibile accreditare una Porta di
              dominio come canale di comunicazione con il Sistema di interscambio
              per la fatturazione elettronica;

         -    la ricezione di fatture elettroniche da porte di dominio già accreditate sarà
              supportata fino alla migrazione del sistema verso il nuovo modello di
              interoperabilità;

         -    le PA che intendono temporaneamente, fino all’adozione del nuovo
              modello di interoperabilità, mantenere la comunicazione con il Sistema di
              Interscambio attraverso Porta di dominio dovranno, almeno 1 mese
              prima della scadenza del certificato, inviare una richiesta di modifica delle
              condizioni definite nell’accordo di servizio registrato in fase di
              accreditamento. Le modalità con le quali trasmettere tale richiesta
              saranno comunicate direttamente alle PA interessate.

         Sarà cura dell’Amministrazione aggiornare di conseguenza anche le
         informazioni di riferimento all’interno dell’indice delle PA (IPA).

3.2.4    SISTEMA DI TRASMISSIONE DATI TRA TERMINALI REMOTI BASATO SU PROTOCOLLO FTP
         (SERVIZIOSDIFTP)
        L’invio dei file, preventivamente crittografati, dal SdI al soggetto ricevente è
        possibile, previo accordo con i soggetti interessati volto a disciplinare aspetti
        particolari di trasmissione, attraverso protocolli di interconnessione e canali
```

<!-- PDF page 19 -->

```text
      trasmissivi anche eventualmente già in uso (seppure per altre finalità) nei
      rapporti con l’Amministrazione finanziaria, in ogni caso all'interno di circuiti
      chiusi che identificano in modo certo i partecipanti          assicurando la
      qualificazione del canale.

      Per utilizzare il servizio è necessario:
          - l’accreditamento al SdI del canale trasmissivo tramite il Sistema di
               Accreditamento;
          - la dotazione di un server SFTP esposto su Internet;
          - il completamento dei test di interoperabilità previsti dal canale; la
               presa visione dell’Accordo di servizio reso disponibile sul Sistema di
               Accreditamento al temine dei test di interoperabilità conclusi
               positivamente; il passaggio in produzione, operazione attraverso la
               quale il canale viene ufficialmente riconosciuto come canale
               accreditato.

      Tutte le informazioni relative alle procedure per l’utilizzo del servizio, così
      come la descrizione formale attraverso WSDL dei servizi, sono riportate nel
      documento Istruzioni per il servizio SDIFTP pubblicato sul sito web
      www.fatturapa.gov.it nella sezione Documentazione Sistema di interscambio.

3.3    MODALITÀ DI INOLTRO DELLE COMUNICAZIONI DA PARTE DEL SDI
      Le comunicazioni prodotte dal SdI vengono inoltrate, salvo diversa
      indicazione, tramite lo stesso canale utilizzato per la trasmissione del file
      fattura. Le comunicazioni sono costituite da file XML firmati elettronicamente,
      con firma XAdES Baseline B (XML Advanced Electronic Signatures) in
      modalità “enveloped” come previsto dalla normativa europea vigente in
      materia di firma elettronica avanzata; i file vengono trasmessi come allegati
      ai messaggi previsti per i diversi canali trasmissivi. La struttura XML dei file di
      comunicazione è dettagliata nel documento “Formato dei file di
      comunicazione del SDI – guida all’utilizzo” (rif. Allegato B-1).
      In particolare:
      -     nel caso di “servizio PEC”, le comunicazioni sono spedite all’indirizzo di
      PEC del mittente o del destinatario del file fattura; i messaggi PEC sono
      costituiti da una versione “Human Readable” (il corpo del messaggio) e dal
      file XML in allegato;
          - nel caso di “servizio SdICoop” l’accordo prevede l’esposizione, da parte
            del soggetto che ha trasmesso o ricevuto il file fattura, di un analogo
            servizio richiamabile dal SdI per l’invio delle comunicazioni come file XML
            in allegato al messaggio soap;
          - nel caso di “servizio SPCoop” l’accordo prevede l’esposizione, da parte
            del soggetto che ha trasmesso o ricevuto il file fattura, di un analogo
```

<!-- PDF page 20 -->

```text
            servizio richiamabile dal SdI per l’invio delle comunicazioni come file XML
            in allegato alla busta e-gov;
          - nel caso di “servizio SdIFtp”, le comunicazioni vengono inoltrate
            utilizzando lo stesso protocollo ed attraverso lo stesso canale (nel caso di
            canale bidirezionale) o canale dedicato (nel caso di canali unidirezionali);
          - nel caso di invio telematico attraverso il servizio esposto su
            www.fatturapa.gov.it o su interfaccia web di fruizione dei servizi “Fatture
            e corrispettivi”, le comunicazioni sono reperibili, come file XML, tramite la
            funzionalità di monitoraggio disponibile sul sito del SdI o sull’interfaccia
            suindicata.

3.4    MODALITÀ DI INOLTRO DELLE COMUNICAZIONI DAL SOGGETTO RICEVENTE AL SDI
        Le comunicazioni che il soggetto ricevente deve inviare al SdI vengono
        inoltrate, salvo diversa indicazione, attraverso lo stesso canale utilizzato per
        la trasmissione del file fattura dal SdI al soggetto ricevente stesso. Le
        comunicazioni sono costituite da file XML che vengono trasmessi come
        allegati ai messaggi previsti per i diversi canali trasmissivi. La struttura XML
        dei file di comunicazione è dettagliata nel documento “Formato dei file di
        comunicazione del SDI – guida all’utilizzo” (rif. Allegato B-1).
        I file, a discrezione del soggetto ricevente, possono essere firmati
        elettronicamente, con firma XAdES Baseline B (XML Advanced Electronic
        Signatures) in modalità “enveloped”, come previsto dalla normativa europea
        vigente in materia di firma elettronica avanzata.
        In particolare:
       - nel caso di “servizio PEC”, le comunicazioni sono spedite dal soggetto
           ricevente al medesimo indirizzo di PEC utilizzato dal SdI per la
           trasmissione; i messaggi PEC sono costituiti da una versione “Human
           Readable” (il corpo del messaggio) e dal file XML in allegato;
       - nel caso di “servizio SdICoop - Ricezione” l’accordo prevede
           l’esposizione, da parte dello SdI, di un analogo servizio richiamabile dal
           soggetto ricevente per l’invio delle comunicazioni come file XML in
           allegato al messaggio soap;
       - nel caso di “servizio SPCoop - Ricezione” l’accordo prevede
           l’esposizione, da parte dello SdI, di un analogo servizio richiamabile dal
           soggetto ricevente per l’invio delle comunicazioni come file XML in
           allegato alla busta e-gov;
       - nel caso di “servizio SdIFtp”, le comunicazioni vengono inoltrate
           utilizzando lo stesso protocollo ed attraverso lo stesso canale (nel caso di
           canale bidirezionale) o canale dedicato (nel caso di canali unidirezionali).
```

<!-- PDF page 21 -->

```text
4.        MODALITÀ DI INTERAZIONE CON IL SISTEMA DI INTERSCAMBIO
Il presente capitolo descrive le procedure operative per la trasmissione delle fatture
elettroniche attraverso il SdI, e lo scambio di informazioni (ricevute, notifiche) tra gli
attori del processo.

4.1       PROCEDURA PER L’ACCREDITAMENTO                               DEL      SOGGETTO      RICEVENTE     AI     FINI
          DELL’INTERAZIONE CON IL SDI

          Affinché il soggetto ricevente possa validamente interagire con il SdI devono
          essere soddisfatti i requisiti indispensabili alla sua qualificazione e al suo
          riconoscimento.

          Il soddisfacimento di tali requisiti passa attraverso la definizione del canale di
          trasmissione tramite:
            -    l’adozione di una casella di posta elettronica certificata (PEC) o di
                 analogo indirizzo di posta elettronica basato su tecnologie che
                 certifichino data e ora dell'invio e della ricezione delle comunicazioni e
                 l'integrità del contenuto delle stesse;
            -    la realizzazione del servizio di ricezione (WS-SDICoop), secondo le
                 modalità e le specifiche pubblicate sul sito www.fatturapa.gov.it;
            -    la qualificazione della porta di dominio (PdD-SPCoop), secondo le
                 regole disposte dall’Agenzia per l’Italia Digitale1, e la realizzazione del
                 servizio di ricezione e pubblicazione del relativo accordo di servizio,
                 secondo le modalità e le specifiche pubblicate sul sito
                 www.fatturapa.gov.it;
            -    la definizione di un accordo per disciplinare la trasmissione della
                 fattura, e relativi messaggi di notifica, attraverso protocolli di “file
                 transfer” all'interno di circuiti chiusi che identificano in modo certo i
                 partecipanti assicurando la qualificazione del canale.

          Inoltre, nel solo caso di amministrazioni pubbliche, passa attraverso
          l’immissione e l’aggiornamento dei dati di competenza all’interno

1 L’Agenzia per l’Italia digitale, con le “Linee guida per transitare al nuovo modello di interoperabilità”,

approvate con Determinazione n. 219/2017, comunica che dal 31 dicembre 2017 non rinnoverà i
certificati di qualificazione delle porte di dominio. Nel medesimo documento è specificato che fino
all’emanazione delle nuove regole tecniche per l’interoperabilità tra le Pubbliche Amministrazioni, è
possibile continuare ad utilizzare la medesima infrastruttura, a patto di dotarsi autonomamente dei
certificati digitali. Le PA che intendono mantenere l’infrastruttura e che rinnoveranno i certificati
pubblicandoli su nuovi domini, dovranno seguire le indicazioni contenute nei parr. 3.1.3 e 3.2.3
```

<!-- PDF page 22 -->

```text
         dell’anagrafica delle Pubbliche Amministrazioni (IPA) secondo le regole
         riportate nell’allegato D del DM n. 55 del 3 aprile 2013 e nelle specifiche
         tecniche pubblicate sul sito www.indicepa.gov.it e secondo le modalità
         previste dalle regole tecniche della SPC-Coop Dpcm 1 aprile 2008
         pubblicate nella G.U. n. 144 del 21 giugno 2008.

4.2      PROCEDURA PER L’ACCREDITAMENTO DEL SOGGETTO TRASMITTENTE AI FINI
         DELL’INTERAZIONE CON IL SDI

         Affinché il soggetto trasmittente possa validamente interagire con il SdI,
         devono essere soddisfatti i requisiti minimi indispensabili alla sua
         identificazione.

        Il soddisfacimento di tali requisiti si ottiene definendo uno o più canali di
        trasmissione/comunicazione attraverso:
        - dotazione di una casella di posta elettronica certificata o di analogo
            indirizzo di posta elettronica basato su tecnologie che certifichino data e
            ora dell'invio e della ricezione delle comunicazioni e l'integrità del
            contenuto delle stesse;
        - presa visione di un accordo di servizio con il SdI;
        - definizione di un accordo per disciplinare la trasmissione della fattura, e
            relativi messaggi di notifica, attraverso protocolli di “file transfer”
            all'interno di circuiti chiusi che identificano in modo certo i partecipanti
            assicurando la qualificazione del canale.
        Nel caso di utilizzo del servizio di invio telematico esposto su
        www.fatturapa.gov.it o su interfaccia web di fruizione dei servizi “Fatture e
        corrispettivi”, l’identificazione avviene attraverso l’indicazione di codice fiscale
        e password rilasciate dai servizi telematici dell’Agenzia delle Entrate
        Fisconline o Entratel oppure, in alternativa, attraverso l’utilizzo di un
        dispositivo di tipo Smartcard rispondente ai requisiti della Carta Nazionale dei
        Servizi-CNS, preventivamente registrata ai servizi telematici dell’Agenzia
        delle Entrate.

  4.3    PROCEDURA DI INVIO FATTURA AL SDI

         La procedura di invio della fattura al SdI vede, quali attori coinvolti:
         - il soggetto emittente;
         - il soggetto trasmittente (se diverso dal soggetto emittente);
         - il Sistema di Interscambio;

         I passaggi rappresentativi si possono schematizzare nei punti seguenti:
```

<!-- PDF page 23 -->

```text
       -     il soggetto emittente predispone la fattura elettronica secondo quanto
             riportato al precedente paragrafo 2;
       -     sulla fattura così predisposta il soggetto emittente appone la firma
             secondo una delle modalità descritte nel paragrafo 2;
       -     il file così generato, viene trasmesso dal soggetto trasmittente al SdI per
             mezzo dei canali e le modalità di cui al precedente paragrafo 3.

4.4    PROCEDURA DI INOLTRO DELLA FATTURA AL SOGGETTO RICEVENTE

       La procedura di inoltro della fattura elettronica dal SdI al soggetto ricevente
       vede, quali attori coinvolti:
       - il Sistema di Interscambio;
       - il soggetto ricevente.
        Il SdI, una volta effettuate le verifiche previste, inoltra al soggetto ricevente
        la fattura elettronica attraverso i canali e con le modalità di cui al
        precedente paragrafo 3.
        Nei casi in cui uno stesso soggetto svolge contemporaneamente il ruolo di
        intermediario sia per chi trasmette la fattura elettronica che per chi la riceve
        attraverso lo stesso canale trasmissivo, è possibile adottare un flusso
        semplificato per le cui particolarità si rimanda alla sezione Sistema di
        Interscambio – File, fatture e messaggi del sito www.fatturapa.gov.it.

4.5    PROCEDURA DI GESTIONE DELLE RICEVUTE E DELLE NOTIFICHE
           Tutti i canali di trasmissione descritti al precedente paragrafo 3 prevedono
           dei messaggi di ritorno a conferma del buon esito della trasmissione; questi
           messaggi sono specifici delle infrastrutture di comunicazione e
           garantiscono la “messa a disposizione” del messaggio e dei file allegati da
           parte di chi invia rispetto a chi riceve.
           Il SdI attesta l’avvenuto svolgimento delle fasi principali del processo di
           trasmissione delle fatture elettroniche attraverso un sistema di
           comunicazione che si basa sull’invio di ricevute e notifiche tramite le
           modalità ed i canali riportati ai precedenti paragrafi 3.3 e 3.4.

           La procedura di gestione delle ricevute e delle notifiche vede, quali attori
           coinvolti:
           - il soggetto trasmittente;
           - il Sistema di Interscambio;
           - il soggetto ricevente.
           La procedura può essere schematizzata nei punti seguenti:
```

<!-- PDF page 24 -->

```text
a) il SdI, ricevuto correttamente il file, assegna un identificativo proprio ed
   effettua le verifiche previste (paragrafo 5);
b) in caso di controlli con esito negativo, il SdI invia una notifica di scarto
   al soggetto trasmittente;
c) nel caso di esito positivo dei controlli il SdI trasmette la fattura elettronica
   al soggetto ricevente e unitamente a questa il SdI trasmette anche una
   notifica di metadati del file fattura tramite la quale sono comunicate le
   informazioni utili all’elaborazione ed alla comunicazione da parte del
   soggetto ricevente;
d) nel caso di buon esito della trasmissione, il SdI invia al soggetto
   trasmittente una ricevuta di consegna della fattura elettronica;
e) nel caso in cui, per cause tecniche non imputabili al SdI, la trasmissione
   al soggetto ricevente non fosse possibile entro i termini previsti riportati
   nel successivo paragrafo 5.1.2, il SdI invia al soggetto trasmittente una
   notifica di mancata consegna; qualora la fattura elettronica fosse
   destinata ad una pubblica amministrazione, resta a carico del SdI l’onere
   di contattare il destinatario affinché provveda tempestivamente alla
   risoluzione del problema ostativo alla trasmissione, e, a problema risolto,
   di procedere con l’invio; se, trascorsi 10 giorni dalla data di trasmissione
   della notifica di mancata consegna, il SdI non è riuscito a recapitare la
   fattura elettronica al soggetto ricevente , inoltra al soggetto trasmittente
   una definitiva attestazione di avvenuta trasmissione della fattura con
   impossibilità di recapito in ottemperanza alle disposizioni riportate nella
   circolare interpretativa n.1 del 31 marzo 2014, emessa dal Dipartimento
   Finanze del MEF di concerto con il Dipartimento della Funzione Pubblica
   della Presidenza del Consiglio dei Ministri; questa attestazione dà diritto
   al soggetto trasmittente           di recapitare direttamente la fattura
   all’amministrazione destinataria utilizzando canali a lui noti (es. Posta
   Elettronica Ordinaria, PEC, etc.), senza ulteriori passaggi attraverso il
   Sistema di Interscambio; nei casi di fattura elettronica destinata a
   soggetti diversa di pubblica amministrazione, tale attestazione attribuisce
   titolo di definitività alla mancata consegna;
f) per ogni fattura elettronica recapitata al soggetto ricevente , il SdI
   permette allo stesso , entro il termine di 15 giorni dalla prima
   comunicazione inviata al soggetto trasmittente, più precisamente dalla
   data riportata nella ricevuta di consegna o dalla data di trasmissione della
   notifica di mancata consegna, di inviare una notifica di
   accettazione/rifiuto (notifica di esito committente) della fattura e, nel
   caso, provvede ad inoltrarla al soggetto trasmittente a completamento
   del ciclo di comunicazione degli esiti della trasmissione della fattura
   elettronica; il soggetto ricevente può scegliere se comunicare o meno,
   tramite il Sistema di Interscambio, l’esito della fattura ricevuta; tale
   possibilità, pertanto, non si configura come un obbligo , né una fattura si
```

<!-- PDF page 25 -->

```text
    considera accettata automaticamente solo perché non è stata rifiutata dal
    soggetto ricevente tramite SdI;
 g) se entro il termine dei 15 giorni suddetto il SdI non riceve alcuna
    comunicazione, il SdI inoltra notifica di decorrenza dei termini sia al
    soggetto trasmittente sia al soggetto ricevente . Tale notifica ha la sola
    funzione di comunicare alle due parti che il SdI considera chiuso il
    processo relativo a quella fattura. La notifica di decorrenza termini
    segnala che il Sistema di Interscambio ha regolarmente concluso la
    gestione (ricezione e consegna) della fattura oggetto della notifica.
    Pertanto dopo tale notifica il SdI scarterà la fattura, se reinviata, e
    qualsiasi comunicazione ad essa relativa. La notifica di decorrenza
    termini non ha alcuna implicazione sulla verifica della correttezza e sulla
    successiva gestione della fattura da parte degli interessati; tali aspetti
    restano di esclusivo dominio del rapporto tra fornitore e cliente.

Le ricevute/notifiche vengono predisposte secondo un formato XML; le
specifiche tecniche e la documentazione di tali messaggi sono riportate nel
documento “Struttura dei messaggi di comunicazione del Sistema di
interscambio – guida all’utilizzo” (rif. Allegato B-1).
```

<!-- PDF page 26 -->

```text
5.       CONTROLLI EFFETTUATI DAL SDI
             Il SdI, per ogni file correttamente ricevuto, effettua una serie di controlli
             propedeutici all’inoltro al soggetto destinatario.
             Questa attività di verifica, nei limiti di ambito in cui è circoscritta, si configura
             come:
                - una operazione necessaria a minimizzare i rischi di errore in fase
                elaborativa;
                - uno strumento di filtro verso il soggetto ricevente per prevenire, da un
                lato, possibili e dispendiose attività di contenzioso, e per accelerare,
                dall’altro, eventuali interventi di rettifica sulle fatture a vantaggio di una
                più rapida conclusione del ciclo fatturazione-pagamento.

             Il mancato superamento di questi controlli genera lo scarto del file che,
             conseguentemente, non viene inoltrato al destinatario della fattura.

       5.1 TIPOLOGIE E MODALITÀ DI VERIFICA
             Le tipologie di controllo effettuate mirano a verificare:
               - nomenclatura ed unicità del file trasmesso;
               - dimensioni del file;
               - integrità del documento;
               - autenticità del certificato di firma;
               - conformità del formato fattura;
               - coerenza e validità del contenuto della fattura;
               - unicità della fattura;
               - recapitabilità della fattura.

     5.1.1       VERIFICHE EFFETTUATE SUI FILE FATTURA
        Nomenclatura ed unicità del file trasmesso
        La verifica viene eseguita al fine di intercettare ed impedire l’invio di un file
        già trasmesso; attraverso un controllo sulla nomenclatura del file ricevuto il
        SDI verifica che il nome file sia conforme con quanto riportato nel precedente
        paragrafo 2.2 e che non sia stato già inviato un file con lo stesso nome (ogni
        file inviato al Sistema di Interscambio deve avere un nome diverso da
        qualsiasi altro file inviato in precedenza); in caso di esito negativo del
        controllo (nome file già presente nel SDI o nome file non conforme) il file
        viene rifiutato con le seguenti motivazioni:
        -    Codice 00001 - Nome file non valido;
        -    Codice 00002 - Nome file duplicato.
```

<!-- PDF page 27 -->

```text
Dimensioni del file
La verifica è effettuata al fine di garantire che il file ricevuto non ecceda le
dimensioni ammesse per il rispettivo canale di trasmissione.
-    Codice 00003 – Le dimensioni del file superano quelle ammesse
Verifica di integrità del documento
La verifica viene effettuata al fine di garantire che il documento ricevuto non
abbia subito modifiche successivamente all’apposizione della firma;
attraverso un controllo sulla firma apposta sull’oggetto trasmesso, il SdI
verifica l’integrità dell’oggetto stesso; laddove dovesse emergere che il
documento ricevuto non corrisponde al documento sul quale è stata
apposta la firma, il documento viene rifiutato con la seguente motivazione:
-    Codice 00102 - La firma elettronica apposta al file non risulta valida.
Verifica di autenticità del certificato di firma
La verifica viene effettuata al fine di garantire la validità del certificato di
firma utilizzato per apporre la firma elettronica qualificata al documento;
sulla base delle informazioni messe a disposizione dalle “Certification
Authorities”, il SdI verifica la validità del certificato di firma, che non deve
risultare scaduto, revocato o sospeso; in caso di certificato di firma non
valido, il documento viene rifiutato con le seguenti motivazioni:

- Codice 00100 - Certificato di firma scaduto;

- Codice 00101 - Certificato di firma revocato;

- Codice 00104 – La CA (Certification Authority) che ha emesso il
certificato di firma non risulta nell’elenco delle CA affidabili;

- Codice 00107 – Il certificato di firma non è valido.

Verifica di conformità del formato fattura
La verifica viene effettuata per garantire che il contenuto del documento sia
rappresentato secondo le regole definite nelle specifiche tecniche; qualora
dovesse risultare una non corretta aderenza alle regole, il documento viene
rifiutato con le seguenti motivazioni:

- Codice 00103 – Alla firma elettronica apposta al file manca il riferimento
temporale;

- Codice 00105 – Il riferimento temporale associato alla firma elettronica
apposta al file è successivo alla data di ricezione del file;

- Codice 00106 – Il file compresso è vuoto oppure non è leggibile;
```

<!-- PDF page 28 -->

```text
- Codice 00200 - File non conforme al formato (nella descrizione del
messaggio è riportata l’indicazione puntuale della non conformità);

- Codice 00201 – Non è possibile procedere con ulteriori controlli perché
gli errori di formato presenti nel file superano il numero massimo previsto
(50);

Verifica di coerenza sul contenuto
La verifica viene effettuata per garantire la coerenza del contenuto degli
elementi informativi per come previsto dalle regole tecniche; in caso di
mancato rispetto di dette regole, il documento viene rifiutato con le seguenti
motivazioni:

- Codice 00400 – 2.2.1.14 <Natura> non presente a fronte di 2.2.1.12
<AliquotaIVA> pari a zero;

- Codice 00401 – 2.2.1.14 <Natura> presenta a fronte di 2.2.1.12
<AliquotaIVA> diversa da zero;

- Codice 00403 – 2.1.1.3 <Data> successiva alla data di ricezione;

- Codice 00411 – 2.1.1.5 <DatiRitenuta> non presente a fronte di almeno
un blocco 2.2.1 <DettaglioLinee> con 2.2.1.13 <Ritenuta> uguale a SI;

- Codice 00413 – 2.1.1.7.7 <Natura> non presente a fronte di 2.1.1.7.5
<AliquotaIVA> pari a zero;

- Codice 00414 – 2.1.1.7.7 <Natura> presente a fronte di 2.1.1.7.5
<AliquotaIVA> diversa da zero;

- Codice 00415 – 2.1.1.5 <DatiRitenuta> non presente a fronte di 2.1.1.7.6
<Ritenuta> uguale a SI;

- Codice 00417 - 1.4.1.1 <IdFiscaleIVA> e 1.4.1.2 <CodiceFiscale> non
valorizzati (almeno uno dei due deve essere valorizzato);

- Codice 00418 - 2.1.1.3 <Data> antecedente a 2.1.6.3 <Data>;

- Codice 00419 -      2.2.2   <DatiRiepilogo>      non     presente    in
corrispondenza di almeno un valore di 2.1.1.7.5 <AliquotaIVA> o 2.2.1.12
<AliquotaIVA>;

- Codice 00420 – 2.2.2.2 <Natura> con valore di tipo N6 a fronte di 2.2.2.7
<EsigibilitaIVA> uguale a S (scissione pagamenti);
```

<!-- PDF page 29 -->

```text
- Codice 00421 – 2.2.2.6 <Imposta> non calcolato secondo le regole
definite nelle specifiche tecniche;

- Codice 00422 – 2.2.2.5 <ImponibileImporto> non calcolato secondo le
regole definite nelle specifiche tecniche;

- Codice 00423 – 2.2.1.11 <PrezzoTotale> non calcolato secondo le
regole definite nelle specifiche tecniche;

- Codice 00424 – 2.2.1.12 <AliquotaIVA> o 2.2.2.1 <AliquotaIVA> o
2.1.1.7.5 <AliquotaIVA> non indicata in termini percentuali;

- Codice 00425 – 2.1.1.4 <Numero> non contenente caratteri numerici;

- Codice 00427 – 1.1.4 <CodiceDestinatario> di 7 caratteri non ammesso
a fronte di 1.1.3 <FormatoTrasmissione> con valore FPA12 o 1.1.4
<CodiceDestinatario> di 6 caratteri non ammesso a fronte di 1.1.3
<FormatoTrasmissione> con valore FPR12 o FSM10;

- Codice 00428 – 1.1.3 <FormatoTrasmissione> non coerente con il valore
dell’attributo VERSION;

- Codice 00429 – 2.2.2.2 <Natura> non presente a fronte di 2.2.2.1
<AliquotaIVA> pari a zero;

- Codice 00430 – 2.2.2.2 <Natura> presente a fronte di 2.2.2.1
<AliquotaIVA> diversa da zero;

- Codice 00437 – 2.1.1.8.2 <Percentuale> e 2.1.1.8.3 <Importo> non
presenti a fronte di 2.1.1.8.1 <Tipo> valorizzato;

- Codice 00438 – 2.2.1.10.2 <Percentuale> e 2.2.1.10.3 <Importo> non
presenti a fronte di 2.2.1.10.1 <Tipo> valorizzato.

- Codice 00443 - non c’è corrispondenza tra i valori indicati nell’elemento
2.2.1.12 <AliquotaIVA> o 2.1.1.7.5 <AliquotaIVA> e quelli dell’elemento
2.2.2.1 <AliquotaIVA>

- Codice 00444 - non c’è corrispondenza tra i valori indicati nell’elemento
2.2.1.14 <Natura> o 2.1.1.7.7 <Natura> e quelli dell’elemento 2.2.2.2
<Natura>

- Codice 00445 (controllo in vigore dal primo gennaio 2021) - non è più
ammesso il valore generico N2, N3 o N6 come codice natura
dell’operazione
```

<!-- PDF page 30 -->

```text
- Codice 00471 – per il valore indicato nell’elemento 2.1.1.1
<TipoDocumento> il cedente/prestatore non può essere uguale al
cessionario/committente

- Codice 00472 – per il valore indicato nell’elemento 2.1.1.1
<TipoDocumento> il cedente/prestatore deve essere uguale al
cessionario/committente

- Codice 00473 – per il valore indicato nell’elemento 2.1.1.1
<TipoDocumento> il valore nell’elemento 1.2.1.1.1 <IdPaese> non è
ammesso

- Codice 00474 – per il valore indicato nell’elemento 2.1.1.1
<TipoDocumento> non sono ammesse linee di dettaglio con l’elemento
2.2.1.12 <AliquotaIVA> contenente valore zero

- Codice: 00475 – per il valore indicato nell’elemento 2.1.1.1
<TipoDocumento> deve essere presente l’elemento 1.4.1.1 <IdFiscaleIVA>
del cessionario/committente

- Codice: 00476 – gli elementi 1.2.1.1.1 <IdPaese> e 1.4.1.1.1 <IdPaese>
non possono essere entrambi valorizzati con codice diverso da IT

- Codice: 00477 – fattura recante titolo di non imponibilità ai fini IVA ai
sensi dell’art. 8, comma 1, lett. c) DPR 26 ottobre 1972, n. 633, con
dichiarazione d’intento invalidata.
Verifica di validità del contenuto della fattura
La verifica viene effettuata per accertare la presenza e la validità dei dati
necessari al corretto inoltro del documento al destinatario e per prevenire
situazioni di dati errati e/o non elaborabili; in particolare viene effettuato un
controllo:
- sulla presenza nell’anagrafica di riferimento (IndicePA) del codice
identificativo del destinatario e delle informazioni necessarie al recapito,
ovvero che il codice destinatario corrisponda a uno dei valori di default
(“0000000” o “XXXXXXX”); il controllo prevede anche, per le fatture
destinate alle pubbliche amministrazioni, una verifica sulla data di avvio del
servizio di fatturazione elettronica presente nell’anagrafica IPA , data che
non deve essere successiva a quella in cui viene effettuato il controllo (data
di sistema);
- sulla presenza, nell’anagrafica IPA, per le fatture destinate alle pubbliche
amministrazioni, di uno o più uffici di fatturazione elettronica attivi associati
al    codice      fiscale   corrispondente      all’identificativo  fiscale   del
cessionario\committente riportato in fattura, nei casi in cui il formato
trasmissione identifichi una fattura destinata a soggetto privato (FPR12);
```

<!-- PDF page 31 -->

```text
- sulla presenza, nell’anagrafica IPA, per le fatture destinate alle pubbliche
amministrazioni, di uno ed un solo ufficio di fatturazione elettronica attivo
(diverso da quello Centrale previsto dalle specifiche operative relative
all’allegato D al DM 3 aprile 2013, n. 55) associato al codice fiscale
corrispondente all’identificativo fiscale del cessionario/committente riportato
in fattura, nei casi in cui il codice identificativo del destinatario sia
valorizzato con il codice di fatturazione elettronica Centrale in ottemperanza
alle disposizioni riportate nella circolare interpretativa n.1 del 31 marzo
2014 . L’ufficio Centrale (denominato “Uff_eFatturaPA”) è un ufficio virtuale
introdotto per far fronte ad eventuali inadempienze delle amministrazioni; è
generato su IPA dall’AgID. Tale ufficio può essere eliminato su richiesta
dell’Amministrazione solo dopo 12 mesi dall’avvio. Trattandosi di una
misura cautelativa, il suo utilizzo deve essere opportunamente controllato.
In particolare su tutte le fatture inviate ad un ufficio Centrale il SdI, sulla
base dell’identificativo fiscale del cessionario/committente indicato in
fattura, verifica su IPA la presenza di uffici con il servizio di fatturazione
elettronica attiva: se in corrispondenza dell’identificativo fiscale del
cessionario/committente esiste un unico ufficio “ordinario” con il servizio di
fatturazione elettronica attivo (oltre all’ufficio Centrale), la fattura viene
scartata con l’indicazione dell’ufficio corretto da utilizzare;
- sulla validità dei codici fiscali e delle partite IVA relative ai soggetti
trasmittente, cedente/prestatore, cessionario/committente, rappresentante
fiscale, attraverso una verifica di presenza nell’anagrafe tributaria; il
controllo non è effettuato per gli identificativi fiscali assegnati da autorità
estere;
- sulla coerenza tra partita IVA e codice fiscale del cedente/prestatore o
del cessionario/committente quando entrambi valorizzati e/o nei casi di
gruppi IVA (per le fatture destinate a PA, il controllo sulla coerenza in caso
di gruppi IVA vale solo per il cedente/prestatore).
Laddove anche uno solo di questi controlli non dovesse essere superato, il
documento viene rifiutato con le seguenti motivazioni:
- Codice 00300 – 1.1.1.2 <IdCodice> non valido;
- Codice 00301 – 1.2.1.1.2 <IdCodice> non valido;
- Codice 00302 – 1.2.1.2 <CodiceFiscale> non valido;
- Codice 00303 – 1.3.1.1.2 <IdCodice> o 1.4.4.1.2 <IdCodice> non valido;
- Codice 00304 – 1.3.1.2 <CodiceFiscale> non valido;
- Codice 00305 – 1.4.1.1.2 <IdCodice> non valido;
- Codice 00306 – 1.4.1.2 <CodiceFiscale> non valido;
- Codice 00311 – 1.1.4 <CodiceDestinatario> non valido
- Codice 00312 – 1.1.4 <CodiceDestinatario> non attivo
- Codice 00320 – 1.2.1.1 <IdFiscaleIVA> e 1.2.1.2 <CodiceFiscale> non
coerenti;
```

<!-- PDF page 32 -->

```text
- Codice 00321 – 1.2.1.2 <CodiceFiscale> di soggetto non partecipante al
gruppo IVA;
- Codice 00322 – 1.2.1.2 <CodiceFiscale> non presente a fronte di 1.2.1.1
<IdFiscaleIVA> di gruppo IVA;
- Codice 00323 – 1.2.1.1.2 <IdCodice> corrispondente a una partita IVA
cessata da oltre 5 anni;
- Codice 00324 – 1.4.1.1 <IdFiscaleIVA> e 1.4.1.2 <CodiceFiscale> non
coerenti;
- Codice 00398 - Codice Ufficio presente ed univocamente identificabile
nell’anagrafica IPA di riferimento, in presenza di 1.1.4 <CodiceDestinatario>
valorizzato con codice ufficio “Centrale”
- Codice 00399 - CessionarioCommittente presente nell’anagrafica IPA di
riferimento in presenza di 1.1.3 <FormatoTrasmissione> valorizzato a
“FPR12”

Verifiche di unicità della fattura
La verifica viene eseguita al fine di intercettare ed impedire l’inoltro di una
fattura già trasmessa e elaborata; in quest’ottica, qualora i dati contenuti
all’interno della fattura e relativi a:
- identificativo cedente/prestatore;
- anno della data fattura;
- numero fattura;
coincidano con quelli di una fattura precedentemente trasmessa e per la
quale non sia stata inviata al soggetto trasmittente una notifica di scarto
oppure una notifica di rifiuto da parte del destinatario, il documento viene
rifiutato con le seguenti motivazioni:
- Codice 00404 - Fattura duplicata
- Codice 00409 – Fattura duplicata nel lotto

Nel caso in cui il documento trasmesso sia una nota di credito
(TipoDocumento = TD04), la verifica tiene conto anche della tipologia di
documento; pertanto è ammessa la presenza di due documenti aventi
stesso cedente/prestatore, stesso anno e stesso numero solo qualora uno
dei due sia di tipo TD04.

Nel caso di fatture emesse secondo modalità e termini stabiliti con decreto
ministeriale ai sensi dell'articolo 73 del DPR 633/72 e per le quali sia stato
valorizzato a “SI” il campo “Art73” (situazione che consente l'emissione
nello stesso anno di più documenti aventi stesso numero), la verifica di
unicità viene effettuata secondo le regole precedenti ma tenendo conto
della data completa e non solo dell’anno.
```

<!-- PDF page 33 -->

```text
          Tutti i controlli descritti in questo paragrafo, qualora il file inviato al SdI è
          relativo ad un documento lotto di fatture (ex art. 1, comma 3, DLGS 20
          febbraio 2004 n. 52), comportano l’accettazione o il rifiuto del file nella sua
          totalità. Diversamente, se al SdI viene inviato un file in formato compresso
          (rif. precedente par. 2.2 lett. c), i controlli, con conseguente accettazione o
          scarto, riguardano ogni singolo file presente al suo interno.

5.1.2    TEMPI DI ELABORAZIONE DA PARTE DEL SDI
          Nella tabella seguente sono indicati gli eventi temporali (data e ora) che
          caratterizzano il processo e vengono riportati nelle ricevute di trasmissione
          e consegna del SdI:
         - T0: il momento in cui la fattura viene ricevuta dal SdI (campo
             DataOraRicezione del messaggio di notifica di consegna o di mancata
             consegna);
         - T1: il momento in cui la stessa viene ricevuta dal destinatario (campo
             DataOraConsegna del messaggio di notifica di consegna).

         Per identificare tali momenti si forniscono di seguito i riferimenti utilizzati
         sulla base del canale di trasmissione scelto dal mittente e dal destinatario. Il
         Sistema di Interscambio registra comunque, nell’ambito dei propri archivi, il
         momento in cui lo SdI invia la fattura al destinatario così come data/ora del
         primo tentativo di invio in caso di mancata consegna.

                CANALE di
                                                               T0                                          T1
              TRASMISSIONE

                                              Data e ora presenti all’interno                Data e ora presenti all’interno
                                               della ricevuta di consegna                      della ricevuta di consegna
                                             inviata al soggetto trasmittente                 inviata al SdI dal gestore di
                 Servizio PEC
                                                 dal suo gestore di PEC                       PEC del soggetto ricevente

                                             Data e ora presenti all’interno             Data e ora presenti all’interno
                                             della “response” del servizio                della “response” del servizio
              servizio SdICoop                      esposto dal SdI                      esposto dal soggetto ricevente
```

<!-- PDF page 34 -->

```text
                                     Data e ora presenti all’interno                 Data e ora presenti all’interno
                                      della “response” del servizio                   della “response” del servizio
                                     esposto dalla porta di dominio                  esposto dalla porta di dominio
       servizio SPCoop
                                                 del SdI                                 del soggetto ricevente

                                    Data e ora restituiti dal servizio           Data e ora restituiti dal servizio
       Servizio SdIFtp
                                            di trasmissione                              di trasmissione

                                        Data e ora restituiti dal
    Invio Telematico via
                                       messaggio di risposta della                          Canale non previsto
            web
                                          funzionalità di invio

La stima del tempo intercorrente tra il momento T0 ed il momento T1 può
essere quantificata in un tempo medio di circa 48 ore, variabile in ragione
della specificità del canale scelto dal soggetto ricevente e della frequenza di
afflusso delle fatture al Sistema di Interscambio.
Tale stima tiene anche conto di una media dei tempi di completamento delle
operazioni sui diversi canali. Le regole tecniche del servizio PEC, ad
esempio, prevedono 24 ore come tempo massimo per la sola fase di
consegna.
```

<!-- PDF page 35 -->

```text
              ALLEGATO B-1
STRUTTURA DEI MESSAGGI DI COMUNICAZIONE
      DEL SISTEMA DI INTERSCAMBIO
           GUIDA ALL’UTILIZZO

           [versione 1.8.4] 20/03/2026
```

<!-- PDF page 36 -->

```text
PREMESSA

I messaggi per la gestione delle ricevute/notifiche da inviare al soggetto trasmittente da
parte del SdI, o al SdI da parte del soggetto ricevente, sono contenuti in file XML descritti
dal file MessaggiTypes_v1.1.xsd disponibile nella sezione Documentazione Sistema di
Interscambio del sito www.fatturapa.gov.it e predisposti secondo le specifiche riportate nei
paragrafi seguenti.

Di seguito si fornisce una breve descrizione del significato delle colonne presenti nelle
tabelle:

Elemento XML:                                è il nome effettivo del tag XML utilizzato in fase di
                                             compilazione del file;

Descrizione Funzionale:                      indica una descrizione di tipo funzionale dell’Elemento
                                             XML;

Formati e Valori Ammessi:                    indica il tipo di formato del dato e, ove previste delle
                                             restrizioni sul dato, l’insieme dei valori ammessi per
                                             quell’Elemento XML (dati racchiusi tra parentesi
                                             quadra);

Obbligatorietà e Occorrenze:                 indica se il dato deve essere obbligatoriamente
                                             presente o meno, e la sua molteplicità:

                  <0.1> dato facoltativo; se presente, può figurare al massimo una volta

                  <0.N> dato facoltativo; se presente, può figurare N volte

                  <1.1> dato obbligatorio; figura al massimo una volta

                  <1.N> dato obbligatorio; figura almeno una volta

 Dimensione min…max:                         indica la dimensione minima e massima che può
                                             assumere l’Elemento XML; nel caso in cui sia indicato
                                             un solo numero di dimensione, la lunghezza del campo
                                             è esattamente uguale a quel preciso valore; la dicitura
                                             “Unbounded” sta per dimensioni illimitate.

                                  [versione 1.8.4] 20/03/2026
```

<!-- PDF page 37 -->

```text
1.            DESCRIZIONE E REGOLE DI COMPILAZIONE

     1.1          NOMENCLATURA DEI FILE PER LA TRASMISSIONE DI RICEVUTE/ NOTIFICHE
Il nome dei file per la trasmissione delle ricevute/notifiche, deve rispettare la
seguente nomenclatura generale:

                    Nome del file fattura ricevuto                     Tipo di                     Progressivo
                        senza estensione                              messaggio                      univoco

Il Nome del file fattura ricevuto senza estensione deve essere conforme alle
regole definite nel paragrafo 2.2. Nel caso in cui il nome file non sia conforme e la
sua lunghezza sia superiore ai 36 caratteri il nome sarà troncato ed i caratteri oltre
il 36-esimo non saranno presenti nella notifica di scarto.
Il Tipo di messaggio può assumere i seguenti valori:

Valore          Descrizione
 RC             Ricevuta di consegna
  NS            Notifica di scarto
 MC             Notifica di mancata consegna
  NE            Notifica esito cedente / prestatore
 MT             File dei metadati
  EC            Notifica di esito cessionario / committente
  SE            Notifica di scarto esito cessionario / committente
  DT            Notifica decorrenza termini
                Attestazione di avvenuta trasmissione della fattura con impossibilità di
     AT
                recapito

Il Progressivo univoco deve essere una stringa alfanumerica di lunghezza
massima 3 caratteri e con valori ammessi [a-z], [A-Z], [0-9] che identifica
univocamente ogni notifica / ricevuta relativa al file inviato.
Il carattere di separazione degli elementi componenti il nome file corrisponde
all’underscore (“_”), codice ASCII 95, l’estensione è sempre “.xml”.
Se il SdI ha ricevuto un file di tipo compresso, di cui al precedente paragrafo 2.2
lett. c), (es.: ITAAABBB99T99X999W_00001.zip ) e non è possibile accedere al
suo contenuto perché “corrotto”, il nome del file con il quale il SdI inoltra al
soggetto trasmittente la notifica di scarto è il seguente:
         ITAAABBB99T99X999W_00001_NS_001.xml
A queste regole di nomenclatura fa eccezione l’Attestazione di avvenuta
trasmissione della fattura con impossibilità di recapito (rif. paragrafo 1.10); in
questo      caso,     se    il   SdI     ha      ricevuto    un file  con    nome
ITAAABBB99T99X999W_00001.xml, inoltra al soggetto trasmittente il seguente
file .zip

                                     [versione 1.8.4] 20/03/2026
```

<!-- PDF page 38 -->

```text
        ITAAABBB99T99X999W_00001_AT_001.zip
che al suo interno contiene il file ricevuto (ITAAABBB99T99X999W_00001.xml) e
l’attestazione (ITAAABBB99T99X999W_00001_AT_001.xml).

  1.2          RICEVUTA DI CONSEGNA DEL FILE AL DESTINATARIO
È la ricevuta inviata dal SdI al soggetto trasmittente per comunicare l’avvenuta
consegna del file al destinatario.
Per tutti i canali trasmissivi essa è rappresentata da un file XML firmato (firma
elettronica non qualificata) contenente le seguenti informazioni:

                                  [versione 1.8.4] 20/03/2026
```

<!-- PDF page 39 -->

```text
                                                                                                            Obbligatorietà          Dim.
                             Descrizione
  Elemento XML                                                Formati e valori ammessi                            e
                             funzionale
                                                                                                             occorrenze           min-max

                          Numero attribuito
                            dal Sistema di
 IdentificativoSDI
                         Interscambio al file
                                                                  Formato numerico                              <1.1>              12
                               ricevuto

                          Nome attribuito al
                            file secondo le
     NomeFile
                          regole riportate su
                                                                Formato alfanumerico                            <1.1>             1 … 50
                         Disciplinare Tecnico

                          Data e ora in cui il
                                                    Il formato della data è rappresentato secondo il
                         file è stato ricevuto
DataOraRicezione
                             dal Sistema di
                                                  formato ISO 8601:2004, con la seguente precisione:            <1.1>              16
                                                                  YYYY-MM-DD-HH:MM
                             Interscambio
                           Data e ora in cui il
                                file è stato        Il formato della data è rappresentato secondo il
DataOraConsegna              consegnato dal       formato ISO 8601:2004, con la seguente precisione:            <1.1>              16
                                Sistema di                        YYYY-MM-DD-HH:MM
                              Interscambio
                          Blocco contenente
                             le informazioni
                          relative al soggetto
                            destinatario della
                            fattura (Codice e
   Destinatario             Denominazione;                       Campo complesso                                <1.1>
                            quest’ultima, per
                           soggetti diversi da
                                pubbliche
                            amministrazioni,
                              vale “NO PA”)
                           Opzionale. Valore
                         presente nel caso di
                               Ricevuta di
RiferimentoArchivio
                          consegna relativa a
                                                                 Campo complesso                                <0.1>
                         fattura appartenente
                              a file archivio.

                           Identificativo del                                                                                1 … 99999999999999
    MessageId
                              messaggio
                                                                  Formato numerico                              <1.1>

                         Opzionale. Presente
                           solo nel caso di
                         messaggi inviati con
  PecMessageId
                            il canale PEC.
                                                                Formato alfanumerico                            <0.1>         unbounded
                         Identificativo proprio
                         del messaggio PEC
                          Opzionale. Campo
                               contenente
       Note                     eventuali                       Formato alfanumerico                            <0.1>         unbounded
                              informazioni
                               aggiuntive

                                                              Tabella 1
                     La ricevuta è firmata mediante tecnologia XAdES, pertanto presenterà al suo
                     interno, oltre gli elementi XML su indicati, il tag ds:Signature. Esso fa riferimento
                     al namespace: http://www.w3.org/2000/09/xmldsig# .

                                          [versione 1.8.4] 20/03/2026
```

<!-- PDF page 40 -->

```text
                   Figura 1

[versione 1.8.4] 20/03/2026
```

<!-- PDF page 41 -->

```text
          1.3           NOTIFICA DI SCARTO
       È la notifica inviata dal SdI al soggetto trasmittente nei casi in cui non sia stato
       superato uno o più controlli tra quelli effettuati dal SdI sul file ricevuto.
       Per tutti i canali trasmissivi essa è rappresentata da un file XML firmato (firma
       elettronica non qualificata) contenente le seguenti informazioni:

                                                                                                            Obbligatorietà          Dim.
                             Descrizione
  Elemento XML                                                Formati e valori ammessi                            e
                             funzionale
                                                                                                             occorrenze           min-max

                          Numero attribuito
                            dal Sistema di
 IdentificativoSDI
                         Interscambio al file
                                                                  Formato numerico                              <1.1>              12
                               ricevuto

                          Nome attribuito al
                            file secondo le
     NomeFile
                          regole riportate su
                                                                Formato alfanumerico                            <1.1>             1 … 50
                         Disciplinare Tecnico

                          Data e ora in cui il
                                                    Il formato della data è rappresentato secondo il
                         file è stato ricevuto
DataOraRicezione
                             dal Sistema di
                                                  formato ISO 8601:2004, con la seguente precisione:            <1.1>              16
                                                                  YYYY-MM-DD-HH:MM
                             Interscambio
                           Opzionale. Valore
                         presente nel caso di
                              Ricevuta di
RiferimentoArchivio
                          consegna relativa a
                                                                 Campo complesso                                <0.1>
                         fattura appartenente
                             a file archivio.

                           Lista degli errori
    ListaErrori
                               rilevato.
                                                                 Campo complesso                                <1.1>

                           Identificativo del                                                                                1 … 99999999999999
    MessageId
                              messaggio
                                                                  Formato numerico                              <1.1>

                         Opzionale. Presente
                           solo nel caso di
                         messaggi inviati con
  PecMessageId
                            il canale PEC.
                                                                Formato alfanumerico                            <0.1>         unbounded
                         Identificativo proprio
                         del messaggio PEC
                          Opzionale. Campo
                               contenente
       Note                     eventuali                       Formato alfanumerico                            <0.1>         unbounded
                              informazioni
                               aggiuntive

                                                              Tabella 2
                     La notifica è firmata mediante tecnologia XAdES, pertanto presenterà al suo
                     interno, oltre gli elementi XML su indicati, il tag ds:Signature. Esso fa riferimento
                     al namespace: http://www.w3.org/2000/09/xmldsig# .

                                          [versione 1.8.4] 20/03/2026
```

<!-- PDF page 42 -->

```text
                                                     Figura 2

  1.4          NOTIFICA DI MANCATA CONSEGNA
È la notifica inviata dal SdI al soggetto trasmittente nei casi in cui fallisca
l’operazione di consegna del file al destinatario.
Per tutti i canali trasmissivi essa è rappresentata da un file XML firmato (firma
elettronica non qualificata) contenente le seguenti informazioni:

                                  [versione 1.8.4] 20/03/2026
```

<!-- PDF page 43 -->

```text
                                                                                                            Obbligatorietà          Dim.
                             Descrizione
  Elemento XML                                                Formati e valori ammessi                            e
                             funzionale
                                                                                                             occorrenze            min-max

                          Numero attribuito
                            dal Sistema di
 IdentificativoSDI
                         Interscambio al file
                                                                  Formato numerico                              <1.1>               12
                               ricevuto

                          Nome attribuito al
                            file secondo le
     NomeFile
                          regole riportate su
                                                                Formato alfanumerico                            <1.1>             1 … 50
                         Disciplinare Tecnico

                          Data e ora in cui il
                                                    Il formato della data è rappresentato secondo il
                         file è stato ricevuto
DataOraRicezione
                             dal Sistema di
                                                  formato ISO 8601:2004, con la seguente precisione:            <1.1>               16
                                                                  YYYY-MM-DD-HH:MM
                             Interscambio
                           Opzionale. Valore
                         presente nel caso di
                              Ricevuta di
RiferimentoArchivio
                          consegna relativa a
                                                                 Campo complesso                                <0.1>
                         fattura appartenente
                             a file archivio.
                              Opzionale.
                           Descrizione delle
   Descrizione
                             motivazioni di
                                                                Formato alfanumerico                            <0.1>             1 … 255
                          mancata consegna

                           Identificativo del                                                                                1 … 99999999999999
    MessageId
                              messaggio
                                                                  Formato numerico                              <1.1>

                         Opzionale. Presente
                           solo nel caso di
                         messaggi inviati con
  PecMessageId
                            il canale PEC.
                                                                Formato alfanumerico                            <0.1>         unbounded
                         Identificativo proprio
                         del messaggio PEC
                          Opzionale. Campo
                               contenente
       Note                     eventuali                       Formato alfanumerico                            <0.1>         unbounded
                              informazioni
                               aggiuntive

                                                              Tabella 3
                     La notifica è firmata mediante tecnologia XAdES, pertanto presenterà al suo
                     interno, oltre gli elementi XML su indicati, il tag ds:Signature. Esso fa riferimento
                     al namespace: http://www.w3.org/2000/09/xmldsig# .

                                          [versione 1.8.4] 20/03/2026
```

<!-- PDF page 44 -->

```text
                                                     Figura 3

  1.5          NOTIFICA DI ESITO COMMITTENTE
È la notifica inviata dal soggetto ricevente al SdI per comunicare l’esito
(accettazione o rifiuto della fattura) dei controlli effettuati sul documento ricevuto.
Per tutti i canali trasmissivi essa è rappresentata da un file XML che contiene le
seguenti informazioni:

                                  [versione 1.8.4] 20/03/2026
```

<!-- PDF page 45 -->

```text
                                                                                                           Obbligatorietà         Dim.
                             Descrizione
   Elemento XML                                              Formati e valori ammessi                            e
                             funzionale
                                                                                                            occorrenze           min-max

                          Numero attribuito
                            dal Sistema di
  IdentificativoSDI
                           Interscambio al
                                                                 Formato numerico                              <1.1>              12
                             file ricevuto
                               Opzionale.
                          Descrive a quale
                          fattura si riferisce
                             l’esito; se non
 RiferimentoFattura
                              valorizzato si
                                                                 Campo complesso                               <0.1>
                           intende riferito a
                             tutte le fatture
                            presenti nel file

                                                               Formato alfanumerico
                          Esito dei controlli
                          svolti sulla fattura
       Esito
                            da parte del
                                                                   Valori ammessi:                             <1.1>               4
                                                                  [EC01]    (vale Accettazione)
                            destinatario.
                                                                  [EC02]    (vale Rifiuto)

                             Opzionale.
                          Descrizione delle
    Descrizione
                           motivazioni di
                                                               Formato alfanumerico                            <0.1>            1 … 255
                                rifiuto
                          Identificativo del
                             messaggio
MessageIdCommittente
                           assegnato dal
                                                               Formato alfanumerico                            <0.1>            1 … 14
                            committente

                                                            Tabella 4

                  La notifica può essere firmata opzionalmente mediante tecnologia XAdES; in tal
                  caso presenterà al suo interno, oltre gli elementi XML su indicati, il tag
                  ds:Signature.        Esso        fa      riferimento     al       namespace:
                  http://www.w3.org/2000/09/xmldsig# .

                                         [versione 1.8.4] 20/03/2026
```

<!-- PDF page 46 -->

```text
                                                     Figura 4

  1.6          NOTIFICA DI SCARTO ESITO COMMITTENTE
È la notifica inviata dal SdI al soggetto ricevente per comunicare eventuali
incoerenze o errori nell’esito inviato al SdI precedentemente (accettazione o rifiuto
della fattura).
Per tutti i canali trasmissivi essa è rappresentata da un file XML firmato (firma
elettronica non qualificata) contenente le seguenti informazioni:

                                                                                                    Obbligatorietà        Dim.
                               Descrizione
   Elemento XML                                            Formati e valori ammessi                       e
                               funzionale
                                                                                                     occorrenze          min-max

                             Numero attribuito
                               dal Sistema di
  IdentificativoSDI
                            Interscambio al file
                                                               Formato numerico                         <1.1>             12
                                  ricevuto

                           Opzionale. Descrive
 RiferimentoFattura         a quale fattura si                 Campo complesso                          <0.1>
                             riferisce l’esito

                                  [versione 1.8.4] 20/03/2026
```

<!-- PDF page 47 -->

```text
                                                            Formato alfanumerico
                           Motivazione dello
      Scarto
                                scarto
                                                               Valori ammessi:                     <1.1>             4
                                                   [EN00]     (vale Non conforme al formato)
                                                   [EN01]     (vale Non ammissibile)

                            Identificativo del                                                                1 … 99999999999999
    MessageId
                               messaggio
                                                              Formato numerico                     <1.1>

                            Identificativo del
                                messaggio
MessageIdCommittente         assegnato dal
                                                            Formato alfanumerico                   <0.1>            1 … 14
                               committente
                          Opzionale. Presente
                            solo nel caso di
                          messaggi inviati con
  PecMessageId
                             il canale PEC.
                                                            Formato alfanumerico                   <0.1>       unbounded
                          Identificativo proprio
                          del messaggio PEC
                           Opzionale. Campo
                                contenente
                                 eventuali                  Formato alfanumerico                   <0.1>       unbounded
       Note                    informazioni
                                aggiuntive

                                                    Tabella 5

         Il campo <IdentificativoSDI> viene valorizzato a “0” nel caso in cui il Sistema di
         Interscambio non sia stato in grado di associare la Notifica di Esito Committente
         ad alcun file fattura ricevuto.
         La notifica è firmata mediante tecnologia XAdES, pertanto presenterà al suo
         interno, oltre gli elementi XML su indicati, il tag ds:Signature. Esso fa riferimento
         al namespace: http://www.w3.org/2000/09/xmldsig# .

                                 [versione 1.8.4] 20/03/2026
```

<!-- PDF page 48 -->

```text
                                                     Figura 5

  1.7          NOTIFICA DI ESITO (CEDENTE)
È la notifica inviata dal SdI al soggetto trasmittente per comunicare l’esito
(accettazione o rifiuto della fattura) dei controlli effettuati sul documento ricevuto
dal destinatario.
Per tutti i canali trasmissivi essa è rappresentata da un file XML firmato (firma
elettronica non qualificata) contenente le seguenti informazioni:

                                  [versione 1.8.4] 20/03/2026
```

<!-- PDF page 49 -->

```text
                                                                                                           Obbligatorietà          Dim.
                            Descrizione
 Elemento XML                                               Formati e valori ammessi                             e
                            funzionale
                                                                                                            occorrenze           min-max

                         Numero attribuito
                           dal Sistema di
IdentificativoSDI
                        Interscambio al file
                                                                Formato numerico                               <1.1>              12
                              ricevuto

                         Nome del file a cui
   NomeFile
                          si riferisce l’esito
                                                              Formato alfanumerico                             <1.1>             1 … 50

                         Esito da parte del
EsitoCommittente        committente circa la                    Campo complesso                                <1.1>
                          fattura inviata

                          Identificativo del                                                                                1 … 99999999999999
   MessageId
                             messaggio
                                                                Formato numerico                               <1.1>

                        Opzionale. Presente
                          solo nel caso di
                        messaggi inviati con
 PecMessageId
                           il canale PEC.
                                                              Formato alfanumerico                             <0.1>         unbounded
                        Identificativo proprio
                        del messaggio PEC
                         Opzionale. Campo
                              contenente
                               eventuali                      Formato alfanumerico                             <0.1>         unbounded
      Note                   informazioni
                              aggiuntive

                                                            Tabella 6

                    La notifica è firmata mediante tecnologia XAdES, pertanto presenterà al suo
                    interno, oltre gli elementi XML su indicati, il tag ds:Signature. Esso fa riferimento
                    al namespace: http://www.w3.org/2000/09/xmldsig# .

                                         [versione 1.8.4] 20/03/2026
```

<!-- PDF page 50 -->

```text
                                                     Figura 6

  1.8          NOTIFICA DI DECORRENZA TERMINI
È la notifica inviata dal SdI sia al soggetto trasmittente che al soggetto
ricevente per comunicare la decorrenza del termine limite per la comunicazione
dell’accettazione/rifiuto.
Per tutti i canali trasmissivi essa è rappresentata da un file XML firmato (firma
elettronica non qualificata) contenente le seguenti informazioni:

                                  [versione 1.8.4] 20/03/2026
```

<!-- PDF page 51 -->

```text
                                                                                                            Obbligatorietà          Dim.
                             Descrizione
  Elemento XML                                               Formati e valori ammessi                             e
                             funzionale
                                                                                                             occorrenze            min-max

                          Numero attribuito
                            dal Sistema di
 IdentificativoSDI
                         Interscambio al file
                                                                 Formato numerico                               <1.1>               12
                               ricevuto

                         Opzionale. Descrive
RiferimentoFattura        a quale fattura si                     Campo complesso                                <0.1>
                           riferisce l’esito

                          Nome del file a cui
    NomeFile
                         si riferisce la notifica
                                                               Formato alfanumerico                             <1.1>             1 … 50

                            Opzionale.
                          Descrizione delle
   Descrizione
                           motivazioni di
                                                               Formato alfanumerico                             <0.1>             1 … 255
                              notifica

                           Identificativo del                                                                                1 … 99999999999999
   MessageId
                              messaggio
                                                                 Formato numerico                               <1.1>

                         Opzionale. Presente
                           solo nel caso di
                         messaggi inviati con
  PecMessageId
                            il canale PEC.
                                                               Formato alfanumerico                             <0.1>         unbounded
                         Identificativo proprio
                         del messaggio PEC
                          Opzionale. Campo
                               contenente
       Note                     eventuali                      Formato alfanumerico                             <0.1>         unbounded
                              informazioni
                               aggiuntive

                                                             Tabella 7

                     La notifica è firmata mediante tecnologia XAdES, pertanto presenterà al suo
                     interno, oltre gli elementi XML su indicati, il tag ds:Signature. Esso fa riferimento
                     al namespace: http://www.w3.org/2000/09/xmldsig# .

                                          [versione 1.8.4] 20/03/2026
```

<!-- PDF page 52 -->

```text
                                                     Figura 7

  1.9          NOTIFICA METADATI DEL FILE FATTURA AL DESTINATARIO
È il file inviato dal SdI al soggetto ricevente insieme al file fattura e contenente i
dati principali di riferimento del file utili per l’elaborazione, ivi compreso
l’identificativo del SdI.

                                  [versione 1.8.4] 20/03/2026
```

<!-- PDF page 53 -->

```text
                                                                                                            Obbligatorietà          Dim.
                            Descrizione
  Elemento XML                                               Formati e valori ammessi                             e
                            funzionale
                                                                                                             occorrenze           min-max

                         Numero attribuito
                           dal Sistema di
 IdentificativoSDI
                        Interscambio al file
                                                                 Formato numerico                               <1.1>              12
                              ricevuto

                        Nome del file a cui
    NomeFile
                       si riferisce la notifica
                                                               Formato alfanumerico                             <1.1>             1 … 50

                        Codice del soggetto
CodiceDestinatario      a cui è indirizzata la                 Formato alfanumerico                             <1,1>             1…7
                               fattura

                        Identificativo della
     Formato           versione del formato                    Formato alfanumerico                             <1,1>               5
                              fattura

                       Numero progressivo
                       che identifica l’invio,
                       normalmente vale 1,
  TentativiInvio          nel caso di più                        Formato numerico                               <1.1>               1
                             tentativi il
                        progressivo viene
                           incrementato

                          Identificativo del                                                                                 1 … 99999999999999
   MessageId
                             messaggio
                                                                 Formato numerico                               <1.1>

                        Opzionale. Campo
                           contenente
       Note                 eventuali                          Formato alfanumerico                             <0.1>         unbounded
                          informazioni
                           aggiuntive

                                                             Tabella 8

                                          [versione 1.8.4] 20/03/2026
```

<!-- PDF page 54 -->

```text
                                                     Figura 8

  1.10         ATTESTAZIONE DI AVVENUTA TRASMISSIONE DELLA FATTURA CON IMPOSSIBILITÀ
               DI RECAPITO

           È la notifica inviata dal SdI al soggetto trasmittente per attestare l’avvenuta
           ricezione della fattura e l’impossibilità di recapitare il file al destinatario; la
           casistica si riferisce alla mancata disponibilità tecnica di comunicazione con il
           destinatario.
Per tutti i canali trasmissivi essa è rappresentata da un file .zip contenente:
              • il file originale così come ricevuto dal mittente
              • un file xml, firmato (firma elettronica non qualificata), contenente le
                    seguenti informazioni:

                                  [versione 1.8.4] 20/03/2026
```

<!-- PDF page 55 -->

```text
                                                                                                           Obbligatorietà          Dim.
                          Descrizione
 Elemento XML                                                Formati e valori ammessi                            e
                          funzionale
                                                                                                            occorrenze           min-max

                        Numero attribuito
                          dal Sistema di
IdentificativoSdI
                       Interscambio al file
                                                                 Formato numerico                              <1.1>              12
                             ricevuto

                       Nome del file a cui
    NomeFile
                      si riferisce la notifica
                                                               Formato alfanumerico                            <1.1>             1 … 50

                        Data e ora in cui il
                                                   Il formato della data è rappresentato secondo il
                       file è stato ricevuto
DataOraRicezione
                           dal Sistema di
                                                 formato ISO 8601:2004, con la seguente precisione:            <1.1>              16
                                                                 YYYY-MM-DD-HH:MM
                           Interscambio
                        Opzionale. Valore
                      presente nel caso di
RiferimentoArchivio      notifica relativa a                    Campo complesso                                <0.1>
                      fattura appartenente
                            a file archivio.
                       Blocco contenente
                           le informazioni
                       relative al soggetto
                         destinatario della
                         fattura (Codice e
  Destinatario           Denominazione;                         Campo complesso                                <1.1>
                          quest’ultima,per
                        soggetti diversi da
                              pubbliche
                         amministrazioni,
                            vale “NO PA”)

                        Identificativo del                                                                                  1 … 99999999999999
   MessageId
                           messaggio
                                                                 Formato numerico                              <1.1>

                      Opzionale. Presente
                          solo nel caso di
                      messaggi inviati con
 PecMessageId
                           il canale PEC.
                                                               Formato alfanumerico                            <0.1>         unbounded
                      Identificativo proprio
                      del messaggio PEC
                       Opzionale. Campo
                              contenente
      Note                     eventuali                       Formato alfanumerico                            <0.1>         unbounded
                             informazioni
                              aggiuntive
                            Obbligatorio.
                       Campo contenente
HashFileOriginale
                        l’hash (SHA-256)
                                                               Formato alfanumerico                            <1.1>         unbounded
                          del file ricevuto

                                                             Tabella 9

                                         [versione 1.8.4] 20/03/2026
```

<!-- PDF page 56 -->

```text
L’attestazione è firmata mediante tecnologia XAdES, pertanto presenterà al suo
interno, oltre gli elementi XML su indicati, il tag ds:Signature. Esso fa riferimento
al namespace: http://www.w3.org/2000/09/xmldsig# .

                                           Figura 9

                        [versione 1.8.4] 20/03/2026
```
