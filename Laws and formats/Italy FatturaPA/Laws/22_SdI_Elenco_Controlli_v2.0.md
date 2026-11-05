# Elenco dei controlli effettuati sul file fattura del Sistema di Interscambio — versione 2.0 (31/01/2025)

**Publication:** fatturapa.gov.it, "Documentazione Sistema d'Interscambio": listed with v1.8.4 (valid from 2026-05-15) and with v1.8.3 (valid until 2026-05-14)  
**Source:** https://www.fatturapa.gov.it/export/documenti/fatturapa/v1.4/Elenco-Controlli-versione-2.0.pdf  
**Page:** https://www.fatturapa.gov.it/it/norme-e-regole/DocumentazioneSDI/  
**Retrieved:** 2026-10-29  
**Text:** Text layer of the published PDF (46 pages) via `pdftotext -layout`, one fenced block per page; running header/footer (title + version, "Pag. N di 46", date) removed.  
**Note:** SdI rejection checks with error codes (00001 … 00477) and criteria; v2.0 changed the criteria of 00471, 00473, 00475. Usable as acceptance criteria: a rejected file = invoice not issued (Provv. 433608/2022 point 2.4).

<!-- PDF page 1 -->

```text
ELENCO DEI CONTROLLI EFFETTUATI SUL FILE FATTURA
            DEL SISTEMA DI INTERSCAMBIO
                   Versione 2.0
```

<!-- PDF page 2 -->

```text
STATO DEL DOCUMENTO

revisione   data                      Note

    1.6       30 luglio 2019          Aggiunti controlli con codice 00320, 00321, 00322, 00323, 00324

    1.7      01 ottobre 2020          Aggiunti controlli con codice 00443, 00444, 00445, 00471, 00472, 00473, 00474.

                                      Modificato criterio controllo con codice 00404, 00409

    1.8      01 ottobre 2022          Aggiunti controlli con codici 00475 e 00476

    1.9     12 dicembre 2023          Aggiunto controllo con codice 00477

    2.0     31 gennaio 2025           Modificato criterio controllo con codice 00471, 00473, 00475
```

<!-- PDF page 3 -->

```text
VERIFICHE EFFETTUATE SUL FILE FATTURA

NOMENCLATURA ED UNICITÀ DEL FILE TRASMESSO

La verifica viene eseguita al fine di intercettare ed impedire l’invio di un file già trasmesso; attraverso un controllo sulla nomenclatura del file ricevuto,
il Sistema di Interscambio verifica che il nome file sia conforme con quanto riportato nelle Specifiche tecniche relative al Sistema di Interscambio e
che non sia stato già inviato un file con lo stesso nome; in caso di esito negativo del controllo (nome file non conforme o nome file già presente nel
Sistema di Interscambio) il file viene rifiutato con le seguenti motivazioni:

    -   Codice 00001           Nome file non valido

    -   Codice 00002           Nome file duplicato

DIMENSIONI DEL FILE

La verifica è effettuata al fine di garantire che il file ricevuto non ecceda le dimensioni ammesse per il rispettivo canale di trasmissione.

    -   Codice 00003           Le dimensioni del file superano quelle ammesse

VERIFICA DI INTEGRITÀ DEL DOCUMENTO

La verifica viene effettuata al fine di garantire che il documento ricevuto non abbia subito modifiche successivamente all’apposizione della firma;
attraverso un controllo sulla firma apposta sull’oggetto trasmesso, il Sistema di Interscambio verifica l’integrità dell’oggetto stesso; se il documento
ricevuto non corrisponde al documento sul quale è stata apposta la firma, il documento viene rifiutato con la seguente motivazione:

    -   Codice 00102           La firma elettronica apposta al file non risulta valida

VERIFICA DI AUTENTICITÀ DEL CERTIFICATO DI FIRMA

La verifica viene effettuata al fine di garantire la validità del certificato di firma utilizzato per apporre la firma elettronica qualificata al documento;
sulla base delle informazioni messe a disposizione dalle “Certification Authorities”, il Sistema di Interscambio verifica la validità del certificato di firma,
che non deve risultare scaduto, revocato o sospeso; in caso di certificato di firma non valido, il documento viene rifiutato con le seguenti motivazioni:
```

<!-- PDF page 4 -->

```text
   -   Codice 00100          Certificato di firma scaduto

   -   Codice 00101          Certificato di firma revocato

   -   Codice 00104          La CA (Certification Authority) che ha emesso il certificato di firma non risulta nell’elenco delle CA affidabili

   -   Codice 00107          Il certificato di firma non è valido

VERIFICA DEL FILE E DEL FORMATO

La verifica viene effettuata per garantire che il contenuto del documento sia rappresentato secondo le regole definite nelle specifiche tecniche; se
risulta una non corretta aderenza alle regole, il documento viene rifiutato con le seguenti motivazioni:

   -   Codice 00103          Alla firma elettronica apposta al file manca il riferimento temporale

   -   Codice 00105          Il riferimento temporale associato alla firma elettronica apposta al file è successivo alla data di ricezione del file

   -   Codice 00106          File / archivio vuoto o corrotto

   -   Codice 00200          File non conforme al formato (nella descrizione del messaggio è riportata l’indicazione puntuale della non conformità)

   -   Codice 00201          Non è possibile procedere con ulteriori controlli, perché gli errori di formato presenti nel file superano il massimo
       previsto (50)

VERIFICHE SUL CONTENUTO

La verifica viene effettuata per garantire la coerenza del contenuto degli elementi informativi per come previsto dalle regole tecniche. In caso di
mancato rispetto di dette regole, il documento viene rifiutato con le seguenti motivazioni:

   -   Codice 00400          2.2.1.14 <Natura> non presente a fronte di 2.2.1.12 <AliquotaIVA> pari a zero

   -   Codice 00401          2.2.1.14 <Natura> presente a fronte di 2.2.1.12 <AliquotaIVA> diversa da zero

   -   Codice 00403          2.1.1.3 <Data> successiva alla data di ricezione
```

<!-- PDF page 5 -->

```text
-   Codice 00411   2.1.1.5 <DatiRitenuta> non presente a fronte di almeno un blocco 2.2.1 <DettaglioLinee> con 2.2.1.13 <Ritenuta>
                   uguale a SI

-   Codice 00413   2.1.1.7.7 <Natura> non presente a fronte di 2.1.1.7.5 <AliquotaIVA> pari a zero

-   Codice 00414   2.1.1.7.7 <Natura> presente a fronte di 2.1.1.7.5 <Aliquota IVA> diversa da zero

-   Codice 00415   2.1.1.5 <DatiRitenuta> non presente a fronte di 2.1.1.7.6 <Ritenuta> uguale a SI

-   Codice 00417   1.4.1.1 <IdFiscaleIVA> e 1.4.1.2 <CodiceFiscale> non valorizzati (almeno uno dei due deve essere valorizzato)

-   Codice 00418   2.1.1.3 <Data> antecedente a 2.1.6.3 <Data>

-   Codice 00419   2.2.2 <DatiRiepilogo> non presente in corrispondenza di almeno un valore di 2.1.1.7.5 <AliquotaIVA> o
                   2.2.1.12 <AliquotaIVA>

-   Codice 00420   2.2.2.2 <Natura> con valore di tipo N6 a fronte di 2.2.2.7 <EsigibilitaIVA> uguale a S (scissione pagamenti)

-   Codice 00421   2.2.2.6 <Imposta> non calcolato secondo le regole definite nelle specifiche tecniche

-   Codice 00422   2.2.2.5 <ImponibileImporto> non calcolato secondo le regole definite nelle specifiche tecniche

-   Codice 00423   2.2.1.11 <PrezzoTotale> non calcolato secondo le regole definite nelle specifiche tecniche

-   Codice 00424   2.2.1.12 <AliquotaIVA> o 2.2.2.1< AliquotaIVA> o 2.1.1.7.5 <AliquotaIVA> non indicata in termini percentuali

-   Codice 00425   2.1.1.4 <Numero> non contenente caratteri numerici

-   Codice 00427   1.1.4 <CodiceDestinatario> di 7 caratteri non ammesso a fronte di 1.1.3 <FormatoTrasmissione> con valore FPA12 o
                   1.1.4 <CodiceDestinatario> di 6 caratteri non ammesso a fronte di 1.1.3 <FormatoTrasmissione> con valore FPR12 o
                   FSM10

-   Codice 00428   1.1.3 <FormatoTrasmissione> non coerente con il valore dell’attributo VERSION

-   Codice 00429   2.2.2.2 <Natura> non presente a fronte di 2.2.2.1 <AliquotaIVA> pari a zero
```

