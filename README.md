# HA-Sip-WindTre
Informazioni utili a chi vuole configurare HA-SIP addon su Home Assistant perchè effettui chiamate con il provider italiano WindTre

Configurare l'addon per Home Assistnat HA-SIP perchè si connetta al Provider italiano WindTre.


# Note

Visto quanto ho dovuto sbattere letteralmente la testa per far funzionare questo ADDON con il Provider italiano WindTre, ho deciso di scrivere questa guida per semplificare la vita a chi si vuole cimentare.

La necessità di avere un SIP funzionante è nata quando ho configurato la mia casa domotica con dei sensori di rilevamento acqua. Ijn precedenza l'unico mezzo di notifica che avevo configurato erano le notifiche attraverso Telegram e quelle in-app. Tuttavia, ho avuto dei problemi con la lavatrice e il sistema, funzionando correttamente, mi ha inviato i messaggi giusti, che però non ho visto perchè il telefono era in tasca. Quando ho rilevato il danno il mio pensiero è stato "se mi avesse telefonato magari l'avrei sentito!". Inoltre, problema simile ho pensato si sarebbe potuto verificare con lo scattare di un allarme intrusione. Insomma... volevo che in caso di alcuni "eventi" gravi il sistema non si limitasse a mandarmi un messaggio Telegrma o una banale notifica in-app, ma mi chiamasse con una cavolo di suoneria da "fine del modo" da un numero che potessi aver salvato sulla mia rubrica come "ALLARME CASA" o qualcosa che richiamasse la mia attenzione.

Bhè il Provider WindTre assegna una numeraizione fissa ad ogni contratto internet che stipula che di solito è preconfigurata all'intenro del router che fornisce in fase di installazione.

Da buon smanettone, il mio router, per scelta personale, non è quello originale di WindTre, ma ho preferito acquistarne uno mio le cui caratteristiche fossero da me scelte e non imposte dal provider. Naturalmente questo router non ha la possibilità di configurare un Voip al suo interno, ma bisogna farlo esternamente. 

Per poter effettuare chiamate Voip con la propria numerazione esistono tre strade:
1) utilizzare il router windtre debitamente configurato (limitato);
2) acquistare un apparato Voip Gateway (prevede una spesa dai 20 ai 200 €);
3) trovare un software che faccia al caso nostro (GRATIS E DI LIBERA SCELTA!!).


# SI INIZIA!

Veniamo al dunque, ecco i passaggi da fare:

