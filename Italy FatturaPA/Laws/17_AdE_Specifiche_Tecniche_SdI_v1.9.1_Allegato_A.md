# Allegato A — Specifiche tecniche, versione 1.9.1 (Agenzia delle entrate, Sistema di Interscambio: fatturazione elettronica e dati fatture transfrontaliere)

**Publication:** agenziaentrate.gov.it; version 1.9.1 dated 31-03-2026, **usable from 2026-05-15**  
**Source:** https://www.agenziaentrate.gov.it/portale/documents/d/guest/allegato-a-specifiche-tecniche-vers-1-9-1  
**Page:** https://www.agenziaentrate.gov.it/portale/specifiche-tecniche-versione-1.9.1-%C2%A0-utilizzabili-dal-15-maggio-2026-  
**Retrieved:** 2026-10-29  
**Text:** Text layer of the published PDF (189 pages) via `pdftotext -layout`, one fenced block per page so indentation and tables survive; only the bare page-number line is removed. Read the PDF for figures.  
**Note:** The **SdI specifiche tecniche** = Allegato A of provvedimento 433608/2022 (file 14, point 1.3). Not the same document as the **FatturaPA format specification V1.4** (fatturapa.gov.it, valid from 2025-04-01, held as `../Specifiche_tecniche_del_formato_FatturaPA_V1.4.pdf`), nor the fatturapa.gov.it *Specifiche tecniche relative al Sistema di Interscambio* v1.8.4 (file 21). Change log ("Stato del documento"): 1.9 of 31-01-2025 introduced TD29 and RF20 (XSD changes); 1.9.1 of 31-03-2026 adds check 00327, updates WS/SFTP accreditation, adds an AltriDatiGestionali coding for sports workers' exempt income and raises the maximum number of codici destinatario — no XSD change (schemas still FPR12 v1.2.3 / VFSM10 v1.0.2).

<!-- PDF page 1 -->

```text
ALLEGATO A - SPECIFICHE TECNICHE

            Versione 1.9.1
```

<!-- PDF page 2 -->

```text
                                         INDICE

STATO DEL DOCUMENTO                                                               5

1. FATTURAZIONE ELETTRONICA – REGOLE DI PROCESSO                              10

  1.1     DEFINIZIONI                                                         10

  1.2     MODALITÀ DI EMISSIONE DELLE FATTURE ELETTRONICHE                    11
  1.2.1   Formati di firma elettronica e riferimento temporale                11
  1.2.2   Nomenclatura dei file da trasmettere                                12

  1.3     MODALITÀ DI TRASMISSIONE DELLE FATTURE E DELLE RICEVUTE             14
  1.3.1   Trasmissione del file al SdI                                        14
  1.3.2   Trasmissione del file al soggetto ricevente                         18

  1.4     MODALITÀ DI INOLTRO DELLE COMUNICAZIONI DA PARTE DEL SDI            22

  1.5     MODALITÀ DI INTERAZIONE CON IL SDI                                  22
  1.5.1   Interazione del soggetto ricevente con il SdI                       23
  1.5.2   Procedura per l’identificazione del soggetto trasmittente ai fini
          dell’interazione con il SdI                                         23
  1.5.3   Procedura per l’accreditamento dei canali web service e sftp        24
  1.5.4   Procedura di invio fattura al SdI                                   25
  1.5.5   Procedura di inoltro della fattura al soggetto ricevente            26
  1.5.6   Flusso semplificato                                                 26
  1.5.7   Procedura di gestione delle ricevute                                27

  1.6     CONTROLLI EFFETTUATI DAL SDI                                        29

  1.7     TEMPI DI ELABORAZIONE DA PARTE DEL SDI                              29

2. FATTURAZIONE ELETTRONICA – FORMATO DELLA FATTURA                           31

  2.1     PARTE PRIMA                                                         32
  2.1.1   FATTURA ELETTRONICA HEADER – Dati di trasmissione                   32
  2.1.2   FATTURA ELETTRONICA HEADER – Dati del cedente/prestatore            34
  2.1.3   FATTURA ELETTRONICA HEADER – Dati del rappresentante fiscale del
          cedente/prestatore                                             38
```

<!-- PDF page 3 -->

```text
   2.1.4    FATTURA ELETTRONICA HEADER – Dati del cessionario/committente 39
   2.1.5    FATTURA ELETTRONICA HEADER – Dati del terzo intermediario
            soggetto emittente                                                            42
   2.1.6    FATTURA ELETTRONICA HEADER – Soggetto emittente                               43
   2.1.7    FATTURA ELETTRONICA BODY – Dati generali                                      43
   2.1.8    FATTURA ELETTRONICA BODY – Dati beni / servizi                                51
   2.1.9    FATTURA ELETTRONICA BODY – Dati sui veicoli                                   59
   2.1.10   FATTURA ELETTRONICA BODY – Dati di pagamento                                  59
   2.1.11   FATTURA ELETTRONICA BODY – Allegati                                           61

   2.2      PARTE SECONDA                                                                 61
   2.2.1    Schema del file XML                                                           61
   2.2.2    L’elemento radice “fattura elettronica”                                       62
   2.2.3    Fattura elettronica header: dati trasmissione                                 64
   2.2.4    Fattura elettronica header: cedente / prestatore                              65
   2.2.5    Fattura elettronica header: rappresentante fiscale del cedente / prestatore
                                                                                          73
   2.2.6    Fattura elettronica header: cessionario / committente                         74
   2.2.7    Fattura elettronica header: terzo intermediario soggetto emittente            77
   2.2.8    Fattura elettronica header: soggetto emittente                                78
   2.2.9    Fattura elettronica body: dati generali                                       79
   2.2.10   Fattura elettronica body: elemento dati dei beni/servizi                      92
   2.2.11   Fattura elettronica body: dati sui veicoli                                    99
   2.2.12   Fattura elettronica body: dati di pagamento                               100
   2.2.13   Fattura elettronica body: allegati                                        103

3. OPERAZIONI VERSO E DA SOGGETTI NON STABILITI IN ITALIA – REGOLE DI
   PROCESSO                                                         105

   3.1      MODALITÀ DI PREDISPOSIZIONE DEI FILE PER LA TRASMISSIONE DELLE
            OPERAZIONI ATTIVE                                                         105

   3.2      MODALITÀ DI PREDISPOSIZIONE DEI FILE PER LA TRASMISSIONE DELLE
            OPERAZIONI PASSIVE                                                        105
   3.2.1    Integrazione/autofattura per acquisto servizi dall'estero - TD17          106
   3.2.2    Integrazione per acquisto di beni intracomunitari - TD18                  108

   3.2.3    Integrazione/autofattura per acquisto di beni ex art. 17 c.2 d.P.R. 633/72 -
```

<!-- PDF page 4 -->

```text
             TD19                                                               109
   3.2.4     Acquisti da San Marino con IVA (fattura cartacea) – TD28           112

INDICAZIONE DI IMPONIBILE E IMPOSTA, COME INDICATO NELLA FATTURA
   CARTACEA RICEVUTA.APPENDICE 1: ELENCO DEI CODICI ERRORE RELATIVI
   ALLA FATTURA ORDINARIA E A QUELLA SEMPLIFICATA                114

   1. TIPOLOGIE E MODALITÀ DI VERIFICA                                          114

APPENDICE 2: FATTURA ORDINARIA                                                  126

   1. SCHEMA XSD FATTURA ORDINARIA                                              126

   1.        RAPPRESENTAZIONE TABELLARE FATTURA ORDINARIA                       154

APPENDICE 3: FATTURA SEMPLIFICATA                                               162

   1.        SCHEMA XSD FATTURA SEMPLIFICATA                                    162

   1.        RAPPRESENTAZIONE TABELLARE FATTURA SEMPLIFICATA                    173

APPENDICE 4: RICEVUTE DEL PROCESSO DI FATTURAZIONE                              177

1. DESCRIZIONE E REGOLE DI COMPILAZIONE                                         178

   1.1       NOMENCLATURA DEI FILE PER LA TRASMISSIONE DI RICEVUTE/ NOTIFICHE   178
   1.1.1     Ricevuta di consegna del file al destinatario                      178
   1.1.2     Ricevuta di scarto                                                 182
   1.1.3     Ricevuta di impossibilità di recapito                              184
   1.1.4     Metadati del file fattura al destinatario                          186

   1.2       SCHEMA XSD MESSAGGIFATTURATYPES_V1.0                               188
```

<!-- PDF page 5 -->

```text
      STATO DEL DOCUMENTO

Versione Data               Lista modifiche

1.1      05/06/2018           1. Correzione di refusi
                              2. Chiarimenti su aspetti di dettaglio. Tra questi si è
                                 esplicitato che il nome dei file fattura e dati fattura può
                                 contenere identificativi di soggetti diversi dal
                                 trasmittente
                              3. Evidenza del limite dimensionale per singolo file fattura
                                 che non può eccedere i 5MB
                              4. Evidenza del comportamento del sistema nei casi in
                                 cui non riesca a consegnare una ricevuta per
                                 indisponibilità del canale del trasmittente
                              5. Istruzioni sull’utilizzo degli elementi facoltativi racchiusi
                                 nel blocco informativo <AltriDatiGestionali> nei casi in
                                 cui:
                                    • siano fatturate operazioni di rifornimenti di
                                        carburante e si intenda riportare in fattura la
                                        targa del mezzo rifornito
                                    • siano riportate in fattura operazioni già
                                        documentate da scontrino fiscale o documento
                                        commerciale
                              6. L’Appendice 1 è stata arricchita con le indicazioni, per
                                 ciascun errore, del fatto che l’errore possa presentarsi
                                 per la fattura ordinaria o per la semplificata o per
                                 entrambe
                              7. Sono stati introdotti gli schemi XSD e le
                                 rappresentazioni tabellari della fattura ordinaria e della
                                 fattura semplificata

1.2      03/10/2018         Introdotto un nuovo controllo per le fatture semplificate al fine
                            di verificare che l’importo totale non ecceda, per questa
                            tipologia di fatture, il limite previsto dall’art. 21 bis del DPR
                            633/72 (100 euro). Il relativo codice di errore è 00460 (pag.
                            141)

1.3      10/10/2018         Indicato esplicitamente l’indirizzo PEC verso il quale
                            effettuare la prima trasmissione di fatture al SdI (pag. 12)

1.4      26/02/2019         Sono indicati, per ciascun canale, i fattori non dipendenti
                            dallo Sdi che incidono sui tempi di consegna delle fatture ai
                            destinatari e delle notifiche ai mittenti.
                            È     introdotta    una      regola     per    ottimizzare   la
                            trasmissione/ricezione attraverso il servizio SDIFTP (efficace
                            del prossimo 4 8 aprile)
```

<!-- PDF page 6 -->

```text
                     Sono dettagliati i tempi e i tentativi di consegna delle fatture
                     al destinatario, per il servizio SDICOOP e di trasmissione
                     delle ricevute al mittente per il servizio SDICOOP e per la
                     PEC
1.4.1   29/03/2019   Le modifiche introdotte con la versione 1.4 saranno efficaci
                     dal prossimo 8 aprile.
                     Chiarite conseguenze superamento dimensione massima
                     per supporti SFTP (pag.14).
                     Esplicitato l’invio dell’ER (pag.14)
                     Esplicitate conseguenze fallimento consegna PEC e WS
                     (pag. 16)
                     Esplicitata modalità di consegna fattura con il canale SDIFTP
                     (pag. 17).
                     Esplicitato il momento a partire dal quale la fattura si
                     considera consegnata al SdI, ai fini della determinazione dei
                     giorni entro i quali deve essere inviata l’eventuale ricevuta di
                     scarto (pag. 25).
                     È stata eliminata la segnalazione via email, nel caso di
                     mancato rispetto della regola sul controllo della eccessiva
                     parcellizzazione dei supporti SFTP (era presente a pag. 16
                     della versione 1.4)

1.4.2   20/05/2019   Modificato il controllo per le fatture semplificate (codice
                     00460) con innalzamento del limite dell’importo totale da 100
                     a 400 euro (pag. 144).
                     Modificata la politica di retry nel caso di invio fatture da SdI
                     verso canale SDICoop: i tentativi passano da 3 a 4 con
                     intervallo di 6 ore l’uno dall’altro (pag. 18)

1.5     01/07/2019   Introdotti nuovi controlli per:
                          - verificare il corretto utilizzo del codice destinatario
                              “XXXXXXX” (codice di errore 00313)
                          - verificare la coerenza di partita IVA e codice fiscale
                              (se entrambi presenti) sia per il cedente/prestatore
                              che per il cessionario/committente (codici di errore
                              00320 e 00324)
                          - verificare la corretta valorizzazione del codice
                              fiscale, sia per il cedente/prestatore che per il
                              cessionario/committente, nei casi in cui la partita IVA
                              sia quella di un gruppo IVA (codici di errore 00321,
                              00322, 00325 e 00326)
                          - verificare, nei casi di autofattura, che la partita IVA
                              del cedente/prestatore non sia cessata da più di 5
                              anni (codice di errore 00323)
                          - verificare che l’indirizzo PEC indicato nel campo
                              PECDestinatario non corrisponda ad una casella
                              PEC del SdI (codice di errore 00330)
                     (paragrafi modificati: 2.1.1 - 2.1.2 - 2.1.4 - Appendice 1)

1.6     04/05/2020   Variato lo schema XSD della fattura ordinaria con:
                         - introduzione del nuovo attribute “SistemaEmittente”
                         - modifica del numero di occorrenze di DatiRitenuta
```

<!-- PDF page 7 -->

```text
                         -    modifica        del       type      di     Importo     in
                              ScontoMaggiorazioneType
                          - modifica              della          enumeration         di
                              CausalePagamentoType,               TipoDocumentoType,
                              TipoRitenutaType,                 NaturaType            e
                              ModalitaPagamentoType
                          - modificato il campo complesso DatiBollo con
                              introduzione       dell’opzionalità      per   l’elemento
                              ImportoBollo non più obbligatorio
                          - introduzione del tipo String35LatinExtType
                          - modifica della definizione di EmailType.
                     Variato lo schema XSD della fattura semplificata con:
                          - introduzione del nuovo attribute “SistemaEmittente”
                          - introduzione del campo “BolloVirtuale” e del type
                              BolloVirtualeType
                          - modifica della enumeration di NaturaType
                          - modifica della definizione di EmailType.
                     Variato lo schema XSD dati fatture transfrontaliere con:
                          - modifica della enumeration di NaturaType.
                     Modificata la descrizione dell’errore 00420 sulle fatture.
                     Introdotti nuovi controlli sulle fatture con codici 00443,
                     00444, 00445, 00471, 00472, 00473, 00474
                     Modificati i criteri di controllo per gli errori 00323, 00404 e
                     00409 sulle fatture.
                     Modificata la descrizione dell’errore 00400 sui dati fattura.
                     Introdotti nuovi controlli sui dati fattura con codici 00321,
                     00325 e 00448.

                     (paragrafi modificati: 2.1.2 - 2.1.7 – 2.1.8 – 2.2.9.1 – 2.2.10.1
                     – 2.2.10.2 – 2.2.12 - 4.2.1.2.2 – 4.2.1.3.2 – Appendice 1 –
                     Appendice 2 – Appendice 3 – Appendice 4 – Appendice 5)
1.6.1   01/10/2020   Indicata la data di fine validità per i codici Natura N2, N3 e
                     N6 (paragrafi 2.2 e 4.2)
                     Modificata la data di entrata in vigore dei controlli con codice
                     00445 (Appendice 1) e 00448 (Appendice 2)
1.6.2   23/11/2020   Corretti alcuni refusi
                     Modificato il criterio di controllo per gli errori 00404 e 00409,
                     con eliminazione della casistica SoggettoEmittente = ‘CC’.
                     Reintrodotta (per le fatture emesse fino al 31/12/2020) la
                     possibilità di utilizzare il codice “Z” come CausalePagamento

1.6.3   02/07/2021   Corretti alcuni refusi
                     Aggiornata la descrizione per il codice natura N6.2 e i tipi
                     documento TD24 e TD25
                     Aggiunte nuove codifiche per TipoDato (INVCONT, NB1,
                     NB2, NB3, N.DOC.COMM)
                     Modificati i criteri dei controlli 00401 e 00460 sui dati fattura.
                     Modificato il criterio di controllo per gli errori 00404 e 00409
                     per i tipi documento TD16, TD17, TD18, TD19, TD20, TD22,
                     TD23
```

<!-- PDF page 8 -->

```text
1.6.4   01/01/2022   Introdotte nuove codifiche per il blocco AltriDatiGestionali per
                     operazioni non imponibili nei confronti di un esportatore
                     abituale
                     Introdotto nuovo controllo sulla fattura con codice 00475
                     Aggiornato il controllo 00313 per le fatture semplificate

1.7     01/07/2022   Adeguamento specifiche tecniche alle nuove modalità di
                     comunicazione delle operazioni verso e da soggetti non
                     stabiliti in Italia dal 1° luglio 2022, attraverso l’utilizzo del
                     tracciato della fatturazione elettronica ordinaria ed
                     eliminazione delle indicazioni per l’invio dei dati delle
                     operazioni transfrontaliere (esterometro) utilizzando il
                     tracciato dati fattura

1.7.1   01/10/2022   Variato lo schema XSD della fattura ordinaria per
                     l’introduzione di un nuovo tipo documento TD28 per
                     comunicare operazioni di acquisto di beni da San Marino per
                     le quali sono state ricevute fatture cartacee con indicazione
                     dell’IVA
                     Modificato il criterio di controllo per l’errore 00471 per i tipi
                     documento TD01, TD02,TD03,TD06,TD24,TD25 e TD28
                     (fattura ordinaria) e TD07 (fattura semplificata)
                     Modificato il criterio di controllo per l’errore 00472 per i tipi
                     documento TD27
                     Aggiornata la descrizione della natura N7
                     Introdotte nuove codifiche per il blocco AltriDatiGestionali per
                     riportare in fattura l’informazione circa l’avvenuta
                     regolarizzazione dell’imposta a debito con versamento
                     tramite modello di versamento F24 e per riportare in fattura
                     il riferimento al corretto periodo di imposta dell’operazione
                     nel caso di operazioni di estrazione beni da deposito IVA
                     Introdotto nuovo controllo sulla fattura con codice 00476
                     Aggiornata la descrizione dei codici di errore 00401, 00430,
                     00473, 00475
                     Correzione refusi Cap. 3
1.8     12/12/2023   Introdotta nuova codifica per il blocco AltriDatiGestionali per
                     produttori agricoli in regime speciale
                     Introdotto controllo per lo scarto di fatture con dichiarazione
                     d'intento invalidata 00477
                     Aggiornate le indicazioni per l’utilizzo del TD28 per
                     operazioni verso e da soggetti non stabiliti in Italia
1.9     31/01/2025   Variato lo schema XSD della fattura ordinaria per
                     l’introduzione di un nuovo tipo documento TD29 per la
                     comunicazione all’Agenzia delle entrate della omessa o
                     irregolare fatturazione
                     Variato lo schema XSD della fattura ordinaria e semplificata
                     per l’introduzione del nuovo regime transfrontaliero di
                     Franchigia IVA RF20 (Direttiva UE 2020/285)
                     Aggiornata la descrizione del tipo documento TD20
                     Aggiornati i codici valori per le fatture di vendita gasolio o
                     carburante, in accordo alla nuova codifica prevista dall’ADM
                     nella tabella TA13 (Tabella prodotti energetici)
                     Aggiornate descrizioni codici errore 00471, 00473 e 00475.
```

<!-- PDF page 9 -->

```text
                     Modificato il criterio di verifica per i codici errore 00404 e
                     00409
                     Modificato il controllo codice 00460 per l’eliminazione del
                     limite a 400 euro dell’importo totale della fattura semplificata
                     nel caso in cui il Cedente/Prestatore emetta in regime
                     forfettario (art. 1, c. 54-89, L. 190/2014) o in regime
                     transfrontaliero di franchigia IVA (Direttiva UE 2020/285)
1.9.1   31/03/2026   Introdotto nuovo controllo sulla fattura con codice 00327
                     Aggiornamento procedure di accreditamento dei canali WS
                     e SFTP
                     Introdotta nuova codifica per il blocco AltriDatiGestionali per
                     il reddito fatturato in esenzione da lavoratori sportivi
                     Aggiornato il numero massimo di codici destinatario
                     disponibili
```

<!-- PDF page 10 -->

```text
1.    FATTURAZIONE ELETTRONICA – REGOLE DI PROCESSO
      La presente sezione del documento riporta le specifiche tecniche per il
      colloquio con il Sistema di Interscambio nell’ambito del processo di
      fatturazione elettronica fra soggetti privati. Tali specifiche sono derivate
      dalle Specifiche delle regole tecniche di cui all’allegato B del DM 55 del
      3 aprile 2013; a tale documento si rimanda per le regole tecniche di
      colloquio con il SdI in caso di fatturazione elettronica nei confronti della
      Pubblica Amministrazione.

1.1   DEFINIZIONI
      Ai fini del presente documento si intende:
      - per SdI, Sistema di Interscambio, vale a dire la struttura istituita dal
           Ministero dell’Economia e delle Finanze attraverso la quale avviene la
           trasmissione delle fatture elettroniche verso l’Amministrazione (art.1,
           comma 211, legge 24 dicembre 2007 n. 244) o verso privati (art. 1,
           comma 2, decreto legislativo 127/2015);
      - per Firma elettronica Entrate, la firma elettronica che garantisce la
           connessione univoca al firmatario e la sua univoca identificazione,
           basata su un certificato emesso dall’Agenzia delle Entrate;
      - per SFTP (Secure File Transfer Protocol), il protocollo di trasferimento
           dati tra sistemi remoti, con connessione sicura;
      - per HTTPS (HyperText Transfer Protocol Secured), il protocollo di
           trasmissione dati su web con ulteriore livello di crittografia ed
           autenticazione dei dati trasmessi (SSL - Secure Sockets Layer);
      - per Intermediario, qualsiasi soggetto terzo, incaricato dal
           cedente/prestatore o dal cessionario/committente di emettere o
           trasmettere o ricevere per proprio conto le fatture elettroniche veicolate
           dal SdI;
      - per Messaggio SOAP, messaggio XML, strutturato in un header e in un
           body, utilizzato nel colloquio tra web services;
      - per Soggetto emittente, il cedente/prestatore o l’Intermediario da questi
           per proprio conto incaricato all’emissione della fattura elettronica;
      - per Soggetto ricevente, il cessionario/committente o l’Intermediario da
           questi per proprio conto incaricato di ricevere la fattura elettronica
           veicolata dal SdI;
      - per Soggetto trasmittente, il cedente/prestatore o l’Intermediario da
           questi per proprio conto incaricato di trasmettere la fattura elettronica al
           SdI;
      - per WSDL (Web Service Definition Language), il linguaggio basato su
           XML per definire un web service e descriverne le modalità di accesso;
```

<!-- PDF page 11 -->

```text
      -     per XML (Extensible Markup Language), l’insieme di regole per
            strutturare in formato testo i dati oggetto di elaborazione.
      -     per Sistema di Accreditamento: il sistema che permette agli utenti di
            effettuare l’accreditamento al SdI tramite una procedura dedicata.
      -     per Accordo di servizio: il documento, generato sul Sistema di
            Accreditamento, che contiene le regole relative al flusso telematico fra
            il soggetto trasmittente e/o ricevente ed il SdI.

1.2   MODALITÀ DI EMISSIONE DELLE FATTURE ELETTRONICHE
      Il presente paragrafo descrive le specifiche tecniche di emissione della
      fattura elettronica, con riferimento al formato di rappresentazione e alle
      caratteristiche del documento da trasmettere.

      I dati della fattura elettronica da trasmettere attraverso il SdI devono essere
      rappresentati in formato XML (eXtensible Markup Language), secondo lo
      schema e le regole riportate al paragrafo 2 Fatturazione elettronica –
      Formato della fattura.

          1.2.1   FORMATI DI FIRMA ELETTRONICA E RIFERIMENTO TEMPORALE
      Il SdI gestisce sia fatture elettroniche prive di firma elettronica che fatture
      elettroniche alle quali sia apposta firma elettronica.
      Nel caso si scelga di apporre la firma elettronica, il SdI verifica che le fatture
      elettroniche siano:
       - firmate digitalmente con certificato di firma elettronica qualificata
          rilasciato da un certificatore accreditato, presente nell’elenco pubblico
          dei certificatori gestito dall’Agenzia per l’Italia Digitale così come
          disciplinato dall’art. 29, comma 1, del DLGS 7 marzo 2005 n. 82 e
          successive modifiche. I formati ammessi per firmare elettronicamente la
          fattura sono i seguenti:
                o CAdES Baseline B (CMS Advanced Electronic Signatures),
                   di seguito CAdES, come previsto dalla normativa europea
                   vigente in materia di firma elettronica avanzata;
                o XAdES Baseline B (XML Advanced Electronic Signatures), di
                   seguito XAdES, come previsto dalla normativa europea vigente
                   in      materia       di    firma       elettronica      avanzata;

      -     firmate in formato CAdES con certificato di firma elettronica rilasciato
            dall’Agenzia delle Entrate. Il certificato è rilasciato agli utenti del Servizio
            Entratel e utilizzato sui propri sistemi ovvero sul proprio PC attraverso
            Desktop telematico o Entratel Multifile. Si ricorda che l’Agenzia delle
            Entrate non è una Certification Authority qualificata e che, pertanto, la
            verifica di una firma elettronica basata sulle chiavi pubbliche di firma e
            cifratura rilasciate dall’Agenzia delle Entrate evidenzierebbe che la CA
            non è affidabile. È quindi opportuno che chi appone tale tipo di firma
            elettronica sulle fatture che emette renda consapevoli i propri clienti di
            questa circostanza e del fatto che la verifica di questo tipo di firma può
            essere fatta solo utilizzando il servizio di verifica di Agenzia delle Entrate
```

<!-- PDF page 12 -->

```text
                  (https://telematici.agenziaentrate.gov.it/Abilitazione/IVerificaFile.jsp).

            Nell’ambito del formato di firma XML l’unica modalità accettata è quella
            “enveloped”.
            Come riferimento temporale di controllo della firma il SdI intende la
            valorizzazione dell’attributo “signing time” che deve essere presente nella
            firma elettronica apposta sul documento e non può essere successivo al
            momento di ricezione.
            L’esito negativo di tale verifica determina lo scarto del file1.

              1.2.2         NOMENCLATURA DEI FILE DA TRASMETTERE
            Le fatture elettroniche possono essere trasmesse al SdI come di seguito
            descritto:
                a) un file in formato xml che rispetti le regole descritte al paragrafo 2
                   Fatturazione elettronica – Formato della fattura;
                b) più file che rispettino quanto descritto al precedente punto a)
                   contenuti in un file compresso; il formato di compressione accettato
                   è il formato ZIP.
            Nel caso a) il sistema verifica che il nome del file rispetti la seguente
            sintassi:

                                        codice              identificativo univoco                  progressivo
                                        paese                                                      univoco del file

            dove:
            - il codice paese va espresso secondo lo standard ISO 3166-1 alpha-2
               code;
            - l’identificativo univoco, è rappresentato dall’identificativo fiscale (codice
               fiscale nel caso di soggetto residente in Italia, identificativo proprio del
               paese di appartenenza nel caso di soggetto residente all’estero) di un
               soggetto persona fisica o persona giuridica diversa da persona fisica; la
               lunghezza di questo identificativo è di:
            - 11 caratteri (minimo) e 16 caratteri (massimo) nel caso di codice paese
               IT;

1 Codici di errore: 00100 (certificato di firma scaduto), 00101 (certificato di firma revocato), 00104 (Certification Authority non
affidabile), 00107 (certificato non valido), 00103 (la firma digitale apposta manca del riferimento temporale), 00105 (il
riferimento temporale della firma apposta non è coerente)
```

<!-- PDF page 13 -->

```text
             -    2 caratteri (minimo) e 28 caratteri (massimo) altrimenti;
             -    l’identificativo usato per il nome del file non è soggetto a controlli di
                  validità, esistenza o coerenza con i dati presenti in fattura.
             -    il progressivo univoco del file è rappresentato da una stringa
                  alfanumerica di lunghezza massima di 5 caratteri e con valori ammessi
                  [a-z], [A-Z], [0-9].

            Il progressivo univoco del file ha il solo scopo di differenziare il nome dei file
            trasmessi al Sistema di Interscambio da parte del medesimo soggetto; non
            deve necessariamente seguire una stretta progressività e può presentare
            anche stili di numerazione differenti.
            Qualora il file sia firmato elettronicamente (come indicato al precedente
            paragrafo 1.2.1), in base al formato di firma elettronica adottato l’estensione
            del file assume il valore “.xml” (per la firma XAdES) oppure “.xml.p7m” (per
            la firma CAdES).
            Il separatore tra il secondo ed il terzo elemento del nome file è il carattere
            underscore (“_”), codice ASCII 95.

                    Es.: ITAAABBB99T99X999W_00001.xml
                         IT99999999999_00002.xml.p7m

            Nel caso b) il nome del file deve rispettare la medesima sintassi; in questo
            caso l’unica estensione ammessa è .zip.
            Se si sceglie di utilizzare una firma elettronica, non è il file compresso (.zip)
            che deve essere firmato digitalmente, ma ogni singolo file in esso contenuto.

                     Es.: ITAAABBB99T99X999W_00001. zip

            che al suo interno contiene, a titolo di esempio

                               ITAAABBB99T99X999W_00002.xml
                               ITAAABBB99T99X999W_00003.xml
                               ITAAABBB99T99X999W_00004.xml.p7m

            Il nome dei file, espressi secondo le regole descritte, viene mantenuto nella
            fase di inoltro al soggetto ricevente. Qualora al SdI sia inviato un file del tipo
            indicato nel caso b) (file compresso), il SdI trasmette al soggetto ricevente
            il file o i file in esso contenuti e non il file compresso.
            Un file inviato al SdI il cui nome non rispetti le regole sopra descritte o sia
            uguale al nome di un file trasmesso in precedenza, sarà scartato 2.

2 Codici di errore: 00001 (nome file non valido), 00002 (nome file duplicato)
```

<!-- PDF page 14 -->

```text
1.3   MODALITÀ DI TRASMISSIONE DELLE FATTURE E DELLE RICEVUTE
      Il presente paragrafo descrive le specifiche tecniche per l’utilizzo dei canali
      di trasmissione delle fatture elettroniche e delle relative ricevute.

       1.3.1    TRASMISSIONE DEL FILE AL SDI
      La trasmissione dei file verso il SdI può essere effettuata utilizzando le
      seguenti modalità:
      - un sistema di posta elettronica certificata, di seguito “servizio PEC”;
      - un sistema di cooperazione applicativa, su rete Internet, con servizio
          esposto tramite modello “web service” fruibile attraverso protocollo
          HTTPS, di seguito “servizio SdICoop”;
      - un sistema di trasmissione dati tra terminali remoti basato su protocollo
          SFTP, di seguito “servizio SdIFtp”;
      - un sistema di trasmissione fruibile mediante la specifica funzionalità del
          servizio web “Fatture e corrispettivi”.

      Il singolo file fattura non può superare la dimensione di 5MB.

      Posta elettronica certificata (servizio PEC)

      Il soggetto trasmittente che intende utilizzare la posta elettronica certificata,
      deve avvalersi di un gestore con il quale mantenere un rapporto finalizzato
      alla disponibilità del servizio di PEC. Tale gestore deve essere tra quelli
      inclusi in apposito elenco pubblico gestito dall’AgID, così come disciplinato
      dagli artt. 14 e 15 del DPR 11 febbraio 2005, n. 68 (“Regolamento recante
      disposizioni per l’utilizzo della posta elettronica certificata, a norma dell’art.
      27 della legge 16 gennaio 2003, n. 3” - G.U. 28 aprile 2005, n. 97).
      Il singolo file (o archivio compresso) da trasmettere costituisce l’allegato del
      messaggio di posta. Un unico messaggio può contenere diversi file allegati.
      Il messaggio con relativi allegati non deve superare la dimensione di 30
      megabyte, valore che costituisce il limite massimo entro il quale il gestore è
      tenuto a garantire il suo invio, come previsto dall’art. 12 del DM 2 novembre
      2005 (“Regole tecniche per la formazione, la trasmissione e la validazione,
      anche temporale, della posta elettronica certificata” - G.U. 15 novembre
      2005, n. 266). Si ricorda che il singolo file fattura (fattura o lotto di fatture)
      non può superare i 5MB.
      L’utilizzo della PEC garantisce di per sé l’identificazione del soggetto
      trasmittente; ciò consente di non ricorrere a procedure di identificazione del
      soggetto come attività propedeutiche alla trasmissione e ne deriva che il SdI
      entra in contatto per la prima volta con il soggetto trasmittente nel momento
      in cui si verifica la ricezione del primo messaggio di posta.
```

<!-- PDF page 15 -->

```text
Per garantire una gestione efficiente del processo di trasmissione il SdI
utilizza, secondo una logica di bilanciamento dei carichi, più indirizzi di PEC
tramite i quali ricevere i file. La prima volta che il soggetto trasmittente invia
una fattura tramite la PEC, deve utilizzare come indirizzo di posta elettronica
certificata del SdI l’indirizzo sdi01@pec.fatturapa.it. Con il primo messaggio
di risposta il SdI comunica al soggetto trasmittente l’indirizzo PEC che dovrà
utilizzare per i successivi invii.
L’utilizzo di un indirizzo di PEC diverso da quello assegnato dal SdI non
garantisce il buon fine della ricezione del messaggio di posta da parte del
SdI stesso.
Il normale flusso di trasmissione tramite PEC prevede, se il processo di invio
e ricezione va a buon fine, che al mittente vengano recapitate due ricevute:
una di accettazione da parte del proprio gestore di posta, e una di avvenuta
consegna da parte del gestore di posta del destinatario; quest’ultima attesta
l’avvenuta trasmissione del file al SdI ma non la correttezza e quindi
l’emissione della fattura elettronica.
L’eventuale scarto del file da parte del SdI o il recapito della fattura allegata
al soggetto ricevente o l’impossibilità di recapito, sono comunicati attraverso
le apposite ricevute, come descritto al successivo paragrafo 1.5.7.
Inoltre, a fronte dell’invio di una PEC priva di allegato da parte del soggetto
trasmittente, il SdI invia un messaggio di cortesia allo stesso notiziandolo
dell’errato invio.

Cooperazione applicativa su rete Internet (servizio SdICoop -
Trasmissione)

Il SdI mette a disposizione su rete Internet un servizio web, richiamabile da
un sistema informatico o da una applicazione, che consente di trasmettere
i file come allegati di un messaggio SOAP.
Questa modalità di trasmissione prevede:
 - l’accreditamento al SdI del canale trasmissivo tramite il Sistema di
      Accreditamento;
 - la richiesta di certificati (validi anche per la produzione) sul Sistema di
      Accreditamento al fine di rendere possibile il colloquio con il SdI;
 - il completamento dei test di interoperabilità previsti dal canale;
 -     la presa visione dell’Accordo di servizio reso disponibile sul Sistema di
      Accreditamento al temine dei test di interoperabilità conclusi
      positivamente;
 - il passaggio in produzione, operazione attraverso la quale il canale
      viene ufficialmente riconosciuto come canale accreditato;
 - la capacità di gestione di certificati digitali.

L’accordo di servizio definisce le regole di comunicazione tra il soggetto
trasmittente e il SdI.

Il file, inviato come allegato SOAP, deve essere identificato secondo le
regole di nomenclatura previste al precedente paragrafo 1.2.2.
La dimensione massima del file allegato al messaggio deve essere di 5
```

<!-- PDF page 16 -->

```text
megabyte.
Diversamente dalla PEC che consente, nel limite dimensionale di 30
megabyte, di inviare con un solo messaggio più file allegati, questa modalità
permette la trasmissione di un solo file (fattura singola piuttosto che lotto di
fatture piuttosto che archivio di fatture) alla volta.
Il servizio esposto ha le caratteristiche seguenti:
 - protocollo HTTPS come trasporto su canale cifrato TLS 1.2;
 - SOAP (with attachments) come standard per i messaggi;
 - MTOM (Message Transmission Optimization Mechanism);
 - WSDL (Web Services Description Language) per descrivere
     l'interfaccia pubblica del web service;
 - autenticazione e autorizzazione basata sull’utilizzo di certificati.

Il riscontro della ricezione è costituito dalla response SOAP prevista dal
servizio; attesta la ricezione del file ma non la correttezza e quindi
l’emissione della fattura.
L’avvenuto scarto del file da parte del SdI, il corretto recapito della fattura al
soggetto ricevente o l’impossibilità di recapito, sono certificati dal SdI stesso
attraverso la predisposizione e l’invio al mittente di ricevute ad hoc, secondo
il sistema di comunicazione descritto al successivo paragrafo 1.5.7.

Gli ulteriori dettagli ai quali attenersi e tutte le informazioni relative alle
procedure per l’utilizzo del servizio, così come la descrizione formale
attraverso WSDL dei servizi, sono riportate nel documento Istruzioni per il
servizio SDICoop - Trasmissione pubblicato sul sito web dell’Agenzia delle
Entrate e sul sito www.fatturapa.gov.it (sezione Documentazione Sistema
di Interscambio).

Sistema di trasmissione dati tra terminali remoti basato su protocollo
SFTP (servizioSdIFtp)

Il SdI prevede la possibilità di trasmissione dei file, preventivamente
crittografati, attraverso protocolli di interconnessione e canali trasmissivi,
anche eventualmente già in uso (seppure per altre finalità) nei rapporti con
l’Amministrazione finanziaria, in ogni caso all'interno di circuiti chiusi che
identificano in modo certo i partecipanti assicurando la qualificazione del
canale.
Questa modalità di trasmissione prevede:

-   l’accreditamento al SdI del canale trasmissivo tramite il Sistema di
    Accreditamento;
-   la dotazione di un server SFTP esposto su Internet;
-   il completamento dei test di interoperabilità previsti dal canale;
-   la presa visione dell’Accordo di servizio reso disponibile sul Sistema di
    Accreditamento al temine dei test di interoperabilità conclusi
    positivamente;
-   il passaggio in produzione, operazione attraverso la quale il canale
    viene ufficialmente riconosciuto come canale accreditato.

L’utilizzo di tale modalità presuppone una struttura a supporto delle attività
```

<!-- PDF page 17 -->

```text
informatiche, la capacità di gestire sistemi informativi ed un centro di
elaborazione dati con caratteristiche di continuità e disponibilità di personale
di presidio. Per le caratteristiche espresse la modalità si adatta a realtà di
soggetti intermediari che si configurano come nodi di concentrazione e di
smistamento.
L’Accordo di servizio definisce le regole di comunicazione tra il soggetto
trasmittente e il SdI, comprese quelle relative al flusso delle ricevute.
Compilata la form sul Sistema di Accreditamento con tutti i dati per
l’attivazione del canale SFTP, il SdI avvia una prima fase di verifica di
corretta trasmissione/ricezione di supporti FTP cifrati; successivamente il
soggetto trasmittente dovrà eseguire i test di interoperabilità descritti
all’interno di apposito kit predisposto dal SdI. Al termine dei suddetti test,
conclusi positivamente, l’utente potrà prendere visione dell’Accordo di
servizio reso disponibile sul sistema di Accreditamento e, successivamente,
effettuare il passaggio in produzione, operazione attraverso la quale il
canale viene ufficialmente riconosciuto come canale accreditato.
La dimensione di ciascun supporto deve essere tale da massimizzare la
velocità di trasferimento dei file e ridurre i tempi legati alla elaborazione dei
supporti stessi. Ciascun operatore dovrà pertanto commisurare le
dimensioni dei propri supporti con la velocità di trasferimento (bit rate) che
potrà garantire; in ogni caso la dimensione massima del supporto
contenente i file è di 150 megabyte. In caso di file maggiori di 150 megabyte,
tali file verranno rinominati, non saranno prelevati e verrà inviata una
segnalazione via email all’indirizzo del referente tecnico del sottoscrittore
contenuto nella form di accreditamento. Esiste inoltre un vincolo alla
eccessiva parcellizzazione dei file fattura all’interno dei supporti: è
ammessa, per ogni accesso al server (che avviene ordinariamente ogni 10
minuti), la presenza sul nodo di scambio di un unico supporto con
dimensione inferiore ai 15 megabyte, mentre eventuali ulteriori supporti
presenti dovranno avere dimensioni comprese tra i 15 megabyte e i 150
megabyte. Il mancato rispetto di tale vincolo determinerà il prelievo di un
unico supporto ad accesso e l’invio di una segnalazione. Si ricorda che il
singolo file fattura (fattura o lotto di fatture) non può superare i 5 megabyte.
Il riscontro della ricezione è costituito da un file di esito (supporto “EO”, in
caso la trasmissione superi la verifica di sicurezza, o “ER”, in caso la verifica
di sicurezza non sia superata) che l’utente trasmittente riceve sullo stesso
nodo sul quale ha depositato il supporto da trasmettere; attesta la ricezione
del file ma non la correttezza e quindi l’emissione della fattura. Il supporto
EO è l’unico supporto a non essere firmato e cifrato, come chiarito nelle
istruzioni per il servizio.

L’avvenuto scarto del file da parte del SdI, il corretto recapito della fattura al
soggetto ricevente o l’impossibilità di recapito, sono certificati dal SdI stesso
attraverso la predisposizione e l’invio al mittente di ricevute ad hoc, secondo
il sistema di comunicazione descritto al successivo paragrafo 1.5.7.

Gli ulteriori dettagli ai quali attenersi e tutte le informazioni relative alle
procedure per l’utilizzo del servizio sono riportati nel documento Istruzioni
per il servizio SDIFTP pubblicato sul sito web dell’Agenzia delle Entrate e
sul sito www.fatturapa.gov.it (sezione Documentazione Sistema di
```

<!-- PDF page 18 -->

```text
Interscambio).

Invio tramite web

Tale modalità prevede l’utilizzo di una funzionalità di trasmissione
telematica via Internet con protocollo HTTPS come trasporto, su canale
cifrato TLS 1.2.
La funzionalità è disponibile in un’area del servizio web “Fatture e
corrispettivi”.
Il file fattura, o il file compresso contenente più file fattura (in ogni caso la
dimensione del file da trasmettere non può eccedere il limite di 5 megabyte),
possono essere trasmessi mediante upload.
Il riscontro della ricezione è costituito da un avviso che, sulla stessa pagina
dalla quale si effettua la trasmissione, viene presentato all’utente; tale
avviso attesta la ricezione del file ma non la correttezza e quindi l’emissione
della fattura.
Successivamente, l’esito dei controlli ed eventualmente del recapito di
ciascun file fattura sarà consultabile tramite le funzionalità di monitoraggio
a disposizione sulla stessa interfaccia web “Fatture e corrispettivi”.

 1.3.2   TRASMISSIONE DEL FILE AL SOGGETTO RICEVENTE
Il SdI trasmette al soggetto ricevente la fattura tramite l’inoltro del file
ricevuto in ingresso utilizzando canali di trasmissione analoghi a quelli
utilizzati per la ricezione.
Le modalità previste per l’inoltro sono:
 - un sistema di posta elettronica certificata, di seguito “servizio PEC”;
 - un sistema di cooperazione applicativa, su rete Internet, con servizio
     esposto tramite modello “web service” fruibile attraverso protocollo
     HTTPS, di seguito “servizio SdICoop - Ricezione”;
 - un sistema di trasmissione dati tra terminali remoti basato su protocollo
     SFTP, di seguito “servizio SdIFtp”;
 - un servizio web messo a disposizione nell’area autenticata dei servizi
     telematici.

Posta elettronica certificata (servizio PEC)

Il soggetto che per ricevere le fatture elettroniche dal SdI intende utilizzare
la posta elettronica certificata, deve avvalersi di un gestore con il quale
mantenere un rapporto finalizzato alla disponibilità del servizio di PEC. Tale
gestore deve essere tra quelli inclusi in apposito elenco pubblico gestito
dall’Agenzia per l’Italia Digitale, così come disciplinato dagli artt. 14 e 15
del DPR 11 febbraio 2005, n. 68 (“Regolamento recante disposizioni per
l’utilizzo della posta elettronica certificata, a norma dell’art. 27 della legge
16 gennaio 2003, n. 3” - G.U. 28 aprile 2005, n. 97).
Il file inoltrato costituisce l’allegato del messaggio di posta; sempre allegato
al medesimo messaggio di posta il SdI trasmetterà il file di “notifica dei
metadati del file fattura” (v. paragrafo 1.5.7): ogni messaggio di posta
riporterà in allegato un solo “file fattura” ed un solo “file di metadati”.
Il normale flusso di trasmissione tramite posta elettronica certificata
```

<!-- PDF page 19 -->

```text
prevede, se il processo di invio e ricezione va a buon fine, che al SdI
vengano recapitate nella propria casella di PEC due ricevute: una di
accettazione da parte del proprio gestore di posta, e una di avvenuta
consegna da parte del gestore di posta del destinatario; quest’ultima attesta
il ‘deposito’, nella casella di PEC del soggetto ricevente, del messaggio e
dei relativi allegati ed ha valore, per il SdI, di “messa a disposizione della
fattura al destinatario” e pertanto dà luogo all’invio al soggetto trasmittente
della “ricevuta di consegna” (v. paragrafo 1.5.7).
Il SdI considera fallita la trasmissione, invia una ricevuta di impossibilità di
recapito (descritta al paragrafo 1.5.7) e mette la fattura a disposizione
nell’area autenticata dei servizi telematici del cessionario/committente, se:
- riceve dal gestore di posta del destinatario una notifica di mancata
     consegna;
- non riceve alcuna comunicazione da parte del gestore di posta del
     destinatario entro un tempo massimo di 40 ore.

Cooperazione applicativa su rete Internet (servizio SdICoop -
Ricezione)

L’utilizzo di tale modalità è possibile per i soggetti che mettono a
disposizione su rete Internet un servizio web che consente al SdI,
richiamando tale servizio, di trasmettere il file fattura ed il file di “notifica dei
metadati del file fattura” come allegato di un messaggio SOAP.
In considerazione del fatto che, per motivi indipendenti dal SdI, il servizio
potrebbe essere temporaneamente indisponibile, il Sistema effettua fino ad
un massimo di 4 tentativi di trasmissione, uno ogni 6 ore, al termine dei
quali il SdI considera fallita la trasmissione, invia una ricevuta di
impossibilità di recapito (descritta al paragrafo 1.5.7) e mette la fattura a
disposizione     nell’area    autenticata    dei      servizi     telematici      del
cessionario/committente.

Questa modalità di ricezione prevede:

-   l’accreditamento al SdI del canale trasmissivo tramite il Sistema di
    Accreditamento;
-   la richiesta di certificati (validi anche per la produzione) sul Sistema di
    Accreditamento al fine di rendere possibile il colloquio con il SdI;
-   il completamento dei test di interoperabilità previsti dal canale;
-   la presa visione dell’Accordo di servizio reso disponibile sul sistema di
    Accreditamento al temine dei test di interoperabilità conclusi
    positivamente;
-   il passaggio in produzione, operazione attraverso la quale il canale
    viene ufficialmente riconosciuto come canale accreditato;
-   la capacità di gestione di certificati digitali.

L’Accordo di servizio definisce le regole di comunicazione fra il SdI e il
soggetto ricevente.
Il servizio esposto ha le caratteristiche seguenti:
- protocollo HTTPS come trasporto su canale cifrato TLS 1.2;
- SOAP (with attachments) come standard per i messaggi;
```

<!-- PDF page 20 -->

```text
-   MTOM (Message Transmission Optimization Mechanism);
-   WSDL (Web Services Description Language) per descrivere
    l'interfaccia pubblica del web service;
-   autenticazione e autorizzazione basata sull’utilizzo di certificati.

Gli ulteriori dettagli ai quali attenersi e tutte le informazioni relative alle
procedure per l’utilizzo del servizio, così come la descrizione formale
attraverso WSDL dei servizi, sono riportate nel documento Istruzioni per il
servizio SDICoop - Ricezione pubblicato sul sito web dell’Agenzia delle
Entrate e sul sito www.fatturapa.gov.it (sezione Documentazione Sistema
di Interscambio).

Sistema di trasmissione dati tra terminali remoti basato su protocollo
SFTP (servizioSdIFtp)

Il SdI prevede la possibilità di trasmissione dei file, preventivamente
crittografati, attraverso protocolli di interconnessione e canali trasmissivi
anche eventualmente già in uso (seppure per altre finalità) nei rapporti con
l’Amministrazione finanziaria, in ogni caso all'interno di circuiti chiusi che
identificano in modo certo i partecipanti assicurando la qualificazione del
canale.
Questa modalità di trasmissione prevede:

-   l’accreditamento al SdI del canale trasmissivo tramite il Sistema di
    Accreditamento;
-   la dotazione di un server SFTP esposto su Internet;
-   il completamento dei test di interoperabilità previsti dal canale;
-   la presa visione dell’Accordo di servizio reso disponibile sul Sistema di
    Accreditamento al temine dei test di interoperabilità conclusi
    positivamente;
-   il passaggio in produzione, operazione attraverso la quale il canale
    viene ufficialmente riconosciuto come canale accreditato.

L’utilizzo di tale modalità presuppone una struttura a supporto delle attività
informatiche, la capacità di gestire sistemi informativi ed un centro di
elaborazione dati con caratteristiche di continuità e disponibilità di
personale di presidio. Per le caratteristiche espresse la modalità si adatta
a realtà di soggetti intermediari che si configurano come nodi di
concentrazione e di smistamento.

L’Accordo di servizio definisce le regole di comunicazione tra il soggetto
ricevente e il SdI, comprese quelle relative al flusso delle ricevute.
Compilata la form sul sistema di accreditamento con tutti i dati per
l’attivazione del canale SFTP, il SdI avvia una prima fase di verifica di
corretta trasmissione/ricezione di supporti FTP cifrati; successivamente il
soggetto trasmittente dovrà eseguire i test di interoperabilità descritti
all’interno di apposito kit predisposto dal SdI. Al termine dei suddetti test,
conclusi positivamente, l’utente potrà prendere visione dell’Accordo di
servizio e successivamente effettuare il passaggio in produzione,
operazione attraverso la quale il canale viene ufficialmente riconosciuto
```

<!-- PDF page 21 -->

```text
      come canale accreditato.

      La trasmissione di una fattura tramite il servizio SDIFTP avviene mediante
      la messa a disposizione di un supporto di tipo “FO” che la contiene nella
      directory di scambio. La data/ora in cui termina con successo la
      trasmissione del supporto viene riportata nelle Ricevute di Consegna che il
      Sistema di Interscambio invia relativamente ai file fattura consegnati.

      Gli ulteriori dettagli ai quali attenersi e tutte le informazioni relative alle
      procedure per l’utilizzo del servizio sono riportati nel documento Istruzioni
      per il servizio SDIFTP pubblicato sul sito web dell’Agenzia delle Entrate e
      sul sito www.fatturapa.gov.it (sezione Documentazione Sistema di
      Interscambio).

      Servizio web di messa a disposizione nell’area autenticata dei servizi
      telematici

      Il SdI, nei casi in cui non sia stato possibile recapitare la fattura elettronica
      al    soggetto      destinatario,    rende     disponibile    la    stessa     al
      cessionario/committente nella sua area riservata del sito web dell’Agenzia
      delle entrate.
      In questo caso comunica tale informazione al soggetto trasmittente,
      attraverso la ricevuta di impossibilità di recapito di cui al paragrafo 1.5.7,
      unitamente alla data di messa a disposizione del file.

      Nel momento in cui il cessionario/committente, accedendo alla sua area
      riservata, prende visione della fattura, il sistema registra tale operazione e
      modifica lo stato del file che risulta così ricevuto dal destinatario.
      L’informazione della data di presa visione della fattura da parte del
      cessionario/committente è resa disponibile al cedente/prestatore nell’area
      di consultazione delle fatture elettroniche di sua competenza.

1.4   MODALITÀ DI INOLTRO DELLE COMUNICAZIONI DA PARTE DEL SDI
      Le comunicazioni prodotte dal SdI vengono inoltrate tramite lo stesso
      canale utilizzato per la trasmissione del file fattura. Le comunicazioni sono
      costituite da file XML firmati elettronicamente, con firma XAdES Baseline B
      (XML Advanced Electronic Signatures) in modalità “enveloped” come
      previsto dalla normativa europea vigente in materia di firma elettronica; i
      file vengono trasmessi come allegati ai messaggi previsti per i diversi canali
      trasmissivi. La struttura XML dei file di comunicazione è dettagliata nel
      “Formato dei file di comunicazione del SDI” riportato in appendice.
      In particolare:
      - nel caso di “servizio PEC”, le comunicazioni sono spedite all’indirizzo di
           PEC del mittente o del destinatario del file fattura; i messaggi PEC sono
           costituiti da una versione “Human Readable” (il corpo del messaggio) e
           dal file XML in allegato;
      - nel caso di “servizio SdICoop” l’accordo prevede l’esposizione, da parte
           del soggetto che ha trasmesso o ricevuto il file fattura, di un analogo
```

<!-- PDF page 22 -->

```text
            servizio richiamabile dal SdI per l’invio delle comunicazioni come file
            XML in allegato al messaggio soap;
      -     nel caso di “servizio SdIFtp”, le comunicazioni vengono inoltrate
            utilizzando lo stesso protocollo ed attraverso lo stesso canale (nel caso
            di canale bidirezionale) o canale dedicato (nel caso di canali
            unidirezionali);
      -     nel caso di invio telematico attraverso il servizio esposto su interfaccia
            web di fruizione dei servizi “Fatture e corrispettivi”, le comunicazioni
            sono reperibili, come file XML, tramite la funzionalità di monitoraggio
            disponibile sull’interfaccia suindicata.

1.5   MODALITÀ DI INTERAZIONE CON IL SDI
      Il presente paragrafo descrive le procedure operative per la trasmissione
      delle fatture elettroniche attraverso il SdI, e lo scambio di informazioni tra
      gli attori del processo.

          1.5.1   INTERAZIONE DEL SOGGETTO RICEVENTE CON IL SDI
      Affinché il soggetto ricevente possa validamente interagire con il SdI
      devono essere soddisfatti i requisiti indispensabili alla sua qualificazione e
      al suo riconoscimento.
      Il soddisfacimento di tali requisiti passa attraverso la definizione del canale
      di trasmissione e/o il censimento della modalità di ricezione delle proprie
      fatture d’acquisto.

      1.5.1.1 Definizione del canale di trasmissione
          La definizione del canale di trasmissione avviene tramite:

      -     l’adozione di una casella di posta elettronica certificata (PEC);
      -     la realizzazione del servizio di ricezione (WS-SDICoop), secondo le
            modalità e le specifiche pubblicate sul sito www.fatturapa.gov.it;
      -     la definizione di un accordo per disciplinare la trasmissione della fattura,
            e relativi messaggi di notifica, attraverso protocolli di “file transfer”
            (servizio SdIFtp) all'interno di circuiti chiusi che identificano in modo
            certo i partecipanti assicurando la qualificazione del canale.

      1.5.1.2 Registrazione dell’indirizzo telematico per la ricezione dei
              file fattura
          La funzionalità web “Registrazione dell’indirizzo telematico preferito per la
          ricezione dei file fattura” è accessibile dall’interfaccia web di fruizione dei
          servizi “Fatture e corrispettivi”, dalla box Fatturazione presente nella home
          page.
          Consente a ciascun soggetto passivo IVA di indicare la modalità con la
          quale desidera ricevere le fatture elettroniche delle quali risulta
          cessionario/committente. L’utente potrà scegliere la modalità di ricezione
```

<!-- PDF page 23 -->

```text
 delle fatture elettroniche. La scelta effettuata attraverso la funzione di
 registrazione verrà considerata dal SdI come prioritaria nella consegna
 della fattura elettronica.

1.5.2    PROCEDURA PER L’IDENTIFICAZIONE DEL SOGGETTO TRASMITTENTE AI
         FINI DELL’INTERAZIONE CON IL SDI

Affinché il soggetto trasmittente possa validamente interagire con il SdI,
devono essere soddisfatti i requisiti minimi indispensabili alla sua
identificazione.
Il soddisfacimento di tali requisiti si ottiene definendo uno o più canali di
trasmissione/comunicazione attraverso:
- dotazione di una casella di posta elettronica certificata o di analogo
    indirizzo di posta elettronica basato su tecnologie che certifichino data
    e ora dell'invio e della ricezione delle comunicazioni e l'integrità del
    contenuto delle stesse;
- presa visione di un Accordo di servizio sul Sistema di Accreditamento
    con il SdI per disciplinare la trasmissione della fattura, e relativi
    messaggi di notifica, attraverso servizio esposto tramite modello “web-
    service”;
- presa visione di un Accordo di servizio sul Sistema di Accreditamento
    con il SdI per disciplinare la trasmissione della fattura, e relativi
    messaggi di notifica, attraverso protocolli di “file transfer” (servizio
    SdIFtp) all'interno di circuiti chiusi che identificano in modo certo i
    partecipanti assicurando la qualificazione del canale.

Nel caso di utilizzo del servizio di invio telematico esposto su interfaccia
web di fruizione dei servizi “Fatture e corrispettivi”, o in caso di accesso al
Sistema di accreditamento, l’autenticazione avviene attraverso l’utilizzo
delle credenziali Fisconline o Entratel oppure, in alternativa, di un
dispositivo di tipo Smartcard rispondente ai requisiti della Carta Nazionale
dei Servizi-CNS, preventivamente registrata ai Servizi Telematici
dell’Agenzia delle Entrate oppure di un’identità SPID.

1.5.3    PROCEDURA PER L’ACCREDITAMENTO DEI CANALI WEB SERVICE E SFTP
I canali Web Service e SFTP possono essere utilizzati solo dopo il loro
accreditamento. La procedura di accreditamento è disponibile accedendo
attraverso il seguente link https://accreditamento.fatturapa.gov.it .
Le finalità e le modalità di accesso al Sistema di Accreditamento sono
descritte sul sito www.fatturapa.gov.it

I passi da seguire sono i seguenti:
- accesso al Sistema di Accreditamento;
- inserimento dei dati, divisi in:
       o dati anagrafici del sottoscrittore e del riferimento dell’Accordo di
           servizio;
       o dati relativi al canale Web Service o SFTP da accreditare
```

<!-- PDF page 24 -->

```text
-   esclusivamente per il canale WS, generazione dei certificati attraverso
    il caricamento di CSR (Certificate Signing Request)
- svolgimento dei test di interoperabilità previsti;
- presa visione dell’Accordo di Servizio a conclusione del superamento
    dei test;
- richiesta del passaggio in produzione.
 Per i canali accreditati in ricezione una volta effettuato il passaggio in
 produzione e , attraverso il Sistema di Accreditamento, l’utente può
 richiedere dei codici destinatario (fino ad un massimo di 300 codici).

    1.5.4   PROCEDURA DI INVIO FATTURA AL SDI
La procedura di invio della fattura al SdI vede, quali attori coinvolti:
- il soggetto emittente;
- il soggetto trasmittente (se diverso dal soggetto emittente);
- il Sistema di Interscambio;
I passaggi rappresentativi si possono schematizzare nei punti seguenti:
- il soggetto emittente predispone la fattura elettronica secondo quanto
    riportato al precedente paragrafo 1.2;
- sulla fattura così predisposta, nel caso si scelga di apporre la firma
    digitale per garantire autenticità e integrità della fattura, il soggetto
    emittente appone la firma secondo una delle modalità descritte nel
    paragrafo 1.2.1;
- il file così generato viene trasmesso dal soggetto trasmittente al SdI per
    mezzo dei canali e le modalità di cui al precedente paragrafo 1.3.1.

    1.5.5   PROCEDURA DI INOLTRO DELLA FATTURA AL SOGGETTO RICEVENTE
La procedura di inoltro della fattura elettronica dal SdI al soggetto ricevente
vede, quali attori coinvolti:
- il Sistema di Interscambio;
- il soggetto ricevente.
Il SdI, una volta effettuate le verifiche previste, inoltra al soggetto ricevente
la fattura elettronica attraverso i canali e con le modalità di cui al precedente
paragrafo 1.3.2.
Per l’individuazione del canale a cui trasmettere il file fattura, il SdI adotta
la seguente procedura:
- se il soggetto ricevente ha registrato l’indirizzo telematico al quale
    desidera ricevere le fatture elettroniche, provvede al recapito secondo
    le indicazioni fornite;
- se l’elemento informativo del file fattura CodiceDestinatario contiene un
    valore corrispondente a un canale di trasmissione attivo, inoltra il file
    fattura al canale individuato;
- se l’elemento informativo del file fattura CodiceDestinatario contiene il
    valore “0000000” ed è valorizzato l’elemento informativo
    PECDestinatario, il SdI inoltra il file alla casella di posta elettronica
    certificata individuata;
- se l’elemento informativo del file fattura CodiceDestinatario contiene il
    valore “0000000” e l’elemento informativo PECDestinatario non è
    valorizzato, il SdI mette a disposizione il file fattura nell’area autenticata
    dei servizi telematici del cessionario/committente.
```

<!-- PDF page 25 -->

```text
Nei casi in cui uno stesso soggetto svolge contemporaneamente il ruolo di
intermediario sia per chi trasmette la fattura elettronica che per chi la riceve
attraverso lo stesso canale trasmissivo, è possibile adottare un flusso
semplificato per le cui particolarità si rimanda al successivo paragrafo 1.5.6.

1.5.6     FLUSSO SEMPLIFICATO
L’adozione del flusso semplificato di recapito dei file fattura, che può essere
richiesto tramite l’accesso al Sistema di Accreditamento, è vincolata al
soddisfacimento di tutte le condizioni seguenti:
- il cessionario/committente deve aver registrato “l’indirizzo telematico”
     preferito per la ricezione dei file, utilizzando il servizio di registrazione di
     cui al precedente paragrafo 1.5.1.2;
- “l’indirizzo telematico” di cui al punto precedente è uno dei canali che
     richiedono accreditamento (web-service o sftp, vedi par 1.5.3) e non
     può essere una casella di Posta Elettronica Certificata;
- il file fattura perviene al SdI attraverso il medesimo canale
     corrispondente “all’indirizzo telematico” di cui al punto precedente;
- il soggetto che ha accreditato il canale ha indicato, sul Sistema di
     Accreditamento, che intende utilizzare anche il flusso semplificato.
Il flusso semplificato si basa sulla coincidenza del soggetto trasmittente e
del soggetto ricevente e consiste nei seguenti passi:
- il SdI riceve il file fattura;
- il SdI effettua i controlli sul file fattura ricevuto;
- in caso di controlli non superati, il SdI invia al soggetto trasmittente la
     ricevuta di scarto;
- in caso di esito positivo dei controlli, il SdI invia la ricevuta di consegna
     senza trasmettere il file fattura.

1.5.7     PROCEDURA DI GESTIONE DELLE RICEVUTE
Tutti i canali di trasmissione descritti al precedente paragrafo 1.3.1
prevedono dei messaggi di ritorno a conferma del buon esito della
trasmissione; questi messaggi sono specifici delle infrastrutture di
comunicazione e garantiscono la “messa a disposizione” del messaggio e
dei file allegati da parte di chi invia rispetto a chi riceve.
In aggiunta a questi messaggi di riscontro della ricezione, il SdI attesta
l’avvenuto svolgimento delle fasi principali del processo di trasmissione
delle fatture elettroniche attraverso un sistema di comunicazione che si
basa sull’invio di ricevute tramite le modalità ed i canali riportati al
precedente paragrafo 1.4.
Tali ricevute vengono inoltrate sullo stesso canale attraverso il quale sono
state trasmesse le relative fatture.
In considerazione del fatto che, per motivi indipendenti dal SdI, il canale
potrebbe essere temporaneamente indisponibile, il Sistema:
- per il canale web-services (Servizio SDICoop – Ricezione) effettua fino
    ad un massimo di 6 tentativi di trasmissione distribuiti in tre giorni (uno
    ogni 12 ore);
- per il canale PEC effettua fino ad un massimo di 2 tentativi; il secondo
    viene eseguito a distanza di:
```

<!-- PDF page 26 -->

```text
         o almeno un’ora dalla ricezione della notifica di mancata
             consegna da parte del gestore PEC del destinatario;
         o almeno 40 ore dal primo tentativo, in assenza di notifica da
             parte del gestore PEC del destinatario.
Se dopo i tentativi previsti il relativo canale risulta ancora irraggiungibile, il
processo si chiude e il cedente/prestatore potrà verificare lo stato della
fattura attraverso le funzionalità di monitoraggio e di consultazione.
La procedura di gestione delle ricevute vede, quali attori coinvolti:
- il soggetto trasmittente;
- il Sistema di Interscambio;
- il soggetto ricevente.

La procedura può essere schematizzata nei punti seguenti:
   a) il SdI, ricevuto correttamente il file, assegna un identificativo proprio
      ed effettua le verifiche previste (v. paragrafo 1.6);
   b) in caso di controlli con esito negativo, il SdI invia una ricevuta di
      scarto al soggetto trasmittente;
   c) nel caso di esito positivo dei controlli il SdI verifica se, sulla base di
      quanto indicato al paragrafo 1.5.5, è individuato un canale per il
      recapito;
         o se non è stato possibile individuare un canale di recapito il SdI
            rende disponibile la fattura nell’area riservata del
            cessionario/committente e invia al soggetto trasmittente una
            ricevuta di impossibilità di recapito con la quale comunica di
            aver messo a disposizione la fattura nell’area autenticata dei
            servizi telematici del cessionario/committente; se il canale è
            individuato, il SdI trasmette la fattura elettronica al soggetto
            ricevente; unitamente al file fattura, il SdI trasmette anche, sotto
            forma di file XML, una notifica di metadati del file fattura
            tramite la quale sono comunicate le informazioni utili
            all’elaborazione ed alla comunicazione da parte del soggetto
            ricevente; se la trasmissione ha avuto esito positivo, il SdI invia
            al soggetto trasmittente una ricevuta di consegna della fattura
            elettronica e mette a disposizione un duplicato informatico della
            fattura       elettronica        nell’area        riservata      del
            cessionario/committente;
         o nel caso in cui, per cause tecniche non imputabili al SdI, la
            trasmissione al soggetto ricevente non fosse possibile, il SdI
            invia al soggetto trasmittente una ricevuta di impossibilità di
            recapito con la quale comunica di aver messo a disposizione
            la fattura nell’area autenticata dei servizi telematici del
            cessionario/committente e trasmette anche, sotto forma di file
            XML, una notifica di metadati del file fattura tramite la quale
            sono comunicate le informazioni utili all’elaborazione ed alla
            comunicazione da parte del soggetto ricevente.

La ricevuta di consegna, inviata al soggetto trasmittente in caso di esito
positivo del recapito della fattura, contiene anche l’indicazione della data di
```

<!-- PDF page 27 -->

```text
      consegna della fattura. Questa data è individuata, sulla base del canale di
      trasmissione, come specificato nel successivo paragrafo 1.7. Per quanto
      descritto nel paragrafo citato, il processo di consegna della fattura può
      influenzare i tempi di consegna della ricevuta per cause non dipendenti dal
      SdI.
      Le ricevute vengono predisposte secondo un formato XML; le specifiche
      tecniche e la documentazione di tali messaggi sono riportate in Appendice
      6.

1.6   CONTROLLI EFFETTUATI DAL SDI
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
      conseguentemente, non viene inoltrato al destinatario. La ricevuta di scarto
      viene recapitata – come previsto al punto 2.4 del provvedimento – entro 5
      giorni dalla corretta ricezione del file fattura stesso, in altri termini
      dall’istante T0 descritto nel successivo paragrafo 1.7.
      La fattura elettronica o il lotto di fatture elettroniche contenute nel file si
      considerano non emesse.
      L’elenco dei controlli e dei relativi codici di errore è riportato in Appendice
      1.

1.7   TEMPI DI ELABORAZIONE DA PARTE DEL SDI
      Nella tabella seguente sono indicati gli eventi temporali (data e ora) che
      caratterizzano il processo e vengono riportati nelle ricevute del SdI:
      - T0: il momento in cui la fattura viene ricevuta dal SdI (DataOraRicezione
         presente nella ricevuta di consegna e nella ricevuta di impossibilità di
         recapito);
      - T1: il momento in cui la stessa viene ricevuta dal destinatario
         (DataOraConsegna presente nella ricevuta di consegna).

      Va tenuto comunque presente che il tempo T1 può essere influenzato da
      fattori indipendenti dall’infrastruttura SdI quali:
      - ritardi del gestore PEC del destinatario nel recapito della notifica di
          avvenuta consegna;
      - bit rate che ciascun operatore è in grado di garantire sul proprio canale
          di ricezione (FTP o WS);
      - temporanea indisponibilità del canale di ricezione.
```

<!-- PDF page 28 -->

```text
  Per identificare i momenti T0 e T1 si forniscono di seguito i riferimenti
  utilizzati sulla base del canale scelto dal mittente e dal destinatario.

  CANALE di
TRASMISSIONE                      T0                              T1

                          Data e ora presenti             Data e ora presenti
                     all’interno della ricevuta di   all’interno della ricevuta di
 Servizio PEC             consegna inviata al        consegna inviata al SdI dal
                     soggetto trasmittente dal            gestore di PEC del
                          suo gestore di PEC              soggetto ricevente

                                                          Data e ora presenti
                          Data e ora presenti
                                                     all’interno della “response”
servizio SdICoop     all’interno della “response”
                                                      del servizio esposto dal
                     del servizio esposto dal SdI          soggetto ricevente

                     Data e ora (datetime) della
                                                      Data e ora di consegna
                      fine del prelevamento (se
                                                       relative al supporto FO
                       l’operazione termina con
                                                         contenente la fattura
                     successo) del file messo a
                                                       ricevuta, contenute nel
                     disposizione nella directory
Servizio SdIFtp                                      report di cui al punto 1 del
                         di scambio restituiti dal
                                                          paragrafo 5.5 del
                        servizio di trasmissione,
                                                     documento Istruzioni per il
                          espressa secondo lo
                                                     servizio SDIFTP (versione
                        standard UTC (CET con
                                                                 4.1.1)
                               ora legale)

                      Data e ora restituiti dal
Invio Telematico
                     messaggio di risposta della        Canale non previsto
     via web            funzionalità di invio

    Messa a                                              Data e ora in cui il
  disposizione                                        soggetto ricevente o suo
    nell’area           Canale non previsto           intermediario prendono
 autenticata dei                                        visione della fattura
servizi telematici                                      nell’area autenticata
```

<!-- PDF page 29 -->

```text
2.   FATTURAZIONE ELETTRONICA – FORMATO DELLA FATTURA
     La presente sezione del documento descrive le regole tecniche relative alle
     soluzioni informatiche da utilizzare per l’emissione delle fatture di cui
     all’articolo 1, comma 3, del decreto legislativo 127/2015.
     Tali regole sono state mutuate dal documento Specifiche tecniche
     operative del formato della fattura del Sistema di Interscambio di cui
     all’allegato A del DM 55/2013; a tale documento si rimanda per
     l’emissione della fattura elettronica destinata alle pubbliche amministrazioni
     ai sensi della Legge 244/2007.
     Per quanto attiene al contenuto informativo della fattura, il DPR 633 del 26
     ottobre 1972 e successive modifiche ed integrazioni riporta le informazioni
     obbligatorie in quanto rilevanti ai fini fiscali. In particolare, le informazioni
     riguardano:
     - data di emissione;
     - numero progressivo che identifichi la fattura in modo univoco;
     - ditta, denominazione o ragione sociale, nome e cognome, residenza o
         domicilio dei soggetti fra cui è effettuata l'operazione
         (cedente/prestatore e cessionario/committente) e dell’eventuale
         rappresentante fiscale, nonché ubicazione della stabile organizzazione
         per i soggetti non residenti;
     - numero di partita IVA del cedente/prestatore;
     - numero di partita IVA del cessionario/committente (o numero di
         identificazione IVA attribuito dallo Stato membro di stabilimento per i
         soggetti stabiliti in altro Stato membro dell’UE) oppure numero di codice
         fiscale se non agisce nell’esercizio di impresa, arte o professione;
     - natura, qualità e quantità dei beni e dei servizi formanti oggetto
         dell'operazione;
     - corrispettivi ed altri dati necessari per la determinazione della base
         imponibile, compresi quelli relativi ai beni ceduti a titolo di sconto,
         premio o abbuono di cui all'art. 15, primo comma, n. 2;
     - corrispettivi relativi agli altri beni ceduti a titolo di sconto, premio o
         abbuono;
     - aliquota, ammontare dell'imposta e dell'imponibile con arrotondamento
         al centesimo di euro;
     - data della prima immatricolazione o iscrizione in pubblici registri e
         numero dei chilometri percorsi, delle ore navigate o delle ore volate, se
         trattasi di cessione intracomunitaria di mezzi di trasporto nuovi, di cui
         all'art. 38, comma 4, del DL 30 agosto 1993, n. 331, convertito, con
         modificazioni, dalla Legge 29 ottobre 1993, n. 427;
     - annotazione che la stessa è emessa per conto del cedente/prestatore,
         dal cessionario/committente ovvero da un terzo.

     Oltre a queste informazioni, nel definire il contenuto informativo della fattura
     elettronica si tiene conto di:
```

<!-- PDF page 30 -->

```text
      -     informazioni necessarie ai fini di una corretta trasmissione della fattura
            al Sistema di Interscambio e da questi al soggetto destinatario;
      -     informazioni necessarie a consentire una completa dematerializzazione
            del processo di ciclo passivo attraverso l’integrazione del documento
            con i processi ed i sistemi gestionali e di pagamento;
      -     ulteriori eventuali informazioni che possono risultare utili sulla base delle
            tipologie di beni/servizi ceduti/prestati e delle esigenze informative
            intercorrenti tra singolo fornitore e singolo cliente.

      La presente sezione si compone di due parti:
      - nella prima i dati vengono rappresentati secondo una suddivisione che
         riprende la struttura del file e per ognuno viene riportata la
         denominazione del relativo elemento informativo del tracciato, la
         descrizione del significato, e le caratteristiche di utilizzo
      - nella seconda i dati vengono descritti nel dettaglio tecnico con
         particolare attenzione alla loro struttura sintattica ed alle caratteristiche
         implementative del file.

          In Appendice 2 è riportato l’”XML Schema” (xsd) della fattura elettronica
          ordinaria, mentre in Appendice 3 sono riportati l’”XML Schema” (xsd) e la
          rappresentazione tabellare della fattura elettronica semplificata (art. 21 bis
          DPR 633/1972).

2.1   PARTE PRIMA

          2.1.1    FATTURA ELETTRONICA HEADER – DATI DI TRASMISSIONE
      La valorizzazione di questi dati è indispensabile ai fini di un corretto recapito
      del documento elettronico; va pertanto effettuata sulla base dei criteri
      descritti di seguito.

      <DatiTrasmissione>

            IdTrasmittente (identificativo fiscale del soggetto trasmittente)

                     IdPaese: codice del paese assegnante l’identificativo fiscale al
                     soggetto trasmittente.

                     IdCodice: numero di identificazione fiscale del trasmittente (per
                     i soggetti stabiliti nel territorio dello Stato Italiano corrisponde al
                     Codice Fiscale; per i non residenti si fa riferimento
                     all’identificativo fiscale assegnato dall’autorità del paese di
                     residenza). In caso di IdPaese uguale a IT, il sistema ne verifica
```

<!-- PDF page 31 -->

```text
        la presenza in Anagrafe Tributaria: se non esiste come codice
        fiscale, il file viene scartato con codice errore 00300.

ProgressivoInvio: progressivo che il soggetto trasmittente attribuisce
al file che inoltra al Sistema di Interscambio per una propria finalità di
identificazione univoca.

FormatoTrasmissione: codice identificativo del tipo di trasmissione
che si sta effettuando e del relativo formato. Va sempre valorizzato con
“FPR12”.

CodiceDestinatario: identifica il canale telematico sul quale recapitare
la fattura; deve contenere un valore alfanumerico di 7 caratteri
corrispondente a:

  - uno dei codici che il Sistema di Interscambio attribuisce ai soggetti,
    con canale accreditato in ricezione, che ne abbiano fatto richiesta
    attraverso la funzione di ‘Richiesta codici destinatario B2B,
    presente nel Sistema di Accreditamento;

  - ‘0000000’, nei casi di fattura destinata ad un soggetto che riceve
    tramite PEC e questa sia stata indicata nel campo
    PECDestinatario;

  - ‘0000000’, nei casi di fattura destinata ad un soggetto per il quale
    non si conosce il canale telematico (PEC o altro) sul quale
    recapitare il file.

  - ‘XXXXXXX’, in caso di fattura emessa verso soggetti non stabiliti
    in Italia, e inviata al Sistema di Interscambio al fine di trasmettere
    i dati ex comma 3-bis dell’articolo 1 del decreto legge n. 127 del
    2015, di cui al capitolo 3 del presente documento; in questo caso
    il    sistema      controlla   che      il campo       IdPaese     del
    cessionario/committente contenga un valore diverso da “IT”:
    diversamente il file viene scartato con codice errore 00313.

ContattiTrasmittente

        Telefono: contatto telefonico fisso / mobile.

        Email: indirizzo di posta elettronica.

PECDestinatario: indirizzo di Posta Elettronica Certificata al quale, se
valorizzato, viene recapitata la fattura nei casi in cui il valore di
CodiceDestinatario sia uguale a ‘0000000’ e non risulti registrato alcun
canale      telematico   associato      alla    partita     IVA      del
```

<!-- PDF page 32 -->

```text
   cessionario/committente. Non sono ammessi indirizzi PEC
   corrispondenti alle caselle di funzionamento del Sistema di
   Interscambio (sdixx@pec.fatturapa.it); diversamente il file viene scartato
   con codice errore 00330.

 2.1.2   FATTURA      ELETTRONICA             HEADER        –     DATI     DEL
         CEDENTE/PRESTATORE

<CedentePrestatore>

   DatiAnagrafici

           IdFiscaleIVA (gli elementi di questo blocco consentono
           l’inserimento della partita IVA italiana per i soggetti residenti in
           Italia e per quelli esteri operanti in Italia identificati:

               ✓ attraverso una Stabile Organizzazione

               ✓ mediante un rappresentante fiscale

               ✓ direttamente.

           Negli altri casi consentono l’inserimento dell’identificativo
           fiscale assegnato dall’autorità del proprio paese per i soggetti
           non residenti)

                  IdPaese: codice del paese assegnante l’identificativo
                  fiscale al soggetto cedente/prestatore. Se vale IT, il
                  sistema verifica che il TipoDocumento sia diverso da
                  TD17, TD18, TD19 e TD28; in caso contrario il file viene
                  scartato con codice errore 00473; oppure se ha un valore
                  diverso da IT, ma il TipoDocumento è TD29, il file viene
                  scartato con codice errore 00473. Se ha un valore
                  diverso da IT, il sistema verifica che non sia diverso da
                  IT anche l’ IdPaese del cessionario/committente; in caso
                  contrario il file viene scartato con codice 00476.

                  IdCodice: numero di identificazione fiscale del
                  cedente/prestatore. In caso di IdPaese uguale a IT, il
                  sistema ne verifica la presenza in Anagrafe Tributaria: se
                  non esiste come partita IVA, il file viene scartato con
                  codice errore 00301. Con lo stesso codice errore il file
                  viene scartato se la partita IVA risulta cessata in data
                  antecedente o uguale alla data del documento e se il
                  TipoDocumento è diverso da TD16, TD20 e TD29. Se
```

<!-- PDF page 33 -->

```text
       invece il TipoDocumento è pari a TD16, TD20 o TD29 e
       la partita IVA risulta cessata da oltre 5 anni rispetto alla
       data del documento, il file viene scartato con codice
       errore 00323.

CodiceFiscale:      numero      di   codice   fiscale      del
cedente/prestatore. Il sistema ne verifica la presenza in
Anagrafe Tributaria: se non esiste come codice fiscale, il file
viene scartato con codice errore 00302.

Nel caso di IdFiscaleIVA di gruppo IVA, il sistema verifica che
sia presente anche il CodiceFiscale e che appartenga a uno dei
partecipanti al gruppo; se il CodiceFiscale non è presente il file
viene scartato con codice errore 00322; se è presente ma non
è di partecipante al gruppo IVA, il file viene scartato con codice
errore 00321.

Nel caso di IdFiscaleIVA non di gruppo IVA e di presenza anche
del CodiceFiscale, il sistema controlla che facciano riferimento
allo stesso soggetto; in caso contrario il file viene scartato con
codice errore 00320.

Anagrafica

       Denominazione: ditta, denominazione o ragione sociale
       del cedente/prestatore del bene/servizio da valorizzare
       nei casi di persona non fisica.

       Nome: nome del cedente/prestatore del bene/servizio
       da valorizzare nei casi di persona fisica.

       Cognome: cognome del cedente/prestatore del
       bene/servizio da valorizzare nei casi di persona fisica.

       Titolo: titolo onorifico del cedente/prestatore.

       CodEORI: numero del Codice EORI (Economic
       Operator Registration and Identification) in base al
       Regolamento (CE) n. 312 del 16 aprile 2009. In vigore
       dal 1° luglio 2009 tale codice identifica gli operatori
       economici nei rapporti con le autorità doganali sull'intero
       territorio dell'Unione Europea.

AlboProfessionale: nome dell'albo             professionale    cui
appartiene il cedente/prestatore.
```

<!-- PDF page 34 -->

```text
        ProvinciaAlbo: provincia dell'albo professionale.

        NumeroIscrizioneAlbo:          numero      di   iscrizione    all'albo
        professionale.

        DataIscrizioneAlbo: data di iscrizione all'albo professionale

        RegimeFiscale: regime fiscale del cedente/prestatore del
        bene/servizio; deve essere valorizzato con uno dei codici
        elencati nella Parte II del presente documento.

Sede (gli elementi indicati di seguito fanno riferimento alla sede legale
per le società e del domicilio fiscale per le ditte individuali e i lavoratori
autonomi)

        Indirizzo: indirizzo del cedente/prestatore del bene/servizio;
        deve essere valorizzato con il nome della via, piazza, etc.,
        comprensivo, se si vuole, del numero civico.

        NumeroCivico: numero civico relativo all’indirizzo specificato
        nell’elemento precedente; si può omettere se già riportato
        nell’elemento precedente.

        CAP: Codice di Avviamento Postale relativo all’indirizzo.

        Comune: comune cui si riferisce l’indirizzo.

        Provincia: sigla della provincia di appartenenza del comune
        (da valorizzare nei soli casi di sede in Italia).

        Nazione: codice della nazione.

StabileOrganizzazione (gli elementi indicati di seguito devono essere
obbligatoriamente valorizzati nei soli casi in cui il cedente/prestatore è
un soggetto non residente ed effettua la transazione oggetto del
documento tramite stabile organizzazione)

        Indirizzo: indirizzo della stabile organizzazione del
        cedente/prestatore del bene/servizio; deve essere valorizzato
        con il nome della via, piazza, etc., comprensivo, se si vuole, del
        numero civico.

        NumeroCivico: numero civico relativo all’indirizzo specificato
        nell’elemento precedente; si può omettere se già riportato
        nell’elemento precedente.
```

<!-- PDF page 35 -->

```text
        CAP: Codice di Avviamento Postale relativo all’indirizzo.

        Comune: comune cui si riferisce l’indirizzo.

        Provincia: sigla della provincia di appartenenza del comune.

        Nazione: codice della nazione.

IscrizioneREA (gli elementi indicati di seguito devono essere
obbligatoriamente valorizzati nei soli casi di società soggette al vincolo
dell’iscrizione nel registro delle imprese ai sensi dell’art. 2250 del codice
civile)

        Ufficio: sigla della provincia ove ha sede l'Ufficio del Registro
        delle Imprese presso il quale è registrata la società.

        NumeroREA: numero di repertorio con il quale la società è
        iscritta nel Registro delle Imprese.

        CapitaleSociale: indica il capitale sociale quale somma
        effettivamente versata e quale risulta esistente dall’ultimo
        bilancio della società; questo campo è valorizzato nei soli casi
        di società di capitali (SpA, SApA, SRL).

        SocioUnico: questo elemento è valorizzato nei casi di società
        per azioni e a responsabilità limitata; indica se queste si
        compongono di un unico socio o di più soci.

        StatoLiquidazione: indica se la società si trova in stato di
        liquidazione oppure no.

Contatti

        Telefono: contatto         telefonico    fisso    /   mobile     del
        cedente/prestatore.

        Fax: numero di fax del cedente/prestatore.

        Email: indirizzo di posta elettronica del cedente/prestatore.

 RiferimentoAmministrazione: eventuale riferimento utile al
 destinatario per automatizzare la gestione amministrativa
 dell’operazione in fattura.
```

<!-- PDF page 36 -->

```text
 2.1.3    FATTURA        ELETTRONICA         HEADER        –     DATI     DEL
          RAPPRESENTANTE FISCALE DEL CEDENTE/PRESTATORE

Da valorizzare qualora il cedente/prestatore si avvalga di un rappresentante
fiscale in Italia, ai sensi del DPR 633 del 1972 e successive modifiche ed
integrazioni.

<RappresentanteFiscale>

   DatiAnagrafici

            IdFiscaleIVA (i seguenti elementi consentono l’inserimento
            della   partita  IVA del     rappresentante fiscale    del
            cedente/prestatore)

                   IdPaese: codice del paese assegnante l’identificativo
                   fiscale al soggetto rappresentante.

                   IdCodice: numero di identificazione fiscale del
                   rappresentante fiscale. In caso di IdPaese uguale a IT, il
                   sistema ne verifica la presenza in Anagrafe Tributaria: se
                   non esiste come partita IVA, il file viene scartato con
                   codice errore 00303.

            CodiceFiscale: numero di codice fiscale del rappresentante
            fiscale del cedente/prestatore. Il sistema ne verifica la presenza
            in Anagrafe Tributaria: se non esiste come codice fiscale, il file
            viene scartato con codice errore 00304.

            Anagrafica

                   Denominazione: ditta, denominazione o ragione sociale
                   del rappresentante fiscale del cedente/prestatore da
                   valorizzare nei casi di persona non fisica.

                   Nome: nome del rappresentante fiscale del
                   cedente/prestatore da valorizzare nei casi di persona
                   fisica.

                   Cognome: cognome del rappresentante fiscale del
                   cedente/prestatore da valorizzare nei casi di persona
                   fisica.

                   Titolo: titolo onorifico del rappresentante fiscale del
                   cedente/prestatore.
```

<!-- PDF page 37 -->

```text
                 CodEORI: numero del Codice EORI (Economic
                 Operator Registration and Identification) in base al
                 Regolamento (CE) n. 312 del 16 aprile 2009. In vigore
                 dal 1° luglio 2009 tale codice identifica gli operatori
                 economici nei rapporti con le autorità doganali sull'intero
                 territorio dell'Unione Europea.

 2.1.4   FATTURA        ELETTRONICA            HEADER         –     DATI     DEL
         CESSIONARIO/COMMITTENTE

<CessionarioCommittente>

   DatiAnagrafici

          IdFiscaleIVA (la valorizzazione di questi elementi è in
          alternativa non esclusiva a quella dell’elemento CodiceFiscale;
          in altri termini, possono non essere valorizzati se è valorizzato
          l’elemento CodiceFiscale; è invece obbligatoria nei casi in cui
          l’elemento TipoDocumento assume il valore TD16, TD17,
          TD18, TD19, TD20, TD22, TD23, TD28 e TD29 altrimenti il file
          viene scartato con codice errore 00475; se non è valorizzato e
          non è valorizzato neanche l’elemento CodiceFiscale, il file viene
          scartato con codice errore 00417; gli elementi di questo blocco
          consentono l’inserimento della partita IVA italiana per i soggetti
          residenti in Italia e per quelli esteri operanti in Italia identificati:

             ✓ attraverso una Stabile Organizzazione

             ✓ mediante un rappresentante fiscale

             ✓ direttamente.

          Negli altri casi consentono l’inserimento dell’identificativo
          fiscale assegnato dall’autorità del proprio paese per i soggetti
          non residenti)

                 IdPaese: codice del paese assegnante l’identificativo
                 fiscale al soggetto cessionario/committente. Se ha un
                 valore diverso da IT, il sistema verifica che non sia
                 diverso da IT anche l’IdPaese del cedente/prestatore; in
                 caso contrario il file viene scartato con codice 00476.

                 IdCodice: numero di identificazione fiscale del
                 cessionario/committente. In caso di IdPaese uguale a IT,
                 il sistema ne verifica la presenza in Anagrafe Tributaria:
```

<!-- PDF page 38 -->

```text
       se non esiste come partita IVA, il file viene scartato con
       codice errore 00305.

CodiceFiscale: la valorizzazione di questo elemento è in
alternativa non esclusiva a quella dell’elemento IdFiscaleIVA
(può non essere valorizzato se è valorizzato l’elemento
IdFiscaleIVA; se non è valorizzato né l’uno né l’altro, il file viene
scartato con codice errore 00417). Consente l’inserimento del
numero di codice fiscale del cessionario/committente. Il sistema
ne verifica la presenza in Anagrafe Tributaria: se non esiste
come codice fiscale, il file viene scartato con codice errore
00306.

Nel caso di IdFiscaleIVA di gruppo IVA, il sistema verifica che
sia presente anche il CodiceFiscale e che appartenga a uno dei
partecipanti al gruppo; se il CodiceFiscale non è presente il file
viene scartato con codice errore 00326; se è presente ma non
è di partecipante al gruppo IVA, il file viene scartato con codice
errore 00325.

Nel caso di CodiceFiscale di gruppo IVA, in assenza di
IdFiscaleIVA, il sistema verifica che il CodiceFiscale
appartenga a uno dei partecipanti al gruppo; se il CodiceFiscale
è presente ma appartiene al gruppo IVA e non a uno dei
partecipanti, il file viene scartato con codice errore 00327.

Nel caso di IdFiscaleIVA non di gruppo IVA e di presenza anche
del CodiceFiscale, il sistema controlla che facciano riferimento
allo stesso soggetto; in caso contrario il file viene scartato con
codice errore 00324.

Anagrafica

       Denominazione: ditta, denominazione o ragione sociale
       del cessionario/committente da valorizzare nei casi di
       persona non fisica.

       Nome: nome del cessionario/committente da valorizzare
       nei casi di persona fisica.

       Cognome: cognome del cessionario/committente da
       valorizzare nei casi di persona fisica.

       Titolo: titolo onorifico del cessionario/committente.

       CodEORI: numero del Codice EORI (Economic
       Operator Registration and Identification) in base al
       Regolamento (CE) n. 312 del 16 aprile 2009. In vigore
       dal 1° luglio 2009 tale codice identifica gli operatori
       economici nei rapporti con le autorità doganali sull'intero
```

<!-- PDF page 39 -->

```text
               territorio dell'Unione Europea.

Sede (nel caso di somministrazione di servizi quali energia elettrica, gas
etc., i dati possono fare riferimento all’ubicazione dell’utenza, ex DM
370/2000)

        Indirizzo: indirizzo del cessionario/committente; deve essere
        valorizzato con il nome della via, piazza, etc., comprensivo, se
        si vuole, del numero civico.

        NumeroCivico: numero civico relativo all’indirizzo specificato
        nell’elemento precedente; si può omettere se già riportato
        nell’elemento precedente.

        CAP: Codice di Avviamento Postale relativo all’indirizzo.

        Comune: comune cui si riferisce l’indirizzo.

        Provincia: sigla della provincia di appartenenza del comune.

        Nazione: codice della nazione espresso secondo lo standard
        ISO 3166-1 alpha-2 code.

StabileOrganizzazione (gli elementi indicati di seguito devono essere
valorizzati nei soli casi in cui il cessionario/committente è un soggetto
non residente e, per l’operazione oggetto del documento, si avvale di
organizzazione residente sul territorio nazionale. Si riferiscono alla
stabile organizzazione in Italia)

        Indirizzo: indirizzo della stabile organizzazione del
        cessionario/committente del bene/servizio; deve essere
        valorizzato con il nome della via, piazza, etc., comprensivo, se
        si vuole, del numero civico.

        NumeroCivico: numero civico relativo all’indirizzo specificato
        nell’elemento precedente; si può omettere se già riportato
        nell’elemento precedente.

        CAP: Codice di Avviamento Postale relativo all’indirizzo.

        Comune: comune cui si riferisce l’indirizzo.

        Provincia: sigla della provincia di appartenenza del comune.

        Nazione: codice della nazione.
```

<!-- PDF page 40 -->

```text
     RappresentanteFiscale

           IdFiscaleIVA (i seguenti elementi consentono l’inserimento
           della   partita IVA del      rappresentante fiscale    del
           cessionario/committente)

                 IdPaese: codice del paese assegnante l’identificativo
                 fiscale al soggetto rappresentante fiscale.

                 IdCodice: numero di         identificazione   fiscale del
                 rappresentante fiscale.

           Denominazione: ditta, denominazione o ragione sociale del
           rappresentante fiscale del cessionario/committente da
           valorizzare nei casi di persona non fisica.

           Nome:      nome     del    rappresentante    fiscale    del
           cessionario/committente da valorizzare nei casi di persona
           fisica.

           Cognome: cognome del rappresentante fiscale del
           cessionario/committente da valorizzare nei casi di persona
           fisica.

 2.1.5   FATTURA     ELETTRONICA        HEADER     – DATI      DEL   TERZO
         INTERMEDIARIO SOGGETTO EMITTENTE

<TerzoIntermediarioOSoggettoEmittente>

   DatiAnagrafici

           IdFiscaleIVA (questi elementi consentono l’inserimento della
           partita IVA per i soggetti residenti in Italia oppure
           dell’identificativo fiscale assegnato dall’autorità del proprio
           paese per i soggetti non residenti)

                 IdPaese: codice del paese assegnante l’identificativo
                 fiscale al soggetto terzo intermediario.

                 IdCodice: numero di identificazione fiscale del terzo
                 intermediario.

           CodiceFiscale: numero di codice fiscale del terzo intermediario
           soggetto emittente.

           Anagrafica
```

<!-- PDF page 41 -->

```text
                  Denominazione: ditta, denominazione o ragione sociale
                  del terzo intermediario soggetto emittente qualora si tratti
                  di persona non fisica.

                  Nome: nome del terzo intermediario soggetto emittente
                  qualora si tratti di persona fisica.

                  Cognome: cognome del terzo intermediario soggetto
                  emittente qualora si tratti di persona fisica.

                  Titolo: titolo onorifico del terzo intermediario soggetto
                  emittente.

                  CodEORI: numero del Codice EORI (Economic
                  Operator Registration and Identification) in base al
                  Regolamento (CE) n. 312 del 16 aprile 2009. In vigore
                  dal 1° luglio 2009 tale codice identifica gli operatori
                  economici nei rapporti con le autorità doganali sull'intero
                  territorio dell'Unione Europea.

 2.1.6   FATTURA ELETTRONICA HEADER – SOGGETTO EMITTENTE
Nei casi di documenti emessi da un soggetto diverso dal cedente/prestatore
va valorizzato l’elemento seguente.

   SoggettoEmittente: codice che sta ad indicare se la fattura è stata
   emessa da parte del cessionario/committente ovvero da parte di un
   terzo per conto del cedente/prestatore.

 2.1.7   FATTURA ELETTRONICA BODY – DATI GENERALI
<Dati Generali>

   DatiGeneraliDocumento

           TipoDocumento: tipologia del documento oggetto della
           trasmissione      (fattura, acconto/anticipo    su     fattura,
           acconto/anticipo su parcella, nota di credito, nota di debito,
           parcella, autofattura).

           Divisa: tipo di valuta utilizzata per l'indicazione degli importi.

           Data: data del documento.
```

<!-- PDF page 42 -->

```text
Numero: numero progressivo attribuito dal cedente/prestatore
al documento. Deve contenere almeno un carattere numerico.
In caso contrario il file viene scartato con codice errore 00425.

DatiRitenuta (i seguenti elementi vanno valorizzati nei casi in
cui sia applicabile la ritenuta o il contributo previdenziale che
non è rappresentabile come cassa)

      TipoRitenuta: tipologia di ritenuta (persone fisiche,
      persone giuridiche, contributo previdenziale).

      ImportoRitenuta: importo della ritenuta.

      AliquotaRitenuta: aliquota (espressa in percentuale %)
      della ritenuta.

      CausalePagamento: codice della causale del
      pagamento (il codice corrisponde a quello utilizzato per
      la compilazione del modello CU).

DatiBollo (nei casi in cui sia prevista l’imposta di bollo va
valorizzato BolloVirtuale, mentre ImportoBollo è opzionale)

      BolloVirtuale: indica l’assolvimento dell’imposta di bollo
      ai sensi del decreto MEF 17 giugno 2014 (bollo virtuale).

      ImportoBollo: importo dell’imposta di bollo.

DatiCassaPrevidenziale (i seguenti elementi vanno valorizzati
nei casi in cui sia previsto il contributo cassa previdenziale)

      TipoCassa: cassa di previdenza della categoria
      professionale di appartenenza.

      AlCassa: aliquota contributiva (espressa in percentuale
      %) prevista per la cassa di previdenza.

      ImportoContributoCassa: importo del contributo
      relativo alla cassa di previdenza della categoria
      professionale.

      ImponibileCassa: importo totale del volume di affari sul
      quale occorre applicare il contributo di cassa
      previdenziale.
```

<!-- PDF page 43 -->

```text
       AliquotaIVA: IVA applicata al contributo cassa
       previdenziale. Va espressa in termini percentuali (es.: il
       10% si esprime come 10.00 e non come 0.10), altrimenti
       il file viene scartato con codice errore 00424. Nel caso di
       non applicabilità, l’elemento deve essere valorizzato a
       zero: se valorizzato a zero il sistema verifica che sia
       presente l’elemento Natura; qualora assente, il file viene
       scartato con codice errore 00413.

       Ritenuta: indica se il contributo cassa è soggetto a
       ritenuta. Se soggetta (elemento valorizzato con SI) il
       sistema controlla la presenza del blocco DatiRitenuta di
       cui sopra: se questo blocco è assente, il file viene
       scartato con codice errore 00415.

       Natura: codice che esprime la natura della non
       imponibilità del contributo cassa. Deve essere presente
       nel solo caso in cui l’elemento AliquotaIVA vale zero. Se
       è presente a fronte di un valore dell’elemento
       AliquotaIVA diverso da zero, il file viene scartato con
       codice errore 0414.

       RiferimentoAmministrazione: eventuale riferimento
       utile al destinatario per automatizzare la gestione
       amministrativa dell’operazione in fattura (capitolo di
       spesa, conto economico …)

ScontoMaggiorazione

       Tipo: indica se si tratta di sconto o di maggiorazione.

       Percentuale: percentuale di sconto o di maggiorazione.

       Importo: importo dello sconto o della maggiorazione.

ImportoTotaleDocumento: importo totale del documento
comprensivo di imposta a debito del cessionario/committente.

Arrotondamento: importo dell’arrotondamento             sul totale
documento, qualora presente.

Causale: descrizione della causale del documento.

Art73: indica se il documento è stato emesso secondo modalità
e termini stabiliti con decreto ministeriale ai sensi dell'articolo
73 del DPR 633/72 (ciò consente al cedente/prestatore
```

<!-- PDF page 44 -->

```text
       l'emissione nello stesso anno di più documenti aventi stesso
       numero).

DatiOrdineAcquisto (dati relativi all’ordine di acquisto dal quale
scaturisce la cessione/prestazione oggetto del documento fattura)

       RiferimentoNumeroLinea: numero della linea o delle linee di
       dettaglio della fattura alle quali si riferisce l’ordine di acquisto
       così come identificato dai tre elementi successivi
       (IdDocumento, Data, NumItem); nel caso in cui l’ordine di
       acquisto si riferisce all’intera fattura, questo elemento non deve
       essere valorizzato.

       IdDocumento: numero dell’ordine di acquisto associato alla
       fattura o alla linea/linee di fattura indicate nell’elemento
       RiferimentoNumeroLinea.

       Data: data dell’ordine di acquisto associato alla fattura o alla
       linea/linee     di     fattura      indicate      nell’elemento
       RiferimentoNumeroLinea.

       NumItem: identificativo della singola voce (linea di ordine)
       all'interno dell’ordine di acquisto associata alla fattura o alla
       linea/linee       di      fattura    indicate      nell’elemento
       RiferimentoNumeroLinea.

       CodiceCommessaConvenzione: codice della commessa o
       della convenzione collegata alla fattura.

       CodiceCUP: codice gestito dal CIPE che caratterizza ogni
       progetto di investimento pubblico (Codice Unitario Progetto).

       CodiceCIG: Codice Identificativo della Gara.

DatiContratto (dati relativi al contratto dal quale scaturisce la
cessione/prestazione oggetto del documento fattura)

       RiferimentoNumeroLinea: numero della linea o delle linee di
       dettaglio della fattura alle quali si riferisce il contratto così come
       identificato dai tre elementi successivi (IdDocumento, Data,
       NumItem); nel caso in cui il contratto si riferisce all’intera fattura,
       questo elemento non deve essere valorizzato.

       IdDocumento: numero del contratto associato alla fattura o alla
       linea/linee   di     fattura     indicate       nell’elemento
       RiferimentoNumeroLinea.
```

<!-- PDF page 45 -->

```text
        Data: data del contratto associato alla fattura o alla linea/linee
        di fattura indicate nell’elemento RiferimentoNumeroLinea.

        NumItem: identificativo della singola voce (linea di contratto)
        all'interno del contratto associata alla fattura o alla linea/linee di
        fattura indicate nell’elemento RiferimentoNumeroLinea.

        CodiceCommessaConvenzione: codice della commessa o
        della convenzione collegata alla fattura.

        CodiceCUP: codice gestito dal CIPE che caratterizza ogni
        progetto di investimento pubblico (Codice Unitario Progetto).

        CodiceCIG: Codice Identificativo della Gara.

DatiConvenzione (dati relativi alla convenzione collegata alla fattura)

        RiferimentoNumeroLinea: numero della linea o delle linee di
        dettaglio della fattura alle quali si riferisce la convenzione così
        come identificata dai tre elementi successivi (IdDocumento,
        Data, NumItem); nel caso in cui la convenzione si riferisce
        all’intera fattura, questo elemento non deve essere valorizzato.

        IdDocumento: numero della convenzione associata alla fattura
        o alla linea/linee di fattura indicate nell’elemento
        RiferimentoNumeroLinea.

        Data: data della convenzione associata alla fattura o alla
        linea/linee   di     fattura   indicate      nell’elemento
        RiferimentoNumeroLinea.

        NumItem: identificativo della singola voce (linea di
        convenzione) all'interno della convenzione associata alla
        fattura o alla linea/linee di fattura indicate nell’elemento
        RiferimentoNumeroLinea.

        CodiceCommessaConvenzione: codice della commessa o
        della convenzione collegata alla fattura.

        CodiceCUP: codice gestito dal CIPE che caratterizza ogni
        progetto di investimento pubblico (Codice Unitario Progetto).

        CodiceCIG: Codice Identificativo della Gara.

DatiRicezione (dati relativi alla ricezione dei beni/servizi oggetto del
documento fattura)
```

<!-- PDF page 46 -->

```text
        RiferimentoNumeroLinea: numero della linea o delle linee di
        dettaglio della fattura alle quali si riferisce la ricezione così come
        identificata dai tre elementi successivi (IdDocumento, Data,
        NumItem); nel caso in cui la ricezione si riferisce all’intera fattura,
        questo elemento non deve essere valorizzato.

        IdDocumento: numero della ricezione associata alla fattura o
        alla   linea/linee di   fattura   indicate   nel     campo
        RiferimentoNumeroLinea.

        Data: data della ricezione associata alla fattura o alla linea/linee
        di fattura indicate nell’elemento RiferimentoNumeroLinea.

        NumItem: identificativo della singola voce (linea di ricezione)
        all'interno della ricezione associata alla fattura o alla linea/linee
        di fattura indicate nell’elemento RiferimentoNumeroLinea.

        CodiceCommessaConvenzione: codice della commessa o
        della convenzione collegata alla fattura.

        CodiceCUP: codice gestito dal CIPE che caratterizza ogni
        progetto di investimento pubblico (Codice Unitario Progetto).

        CodiceCIG: Codice Identificativo della Gara.

DatiFattureCollegate (dati relativi alla fattura alla quale si collega il
documento in oggetto)

        RiferimentoNumeroLinea: numero della linea o delle linee di
        dettaglio del documento alle quali si riferisce la fattura collegata
        così come identificata dai tre elementi successivi
        (IdDocumento, Data, NumItem); nel caso in cui la fattura
        collegata si riferisce all’intero documento, questo elemento non
        deve essere valorizzato.

        IdDocumento: numero della fattura collegata associata al
        documento o alla linea/linee del documento indicate
        nell’elemento RiferimentoNumeroLinea.

        Data: data della fattura collegata associata al documento o alla
        linea/linee   del      documento      indicate     nell’elemento
        RiferimentoNumeroLinea. Non può essere mai successiva alla
        data del documento in oggetto; in caso contrario il file viene
        scartato con codice errore 00418.
```

<!-- PDF page 47 -->

```text
        NumItem: identificativo della singola voce (linea di fattura
        collegata) all'interno della fattura collegata associata al
        documento o alla linea/linee del documento indicate
        nell’elemento RiferimentoNumeroLinea.

        CodiceCommessaConvenzione: codice della commessa o
        della convenzione collegata alla fattura.

        CodiceCUP: codice gestito dal CIPE che caratterizza ogni
        progetto di investimento pubblico (Codice Unitario Progetto).

        CodiceCIG: Codice Identificativo della Gara.

DatiSAL

        RiferimentoFase: fase dello stato avanzamento cui la fattura
        si riferisce.

DatiDDT (nei casi in cui sia presente un documento di trasporto
collegato alla fattura, casi di fatturazione differita, vanno valorizzati i
seguenti elementi per ogni documento di trasporto)

        NumeroDDT: numero del Documento Di Trasporto.

        DataDDT: data del Documento Di Trasporto (espressa
        secondo il formato ISO 8601:2004).

        RiferimentoNumeroLinea: numero della linea o delle linee di
        dettaglio della fattura alle quali si riferisce il DDT (così come
        identificato dagli elementi NumeroDDT e DataDDT); nel caso in
        cui il documento di trasporto si riferisce all’intera fattura, questo
        elemento non deve essere valorizzato.

DatiTrasporto

        DatiAnagraficiVettore

                IdFiscaleIVA (questi elementi consentono l’inserimento
                della partita IVA per i soggetti residenti in Italia oppure
                dell’identificativo fiscale assegnato dall’autorità del
                proprio paese per i soggetti non residenti)

                       IdPaese: codice del paese assegnante
                       l’identificativo fiscale al soggetto vettore.
```

<!-- PDF page 48 -->

```text
             IdCodice: numero di identificazione fiscale del
            vettore.

      CodiceFiscale: numero di codice fiscale del vettore.

      Anagrafica

             Denominazione: ditta, denominazione o ragione
             sociale del vettore qualora si tratti di persona non
             fisica.

             Nome: nome del vettore qualora si tratti di
             persona fisica.

             Cognome: cognome del vettore qualora si tratti di
             persona fisica.

             Titolo: titolo onorifico del vettore.

             CodEORI: numero del Codice EORI (Economic
             Operator Registration and Identification) in base
             al Regolamento (CE) n. 312 del 16 aprile 2009. In
             vigore dal 1° luglio 2009 tale codice identifica gli
             operatori economici nei rapporti con le autorità
             doganali sull'intero territorio dell'Unione Europea.

      NumeroLicenzaGuida: numero identificativo            della
      licenza di guida (es. numero patente).

MezzoTrasporto: mezzo utilizzato per il trasporto.

CausaleTrasporto: causale del trasporto.

NumeroColli: numero dei colli trasportati.

Descrizione: descrizione (natura, qualità, aspetto …) relativa
ai colli trasportati.

UnitaMisuraPeso: unità di misura riferita al peso della merce
trasportata.

PesoLordo: peso lordo della merce.

PesoNetto: peso netto della merce.

DataOraRitiro: data e ora del ritiro della merce.
```

<!-- PDF page 49 -->

```text
           DataInizioTrasporto: data di inizio del trasporto.

           TipoResa: codice che identifica la tipologia di resa.

           IndirizzoResa

                Indirizzo: indirizzo relativo alla resa (nome della via,
                piazza, etc. ed eventuale numero civico).

                NumeroCivico: numero civico dell’indirizzo relativo alla
                resa, se non presente nel campo precedente.

                CAP: codice di avviamento postale dell’indirizzo relativo
                alla resa.

                Comune: comune cui si riferisce l’indirizzo relativo alla
                resa.

                Provincia: sigla della provincia di appartenenza del
                comune.

                Nazione: codice della nazione.

           DataOraConsegna: data e ora della consegna della merce.

   FatturaPrincipale (presente nei casi di fatture per operazioni
   accessorie emesse dagli ‘autotrasportatori’ per usufruire delle
   agevolazioni in materia di registrazione e pagamento IVA)

           NumeroFatturaPrincipale: numero della fattura relativa al
           trasporto di beni, da indicare sulle fatture emesse dagli
           autotrasportatori per certificare le operazioni accessorie.

           DataFatturaPrincipale: data della fattura principale.

 2.1.8   FATTURA ELETTRONICA BODY – DATI BENI / SERVIZI
<DatiBeniServizi>

   DettaglioLinee

           NumeroLinea: numero che identifica la linea di dettaglio del
           bene/servizio riportata sul documento.
```

<!-- PDF page 50 -->

```text
TipoCessionePrestazione: codice che identifica la tipologia di
cessione/prestazione qualora si tratti di sconto, premio,
abbuono, spesa accessoria; è quindi valorizzabile soltanto in
presenza di questi casi.

CodiceArticolo

       CodiceTipo: indica la tipologia di codice articolo (i.e.:
       TARIC, CPV, EAN, SSC, ...). Se la fattura è relativa alla
       vendita di gasolio o benzina, l’elemento deve essere
       valorizzato con la dicitura “CARB”.

       CodiceValore: valore del codice articolo corrispondente
       alla tipologia. Se l’elemento CodiceTipo è uguale a
       “CARB”, l’elemento CodiceValore deve contenere uno
       dei seguenti codici:

          -   “27101245” (per vendita di Benzina senza piombo
              ottani => 95 e < 98);

          -   “27101249” (per vendita di Benzina senza piombo
              ottani => 98);

          -   “27101942” (per “Olii da gas, destinati ad altri usi,
              aventi tenore, in peso, di zolfo inferiore o uguale
              a 0,001% - Con un tenore di carbonio di origine
              biologica pari ad almeno l'80 % in peso”);

          -   “27101944” (per “Olii da gas, destinati ad altri usi,
              aventi tenore, in peso, di zolfo inferiore o uguale
              a 0,001% - Altri”, per indicare gasolio con
              carbonio di origine non biologica);

          -   “27102011” (per vendita di Olio da gas denaturato
              tenore in peso di zolfo nell'olio da gas =< 0,001%)

       riportati nella tabella di riferimento per i prodotti
       energetici TA13 – pubblicata sul sito dell’Agenzia delle
       Dogane

Descrizione: natura e qualità del bene/servizio oggetto della
cessione/prestazione; può fare anche riferimento ad un
precedente documento emesso a titolo di anticipo/acconto.

Quantita: numero di unità cedute/prestate; può non essere
valorizzato nei casi in cui la prestazione non sia quantificabile.
```

<!-- PDF page 51 -->

```text
UnitaMisura: unità di misura in cui è espresso il campo
Quantità.

DataInizioPeriodo: data iniziale del periodo di riferimento cui
si riferisce l'eventuale servizio prestato.

DataFinePeriodo: data finale del periodo di riferimento cui si
riferisce l'eventuale servizio prestato.

PrezzoUnitario: prezzo unitario del bene/servizio; nel caso di
beni ceduti a titolo di sconto, premio o abbuono, l'importo
indicato rappresenta il "valore normale”.

ScontoMaggiorazione

      Tipo: indica se si tratta di sconto o di maggiorazione.

      Percentuale: percentuale di sconto o di maggiorazione.

      Importo: importo dello sconto o della maggiorazione.

PrezzoTotale: importo totale del bene/servizio. Il valore
indicato deve corrispondere al prodotto del precedente
elemento PrezzoUnitario, rettificato in diminuzione o in
aumento        in    base     alla   presenza       dell’elemento
ScontoMaggiorazione, per l’elemento Quantita. Se non viene
verificata la suddetta corrispondenza e la differenza è superiore
a ±1 centesimo di euro (tolleranza ammessa), il file viene
scartato con codice errore 00423.

AliquotaIVA: IVA applicata alla cessione/prestazione. Va
espressa in termini percentuali (es.: il 10% si esprime come
10.00 e non come 0.10), altrimenti il file viene scartato con
codice errore 00424. Nel caso di non applicabilità, il campo
deve essere valorizzato a zero: se valorizzato a zero il sistema
verifica che sia presente l’elemento Natura, altrimenti il file
viene scartato con codice errore 00400.

Ritenuta: indica se la linea della fattura si riferisce ad una
cessione/prestazione soggetta a ritenuta. Se soggetta
(elemento valorizzato con SI) il sistema controlla la presenza
del blocco DatiRitenuta di cui al paragrafo 2.1.7: se questo
blocco è assente, il file viene scartato con codice errore 00411.

Natura: codice che esprime la natura delle operazioni che non
rientrano tra quelle imponibili o i casi di inversione contabile.
```

<!-- PDF page 52 -->

```text
                        Deve essere presente nel solo caso in cui l’elemento
                        AliquotaIVA vale zero. Se è presente a fronte di un valore
                        dell’elemento AliquotaIVA diverso da zero, il file viene scartato
                        con codice errore 00401. Nel caso in cui, invece, l’elemento
                        TipoDocumento assume il valore TD16, è possibile indicare sia
                        la natura delle operazioni, sia un valore diverso da zero per
                        l’elemento AliquotaIVA.

                        RiferimentoAmministrazione: eventuale riferimento utile al
                        destinatario per automatizzare la gestione amministrativa
                        dell’operazione in fattura (capitolo di spesa, conto economico
                        …)

                        AltriDatiGestionali
                        Questo blocco informativo, facoltativo, permette di
                        rappresentare dati utili alla gestione automatica della fattura
                        elettronica e dei dati in essa contenuti. Gli elementi che lo
                        compongono possono per esempio essere utilizzati per
                        scambio di informazioni pattuite tra le parti. È possibile inserire
                        più blocchi AltriDatiGestionali per ciascuna riga di fattura (non
                        esiste un limite fissato del numero di blocchi che si possono
                        inserire ma l’intera fattura non può “pesare” di 5 MB).
                        Per specifiche operazioni gli elementi che compongono questo
                        blocco possono essere usati per ottemperare a determinate
                        previsioni di norma, come indicato nel seguito.

                                TipoDato: codice           che     identifica    la    tipologia    di
                                informazione.

                                Al fine di riportare il dato della targa in fattura (es. d.P.R.
                                del 09/06/2000 n. 277), l’elemento TipoDato va
                                valorizzato con la stringa “TARGA”.

                                Al fine di riportare in fattura il riferimento ad uno
                                scontrino7 o ad un documento commerciale8, nei casi in
                                cui    l’operazione     “fatturata” sia    stata  così
                                precedentemente rendicontata, l’elemento TipoDato va
                                valorizzato con la stringa “N.DOC.COMM”.

                                Al fine di riportare in fattura il riferimento ad operazioni di
                                cui all’articolo 21 comma 6 bis lettera a) del d.P.R. n.

7 Di cui al DM 30.03.1992
8 Definito nel Provvedimento del Direttore dell’Agenzia delle entrate del 28.10.2016
```

<!-- PDF page 53 -->

```text
633/72, l’elemento TipoDato va valorizzato con la stringa
“INVCONT”.

Al fine di riportare in fattura il riferimento ad operazioni
non imponibili, ai sensi dell’art. 8, comma 1, lett. c) del
decreto del Presidente della Repubblica 26 ottobre 1972,
n. 633 e da trasmettere al sistema SdI, nei confronti di
un esportatore abituale, l’elemento TipoDato va
valorizzato con la stringa “INTENTO”.

Al fine di riportare in fattura l’informazione circa
l’avvenuta regolarizzazione dell’imposta a debito con
versamento tramite modello di versamento F24 in sede
di liquidazione periodica dell’IVA, l’elemento TipoDato va
valorizzato con la stringa “F24”.

Nel caso di operazioni di estrazione beni da deposito
IVA, al fine di riportare in fattura il riferimento al corretto
periodo di imposta dell’operazione, l’elemento TipoDato
va valorizzato con la stringa:

   •   “NellAnno” nel caso in cui l’estrazione dal
       deposito IVA avvenga nello stesso periodo
       d’imposta in cui è stata effettuata l’immissione o
       l’acquisto del bene custodito in deposito;

   •   “AnniPreced” nel caso in cui l’estrazione dal
       deposito avvenga in un periodo d’imposta
       successivo a quello in cui è stato effettuato
       l’acquisto senza pagamento d’imposta.

Nel caso si volessero rappresentare operazioni che,
seppur formalmente rientranti nel campo applicativo
dell’imposta di bollo, non devono essere assoggettate
all’imposta per via di specifiche disposizioni normative,
l’elemento TipoDato va valorizzato:

   •   con la stringa “NB1” se l’imposta di bollo non è
       applicabile ai documenti assicurativi in quanto
       assorbita nell’imposta sulle assicurazioni;

   •   con la stringa “NB2” se l’imposta di bollo non è
       applicabile ai documenti emessi da soggetti
       appartenenti al terzo settore;
```

<!-- PDF page 54 -->

```text
   •   con la stringa “NB3” se l’imposta di bollo non è
       applicabile ai documenti emessi fra banca e
       cliente correntista, poiché l’imposta di bollo su tali
       documenti è già assorbita nell’imposta di bollo
       sull’estratto conto.

   Nel caso di cessioni di beni e servizi effettuati da un
   produttore agricolo in regime speciale di cui
   all’articolo 34 del d.P.R. del 1972 n. 633, l’elemento
   TipoDato può essere valorizzato, in maniera
   facoltativa utile per la gestione automatica della
   liquidazione IVA:

   •   con la stringa “ALI-COMP”, nel caso di cessioni di
       prodotti agricoli e ittici compresi nella parte prima
       della Tabella A del d.P.R. del 1972 n. 633 (in
       presenza di tale stringa deve essere valorizzato
       anche l’elemento RiferimentoNumero con la
       percentuale di compensazione applicabile ai
       prodotti agricoli o ittici ceduti);

   •   con la stringa “NO-COMP”, nel caso di cessioni di
       prodotti agricoli e ittici non compresi nella parte
       prima della Tabella A del d.P.R. del 1972 n. 633;

   •   con la stringa “OCC34BIS”, nel caso di operazioni
       occasionali rientranti nel regime di cui all’articolo
       34-bis del d.P.R. del 1972 n. 633 effettuate dal
       produttore agricolo.

    Al fine di riportare in fattura il riferimento a compensi
    riferiti all’ambito del lavoro sportivo dilettantistico di
    cui all’articolo 36, comma 6 del decreto legislativo 36
    del 2021, i quali godono di un’esenzione
    dall’imponibile fino a 15.000,00 euro annui,
    l’elemento TipoDato può essere valorizzato con la
    stringa “ESENZSPORT”.

RiferimentoTesto: valore alfanumerico riferito alla
tipologia di informazione.
Al fine di riportare il dato della targa in fattura (es. d.P.R.
del 09/06/2000 n. 277), l’elemento RiferimentoTesto va
valorizzato con la targa del mezzo per il quale si effettua
il rifornimento.

Al fine di riportare in fattura il riferimento ad uno scontrino
o ad un documento commerciale, nei casi in cui
l’operazione “fatturata” sia stata così precedentemente
```

<!-- PDF page 55 -->

```text
              rendicontata, l’elemento RiferimentoTesto va valorizzato
              con l’identificativo alfanumerico del documento
              commerciale.

              Al fine di riportare in fattura il riferimento ad una
              dichiarazione di intento, l’elemento RiferimentoTesto va
              valorizzato riportando il protocollo di ricezione della
              dichiarazione d’intento e il suo progressivo separato dal
              segno "-" oppure dal segno "/" (es. 08060120341234567-
              000001). Qualora la dichiarazione d’intento indicata
              risultasse invalidata, la fattura verrà scartata con codice
              d’errore 00477.

              RiferimentoNumero: valore numerico riferito alla
              tipologia di informazione.

              Nel caso di cessioni di prodotti agricoli e ittici compresi
              nella parte prima della Tabella A del d.P.R. del 1972 n.
              633 effettuate da un produttore agricolo in regime
              speciale di cui all’articolo 34 del medesimo decreto,
              l’elemento RiferimentoNumero va valorizzato con la
              percentuale di compensazione stabilita con decreto del
              Ministro delle finanze di concerto con il Ministro per le
              politiche agricole, per il gruppo di prodotti di riferimento
              del bene ceduto (questo elemento deve essere
              compilato quando il campo TipoDato è valorizzato con
              “ALI-COMP”).

              RiferimentoData: data riferita alla tipologia di
              informazione.
              Al fine di riportare in fattura il riferimento ad uno scontrino
              o ad un documento commerciale, nei casi in cui
              l’operazione “fatturata” sia stata così precedentemente
              rendicontata, l’elemento RiferimentoData va valorizzato
              con la data di emissione dello scontrino (o del documento
              commerciale), secondo il formato ISO 8601:2004, con la
              seguente precisione: YYYY-MM-DD

              Al fine di riportare in fattura il riferimento ad una
              dichiarazione di intento, l’elemento RiferimentoData
              deve riportare la data della ricevuta telematica rilasciata
              dall’Agenzia delle entrate e contenente il protocollo della
              dichiarazione d’intento. Qualora la dichiarazione
              d’intento indicata risultasse invalidata, la fattura verrà
              scartata con codice d’errore 00477.

DatiRiepilogo (dati obbligatori; questo blocco deve essere valorizzato
per ogni aliquota IVA e modalità di versamento dell’imposta (“scissione
```

<!-- PDF page 56 -->

```text
dei pagamenti “ od ordinaria), e/o per ogni natura; in caso contrario il file
viene scartato con codice errore 00443 o 00444)

        AliquotaIVA: aliquota IVA. Va espressa in termini percentuali
        (es.: il 10% si esprime come 10.00 e non come 0.10), altrimenti
        il file viene scartato con codice errore 00424.

        Natura: codice che esprime la natura delle operazioni che non
        rientrano tra quelle imponibili. Se l’elemento AliquotaIVA
        precedente è pari a zero, l’elemento Natura è obbligatorio;
        viceversa non deve essere presente se l’elemento AliquotaIVA
        è diverso da zero ad eccezione del caso in cui l’elemento
        TipoDocumento assume valore TD16; in caso contrario il file
        viene scartato rispettivamente con codice errore 00429 e
        00430. Inoltre, non può assumere valore di tipo N6 se il
        successivo elemento EsigibilitaIVA viene valorizzato con S
        (scissione pagamenti); in caso contrario il file viene scartato con
        codice errore 00420.

        SpeseAccessorie:       corrispettivi   relativi  alle       cessioni
        accessorie, (es. imballaggi etc.) qualora presenti.

        Arrotondamento:         arrotondamento         sull'imponibile     o
        sull’importo, qualora presente.

        ImponibileImporto: valore che rappresenta la base
        imponibile, per le operazioni soggette ad IVA, oppure l’importo
        per le operazioni che non rientrano tra quelle 'imponibili'. Il
        valore indicato deve corrispondere, per ogni aliquota IVA
        distinta, alla somma dei seguenti elementi:

            ✓ PrezzoTotale indicato in DettaglioLinee

            ✓ ImportoContributoCassa            indicato                  in
              DatiCassaPrevidenziale (se presente)

            ✓ Arrotondamento indicato in DatiRiepilogo

        Se non viene verificata la suddetta corrispondenza e la
        differenza è superiore a ±1 euro (tolleranza ammessa), il file
        viene scartato con codice errore 00422.

        Imposta: imposta corrispondente all’applicazione dell’aliquota
        IVA sul relativo imponibile. Il valore indicato deve corrispondere
        al prodotto dei precedenti elementi AliquotaIVA e
        ImponibileImporto, diviso 100, arrotondato alla seconda cifra
        decimale (per difetto se la terza cifra decimale è inferiore a 5,
        per eccesso se la terza cifra decimale è uguale o maggiore di
        5). Se non viene verificata la suddetta corrispondenza e la
        differenza è superiore a ±1 centesimo di euro (tolleranza
        ammessa), il file viene scartato con codice errore 00421.
```

<!-- PDF page 57 -->

```text
            EsigibilitaIVA: codice che esprime il regime di esigibilità
            dell’IVA (differita o immediata) o la modalità di versamento
            dell’imposta (scissione dei pagamenti). Non può assumere
            valore S (scissione dei pagamenti) se il precedente elemento
            Natura viene valorizzato con un codice di tipo N6; in caso
            contrario il file viene scartato con codice errore 00420.

            RiferimentoNormativo: normativa di riferimento (obbligatorio
            nei casi di operazioni di cui all’elemento Natura).

 2.1.9    FATTURA ELETTRONICA BODY – DATI SUI VEICOLI
Presenti nei casi di cessioni tra paesi membri di mezzi di trasporto nuovi.
Dati relativi ai veicoli di cui all’art. 38, comma 4 del DL 331 del 1993.
<DatiVeicoli>

   Data: data di prima immatricolazione o di iscrizione del mezzo di
   trasporto nei pubblici registri.

   TotalePercorso: totale chilometri percorsi, oppure totale ore navigate
   o volate del mezzo di trasporto.

 2.1.10   FATTURA ELETTRONICA BODY – DATI DI PAGAMENTO
<DatiPagamento>

   CondizioniPagamento:        codice   che   identifica   le   condizioni    di
   pagamento.

   DettaglioPagamento

            Beneficiario: estremi anagrafici del beneficiario del
            pagamento (utilizzabile se si intende indicare un beneficiario
            diverso dal cedente/prestatore).

            ModalitaPagamento: codice che identifica le modalità di
            pagamento.

            DataRiferimentoTerminiPagamento:           data     dalla   quale
            decorrono i termini di pagamento.

            GiorniTerminiPagamento: termine di pagamento espresso in
            giorni a partire dalla data di riferimento di cui all’elemento
            DataRiferimentoTerminiPagamento.

            DataScadenzaPagamento: data di scadenza del pagamento.

            ImportoPagamento: importo relativo al pagamento.
```

<!-- PDF page 58 -->

```text
             CodUfficioPostale: codice dell’ufficio postale (nei casi di
             modalità di pagamento che ne presuppongono l’indicazione).

             CognomeQuietanzante: cognome del quietanzante, nei casi
             di modalità di pagamento di “contanti presso tesoreria”.

             NomeQuietanzante: nome del quietanzante, nei casi di
             modalità di pagamento di “contanti presso tesoreria”.

             CFQuietanzante: codice fiscale del quietanzante nei casi di
             modalità di pagamento di “contanti presso tesoreria”.

             TitoloQuietanzante: titolo del quietanzante nei casi di modalità
             di pagamento di “contanti presso tesoreria”.

             IstitutoFinanziario: nome dell'Istituto Finanziario presso il
             quale effettuare il pagamento.

             IBAN: coordinata bancaria internazionale che consente di
             identificare, in maniera standard, il conto corrente del
             beneficiario (International Bank Account Number.)

             ABI: codice ABI (Associazione Bancaria Italiana).

             CAB: codice CAB (Codice di Avviamento Bancario).

             BIC: codice BIC (Bank Identifier Code).

             ScontoPagamentoAnticipato: ammontare dello sconto per
             pagamento anticipato.

             DataLimitePagamentoAnticipato: data limite stabilita per il
             pagamento anticipato.

             PenalitaPagamentiRitardati: ammontare            della   penalità
             dovuta per pagamenti ritardati.

             DataDecorrenzaPenale: data di decorrenza della penale.

             CodicePagamento: codice da utilizzare per la riconciliazione
             degli incassi da parte del cedente/prestatore.

 2.1.11   FATTURA ELETTRONICA BODY – ALLEGATI
<Allegati>

   Nome Attachment: contiene il nome del documento allegato alla
   fattura elettronica.

   AlgoritmoCompressione:         algoritmo    utilizzato   per   comprimere
   l’allegato.
```

<!-- PDF page 59 -->

```text
          FormatoAttachment: formato dell’allegato.

          DescrizioneAttachment: descrizione del documento allegato alla
          fattura elettronica.

          Attachment: contiene il documento allegato alla fattura elettronica.

2.2   PARTE SECONDA

       2.2.1    SCHEMA DEL FILE XML
      L’”XML Schema” è il linguaggio di descrizione del contenuto di un file XML;
      il suo scopo è definire quali elementi sono permessi, quali tipi di dati sono
      ad essi associati e quale relazione gerarchica hanno fra loro gli elementi del
      file. Ciò permette principalmente la validazione del file XML, ovvero la
      verifica che i suoi elementi siano in accordo con la descrizione in linguaggio
      XML Schema. Una XML Schema Definition (XSD) è un esempio concreto
      (instance) di schema XML scritto in linguaggio XML Schema.
      Si richiama l’attenzione su una regola generale da tener presente in fase di
      creazione del file XML: i tag relativi ai campi che, nel rispetto delle
      regole di seguito esposte, si decide di non valorizzare, non devono
      essere presenti all’interno del file.

       2.2.2    L’ELEMENTO RADICE “FATTURA ELETTRONICA”
      Il file XML è interamente delimitato dall’elemento radice chiamato
      FatturaElettronica ed è composto dalla sequenza di 3 Tipi Complessi figli,
      e cioè:

          FatturaElettronicaHeader
          FatturaElettronicaBody
          ds:Signature

      Il primo tipo complesso, FatturaElettronicaHeader, è obbligatorio e può
      essere inserito una sola volta all’interno della fattura elettronica. Contiene i
      seguenti macroblocchi di dati:

          DatiTrasmissione
          CedentePrestatore
          RappresentanteFiscale
          CessionarioCommittente
          TerzoIntermediarioOSoggettoEmittente
          SoggettoEmittente

      Il secondo tipo complesso, FatturaElettronicaBody, è obbligatorio e può
```

<!-- PDF page 60 -->

```text
essere inserito anche N volte all’interno della fattura elettronica nel caso in
cui si intenda spedire un Lotto di fatture. Contiene i seguenti macroblocchi
di dati:

    DatiGenerali
    DatiBeniServizi
    DatiVeicoli
    DatiPagamento
    Allegati

Il terzo tipo, ds:Signature, è opzionale e fa riferimento al seguente
namespace: http://www.w3.org/2000/09/xmldsig#. Serve per la firma con
tecnologia XAdES-Bes.

Una visione generale di quanto descritto sopra si può ottenere dalla figura
che segue:
```

<!-- PDF page 61 -->

```text
     Figura 1: Elemento radice Fattura Elettronica: Header, Body e ds:Signature

Nei paragrafi successivi verranno analizzati e descritti tutti gli elementi dei
due primi “macroblocchi”.

Si richiama l’attenzione su una particolarità grafica delle figure presentate
di seguito: i riquadri a linea continua rappresentano elementi
```

<!-- PDF page 62 -->

```text
OBBLIGATORI, quelli a linea tratteggiata              rappresentano   elementi
FACOLTATIVI.

2.2.3   FATTURA ELETTRONICA HEADER: DATI TRASMISSIONE

                Figura 2: Elemento DatiTrasmissione

L’elemento complesso DatiTrasmissione contiene i campi seguenti:

    IdTrasmittente che si compone di

        IdPaese: sigla della nazione espressa secondo lo standard ISO
        3166-1 alpha-2 code.

        IdCodice: formato alfanumerico; lunghezza massima di 28
        caratteri.
```

<!-- PDF page 63 -->

```text
    ProgressivoInvio: formato alfanumerico; lunghezza massima di 10
    caratteri.

    FormatoTrasmissione: formato alfanumerico; lunghezza di 5
    caratteri; i valori ammessi sono i seguenti:

          FPR12         Formato di Trasmissione fattura verso privati

    CodiceDestinatario: formato alfanumerico; lunghezza di 7 caratteri.

    ContattiTrasmittente che si compone di

        Telefono: formato alfanumerico; lunghezza che va da 5 a 12
        caratteri.

        Email: formato alfanumerico; lunghezza che va da 7 a 256
        caratteri.

    PECDestinatario: formato alfanumerico; lunghezza che va da 7 a 256
    caratteri.

2.2.4   FATTURA ELETTRONICA HEADER: CEDENTE / PRESTATORE

               Figura 3: Elemento CedentePrestatore

L’elemento complesso CedentePrestatore si compone dei campi illustrati
in figura 3 e descritti di seguito nel dettaglio.
```

<!-- PDF page 64 -->

```text
2.2.4.1 Dati Anagrafici

                Figura 4: Elemento Dati Anagrafici del Cedente Prestatore

        L’elemento complesso DatiAnagrafici usa il tipo complesso
        DatiAnagraficiCedenteType, che al suo interno presenta gli elementi
        IdFiscaleIVA e Anagrafica le cui strutture, IdFiscaleType e
        AnagraficaType, sono comuni a più elementi dello schema e vengono
        descritte di seguito nel dettaglio.
```

<!-- PDF page 65 -->

```text
IdFiscaleType

                Figura 5: Tipo complesso IdFiscaleType

Il tipo complesso IdFiscaleType è composto dai seguenti campi:

    IdPaese: sigla della nazione espressa secondo lo standard ISO 3166-
    1 alpha-2 code.

    IdCodice: formato alfanumerico; lunghezza massima di 28 caratteri.

AnagraficaType

                Figura 6: Tipo complesso AnagraficaType

Il tipo complesso AnagraficaType è composto dai seguenti campi:

    Denominazione: formato alfanumerico; lunghezza massima di 80
    caratteri. Da valorizzare in alternativa ai campi Nome e Cognome
    seguenti.
```

<!-- PDF page 66 -->

```text
    Nome: formato alfanumerico; lunghezza massima di 60 caratteri. Da
    valorizzare insieme al campo Cognome ed in alternativa al campo
    Denominazione.

    Cognome: formato alfanumerico; lunghezza massima di 60 caratteri.
    Da valorizzare insieme al campo Nome ed in alternativa al campo
    Denominazione.

    Titolo: formato alfanumerico; lunghezza che va da 2 a 10 caratteri.

    CodEORI: formato alfanumerico; lunghezza che va da 13 a 17
    caratteri.

Gli altri campi del tipo complesso DatiAnagraficiCedenteType sono:

    CodiceFiscale: formato alfanumerico; lunghezza compresa tra 11 e
    16 caratteri.

    AlboProfessionale: formato alfanumerico; lunghezza massima di 60
    caratteri.

    ProvinciaAlbo: formato alfanumerico; lunghezza di 2 caratteri.

    NumeroIscrizioneAlbo: formato alfanumerico; lunghezza massima di
    60 caratteri.

    DataIscrizioneAlbo: la data deve essere rappresentata secondo il
    formato ISO 8601:2004, con la seguente precisione: YYYY-MM-DD.

    RegimeFiscale: formato alfanumerico; lunghezza di 4 caratteri; i
    valori ammessi sono i seguenti:

           RF01         Ordinario;

           RF02         Contribuenti   minimi    (art.   1,   c.96-117,    L.
                        244/2007);

           RF04         Agricoltura e attività connesse e pesca (artt. 34 e
                        34-bis, D.P.R. 633/1972);

           RF05         Vendita sali e tabacchi (art. 74, c.1, D.P.R.
                        633/1972);
```

<!-- PDF page 67 -->

```text
RF06   Commercio dei fiammiferi (art. 74, c.1, D.P.R.
       633/1972);

RF07   Editoria (art. 74, c.1, D.P.R. 633/1972);

RF08   Gestione di servizi di telefonia (art. 74, c.1, D.P.R.
       633/1972);

RF09   Rivendita di documenti di trasporto pubblico e di
       sosta (art. 74, c.1, D.P.R. 633/1972);

RF10   Intrattenimenti, giochi e altre attività di cui alla
       tariffa allegata al D.P.R. n. 640/72 (art. 74, c.6,
       D.P.R. 633/1972);

RF11   Agenzie di viaggi e turismo (art. 74-ter, D.P.R.
       633/1972);

RF12   Agriturismo (art. 5, c.2, L. 413/1991);

RF13   Vendite a domicilio (art. 25-bis, c.6, D.P.R.
       600/1973);

RF14   Rivendita di beni usati, di oggetti d’arte,
       d’antiquariato o da collezione (art. 36, D.L.
       41/1995);

RF15   Agenzie di vendite all’asta di oggetti d’arte,
       antiquariato o da collezione (art. 40-bis, D.L.
       41/1995);

RF16   IVA per cassa P.A. (art. 6, c.5, D.P.R. 633/1972);

RF17   IVA per cassa (art. 32-bis, D.L. 83/2012);

RF18   Altro;

RF19   Regime forfettario (art.1, c. 54-89, L. 190/2014)

RF20   Regime transfrontaliero       di   Franchigia     IVA
       (Direttiva UE 2020/285)
```

<!-- PDF page 68 -->

```text
2.2.4.2 Dati Sede

                    Figura 7: Elemento Sede del CedentePrestatore

       All’interno dell’elemento complesso Sede troviamo i seguenti campi:

            Indirizzo: formato alfanumerico; lunghezza massima di 60 caratteri.

            NumeroCivico: formato alfanumerico; lunghezza massima di 8
            caratteri.

            CAP: formato numerico; lunghezza di 5 caratteri.

            Comune: formato alfanumerico; lunghezza massima di 60 caratteri.

            Provincia: formato alfanumerico; lunghezza di 2 caratteri.

            Nazione: sigla della nazione espressa secondo lo standard ISO 3166-
            1 alpha-2 code.
```

<!-- PDF page 69 -->

```text
2.2.4.3 Dati Stabile Organizzazione

                       Figura 8: Elemento StabileOrganizzazione

             L’elemento StabileOrganizzazione è costituito dagli stessi campi
             previsti per l’elemento Sede del CedentePrestatore contenuti nel tipo
             IndirizzoType e descritti al paragrafo 2.2.4.2.

2.2.4.4 Dati iscrizione R.E.A.

                          Figura 9: Elemento IscrizioneREA

        L’elemento IscrizioneREA è costituito dai campi descritti di seguito:
```

<!-- PDF page 70 -->

```text
            Ufficio: formato alfanumerico; lunghezza di 2 caratteri.

            NumeroREA: formato alfanumerico; lunghezza massima di 20
            caratteri.

            CapitaleSociale: formato numerico nel quale i decimali vanno
            separati dall'intero con il carattere '.' (punto). La sua lunghezza va da
            4 a 15 caratteri.

            SocioUnico: formato alfanumerico; lunghezza di 2 caratteri; i valori
            ammessi sono i seguenti:

                   SU            la società è a socio unico.

                   SM            la società NON è a socio unico.

            StatoLiquidazione: formato alfanumerico; lunghezza di 2 caratteri; i
            valori ammessi sono i seguenti:

                   LS            la società è in stato di liquidazione.

                   LN            la società NON è in stato di liquidazione.

2.2.4.5 Contatti

                            Figura 10: Elemento Contatti

            Telefono: formato alfanumerico; lunghezza che va da 5 a 12 caratteri.

            Fax: formato alfanumerico; lunghezza che va da 5 a 12 caratteri.

            Email: formato alfanumerico; lunghezza che va da 7 a 256 caratteri.

2.2.4.6 Riferimento Amministrazione
         Ultimo elemento all’interno di CedentePrestatoreType è il seguente:
```

<!-- PDF page 71 -->

```text
    RiferimentoAmministrazione:        formato alfanumerico;   lunghezza
    massima di 20 caratteri.

2.2.5   FATTURA ELETTRONICA HEADER: RAPPRESENTANTE FISCALE DEL
        CEDENTE / PRESTATORE

             Figura 11: Elemento RappresentanteFiscale

L’elemento complesso RappresentanteFiscale è formato dal macroblocco
DatiAnagrafici che si compone di:

    IdFiscaleIVA: utilizza la struttura IdFiscaleType già descritta al
    paragrafo 2.2.4.1 al quale si rimanda.

    CodiceFiscale: formato alfanumerico; lunghezza compresa tra 11 e
    16 caratteri.

    Anagrafica: utilizza la struttura AnagraficaType già descritta al
    paragrafo 2.2.4.1 al quale si rimanda.
```

<!-- PDF page 72 -->

```text
        2.2.6      FATTURA ELETTRONICA HEADER: CESSIONARIO / COMMITTENTE

                    Figura 12: Elemento CessionarioCommittente

       L’elemento complesso CessionarioCommittente si compone dei
       macroblocchi  DatiAnagrafici, Sede, StabileOrganizzazione e
       RappresentanteFiscale descritti di seguito.

2.2.6.1 DatiAnagrafici

                Figura 13: Elemento DatiAnagrafici del CessionarioCommittente
```

<!-- PDF page 73 -->

```text
            Il macroblocco DatiAnagrafici si compone di:

            IdFiscaleIVA: utilizza la struttura IdFiscaleType già descritta al
            paragrafo 2.2.4.1 al quale si rimanda.

            CodiceFiscale: formato alfanumerico; lunghezza compresa tra 11 e
            16 caratteri.

            Anagrafica: utilizza la struttura AnagraficaType già descritta al
            paragrafo 2.2.4.1 al quale si rimanda.

2.2.6.2 Dati Sede

                Figura 14: Elemento Sede del CessionarioCommittente

       L’elemento Sede si compone degli stessi campi previsti per l’elemento
       Sede del CedentePrestatore contenuti nel tipo IndirizzoType e descritti al
       paragrafo 2.2.4.2.
```

<!-- PDF page 74 -->

```text
2.2.6.3 Dati Stabile Organizzazione

          Figura 15: Elemento StabileOrganizzazione del CessionarioCommittente

            L’elemento StabileOrganizzazione è costituito dagli stessi campi
            previsti per l’elemento StabileOrganizzazione del CedentePrestatore
            contenuti nel tipo IndirizzoType e descritti al paragrafo 2.2.4.2.

2.2.6.4 Dati Rappresentante Fiscale

          Figura 16: Elemento RappresentanteFiscale del CessionarioCommittente
```

<!-- PDF page 75 -->

```text
L’elemento RappresentanteFiscale è costituito dai campi descritti di
seguito:

    IdFiscaleIVA: utilizza la struttura IdFiscaleType già descritta al
    paragrafo 2.2.4.1 al quale si rimanda.

    Denominazione: formato alfanumerico; lunghezza massima di 80
    caratteri. Da valorizzare in alternativa ai campi Nome e Cognome
    seguenti.

    Nome: formato alfanumerico; lunghezza massima di 60 caratteri. Da
    valorizzare insieme al campo Cognome ed in alternativa al campo
    Denominazione.

    Cognome: formato alfanumerico; lunghezza massima di 60 caratteri.
    Da valorizzare insieme al campo Nome ed in alternativa al campo
    Denominazione.

2.2.7   FATTURA ELETTRONICA HEADER: TERZO INTERMEDIARIO SOGGETTO
        EMITTENTE

        Figura 17: Elemento TerzoIntermediarioSoggettoEmittente
```

<!-- PDF page 76 -->

```text
Come       si      nota      dalla  figura      17,   l’elemento
TerzoIntermediarioSoggettoEmittente è costituito dal macroblocco
DatiAnagrafici che si compone di:

     IdFiscaleIVA: utilizza la struttura IdFiscaleType già descritta al
     paragrafo 2.2.4.1 al quale si rimanda.

     CodiceFiscale: formato alfanumerico; lunghezza compresa tra 11 e
     16 caratteri.

     Anagrafica: utilizza la struttura AnagraficaType già descritta al
     paragrafo 2.2.4.1 al quale si rimanda.

 2.2.8   FATTURA ELETTRONICA HEADER: SOGGETTO EMITTENTE
Ultimo elemento del macroblocco FatturaElettronicaHeader è il seguente:

     SoggettoEmittente: formato alfanumerico; lunghezza di 2 caratteri; i
     valori ammessi sono i seguenti:

           CC            cessionario/committente.

           TZ            soggetto terzo.
```

<!-- PDF page 77 -->

```text
2.2.9   FATTURA ELETTRONICA BODY: DATI GENERALI

                  Figura 18: Elemento DatiGenerali

L’elemento complesso Dati Generali si compone dei campi illustrati in
figura 18 e descritti di seguito.
```

<!-- PDF page 78 -->

```text
2.2.9.1 Dati Generali Documento

                   Figura 19: Elemento DatiGeneraliDocumento
```

<!-- PDF page 79 -->

```text
L’elemento complesso DatiGeneraliDocumento è composto dai campi di
seguito descritti:

    TipoDocumento: formato alfanumerico; lunghezza di 4 caratteri; i
    valori ammessi sono i seguenti:

          TD01        Fattura

          TD02        Acconto/Anticipo su fattura

          TD03        Acconto/Anticipo su parcella

          TD04        Nota di Credito

          TD05        Nota di Debito

          TD06        Parcella

          TD16        Integrazione fattura reverse charge interno

          TD17        Integrazione/autofattura     per   acquisto    servizi
                      dall’estero

          TD18        Integrazione per acquisto di beni intracomunitari

          TD19        Integrazione/autofattura per acquisto di beni ex
                      art.17 c.2 DPR 633/72

          TD20        Autofattura per regolarizzazione e integrazione
                      delle fatture (ex art. 6 c.9-bis d.lgs. 471/97 o art.46
                      c.5 D.L. 331/93)

          TD21        Autofattura per splafonamento

          TD22        Estrazione beni da Deposito IVA

          TD23        Estrazione beni da Deposito IVA con versamento
                      dell’IVA

          TD24        Fattura differita di cui all’art.21, comma 4, terzo
                      periodo lett. a) DPR 633/72

          TD25        Fattura differita di cui all’art.21, comma 4, terzo
                      periodo lett. b) DPR 633/72
```

<!-- PDF page 80 -->

```text
      TD26          Cessione di beni ammortizzabili e per passaggi
                    interni (ex art.36 DPR 633/72)

      TD27          Fattura per autoconsumo o per cessioni gratuite
                    senza rivalsa

      TD28          Acquisti da San Marino con IVA (fattura cartacea)

      TD29          Comunicazione per omessa o irregolare
                    fatturazione (art. 6, comma 8, D.Lgs. 471/97)

Divisa: questo campo deve essere espresso secondo lo standard ISO
4217 alpha-3:2001 (es.: EUR, USD, GBP, CZK………).

Data: la data deve essere rappresentata secondo il formato ISO
8601:2004, con la seguente precisione: YYYY-MM-DD.

Numero: formato alfanumerico; lunghezza massima di 20 caratteri.

DatiRitenuta che si compone di:

   TipoRitenuta: formato alfanumerico; lunghezza di 4 caratteri; i
   valori ammessi sono i seguenti:

      RT01          Ritenuta persone fisiche

      RT02          Ritenuta persone giuridiche

      RT03          Contributo INPS

      RT04          Contributo ENASARCO

      RT05          Contributo ENPAM

      RT06          Altro contributo previdenziale

   ImportoRitenuta: formato numerico nel quale i decimali vanno
   separati dall'intero con il carattere '.' (punto). La sua lunghezza va
   da 4 a 15 caratteri.

   AliquotaRitenuta: formato numerico nel quale i decimali vanno
   separati dall'intero con il carattere '.' (punto). La sua lunghezza va
   da 4 a 6 caratteri.

   CausalePagamento: formato alfanumerico; lunghezza di
   massimo 2 caratteri; i valori ammessi sono quelli della CU
```

<!-- PDF page 81 -->

```text
   consultabili alla pagina delle istruzioni di compilazione del modello,
   ove applicabili.

BolloVirtuale: formato alfanumerico, lunghezza di 2 caratteri; il valore
ammesso è

    SI          bollo assolto ai sensi del decreto MEF 14 giugno 2014

DatiCassaPrevidenziale che si compone di:

   TipoCassa: formato alfanumerico; lunghezza di 4 caratteri; i valori
   ammessi sono i seguenti:

         TC01         Cassa Nazionale Previdenza         e   Assistenza
                      Avvocati e Procuratori Legali

         TC02         Cassa Previdenza Dottori Commercialisti

         TC03         Cassa Previdenza e Assistenza Geometri

         TC04         Cassa Nazionale Previdenza e Assistenza
                      Ingegneri e Architetti Liberi Professionisti

         TC05         Cassa Nazionale del Notariato

         TC06         Cassa Nazionale Previdenza         e   Assistenza
                      Ragionieri e Periti Commerciali

         TC07         Ente   Nazionale     Assistenza  Agenti           e
                      Rappresentanti di Commercio (ENASARCO)

         TC08         Ente Nazionale Previdenza e            Assistenza
                      Consulenti del Lavoro (ENPACL)

         TC09         Ente Nazionale Previdenza e Assistenza Medici
                      (ENPAM)

         TC10         Ente Nazionale Previdenza          e   Assistenza
                      Farmacisti (ENPAF)

         TC11         Ente Nazionale Previdenza          e   Assistenza
                      Veterinari (ENPAV)

         TC12         Ente Nazionale Previdenza e            Assistenza
                      Impiegati dell'Agricoltura (ENPAIA)
```

<!-- PDF page 82 -->

```text
   TC13          Fondo Previdenza Impiegati               Imprese     di
                 Spedizione e Agenzie Marittime

   TC14          Istituto Nazionale Previdenza Giornalisti Italiani
                 (INPGI)

   TC15          Opera Nazionale Assistenza          Orfani Sanitari
                 Italiani (ONAOSI)

   TC16          Cassa      Autonoma       Assistenza        Integrativa
                 Giornalisti Italiani (CASAGIT)

   TC17          Ente Previdenza Periti        Industriali    e   Periti
                 Industriali Laureati (EPPI)

   TC18          Ente Previdenza e Assistenza Pluricategoriale
                 (EPAP)

   TC19          Ente Nazionale Previdenza e Assistenza Biologi
                 (ENPAB)

   TC20          Ente Nazionale Previdenza e Assistenza
                 Professione Infermieristica (ENPAPI)

   TC21          Ente Nazionale Previdenza            e      Assistenza
                 Psicologi (ENPAP)

   TC22          INPS

AlCassa: formato numerico nel quale i decimali vanno separati
dall'intero con il carattere '.' (punto). La sua lunghezza va da 4 a 6
caratteri.

ImportoContributoCassa: formato numerico nel quale i decimali
vanno separati dall'intero con il carattere '.' (punto). La sua
lunghezza va da 4 a 15 caratteri.

ImponibileCassa: formato numerico nel quale i decimali vanno
separati dall’intero con il carattere ‘.’ (punto). La sua lunghezza va
da 4 a 15 caratteri.

AliquotaIVA: formato numerico nel quale i decimali vanno separati
dall'intero con il carattere '.' (punto). La sua lunghezza va da 4 a 6
caratteri.
```

<!-- PDF page 83 -->

```text
Ritenuta: formato alfanumerico; lunghezza di 2 caratteri; il valore
ammesso è:

   SI            contributo cassa soggetto a ritenuta

Natura: formato alfanumerico; lunghezza da 2 a 4 caratteri; i valori
ammessi sono i seguenti:

   N1            escluse ex art.15 del DPR 633/72

   N2.1         non soggette ad IVA ai sensi degli artt. Da 7 a 7-
                septies del DPR 633/72

   N2.2          non soggette – altri casi

   N3.1          non imponibili – esportazioni

   N3.2          non imponibili – cessioni intracomunitarie

   N3.3          non imponibili – cessioni verso San Marino

   N3.4         non imponibili – operazioni assimilate alle cessioni
                all’esportazione

   N3.5          non imponibili – a seguito di dichiarazioni d’intento

   N3.6         non imponibili – altre operazioni che non
                concorrono alla formazione del plafond

   N4            esenti

   N5            regime del margine / IVA non esposta in fattura

   N6.1         inversione contabile – cessione di rottami e altri
                materiali di recupero

   N6.2         inversione contabile – cessione di oro e argento ai
                sensi della legge 7/2000 nonché di oreficeria
                usata ad OPO

   N6.3         inversione contabile – subappalto nel settore edile

   N6.4         inversione contabile – cessione di fabbricati
```

<!-- PDF page 84 -->

```text
       N6.5         inversione contabile – cessione di telefoni cellulari

       N6.6         inversione contabile – cessione di prodotti
                    elettronici

       N6.7         inversione contabile – prestazioni comparto edile
                    e settori connessi

       N6.8         inversione    contabile    –   operazioni     settore
                    energetico

       N6.9         inversione contabile – altri casi

       N7           IVA assolta in altro stato UE (prestazione di servizi
                    di telecomunicazioni, tele-radiodiffusione ed
                    elettronici ex art. 7-octies, comma 1 lett. a, b, art.
                    74-sexies DPR 633/72)

   RiferimentoAmministrazione: formato alfanumerico; lunghezza
   massima di 20 caratteri.

ScontoMaggiorazione che si compone di:

   Tipo: formato alfanumerico; lunghezza di 2 caratteri; i valori
   ammessi sono i seguenti:

       SC            sconto

       MG            maggiorazione

   Percentuale: formato numerico nel quale i decimali vanno separati
   dall’intero con il carattere ‘.’ (punto). La sua lunghezza va da 4 a 6
   caratteri.

   Importo: formato numerico nel quale i decimali vanno separati
   dall’intero con il carattere ‘.’ (punto). La sua lunghezza va da 4 a
   21 caratteri.

ImportoTotaleDocumento: formato numerico nel quale i decimali
vanno separati dall'intero con il carattere '.' (punto). La sua lunghezza
va da 4 a 15 caratteri.

Arrotondamento: formato numerico nel quale i decimali vanno
separati dall'intero con il carattere '.' (punto). La sua lunghezza va da
4 a 15 caratteri.
```

<!-- PDF page 85 -->

```text
              Causale: formato alfanumerico; lunghezza massima di 200 caratteri.

              Art73: formato alfanumerico; lunghezza di 2 caratteri; il valore
              ammesso è:

                      SI             documento emesso secondo modalità e termini
                                     stabiliti con DM ai sensi dell’art. 73 del DPR
                                     633/72.

2.2.9.2 Dati Ordine Acquisto, Contratto, Convenzione, Ricezione e Fatture
        Collegate

   Figura 20: Elementi DatiOrdineAcquisto, DatiContratto, DatiConvenzione, DatiRicezione e
                                    DatiFattureCollegate.

        I 5 elementi in questione utilizzano tutti il tipo DatiDocumentiCorrelatiType,
        di seguito descritto.
```

<!-- PDF page 86 -->

```text
                     Figura 21: Tipo DatiDocumentiCorrelatiType

            RiferimentoNumeroLinea: formato numerico; lunghezza massima di
            4 caratteri.

            IdDocumento: formato alfanumerico; lunghezza massima di 20
            caratteri.

            Data: la data deve essere rappresentata secondo il formato ISO
            8601:2004, con la seguente precisione: YYYY-MM-DD.

            NumItem: formato alfanumerico; lunghezza massima di 20 caratteri.

            CodiceCommessaConvenzione: formato alfanumerico; lunghezza
            massima di 100 caratteri.

            CodiceCUP: formato alfanumerico; lunghezza massima di 15
            caratteri.

            CodiceCIG: formato alfanumerico; lunghezza massima di 15 caratteri.

2.2.9.3 Dati Riferimento Sal
       Blocco da valorizzare nei casi di fattura per stato di avanzamento
```

<!-- PDF page 87 -->

```text
                           Figura 22: Elemento DatiSal

            RiferimentoFase: formato numerico; lunghezza massima di 3
            caratteri.

2.2.9.4 Dati Ddt (Documento Di Trasporto)

                          Figura 23: Elemento DatiDDT

            NumeroDDT: formato alfanumerico; lunghezza massima di 20
            caratteri.

            DataDDT: la data deve essere rappresentata secondo il formato ISO
            8601:2004, con la seguente precisione: YYYY-MM-DD.

            RiferimentoNumeroLinea: formato numerico; lunghezza massima di
            4 caratteri.
```

<!-- PDF page 88 -->

```text
2.2.9.5 Dati del Trasporto

                          Figura 24: Elemento DatiTrasporto

        L’elemento DatiTrasporto è costituito dai seguenti campi:
```

<!-- PDF page 89 -->

```text
DatiAnagraficiVettore che si compone di:

   IdFiscaleIVA: utilizza la struttura IdFiscaleType già descritta al
   paragrafo 2.2.4.1 al quale si rimanda.

   CodiceFiscale: formato alfanumerico; lunghezza compresa tra 11
   e 16 caratteri.

   Anagrafica: utilizza la struttura AnagraficaType già descritta al
   paragrafo 2.2.4.1 al quale si rimanda.

   NumeroLicenzaGuida:         formato    alfanumerico;    lunghezza
   massima di 20 caratteri.

MezzoTrasporto: formato alfanumerico; lunghezza massima di 80
caratteri.

CausaleTrasporto: formato alfanumerico; lunghezza massima di 100
caratteri.

NumeroColli: formato numerico; lunghezza massima di 4 caratteri.

Descrizione: formato alfanumerico; lunghezza massima di 100
caratteri.

UnitaMisuraPeso: formato alfanumerico; lunghezza massima di 10
caratteri.

PesoLordo: formato numerico nel quale i decimali vanno separati
dall'intero con il carattere '.' (punto). La sua lunghezza va da 4 a 7
caratteri.

PesoNetto: formato numerico nel quale i decimali vanno separati
dall'intero con il carattere '.' (punto). La sua lunghezza va da 4 a 7
caratteri.

DataOraRitiro: la data deve essere rappresentata secondo il formato
ISO 8601:2004, con la seguente precisione: YYYY-MM-
DDTHH:MM:SS.

DataInizioTrasporto: la data deve essere rappresentata secondo il
formato ISO 8601:2004, con la seguente precisione: YYYY-MM-DD.

TipoResa: codifica del termine di resa (Incoterms) espresso secondo
lo standard ICC-Camera di Commercio Internazionale (formato
alfanumerico di 3 caratteri)
```

<!-- PDF page 90 -->

```text
             IndirizzoResa che si compone degli stessi campi previsti per
             l’elemento Sede del CedentePrestatore contenuti nel tipo
             IndirizzoType e descritti al paragrafo 2.2.4.2.

             DataOraConsegna la data deve essere rappresentata secondo il
             formato ISO 8601:2004, con la seguente precisione: YYYY-MM-
             DDTHH:MM:SS.

2.2.9.6 Fattura Principale

                         Figura 25: Elemento FatturaPrincipale

            NumeroFatturaPrincipale:           formato      alfanumerico;   lunghezza
            massima di 20 caratteri.

            DataFatturaPrincipale: la data deve essere rappresentata secondo il
            formato ISO 8601:2004, con la seguente precisione: YYYY-MM-DD.

        2.2.10   FATTURA ELETTRONICA BODY: ELEMENTO DATI DEI BENI/SERVIZI

                         Figura 26: Elemento DatiBeniServizi

        Come si intuisce dalla Figura 26, l’elemento complesso DatiBeniServizi è
        costituito a sua volta dagli elementi DettaglioLinee e DatiRiepilogo.

        Vediamoli di seguito in maniera approfondita.
```

<!-- PDF page 91 -->

```text
2.2.10.1 Dettaglio Linee

        È un blocco contenente natura, qualità e quantità dei beni / servizi formanti
        oggetto dell'operazione. Quando si parla di N occorrenze si intende che i
        suoi campi si ripetono per ogni riga di dettaglio del documento.

                           Figura 27: Elemento DettaglioLinee
```

<!-- PDF page 92 -->

```text
Di seguito vengono fornite le descrizioni dei campi presenti in Figura 27.

    NumeroLinea: formato numerico; lunghezza massima di 4 caratteri.

    TipoCessionePrestazione: formato alfanumerico; lunghezza di 2
    caratteri; i valori ammessi sono:

           SC            Sconto

           PR            Premio

           AB            Abbuono

           AC            Spesa accessoria

    CodiceArticolo che si compone di:

        CodiceTipo: formato alfanumerico; lunghezza massima di 35
        caratteri.

        CodiceValore: formato alfanumerico; lunghezza massima di 35
        caratteri.

    Descrizione: formato alfanumerico; lunghezza massima di 1000
    caratteri.

    Quantita: formato numerico nel quale i decimali vanno separati
    dall'intero con il carattere '.' (punto). La sua lunghezza va da 4 a 21
    caratteri.

    UnitaMisura: formato alfanumerico; lunghezza massima di 10
    caratteri.

    DataInizioPeriodo: la data deve essere rappresentata secondo il
    formato ISO 8601:2004, con la seguente precisione: YYYY-MM-DD.

    DataFinePeriodo: la data deve essere rappresentata secondo il
    formato ISO 8601:2004, con la seguente precisione: YYYY-MM-DD.

    PrezzoUnitario: formato numerico nel quale i decimali vanno separati
    dall'intero con il carattere '.' (punto). La sua lunghezza va da 4 a 21
    caratteri.
```

<!-- PDF page 93 -->

```text
ScontoMaggiorazione che si compone degli stessi campi del tipo
ScontoMaggiorazioneType già descritti al paragrafo 2.2.9.1.

PrezzoTotale: formato numerico nel quale i decimali vanno separati
dall'intero con il carattere '.' (punto). La sua lunghezza va da 4 a 21
caratteri.

AliquotaIVA: formato numerico nel quale i decimali vanno separati
dall'intero con il carattere '.' (punto). La sua lunghezza va da 4 a 6
caratteri.

Ritenuta: formato alfanumerico; lunghezza di 2 caratteri; il valore
ammesso è:

      SI            linea di fattura soggetta a ritenuta

Natura: formato alfanumerico; lunghezza da 2 a 4 caratteri; i valori
ammessi sono i seguenti:

      N1            escluse ex art.15 del DPR 633/72

      N2.1          non soggette ad IVA ai sensi degli artt. Da 7 a 7-
                    septies del DPR 633/72

      N2.2          non soggette – altri casi

      N3.1          non imponibili – esportazioni

      N3.2          non imponibili – cessioni intracomunitarie

      N3.3          non imponibili – cessioni verso San Marino

      N3.4          non imponibili – operazioni            assimilate   alle
                    cessioni all’esportazione

      N3.5          non imponibili – a seguito di dichiarazioni d’intento

      N3.6          non imponibili – altre operazioni che non
                    concorrono alla formazione del plafond

      N4            esenti

      N5            regime del margine / IVA non esposta in fattura

      N6.1          inversione contabile – cessione di rottami e altri
                    materiali di recupero
```

<!-- PDF page 94 -->

```text
      N6.2          inversione contabile – cessione di oro e argento ai
                    sensi della legge 7/2000 nonché di oreficeria
                    usata ad OPO

      N6.3          inversione contabile – subappalto nel settore edile

      N6.4          inversione contabile – cessione di fabbricati

      N6.5          inversione contabile – cessione di telefoni cellulari

      N6.6          inversione contabile – cessione di prodotti
                    elettronici

      N6.7          inversione contabile – prestazioni comparto edile
                    e settori connessi

      N6.8          inversione    contabile    –   operazioni     settore
                    energetico

      N6.9          inversione contabile – altri casi

      N7            IVA assolta in altro stato UE (prestazione di servizi
                    di telecomunicazioni, tele-radiodiffusione ed
                    elettronici ex art. 7-octies, comma 1 lett. a, b, art.
                    74-sexies DPR 633/72)

RiferimentoAmministrazione:       formato alfanumerico;       lunghezza
massima di 20 caratteri.

AltriDatiGestionali che si compone di:

   TipoDato: formato alfanumerico; lunghezza massima di 10
   caratteri.

   RiferimentoTesto: formato alfanumerico; lunghezza massima di
   60 caratteri.

   RiferimentoNumero: formato numerico nel quale i decimali vanno
   separati dall'intero con il carattere '.' (punto). La sua lunghezza va
   da 4 a 21 caratteri.

   RiferimentoData: la data deve essere rappresentata secondo il
   formato ISO 8601:2004, con la seguente precisione: YYYY-MM-
   DD.
```

<!-- PDF page 95 -->

```text
2.2.10.2 Dati Di Riepilogo

        I campi che contiene si ripetono per ogni aliquota e modalità di versamento
        dell’imposta diversa (“scissione dei pagamenti” od ordinaria) o per natura
        diversa. Ne deriva una molteplicità da 0 ad N.

                             Figura 28: Elemento DatiRiepilogo

             AliquotaIVA: formato numerico nel quale i decimali vanno separati
             dall'intero con il carattere '.' (punto). La sua lunghezza va da 4 a 6
             caratteri.

             Natura: formato alfanumerico; lunghezza da 2 a 4 caratteri; i valori
             ammessi sono i seguenti:

                   N1               escluse ex art.15 del DPR 633/72

                   N2.1             non soggette ad IVA ai sensi degli artt. Da 7 a 7-
                                    septies del DPR 633/72

                   N2.2             non soggette – altri casi

                   N3.1             non imponibili – esportazioni

                   N3.2             non imponibili – cessioni intracomunitarie
```

<!-- PDF page 96 -->

```text
       N3.3          non imponibili – cessioni verso San Marino

       N3.4         non imponibili – operazioni assimilate alle cessioni
                    all’esportazione

       N3.5          non imponibili – a seguito di dichiarazioni d’intento

       N3.6         non imponibili – altre operazioni che non
                    concorrono alla formazione del plafond

       N4            esenti

       N5            regime del margine / IVA non esposta in fattura

       N6.1         inversione contabile – cessione di rottami e altri
                    materiali di recupero

       N6.2         inversione contabile – cessione di oro e argento ai
                    sensi della legge 7/2000 nonché di oreficeria
                    usata ad OPO

       N6.3         inversione contabile – subappalto nel settore edile

       N6.4         inversione contabile – cessione di fabbricati

       N6.5         inversione contabile – cessione di telefoni cellulari

       N6.6         inversione contabile – cessione di prodotti
                    elettronici

       N6.7         inversione contabile – prestazioni comparto edile
                    e settori connessi

       N6.8         inversione     contabile    –   operazioni     settore
                    energetico

       N6.9         inversione contabile – altri casi

       N7           IVA assolta in altro stato UE (prestazione di servizi
                    di telecomunicazioni, tele-radiodiffusione ed
                    elettronici ex art. 7-octies, comma 1 lett. a, b, art.
                    74-sexies DPR 633/72)

SpeseAccessorie: formato numerico nel quale i decimali vanno
separati dall'intero con il carattere '.' (punto). La sua lunghezza va da
4 a 15 caratteri.
```

<!-- PDF page 97 -->

```text
   Arrotondamento: formato numerico nel quale i decimali vanno
   separati dall'intero con il carattere '.' (punto). La sua lunghezza va da
   4 a 21 caratteri.

   ImponibileImporto: formato numerico nel quale i decimali vanno
   separati dall'intero con il carattere '.' (punto). La sua lunghezza va da
   4 a 15 caratteri.

   Imposta: formato numerico nel quale i decimali vanno separati
   dall'intero con il carattere '.' (punto). La sua lunghezza va da 4 a 15
   caratteri.

   EsigibilitaIVA: formato alfanumerico; lunghezza di 1 carattere; i
   valori ammessi sono i seguenti:
           I            IVA ad esigibilità immediata
           D            IVA ad esigibilità differita
           S            scissione dei pagamenti

   RiferimentoNormativo: formato alfanumerico; lunghezza massima di
   100 caratteri.

2.2.11   FATTURA ELETTRONICA BODY: DATI SUI VEICOLI

                  Figura 29: Elemento DatiVeicoli

    Data: la data deve essere rappresentata secondo il formato ISO
    8601:2004, con la seguente precisione: YYYY-MM-DD.

    TotalePercorso: formato alfanumerico; lunghezza massima di 15
    caratteri.
```

<!-- PDF page 98 -->

```text
2.2.12   FATTURA ELETTRONICA BODY: DATI DI PAGAMENTO

                Figura 30: Elemento DatiPagamento
```

<!-- PDF page 99 -->

```text
CondizioniPagamento: formato alfanumerico; lunghezza di 4
caratteri; i valori ammessi sono i seguenti:

     TP01         pagamento a rate

     TP02         pagamento completo

     TP03         anticipo

DettaglioPagamento che si compone dei campi seguenti:

   Beneficiario: formato alfanumerico; lunghezza massima di 200
   caratteri.

   ModalitaPagamento: formato alfanumerico; lunghezza di 4
   caratteri; i valori ammessi sono i seguenti:

      MP01        contanti

     MP02         assegno

     MP03         assegno circolare

     MP04         contanti presso Tesoreria

     MP05         bonifico

     MP06         vaglia cambiario

     MP07         bollettino bancario

     MP08         carta di pagamento

     MP09         RID

     MP10         RID utenze

     MP11         RID veloce

     MP12         Riba

     MP13         MAV

     MP14         quietanza erario stato

     MP15         giroconto su conti di contabilità speciale
```

<!-- PDF page 100 -->

```text
   MP16          domiciliazione bancaria

   MP17          domiciliazione postale

   MP18          bollettino di c/c postale

   MP19          SEPA Direct Debit

   MP20          SEPA Direct Debit CORE

   MP21          SEPA Direct Debit B2B

   MP22          Trattenuta su somme già riscosse

   MP23          PagoPA

DataRiferimentoTerminiPagamento: la data deve essere
rappresentata secondo il formato ISO 8601:2004, con la seguente
precisione: YYYY-MM-DD.

GiorniTerminiPagamento: formato numerico di lunghezza
massima pari a 3. Vale 0 (zero) per pagamenti a vista.

DataScadenzaPagamento: la data deve essere rappresentata
secondo il formato ISO 8601:2004, con la seguente precisione:
YYYY-MM-DD.

ImportoPagamento: formato numerico nel quale i decimali vanno
separati dall'intero con il carattere '.' (punto). La sua lunghezza va
da 4 a 15 caratteri.

CodUfficioPostale: formato alfanumerico; lunghezza massima di
20 caratteri.

CognomeQuietanzante:          formato     alfanumerico;   lunghezza
massima di 60 caratteri.

NomeQuietanzante: formato alfanumerico; lunghezza massima
di 60 caratteri.

CFQuietanzante: formato alfanumerico; lunghezza di 16 caratteri.

TitoloQuietanzante: formato alfanumerico; lunghezza che va da
2 a 10 caratteri.
```

<!-- PDF page 101 -->

```text
         IstitutoFinanziario: formato alfanumerico; lunghezza massima di
         80 caratteri.

         IBAN: formato alfanumerico; lunghezza che va da 15 a 34
         caratteri.

         ABI: formato numerico di 5 caratteri.

         CAB: formato numerico di 5 caratteri.

         BIC: formato alfanumerico; lunghezza che va da 8 a 11 caratteri.

         ScontoPagamentoAnticipato: formato numerico nel quale i
         decimali vanno separati dall'intero con il carattere '.' (punto). La
         sua lunghezza va da 4 a 15 caratteri.

         DataLimitePagamentoAnticipato: la data deve essere
         rappresentata secondo il formato ISO 8601:2004, con la seguente
         precisione: YYYY-MM-DD.

         PenalitaPagamentiRitardati: formato numerico nel quale i
         decimali vanno separati dall'intero con il carattere '.' (punto). La
         sua lunghezza va da 4 a 15 caratteri.

         DataDecorrenzaPenale: la data deve essere rappresentata
         secondo il formato ISO 8601:2004, con la seguente precisione:
         YYYY-MM-DD.

         CodicePagamento: formato alfanumerico; lunghezza massima di
         60 caratteri.

2.2.13   FATTURA ELETTRONICA BODY: ALLEGATI

Dati relativi ad eventuali allegati. La struttura del macroblocco è riportata
nella figura seguente.
```

<!-- PDF page 102 -->

```text
               Figura 31: Elemento Allegati

NomeAttachment: formato alfanumerico; lunghezza massima di 60
caratteri.

AlgoritmoCompressione:          formato       alfanumerico;   lunghezza
massima di 10 caratteri.

FormatoAttachment: formato alfanumerico; lunghezza massima di
10 caratteri.

DescrizioneAttachment: formato alfanumerico; lunghezza massima
di 100 caratteri.

Attachment: è in formato xs:base64Binary.
```

<!-- PDF page 103 -->

```text
3.   OPERAZIONI VERSO E DA SOGGETTI NON STABILITI IN ITALIA –
     REGOLE DI PROCESSO

     I dati relativi ad operazioni verso o da soggetti non stabiliti in Italia, da
     trasmettere all’Agenzia delle Entrate devono essere rappresentati e
     trasmessi nel formato XML (eXtensible Markup Language), della fattura
     elettronica ordinaria (utilizzando l’intera fattura o l’intero lotto di fatture),
     secondo lo stesso schema e le stesse regole della fatturazione,
     rappresentate secondo il formato di cui al capitolo 2. Dal 1° luglio 2022,
     eventuali dati delle fatture relativi ad operazioni verso o da soggetti non
     residenti e non stabiliti, trasmessi all’Agenzia delle Entrate in formato XML
     (eXtensible Markup Language), secondo lo schema e le regole in vigore
     fino al 30 giugno 2022 e riportanti fatture relative ad operazioni la cui data
     del documento è successiva a tale data, verranno scartati.

            3.1      MODALITÀ DI PREDISPOSIZIONE DEI FILE PER LA TRASMISSIONE
                     DELLE OPERAZIONI ATTIVE

     Esclusivamente per i dati delle operazioni attive, verso soggetti non stabiliti
     in Italia, l’elemento <CodiceDestinatario> andrà valorizzato con
     “XXXXXXX”. Il Cedente/Prestatore italiano entro i termini di emissione delle
     fatture o dei documenti che ne certificano i corrispettivi, trasmette al Sistema
     di Interscambio (SdI) il file xml predisposto per la comunicazione dei dati.

     La comunicazione dei dati delle operazioni attive non deve essere effettuata
     se è stata emessa una bolletta doganale.

            3.2      MODALITÀ DI PREDISPOSIZIONE DEI FILE PER LA TRASMISSIONE
                     DELLE OPERAZIONI PASSIVE

     Il file xml della fattura, compilato per la comunicazione dei dati delle
     operazioni passive (cessioni o prestazioni ricevute da soggetti non stabiliti
     in Italia), deve essere predisposto in modo che superi i controlli effettuati
     da SdI.

     Relativamente ai campi da compilare ai fini della ricezione del file
     l’elemento <CodiceDestinatario> potrà essere valorizzato con:

     -   un codice relativo ad un canale accreditato a SdI (canale web services
     o SFTP);

     -     ‘0000000’, nei casi venga indicato come canale di ricezione un
     indirizzo PEC e questa sia stata indicata nel campo PECDestinatario;
```

<!-- PDF page 104 -->

```text
-     ‘0000000’, non compilando e il campo PECDestinatario; in tal caso il
file verrà recapitato all’indirizzo che il cessionario/committente ha registrato
come canale di ricezione delle fatture elettroniche ovvero verrà messo a
disposizione in area riservata del portale Fatture e Corrispettivi.

Ad esempio, l’elemento <CodiceDestinatario> potrà essere valorizzato con
il valore che il cessionario/committente solitamente utilizza per ricevere le
fatture elettroniche.

La comunicazione dei dati delle operazioni passive non deve essere
effettuata se è stata emessa una bolletta doganale o ricevuta una fattura
elettronica transitata per SdI.

I tipi di documento che possono essere utilizzati sono di seguito riportati:

TD17               Integrazione/autofattura per acquisto servizi dall’estero

TD18               Integrazione per acquisto di beni intracomunitari

TD19               Integrazione/autofattura per acquisto di beni ex art.17 c.2
                   DPR 633/72

TD28               Acquisti da San Marino con IVA (fattura cartacea)

Il Cessionario/Committente trasmette al Sistema di Interscambio (SdI) il file
xml predisposto per la comunicazione dei dati entro il quindicesimo giorno
del mese successivo a quello di ricevimento del documento comprovante
l'operazione (in caso di operazioni Intra UE) o di effettuazione
dell'operazione (in caso di operazioni extra UE).

Si riportano di seguito le regole di compilazione del file.

3.2.1     INTEGRAZIONE/AUTOFATTURA PER ACQUISTO SERVIZI DALL'ESTERO -
          TD17

Descrizione dell’operazione: il cedente/prestatore (C/P) non residente e
privo di stabile organizzazione nel territorio dello Stato (anche residente
nella Repubblica di San Marino o nello Stato della Città del Vaticano)
emette una fattura per prestazioni di servizi al cessionario/committente
(C/C) residente o stabilito nel territorio nazionale indicando l’imponibile ma
non la relativa imposta in quanto l’operazione, vista dal lato dell’emittente,
```

<!-- PDF page 105 -->

```text
          è non soggetta ed è imponibile in Italia e l’imposta è assolta dal committente
          residente o stabilito in Italia9.

          Il C/C, ai sensi dell’articolo 17, secondo comma, del d.P.R. 26 ottobre 1972,
          n. 633, deve integrare il documento ricevuto (nel caso di servizi intra-UE) o
          emettere un’autofattura (nel caso di servizi extra-UE) per indicare l’imposta
          dovuta che dovrà poi confluire nella propria liquidazione.
          Al fine di adempiere agli obblighi comunicativi di cui all’ articolo 1, comma
          3-bis, del decreto legislativo 5 agosto 2015, n. 127, per le operazioni
          effettuate a partire dal 1° luglio 2022 il C/C deve predisporre un altro
          documento, ad integrazione della fattura ricevuta dal soggetto passivo UE,
          ed inviarlo tramite SdI con tipo documento TD17 che sarà recapitato al solo
          soggetto emittente (dato che è quest'ultimo ad essere tenuto ad integrare
          la fattura con l'IVA). La trasmissione di un tipo documento TD17 consente
          anche di adempiere all’obbligo d’integrazione ai sensi dell’articolo 17,
          secondo comma, del d.P.R. 26 ottobre 1972, n. 633.

          Il TD17 deve essere usato anche per comunicare i dati relativi a prestazioni
          di servizi (non rilevanti ai fini IVA in Italia) effettuate da un soggetto non
          stabilito ricevute da un soggetto passivo italiano.

          Per le operazioni effettuate fino al 30 giugno 2022, alternativamente alla
          trasmissione del TD17 via SdI, il C/C può integrare manualmente la fattura
          o emettere un’autofattura cartacea o elettronica extra SdI ed è obbligato a
          comunicare i dati dell’operazione ricevuta dal fornitore estero, integrati con
          quelli dell’imposta, tramite la vecchia modalità di comunicazione
          dell’esterometro.

3.2.1.1 Compilazione del documento XML con codice TD17

         • <CedentePrestatore>: dati del prestatore estero con l’indicazione del
           paese di residenza dello stesso10

9 Ai sensi dell’art. 17, quarto comma, del d.P.R. n. 633/72 il reverse charge non trova applicazione

(e il TD17 non deve essere utilizzato) qualora la prestazione di servizi territorialmente rilevante sia
stata effettuata dal soggetto non residente per il tramite di una stabile organizzazione nel territorio
dello Stato.
10 Si ricorda che qualora il C/P emetta una fattura riportante la partita IVA italiana aperta tramite un

rappresentante fiscale o identificazione diretta ai sensi dell’art. 35-ter del d.P.R. n. 633/72 il
documento emesso non avrà rilevanza ai fini IVA (risoluzione n. 89/E del 25 agosto 2010). Pertanto,
qualora il C/P sia extra-UE, il C/C deve trasmettere un TD17 avente valore di autofattura indicando
```

<!-- PDF page 106 -->

```text
         • <CessionarioCommittente>: dati del C/C che effettua l’integrazione o
           emette l’autofattura
         • <Data> della sezione “Dati Generali”:
                     • la data di ricezione (o comunque una data ricadente nel mese
                         di ricezione della fattura emessa dal fornitore estero), nel caso
                         di emissione del documento integrativo relativo all’acquisto di
                         servizi intra-UE;
                     • la data di effettuazione dell’operazione, nel caso di emissione
                         dell’autofattura relativa all’acquisto di servizi extra-UE o
                         acquisti di servizi da prestatore residente nella Repubblica di
                         San Marino o nello Stato della Città del Vaticano.
         • Per ogni operazione: compilare la sezione <DettaglioLinee> con i dati
           caratteristici di ogni operazione rispettando l’obbligatorietà dei campi pena
           lo scarto: indicazione dell’imponibile presente nella fattura inviata dal C/P
           e della relativa imposta calcolata dal C/C o della Natura nel caso non si
           tratti di un’operazione imponibile (ad esempio codice N3.4 nel caso di non
           imponibilità, codice N4 nel caso di esenzione, codice N2.2 in caso di
           acquisto di beni non rilevante ai fini IVA in Italia).
         • <DatiFattureCollegate>: gli estremi della fattura di riferimento e, a questo
           fine, l’IdSdi attribuito alla stessa dal Sistema di interscambio, quando
           disponibile
         • <Numero>: consigliabile adoperare una numerazione progressiva ad hoc

          3.2.2    INTEGRAZIONE PER ACQUISTO DI BENI INTRACOMUNITARI - TD18
         Descrizione dell’operazione: il C/P residente in altro paese UE emette una
         fattura per la vendita di beni al C/C residente o stabilito nel territorio
         nazionale indicando l’imponibile ma non la relativa imposta in quanto
         l’operazione vista dal lato dell’emittente, è non imponibile nel Paese di
         residenza del cedente, mentre è imponibile in Italia e l’imposta è assolta
         dal cessionario.
         Al fine di adempiere agli obblighi comunicativi di cui all’articolo 1, comma
         3-bis, del decreto legislativo 5 agosto 2015, n. 127, per le operazioni
         effettuate a partire dal 1° luglio 2022 il C/C deve predisporre un altro
         documento, ad integrazione della fattura ricevuta dal soggetto passivo UE,
         ed inviarlo tramite SdI con tipo documento TD18 che verrà recapitato solo
         al soggetto emittente (dato che è quest'ultimo ad essere tenuto ad integrare
         l'IVA in fattura). La trasmissione di un tipo documento TD18 consente
         anche di adempiere all’obbligo d’integrazione ai sensi dell’articolo 46 del
         d.l. n. 331 del 1993.

l’identificativo estero del C/P; qualora invece il C/P sia residente in un paese UE, il C/C deve
trasmettere un TD17 avente valore di integrazione della fattura emessa tramite la posizione IVA
estera avente rilevanza ai fini IVA indicando l’identificativo estero del C/P.
```

<!-- PDF page 107 -->

```text
       Il codice TD18 deve essere utilizzato anche per gli acquisti intracomunitari
       con introduzione dei beni in un deposito IVA.
       Per le operazioni effettuate fino al 30 giugno 2022, alternativamente alla
       trasmissione del TD18 via SdI, il C/C può integrare manualmente la fattura
       ed è obbligato a comunicare i dati della fattura ricevuta dal fornitore estero,
       integrata con i dati dell’imposta, tramite la vecchia modalità di
       comunicazione dell’esterometro.

3.2.2.1 Compilazione del documento XML TD18
       • <CedentePrestatore>: dati del cedente estero con l’indicazione del
         paese di residenza dello stesso
       • <CessionarioCommittente>: dati del C/C che effettua l’integrazione
       • <Data> della sezione “Dati Generali”: data di ricezione (o comunque una
         data ricadente nel mese di ricezione della fattura emessa dal fornitore
         estero)
      • Per ogni operazione: compilare la sezione <DettaglioLinee> con i dati
         caratteristici di ogni operazione rispettando l’obbligatorietà dei campi
         pena lo scarto: indicazione dell’imponibile presente nella fattura inviata
         dal C/P e della relativa imposta calcolata dal C/C o della Natura nel caso
         non si tratti di un’operazione imponibile (ad esempio per gli acquisti non
         imponibili con uso del Plafond occorre indicare N3.5; nel caso di
         introduzione di beni in un deposito IVA a seguito di acquisto
         intracomunitario occorre indicare la Natura N3.6; nel caso di acquisti
         esenti occorre indicare la Natura N4).
       • <DatiFattureCollegate>: gli estremi della fattura di riferimento e, a questo
         fine, l’IdSdi attribuito alla stessa dal Sistema di interscambio, quando
         disponibile
       • <Numero>: consigliabile adoperare una numerazione progressiva ad hoc

       3.2.3    INTEGRAZIONE/AUTOFATTURA PER ACQUISTO DI BENI EX ART. 17 C.2
                 D.P.R. 633/72 - TD19

       Descrizione dell’operazione: il C/P non residente e privo di stabile
       organizzazione nel territorio dello Stato emette una fattura per la vendita di
       beni già presenti in Italia (non sono quindi importazioni o acquisti
       intracomunitari) al C/C residente o stabilito nel territorio nazionale,
       indicando l’imponibile ma non la relativa imposta in quanto l’operazione
```

<!-- PDF page 108 -->

```text
          vista dal lato dell’emittente, è non soggetta, imponibile in Italia e l’imposta
          è assolta dal cessionario11.
          Il C/C, ai sensi dell’articolo 17, secondo comma del d.P.R. 26 ottobre 1972,
          n. 633, deve integrare il documento ricevuto (nel caso di C/P intra-UE) o
          emettere un’autofattura (nel caso di C/P extra-UE) per indicare l’imposta
          dovuta che dovrà poi confluire nella propria liquidazione.
          Al fine di adempiere agli obblighi comunicativi di cui all’articolo 1, comma
          3-bis, del decreto legislativo 5 agosto 2015, n. 127, per le operazioni
          effettuate a partire dal 1° luglio 2022 il C/C deve predisporre un altro
          documento, contenente sia i dati necessari per l’integrazione sia gli estremi
          della fattura ricevuta dal fornitore estero, ed inviarlo tramite SdI con tipo
          documento TD19 che verrà recapitata solo al soggetto emittente. La
          trasmissione di un tipo documento TD19 consente anche di adempiere
          all’obbligo d’integrazione ai sensi dell’articolo 17, secondo comma, del
          d.P.R. 26 ottobre 1972, n. 633.

          La trasmissione allo SdI di un tipo documento TD19 potrà essere effettuata
          dal C/C anche nel caso di emissione di un’autofattura per acquisto di beni
          provenienti dallo Stato della Città del Vaticano ai sensi dell’articolo 17,
          secondo comma del d.P.R. 26 ottobre 1972, n. 633. Il TD19 deve essere
          invece obbligatoriamente utilizzato nel caso di acquisto di beni provenienti
          dalla Repubblica di San Marino ai sensi dell’articolo 17, secondo comma
          del d.P.R. 26 ottobre 1972, n. 633 al fine di integrare la fattura emessa dal
          fornitore sanmarinese in formato elettronico (cfr. Circolare n. 26 del
          13/07/22).

          Il codice TD19 deve essere utilizzato anche:
                • in caso di integrazione/autofattura ex articolo 17 c.2 d.P.R. 633/72
                   per acquisti da soggetti non stabiliti, di beni già presenti in Italia con
                   introduzione in un deposito IVA (articolo 50-bis, comma 4, lettera
                   c));
                • per acquisti da soggetti non residenti di beni (o di servizi su beni)
                   che si trovano all’interno di un deposito IVA utilizzando la natura
                   N3.6;
                • per comunicare i dati relativi all’acquisto, da un soggetto non
                   stabilito, di beni situati all’estero (non rilevante ai fini IVA in Italia)
                   effettuato da un soggetto passivo italiano.

          Per le operazioni effettuate fino al 30 giugno 2022, alternativamente alla
          trasmissione del TD19 via SdI, il C/C può integrare manualmente la fattura
11 Ai sensi dell’art. 17, quarto comma, del d.P.R. n. 633/72 il reverse charge non trova applicazione

(e il TD19 non deve essere utilizzato) qualora la cessione di beni territorialmente rilevante sia stata
effettuata dal soggetto non residente per il tramite di una stabile organizzazione nel territorio dello
Stato.
```

<!-- PDF page 109 -->

```text
          (cartacea o elettronica extra SdI) o emettere un’autofattura cartacea o
          elettronica extra SdI ed è obbligato a comunicare i dati dell’operazione
          ricevuta dal fornitore estero, integrati con quelli dell’imposta, tramite la
          vecchia modalità di comunicazione dell’esterometro.

3.2.3.1 Compilazione del documento XML TD19
          • <CedentePrestatore>: dati del cedente estero con l’indicazione del
             paese di residenza dello stesso12
          • <CessionarioCommittente>: dati del C/C, che effettua l’integrazione o
             emette l’autofattura
          • <Data> della sezione “Dati Generali”:
                    • la data di ricezione della fattura emessa dal fornitore UE o dal
                        fornitore residente nella Repubblica di San Marino (o
                        comunque una data ricadente nel mese di ricezione della
                        fattura stessa)
                    • la data di effettuazione dell’operazione con il fornitore extra-UE
                        o con fornitore residente nello Stato della Città del Vaticano, nel
                        caso di emissione dell’autofattura
         • Per ogni operazione: compilare la sezione <DettaglioLinee> con i dati
           caratteristici di ogni operazione rispettando l’obbligatorietà dei campi pena
           lo scarto: indicazione di imponibile presente nella fattura inviata dal C/P e
           della relativa imposta calcolata dal C/C o della Natura nel caso non si tratti
           di un’operazione imponibile (ad esempio per i non imponibili con uso del
           Plafond occorre indicare N3.5, nel caso di acquisti da soggetti non
           residenti di beni già presenti in Italia con introduzione in un deposito IVA
           (articolo 50-bis, comma 4, lettera c)) oppure per acquisti da soggetti non
           residenti di beni (o di servizi su beni) che si trovano all’interno di un
           deposito IVA occorre indicare la Natura N3.6, nel caso di acquisto di beni
           non rilevante ai fini IVA in Italia occorre indicare la Natura N2.2
          • <DatiFattureCollegate>: gli estremi della fattura di riferimento, ad
             esempio, l’IdSdi attribuito alla stessa dal SdI
          • <Numero>: consigliabile adoperare una numerazione progressiva ad
             hoc

12 Si ricorda che qualora il C/P emetta una fattura riportante la partita IVA italiana aperta tramite un

rappresentante fiscale o identificazione diretta ai sensi dell’art. 35-ter del d.P.R. n. 633/72, il
documento emesso non avrà rilevanza ai fini IVA (risoluzione n. 89/E del 25 agosto 2010). Pertanto,
qualora il C/P sia extra-UE, il C/C deve trasmettere un TD19 avente valore di autofattura indicando
l’identificativo estero del C/P; qualora invece il C/P sia residente in un paese UE, il C/C deve
trasmettere un TD19 avente valore di integrazione della fattura emessa tramite la posizione IVA
estera avente rilevanza ai fini IVA indicando l’identificativo estero del C/P.
```

<!-- PDF page 110 -->

```text
3.2.4     ACQUISTI DA SAN MARINO CON IVA (FATTURA CARTACEA) – TD28
          Descrizione dell’operazione: il soggetto passivo IVA residente o stabilito in
          Italia che riceve una fattura cartacea 13 con addebito dell’imposta da
          soggetto residente nella Repubblica di San Marino, deve predisporre un
          documento con Tipo Documento TD28 e inviarlo tramite SdI per rispettare
          l’obbligo di comunicazione di cui all’articolo 1, comma 3-bis, del decreto
          legislativo 5 agosto 2015, n. 127. In presenza di una fattura cartacea
          emessa da un fornitore di San Marino con addebito dell’imposta va
          utilizzato il documento TD28 e non il TD19 che, invece, deve essere
          adoperato per l’assolvimento dell’imposta ai sensi dell’articolo 17, secondo
          comma del d.P.R. 26 ottobre 1972, n. 633, nel caso in cui la fattura ricevuta
          dall’operatore sammarinese (elettronica o cartacea) sia senza addebito
          dell’imposta.
          Il TD28 può essere utilizzato anche per adempiere all’obbligo di
          comunicazione di cui all’articolo 1, comma 3-bis, del decreto legislativo 5
          agosto 2015, n. 127 nell’ipotesi di cui all’articolo 6, comma 9-bis. 1, del
          d.lgs. n. 471/97 in cui il cessionario/committente, anziché assolvere
          l’imposta con il regime dell’inversione contabile, riceva una fattura cartacea
          con addebito dell’imposta in rivalsa dal fornitore non stabilito, ancorché
          identificato in Italia.

3.2.4.1 Compilazione del documento XML TD28 nel caso di acquisti da San
        Marino con IVA (Fattura cartacea)
          • <CedentePrestatore>: dati del C/P residente nella Repubblica di San
            Marino emittente la fattura cartacea con addebito dell’imposta.
          • <CessionarioCommittente>: dati del C/C italiano.
          • <Data> della sezione “Dati Generali”: la data di effettuazione
            dell’operazione indicata nella fattura cartacea emessa dal C/P
            sammarinese.
          • <Numero>: consigliabile adoperare una numerazione progressiva scelta
            dal mittente (ad esempio il protocollo del registro acquisti).
          • <DatiFattureCollegate>: numero e data della fattura cartacea originale
            emessa dal fornitore sammarinese.

13 A decorrere dal 1° luglio 2022, in applicazione delle disposizioni del decreto ministeriale del 21

giugno 2021, negli scambi con la repubblica di San Marino le fatture saranno solo elettroniche, fatte
salve le ipotesi di esonero. Infatti, il Decreto Delegato n. 147 del 5 agosto 2021 della Repubblica di
San Marino stabilisce che: “Sono esclusi dall’obbligo di emettere le fatture elettroniche di cui al
comma 3 gli operatori economici stabiliti o identificati nel territorio della Repubblica di San Marino
che hanno dichiarato ricavi nell’anno solare precedente per un importo inferiore a euro 100.000,00
(centomila/00). Essi possono comunque decidere per l’emissione delle fatture elettroniche
presentando apposita opzione”.
```

<!-- PDF page 111 -->

```text
      Indicazione di imponibile e imposta, come indicato nella fattura cartacea
      ricevuta.

3.2.4.2 Compilazione del documento XML TD28 nell’ipotesi dell’art. 6, comma
        9-bis. 1, del d.lgs. n. 471/97 con fornitore non residente identificato in
        Italia
        • <CedentePrestatore>: dati identificativi del C/P estero che ha emesso la
          fattura con addebito dell’imposta tramite la posizione IVA aperta in Italia
          (i dati devono riferirsi al soggetto estero e non alla posizione IVA italiana).
        • <CessionarioCommittente>: dati del C/C italiano.
        • <Data> della sezione “Dati Generali”: la data di effettuazione
          dell’operazione indicata nella fattura emessa dal C/P non stabilito
          identificato in Italia.
        • <Numero>: consigliabile adoperare una numerazione progressiva scelta
          dal mittente.
        • <DatiFattureCollegate>: numero e data della fattura originale emessa dal
          fornitore non stabilito identificato in Italia.
```

<!-- PDF page 112 -->

```text
INDICAZIONE DI IMPONIBILE E IMPOSTA, COME INDICATO NELLA FATTURA
CARTACEA RICEVUTA.APPENDICE 1: ELENCO DEI CODICI ERRORE
RELATIVI ALLA FATTURA ORDINARIA E A QUELLA SEMPLIFICATA

1. TIPOLOGIE E MODALITÀ DI VERIFICA
Le tipologie di controllo effettuate mirano a verificare:
          - nomenclatura ed unicità del file trasmesso;

           -   dimensioni del file;

           -   integrità del documento se firmato con le tecnologie elencate al
               paragrafo 1.2.1;

           -   autenticità del certificato di firma, se presente;

           -   conformità del formato fattura;

           -   coerenza e validità del contenuto della fattura;

           -   unicità della fattura;

           -   recapitabilità della fattura.

Nomenclatura ed unicità del file trasmesso
La verifica viene eseguita al fine di intercettare ed impedire l’invio di un file già
trasmesso; attraverso un controllo sulla nomenclatura del file ricevuto il SdI verifica
che il nome file sia conforme con quanto riportato nel precedente paragrafo 1.2.2 e
che non sia stato già inviato un file con lo stesso nome (ogni file inviato al Sistema
di Interscambio deve avere un nome diverso da qualsiasi altro file inviato in
precedenza); in caso di esito negativo del controllo (nome file già presente nel SdI
o nome file non conforme) il file viene rifiutato con le seguenti motivazioni:

           -   Codice: 00001
               Descrizione: Nome file non valido
               (vale sia per le fatture ordinarie che per le semplificate)

           -   Codice: 00002
               Descrizione: Nome file duplicato
               (vale sia per le fatture ordinarie che per le semplificate)

Dimensioni del file
La verifica è effettuata al fine di garantire che il file ricevuto non ecceda le dimensioni
ammesse per il rispettivo canale di trasmissione.
```

<!-- PDF page 113 -->

```text
           -   Codice: 00003
               Descrizione: Le dimensioni del file superano quelle ammesse
               (vale sia per le fatture ordinarie che per le semplificate)

Verifica di integrità del documento
Nel caso in cui il file sia firmato, la verifica viene effettuata al fine di garantire che il
documento ricevuto non abbia subito modifiche successivamente all’apposizione
della firma; attraverso un controllo sulla firma apposta sull’oggetto trasmesso, il SdI
verifica l’integrità dell’oggetto stesso; laddove dovesse emergere che il documento
ricevuto non corrisponde al documento sul quale è stata apposta la firma, il
documento viene rifiutato con la seguente motivazione:

           -   Codice: 00102
               Descrizione: La firma elettronica apposta al file non risulta valida
               (vale sia per le fatture ordinarie che per le semplificate)

Verifica di autenticità del certificato di firma
Nel caso in cui il file sia firmato, la verifica viene effettuata al fine di garantire la
validità del certificato di firma utilizzato per apporre la firma elettronica qualificata al
documento; sulla base delle informazioni messe a disposizione dalle “Certification
Authorities”, il SdI verifica la validità del certificato di firma, che non deve risultare
scaduto, revocato o sospeso; in caso di certificato di firma non valido, il documento
viene rifiutato con le seguenti motivazioni:

           -   Codice: 00100
               Descrizione: Certificato di firma scaduto
               (vale sia per le fatture ordinarie che per le semplificate)

           -   Codice: 00101
               Descrizione: Certificato di firma revocato
               (vale sia per le fatture ordinarie che per le semplificate)

           -   Codice: 00104
               Descrizione: La CA (Certification Authority) che ha emesso il certificato di
               firma non risulta nell’elenco delle CA affidabili (vale sia per le fatture ordinarie
               che per le semplificate)

           -   Codice: 00107
               Descrizione: Il certificato di firma non è valido

               (vale sia per le fatture ordinarie che per le semplificate)

Verifica di conformità del formato fattura
La verifica viene effettuata per garantire che il contenuto del documento sia
rappresentato secondo le regole definite nelle specifiche tecniche; qualora dovesse
```

<!-- PDF page 114 -->

```text
risultare una non corretta aderenza alle regole, il documento viene rifiutato con le
seguenti motivazioni:

          -   Codice: 00103
              Descrizione: Alla firma elettronica apposta al file manca il riferimento
              temporale
              (nel caso in cui il file sia firmato con firma elettronica qualificata)
              (vale sia per le fatture ordinarie che per le semplificate)

          -   Codice: 00105
              Descrizione: Il riferimento temporale associato alla firma elettronica apposta
              al file è successivo alla data di ricezione del file (nel caso in cui il file sia
              firmato con firma elettronica qualificata)
              (vale sia per le fatture ordinarie che per le semplificate)

          -   Codice: 00106
              Descrizione: Il file compresso è vuoto oppure non è leggibile
              (vale sia per le fatture ordinarie che per le semplificate)

          -   Codice: 00200
              Descrizione: File non conforme al formato (nella descrizione del messaggio
              è riportata l’indicazione puntuale della non conformità)
              (vale sia per le fatture ordinarie che per le semplificate)

          -   Codice: 00201
              Descrizione: Non è possibile procedere con ulteriori controlli perché gli errori
              di formato presenti nel file superano il numero massimo previsto (50)
              (in questo caso nella ricevuta di scarto non sono riportati gli errori ulteriori)
              (vale sia per le fatture ordinarie che per le semplificate)

Verifica di coerenza sul contenuto
La verifica viene effettuata per garantire la coerenza del contenuto degli elementi
informativi per come previsto dalle regole tecniche; in caso di mancato rispetto di
dette regole, il documento viene rifiutato con le seguenti motivazioni:

          -   Codice: 00400
              Descrizione in caso di fatture ordinarie: 2.2.1.14 <Natura> non presente a
              fronte di 2.2.1.12 <AliquotaIVA> pari a zero
              Descrizione in caso di fatture semplificate: 2.2.4 <Natura> non presente a
              fronte di 2.2.3.2 <Aliquota> pari a zero
              (l’indicazione di un’aliquota IVA pari a zero obbliga all’indicazione della
              natura dell’operazione che giustifichi la non imponibilità della stessa)

          -   Codice: 00401
              Descrizione in caso di fatture ordinarie: 2.2.1.14 <Natura> presente a fronte
              di 2.2.1.12 <AliquotaIVA> diversa da zero
              Descrizione in caso di fatture semplificate: 2.2.4 <Natura> presente a fronte
              di 2.2.3.2 <Aliquota> diversa da zero
```

<!-- PDF page 115 -->

```text
    (l’indicazione di un’aliquota IVA diversa da zero qualifica l’operazione come
    imponibile e quindi non è ammessa la presenza dell’elemento <Natura>, ad
    eccezione del caso in cui l’elemento TipoDocumento assume valore TD16)

-   Codice: 00403
    Descrizione: 2.1.1.3 <Data> successiva alla data di ricezione (la data della
    fattura non può essere successiva alla data in cui la stessa è ricevuta dal
    SdI)
    (vale sia per le fatture ordinarie che per le semplificate)

-   Codice: 00411
    Descrizione: 2.1.1.5 <DatiRitenuta> non presente a fronte di almeno un
    blocco 2.2.1 <DettaglioLinee> con 2.2.1.13 <Ritenuta> uguale a SI (la
    presenza di una linea di fattura soggetta a ritenuta obbliga alla valorizzazione
    del blocco <DatiRitenuta>)
    (vale solo per le fatture ordinarie)

-   Codice: 00413
    Descrizione: 2.1.1.7.7 <Natura> non presente a fronte di 2.1.1.7.5
    <AliquotaIVA> pari a zero (l’indicazione di un’aliquota IVA pari a zero obbliga
    all’indicazione della natura del contributo cassa previdenziale che giustifichi
    la non imponibilità dello stesso)
    (vale solo per le fatture ordinarie)

-   Codice: 00414
    Descrizione: 2.1.1.7.7 <Natura> presente a fronte di 2.1.1.7.5 <AliquotaIVA>
    diversa da zero (l’indicazione di un’aliquota IVA diversa da zero qualifica il
    contributo cassa previdenziale come imponibile e quindi non è ammessa la
    presenza dell’elemento <Natura>)
    (vale solo per le fatture ordinarie)

-   Codice: 00415
    Descrizione: 2.1.1.5 <DatiRitenuta> non presente a fronte di 2.1.1.7.6
    <Ritenuta> uguale a SI (la presenza di un contributo cassa previdenziale
    soggetto a ritenuta obbliga alla valorizzazione del blocco <DatiRitenuta>)
    (vale solo per le fatture ordinarie)

-   Codice: 00417
    Descrizione in caso di fatture ordinarie: 1.4.1.1 <IdFiscaleIVA> e 1.4.1.2
    <CodiceFiscale> non valorizzati
    Descrizione in caso di fatture semplificate: 1.3.1.1 <IdFiscaleIVA> e 1.3.1.2
    <CodiceFiscale> non valorizzati
    (per il cessionario/committente deve essere indicato almeno uno tra partita
    IVA e codice fiscale)

-   Codice: 00418
    Descrizione in caso di fatture ordinarie: 2.1.1.3 <Data> antecedente a 2.1.6.3
    <Data>
```

<!-- PDF page 116 -->

```text
    Descrizione in caso di fatture semplificate: 2.1.1.3 <Data> antecedente a
    2.1.2.2 <DataFR>
    (la data della fattura non può essere antecedente a quella del documento al
    quale la stessa si collega)

-   Codice: 00419
    Descrizione: 2.2.2 <DatiRiepilogo> non presente in corrispondenza di
    almeno un valore di 2.1.1.7.5 <AliquotaIVA> o 2.2.1.12 <AliquotaIVA> (per
    ogni aliquota IVA presente in fattura deve esistere il corrispondente blocco
    di <DatiRiepilogo>)
    (vale solo per le fatture ordinarie)

-   Codice: 00420
    Descrizione: 2.2.2.2 <Natura> con valore di tipo N6 a fronte di 2.2.2.7
    <EsigibilitaIVA> uguale a S (scissione pagamenti) (il regime di scissione
    pagamenti non è compatibile con quello di inversione contabile – reverse
    charge)
    (vale solo per le fatture ordinarie)

-   Codice: 00421
    Descrizione: 2.2.2.6 <Imposta> non calcolato secondo le regole definite nelle
    specifiche tecniche (il valore dell’elemento <Imposta> deve essere uguale al
    risultato della seguente operazione:

                      ( AliquotaIVA * ImponibileImporto ) ∕ 100 )

    il risultato di questa operazione va arrotondato alla seconda cifra decimale,
    per difetto se la terza cifra decimale è inferiore a 5, per eccesso se uguale o
    superiore a 5; è ammessa la tolleranza di ±1 centesimo di euro)
    (vale solo per le fatture ordinarie)

-   Codice: 00422
    Descrizione: 2.2.2.5 <ImponibileImporto> non calcolato secondo le regole
    definite     nelle    specifiche   tecniche (il valore    dell’elemento
    <ImponibileImporto> deve essere uguale, per ogni valore distinto di aliquota
    IVA, al risultato della seguente operazione:
        n                    m                               t
       Σ PrezzoTotale[y] + Σ ImportoContributoCassa[x] + Σ Arrotondamento[z]
       y=1                  x=1                            z=1

    dove n è il numero di linee di dettaglio con stessa aliquota IVA, m è il numero
    di blocchi di dati cassa previdenziale con stessa aliquota IVA, t è il numero
    di blocchi di dati riepilogo con stessa aliquota IVA; è ammessa la tolleranza
    di ±1 euro)
    (vale solo per le fatture ordinarie)

-   Codice: 00423
```

<!-- PDF page 117 -->

```text
    Descrizione: 2.2.1.11 <PrezzoTotale> non calcolato secondo le regole
    definite nelle specifiche tecniche (il valore dell’elemento <PrezzoTotale>
    deve essere uguale al risultato della seguente operazione:

                   (PrezzoUnitario ± ScontoMaggiorazione) * Quantita

    è ammessa la tolleranza di ±1 centesimo di euro)

    (vale solo per le fatture ordinarie)

-   Codice: 00424
    Descrizione in caso di fatture ordinarie: 2.2.1.12 <AliquotaIVA> o 2.2.2.1
    <AliquotaIVA> o 2.1.1.7.5 <AliquotaIVA> non indicata in termini percentuali
    Descrizione in caso di fatture semplificate: 2.2.3.2 <Aliquota> non indicata in
    termini percentuali
    (l’aliquota IVA va sempre espressa in termini percentuali; ad esempio,
    un’aliquota del 10% va indicata con 10.00 e non con 0.10)

-   Codice: 00425
    Descrizione: 2.1.1.4 <Numero> non contenente caratteri numerici (il numero
    della fattura deve contenere almeno un carattere numerico)
    (vale sia per le fatture ordinarie che per le semplificate)

-   Codice: 00427
    Descrizione: 1.1.4 <CodiceDestinatario> di 7 caratteri a fronte di 1.1.3
    <FormatoTrasmissione> con valore FPA12 o 1.1.4 <CodiceDestinatario> di
    6 caratteri a fronte di 1.1.3 <FormatoTrasmissione> con valore FPR12
    (vale solo per le fatture ordinarie)

-   Codice: 00428
    Descrizione: 1.1.3 <FormatoTrasmissione> non coerente con il valore
    dell’attributo VERSION
    (vale sia per le fatture ordinarie che per le semplificate)

-   Codice: 00429
    Descrizione: 2.2.2.2 <Natura> non presente a fronte di 2.2.2.1
    <AliquotaIVA> pari a zero (nei <DatiRiepilogo>, l’indicazione di un’aliquota
    IVA pari a zero obbliga all’indicazione della natura che giustifichi la non
    imponibilità)
    (vale solo per le fatture ordinarie)

-   Codice: 00430
    Descrizione: 2.2.2.2 <Natura> presente a fronte di 2.2.2.1 <AliquotaIVA>
    diversa da zero (l’indicazione di un’aliquota IVA diversa da zero qualifica i
    dati di riepilogo come dati riferiti ad operazioni imponibili e quindi non è
    ammessa la presenza dell’elemento <Natura>, ad eccezione del caso in cui
    l’elemento TipoDocumento assume valore TD16)

    (vale solo per le fatture ordinarie)
```

<!-- PDF page 118 -->

```text
-   Codice: 00437
    Descrizione: 2.1.1.8.2 <Percentuale> e 2.1.1.8.3 <Importo> non presenti a
    fronte di 2.1.1.8.1 <Tipo> valorizzato (l’indicazione della presenza di uno
    sconto o di una maggiorazione, obbliga all’indicazione di almeno uno degli
    elementi <Percentuale> e <Importo> dello sconto/maggiorazione)
    (vale solo per le fatture ordinarie)

-   Codice: 00438
    Descrizione: 2.2.1.10.2 <Percentuale> e 2.2.1.10.3 <Importo> non presenti
    a fronte di 2.2.1.10.1 <Tipo> valorizzato (l’indicazione della presenza di uno
    sconto o di una maggiorazione, obbliga all’indicazione di almeno uno degli
    elementi <Percentuale> e <Importo> dello sconto/maggiorazione)
    (vale solo per le fatture ordinarie)

-   Codice: 00443
    Descrizione: non c’è corrispondenza tra i valori indicati nell’elemento
    2.2.1.12 <AliquotaIVA> o 2.1.1.7.5 <AliquotaIVA> e quelli dell’elemento
    2.2.2.1 <AliquotaIVA> (tutti i valori delle aliquote IVA presenti nelle linee di
    dettaglio di una fattura o nei dati di cassa previdenziale devono essere
    presenti anche nei dati di riepilogo)
    (vale solo per le fatture ordinarie)

-   Codice: 00444
    Descrizione: non c’è corrispondenza tra i valori indicati nell’elemento
    2.2.1.14 <Natura> o 2.1.1.7.7 <Natura> e quelli dell’elemento 2.2.2.2
    <Natura> (tutti i valori di natura dell’operazione presenti nelle linee di
    dettaglio di una fattura o nei dati di cassa previdenziale devono essere
    presenti anche nei dati di riepilogo)
    (vale solo per le fatture ordinarie)

-   Codice: 00445
    Descrizione in caso di fatture ordinarie: non è più ammesso il valore generico
    N2, N3 o N6 come codice natura dell’operazione
    Descrizione in caso di fatture semplificate: non è più ammesso il valore
    generico N2 o N3 come codice natura dell’operazione (a partire dal primo
    gennaio 2021 non è più consentito utilizzare i codici natura ‘padre’ ma solo
    quelli di dettaglio, laddove previsti; in particolare per le fatture ordinarie non
    sono più utilizzabili i codici N2, N3 e N6, mente per quelle semplificate non
    sono più utilizzabili i codici N2 e N3)

-   Codice: 00460
    Descrizione: Importo totale superiore al limite previsto per le fatture
    semplificate ai sensi del DPR 633/72, art. 21 bis (nelle fatture semplificate il
    valore dell’elemento <Importo>, o la somma dei valori di tale elemento se
    presente più volte, non può eccedere il limite di euro 400, salvo il caso in
    cui la fattura sia emessa in regime forfettario (RF19 - Regime forfettario
    (art. 1, c. 54-89, L. 190/2014)) o in regime transfrontaliero di Franchigia IVA
```

<!-- PDF page 119 -->

```text
    (RF20 - Regime transfrontaliero di Franchigia IVA (Direttiva UE 2020/285)),
    e salvo il caso in cui si stia modificando una fattura già emessa e quindi sia
    valorizzato il blocco <DatiFatturaRettificata>)
    (vale solo per le fatture semplificate)

-   Codice: 00471
    Descrizione: per il valore indicato nell’elemento 2.1.1.1 <TipoDocumento> il
    cedente/prestatore non può essere uguale al cessionario/committente (i
    valori del tipo documento TD01, TD02, TD03, TD06, TD16, TD17, TD18,
    TD19, TD20, TD24, TD25, TD28 e TD29 (fatture ordinarie) e TD07 (fatture
    semplificate) non ammettono l’indicazione in fattura dello stesso soggetto sia
    come cedente che come cessionario)

-   Codice: 00472
    Descrizione: per il valore indicato nell’elemento 2.1.1.1 <TipoDocumento> il
    cedente/prestatore deve essere uguale al cessionario/committente (i tipi
    documento TD21 e TD27 non ammettono l’indicazione in fattura di un
    cedente diverso dal cessionario)
    (vale solo per le fatture ordinarie)

-   Codice: 00473
    Descrizione: per il valore indicato nell’elemento 2.1.1.1 <TipoDocumento> il
    valore presente nell’elemento 1.2.1.1.1 <IdPaese> non è ammesso (i valori
    TD17, TD18, TD19 e TD28 del tipo documento non ammettono l’indicazione
    in fattura di un cedente italiano. Il valore TD29 del tipo documento non
    ammette l’indicazione di un valore diverso da “IT” nell’elemento 1.2.1.1.1
    <IdPaese>. Nei casi di TD17 e TD19 è ammessa l’indicazione del valore
    ‘OO’ nell’elemento 1.2.1.1.1 <IdPaese> per operazioni effettuate da soggetti
    residenti in Livigno e Campione d’Italia.
    (vale solo per le fatture ordinarie)

-   Codice: 00474
    Descrizione: per il valore indicato nell’elemento 2.1.1.1 <TipoDocumento>
    non sono ammesse linee di dettaglio con l’elemento 2.2.1.12 <AliquotaIVA>
    contenente valore zero (nel tipo documento ‘autofattura per splafonamento’
    tutte le linee di dettaglio devo avere un’aliquota IVA diversa da zero)
    (vale solo per le fatture ordinarie)

-   Codice: 00475
    Descrizione: per il valore indicato nell’elemento 2.1.1.1 <TipoDocumento>
    deve essere presente l’elemento 1.4.1.1 <IdFiscaleIVA> del
    cessionario/committente (i tipi documento TD16, TD17, TD18, TD19, TD20,
    TD22, TD23, TD28 e TD29 prevedono obbligatoriamente la presenza della
    partita IVA del cessionario/committente)
    (vale solo per le fatture ordinarie)
```

<!-- PDF page 120 -->

```text
          -   Codice: 00476
              Descrizione: in caso di fatture ordinarie: gli elementi 1.2.1.1.1 <IdPaese> e
              1.4.1.1.1 <IdPaese> non possono essere entrambi valorizzati con codice
              diverso da IT.
              Descrizione in caso di fatture semplificate: gli elementi 1.2.1.1 <IdPaese> e
              1.3.1.1.1 <IdPaese> non possono essere entrambi valorizzati con codice
              diverso da IT
              (non è ammessa una fattura riportante contemporaneamente nel paese
              dell’identificativo   fiscale      del      cedente/prestatore       e     del
              cessionario/committente un valore diverso da IT)

          -   Codice: 00477
              Descrizione: fattura recante titolo di non imponibilità ai fini IVA ai sensi
              dell’art. 8, comma 1, lett. c) DPR 26 ottobre 1972, n. 633, con Dichiarazione
              d’intento invalidata

Verifica di validità del contenuto della fattura
La verifica viene effettuata per accertare la presenza e la validità dei dati necessari
al corretto inoltro del documento al destinatario e per prevenire situazioni di dati
errati e/o non elaborabili; in particolare viene effettuato un controllo:
           - sull’esistenza del codice destinatario qualora sia diverso dal valore di
               default (“0000000”) e la partita IVA del cessionario/committente non
               risulti registrata secondo le modalità descritte al paragrafo 1.5.1.2
           - sulla validità dei codici fiscali e delle partite IVA relative ai soggetti
               trasmittente,        cedente/prestatore,       cessionario/committente,
               rappresentante fiscale, attraverso una verifica di presenza in anagrafe
               tributaria; il controllo non è effettuato per gli identificativi fiscali
               assegnati da autorità estere
           - sulla coerenza tra partita IVA e codice fiscale del cedente/prestatore
               o del cessionario/committente quando entrambi valorizzati e/o nei casi
               di gruppi IVA.

Laddove anche uno solo di questi controlli non dovesse essere superato, il
documento viene rifiutato con le seguenti motivazioni:

          -   Codice: 00300
              Descrizione: 1.1.1.2 <IdCodice> non valido
              (vale sia per le fatture ordinarie che per le semplificate)

          -   Codice: 00301
              Descrizione in caso di fatture ordinarie: 1.2.1.1.2 <IdCodice> non valido
```

<!-- PDF page 121 -->

```text
    Descrizione in caso di fatture semplificate: 1.2.1.2 <IdCodice> non valido

-   Codice: 00303
    Descrizione in caso di fatture ordinarie: 1.3.1.1.2 <IdCodice> o 1.4.4.1.2
    <IdCodice> non valido
    Descrizione in caso di fatture semplificate: 1.2.8.1.2 <IdCodice> o 1.3.2.6.1.2
    <IdCodice> non valido

-   Codice: 00305
    Descrizione in caso di fatture ordinarie: 1.4.1.1.2 <IdCodice> non valido
    Descrizione in caso di fatture semplificate: 1.3.1.1.2 <IdCodice> non valido

-   Codice: 00306
    Descrizione in caso di fatture ordinarie: 1.4.1.2 <CodiceFiscale> non valido
    Descrizione in caso di fatture semplificate: 1.3.1.2 <CodiceFiscale> non
    valido
    (vale solo se non è stato valorizzato rispettivamente l’elemento 1.4.1.1.2
    <IdCodice> per le fatture ordinarie e 1.3.1.1.2 <IdCodice> per le fatture
    semplificate, vale a dire la partita IVA del cessionario/committente)

-   Codice: 00311
    Descrizione: 1.1.4 <CodiceDestinatario> non valido
    (vale sia per le fatture ordinarie che per le semplificate)

-   Codice: 00312
    Descrizione: 1.1.4 <CodiceDestinatario> non attivo
    (vale sia per le fatture ordinarie che per le semplificate)

-   Codice: 00313
    Descrizione in caso di fatture ordinarie: l’elemento 1.1.4
    <CodiceDestinatario> può essere valorizzato con “XXXXXXX” per
    comunicare i dati di fatture emesse esclusivamente verso soggetti non
    residenti (1.4.1.1 <IdFiscaleIVA> deve essere valorizzato e 1.4.1.1.1
    <IdPaese> deve essere diverso da “IT”)

-   Codice: 00320
    Descrizione in caso di fatture ordinarie: 1.2.1.1 <IdFiscaleIVA> e 1.2.1.2
    <CodiceFiscale> non coerenti
    Descrizione in caso di fatture semplificate: 1.2.1 <IdFiscaleIVA> e 1.2.2
    <CodiceFiscale> non coerenti

-   Codice: 00321
    Descrizione in caso di fatture ordinarie: 1.2.1.2 <CodiceFiscale> di soggetto
    non partecipante al gruppo IVA
    Descrizione in caso di fatture semplificate: 1.2.2 <CodiceFiscale> di soggetto
    non partecipante al gruppo IVA

-   Codice: 00322
```

<!-- PDF page 122 -->

```text
               Descrizione in caso di fatture ordinarie: 1.2.1.2 <CodiceFiscale> non
               presente a fronte di 1.2.1.1 <IdFiscaleIVA> di gruppo IVA
               Descrizione in caso di fatture semplificate: 1.2.2 <CodiceFiscale> non
               presente a fronte di 1.2.1 <IdFiscaleIVA> di gruppo IVA

           -   Codice: 00323
               Descrizione: 1.2.1.1.2 <IdCodice> corrispondente a una partita IVA cessata
               da oltre 5 anni
               (vale solo per le fatture ordinarie)

           -   Codice: 00324
               Descrizione in caso di fatture ordinarie: 1.4.1.1 <IdFiscaleIVA> e 1.4.1.2
               <CodiceFiscale> non coerenti
               Descrizione in caso di fatture semplificate: 1.3.1.1 <IdFiscaleIVA> e 1.3.1.2
               <CodiceFiscale> non coerenti

           -   Codice: 00325
               Descrizione in caso di fatture ordinarie: 1.4.1.2 <CodiceFiscale> di soggetto
               non partecipante al gruppo IVA
               Descrizione in caso di fatture semplificate: 1.3.1.2 <CodiceFiscale> di
               soggetto non partecipante al gruppo IVA

           -   Codice: 00326
               Descrizione in caso di fatture ordinarie: 1.4.1.2 <CodiceFiscale> non
               presente a fronte di 1.4.1.1 <IdFiscaleIVA> di gruppo IVA
               Descrizione in caso di fatture semplificate: 1.3.1.2 <CodiceFiscale> non
               presente a fronte di 1.3.1.1 <IdFiscaleIVA> di gruppo IVA

           -   Codice: 00327
               Descrizione in caso di fatture ordinarie: 1.4.1.2 <CodiceFiscale> di gruppo
               IVA non riferito ad un partecipante, in assenza di <IdFiscaleIVA> di gruppo
               IVA
               Descrizione in caso di fatture semplificate: 1.3.1.2 <CodiceFiscale> di
               gruppo IVA non riferito ad un partecipante, in assenza di <IdFiscaleIVA> di
               gruppo IVA
               (Lo scarto avviene se, in assenza di PIVA del cessionario/committente, il
               CF è valorizzato con quello di un gruppo IVA e non di una sua partecipata.
               Il CF deve essere valorizzato con il CF della società partecipante al gruppo
               Iva).

           -   Codice: 00330
               Descrizione in caso di fatture ordinarie: 1.1.6 <PECDestinatario> contenente
               un valore non ammesso
               Descrizione in caso di fatture semplificate: 1.1.5 <PECDestinatario>
               contenente un valore non ammesso

Verifiche di unicità della fattura
La verifica viene eseguita al fine di intercettare ed impedire l’inoltro di una fattura già
trasmessa e elaborata; in quest’ottica, qualora i dati contenuti all’interno della fattura e
relativi a:

           -   identificativo cedente/prestatore
```

<!-- PDF page 123 -->

```text
           -   anno della data fattura

           -   numero fattura

coincidano con quelli di una fattura precedentemente trasmessa e per la quale non sia stata
inviata al soggetto trasmittente una ricevuta di scarto, il documento viene rifiutato con le
seguenti motivazioni:

           -   Codice: 00404
               Descrizione: Fattura duplicata
               (vale sia per le fatture ordinarie che per le semplificate)

           -   Codice: 00409
               Descrizione: Fattura duplicata nel lotto
               (vale sia per le fatture ordinarie che per le semplificate)

Nel caso in cui il documento trasmesso sia una nota di credito (TipoDocumento = TD04 per
le fatture ordinarie o TD08 per le fatture semplificate), la verifica tiene conto anche della
tipologia di documento; pertanto, è ammessa la presenza di due documenti aventi stesso
cedente/prestatore, stesso anno e stesso numero solo qualora uno dei due sia di tipo TD04
per le fatture ordinarie o TD08 per le fatture semplificate.

Solo per le fatture ordinarie, nel caso di fatture emesse secondo modalità e termini stabiliti
con decreto ministeriale ai sensi dell'articolo 73 del DPR 633/72 e per le quali sia stato
valorizzato a “SI” il campo “Art73” (situazione che consente l'emissione nello stesso anno
di più documenti aventi stesso numero), la verifica di unicità viene effettuata secondo le
regole precedenti ma tenendo conto della data completa e non solo dell’anno.

Solo per le fatture ordinarie, nel caso di fatture emesse con TipoDocumento TD16, TD17,
TD18, TD19, TD20, TD22, TD23, TD28 e TD29 il controllo di unicità del documento è
effettuato in capo al soggetto che in fattura è indicato come Cessionario/Committente, ossia
sul soggetto che può avere effettivo dominio della numerazione.

Tutti i controlli descritti in questo paragrafo, qualora il file inviato al SdI sia relativo ad un
documento lotto di fatture (ex art. 1, comma 3, DLGS 20 febbraio 2004 n. 52), comportano
l’accettazione o il rifiuto del file nella sua totalità. Diversamente, se al SdI viene inviato un
file in formato compresso (rif. precedente paragrafo 1.2.2 lett. b), i controlli, con
conseguente accettazione o scarto, riguardano ogni singolo file presente al suo interno.
```

<!-- PDF page 124 -->

```text
APPENDICE 2: FATTURA ORDINARIA

1. SCHEMA XSD FATTURA ORDINARIA

<?xml version="1.0" encoding="utf-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
         xmlns:ds="http://www.w3.org/2000/09/xmldsig#"
         xmlns="http://ivaservizi.agenziaentrate.gov.it/docs/xsd/fatture/v1.2"
         targetNamespace="http://ivaservizi.agenziaentrate.gov.it/docs/xsd/fatture/v1.2"
         version="1.2.3">

 <xs:import namespace="http://www.w3.org/2000/09/xmldsig#" schemaLocation="http://www.w3.org/TR/2002/REC-
xmldsig-core-20020212/xmldsig-core-schema.xsd" />

 <xs:element name="FatturaElettronica" type="FatturaElettronicaType">
  <xs:annotation>
   <xs:documentation>XML schema fatture destinate a PA e privati in forma ordinaria 1.2.3</xs:documentation>
  </xs:annotation>
 </xs:element>

 <xs:complexType name="FatturaElettronicaType">
  <xs:sequence>
   <xs:element name="FatturaElettronicaHeader" type="FatturaElettronicaHeaderType"                   />
   <xs:element name="FatturaElettronicaBody" type="FatturaElettronicaBodyType" maxOccurs="unbounded" />
   <xs:element ref="ds:Signature"                              minOccurs="0"       />
  </xs:sequence>
  <xs:attribute name="versione" type="FormatoTrasmissioneType" use="required" />
  <xs:attribute name="SistemaEmittente" type="String10Type" use="optional" />
 </xs:complexType>
 <xs:complexType name="FatturaElettronicaHeaderType">
  <xs:sequence>
   <xs:element name="DatiTrasmissione"               type="DatiTrasmissioneType"                      />
   <xs:element name="CedentePrestatore"                type="CedentePrestatoreType"                      />
   <xs:element name="RappresentanteFiscale"              type="RappresentanteFiscaleType"          minOccurs="0" />
   <xs:element name="CessionarioCommittente"               type="CessionarioCommittenteType"                />
   <xs:element name="TerzoIntermediarioOSoggettoEmittente" type="TerzoIntermediarioSoggettoEmittenteType"
minOccurs="0" />
   <xs:element name="SoggettoEmittente"                type="SoggettoEmittenteType"             minOccurs="0" />
  </xs:sequence>
 </xs:complexType>
 <xs:complexType name="FatturaElettronicaBodyType">
  <xs:sequence>
   <xs:element name="DatiGenerali" type="DatiGeneraliType"                              />
   <xs:element name="DatiBeniServizi" type="DatiBeniServiziType"                         />
   <xs:element name="DatiVeicoli" type="DatiVeicoliType" minOccurs="0"                       />
   <xs:element name="DatiPagamento" type="DatiPagamentoType" minOccurs="0" maxOccurs="unbounded" />
   <xs:element name="Allegati"     type="AllegatiType"        minOccurs="0" maxOccurs="unbounded" />
  </xs:sequence>
 </xs:complexType>
 <xs:complexType name="DatiTrasmissioneType">
  <xs:annotation>
   <xs:documentation>Blocco relativo ai dati di trasmissione della Fattura Elettronica</xs:documentation>
```

<!-- PDF page 125 -->

```text
 </xs:annotation>
 <xs:sequence>
  <xs:element name="IdTrasmittente"        type="IdFiscaleType"                  />
  <xs:element name="ProgressivoInvio" type="String10Type"                         />
  <xs:element name="FormatoTrasmissione" type="FormatoTrasmissioneType"                      />
  <xs:element name="CodiceDestinatario" type="CodiceDestinatarioType"                    />
  <xs:element name="ContattiTrasmittente" type="ContattiTrasmittenteType" minOccurs="0" />
  <xs:element name="PECDestinatario"        type="EmailType"            minOccurs="0" />
         </xs:sequence>
</xs:complexType>
<xs:simpleType name="CodiceDestinatarioType">
 <xs:restriction base="xs:string">
  <xs:pattern value="[A-Z0-9]{6,7}" />
 </xs:restriction>
</xs:simpleType>
<xs:complexType name="IdFiscaleType">
 <xs:sequence>
  <xs:element name="IdPaese" type="NazioneType" />
  <xs:element name="IdCodice" type="CodiceType" />
 </xs:sequence>
</xs:complexType>
<xs:simpleType name="CodiceType">
 <xs:restriction base="xs:string">
  <xs:minLength value="1" />
  <xs:maxLength value="28" />
 </xs:restriction>
</xs:simpleType>
<xs:simpleType name="FormatoTrasmissioneType">
 <xs:restriction base="xs:string">
  <xs:length value="5" />
          <xs:enumeration value="FPA12">
            <xs:annotation>
             <xs:documentation>Fattura verso PA</xs:documentation>
            </xs:annotation>
          </xs:enumeration>
          <xs:enumeration value="FPR12">
            <xs:annotation>
             <xs:documentation>Fattura verso privati</xs:documentation>
            </xs:annotation>
          </xs:enumeration>
 </xs:restriction>
</xs:simpleType>
<xs:complexType name="ContattiTrasmittenteType">
 <xs:sequence>
  <xs:element name="Telefono" type="TelFaxType" minOccurs="0" />
  <xs:element name="Email" type="EmailContattiType" minOccurs="0" />
 </xs:sequence>
</xs:complexType>
<xs:complexType name="DatiGeneraliType">
 <xs:annotation>
  <xs:documentation>
                                      Blocco relativo ai Dati Generali della Fattura Elettronica
                             </xs:documentation>
 </xs:annotation>
 <xs:sequence>
```

<!-- PDF page 126 -->

```text
   <xs:element name="DatiGeneraliDocumento" type="DatiGeneraliDocumentoType"                              />
   <xs:element    name="DatiOrdineAcquisto"                    type="DatiDocumentiCorrelatiType"      minOccurs="0"
maxOccurs="unbounded" />
   <xs:element name="DatiContratto"                             type="DatiDocumentiCorrelatiType" minOccurs="0"
maxOccurs="unbounded" />
   <xs:element name="DatiConvenzione"                           type="DatiDocumentiCorrelatiType"     minOccurs="0"
maxOccurs="unbounded" />
   <xs:element name="DatiRicezione"                             type="DatiDocumentiCorrelatiType"     minOccurs="0"
maxOccurs="unbounded" />
   <xs:element     name="DatiFattureCollegate"               type="DatiDocumentiCorrelatiType"        minOccurs="0"
maxOccurs="unbounded" />
   <xs:element name="DatiSAL"          type="DatiSALType"           minOccurs="0" maxOccurs="unbounded" />
   <xs:element name="DatiDDT"           type="DatiDDTType"           minOccurs="0" maxOccurs="unbounded" />
   <xs:element name="DatiTrasporto"       type="DatiTrasportoType"       minOccurs="0"               />
   <xs:element name="FatturaPrincipale" type="FatturaPrincipaleType"       minOccurs="0"                />
  </xs:sequence>
 </xs:complexType>
 <xs:complexType name="DatiGeneraliDocumentoType">
  <xs:sequence>
   <xs:element name="TipoDocumento"          type="TipoDocumentoType"                             />
   <xs:element name="Divisa"          type="DivisaType"                              />
   <xs:element name="Data"            type="DataFatturaType"                             />
   <xs:element name="Numero"             type="String20Type"                             />
   <xs:element name="DatiRitenuta"        type="DatiRitenutaType"        minOccurs="0" maxOccurs="unbounded" />
   <xs:element name="DatiBollo"         type="DatiBolloType"         minOccurs="0"              />
   <xs:element      name="DatiCassaPrevidenziale"           type="DatiCassaPrevidenzialeType"         minOccurs="0"
maxOccurs="unbounded" />
   <xs:element name="ScontoMaggiorazione"                type="ScontoMaggiorazioneType"               minOccurs="0"
maxOccurs="unbounded" />
   <xs:element name="ImportoTotaleDocumento" type="Amount2DecimalType"             minOccurs="0"               />
   <xs:element name="Arrotondamento"         type="Amount2DecimalType"         minOccurs="0"                />
   <xs:element name="Causale"           type="String200LatinType"      minOccurs="0" maxOccurs="unbounded" />
   <xs:element name="Art73"           type="Art73Type"             minOccurs="0"             />
  </xs:sequence>
 </xs:complexType>
 <xs:complexType name="DatiRitenutaType">
  <xs:sequence>
   <xs:element name="TipoRitenuta" type="TipoRitenutaType" />
   <xs:element name="ImportoRitenuta" type="Amount2DecimalType" />
   <xs:element name="AliquotaRitenuta" type="RateType"           />
   <xs:element name="CausalePagamento" type="CausalePagamentoType" />
  </xs:sequence>
 </xs:complexType>
 <xs:complexType name="DatiBolloType">
  <xs:sequence>
   <xs:element name="BolloVirtuale" type="BolloVirtualeType" />
   <xs:element name="ImportoBollo" type="Amount2DecimalType" minOccurs="0" />
  </xs:sequence>
 </xs:complexType>
 <xs:complexType name="DatiCassaPrevidenzialeType">
  <xs:sequence>
   <xs:element name="TipoCassa"            type="TipoCassaType"             />
   <xs:element name="AlCassa"             type="RateType"               />
   <xs:element name="ImportoContributoCassa" type="Amount2DecimalType"                  />
   <xs:element name="ImponibileCassa"         type="Amount2DecimalType" minOccurs="0" />
```

<!-- PDF page 127 -->

```text
    <xs:element name="AliquotaIVA"             type="RateType"            />
    <xs:element name="Ritenuta"              type="RitenutaType"    minOccurs="0" />
    <xs:element name="Natura"               type="NaturaType"      minOccurs="0" />
    <xs:element name="RiferimentoAmministrazione" type="String20Type"        minOccurs="0" />
  </xs:sequence>
 </xs:complexType>
 <xs:complexType name="ScontoMaggiorazioneType">
  <xs:sequence>
    <xs:element name="Tipo"        type="TipoScontoMaggiorazioneType"         />
    <xs:element name="Percentuale" type="RateType"               minOccurs="0" />
    <xs:element name="Importo" type="Amount8DecimalType"              minOccurs="0" />
  </xs:sequence>
 </xs:complexType>
 <xs:simpleType name="CausalePagamentoType">
  <xs:restriction base="xs:string">
    <!--I CODICI SEGUENTI FANNO RIFERIMENTO A QUELLI PREVISTI NEL MOD. CU-->
    <xs:enumeration value="A" />
    <xs:enumeration value="B" />
    <xs:enumeration value="C" />
    <xs:enumeration value="D" />
    <xs:enumeration value="E" />
    <xs:enumeration value="G" />
    <xs:enumeration value="H" />
    <xs:enumeration value="I" />
    <xs:enumeration value="L" />
    <xs:enumeration value="M" />
    <xs:enumeration value="N" />
    <xs:enumeration value="O" />
    <xs:enumeration value="P" />
    <xs:enumeration value="Q" />
    <xs:enumeration value="R" />
    <xs:enumeration value="S" />
    <xs:enumeration value="T" />
    <xs:enumeration value="U" />
    <xs:enumeration value="V" />
    <xs:enumeration value="W" />
    <xs:enumeration value="X" />
    <xs:enumeration value="Y" />
<!-- IL CODICE SEGUENTE (Z) NON SARA' PIU' VALIDO PER LE FATTURE EMESSE A PARTIRE DAL PRIMO GENNAIO 2021-->
    <xs:enumeration value="Z" />
    <xs:enumeration value="L1" />
    <xs:enumeration value="M1" />
    <xs:enumeration value="M2" />
    <xs:enumeration value="O1" />
    <xs:enumeration value="V1" />
    <xs:enumeration value="ZO" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="TipoScontoMaggiorazioneType">
  <xs:restriction base="xs:string">
    <xs:length value="2" />
    <xs:enumeration value="SC">
      <xs:annotation>
       <xs:documentation>SC = Sconto</xs:documentation>
      </xs:annotation>
```

<!-- PDF page 128 -->

```text
   </xs:enumeration>
   <xs:enumeration value="MG">
    <xs:annotation>
      <xs:documentation>MG = Maggiorazione</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="Art73Type">
  <xs:restriction base="xs:string">
   <xs:length value="2" />
   <xs:enumeration value="SI">
    <xs:annotation>
      <xs:documentation>SI = Documento emesso secondo modalità e termini stabiliti con DM ai sensi dell'art. 73 DPR
633/72</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="TipoCassaType">
  <xs:restriction base="xs:string">
   <xs:length value="4" />
   <xs:enumeration value="TC01">
    <xs:annotation>
      <xs:documentation>Cassa nazionale previdenza e assistenza avvocati e procuratori legali</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TC02">
    <xs:annotation>
      <xs:documentation>Cassa previdenza dottori commercialisti</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TC03">
    <xs:annotation>
      <xs:documentation>Cassa previdenza e assistenza geometri</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TC04">
    <xs:annotation>
      <xs:documentation>Cassa       nazionale  previdenza     e    assistenza       ingegneri   e    architetti  liberi
professionisti</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TC05">
    <xs:annotation>
      <xs:documentation>Cassa nazionale del notariato</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TC06">
    <xs:annotation>
      <xs:documentation>Cassa nazionale previdenza e assistenza ragionieri e periti commerciali</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TC07">
    <xs:annotation>
```

<!-- PDF page 129 -->

```text
     <xs:documentation>Ente      nazionale      assistenza       agenti       e     rappresentanti      di     commercio
(ENASARCO)</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TC08">
    <xs:annotation>
     <xs:documentation>Ente nazionale previdenza e assistenza consulenti del lavoro (ENPACL)</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TC09">
    <xs:annotation>
     <xs:documentation>Ente nazionale previdenza e assistenza medici (ENPAM)</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TC10">
    <xs:annotation>
     <xs:documentation>Ente nazionale previdenza e assistenza farmacisti (ENPAF)</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TC11">
    <xs:annotation>
     <xs:documentation>Ente nazionale previdenza e assistenza veterinari (ENPAV)</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TC12">
    <xs:annotation>
     <xs:documentation>Ente       nazionale      previdenza          e      assistenza       impiegati     dell'agricoltura
(ENPAIA)</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TC13">
    <xs:annotation>
     <xs:documentation>Fondo previdenza impiegati imprese di spedizione e agenzie marittime</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TC14">
    <xs:annotation>
     <xs:documentation>Istituto nazionale previdenza giornalisti italiani (INPGI)</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TC15">
    <xs:annotation>
     <xs:documentation>Opera nazionale assistenza orfani sanitari italiani (ONAOSI)</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TC16">
    <xs:annotation>
     <xs:documentation>Cassa autonoma assistenza integrativa giornalisti italiani (CASAGIT)</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TC17">
    <xs:annotation>
     <xs:documentation>Ente previdenza periti industriali e periti industriali laureati (EPPI)</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
```

<!-- PDF page 130 -->

```text
   <xs:enumeration value="TC18">
    <xs:annotation>
      <xs:documentation>Ente previdenza e assistenza pluricategoriale (EPAP)</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TC19">
    <xs:annotation>
      <xs:documentation>Ente nazionale previdenza e assistenza biologi (ENPAB)</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TC20">
    <xs:annotation>
      <xs:documentation>Ente        nazionale     previdenza     e      assistenza    professione   infermieristica
(ENPAPI)</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TC21">
    <xs:annotation>
      <xs:documentation>Ente nazionale previdenza e assistenza psicologi (ENPAP)</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TC22">
    <xs:annotation>
      <xs:documentation>INPS</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="TipoDocumentoType">
  <xs:restriction base="xs:string">
   <xs:length value="4" />
   <xs:enumeration value="TD01">
    <xs:annotation>
      <xs:documentation>Fattura</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TD02">
    <xs:annotation>
      <xs:documentation>Acconto / anticipo su fattura</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TD03">
    <xs:annotation>
      <xs:documentation>Acconto / anticipo su parcella</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TD04">
    <xs:annotation>
      <xs:documentation>Nota di credito</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TD05">
    <xs:annotation>
      <xs:documentation>Nota di debito</xs:documentation>
    </xs:annotation>
```

<!-- PDF page 131 -->

```text
    </xs:enumeration>
    <xs:enumeration value="TD06">
     <xs:annotation>
      <xs:documentation>Parcella</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="TD16">
     <xs:annotation>
      <xs:documentation>Integrazione fattura reverse charge interno</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="TD17">
     <xs:annotation>
      <xs:documentation>Integrazione/autofattura per acquisto servizi dall'estero</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="TD18">
     <xs:annotation>
      <xs:documentation>Integrazione per acquisto di beni intracomunitari</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="TD19">
     <xs:annotation>
      <xs:documentation>Integrazione/autofattura per acquisto di beni ex art.17 c.2 DPR 633/72</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="TD20">
     <xs:annotation>
      <xs:documentation>Autofattura per regolarizzazione e integrazione delle fatture (ex art. 6 c.9-bis d.lgs. 471/97 o
art.46 c.5 D.L. 331/93)</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="TD21">
     <xs:annotation>
      <xs:documentation>Autofattura per splafonamento</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="TD22">
     <xs:annotation>
      <xs:documentation>Estrazione benida Deposito IVA</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="TD23">
     <xs:annotation>
      <xs:documentation>Estrazione beni da Deposito IVA con versamento dell'IVA</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="TD24">
     <xs:annotation>
      <xs:documentation>Fattura differita di cui all'art.21, comma 4, terzo periodo lett. a) DPR
633/72</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="TD25">
     <xs:annotation>
```

<!-- PDF page 132 -->

```text
       <xs:documentation>Fattura differita di cui all'art.21, comma 4, terzo periodo lett. b) DPR
633/72</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="TD26">
     <xs:annotation>
       <xs:documentation>Cessione     di beni ammortizzabili e per passaggi interni (ex art.36 DPR
633/72)</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="TD27">
     <xs:annotation>
       <xs:documentation>Fattura per autoconsumo o per cessioni gratuite senza rivalsa</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="TD28">
     <xs:annotation>
       <xs:documentation>Acquisti da San Marino con IVA (fattura cartacea)</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="TD29">
     <xs:annotation>
       <xs:documentation>Comunicazione per omessa o irregolare fatturazione da parte del cedente/prestatore italiano
(art. 6, comma 8, D.Lgs. 471/97)</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="TipoRitenutaType">
  <xs:restriction base="xs:string">
    <xs:length value="4" />
    <xs:enumeration value="RT01">
     <xs:annotation>
       <xs:documentation>Ritenuta di acconto persone fisiche</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="RT02">
     <xs:annotation>
       <xs:documentation>Ritenuta di acconto persone giuridiche</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="RT03">
     <xs:annotation>
       <xs:documentation>Contributo INPS</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="RT04">
     <xs:annotation>
       <xs:documentation>Contributo ENASARCO</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="RT05">
     <xs:annotation>
       <xs:documentation>Contributo ENPAM</xs:documentation>
     </xs:annotation>
```

<!-- PDF page 133 -->

```text
   </xs:enumeration>
   <xs:enumeration value="RT06">
    <xs:annotation>
      <xs:documentation>Altro contributo previdenziale</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
  </xs:restriction>
 </xs:simpleType>
 <xs:complexType name="DatiSALType">
  <xs:sequence>
   <xs:element name="RiferimentoFase" type="RiferimentoFaseType" />
  </xs:sequence>
 </xs:complexType>
 <xs:complexType name="DatiDocumentiCorrelatiType">
  <xs:sequence>
   <xs:element name="RiferimentoNumeroLinea"                    type="RiferimentoNumeroLineaType"     minOccurs="0"
maxOccurs="unbounded" />
   <xs:element name="IdDocumento"              type="String20Type"                             />
   <xs:element name="Data"              type="xs:date"             minOccurs="0"              />
   <xs:element name="NumItem"                type="String20Type"          minOccurs="0"           />
   <xs:element name="CodiceCommessaConvenzione" type="String100LatinType"             minOccurs="0"            />
   <xs:element name="CodiceCUP"              type="String15Type"          minOccurs="0"            />
   <xs:element name="CodiceCIG"             type="String15Type"           minOccurs="0"           />
  </xs:sequence>
 </xs:complexType>
 <xs:simpleType name="RiferimentoNumeroLineaType">
  <xs:restriction base="xs:integer">
   <xs:minInclusive value="1" />
   <xs:maxInclusive value="9999" />
  </xs:restriction>
 </xs:simpleType>
 <xs:complexType name="DatiDDTType">
  <xs:sequence>
   <xs:element name="NumeroDDT"              type="String20Type"                             />
   <xs:element name="DataDDT"            type="xs:date"                                 />
   <xs:element        name="RiferimentoNumeroLinea"          type="RiferimentoNumeroLineaType"        minOccurs="0"
maxOccurs="unbounded" />
  </xs:sequence>
 </xs:complexType>
 <xs:complexType name="DatiTrasportoType">
  <xs:sequence>
   <xs:element name="DatiAnagraficiVettore" type="DatiAnagraficiVettoreType" minOccurs="0" />
   <xs:element name="MezzoTrasporto"          type="String80LatinType"      minOccurs="0" />
   <xs:element name="CausaleTrasporto"        type="String100LatinType"      minOccurs="0" />
   <xs:element name="NumeroColli"          type="NumeroColliType"         minOccurs="0" />
   <xs:element name="Descrizione"        type="String100LatinType"        minOccurs="0" />
   <xs:element name="UnitaMisuraPeso"         type="String10Type"         minOccurs="0" />
   <xs:element name="PesoLordo"          type="PesoType"             minOccurs="0" />
   <xs:element name="PesoNetto"          type="PesoType"             minOccurs="0" />
   <xs:element name="DataOraRitiro"        type="xs:dateTime"          minOccurs="0" />
   <xs:element name="DataInizioTrasporto" type="xs:date"               minOccurs="0" />
   <xs:element name="TipoResa"          type="TipoResaType"            minOccurs="0" />
   <xs:element name="IndirizzoResa"       type="IndirizzoType"         minOccurs="0" />
   <xs:element name="DataOraConsegna"          type="xs:dateTime"          minOccurs="0" />
  </xs:sequence>
```

<!-- PDF page 134 -->

```text
</xs:complexType>
<xs:complexType name="IndirizzoType">
 <xs:sequence>
  <xs:element name="Indirizzo" type="String60LatinType"                       />
  <xs:element name="NumeroCivico" type="NumeroCivicoType" minOccurs="0"                    />
  <xs:element name="CAP"           type="CAPType"                        />
  <xs:element name="Comune"           type="String60LatinType"                  />
  <xs:element name="Provincia" type="ProvinciaType" minOccurs="0"                     />
  <xs:element name="Nazione" type="NazioneType"                       default="IT" />
 </xs:sequence>
</xs:complexType>
<xs:complexType name="FatturaPrincipaleType">
 <xs:sequence>
  <xs:element name="NumeroFatturaPrincipale" type="String20Type" />
  <xs:element name="DataFatturaPrincipale" type="xs:date"           />
 </xs:sequence>
</xs:complexType>
<xs:simpleType name="SoggettoEmittenteType">
 <xs:restriction base="xs:string">
  <xs:length value="2" />
  <xs:enumeration value="CC">
   <xs:annotation>
     <xs:documentation>Cessionario / Committente</xs:documentation>
   </xs:annotation>
  </xs:enumeration>
  <xs:enumeration value="TZ">
   <xs:annotation>
     <xs:documentation>Terzo</xs:documentation>
   </xs:annotation>
  </xs:enumeration>
 </xs:restriction>
</xs:simpleType>
<xs:complexType name="CedentePrestatoreType">
 <xs:annotation>
  <xs:documentation>Blocco relativo ai dati del Cedente / Prestatore</xs:documentation>
 </xs:annotation>
 <xs:sequence>
  <xs:element name="DatiAnagrafici"             type="DatiAnagraficiCedenteType"          />
  <xs:element name="Sede"                  type="IndirizzoType"                 />
  <xs:element name="StabileOrganizzazione"          type="IndirizzoType"        minOccurs="0" />
  <xs:element name="IscrizioneREA"             type="IscrizioneREAType"       minOccurs="0" />
  <xs:element name="Contatti"                type="ContattiType"         minOccurs="0" />
  <xs:element name="RiferimentoAmministrazione" type="String20Type"                  minOccurs="0" />
 </xs:sequence>
</xs:complexType>
<xs:complexType name="DatiAnagraficiCedenteType">
 <xs:sequence>
  <xs:element name="IdFiscaleIVA"         type="IdFiscaleType"             />
  <xs:element name="CodiceFiscale"         type="CodiceFiscaleType" minOccurs="0" />
  <xs:element name="Anagrafica"           type="AnagraficaType"             />
  <xs:element name="AlboProfessionale" type="String60LatinType" minOccurs="0" />
  <xs:element name="ProvinciaAlbo"          type="ProvinciaType" minOccurs="0" />
  <xs:element name="NumeroIscrizioneAlbo" type="String60Type"            minOccurs="0" />
  <xs:element name="DataIscrizioneAlbo" type="xs:date"              minOccurs="0" />
  <xs:element name="RegimeFiscale"          type="RegimeFiscaleType"            />
```

<!-- PDF page 135 -->

```text
  </xs:sequence>
 </xs:complexType>
 <xs:simpleType name="RegimeFiscaleType">
  <xs:restriction base="xs:string">
   <xs:length value="4" />
   <xs:enumeration value="RF01">
    <xs:annotation>
      <xs:documentation> Regime ordinario</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF02">
    <xs:annotation>
      <xs:documentation>Regime dei contribuenti minimi (art. 1,c.96-117, L. 244/2007)</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF04">
    <xs:annotation>
      <xs:documentation>Agricoltura e attività connesse e pesca (artt. 34 e 34-bis, D.P.R. 633/1972)</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF05">
    <xs:annotation>
      <xs:documentation>Vendita sali e tabacchi (art. 74, c.1, D.P.R. 633/1972)</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF06">
    <xs:annotation>
      <xs:documentation>Commercio dei fiammiferi (art. 74, c.1, D.P.R. 633/1972)</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF07">
    <xs:annotation>
      <xs:documentation>Editoria (art. 74, c.1, D.P.R. 633/1972)</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF08">
    <xs:annotation>
      <xs:documentation>Gestione di servizi di telefonia pubblica (art. 74, c.1, D.P.R. 633/1972)</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF09">
    <xs:annotation>
      <xs:documentation>Rivendita di documenti di trasporto pubblico e di sosta (art. 74, c.1, D.P.R.
633/1972)</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF10">
    <xs:annotation>
      <xs:documentation>Intrattenimenti, giochi e altre attività      di cui alla tariffa allegata al D.P.R. 640/72 (art. 74, c.6,
D.P.R. 633/1972)</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF11">
    <xs:annotation>
      <xs:documentation>Agenzie di viaggi e turismo (art. 74-ter, D.P.R. 633/1972)</xs:documentation>
```

<!-- PDF page 136 -->

```text
     </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF12">
     <xs:annotation>
      <xs:documentation>Agriturismo (art. 5, c.2, L. 413/1991)</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF13">
     <xs:annotation>
      <xs:documentation>Vendite a domicilio (art. 25-bis, c.6, D.P.R. 600/1973)</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF14">
     <xs:annotation>
      <xs:documentation>Rivendita di beni usati, di oggetti          d’arte, d’antiquariato o da collezione (art. 36,
D.L. 41/1995)</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF15">
     <xs:annotation>
      <xs:documentation>Agenzie di vendite all’asta di oggetti d’arte, antiquariato o da collezione (art. 40-bis, D.L.
41/1995)</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF16">
     <xs:annotation>
      <xs:documentation>IVA per cassa P.A. (art. 6, c.5, D.P.R. 633/1972)</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF17">
     <xs:annotation>
      <xs:documentation>IVA per cassa (art. 32-bis, D.L. 83/2012)</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF18">
     <xs:annotation>
      <xs:documentation>Altro</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
           <xs:enumeration value="RF19">
     <xs:annotation>
      <xs:documentation>Regime forfettario</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
           <xs:enumeration value="RF20">
     <xs:annotation>
      <xs:documentation>Regime transfrontaliero di Franchigia IVA (Direttiva UE 2020/285)</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
  </xs:restriction>
 </xs:simpleType>
 <xs:complexType name="AnagraficaType">
  <xs:annotation>
   <xs:documentation>Il campo Denominazione è in alternativa ai campi Nome e Cognome</xs:documentation>
  </xs:annotation>
```

<!-- PDF page 137 -->

```text
 <xs:sequence>
  <xs:choice>
   <xs:sequence>
     <xs:element name="Denominazione" type="String80LatinType"/>
   </xs:sequence>
   <xs:sequence>
     <xs:element name="Nome"        type="String60LatinType"/>
     <xs:element name="Cognome"       type="String60LatinType"/>
   </xs:sequence>
  </xs:choice>
  <xs:element name="Titolo" type="TitoloType" minOccurs="0" />
  <xs:element name="CodEORI" type="CodEORIType" minOccurs="0" />
 </xs:sequence>
</xs:complexType>
<xs:complexType name="DatiAnagraficiVettoreType">
 <xs:sequence>
  <xs:element name="IdFiscaleIVA"     type="IdFiscaleType"             />
  <xs:element name="CodiceFiscale"     type="CodiceFiscaleType" minOccurs="0" />
  <xs:element name="Anagrafica"       type="AnagraficaType"              />
  <xs:element name="NumeroLicenzaGuida" type="String20Type"          minOccurs="0" />
 </xs:sequence>
</xs:complexType>
<xs:complexType name="IscrizioneREAType">
 <xs:sequence>
  <xs:element name="Ufficio"      type="ProvinciaType"                />
  <xs:element name="NumeroREA"          type="String20Type"                />
  <xs:element name="CapitaleSociale" type="Amount2DecimalType" minOccurs="0" />
  <xs:element name="SocioUnico"       type="SocioUnicoType"       minOccurs="0" />
  <xs:element name="StatoLiquidazione" type="StatoLiquidazioneType"             />
 </xs:sequence>
</xs:complexType>
<xs:complexType name="ContattiType">
 <xs:sequence>
  <xs:element name="Telefono" type="TelFaxType" minOccurs="0" />
  <xs:element name="Fax" type="TelFaxType" minOccurs="0" />
  <xs:element name="Email" type="EmailContattiType" minOccurs="0" />
 </xs:sequence>
</xs:complexType>
<xs:complexType name="RappresentanteFiscaleType">
 <xs:annotation>
         <xs:documentation>Blocco relativo ai dati del Rappresentante Fiscale</xs:documentation>
 </xs:annotation>
 <xs:sequence>
  <xs:element name="DatiAnagrafici" type="DatiAnagraficiRappresentanteType" />
 </xs:sequence>
</xs:complexType>
<xs:complexType name="DatiAnagraficiRappresentanteType">
 <xs:sequence>
  <xs:element name="IdFiscaleIVA" type="IdFiscaleType"             />
  <xs:element name="CodiceFiscale" type="CodiceFiscaleType" minOccurs="0" />
  <xs:element name="Anagrafica" type="AnagraficaType"                />
 </xs:sequence>
</xs:complexType>
<xs:complexType name="CessionarioCommittenteType">
 <xs:annotation>
```

<!-- PDF page 138 -->

```text
   <xs:documentation>Blocco relativo ai dati del Cessionario / Committente</xs:documentation>
  </xs:annotation>
  <xs:sequence>
   <xs:element name="DatiAnagrafici"      type="DatiAnagraficiCessionarioType"                   />
   <xs:element name="Sede"            type="IndirizzoType"                           />
           <xs:element name="StabileOrganizzazione" type="IndirizzoType"                    minOccurs="0" />
   <xs:element name="RappresentanteFiscale" type="RappresentanteFiscaleCessionarioType" minOccurs="0" />
  </xs:sequence>
 </xs:complexType>
 <xs:complexType name="RappresentanteFiscaleCessionarioType">
  <xs:sequence>
           <xs:element name="IdFiscaleIVA"      type="IdFiscaleType" />
           <xs:choice>
    <xs:sequence>
      <xs:element name="Denominazione" type="String80LatinType"/>
    </xs:sequence>
    <xs:sequence>
      <xs:element name="Nome"        type="String60LatinType" />
      <xs:element name="Cognome"       type="String60LatinType" />
    </xs:sequence>
   </xs:choice>
  </xs:sequence>
 </xs:complexType>
 <xs:complexType name="DatiAnagraficiCessionarioType">
  <xs:sequence>
   <xs:element name="IdFiscaleIVA" type="IdFiscaleType" minOccurs="0" />
   <xs:element name="CodiceFiscale" type="CodiceFiscaleType" minOccurs="0" />
   <xs:element name="Anagrafica" type="AnagraficaType"                   />
  </xs:sequence>
 </xs:complexType>
 <xs:complexType name="DatiBeniServiziType">
  <xs:annotation>
   <xs:documentation>Blocco relativo ai dati di Beni Servizi della Fattura     Elettronica</xs:documentation>
  </xs:annotation>
  <xs:sequence>
   <xs:element name="DettaglioLinee" type="DettaglioLineeType" maxOccurs="unbounded" />
   <xs:element name="DatiRiepilogo" type="DatiRiepilogoType" maxOccurs="unbounded" />
  </xs:sequence>
 </xs:complexType>
 <xs:complexType name="DatiVeicoliType">
  <xs:annotation>
   <xs:documentation>Blocco relativo ai dati dei Veicoli della Fattura Elettronica (da indicare nei casi di cessioni tra Paesi
                           membri di mezzi di trasporto nuovi, in base all'art. 38, comma 4 del dl 331 del
1993)</xs:documentation>
  </xs:annotation>
  <xs:sequence>
   <xs:element name="Data"        type="xs:date" />
   <xs:element name="TotalePercorso" type="String15Type" />
  </xs:sequence>
 </xs:complexType>
 <xs:complexType name="DatiPagamentoType">
  <xs:annotation>
   <xs:documentation>Blocco relativo ai dati di Pagamento della Fattura Elettronica</xs:documentation>
  </xs:annotation>
  <xs:sequence>
```

<!-- PDF page 139 -->

```text
  <xs:element name="CondizioniPagamento" type="CondizioniPagamentoType"                   />
  <xs:element name="DettaglioPagamento" type="DettaglioPagamentoType" maxOccurs="unbounded" />
 </xs:sequence>
</xs:complexType>
<xs:simpleType name="CondizioniPagamentoType">
 <xs:restriction base="xs:string">
  <xs:minLength value="4" />
  <xs:maxLength value="4" />
  <xs:enumeration value="TP01">
   <xs:annotation>
     <xs:documentation>pagamento a rate</xs:documentation>
   </xs:annotation>
  </xs:enumeration>
  <xs:enumeration value="TP02">
   <xs:annotation>
     <xs:documentation>pagamento completo</xs:documentation>
   </xs:annotation>
  </xs:enumeration>
  <xs:enumeration value="TP03">
   <xs:annotation>
     <xs:documentation>anticipo</xs:documentation>
   </xs:annotation>
  </xs:enumeration>
 </xs:restriction>
</xs:simpleType>
<xs:complexType name="DettaglioPagamentoType">
 <xs:sequence>
  <xs:element name="Beneficiario"            type="String200LatinType"       minOccurs="0" />
  <xs:element name="ModalitaPagamento"             type="ModalitaPagamentoType"               />
  <xs:element name="DataRiferimentoTerminiPagamento" type="xs:date"                 minOccurs="0" />
  <xs:element name="GiorniTerminiPagamento"          type="GiorniTerminePagamentoType" minOccurs="0" />
  <xs:element name="DataScadenzaPagamento"            type="xs:date"           minOccurs="0" />
  <xs:element name="ImportoPagamento"              type="Amount2DecimalType"                />
  <xs:element name="CodUfficioPostale"          type="String20Type"          minOccurs="0" />
  <xs:element name="CognomeQuietanzante"             type="String60LatinType"      minOccurs="0" />
  <xs:element name="NomeQuietanzante"              type="String60LatinType"      minOccurs="0" />
  <xs:element name="CFQuietanzante"             type="CodiceFiscalePFType"      minOccurs="0" />
  <xs:element name="TitoloQuietanzante"          type="TitoloType"          minOccurs="0" />
  <xs:element name="IstitutoFinanziario"       type="String80LatinType"       minOccurs="0" />
  <xs:element name="IBAN"                 type="IBANType"              minOccurs="0" />
  <xs:element name="ABI"                 type="ABIType"              minOccurs="0" />
  <xs:element name="CAB"                  type="CABType"              minOccurs="0" />
  <xs:element name="BIC"                 type="BICType"              minOccurs="0" />
  <xs:element name="ScontoPagamentoAnticipato"         type="Amount2DecimalType"        minOccurs="0" />
  <xs:element name="DataLimitePagamentoAnticipato" type="xs:date"                 minOccurs="0" />
  <xs:element name="PenalitaPagamentiRitardati"      type="Amount2DecimalType"         minOccurs="0" />
  <xs:element name="DataDecorrenzaPenale"           type="xs:date"            minOccurs="0" />
  <xs:element name="CodicePagamento"              type="String60Type"          minOccurs="0" />
 </xs:sequence>
</xs:complexType>
<xs:simpleType name="ModalitaPagamentoType">
 <xs:restriction base="xs:string">
  <xs:length value="4" />
  <xs:enumeration value="MP01">
   <xs:annotation>
```

<!-- PDF page 140 -->

```text
  <xs:documentation>contanti</xs:documentation>
 </xs:annotation>
</xs:enumeration>
<xs:enumeration value="MP02">
 <xs:annotation>
  <xs:documentation>assegno</xs:documentation>
 </xs:annotation>
</xs:enumeration>
<xs:enumeration value="MP03">
 <xs:annotation>
  <xs:documentation>assegno circolare</xs:documentation>
 </xs:annotation>
</xs:enumeration>
<xs:enumeration value="MP04">
 <xs:annotation>
  <xs:documentation>contanti presso Tesoreria</xs:documentation>
 </xs:annotation>
</xs:enumeration>
<xs:enumeration value="MP05">
 <xs:annotation>
  <xs:documentation>bonifico</xs:documentation>
 </xs:annotation>
</xs:enumeration>
<xs:enumeration value="MP06">
 <xs:annotation>
  <xs:documentation>vaglia cambiario</xs:documentation>
 </xs:annotation>
</xs:enumeration>
<xs:enumeration value="MP07">
 <xs:annotation>
  <xs:documentation>bollettino bancario</xs:documentation>
 </xs:annotation>
</xs:enumeration>
<xs:enumeration value="MP08">
 <xs:annotation>
  <xs:documentation>carta di pagamento</xs:documentation>
 </xs:annotation>
</xs:enumeration>
<xs:enumeration value="MP09">
 <xs:annotation>
  <xs:documentation>RID</xs:documentation>
 </xs:annotation>
</xs:enumeration>
<xs:enumeration value="MP10">
 <xs:annotation>
  <xs:documentation>RID utenze</xs:documentation>
 </xs:annotation>
</xs:enumeration>
<xs:enumeration value="MP11">
 <xs:annotation>
  <xs:documentation>RID veloce</xs:documentation>
 </xs:annotation>
</xs:enumeration>
<xs:enumeration value="MP12">
 <xs:annotation>
```

<!-- PDF page 141 -->

```text
  <xs:documentation>RIBA</xs:documentation>
 </xs:annotation>
</xs:enumeration>
<xs:enumeration value="MP13">
 <xs:annotation>
  <xs:documentation>MAV</xs:documentation>
 </xs:annotation>
</xs:enumeration>
<xs:enumeration value="MP14">
 <xs:annotation>
  <xs:documentation>quietanza erario</xs:documentation>
 </xs:annotation>
</xs:enumeration>
<xs:enumeration value="MP15">
 <xs:annotation>
  <xs:documentation>giroconto su conti di contabilità speciale</xs:documentation>
 </xs:annotation>
</xs:enumeration>
<xs:enumeration value="MP16">
 <xs:annotation>
  <xs:documentation>domiciliazione bancaria</xs:documentation>
 </xs:annotation>
</xs:enumeration>
<xs:enumeration value="MP17">
 <xs:annotation>
  <xs:documentation>domiciliazione postale</xs:documentation>
 </xs:annotation>
</xs:enumeration>
<xs:enumeration value="MP18">
 <xs:annotation>
  <xs:documentation>bollettino di c/c postale</xs:documentation>
 </xs:annotation>
</xs:enumeration>
<xs:enumeration value="MP19">
 <xs:annotation>
  <xs:documentation>SEPA Direct Debit</xs:documentation>
 </xs:annotation>
</xs:enumeration>
<xs:enumeration value="MP20">
 <xs:annotation>
  <xs:documentation>SEPA Direct Debit CORE</xs:documentation>
 </xs:annotation>
</xs:enumeration>
<xs:enumeration value="MP21">
 <xs:annotation>
  <xs:documentation>SEPA Direct Debit B2B</xs:documentation>
 </xs:annotation>
</xs:enumeration>
<xs:enumeration value="MP22">
 <xs:annotation>
  <xs:documentation>Trattenuta su somme già riscosse</xs:documentation>
 </xs:annotation>
</xs:enumeration>
<xs:enumeration value="MP23">
 <xs:annotation>
```

<!-- PDF page 142 -->

```text
      <xs:documentation>PagoPA</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
   </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="IBANType">
   <xs:restriction base="xs:string">
    <xs:pattern value="[a-zA-Z]{2}[0-9]{2}[a-zA-Z0-9]{11,30}" />
   </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="BICType">
   <xs:restriction base="xs:string">
    <xs:pattern value="[A-Z]{6}[A-Z2-9][A-NP-Z0-9]([A-Z0-9]{3}){0,1}" />
   </xs:restriction>
 </xs:simpleType>
 <xs:complexType name="TerzoIntermediarioSoggettoEmittenteType">
   <xs:annotation>
    <xs:documentation>Blocco relativo ai dati del Terzo Intermediario che emette fattura elettronica per conto del
Cedente/Prestatore</xs:documentation>
   </xs:annotation>
   <xs:sequence>
    <xs:element name="DatiAnagrafici" type="DatiAnagraficiTerzoIntermediarioType" />
   </xs:sequence>
 </xs:complexType>
 <xs:complexType name="DatiAnagraficiTerzoIntermediarioType">
   <xs:sequence>
    <xs:element name="IdFiscaleIVA" type="IdFiscaleType" minOccurs="0" />
    <xs:element name="CodiceFiscale" type="CodiceFiscaleType" minOccurs="0" />
    <xs:element name="Anagrafica" type="AnagraficaType"                  />
   </xs:sequence>
 </xs:complexType>
 <xs:complexType name="AllegatiType">
   <xs:annotation>
    <xs:documentation>Blocco relativo ai dati di eventuali allegati</xs:documentation>
   </xs:annotation>
   <xs:sequence>
    <xs:element name="NomeAttachment"            type="String60LatinType"          />
    <xs:element name="AlgoritmoCompressione" type="String10Type"            minOccurs="0" />
    <xs:element name="FormatoAttachment" type="String10Type"               minOccurs="0" />
    <xs:element name="DescrizioneAttachment" type="String100LatinType" minOccurs="0" />
    <xs:element name="Attachment"            type="xs:base64Binary"           />
   </xs:sequence>
 </xs:complexType>
 <xs:complexType name="DettaglioLineeType">
   <xs:sequence>
    <xs:element name="NumeroLinea"               type="NumeroLineaType"                           />
    <xs:element name="TipoCessionePrestazione" type="TipoCessionePrestazioneType" minOccurs="0"                  />
    <xs:element name="CodiceArticolo"           type="CodiceArticoloType"      minOccurs="0" maxOccurs="unbounded"
/>
    <xs:element name="Descrizione"             type="String1000LatinType"                       />
    <xs:element name="Quantita"               type="QuantitaType"           minOccurs="0"           />
    <xs:element name="UnitaMisura"              type="String10Type"          minOccurs="0"            />
    <xs:element name="DataInizioPeriodo"          type="xs:date"            minOccurs="0"            />
    <xs:element name="DataFinePeriodo"            type="xs:date"            minOccurs="0"            />
    <xs:element name="PrezzoUnitario"            type="Amount8DecimalType"                          />
```

<!-- PDF page 143 -->

```text
   <xs:element name="ScontoMaggiorazione"                       type="ScontoMaggiorazioneType"          minOccurs="0"
maxOccurs="unbounded" />
   <xs:element name="PrezzoTotale"               type="Amount8DecimalType"                           />
   <xs:element name="AliquotaIVA"               type="RateType"                                 />
   <xs:element name="Ritenuta"                type="RitenutaType"            minOccurs="0"           />
   <xs:element name="Natura"                 type="NaturaType"             minOccurs="0"            />
   <xs:element name="RiferimentoAmministrazione" type="String20Type"                  minOccurs="0"           />
   <xs:element name="AltriDatiGestionali"                        type="AltriDatiGestionaliType"         minOccurs="0"
maxOccurs="unbounded" />
  </xs:sequence>
 </xs:complexType>
 <xs:complexType name="CodiceArticoloType">
  <xs:sequence>
   <xs:element name="CodiceTipo" type="String35Type" />
   <xs:element name="CodiceValore" type="String35LatinExtType" />
  </xs:sequence>
 </xs:complexType>
 <xs:complexType name="AltriDatiGestionaliType">
  <xs:sequence>
   <xs:element name="TipoDato"          type="String10Type"               />
   <xs:element name="RiferimentoTesto" type="String60LatinType" minOccurs="0" />
   <xs:element name="RiferimentoNumero" type="Amount8DecimalType" minOccurs="0" />
   <xs:element name="RiferimentoData" type="xs:date"               minOccurs="0" />
  </xs:sequence>
 </xs:complexType>
 <xs:simpleType name="RitenutaType">
  <xs:restriction base="xs:string">
   <xs:length value="2" />
   <xs:enumeration value="SI">
    <xs:annotation>
      <xs:documentation>SI = Cessione / Prestazione soggetta a ritenuta</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
  </xs:restriction>
 </xs:simpleType>
 <xs:complexType name="DatiRiepilogoType">
  <xs:sequence>
   <xs:element name="AliquotaIVA"           type="RateType"                 />
   <xs:element name="Natura"             type="NaturaType"         minOccurs="0" />
   <xs:element name="SpeseAccessorie"          type="Amount2DecimalType" minOccurs="0" />
   <xs:element name="Arrotondamento"            type="Amount8DecimalType" minOccurs="0" />
   <xs:element name="ImponibileImporto" type="Amount2DecimalType"                     />
   <xs:element name="Imposta"             type="Amount2DecimalType"              />
   <xs:element name="EsigibilitaIVA"       type="EsigibilitaIVAType" minOccurs="0" />
   <xs:element name="RiferimentoNormativo" type="String100LatinType" minOccurs="0" />
  </xs:sequence>
 </xs:complexType>
 <xs:simpleType name="EsigibilitaIVAType">
  <xs:restriction base="xs:string">
   <xs:minLength value="1" />
   <xs:maxLength value="1" />
   <xs:enumeration value="D">
    <xs:annotation>
      <xs:documentation>esigibilità differita</xs:documentation>
    </xs:annotation>
```

<!-- PDF page 144 -->

```text
    </xs:enumeration>
    <xs:enumeration value="I">
      <xs:annotation>
       <xs:documentation>esigibilità immediata</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="S">
      <xs:annotation>
       <xs:documentation>scissione dei pagamenti</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="NaturaType">
  <xs:restriction base="xs:string">
    <xs:enumeration value="N1">
      <xs:annotation>
       <xs:documentation>Escluse ex. art. 15 del D.P.R. 633/1972</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
<!-- IL CODICE SEGUENTE (N2) NON SARA' PIU' VALIDO PER LE FATTURE EMESSE A PARTIRE DAL PRIMO GENNAIO 2021--
>
    <xs:enumeration value="N2">
      <xs:annotation>
       <xs:documentation>Non soggette</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="N2.1">
      <xs:annotation>
       <xs:documentation>Non soggette ad IVA ai sensi degli artt. da 7 a 7-septies del DPR 633/72</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="N2.2">
      <xs:annotation>
       <xs:documentation>Non soggette - altri casi</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
<!-- IL CODICE SEGUENTE (N3) NON SARA' PIU' VALIDO PER LE FATTURE EMESSE A PARTIRE DAL PRIMO GENNAIO 2021--
>
    <xs:enumeration value="N3">
      <xs:annotation>
       <xs:documentation>Non imponibili</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="N3.1">
      <xs:annotation>
       <xs:documentation>Non Imponibili - esportazioni</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="N3.2">
      <xs:annotation>
       <xs:documentation>Non Imponibili - cessioni intracomunitarie</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="N3.3">
```

<!-- PDF page 145 -->

```text
      <xs:annotation>
       <xs:documentation>Non Imponibili - cessioni verso San Marino</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="N3.4">
      <xs:annotation>
       <xs:documentation>Non Imponibili - operazioni assimilate alle cessioni all'esportazione</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="N3.5">
      <xs:annotation>
       <xs:documentation>Non Imponibili - a seguito di dichiarazioni d'intento</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="N3.6">
      <xs:annotation>
       <xs:documentation>Non Imponibili - altre operazioni che non concorrono alla formazione del
plafond</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="N4">
      <xs:annotation>
       <xs:documentation>Esenti</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="N5">
      <xs:annotation>
       <xs:documentation>Regime del margine/IVA non esposta in fattura</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
<!-- IL CODICE SEGUENTE (N6) NON SARA' PIU' VALIDO PER LE FATTURE EMESSE A PARTIRE DAL PRIMO GENNAIO 2021--
>
    <xs:enumeration value="N6">
      <xs:annotation>
       <xs:documentation>Inversione contabile (per le operazioni in reverse charge ovvero nei casi di autofatturazione per
acquisti extra UE di servizi ovvero per importazioni di beni nei soli casi previsti)</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="N6.1">
      <xs:annotation>
       <xs:documentation>Inversione contabile - cessione di rottami e altri materiali di recupero</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="N6.2">
      <xs:annotation>
       <xs:documentation>Inversione contabile - cessione di oro e argento ai sensi della legge 7/2000 nonché di oreficeria
usata ad OPO</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="N6.3">
      <xs:annotation>
       <xs:documentation>Inversione contabile - subappalto nel settore edile</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="N6.4">
```

<!-- PDF page 146 -->

```text
     <xs:annotation>
      <xs:documentation>Inversione contabile - cessione di fabbricati</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="N6.5">
     <xs:annotation>
      <xs:documentation>Inversione contabile - cessione di telefoni cellulari</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="N6.6">
     <xs:annotation>
      <xs:documentation>Inversione contabile - cessione di prodotti elettronici</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="N6.7">
     <xs:annotation>
      <xs:documentation>Inversione contabile - prestazioni comparto edile e settori connessi</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="N6.8">
     <xs:annotation>
      <xs:documentation>Inversione contabile - operazioni settore energetico</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="N6.9">
     <xs:annotation>
      <xs:documentation>Inversione contabile - altri casi</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="N7">
     <xs:annotation>
      <xs:documentation>IVA assolta in altro stato UE (prestazione di servizi di telecomunicazioni, tele-radiodiffusione ed
elettronici ex art. 7-octies lett. a, b, art. 74-sexies DPR 633/72)</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="CodiceFiscaleType">
  <xs:restriction base="xs:string">
   <xs:pattern value="[A-Z0-9]{11,16}" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="CodiceFiscalePFType">
  <xs:restriction base="xs:string">
   <xs:pattern value="[A-Z0-9]{16}" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="CodEORIType">
  <xs:restriction base="xs:string">
   <xs:minLength value="13" />
   <xs:maxLength value="17" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="SocioUnicoType">
  <xs:restriction base="xs:string">
```

<!-- PDF page 147 -->

```text
  <xs:enumeration value="SU">
   <xs:annotation>
     <xs:documentation>socio unico</xs:documentation>
   </xs:annotation>
  </xs:enumeration>
  <xs:enumeration value="SM">
   <xs:annotation>
     <xs:documentation>più soci</xs:documentation>
   </xs:annotation>
  </xs:enumeration>
 </xs:restriction>
</xs:simpleType>
<xs:simpleType name="StatoLiquidazioneType">
 <xs:restriction base="xs:string">
  <xs:enumeration value="LS">
   <xs:annotation>
     <xs:documentation>in liquidazione</xs:documentation>
   </xs:annotation>
  </xs:enumeration>
  <xs:enumeration value="LN">
   <xs:annotation>
     <xs:documentation>non in liquidazione</xs:documentation>
   </xs:annotation>
  </xs:enumeration>
 </xs:restriction>
</xs:simpleType>
<xs:simpleType name="TipoCessionePrestazioneType">
 <xs:restriction base="xs:string">
  <xs:length value="2" />
  <xs:enumeration value="SC">
   <xs:annotation>
     <xs:documentation>Sconto</xs:documentation>
   </xs:annotation>
  </xs:enumeration>
  <xs:enumeration value="PR">
   <xs:annotation>
     <xs:documentation>Premio</xs:documentation>
   </xs:annotation>
  </xs:enumeration>
  <xs:enumeration value="AB">
   <xs:annotation>
     <xs:documentation>Abbuono</xs:documentation>
   </xs:annotation>
  </xs:enumeration>
  <xs:enumeration value="AC">
   <xs:annotation>
     <xs:documentation>Spesa accessoria</xs:documentation>
   </xs:annotation>
  </xs:enumeration>
 </xs:restriction>
</xs:simpleType>
<xs:simpleType name="TitoloType">
 <xs:restriction base="xs:normalizedString">
  <xs:whiteSpace value="collapse" />
  <xs:pattern value="(\p{IsBasicLatin}{2,10})" />
```

<!-- PDF page 148 -->

```text
 </xs:restriction>
</xs:simpleType>
<xs:simpleType name="String10Type">
 <xs:restriction base="xs:normalizedString">
  <xs:pattern value="(\p{IsBasicLatin}{1,10})" />
 </xs:restriction>
</xs:simpleType>
<xs:simpleType name="String15Type">
 <xs:restriction base="xs:normalizedString">
  <xs:pattern value="(\p{IsBasicLatin}{1,15})" />
 </xs:restriction>
</xs:simpleType>
<xs:simpleType name="String20Type">
 <xs:restriction base="xs:normalizedString">
  <xs:pattern value="(\p{IsBasicLatin}{1,20})" />
 </xs:restriction>
</xs:simpleType>
<xs:simpleType name="String35Type">
 <xs:restriction base="xs:normalizedString">
  <xs:pattern value="(\p{IsBasicLatin}{1,35})" />
 </xs:restriction>
</xs:simpleType>
<xs:simpleType name="String35LatinExtType">
 <xs:restriction base="xs:normalizedString">
  <xs:minLength value="1" />
  <xs:maxLength value="35" />
 </xs:restriction>
</xs:simpleType>
<xs:simpleType name="String60Type">
 <xs:restriction base="xs:normalizedString">
  <xs:pattern value="(\p{IsBasicLatin}{1,60})" />
 </xs:restriction>
</xs:simpleType>
<xs:simpleType name="String80Type">
 <xs:restriction base="xs:normalizedString">
  <xs:pattern value="(\p{IsBasicLatin}{1,80})" />
 </xs:restriction>
</xs:simpleType>
<xs:simpleType name="String100Type">
 <xs:restriction base="xs:normalizedString">
  <xs:pattern value="(\p{IsBasicLatin}{1,100})" />
 </xs:restriction>
</xs:simpleType>
<xs:simpleType name="String60LatinType">
 <xs:restriction base="xs:normalizedString">
  <xs:pattern value="[\p{IsBasicLatin}\p{IsLatin-1Supplement}]{1,60}" />
 </xs:restriction>
</xs:simpleType>
<xs:simpleType name="String80LatinType">
 <xs:restriction base="xs:normalizedString">
  <xs:pattern value="[\p{IsBasicLatin}\p{IsLatin-1Supplement}]{1,80}" />
 </xs:restriction>
</xs:simpleType>
<xs:simpleType name="String100LatinType">
 <xs:restriction base="xs:normalizedString">
```

<!-- PDF page 149 -->

```text
   <xs:pattern value="[\p{IsBasicLatin}\p{IsLatin-1Supplement}]{1,100}" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="String200LatinType">
  <xs:restriction base="xs:normalizedString">
   <xs:pattern value="[\p{IsBasicLatin}\p{IsLatin-1Supplement}]{1,200}" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="String1000LatinType">
  <xs:restriction base="xs:normalizedString">
   <xs:pattern value="[\p{IsBasicLatin}\p{IsLatin-1Supplement}]{1,1000}" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="ProvinciaType">
  <xs:restriction base="xs:string">
   <xs:pattern value="[A-Z]{2}" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="NazioneType">
  <xs:restriction base="xs:string">
   <xs:pattern value="[A-Z]{2}" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="DivisaType">
  <xs:restriction base="xs:string">
   <xs:pattern value="[A-Z]{3}" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="TipoResaType">
  <xs:restriction base="xs:string">
   <xs:pattern value="[A-Z]{3}" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="NumeroCivicoType">
  <xs:restriction base="xs:normalizedString">
   <xs:pattern value="(\p{IsBasicLatin}{1,8})" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="BolloVirtualeType">
  <xs:restriction base="xs:string">
   <xs:enumeration value="SI" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="TelFaxType">
  <xs:restriction base="xs:normalizedString">
   <xs:pattern value="(\p{IsBasicLatin}{5,12})" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="EmailType">
  <xs:restriction base="xs:token">
   <xs:maxLength value="256" />
   <xs:pattern value="([!#-'*+/-9=?A-Z^-~-]+(\.[!#-'*+/-9=?A-Z^-~-]+)*|&quot;(\[\]!#-[^-~ \t]|(\\[\t -~]))+&quot;)@([!#-
'*+/-9=?A-Z^-~-]+(\.[!#-'*+/-9=?A-Z^-~-]+)*|\[[\t -Z^-~]*\])" />
  </xs:restriction>
 </xs:simpleType>
```

<!-- PDF page 150 -->

```text
<xs:simpleType name="EmailContattiType">
 <xs:restriction base="xs:string">
  <xs:minLength value="7" />
  <xs:maxLength value="256" />
  <xs:pattern value=".+@.+[.]+.+" />
 </xs:restriction>
</xs:simpleType>
<!--                    NUMBERS                         -->
<xs:simpleType name="PesoType">
 <xs:restriction base="xs:decimal">
  <xs:pattern value="[0-9]{1,4}\.[0-9]{1,2}" />
 </xs:restriction>
</xs:simpleType>
<xs:simpleType name="Amount8DecimalType">
 <xs:restriction base="xs:decimal">
  <xs:pattern value="[\-]?[0-9]{1,11}\.[0-9]{2,8}" />
 </xs:restriction>
</xs:simpleType>
<xs:simpleType name="Amount2DecimalType">
 <xs:restriction base="xs:decimal">
  <xs:pattern value="[\-]?[0-9]{1,11}\.[0-9]{2}" />
 </xs:restriction>
</xs:simpleType>
<xs:simpleType name="RateType">
 <xs:restriction base="xs:decimal">
  <xs:maxInclusive value="100.00" />
  <xs:pattern value="[0-9]{1,3}\.[0-9]{2}" />
 </xs:restriction>
</xs:simpleType>
<xs:simpleType name="RiferimentoFaseType">
 <xs:restriction base="xs:integer">
  <xs:minInclusive value="1" />
  <xs:maxInclusive value="999" />
 </xs:restriction>
</xs:simpleType>
<xs:simpleType name="NumeroColliType">
 <xs:restriction base="xs:integer">
  <xs:minInclusive value="1" />
  <xs:maxInclusive value="9999" />
 </xs:restriction>
</xs:simpleType>
<xs:simpleType name="NumeroLineaType">
 <xs:restriction base="xs:integer">
  <xs:minInclusive value="1" />
  <xs:maxInclusive value="9999" />
 </xs:restriction>
</xs:simpleType>
<xs:simpleType name="CAPType">
 <xs:restriction base="xs:string">
  <xs:pattern value="[0-9][0-9][0-9][0-9][0-9]" />
 </xs:restriction>
</xs:simpleType>
<xs:simpleType name="ABIType">
 <xs:restriction base="xs:string">
  <xs:pattern value="[0-9][0-9][0-9][0-9][0-9]" />
```

<!-- PDF page 151 -->

```text
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="CABType">
  <xs:restriction base="xs:string">
   <xs:pattern value="[0-9][0-9][0-9][0-9][0-9]" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="GiorniTerminePagamentoType">
  <xs:restriction base="xs:integer">
   <xs:minInclusive value="0" />
   <xs:maxInclusive value="999" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="QuantitaType">
  <xs:restriction base="xs:decimal">
   <xs:pattern value="[0-9]{1,12}\.[0-9]{2,8}" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="DataFatturaType">
  <xs:restriction base="xs:date">
   <xs:minInclusive value="1970-01-01" />
  </xs:restriction>
 </xs:simpleType>
</xs:schema>
```

<!-- PDF page 152 -->

```text
1. RAPPRESENTAZIONE TABELLARE FATTURA ORDINARIA
```

<!-- PDF page 160 -->

```text
APPENDICE 3: FATTURA SEMPLIFICATA

1. SCHEMA XSD FATTURA SEMPLIFICATA

<?xml version="1.0" encoding="utf-8"?>
<xs:schema
         xmlns:xs="http://www.w3.org/2001/XMLSchema"
         xmlns:ds="http://www.w3.org/2000/09/xmldsig#"
         xmlns="http://ivaservizi.agenziaentrate.gov.it/docs/xsd/fatture/v1.0"
         targetNamespace="http://ivaservizi.agenziaentrate.gov.it/docs/xsd/fatture/v1.0"
         version="1.0.2">

 <xs:import namespace="http://www.w3.org/2000/09/xmldsig#" schemaLocation="http://www.w3.org/TR/2002/REC-
xmldsig-core-20020212/xmldsig-core-schema.xsd" />

 <xs:element name="FatturaElettronicaSemplificata" type="FatturaElettronicaType">
  <xs:annotation>
   <xs:documentation>
                                     XML schema fatture destinate a privati in forma semplificata 1.0.2
                            </xs:documentation>
  </xs:annotation>
 </xs:element>
 <xs:complexType name="FatturaElettronicaType">
  <xs:sequence>
   <xs:element name="FatturaElettronicaHeader" type="FatturaElettronicaHeaderType" />
   <xs:element name="FatturaElettronicaBody" type="FatturaElettronicaBodyType" maxOccurs="unbounded" />
   <xs:element ref="ds:Signature" minOccurs="0" />
  </xs:sequence>
  <xs:attribute name="versione" type="FormatoTrasmissioneType" use="required" />
  <xs:attribute name="SistemaEmittente" type="String10Type" use="optional" />
 </xs:complexType>
 <xs:complexType name="FatturaElettronicaHeaderType">
  <xs:sequence>
   <xs:element name="DatiTrasmissione" type="DatiTrasmissioneType" />
   <xs:element name="CedentePrestatore" type="CedentePrestatoreType" />
   <xs:element name="CessionarioCommittente" type="CessionarioCommittenteType" />
   <xs:element name="SoggettoEmittente" type="SoggettoEmittenteType" minOccurs="0" />
  </xs:sequence>
 </xs:complexType>
 <xs:complexType name="FatturaElettronicaBodyType">
  <xs:sequence>
   <xs:element name="DatiGenerali" type="DatiGeneraliType" />
   <xs:element name="DatiBeniServizi" type="DatiBeniServiziType" maxOccurs="unbounded" />
   <xs:element name="Allegati" type="AllegatiType" minOccurs="0" maxOccurs="unbounded" />
  </xs:sequence>
 </xs:complexType>
 <xs:complexType name="DatiTrasmissioneType">
  <xs:annotation>
   <xs:documentation>Blocco relativo ai dati di trasmissione della Fattura Elettronica</xs:documentation>
  </xs:annotation>
  <xs:sequence>
   <xs:element name="IdTrasmittente" type="IdFiscaleType" />
   <xs:element name="ProgressivoInvio" type="String10Type" />
```

<!-- PDF page 161 -->

```text
  <xs:element name="FormatoTrasmissione" type="FormatoTrasmissioneType" />
  <xs:element name="CodiceDestinatario" type="CodiceDestinatarioType" />
  <xs:element name="PECDestinatario" type="EmailType" minOccurs="0" />
 </xs:sequence>
</xs:complexType>
<xs:simpleType name="CodiceDestinatarioType">
 <xs:restriction base="xs:string">
  <xs:pattern value="[A-Z0-9]{7}" />
 </xs:restriction>
</xs:simpleType>
<xs:complexType name="IdFiscaleType">
 <xs:sequence>
  <xs:element name="IdPaese" type="NazioneType" />
  <xs:element name="IdCodice" type="CodiceType" />
 </xs:sequence>
</xs:complexType>
<xs:simpleType name="CodiceType">
 <xs:restriction base="xs:string">
  <xs:minLength value="1" />
  <xs:maxLength value="28" />
 </xs:restriction>
</xs:simpleType>
<xs:simpleType name="FormatoTrasmissioneType">
 <xs:restriction base="xs:string">
  <xs:length value="5" />
          <xs:enumeration value="FSM10">
            <xs:annotation>
             <xs:documentation>Fattura verso privati semplificata</xs:documentation>
            </xs:annotation>
          </xs:enumeration>
 </xs:restriction>
</xs:simpleType>
<xs:complexType name="DatiGeneraliType">
 <xs:annotation>
  <xs:documentation>Blocco relativo ai Dati Generali della Fattura Elettronica</xs:documentation>
 </xs:annotation>
 <xs:sequence>
  <xs:element name="DatiGeneraliDocumento" type="DatiGeneraliDocumentoType" />
  <xs:element name="DatiFatturaRettificata" type="DatiFatturaRettificataType" minOccurs="0" />
 </xs:sequence>
</xs:complexType>
<xs:complexType name="DatiGeneraliDocumentoType">
 <xs:sequence>
  <xs:element name="TipoDocumento" type="TipoDocumentoType" />
  <xs:element name="Divisa" type="DivisaType" />
  <xs:element name="Data" type="DataFatturaType" />
  <xs:element name="Numero" type="String20Type" />
  <xs:element name="BolloVirtuale" type="BolloVirtualeType" minOccurs="0" />
 </xs:sequence>
</xs:complexType>
<xs:complexType name="DatiFatturaRettificataType">
 <xs:sequence>
  <xs:element name="NumeroFR" type="String20Type" />
  <xs:element name="DataFR" type="DataFatturaType" />
  <xs:element name="ElementiRettificati" type="String1000LatinType" />
```

<!-- PDF page 162 -->

```text
  </xs:sequence>
 </xs:complexType>
 <xs:simpleType name="Art73Type">
  <xs:restriction base="xs:string">
   <xs:length value="2" />
   <xs:enumeration value="SI">
    <xs:annotation>
      <xs:documentation>SI = Documento emesso secondo modalità e termini stabiliti con DM ai sensi dell'art. 73 DPR
633/72</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="TipoDocumentoType">
  <xs:restriction base="xs:string">
   <xs:length value="4" />
   <xs:enumeration value="TD07">
    <xs:annotation>
      <xs:documentation>Fattura semplificata</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TD08">
    <xs:annotation>
      <xs:documentation>Nota di credito semplificata</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TD09">
    <xs:annotation>
      <xs:documentation>Nota di debito semplificata</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
  </xs:restriction>
 </xs:simpleType>
 <xs:complexType name="IndirizzoType">
  <xs:sequence>
   <xs:element name="Indirizzo" type="String60LatinType" />
   <xs:element name="NumeroCivico" type="NumeroCivicoType" minOccurs="0" />
   <xs:element name="CAP" type="CAPType" />
   <xs:element name="Comune" type="String60LatinType" />
   <xs:element name="Provincia" type="ProvinciaType" minOccurs="0" />
   <xs:element name="Nazione" type="NazioneType" default="IT" />
  </xs:sequence>
 </xs:complexType>
 <xs:simpleType name="SoggettoEmittenteType">
  <xs:restriction base="xs:string">
   <xs:length value="2" />
   <xs:enumeration value="CC">
    <xs:annotation>
      <xs:documentation>Cessionario / Committente</xs:documentation>
    </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="TZ">
    <xs:annotation>
      <xs:documentation>Terzo</xs:documentation>
    </xs:annotation>
```

<!-- PDF page 163 -->

```text
  </xs:enumeration>
 </xs:restriction>
</xs:simpleType>
<xs:complexType name="CedentePrestatoreType">
 <xs:annotation>
  <xs:documentation>Blocco relativo ai dati del Cedente / Prestatore</xs:documentation>
 </xs:annotation>
 <xs:sequence>
  <xs:element name="IdFiscaleIVA" type="IdFiscaleType" />
  <xs:element name="CodiceFiscale" type="CodiceFiscaleType" minOccurs="0" />
  <xs:choice>
   <xs:sequence>
     <xs:element name="Denominazione" type="String80LatinType"/>
   </xs:sequence>
   <xs:sequence>
     <xs:element name="Nome"         type="String60LatinType"/>
     <xs:element name="Cognome"        type="String60LatinType"/>
   </xs:sequence>
  </xs:choice>
  <xs:element name="Sede" type="IndirizzoType" />
  <xs:element name="StabileOrganizzazione" type="IndirizzoType" minOccurs="0" />
  <xs:element name="RappresentanteFiscale" type="RappresentanteFiscaleType" minOccurs="0" />
  <xs:element name="IscrizioneREA" type="IscrizioneREAType" minOccurs="0" />
          <xs:element name="RegimeFiscale" type="RegimeFiscaleType" />
 </xs:sequence>
</xs:complexType>
<xs:simpleType name="RegimeFiscaleType">
 <xs:restriction base="xs:string">
  <xs:length value="4" />
  <xs:enumeration value="RF01">
   <xs:annotation>
     <xs:documentation> Regime ordinario</xs:documentation>
   </xs:annotation>
  </xs:enumeration>
  <xs:enumeration value="RF02">
   <xs:annotation>
     <xs:documentation>Regime dei contribuenti minimi (art. 1,c.96-117, L. 244/2007)</xs:documentation>
   </xs:annotation>
  </xs:enumeration>
  <xs:enumeration value="RF04">
   <xs:annotation>
     <xs:documentation>Agricoltura e attività connesse e pesca (artt. 34 e 34-bis, D.P.R. 633/1972)</xs:documentation>
   </xs:annotation>
  </xs:enumeration>
  <xs:enumeration value="RF05">
   <xs:annotation>
     <xs:documentation>Vendita sali e tabacchi (art. 74, c.1, D.P.R. 633/1972)</xs:documentation>
   </xs:annotation>
  </xs:enumeration>
  <xs:enumeration value="RF06">
   <xs:annotation>
     <xs:documentation>Commercio dei fiammiferi (art. 74, c.1, D.P.R. 633/1972)</xs:documentation>
   </xs:annotation>
  </xs:enumeration>
  <xs:enumeration value="RF07">
```

<!-- PDF page 164 -->

```text
     <xs:annotation>
      <xs:documentation>Editoria (art. 74, c.1, D.P.R. 633/1972)</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF08">
     <xs:annotation>
      <xs:documentation>Gestione di servizi di telefonia pubblica (art. 74, c.1, D.P.R. 633/1972)</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF09">
     <xs:annotation>
      <xs:documentation>Rivendita di documenti di trasporto pubblico e di sosta (art. 74, c.1, D.P.R.
633/1972)</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF10">
     <xs:annotation>
      <xs:documentation>Intrattenimenti, giochi e altre attività     di cui alla tariffa allegata al D.P.R. 640/72 (art. 74, c.6,
D.P.R. 633/1972)</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF11">
     <xs:annotation>
      <xs:documentation>Agenzie di viaggi e turismo (art. 74-ter, D.P.R. 633/1972)</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF12">
     <xs:annotation>
      <xs:documentation>Agriturismo (art. 5, c.2, L. 413/1991)</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF13">
     <xs:annotation>
      <xs:documentation>Vendite a domicilio (art. 25-bis, c.6, D.P.R. 600/1973)</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF14">
     <xs:annotation>
      <xs:documentation>Rivendita di beni usati, di oggetti          d’arte, d’antiquariato o da collezione (art.            36,
D.L. 41/1995)</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF15">
     <xs:annotation>
      <xs:documentation>Agenzie di vendite all’asta di oggetti d’arte, antiquariato o da collezione (art. 40-bis, D.L.
41/1995)</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF16">
     <xs:annotation>
      <xs:documentation>IVA per cassa P.A. (art. 6, c.5, D.P.R. 633/1972)</xs:documentation>
     </xs:annotation>
   </xs:enumeration>
   <xs:enumeration value="RF17">
     <xs:annotation>
```

<!-- PDF page 165 -->

```text
     <xs:documentation>IVA per cassa (art. 32-bis, D.L. 83/2012)</xs:documentation>
   </xs:annotation>
  </xs:enumeration>
  <xs:enumeration value="RF18">
   <xs:annotation>
     <xs:documentation>Altro</xs:documentation>
   </xs:annotation>
  </xs:enumeration>
          <xs:enumeration value="RF19">
   <xs:annotation>
     <xs:documentation>Regime forfettario</xs:documentation>
   </xs:annotation>
  </xs:enumeration>
          <xs:enumeration value="RF20">
   <xs:annotation>
     <xs:documentation>Regime transfrontaliero di Franchigia IVA (Direttiva UE 2020/285)</xs:documentation>
   </xs:annotation>
  </xs:enumeration>
 </xs:restriction>
</xs:simpleType>
<xs:complexType name="IscrizioneREAType">
 <xs:sequence>
  <xs:element name="Ufficio" type="ProvinciaType" />
  <xs:element name="NumeroREA" type="String20Type" />
  <xs:element name="CapitaleSociale" type="Amount2DecimalType" minOccurs="0" />
  <xs:element name="SocioUnico" type="SocioUnicoType" minOccurs="0" />
  <xs:element name="StatoLiquidazione" type="StatoLiquidazioneType" />
 </xs:sequence>
</xs:complexType>
<xs:complexType name="RappresentanteFiscaleType">
 <xs:annotation>
  <xs:documentation>Blocco relativo ai dati del Rappresentante Fiscale</xs:documentation>
 </xs:annotation>
 <xs:sequence>
          <xs:element name="IdFiscaleIVA" type="IdFiscaleType" />
  <xs:choice>
   <xs:sequence>
     <xs:element name="Denominazione" type="String80LatinType"/>
   </xs:sequence>
   <xs:sequence>
     <xs:element name="Nome"         type="String60LatinType"/>
     <xs:element name="Cognome"        type="String60LatinType"/>
   </xs:sequence>
  </xs:choice>
 </xs:sequence>
</xs:complexType>
<xs:complexType name="CessionarioCommittenteType">
 <xs:annotation>
  <xs:documentation>Blocco relativo ai dati del Cessionario / Committente</xs:documentation>
 </xs:annotation>
 <xs:sequence>
  <xs:element name="IdentificativiFiscali" type="IdentificativiFiscaliType"         />
  <xs:element name="AltriDatiIdentificativi" type="AltriDatiIdentificativiType" minOccurs="0" />
 </xs:sequence>
</xs:complexType>
```

<!-- PDF page 166 -->

```text
<xs:complexType name="IdentificativiFiscaliType">
 <xs:sequence>
  <xs:element name="IdFiscaleIVA" type="IdFiscaleType" minOccurs="0" />
  <xs:element name="CodiceFiscale" type="CodiceFiscaleType" minOccurs="0" />
 </xs:sequence>
</xs:complexType>
<xs:complexType name="AltriDatiIdentificativiType">
 <xs:sequence>
  <xs:choice>
   <xs:sequence>
     <xs:element name="Denominazione" type="String80LatinType"/>
   </xs:sequence>
   <xs:sequence>
     <xs:element name="Nome"        type="String60LatinType"/>
     <xs:element name="Cognome"       type="String60LatinType"/>
   </xs:sequence>
  </xs:choice>
  <xs:element name="Sede" type="IndirizzoType" />
  <xs:element name="StabileOrganizzazione" type="IndirizzoType" minOccurs="0" />
  <xs:element name="RappresentanteFiscale" type="RappresentanteFiscaleType" minOccurs="0" />
 </xs:sequence>
</xs:complexType>
<xs:complexType name="DatiBeniServiziType">
 <xs:annotation>
  <xs:documentation>Blocco relativo ai dati di Beni Servizi della Fattura   Elettronica</xs:documentation>
 </xs:annotation>
 <xs:sequence>
  <xs:element name="Descrizione" type="String1000LatinType" />
  <xs:element name="Importo" type="Amount2DecimalType" />
  <xs:element name="DatiIVA" type="DatiIVAType" />
  <xs:element name="Natura" type="NaturaType" minOccurs="0" />
  <xs:element name="RiferimentoNormativo" type="String100LatinType" minOccurs="0" />
 </xs:sequence>
</xs:complexType>
<xs:complexType name="DatiIVAType">
 <xs:sequence>
  <xs:element name="Imposta" type="Amount2DecimalType" minOccurs="0" />
          <xs:element name="Aliquota" type="RateType" minOccurs="0"/>
         </xs:sequence>
</xs:complexType>
<xs:complexType name="AllegatiType">
 <xs:annotation>
  <xs:documentation>Blocco relativo ai dati di eventuali allegati</xs:documentation>
 </xs:annotation>
 <xs:sequence>
  <xs:element name="NomeAttachment" type="String60LatinType" />
  <xs:element name="AlgoritmoCompressione" type="String10Type" minOccurs="0" />
  <xs:element name="FormatoAttachment" type="String10Type" minOccurs="0" />
  <xs:element name="DescrizioneAttachment" type="String100LatinType" minOccurs="0" />
  <xs:element name="Attachment" type="xs:base64Binary" />
 </xs:sequence>
</xs:complexType>
<xs:simpleType name="NaturaType">
 <xs:restriction base="xs:string">
  <xs:enumeration value="N1">
```

<!-- PDF page 167 -->

```text
      <xs:annotation>
       <xs:documentation>Escluse ex. art. 15 del D.P.R. 633/1972</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
<!-- IL CODICE SEGUENTE (N2) NON SARA' PIU' VALIDO PER LE FATTURE EMESSE A PARTIRE DAL PRIMO GENNAIO 2021--
>
    <xs:enumeration value="N2">
      <xs:annotation>
       <xs:documentation>Non soggette</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="N2.1">
      <xs:annotation>
       <xs:documentation>Non soggette ad IVA ai sensi degli artt. da 7 a 7-septies del DPR 633/72</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="N2.2">
      <xs:annotation>
       <xs:documentation>Non soggette - altri casi</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
<!-- IL CODICE SEGUENTE (N3) NON SARA' PIU' VALIDO PER LE FATTURE EMESSE A PARTIRE DAL PRIMO GENNAIO 2021--
>
    <xs:enumeration value="N3">
      <xs:annotation>
       <xs:documentation>Non imponibili</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="N3.1">
      <xs:annotation>
       <xs:documentation>Non Imponibili - esportazioni</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="N3.2">
      <xs:annotation>
       <xs:documentation>Non Imponibili - cessioni intracomunitarie</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="N3.3">
      <xs:annotation>
       <xs:documentation>Non Imponibili - cessioni verso San Marino</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="N3.4">
      <xs:annotation>
       <xs:documentation>Non Imponibili - operazioni assimilate alle cessioni all'esportazione</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="N3.5">
      <xs:annotation>
       <xs:documentation>Non Imponibili - a seguito di dichiarazioni d'intento</xs:documentation>
      </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="N3.6">
      <xs:annotation>
```

<!-- PDF page 168 -->

```text
      <xs:documentation>Non Imponibili - altre operazioni che non concorrono               alla   formazione   del
plafond</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="N4">
     <xs:annotation>
      <xs:documentation>Esenti</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="N5">
     <xs:annotation>
      <xs:documentation>Regime del margine/IVA non esposta in fattura</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="CodiceFiscaleType">
  <xs:restriction base="xs:string">
    <xs:pattern value="[A-Z0-9]{11,16}" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="SocioUnicoType">
  <xs:restriction base="xs:string">
    <xs:enumeration value="SU">
     <xs:annotation>
      <xs:documentation>socio unico</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="SM">
     <xs:annotation>
      <xs:documentation>più soci</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="StatoLiquidazioneType">
  <xs:restriction base="xs:string">
    <xs:enumeration value="LS">
     <xs:annotation>
      <xs:documentation>in liquidazione</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
    <xs:enumeration value="LN">
     <xs:annotation>
      <xs:documentation>non in liquidazione</xs:documentation>
     </xs:annotation>
    </xs:enumeration>
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="String10Type">
  <xs:restriction base="xs:normalizedString">
    <xs:pattern value="(\p{IsBasicLatin}{1,10})" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="String20Type">
```

<!-- PDF page 169 -->

```text
  <xs:restriction base="xs:normalizedString">
   <xs:pattern value="(\p{IsBasicLatin}{1,20})" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="String60LatinType">
  <xs:restriction base="xs:normalizedString">
   <xs:pattern value="[\p{IsBasicLatin}\p{IsLatin-1Supplement}]{1,60}" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="String80LatinType">
  <xs:restriction base="xs:normalizedString">
   <xs:pattern value="[\p{IsBasicLatin}\p{IsLatin-1Supplement}]{1,80}" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="String100LatinType">
  <xs:restriction base="xs:normalizedString">
   <xs:pattern value="[\p{IsBasicLatin}\p{IsLatin-1Supplement}]{1,100}" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="String1000LatinType">
  <xs:restriction base="xs:normalizedString">
   <xs:pattern value="[\p{IsBasicLatin}\p{IsLatin-1Supplement}]{1,1000}" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="ProvinciaType">
  <xs:restriction base="xs:string">
   <xs:pattern value="[A-Z]{2}" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="NazioneType">
  <xs:restriction base="xs:string">
   <xs:pattern value="[A-Z]{2}" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="DivisaType">
  <xs:restriction base="xs:string">
   <xs:pattern value="[A-Z]{3}" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="NumeroCivicoType">
  <xs:restriction base="xs:normalizedString">
   <xs:pattern value="(\p{IsBasicLatin}{1,8})" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="EmailType">
  <xs:restriction base="xs:token">
   <xs:maxLength value="256" />
   <xs:pattern value="([!#-'*+/-9=?A-Z^-~-]+(\.[!#-'*+/-9=?A-Z^-~-]+)*|&quot;(\[\]!#-[^-~ \t]|(\\[\t -~]))+&quot;)@([!#-
'*+/-9=?A-Z^-~-]+(\.[!#-'*+/-9=?A-Z^-~-]+)*|\[[\t -Z^-~]*\])" />
          </xs:restriction>
 </xs:simpleType>
 <!--                     NUMBERS                             -->
 <xs:simpleType name="Amount2DecimalType">
  <xs:restriction base="xs:decimal">
   <xs:pattern value="[\-]?[0-9]{1,11}\.[0-9]{2}" />
```

<!-- PDF page 170 -->

```text
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="RateType">
  <xs:restriction base="xs:decimal">
   <xs:maxInclusive value="100.00" />
   <xs:pattern value="[0-9]{1,3}\.[0-9]{2}" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="CAPType">
  <xs:restriction base="xs:string">
   <xs:pattern value="[0-9][0-9][0-9][0-9][0-9]" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="DataFatturaType">
  <xs:restriction base="xs:date">
   <xs:minInclusive value="1970-01-01" />
  </xs:restriction>
 </xs:simpleType>
 <xs:simpleType name="BolloVirtualeType">
  <xs:restriction base="xs:string">
   <xs:enumeration value="SI" />
  </xs:restriction>
 </xs:simpleType>
</xs:schema>
```

<!-- PDF page 171 -->

```text
1. RAPPRESENTAZIONE TABELLARE FATTURA SEMPLIFICATA
```

<!-- PDF page 175 -->

```text
APPENDICE 4: RICEVUTE DEL PROCESSO DI FATTURAZIONE
I messaggi per la gestione delle ricevute/notifiche da inviare al soggetto trasmittente
da parte del SdI, o al SdI da parte del soggetto ricevente, sono contenuti in file XML,
descritti dal file MessaggiFatturaTypes_v1.0.xsd (v. paragrafo 10.2) e predisposti
secondo le specifiche riportate di seguito.
Di seguito si fornisce una breve descrizione del significato delle colonne presenti
nelle tabelle:

          Elemento XML:                       è il nome effettivo del tag XML utilizzato
                                              in fase di compilazione del file;
          Descrizione Funzionale:             indica una descrizione di tipo funzionale
                                              dell’Elemento XML;
          Formati e Valori Ammessi:           indica il tipo di formato del dato e, ove
                                              previste delle restrizioni sul dato,
                                              l’insieme dei valori ammessi per
                                              quell’Elemento XML (dati racchiusi tra
                                              parentesi quadra);
          Obbligatorietà e Occorrenze:        indica se il dato deve essere
                                              obbligatoriamente presente o meno, e la
                                              sua molteplicità:
            <0.1> dato facoltativo; se presente, può figurare al massimo una volta
            <0.N> dato facoltativo; se presente, può figurare N volte
            <1.1> dato obbligatorio; figura al massimo una volta
            <1.N> dato obbligatorio; figura almeno una volta

           Dimensione min…max:               indica la dimensione minima e massima
                                             che può assumere l’Elemento XML; nel
                                             caso in cui sia indicato un solo numero di
                                             dimensione, la lunghezza del campo è
                                             esattamente uguale a quel preciso valore;
                                             la dicitura “Unbounded” sta per dimensioni
                                             illimitate.
```

<!-- PDF page 176 -->

```text
1.    DESCRIZIONE E REGOLE DI COMPILAZIONE

1.1   NOMENCLATURA DEI FILE PER LA TRASMISSIONE DI RICEVUTE/ NOTIFICHE
      Il nome dei file per la trasmissione delle ricevute/notifiche, deve rispettare
      la seguente nomenclatura generale:

        Nome del file fattura ricevuto        Tipo di            Progressivo
            senza estensione                 messaggio             univoco

      Il Nome del file fattura ricevuto senza estensione deve essere conforme alle
      regole definite nel paragrafo 1.2.2. Nel caso in cui il nome file non sia
      conforme e la sua lunghezza sia superiore ai 36 caratteri il nome sarà
      troncato ed i caratteri oltre il 36-esimo non saranno presenti nella notifica di
      scarto.
      Il Tipo di messaggio può assumere i seguenti valori:

         Valore     Descrizione
          RC        Ricevuta di consegna
          NS        Ricevuta di scarto
          MC        Ricevuta di impossibilità di recapito
          MT        Notifica dei metadati

      Il Progressivo univoco deve essere una stringa alfanumerica di lunghezza
      massima 3 caratteri e con valori ammessi [a-z], [A-Z], [0-9] che identifica
      univocamente ogni notifica / ricevuta relativa al file inviato.
      Il carattere di separazione degli elementi componenti il nome file
      corrisponde all’underscore (“_”), codice ASCII 95, l’estensione è sempre
      “.xml”.
      Se il SdI ha ricevuto un file di tipo compresso, di cui al precedente paragrafo
      1.2.2 lett. b), (es.: ITAAABBB99T99X999W_00001.zip ) e non è possibile
      accedere al suo contenuto perché “corrotto”, il nome del file con il quale il
      SdI inoltra al soggetto trasmittente la notifica di scarto è il seguente:
            ITAAABBB99T99X999W_00001_NS_001.xml

       1.1.1    RICEVUTA DI CONSEGNA DEL FILE AL DESTINATARIO
      È la ricevuta inviata dal SdI al soggetto trasmittente per comunicare
      l’avvenuta consegna del file al destinatario.
      Per tutti i canali trasmissivi essa è rappresentata da un file XML firmato
      (firma elettronica non qualificata) contenente le seguenti informazioni:
```

<!-- PDF page 177 -->

```text
                                                                                                    Obbligatorietà         Dim.
                          Descrizione
  Elemento XML                                             Formati e valori ammessi                       e
                          funzionale
                                                                                                     occorrenze        min-max

                       Numero attribuito
                        dal Sistema di
 IdentificativoSDI
                      Interscambio al file
                                                             Formato alfanumerico                       <1.1>          1 … 36
                            ricevuto

                       Nome attribuito al
                         file secondo le
     NomeFile
                       regole riportate su
                                                             Formato alfanumerico                       <1.1>          1 … 50
                      Disciplinare Tecnico

                          Hash del file
                      trasmesso calcolato
       Hash
                      con algoritmo SHA-
                                                             Formato alfanumerico                       <1.1>        unbounded
                              256

                       Data e ora in cui il
                                                 Il formato della data è rappresentato secondo il
                       file è stato ricevuto
 DataOraRicezione
                          dal Sistema di
                                               formato ISO 8601:2004, con la seguente precisione:       <1.1>              16
                                                               YYYY-MM-DD-HH:MM
                           Interscambio
                        Data e ora in cui il
                            file è stato         Il formato della data è rappresentato secondo il
DataOraConsegna          consegnato dal        formato ISO 8601:2004, con la seguente precisione:       <1.1>              16
                            Sistema di                         YYYY-MM-DD-HH:MM
                          Interscambio
                       Blocco contenente
                         le informazioni
                       relative al soggetto
   Destinatario
                        destinatario della
                                                               Campo complesso                          <1.1>
                        fattura (Codice e
                        Denominazione)
                       Opzionale. Valore
                      presente nel caso di
                           Ricevuta di
RiferimentoArchivio
                      consegna relativa a
                                                               Campo complesso                          <0.1>
                      fattura appartenente
                          a file archivio.

                        Identificativo del
    MessageId
                           messaggio
                                                             Formato alfanumerico                       <1.1>          1 … 36

                      Opzionale. Presente
                         solo nel caso di
                      messaggi inviati con
  PecMessageId
                          il canale PEC.
                                                             Formato alfanumerico                       <0.1>        unbounded
                      Identificativo proprio
                       del messaggio PEC
                       Opzionale. Campo
                             contenente
       Note                   eventuali                      Formato alfanumerico                       <0.1>        unbounded
                            informazioni
                             aggiuntive

                                                         Tabella 1
```

<!-- PDF page 178 -->

```text
La ricevuta è firmata mediante tecnologia XAdES, pertanto presenterà al
suo interno, oltre gli elementi XML su indicati, il tag ds:Signature. Esso fa
riferimento al namespace: http://www.w3.org/2000/09/xmldsig# .
```

<!-- PDF page 179 -->

```text
Figura 1
```

<!-- PDF page 180 -->

```text
                     1.1.2        RICEVUTA DI SCARTO
                     È la notifica inviata dal SdI al soggetto trasmittente nei casi in cui non
                     sia stato superato uno o più controlli tra quelli effettuati dal SdI sul file
                     ricevuto.
                     Per tutti i canali trasmissivi essa è rappresentata da un file XML firmato
                     (firma elettronica non qualificata) contenente le seguenti informazioni:

                                                                                                       Obbligatorietà         Dim.
                             Descrizione
  Elemento XML                                                Formati e valori ammessi                       e
                             funzionale
                                                                                                        occorrenze        min-max

                          Numero attribuito
                           dal Sistema di
 IdentificativoSDI
                         Interscambio al file
                                                                Formato alfanumerico                       <1.1>          1 … 36
                               ricevuto

                          Nome attribuito al
                            file secondo le
     NomeFile
                          regole riportate su
                                                                Formato alfanumerico                       <1.1>          1 … 50
                         Disciplinare Tecnico

                             Hash del file
                         trasmesso calcolato
       Hash
                         con algoritmo SHA-
                                                                Formato alfanumerico                       <1.1>        unbounded
                                 256

                          Data e ora in cui il
                                                    Il formato della data è rappresentato secondo il
                          file è stato ricevuto
 DataOraRicezione
                             dal Sistema di
                                                  formato ISO 8601:2004, con la seguente precisione:       <1.1>              16
                                                                  YYYY-MM-DD-HH:MM
                              Interscambio
                          Opzionale. Valore
                         presente nel caso di
                              Ricevuta di
RiferimentoArchivio
                         consegna relativa a
                                                                  Campo complesso                          <0.1>
                         fattura appartenente
                             a file archivio.

                           Lista degli errori
    ListaErrori
                                rilevati.
                                                                  Campo complesso                          <1.1>

                           Identificativo del
    MessageId
                              messaggio
                                                                Formato alfanumerico                       <1.1>          1 … 36

                         Opzionale. Presente
                           solo nel caso di
                         messaggi inviati con
  PecMessageId
                            il canale PEC.
                                                                Formato alfanumerico                       <0.1>        unbounded
                         Identificativo proprio
                         del messaggio PEC
                          Opzionale. Campo
                               contenente
       Note                     eventuali                       Formato alfanumerico                       <0.1>        unbounded
                              informazioni
                               aggiuntive

                                                            Tabella 2
```

<!-- PDF page 181 -->

```text
La ricevuta è firmata mediante tecnologia XAdES, pertanto presenterà al
suo interno, oltre gli elementi XML su indicati, il tag ds:Signature. Esso fa
riferimento al namespace: http://www.w3.org/2000/09/xmldsig# .

                             Figura 2
```

<!-- PDF page 182 -->

```text
                       1.1.3         RICEVUTA DI IMPOSSIBILITÀ DI RECAPITO
                       È la ricevuta inviata dal SdI al soggetto trasmittente nei casi in cui fallisca
                       l’operazione di consegna del file al destinatario.
                       Per tutti i canali trasmissivi essa è rappresentata da un file XML firmato
                       (firma elettronica non qualificata) contenente le seguenti informazioni:

                                                                                                          Obbligatorietà      Dim.
                                  Descrizione
   Elemento XML                                                   Formati e valori ammessi                      e
                                  funzionale
                                                                                                           occorrenze        min-max

                                Numero attribuito
                                 dal Sistema di
  IdentificativoSDI
                               Interscambio al file
                                                                   Formato alfanumerico                       <1.1>         1 … 36
                                     ricevuto

                             Nome attribuito al
                               file secondo le
      NomeFile
                             regole riportate su
                                                                    Formato alfanumerico                      <1.1>         1 … 50
                            Disciplinare Tecnico

                                Hash del file
                            trasmesso calcolato
        Hash
                            con algoritmo SHA-
                                                                    Formato alfanumerico                      <1.1>        unbounded
                                    256

                               Data e ora in cui il    Il formato della data è rappresentato secondo il
                               file è stato ricevuto        formato ISO 8601:2004, con la seguente
  DataOraRicezione
                                  dal Sistema di                           precisione:                        <1.1>           16
                                   Interscambio                      YYYY-MM-DD-HH:MM

                            Data entro la quale        Il formato della data è rappresentato secondo il
                            il file viene messo a           formato ISO 8601:2004, con la seguente
DataMessaADisposizione
                                  disposizione                             precisione:
                                                                                                              <1.1>           10
                             nell’area riservata                         YYYY-MM-DD
                             Opzionale. Valore
                            presente nel caso di
                                 Ricevuta di
 RiferimentoArchivio
                            consegna relativa a
                                                                     Campo complesso                          <0.1>
                            fattura appartenente
                                a file archivio.
                                 Opzionale.
                              Descrizione delle
     Descrizione
                                motivazioni di
                                                                    Formato alfanumerico                      <0.1>         1 … 255
                             mancata consegna
                                Identificativo del
     MessageId                                                     Formato alfanumerico                       <1.1>         1 … 36
                                   messaggio
                            Opzionale. Presente
                              solo nel caso di
                            messaggi inviati con
   PecMessageId
                               il canale PEC.
                                                                    Formato alfanumerico                      <0.1>        unbounded
                            Identificativo proprio
                            del messaggio PEC
                             Opzionale. Campo
                                  contenente
        Note                       eventuali                        Formato alfanumerico                      <0.1>        unbounded
                                 informazioni
                                  aggiuntive
                                                                  Tabella 3
```

<!-- PDF page 183 -->

```text
La ricevuta è firmata mediante tecnologia XAdES, pertanto presenterà al
suo interno, oltre gli elementi XML su indicati, il tag ds:Signature. Esso fa
riferimento al namespace: http://www.w3.org/2000/09/xmldsig# .

                             Figura 3
```

<!-- PDF page 184 -->

```text
                     1.1.4        METADATI DEL FILE FATTURA AL DESTINATARIO
                     È il file inviato dal SdI al soggetto ricevente insieme al file fattura e
                     contenente i dati principali di riferimento del file utili per l’elaborazione, ivi
                     compreso l’identificativo del SdI.

                                                                                      Obbligatorietà         Dim.
                             Descrizione
  Elemento XML                                       Formati e valori ammessi               e
                             funzionale
                                                                                       occorrenze        min-max

                           Numero attribuito
                            dal Sistema di
 IdentificativoSDI
                          Interscambio al file
                                                      Formato alfanumerico                <1.1>          1 … 36
                                ricevuto

                          Nome del file a cui
    NomeFile
                         si riferisce la notifica
                                                      Formato alfanumerico                <1.1>          1 … 50

                             Hash del file
                         trasmesso calcolato
      Hash
                         con algoritmo SHA-
                                                      Formato alfanumerico                <1.1>        unbounded
                                 256

                         Codice del soggetto
CodiceDestinatario       a cui è indirizzata la       Formato alfanumerico                <1,1>          1…7
                                fattura

                          Identificativo della
     Formato             versione del formato         Formato alfanumerico                <1,1>              5
                                fattura

                         Numero progressivo
                         che identifica l’invio,
                         normalmente vale 1,
  TentativiInvio           nel caso di più              Formato numerico                  <1.1>              1
                              tentativi il
                          progressivo viene
                            incrementato

                           Identificativo del
   MessageId
                              messaggio
                                                      Formato alfanumerico                <1.1>          1 … 36

                          Opzionale. Campo
                             contenente
       Note                   eventuali               Formato alfanumerico                <0.1>        unbounded
                            informazioni
                             aggiuntive

                                                    Tabella 8
```

<!-- PDF page 185 -->

```text
Figura 8
```

<!-- PDF page 186 -->

```text
 1.2         SCHEMA XSD MESSAGGIFATTURATYPES_V1.0

<?xml version="1.0" encoding="UTF-8"?>
<xsd:schema xmlns:xsd="http://www.w3.org/2001/XMLSchema"
         xmlns:ds="http://www.w3.org/2000/09/xmldsig#"
         xmlns:types="http://ivaservizi.agenziaentrate.gov.it/docs/xsd/fattura/messaggi/v1.0"
         targetNamespace="http://ivaservizi.agenziaentrate.gov.it/docs/xsd/fattura/messaggi/v1.0"
         version="1.0">

        <xsd:import                                          namespace="http://www.w3.org/2000/09/xmldsig#"
schemaLocation="http://www.w3.org/TR/2002/REC-xmldsig-core-20020212/xmldsig-core-schema.xsd"/>

         <!-- =================================================================== -->
         <!-- ELEMENTI PER TRASMITTENTI
                            -->
         <!-- =================================================================== -->
         <xsd:element name="RicevutaConsegna" type="types:RicevutaConsegna_Type" />
         <xsd:element name="RicevutaScarto" type="types:RicevutaScarto_Type" />
         <xsd:element name="RicevutaImpossibilitaRecapito" type="types:RicevutaImpossibilitaRecapito_Type" />

         <!-- =================================================================== -->
         <!-- ELEMENTI PER RICEVENTI
                                     -->
         <!-- =================================================================== -->
         <xsd:element name="FileMetadati" type="types:FileMetadati_Type" />

        <!-- =================================================================== -->
        <!-- TIPI EXTRA PER TRASMITTENTI
                    -->
        <!-- =================================================================== -->
        <xsd:complexType name="RicevutaConsegna_Type">
                   <xsd:sequence>
                             <xsd:element name="IdentificativoSdI" type="types:IdentificativoSdI_Type" />
                             <xsd:element name="NomeFile" type="types:NomeFile_Type" />
                             <xsd:element name="Hash" type="xsd:string" />
                             <xsd:element name="DataOraRicezione" type="xsd:dateTime" />
                             <xsd:element name="DataOraConsegna" type="xsd:dateTime" />
                             <xsd:element name="Destinatario" type="types:Destinatario_Type"/>
                             <xsd:element     name="RiferimentoArchivio"       type="types:RiferimentoArchivio_Type"
minOccurs="0" maxOccurs="1"/>
                             <xsd:element name="MessageId" type="types:MessageId_Type"/>
                             <xsd:element name="PecMessageId" type="types:PecMessageId_Type" minOccurs="0"
maxOccurs="1"/>
                             <xsd:element name="Note" type="xsd:string" minOccurs="0" maxOccurs="1"/>
                             <xsd:element ref="ds:Signature"/>
                   </xsd:sequence>
                   <xsd:attribute name="versione" type="types:Versione_Type" use="required" fixed="1.0"/>
                   <xsd:attribute name="FlussoSemplificato" type="xsd:string" use="optional" fixed="Si"/>
        </xsd:complexType>

         <xsd:complexType name="RicevutaScarto_Type">
                 <xsd:sequence>
                          <xsd:element name="IdentificativoSdI" type="types:IdentificativoSdI_Type" />
                          <xsd:element name="NomeFile" type="types:NomeFile_Type" />
```

<!-- PDF page 187 -->

```text
                           <xsd:element name="Hash" type="xsd:string" />
                           <xsd:element name="DataOraRicezione" type="xsd:dateTime" />
                           <xsd:element     name="RiferimentoArchivio"      type="types:RiferimentoArchivio_Type"
minOccurs="0" maxOccurs="1"/>
                           <xsd:element name="ListaErrori" type="types:ListaErrori_Type" />
                           <xsd:element name="MessageId" type="types:MessageId_Type"/>
                           <xsd:element name="PecMessageId" type="types:PecMessageId_Type" minOccurs="0"
maxOccurs="1"/>
                           <xsd:element name="Note" type="xsd:string" minOccurs="0" maxOccurs="1"/>
                           <xsd:element ref="ds:Signature"/>
                 </xsd:sequence>
                 <xsd:attribute name="versione" type="types:Versione_Type" use="required" fixed="1.0"/>
        </xsd:complexType>

        <xsd:complexType name="RicevutaImpossibilitaRecapito_Type">
                 <xsd:sequence>
                           <xsd:element name="IdentificativoSdI" type="types:IdentificativoSdI_Type" />
                           <xsd:element name="NomeFile" type="types:NomeFile_Type" />
                           <xsd:element name="Hash" type="xsd:string" />
                           <xsd:element name="DataOraRicezione" type="xsd:dateTime" />
                           <xsd:element name="DataMessaADisposizione" type="xsd:date" />
                           <xsd:element     name="RiferimentoArchivio"      type="types:RiferimentoArchivio_Type"
minOccurs="0" maxOccurs="1"/>
                           <xsd:element    name="Descrizione"     type="types:Descrizione_Type"      minOccurs="0"
maxOccurs="1"/>
                           <xsd:element name="MessageId" type="types:MessageId_Type"/>
                           <xsd:element name="PecMessageId" type="types:PecMessageId_Type" minOccurs="0"
maxOccurs="1"/>
                           <xsd:element name="Note" type="xsd:string" minOccurs="0" maxOccurs="1"/>
                           <xsd:element ref="ds:Signature"/>
                 </xsd:sequence>
                 <xsd:attribute name="versione" type="types:Versione_Type" use="required" fixed="1.0"/>
        </xsd:complexType>

         <xsd:complexType name="RiferimentoArchivio_Type">
                  <xsd:sequence>
                           <xsd:element name="IdentificativoSdI" type="types:IdentificativoSdI_Type" />
                           <xsd:element name="NomeFile" type="types:NomeFile_Type"/>
                  </xsd:sequence>
         </xsd:complexType>

        <xsd:complexType name="ListaErrori_Type">
        <xsd:sequence>
                          <xsd:element       name="Errore"           type="types:Errore_Type"        minOccurs="1"
maxOccurs="200"/>
                 </xsd:sequence>
 </xsd:complexType>

         <xsd:complexType name="Errore_Type">
         <xsd:sequence>
                           <xsd:element name="Codice" type="types:CodiceErrore_Type" />
                           <xsd:element name="Descrizione" type="types:Descrizione_Type" />
                           <xsd:element name="Suggerimento" type="types:Suggerimento_Type" />
                  </xsd:sequence>
  </xsd:complexType>
```

<!-- PDF page 188 -->

```text
<xsd:simpleType name="CodiceErrore_Type">
         <xsd:restriction base="xsd:string">
                   <xsd:minLength value="1" />
                   <xsd:maxLength value="5" />
         </xsd:restriction>
</xsd:simpleType>

<xsd:simpleType name="Suggerimento_Type">
           <xsd:restriction base="xsd:string">
                     <xsd:maxLength value="2000" />
           </xsd:restriction>
</xsd:simpleType>
<!-- =================================================================== -->
<!-- TIPI PER RICEVENTI
                               -->
<!-- =================================================================== -->
<xsd:complexType name="FileMetadati_Type">
           <xsd:sequence>
                     <xsd:element name="IdentificativoSdI" type="types:IdentificativoSdI_Type" />
                     <xsd:element name="NomeFile" type="types:NomeFile_Type" />
                     <xsd:element name="Hash" type="xsd:string" />
                     <xsd:element name="CodiceDestinatario" type="types:CodiceDestinatario_Type" />
                     <xsd:element name="Formato" type="types:Formato_Type" />
                     <xsd:element name="TentativiInvio" type="xsd:integer" />
                     <xsd:element name="MessageId" type="types:MessageId_Type"/>
                     <xsd:element name="Note" type="xsd:string" minOccurs="0" maxOccurs="1"/>
           </xsd:sequence>
           <xsd:attribute name="versione" type="types:Versione_Type" use="required" fixed="1.0"/>
</xsd:complexType>

<xsd:simpleType name="CodiceDestinatario_Type">
         <xsd:restriction base="xsd:string">
                   <xsd:pattern value="[A-Z0-9]{6,7}"/>
         </xsd:restriction>
</xsd:simpleType>

<xsd:simpleType name="Formato_Type">
           <xsd:restriction base="xsd:string">
                     <xsd:maxLength value="5"/>
           </xsd:restriction>
</xsd:simpleType>
<!-- =================================================================== -->
<!-- TIPI COMUNI
                               -->
<!-- =================================================================== -->
<xsd:simpleType name="IdentificativoSdI_Type">
           <xsd:restriction base="xsd:string">
                     <xsd:minLength value="1" />
                     <xsd:maxLength value="36" />
           </xsd:restriction>
</xsd:simpleType>

<xsd:simpleType name="NomeFile_Type">
         <xsd:restriction base="xsd:string">
```

<!-- PDF page 189 -->

```text
                           <xsd:maxLength value="50" />
                 </xsd:restriction>
        </xsd:simpleType>

       <xsd:complexType name="RiferimentoFattura_Type">
                 <xsd:sequence>
                          <xsd:element name="NumeroFattura" type="types:NumeroFattura_Type" />
                          <xsd:element name="AnnoFattura" type="types:AnnoFattura_Type" />
                          <xsd:element name="PosizioneFattura"     type="xsd:positiveInteger" minOccurs="0"
maxOccurs="1" />
                 </xsd:sequence>
       </xsd:complexType>

        <xsd:simpleType name="NumeroFattura_Type">
                 <xsd:restriction base="xsd:normalizedString">
                           <xsd:pattern value="(\p{IsBasicLatin}{1,20})" />
                 </xsd:restriction>
        </xsd:simpleType>

        <xsd:simpleType name="AnnoFattura_Type">
                 <xsd:restriction base="xsd:nonNegativeInteger">
                 </xsd:restriction>
        </xsd:simpleType>

         <xsd:simpleType name="Descrizione_Type">
                  <xsd:restriction base="xsd:string">
                           <xsd:maxLength value="1000" />
                 </xsd:restriction>
        </xsd:simpleType>

         <xsd:simpleType name="MessageId_Type">
                  <xsd:restriction base="xsd:string">
                           <xsd:minLength value="1" />
                           <xsd:maxLength value="36" />
                 </xsd:restriction>
        </xsd:simpleType>

        <xsd:simpleType name="PecMessageId_Type">
                 <xsd:restriction base="xsd:string" />
        </xsd:simpleType>

        <xsd:simpleType name="Versione_Type">
                 <xsd:restriction base="xsd:string">
                           <xsd:maxLength value="5"/>
                 </xsd:restriction>
        </xsd:simpleType>

         <xsd:complexType name="Destinatario_Type">
                  <xsd:sequence>
                           <xsd:element name="Codice" type="types:CodiceDestinatario_Type" />
                           <xsd:element name="Descrizione" type="xsd:string" minOccurs="0" />
                  </xsd:sequence>
         </xsd:complexType>
</xsd:schema>a
```