<!-- PDF page 6 -->

```text
-   Codice 00430   2.2.2.2 <Natura> presente a fronte di 2.2.2.1 <Aliquota IVA> diversa da zero

-   Codice 00437   2.1.1.8.2 <Percentuale> e 2.1.1.8.3 <Importo> non presenti a fronte di 2.1.1.8.1 <Tipo> valorizzato

-   Codice 00438   2.2.1.10.2 <Percentuale> e 2.2.1.10.3 <Importo> non presenti a fronte di 2.2.1.10.1 <Tipo> valorizzato

-   Codice 00443   Non c’è corrispondenza tra i valori indicati nell’elemento 2.2.1.12 <AliquotaIVA> o 2.1.1.7.5 <AliquotaIVA> e
                   quelli dell’elemento 2.2.2.1 <ALiquotaIVA>

-   Codice 00444   Non c’è corrispondenza tra i valori indicati nell’elemento 2.2.1.14 <Natura> o 2.1.1.7.7 <Natura> e quelli
                   dell’elemento 2.2.2.2 <Natura>

-   Codice 00445   (controllo in vigore dal primo gennaio 2021) - Non è più ammesso il valore generico N2, N3 o N6 come codice natura
                   dell’operazione

-   Codice 00471   Per il valore indicato nell’elemento 2.1.1.1 <TipoDocumento> il cedente/prestatore non può essere uguale al
                   cessionario/committente

-   Codice 00472   Per il valore indicato nell’elemento 2.1.1.1 <TipoDocumento> il cedente/prestatore deve essere uguale al
                   cessionario/committente

-   Codice 00473   Per il valore indicato nell’elemento 2.1.1.1 <TipoDocumento> il valore nell’elemento 1.2.1.1.1 <IdPaese>
                   non è ammesso

-   Codice 00474   Per il valore indicato nell’elemento 2.1.1.1 <TipoDocumento> non sono ammesse linee di dettaglio con l’elemento
                   2.2.1.12 <AliquotaIVA> contenente valore zero

-   Codice 00475   Per il valore indicato nell’elemento 2.1.1.1 <TipoDocumento> deve essere presente l’elemento 1.4.1.1 <IdFiscaleIVA>
                   del cessionario/committente

-   Codice 00476   Gli elementi 1.2.1.1.1 <IdPaese> e 1.4.1.1.1 <IdPaese> non possono essere entrambi valorizzati con codice diverso
                   da IT

-   Codice 00477   Fattura recante titolo di non imponibilità ai fini IVA ai sensi dell’art. 8, comma 1, lett. c) DPR 26 ottobre 1972, n. 633,
                   con Dichiarazione d’intento invalidata
```

<!-- PDF page 7 -->

```text
VERIFICA DI VALIDITÀ DEL CONTENUTO DELLA FATTURA

Il Sistema di Interscambio verifica la valorizzazione e validità di alcune informazioni presenti nel documento trasmesso per accertare la presenza
dei dati necessari al corretto inoltro del documento al destinatario e per prevenire situazioni di dati errati e/o non elaborabili; in particolare viene
effettuato un controllo:

   -    sull’elemento <CodiceDestinatario>: si verifica che sia presente e attivo nell’anagrafica di riferimento (IndicePA); il controllo prevede anche
        una verifica sulla data di avvio del servizio di fatturazione elettronica presente nell’anagrafica IPA, data che non deve essere successiva a
        quella in cui viene effettuato il controllo (data di sistema);

   -    sulla presenza, nell’anagrafica IPA, di uno o più uffici di fatturazione elettronica attivi associati al codice fiscale corrispondente all’identificativo
        fiscale del cessionario\committente riportato in fattura, nei casi in cui il formato trasmissione identifichi una fattura destinata a soggetto privato
        (FPR12);

   -    sulla presenza, nell’anagrafica IPA, di uno ed un solo ufficio di fatturazione elettronica attivo (diverso da quello Centrale previsto dalle
        specifiche operative relative all’allegato D al DM 3 aprile 2013, n. 55) associato al codice fiscale corrispondente all’identificativo fiscale del
        cessionario/committente riportato in fattura, nei casi in cui il codice identificativo del destinatario sia valorizzato con il codice di fatturazione
        elettronica Centrale in ottemperanza alle disposizioni riportate nella circolare interpretativa del MEF n.1 del 31 marzo 2014;

   -    sulla validità dei codici fiscali e delle partite IVA, relative ai soggetti trasmittente, cedente/prestatore, rappresentante fiscale,
        cessionario/committente, attraverso una verifica di presenza nell’anagrafe tributaria; il controllo non è effettuato per gli identificativi fiscali
        assegnati da autorità estere;

   -    sulla coerenza tra partita IVA e codice fiscale del cedente/prestatore o del cessionario/committente quando entrambi valorizzati e/o nei casi
        di gruppi IVA (per fatture destinate a PA il controllo sulla coerenza in caso di gruppi IVA vale solo per il cedente/prestatore).

Se anche uno solo di questi controlli non viene superato, il documento viene rifiutato con le seguenti motivazioni:

    -   Codice 00300           1.1.1.2 <IdCodice> non valido

    -   Codice 00301           1.2.1.1.2 <IdCodice> non valido

    -   Codice 00302           1.2.1.2 <CodiceFiscale> non valido
```

<!-- PDF page 8 -->