1. Contattare il Provider WindTre:
    Per prima cosa bisogna contattare il servizio clienti WindTre (159) per farsi dare i parametri di configurazione del proprio account Voip. il centralinista che risponderà non ve li fornirà subito, non li sanno neanche loro, ma verrete ricontattati a distanza di circa 2/3 giorni da un tecnico che vi dirà A VOCE quello che vi serve.
    In realtà quello che vi serve da loro è solo un parametro, gli altri sono standard e sono:
    1. il dominio (o realm): windtre.it
    2. il proxy SIP da utilizzare (uguale per tutti): voip.windtre.it
    3. il vostro username (anche questo parametro già l'avete): 39 seguito senza spazi dall'utenza telefonica di linea fissa che trovate nelle bollette windtre che vi arrivano tutti i mesi (compreso di prefisso regionale) ad esempo 39041555666.  
    4. la password del Voip: questa ve la danno loro e solo loro la conoscono. (in realtà ho trovato una guida su come estrarla dal router originale windtre, ma per favore... ve la comunicano... non serve fare analisi forensi su hardware strano) ma se vi volete cimentare... google vi può aiutare....  

2. scelta del software da utilizzare:
    per la mia configurazione ho scelto [HA-Sip](https://github.com/arnonym/ha-plugins), ne esistono altri, tipo asterisk per home assistnat... ma è troppo importante.... sarebbe utilizzare un carro armato per andare a fare la spesa.... ma potrebbe essere decisiva come scelta se vogliamo creare un centralino telefonico con diversi interni.... non è il mio caso... io voglio ricevere una telefonata se scatta un allarme, non trasformare la mia domotica in un centralino di Nuova Delhi.  

3. configurare il software:
    ed ecco la parte che non troverete da nessuna parte (3 settimane di ricerche e motivo di questa guida).  
    Vi scrivo le configurazioni grezze, così come vanno messe... inserirò alcune indicazioni qua e là tra parentesi, spero di non farvi confusione. Facciamo finta che la mia numerazione sia 041555666 e la password ABC123:  

    PARAMETRI DELLA SEZIONE sip_global:  
    port    5060  
    log_level   5  
    name_server     (lasciate in bianco così prende i DNS di windtre - fondamentali per il funzionamento)  
    cache_dir   /config/audio_cache
    global_options  --stun-server stun3.l.google.com:3478 --tcp disable (lo stun server è importante per ricevere l'audio altrimenti si connette, vi chiama ma non sentirete nulla, mentre il TCP disable è fondamentale per instaurare la chiamata, se non lo mettete il plugin farà una chiamata TCP e il provider "dropperà" la chiamata)  

    PARAMETRI DELLA SEZIONE sip  
    sip enabled  
    registrar_uri   sip:windtre.it  
    id_uri  sip:39041555666@windtre.it  
    realm   *   (non serve scrivere nulla... lo prende in automatico, tuttavia per la precisone andrebbe windtre.it)  
    user_name   39041555666  
    password    ABC123  
    answer_mode  listen  
    settle_time     3  
    incoming_call_file  /config/HA-Sip/sip-1-incoming.yaml (in questo file potrete inserire i parametri nel caso vogliate configurare delle risposte automatiche, del tipo "inserisci una pasword" e utilizzare i toni DTMF o altro...)  
    options --proxy sip:39041555666@voip.windtre.it (se vi da errore 407 è perchè non avete disabilitato il TCP o lo STUN non funziona... provatene un altro - vedi sezione sip_global)  

    PARAMETRI DELLA SEZIONE tts  
    platform    picotts (ho scelto questa per la facilità di installazione e utilizzo... 3 righe su configuration.yaml sotto ve le riporto)  
    engine_id   (se usate picotts lasciate vuoto)  
    language    it-IT (se usate picotts va scritto così, con il trattino, no underscore)  
    voice   (anche in questo caso se usate picotts lasciate vuoto)  
    debug_print     off     (questo restituisce sui log l'elenco dei languages supportati dal vostro tts... se non vi serve disabilitatelo)  
  
    PARAMETRI DELLA SEZIONE webhook  
    id  sip_call_webhook_id (inventatevelo voi.. serve per poter azionare dei meccanismi quando si presenta questo id)  
  
    Questo è tutto... ora il vostro addon dovrebbe connettersi al provider (REGISTER 200) e qualora sollecitato effettuare una chiamata e "leggere" un messaggio da voi scritto.  


    IMPOSTAZIONE DEL tts:  
    per attivare il picotts basta aggiungere al configuration.yaml queste tre righe: (rispettate i rientri!!!!)  
    
```yaml
    tts:
      - platform: picotts
        language: "it-IT"
```

**ATTENZIONE!!!!!!!** il plugin dà la possibilità di poter far rispondere il sistema solo se riceve una chiamata da un determitato numero o di rispondere a chiamate da tutti i numeri tranne che alcuni, una sorta di firewall delle chiamate. 
NON FUNZIONA!!!! ma non perchè il plugin ha errori, ma perchè la WINDTRE oscura i numeri chiamanti. Per poter ricevere il numero in chiaro è necessario attivare col provider l'opzione a pagamento IN VISTA... Lazzaroni!!


Ora facciamo una prova!!!
creiamo un'automazione che ad un certo evento genera una "chiamata" all'addon passandogli determinati parametri, ad esempio:

l'automazione qui sotto fa fare una chimata ad un numero di cellulare se l'entità alarmo rileva un trigger quando è alarmato. il numero di telefono da chiamare può essere un cellulare o un numero fisso di qualsiasi operatore, ma dovrà avere a seguito @windtre.it (anche se è di altro operatore!). Inoltre ho utilizzato diversi webhook a seconda dell'evento che viene generato in modo da poter richiamare, con altre automazioni, specifici eventi. per le chiamate semplici non servono, ma se fate una ricerca vedrete che potrebbero tornare utili sopratutto quando si ricevono chiamate.

ANCHE QUA RISPETTATE I RIENTRI SE FATE UN MERO COPIA-INCOLLA!  
```yaml
alias: ha-plugin-sip
description: ""
triggers:
  - trigger: state
    entity_id:
      - alarm_control_panel.alarmo
    from:
      - armed_away
    to:
      - triggered
conditions: []
actions:
  - action: hassio.addon_stdin
    metadata: {}
    data:
      addon: c7744bff_ha-sip
      input:
        command: dial
        number: sip:+393472221113@windtre.it
        webhook_to_call:
          ring_timeout: ring_timeout_webhook_id
          call_established: call_estabilished_webhook_id
          entered_menu: entered_menu_webhook_id
          timeout: timeout_webhook_id
          dtmf_digit: dmtf_webhook_id
          call_disconnected: call_disconnected_webhook_id
          playback_done: playback_webhook_id
        ring_timeout: 15
        sip_account: 1
        menu:
          message: Allarme! Intrusi in casa!
mode: single
```

Sul repo github di arnonym troverete diverse valide funzionalità... tutte da provare e adattare alle vostre esigenze!
Tutto qua... o meglio... dopo tre settimane di lavoro e nessun docuemnto valido e specifico trovato, tutto qua! 

Buon divertimento! 