```text
    -   Codice 00303           1.3.1.1.2 <IdCodice> o 1.4.4.1.2 <IdCodice> non valido

    -   Codice 00304           1.3.1.2 <CodiceFiscale> non valido

    -   Codice 00305           1.4.1.1.2 <IdCodice> non valido

    -   Codice 00306           1.4.1.2 <CodiceFiscale> non valido

    -   Codice 00311           1.1.4 <CodiceDestinatario> non valido

    -   Codice 00312           1.1.4 <CodiceDestinatario> non attivo

    -   Codice 00320           1.2.1.1 <IdFiscaleIVA> e 1.2.1.2 <CodiceFiscale> non coerenti

    -   Codice 00321           1.2.1.2 <CodiceFiscale> di soggetto non partecipante al gruppo IVA

    -   Codice 00322           1.2.1.2 <CodiceFiscale> non presente a fronte di 1.2.1.1 <IdFiscaleIVA> di gruppo IVA

    -   Codice 00323           1.2.1.1.2 <IdCodice> corrispondente a una partita IVA cessata da oltre 5 anni

    -   Codice 00324           1.4.1.1 <IdFiscaleIVA> e 1.4.1.2 <CodiceFiscale> non coerenti

    -   Codice 00398           Codice Ufficio presente ed univocamente identificabile nell’anagrafica IPA di riferimento, in presenza di
                               1.1.4 <CodiceDestinatario> valorizzato con codice ufficio “Centrale”

    -   Codice 00399           CessionarioCommittente presente nell’anagrafica IPA di riferimento in presenza di
                               1.1.3 <FormatoTrasmissione> valorizzato a “FPR12”

VERIFICA DI UNICITÀ DELLA FATTURA

La verifica viene eseguita al fine di intercettare ed impedire l’inoltro di una fattura già trasmessa ed elaborata; in quest’ottica, qualora i dati
contenuti all’interno della fattura e relativi a:

   -    identificativo cedente/prestatore;
```

<!-- PDF page 9 -->

```text
   -   anno della data fattura;

   -   numero fattura;

coincidono con quelli di una fattura precedentemente trasmessa e per la quale non è stata inviata al soggetto trasmittente una notifica di scarto
oppure una notifica di rifiuto da parte del destinatario, il documento viene rifiutato con le seguenti motivazioni:

   -   Codice 00404            Fattura duplicata

   -   Codice 00409            Fattura duplicata nel lotto

Nel caso in cui il documento trasmesso sia una nota di credito (<TipoDocumento> = TD04), la verifica tiene conto anche della tipologia di documento;
pertanto è ammessa la presenza di due documenti aventi stesso cedente/prestatore, stesso anno e stesso numero solo qualora uno dei due sia di
tipo TD04.
Nel caso di fatture emesse secondo modalità e termini stabiliti con decreto ministeriale ai sensi dell'articolo 73 del DPR 633/72 e per le quali è stato
valorizzato a “SI” l’elemento informativo <Art73> (situazione che consente l'emissione nello stesso anno di più documenti aventi stesso numero), la
verifica di unicità viene effettuata secondo le regole precedenti ma tenendo conto della data completa e non solo dell’anno.

Tutti i controlli descritti in questo paragrafo, qualora il file inviato al Sistema di Interscambio è relativo ad un documento lotto di fatture (ex art. 1,
comma 3, DLGS 20 febbraio 2004 n. 52), comportano l’accettazione o il rifiuto del file nella sua totalità. Diversamente, se al Sistema di Interscambio
viene inviato un file in formato compresso i controlli, con conseguente accettazione o scarto, riguardano ogni singolo file fattura presente al suo
interno.

Si precisa inoltre che gli stessi controlli sono da riferire all’identificativo del cessionario/committente (e non del cedente/prestatore) nel caso in cui in
fattura risulti valorizzato con CC il campo SoggettoEmittente.
```

<!-- PDF page 10 -->

```text
Controlli AliquotaIVA/Natura per singola linea di dettaglio
 codice errore 00400:      2.2.1.14 <Natura> non presente a fronte di 2.2.1.12 <AliquotaIVA> pari a zero
 codice errore 00401:      2.2.1.14 <Natura> presente a fronte di 2.2.1.12 <AliquotaIVA> diversa da zero

AliquotaIVA   = “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee/AliquotaIVA”
Natura        = “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee/Natura”

      IF (AliquotaIVA == 0.00)
              IF (non esiste Natura)
                      E R R O R E (00400)
              END-IF
      ELSE
              IF (esiste Natura)
                      E R R O R E (00401)
              END-IF
      END-IF
```

<!-- PDF page 11 -->

```text
Controllo coerenza data fattura
 codice errore 00403:      2.1.1.3 <Data> successiva alla data di ricezione

DataFattura          = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/Data”
DataRicezione        = data in cui il file è ricevuto dal Sistema di Interscambio

       IF (DataFattura > DataRicezione)
              E R R O R E (00403)
       END-IF
```

<!-- PDF page 12 -->

```text
Controllo fattura duplicata
 codice errore 00404:       fattura duplicata

TipoDocumento = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/TipoDocumento”
PIVAcedente    = “/FatturaElettronica/FatturaElettronicaHeader/CedentePrestatore/DatiAnagrafici/Anagrafica/IdFiscaleIVA(IdPaese+IdCodice)”
PIVAcessionario = “/FatturaElettronica/FatturaElettronicaHeader/CessionarioCommittente/DatiAnagrafici/IdFiscaleIVA(IdPaese+IdCodice)”
NumeroFattura = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/Numero”
DataFattura    = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/Data”
AnnoFattura    = Year-From-Date (DataFattura)
Articolo73     = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/Art73”
Emittente      = “/FatturaElettronica/FatturaElettronicaHeader/SoggettoEmittente”

Per ogni fattura F all’interno di file NON SCARTATI dal SdI e con esito diverso da “RIFIUTO”
      IF (Emittente non presente OR Emittente != “CC”)
               IF (Articolo73 != “SI” )
                        IF ( PIVAcedente == PIVAcedente di F && NumeroFattura == NumeroFattura di F      &&
                             AnnoFattura == AnnoFattura di F && TipoDocumento != “TD04” &&
                             TipoDocumento di F != “TD04” )
                                E R R O R E (00404)
                         ELSE-IF (TipoDocumento == “TD04” OR TipoDocumento di F == “TD04” )
                                IF (TipoDocumento == TipoDocumento di F && PIVAcedente == PIVAcedente di F &&
                                     NumeroFattura == NumeroFattura di F && AnnoFattura == AnnoFattura di F )
                                         E R R O R E (00404)
                                END-IF
                        END-IF

       (segue)
```

<!-- PDF page 13 -->

```text
          ELSE
              IF ( PIVAcedente == PIVAcedente di F     &&     NumeroFattura == NumeroFattura di F &&
                  DataFattura == DataFattura di F      && TipoDocumento != “TD04”        &&
                  TipoDocumento di F != “TD04” )
                      E R R O R E (00404)
              ELSE-IF (TipoDocumento == “TD04” OR TipoDocumento di F == “TD04” )
                      IF (TipoDocumento == TipoDocumento di F && PIVAcedente == PIVAcedente di F      &&
                          NumeroFattura == NumeroFattura di F && DataFattura == DataFattura di F )
                              E R R O R E (00404)
                      END-IF
              END-IF
      END-IF
ELSE IF (Emittente == “CC”)
      IF (Articolo73 != “SI” )
              IF ( PIVAcessionario == PIVAcedente di F && NumeroFattura == NumeroFattura di F      &&
                   AnnoFattura == AnnoFattura di F && TipoDocumento != “TD04” &&
                   TipoDocumento di F != “TD04” )
                      E R R O R E (00404)
              ELSE-IF (TipoDocumento == “TD04” OR TipoDocumento di F == “TD04” )
                      IF (TipoDocumento == TipoDocumento di F && PIVAcessionario == PIVAcedente di F &&
                          NumeroFattura == NumeroFattura di F && AnnoFattura == AnnoFattura di F )
                               E R R O R E (00404)
                      END-IF
              END-IF

(segue)
```

<!-- PDF page 14 -->

```text
     ELSE
            IF ( PIVAcessionario == PIVAcedente di F       && NumeroFattura == NumeroFattura di F                                    &&
                DataFattura == DataFattura di F && TipoDocumento != “TD04”           &&
                TipoDocumento di F != “TD04” )
                    E R R O R E (00404)
            ELSE-IF (TipoDocumento == “TD04” OR TipoDocumento di F == “TD04” )
                    IF (TipoDocumento == TipoDocumento di F && PIVAcessionario == PIVAcedente di F                                   &&
                        NumeroFattura == NumeroFattura di F && DataFattura == DataFattura di F )
                            E R R O R E (00404)
                    END-IF
            END-IF
END-IF
```

<!-- PDF page 15 -->

```text
Controllo fattura duplicata nel lotto
 codice errore 00409:       fattura duplicata nel lotto

TipoDocumento         = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/TipoDocumento”
NumeroFattura         = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/Numero”
DataFattura           = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/Data”
AnnoFattura           = Year-From-Date (DataFattura)
Articolo73            = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/Art73”

       Per ogni fattura F all’interno del lotto
             IF ( Articolo73 di F != “SI” )
                      Per ogni fattura Fn diversa da F all’interno del lotto
                             IF ( NumeroFattura di Fn == NumeroFattura di F      &&
                                  AnnoFattura di Fn == AnnoFattura di F          &&
                                  TipoDocumento di Fn != “TD04”                  &&
                                  TipoDocumento di F != “TD04” )
                                     E R R O R E (00409)
                             ELSE-IF (TipoDocumento di Fn == “TD04” OR TipoDocumento di F == “TD04” )
                                     IF (TipoDocumento di Fn == TipoDocumento di F  &&
                                         NumeroFattura di Fn == NumeroFattura di F  &&
                                         AnnoFattura di Fn == AnnoFattura di F )
                                              E R R O R E (00409)
                                     END-IF
                             END-IF

       (segue)
```

<!-- PDF page 16 -->

```text
ELSE
      Per ogni fattura Fn diversa da F all’interno del lotto
            IF ( NumeroFattura di Fn == NumeroFattura di F         &&
                 DataFattura di Fn == DataFattura di F             &&
                 TipoDocumento di Fn != “TD04”                     &&
                 TipoDocumento di F != “TD04” )
                       E R R O R E (00409)
            ELSE-IF (TipoDocumento di Fn == “TD04” OR TipoDocumento di F == “TD04” )
                     IF (TipoDocumento di Fn == TipoDocumento di F    &&
                         NumeroFattura di Fn == NumeroFattura di F    &&
                         DataFattura di Fn == DataFattura di F )
                             E R R O R E (00409)
                     END-IF
            END-IF
END-IF
```

<!-- PDF page 17 -->

```text
Controllo Ritenuta su riga di dettaglio
 codice errore 00411:       2.1.1.5 <DatiRitenuta> non presente a fronte di almeno un blocco 2.2.1 <DettaglioLinee> con 2.2.1.13 <Ritenuta>
                            uguale a SI

BloccoDettaglioLinee        = “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee”
Ritenuta                    = BloccoDettaglioLinee + “Ritenuta”
BloccoDatiRitenuta          = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/DatiRitenuta”
DatiRitenutaNecessari       = false

      Per ogni occorrenza di BloccoDettaglioLinee
            IF ( Ritenuta == “SI” )
                    DatiRitenutaNecessari = true
            END-IF

      IF ( DatiRitenutaNecessari == true )
              IF ( non esiste BloccoDatiRitenuta )
                      E R R O R E (00411)
              END-IF
      END-IF
```

<!-- PDF page 18 -->

```text
Controlli AliquotaIVA/Natura per singola linea di Cassa Previdenziale
 codice errore 00413:      2.1.1.7.7 <Natura> non presente a fronte di 2.1.1.7.5 <AliquotaIVA> pari a zero
 codice errore 00414:      2.1.1.7.7 <Natura> presente a fronte di 2.1.1.7.5 <Aliquota IVA> diversa da zero

BloccoCassaPrev      = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/DatiCassaPrevidenziale”
AliquotaIVA          = BloccoCassaPrev + “AliquotaIVA”
Natura               = BloccoCassaPrev + “Natura”

       IF (AliquotaIVA == 0.00)
               IF (non esiste Natura)
                       E R R O R E (00413)
               END-IF
       ELSE
               IF (esiste Natura)
                       E R R O R E (00414)
               END-IF
       END-IF
```

<!-- PDF page 19 -->

```text
Controllo Ritenuta su dati cassa previdenziale
 codice errore 00415:       2.1.1.5 <DatiRitenuta> non presente a fronte di 2.1.1.7.6 <Ritenuta> uguale a SI

BloccoCassaPrev              = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/DatiCassaPrevidenziale”
Ritenuta                     = BloccoCassaPrev + “Ritenuta”
BloccoDatiRitenuta           = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/DatiRitenuta”
DatiRitenutaNecessari        = false

       Per ogni occorrenza di BloccoCassaPrev
             IF ( Ritenuta == “SI” )
                     DatiRitenutaNecessari = true
             END-IF

       IF ( DatiRitenutaNecessari == true )
               IF ( non esiste BloccoDatiRitenuta )
                       E R R O R E (00415)
               END-IF
       END-IF
```

<!-- PDF page 20 -->

```text
Controllo presenza identificativo fiscale del Cessionario/Committente
 codice errore 00417:      1.4.1.1 <IdFiscaleIVA> e 1.4.1.2 <CodiceFiscale> non valorizzati (almeno uno dei due deve essere valorizzato)

IdFiscaleIVA        = “/FatturaElettronica/FatturaElettronicaHeader/CessionarioCommittente/DatiAnagrafici/IdFiscaleIVA”
CodiceFiscale       = “/FatturaElettronica/FatturaElettronicaHeader/CessionarioCommittente/DatiAnagrafici/CodiceFiscale”

      IF ( non esiste IdFiscaleIVA && non esiste CodiceFiscale )
                     E R R O R E (00417)
      END-IF
```

<!-- PDF page 21 -->

```text
Controllo coerenza data nota di credito
 codice errore 00418:     2.1.1.3 <Data> antecedente a 2.1.6.3 <Data>

TipoDoc            = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/TipoDocumento”
DataNC             = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/Data”
DataFattColl       = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiFattureCollegate/Data”

      IF ( TipoDoc == “TD04” )
             IF ( DataNC < DataFattColl )
                     E R R O R E (00418)
             END-IF
      END-IF
```

<!-- PDF page 22 -->

```text
Controllo corrispondenza DatiRiepilogo e aliquote IVA
 codice errore 00419:     2.2.2 <DatiRiepilogo> non presente in corrispondenza di almeno un valore di 2.1.1.7.5 <AliquotaIVA> o
                          2.2.1.12 <AliquotaIVA>

BloccoCassaPrev            = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/DatiCassaPrevidenziale”
AliquotaIVACP              = BloccoCassaPrev + “AliquotaIVA”
BloccoDettaglioLinee       = “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee”
AliquotaIVADL              = BloccoDettaglioLinee + “AliquotaIVA”
TotAliquoteIVA             =0
AliquoteIVA                = aliquote IVA trovate nel documento
TotRiepiloghi              = numero di blocchi “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/DatiRiepilogo”

      Per ogni occorrenza di BloccoCassaPrev
            IF ( AliquotaIVACP non presente in AliquoteIVA )
                    scrivi AliquotaIVACP in AliquoteIVA
                    add 1 to TotAliquoteIVA
            END-IF

      Per ogni occorrenza di BloccoDettaglioLinee
            IF ( AliquotaIVADL non presente in AliquoteIVA )
                    scrivi AliquotaIVADL in AliquoteIVA
                    add 1 to TotAliquoteIVA
            END-IF

      IF ( TotRiepiloghi < TotAliquoteIVA )
                     E R R O R E (00419)
      END-IF
```

<!-- PDF page 23 -->

```text
Controllo EsigibilitaIVA e Natura
 codice errore 00420:       2.2.2.2 <Natura> con valore di tipo N6 a fronte di 2.2.2.7 <EsigibilitaIVA> uguale a S
                            (scissione pagamenti)

BloccoDatiRiepilogo = “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/DatiRiepilogo”
Esigibilità         = BloccoDatiRiepilogo + “EsigibilitaIVA”
Natura              = BloccoDatiRiepilogo + “Natura”

Per ogni BloccoDatiRiepilogo

       IF (Esigibilità == ”S”   &&     (Natura ==          ”N6” OR
                                        Natura ==          ”N6.1” OR
                                        Natura ==          ”N6.2” OR
                                        Natura ==          ”N6.3” OR
                                        Natura ==          ”N6.4” OR
                                        Natura ==          ”N6.5” OR
                                        Natura ==          ”N6.6” OR
                                        Natura ==          ”N6.7” OR
                                        Natura ==          ”N6.8” OR
                                        Natura ==          ”N6.9” ))

             E R R O R E (00420)
       END-IF
```

<!-- PDF page 24 -->

```text
Controllo corretto calcolo dell’Imposta
 codice errore 00421:      2.2.2.6 <Imposta> non calcolato secondo le regole definite nelle specifiche tecniche

BloccoDatiRiepilogo = “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/DatiRiepilogo”
Imposta             = BloccoDatiRiepilogo + “Imposta”
AliquotaIVA         = BloccoDatiRiepilogo + “AliquotaIVA”
ImponibileImporto = BloccoDatiRiepilogo + “ImponibileImporto”

Per ogni BloccoDatiRiepilogo

       IF ( Imposta != ( AliquotaIVA * ImponibileImporto ) ∕ 100 )
              E R R O R E (00421)
       END-IF

Nota:
l’operazione ( AliquotaIVA * ImponibileImporto ) ∕ 100 ) può dare un risultato contenente più di due cifre decimali; in questo caso il
valore da riportare nel campo Imposta deve essere arrotondato alla seconda cifra decimale; l’arrotondamento va effettuato per difetto, se la terza
cifra decimale è inferiore a 5, per eccesso se la terza cifra decimale è uguale o maggiore di 5.
Esempio:       se ( AliquotaIVA * ImponibileImporto ) ∕ 100 ) = 1500.2448              il valore del campo Imposta deve essere 1500.24
               se ( AliquotaIVA * ImponibileImporto ) ∕ 100 ) = 1500.2458              il valore del campo Imposta deve essere 1500.25

Tolleranza: 1 centesimo di euro. Se la differenza tra i valori confrontati è inferiore a ±0,01 il controllo si ritiene superato
```

<!-- PDF page 25 -->

```text
Controllo corretto calcolo dell’ImponibileImporto
 codice errore 00422:      2.2.2.5 <ImponibileImporto> non calcolato secondo le regole definite nelle specifiche tecniche

ImponibileImporto      = “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/DatiRiepilogo/ImponibileImporto”
Arrotondamento         = “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/DatiRiepilogo/Arrotondamento”
PrezzoTotale           = “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee/PrezzoTotale”
ImportoContrCassa      = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/DatiCassaPrevidenziale/
                          ImportoContributoCassa”
AliquotaIVA            = “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee/AliquotaIVA” +
                         “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/DatiCassaPrevidenziale/AliquotaIVA”

Per ogni valore distinto di AliquotaIVA

                                                                                                  𝑝
       IF ( ∑𝑛                            𝑚                                                  𝑛
             𝑥=1 𝑰𝒎𝒑𝒐𝒏𝒊𝒃𝒊𝒍𝒆𝑰𝒎𝒑𝒐𝒓𝒕𝒐[𝒙] != ∑𝑦=1 𝑷𝒓𝒆𝒛𝒛𝒐𝑻𝒐𝒕𝒂𝒍𝒆[𝒚] + ∑𝑧=1 𝑰𝒎𝒑𝒐𝒓𝒕𝒐𝑪𝒐𝒏𝒕𝒓𝑪𝒂𝒔𝒔𝒂[𝒛] + ∑𝑥=1 𝑨𝒓𝒓𝒐𝒕𝒐𝒏𝒅𝒂𝒎𝒆𝒏𝒕𝒐[𝒙] )
                       E R R O R E (00422)
       END-IF

Nota:
n = numero di blocchi “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/DatiRiepilogo” contenenti una stessa aliquota IVA
m = numero di blocchi “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee” contenenti una stessa aliquota IVA
p = numero di blocchi “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/DatiCassaPrevidenziale” contenenti una
    stessa aliquota IVA

Tolleranza: 1 euro. Se la differenza tra i valori confrontati è inferiore a ±1 il controllo si ritiene superato
```

<!-- PDF page 26 -->

```text
Controllo corretto calcolo del PrezzoTotale
 codice errore 00423:      2.2.1.11 <PrezzoTotale> non calcolato secondo le regole definite nelle specifiche tecniche

BloccoDettaglioLinee         = “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee”
PrezzoTotale                 = BloccoDettaglioLinee + “PrezzoTotale”
PrezzoUnitario               = BloccoDettaglioLinee + “PrezzoUnitario”
Quantità                     = BloccoDettaglioLinee + “Quantita” (se non presente si considera pari a 1)
ScontoTot                    = sconto totale
MaggiorazioneTot             = maggiorazione totale

Per ogni BloccoDettaglioLinee

          IF ( PrezzoTotale != ( PrezzoUnitario ± MaggiorazioneTot o ScontoTot ) * Quantità )
                  E R R O R E (00423)
          END-IF

Nota:
Si consideri quanto segue nel caso in cui siano presenti più blocchi relativi allo sconto o maggiorazione

          BloccoScontoMagg   = BloccoDettaglioLinee + “ScontoMaggiorazione”
          Tipo               = BloccoScontoMagg + “Tipo”
          Importo            = BloccoScontoMagg + “Importo”
          Percentuale        = BloccoScontoMagg + “Percentuale”

(segue)
```

<!-- PDF page 27 -->

```text
Sia lo sconto che la maggiorazione possono essere rappresentati in fattura come importo o come percentuale.
Se sono rappresentati come importo, il valore totale da considerare è dato dalla somma degli importi

Per ogni BloccoScontoMagg

          IF ( Tipo == “SC” )
                  add Importo to ScontoTot
          ELSE-IF ( Tipo == “MG” )
                  add Importo to MaggiorazioneTot
          END-IF

Se sono rappresentati come percentuale, il calcolo tiene conto dell’effetto ‘a cascata’ ; pertanto il valore totale è dato dalla somma dei valori
di sconto/maggiorazione ottenuti applicando al prezzo unitario la percentuale secondo la regola che segue

                                                 ( PrezzoUnitario * Percentuale ) ∕ 100

ma dove il PrezzoUnitario assume di volta in volta un valore più basso/più alto per effetto dell’applicazione dello sconto/maggiorazione
precedentemente applicato/a.

In caso di presenza contemporanea di Importo e Percentuale, si considera solo il primo ai fini del calcolo.

Tolleranza: 1 centesimo di euro. Se la differenza tra i valori confrontati è inferiore a ±0,01 il controllo si ritiene superato

(segue)
```

<!-- PDF page 28 -->

```text
Di seguito alcuni esempi sulle modalità di calcolo, con particolare riferimento alla presenza di più blocchi di sconto/maggiorazione.

    Caso 1: sono presenti più sconti a cascata, tutti rappresentati come importo

    […..]
    <DettaglioLinee>
        <NumeroLinea>1</NumeroLinea>                                                         Il valore inserito nel campo <PrezzoTotale> risulta corretto in quanto
        <Descrizione>Fornitura bene x</Descrizione>                                          determinato secondo il processo descritto nella Nota sopra e riportato di seguito:
        <Quantita>100.00</Quantita>
        <PrezzoUnitario>500.00</PrezzoUnitario>                                              <PrezzoTotale> = [ <PrezzoUnitario> - ( somma <Importo> ) ] * <Quantita>
        <ScontoMaggiorazione>
                <Tipo>SC</Tipo>                                                                         [ 500.00 - ( 10.05 + 5.25 + 4.70 ) ] * 100.00
                <Importo>10.05</Importo>
        </ScontoMaggiorazione>                                                                          [ 500.00 - 20.00 ] * 100.00
        <ScontoMaggiorazione>
                <Tipo>SC</Tipo>                                                                         480.00 * 100.00 = 48000.00
                <Importo>5.25</Importo>
        </ScontoMaggiorazione>
        <ScontoMaggiorazione>
                <Tipo>SC</Tipo>
                <Importo>4.70</Importo>
        </ScontoMaggiorazione>
        <PrezzoTotale>48000.00</PrezzoTotale>
    […..]

(segue)
```

<!-- PDF page 29 -->

```text
    Caso 2: sono presenti più sconti a cascata, tutti rappresentati come percentuale

    […..]
    <DettaglioLinee>
        <NumeroLinea>1</NumeroLinea>                                                         Il valore inserito nel campo <PrezzoTotale> risulta corretto in quanto
        <Descrizione>Fornitura bene x</Descrizione>                                          determinato secondo il processo descritto nella Nota sopra e riportato di seguito:
        <Quantita>100.00</Quantita>
        <PrezzoUnitario>500.00</PrezzoUnitario>                                              ( <PrezzoUnitario> - <Percentuale> primo sconto ) = Risultato1
        <ScontoMaggiorazione>                                                                ( Risultato1 - <Percentuale> secondo sconto ) = Risultato2
                <Tipo>SC</Tipo>                                                              ( Risultato2 - <Percentuale> terzo sconto ) = Risultato3
                <Percentuale>8.50</Percentuale>                                              <PrezzoTotale> = Risultato3 * <Quantita>
        </ScontoMaggiorazione>
        <ScontoMaggiorazione>                                                                           ( 500.00 * 8.50 ) / 100 = 42.50
                <Tipo>SC</Tipo>                                                                         500.00 - 42.50 = 457.50
                <Percentuale>5.15</Percentuale>
        </ScontoMaggiorazione>                                                                          ( 457.50 * 5.15 ) / 100 = 23.56125
        <ScontoMaggiorazione>                                                                           457.50 - 23.56125 = 433.93875
                <Tipo>SC</Tipo>
                <Percentuale>3.00</Percentuale>                                                         ( 433.93875 * 3.00 ) / 100 = 13.0181625
        </ScontoMaggiorazione>                                                                          433.93875 - 13.0181625 = 420.9205875
        <PrezzoTotale>42092.05875</PrezzoTotale>
    […..]                                                                                               420.9205875 * 100.00 = 42092.05875

                                                                                             Per effetto della tolleranza, è ammissibile anche un <PrezzoTotale> con i valori
                                                                                             42092.05 oppure 42092.06

(segue)
```

<!-- PDF page 30 -->

```text
Caso 3: sono presenti più sconti a cascata, alcuni rappresentati come importo, altri come percentuale, altri come entrambi

[…..]
<DettaglioLinee>
    <NumeroLinea>1</NumeroLinea>                                                         Il valore inserito nel campo <PrezzoTotale> risulta corretto in quanto
    <Descrizione>Fornitura bene x</Descrizione>                                          determinato secondo il processo descritto nella Nota sopra e riportato di seguito:
    <Quantita>100.00</Quantita>
    <PrezzoUnitario>500.00</PrezzoUnitario>                                              ( <PrezzoUnitario> - <Percentuale> primo sconto ) = Risultato1
    <ScontoMaggiorazione>                                                                ( Risultato1 - <Importo> secondo sconto ) = Risultato2
            <Tipo>SC</Tipo>                                                              ( Risultato2 - <Importo> terzo sconto ) = Risultato3
            <Percentuale>8.50</Percentuale>                                              ( Risultato3 - <Percentuale> quarto sconto ) = Risultato4
    </ScontoMaggiorazione>                                                               <PrezzoTotale> = Risultato4 * <Quantita>
    <ScontoMaggiorazione>
            <Tipo>SC</Tipo>                                                                         ( 500.00 * 8.50 ) / 100 = 42.50
            <Percentuale>5.15</Percentuale>                                                         500.00 - 42.50 = 457.50
            <Importo>23.56</Importo>
    </ScontoMaggiorazione>                                                                          457.50 - 23.56 = 433.94
    <ScontoMaggiorazione>
            <Tipo>SC</Tipo>                                                                         433.94 - 15.00 = 418.94
            <Importo>15.00</Importo>
    </ScontoMaggiorazione>                                                                          ( 418.94 * 2.00 ) / 100 = 8.3788
    <ScontoMaggiorazione>                                                                           418.94 - 8.3788 = 410.5612
            <Tipo>SC</Tipo>
            <Percentuale>2.00</Percentuale>                                                         410.5612 * 100.00 = 41056.12
    </ScontoMaggiorazione>
    <PrezzoTotale>41056.12</PrezzoTotale>
[…..]
```

<!-- PDF page 31 -->

```text
Controllo valore AliquotaIVA
 codice errore 00424:      2.2.1.12 <AliquotaIVA> o 2.2.2.1< AliquotaIVA> o 2.1.1.7.5 <AliquotaIVA> non indicata in termini percentuali

AliquotaIVACassaP   = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/DatiCassaPrevidenziale/AliquotaIVA”
AliquotaIVALinea    = “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee/AliquotaIVA”
AliquotaIVARiep     = “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/DatiRiepilogo/AliquotaIVA”

      IF ( AliquotaIVACassaP != 0.00 && AliquotaIVACassaP < 1.00 )
              E R R O R E (00424)
      END-IF

      IF ( AliquotaIVALinea != 0.00 && AliquotaIVALinea < 1.00 )
              E R R O R E (00424)
      END-IF

      IF ( AliquotaIVARiep != 0.00 && AliquotaIVARiep < 1.00 )
              E R R O R E (00424)
      END-IF
```

<!-- PDF page 32 -->

```text
Controllo presenza caratteri numerici nell’elemento informativo Numero fattura
 codice errore 00425:     2.1.1.4 <Numero> non contenente caratteri numerici

Numero       = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/Numero”
ValoreNum    = carattere numerico da 0 a 9

      IF ( Numero non contiene ValoreNum )
             E R R O R E (00425)
      END-IF
```

<!-- PDF page 33 -->

```text
Controllo coerenza elemento informativo FormatoTrasmissione e elemento informativo CodiceDestinatario
 codice errore 00427:    1.1.4 <CodiceDestinatario> di 7 caratteri a fronte di 1.1.3 <FormatoTrasmissione> con valore “FPA12” o
                         1.1.4 <CodiceDestinatario> di 6 caratteri a fronte di 1.1.3 <FormatoTrasmissione> con valore “FPR12” o “FSM10”

FormatoTrasm        = “/FatturaElettronica/FatturaElettronicaHeader/DatiTrasmissione/FormatoTrasmissione”
CodDest             = “/FatturaElettronica/FatturaElettronicaHeader/DatiTrasmissione/CodiceDestinatario”

      LEN = lunghezza CodDest

      IF (FormatoTrasm == “FPA12”)
             IF (LEN == 7)
                    E R R O R E (00427)
             END-IF
      ELSE
             IF (LEN == 6)
                    E R R O R E (00427)
             END-IF
      END-IF
```

<!-- PDF page 34 -->

```text
Controllo coerenza attributo versione con elemento informativo FormatoTrasmissione
 codice errore 00428:     1.1.3 <FormatoTrasmissione> non coerente con il valore dell’attributo VERSIONE

versione              = valore che segue a “<p:FatturaElettronica versione=”
FormatoTrasm          = “/FatturaElettronica/FatturaElettronicaHeader/DatiTrasmissione/FormatoTrasmissione”

       IF (FormatoTrasm != versione)
              E R R O R E (00428)
       END-IF

Esempio del posizionamento dell’attributo versione all’interno del file XML

       <?xml version="1.0" encoding="UTF-8"?>
       <p:FatturaElettronica versione="FPR12" ……………….
```

<!-- PDF page 35 -->

```text
Controlli AliquotaIVA/Natura nei dati di riepilogo
 codice errore 00429:      2.2.2.2 <Natura> non presente a fronte di 2.2.2.1 <AliquotaIVA> pari a zero
 codice errore 00430:      2.2.2.2 <Natura> presente a fronte di 2.2.2.1 <AliquotaIVA> diversa da zero

AliquotaIVA   = “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/DatiRiepilogo/AliquotaIVA”
Natura        = “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/DatiRiepilogo/Natura”

      IF (AliquotaIVA == 0.00)
              IF (non esiste Natura)
                      E R R O R E (00429)
              END-IF
      ELSE
              IF (esiste Natura)
                      E R R O R E (00430)
              END-IF
      END-IF
```

<!-- PDF page 36 -->

```text
Controlli Sconto/Maggiorazione
 codice errore 00437:     2.1.1.8.2 <Percentuale> e 2.1.1.8.3 <Importo> non presenti a fronte di 2.1.1.8.1 <Tipo> valorizzato
 codice errore 00438:     2.2.1.10.2 <Percentuale> e 2.2.1.10.3 <Importo> non presenti a fronte di 2.2.1.10.1 <Tipo> valorizzato

ScontoMaggGen       = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/ScontoMaggiorazione”
TipoGen             = ScontoMaggGen + “Tipo”
PercentualeGen      = ScontoMaggGen + “Percentuale”
ImportoGen          = ScontoMaggGen + “Importo”
ScontoMaggRiga      = “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee/ScontoMaggiorazione”
TipoRiga            = ScontoMaggRiga + “Tipo”
PercentualeRiga     = ScontoMaggRiga + “Percentuale”
ImportoRiga         = ScontoMaggRiga + “Importo”

      IF (esiste TipoGen)
              IF (non esiste PercentualeGen && non esiste ImportoGen)
                     E R R O R E (00437)
              END-IF
      END-IF

      IF (esiste TipoRiga)
              IF (non esiste PercentualeRiga && non esiste ImportoRiga)
                     E R R O R E (00438)
              END-IF
      END-IF
```

<!-- PDF page 37 -->

```text
Controllo corrispondenza aliquote IVA e relativi blocchi di riepilogo
 codice errore 00443:     non c’è corrispondenza tra i valori indicati nell’elemento 2.2.1.12 <AliquotaIVA> o 2.1.1.7.5 <AliquotaIVA> e quelli
                          dell’elemento 2.2.2.1 <AliquotaIVA>

BloccoDettaglioLinee        = “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee”
AliquotaIVALinea            = BloccoDettaglioLinee + “AliquotaIVA”
BloccoCassaPrev             = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/DatiCassaPrevidenziale”
AliquotaIVACassaPrev        = BloccoCassaPrev + AliquotaIVA”
BloccoRiepilogo             = “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/ DatiRiepilogo”
AliquotaIVARiepilogo        = BloccoRiepilogo + “AliquotaIVA”

Per ogni BloccoCassaPrev
      IF ( AliquotaIVACassaPrev != da tutte le AliquotaIVARiepilogo )
              E R R O R E (00443)
      END-IF

Per ogni BloccoDettaglioLinee
      IF ( AliquotaIVALinea != da tutte le AliquotaIVARiepilogo )
              E R R O R E (00443)
      END-IF
```

<!-- PDF page 38 -->

```text
Controllo corrispondenza nature e relativi blocchi di riepilogo
 codice errore 00444:     non c’è corrispondenza tra i valori indicati nell’elemento 2.2.1.14 <Natura> o 2.1.1.7.7 <Natura> e quelli dell’elemento
                          2.2.2.2 <Natura>

BloccoDettaglioLinee         = “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee”
NaturaLinea                  = BloccoDettaglioLinee + “Natura”
BloccoCassaPrev              = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/DatiCassaPrevidenziale”
NaturaCassaPrev              = BloccoCassaPrev + Natura”
BloccoRiepilogo              = “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/ DatiRiepilogo”
NaturaRiepilogo              = BloccoRiepilogo + “Natura”

Per ogni BloccoCassaPrev
      IF ( NaturaCassaPrev != da tutte le NaturaRiepilogo )
              E R R O R E (00444)
      END-IF

Per ogni BloccoDettaglioLinee
      IF ( NaturaLinea != da tutte le NaturaRiepilogo )
              E R R O R E (00443)
      END-IF
```

<!-- PDF page 39 -->

```text
Controllo presenza valore natura non più valido (in vigore a partire dal primo gennaio 2021)
 codice errore 00445:      non è più ammesso il valore generico N2, N3 o N6 come codice natura dell’operazione

BloccoDettaglioLinee       = “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee/Natura”
NaturaLinea                = BloccoDettaglioLinee + “Natura”
BloccoCassaPrev            = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/DatiCassaPrevidenziale”
NaturaCassaPrev            = BloccoCassaPrev + Natura”
BloccoRiepilogo            = “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/ DatiRiepilogo”
NaturaRiepilogo            = BloccoRiepilogo + “Natura”

Per ogni BloccoCassaPrev
      IF ( NaturaCassaPrev == N2 OR NaturaCassaPrev == N3 OR NaturaCassaPrev == N6 )
              E R R O R E (00445)
      END-IF

Per ogni BloccoDettaglioLinee
      IF ( NaturaLinea == N2 OR NaturaLinea == N3 OR NaturaLinea == N6 )
              E R R O R E (00445)
      END-IF

Per ogni BloccoRiepilogo
      IF ( NaturaRiepilogo == N2 OR NaturaRiepilogo == N3 OR NaturaRiepilogo == N6 )
              E R R O R E (00445)
      END-IF
```

<!-- PDF page 40 -->

```text
Controllo cedente e cessionario in relazione al tipo documento
 codice errore 00471:     per il valore indicato nell’elemento 2.1.1.1 <TipoDocumento> il cedente/prestatore non può essere uguale al
                          cessionario/committente

TipoDocumento                 = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/TipoDocumento”
IDCedente                     = “/FatturaElettronica/FatturaElettronicaHeader/CedentePrestatore/ DatiAnagrafici/IdFiscaleIVA”
IDCessionario PIVA            = “/FatturaElettronica/FatturaElettronicaHeader/CessionarioCommittente/ DatiAnagrafici/IdFiscaleIVA”
IDCessionarioCF               = “/FatturaElettronica/FatturaElettronicaHeader/CessionarioCommittente/ DatiAnagrafici/CodiceFiscale”

IF ( TipoDocumento == TD16 OR TipoDocumento == TD17 OR TipoDocumento == TD18 OR TipoDocumento == TD19
     OR TipoDocumento == TD20 OR TipoDocumento == TD29 )
        IF ( IDCedente == IDCessionarioIVA )
                E R R O R E (00471)
        END-IF
END-IF

Nei casi in cui in fattura sia presente il solo IDCessionarioCF per il cessionario, si risale all’ IDCessionarioPIVA attraverso interrogazione di
Anagrafe Tributaria.
```

<!-- PDF page 41 -->

```text
Controllo cedente e cessionario in relazione al tipo documento
 codice errore 00472:     per il valore indicato nell’elemento 2.1.1.1 <TipoDocumento> il cedente/prestatore deve essere uguale al
                          cessionario/committente

TipoDocumento                 = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/TipoDocumento”
IDCedente                     = “/FatturaElettronica/FatturaElettronicaHeader/CedentePrestatore/ DatiAnagrafici/IdFiscaleIVA”
IDCessionario PIVA            = “/FatturaElettronica/FatturaElettronicaHeader/CessionarioCommittente/ DatiAnagrafici/IdFiscaleIVA”
IDCessionarioCF               = “/FatturaElettronica/FatturaElettronicaHeader/CessionarioCommittente/ DatiAnagrafici/CodiceFiscale”

IF ( TipoDocumento == TD21 )
        IF ( IDCedente != IDCessionarioIVA )
                E R R O R E (00472)
        END-IF
END-IF

Nei casi in cui in fattura sia presente il solo IDCessionarioCF per il cessionario, si risale all’ IDCessionarioPIVA attraverso interrogazione di
Anagrafe Tributaria.
```

<!-- PDF page 42 -->

```text
Controllo cedente in relazione al tipo documento
 codice errore 00473:       per il valore indicato nell’elemento 2.1.1.1 <TipoDocumento> il valore nell’elemento 1.2.1.1.1 <IdPaese>
 non è ammesso

TipoDocumento               = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/TipoDocumento”
IDPaeseCedente              = “/FatturaElettronica/FatturaElettronicaHeader/CedentePrestatore/ DatiAnagrafici/IdFiscaleIVA/IdPaese”

IF ( TipoDocumento == TD17 OR TipoDocumento == TD18 OR TipoDocumento == TD19 OR TipoDocumento == TD28 )
        IF ( IDPaeseCedente == IT )
                E R R O R E (00473)
        END-IF
END-IF
IF ( TipoDocumento == TD29 )
        IF ( IDPaeseCedente != IT )
                E R R O R E (00473)
        END-IF
```

<!-- PDF page 43 -->

```text
Controllo tipo documento e aliquota IVA
 codice errore 00474:     per il valore indicato nell’elemento 2.1.1.1 <TipoDocumento> non sono ammesse linee di dettaglio con l’elemento
                          2.2.1.12 <AliquotaIVA> contenente valore zero

TipoDocumento              = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/TipoDocumento”
BloccoDettaglioLinee       = “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee”
AliquotaIVA                = BloccoDettaglioLinee + “AliquotaIVA”

IF (TipoDocumento == TD21)
       Per ogni BloccoDettaglioLinee
       IF ( AliquotaIVA == 0.00 )
               E R R O R E (00474)
       END-IF
END-IF
```

<!-- PDF page 44 -->

```text
Controllo tipo documento e partita IVA cessionario/committente
 codice errore 00475:     per il valore indicato nell’elemento 2.1.1.1 <TipoDocumento> deve essere presente l’elemento 1.4.1.1
                          <IdFiscaleIVA> del cessionario/committente

TipoDocumento              = “/FatturaElettronica/FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/TipoDocumento”
IDCessionario PIVA         = “/FatturaElettronica/FatturaElettronicaHeader/CessionarioCommittente/ DatiAnagrafici/IdFiscaleIVA”

IF ( TipoDocumento == TD16 OR TipoDocumento == TD17 OR TipoDocumento == TD18 OR TipoDocumento == TD19
     OR TipoDocumento == TD20 OR TipoDocumento == TD22 OR TipoDocumento == TD23 OR TipoDocumento == TD28 OR
TipoDocumento == TD29)
        IF ( non esiste IDCessionarioPIVA )
                E R R O R E (00475)
        END-IF
END-IF
```

<!-- PDF page 45 -->

```text
Controllo codice Paese cedente/prestatore e cessionario/committente
 codice errore 00476:     gli elementi 1.2.1.1.1 <IdPaese> e 1.4.1.1.1 <IdPaese> non possono essere entrambi valorizzati con codice diverso
                          da IT

IDPaeseCedente             = “/FatturaElettronica/FatturaElettronicaHeader/CedentePrestatore/ DatiAnagrafici/IdFiscaleIVA/IdPaese”
IDPaeseCessionario         = “/FatturaElettronica/FatturaElettronicaHeader/CessionarioCommittente/ DatiAnagrafici/IdFiscaleIVA/IdPaese”

IF ( IDPaeseCedente != IT && IDPaeseCessionario != IT )
              E R R O R E (00476)
END-IF
```

<!-- PDF page 46 -->

```text
Controllo dichiarazioni d’intento
 codice errore 00477:       fattura recante titolo di non imponibilità ai fini IVA ai sensi dell’art. 8, comma 1, lett. C) DPR 26 ottobre 1972, n. 633,
                            con Dichiarazione d’intento invalidata

BloccoDettaglioLinee          = “/FatturaElettronica/FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee”
BloccoAltriDatiGestionali     = BloccoDettaglioLinee + “AltriDatiGestionali”
Natura                        = BloccoDettaglioLinee + “Natura”
RiferimentoTesto              = BloccoAltriDatiGestionali + “RiferimentoTesto”

Per ogni BloccoDettaglioLinee
IF ( Natura == N3.5 )
        Per ogni BloccoAltriDatiGestionali
        IF (esiste RiferimentoTesto )
                IF (RiferimentoTesto è presente in Banca dati Dichiarazioni d’intento non valide)
                        E R R O R E (00477)
                END-IF
        END-IF
END-IF
```
