<p align="center"><img src="logo.svg" width="128" alt="Logo di PrendiNota"></p>

# PrendiNota

App Android per **registrare riunioni, trascriverle sul telefono e salvare le note su un cloud privato** (Nextcloud o WebDAV).

L'audio **non lascia mai il telefono**: la trascrizione avviene sul dispositivo con [whisper.cpp](https://github.com/ggml-org/whisper.cpp), e sul cloud vanno solo la nota in Markdown e le foto ridotte. Niente account, niente server centrale, niente telemetria.

## Installazione

Scarica il file `.apk` dall'[ultima release](https://github.com/SereSol/PrendiNota-releases/releases/latest) e aprilo sul telefono (Android chiede di consentire l'installazione da quella app).

Per ricevere gli aggiornamenti in automatico usa [Obtainium](https://github.com/ImranR98/Obtainium): «Aggiungi app» → incolla `https://github.com/SereSol/PrendiNota-releases`.

Ogni APK è firmato con la stessa chiave. Impronta SHA-256 del certificato:
`45:ed:fc:20:26:6a:ec:d2:31:03:ec:c0:a1:62:db:ab:52:b7:d1:52:9b:1a:40:d0:09:09:b5:e6:9e:81:12:31`

---

## Cosa fa

### Registrazione
- Avvio, pausa, ripresa e stop, con waveform e timer in tempo reale.
- La registrazione continua con lo schermo spento, in background o mentre si scatta una foto (servizio in primo piano).
- Audio Opus 16 kHz mono, salvato solo sul telefono.

### Trascrizione sul telefono
- whisper.cpp integrato nell'app, senza connessione: l'audio non viene inviato a nessun servizio.
- Rilevamento automatico della lingua, oppure **«Forza italiano»**.
- Due modelli tra cui scegliere, scaricati una volta dal repository ufficiale di whisper.cpp e verificati con SHA-256:

  | Modello | Spazio | Uso |
  |---|---|---|
  | **Small** (q4_0, quantizzato sul telefono) | 138 MB | Consigliato: errori di trascrizione intorno al 10 % sull'italiano, circa metà del tempo dell'audio su un telefono recente |
  | **Base** (q5_1) | 56 MB | Più leggero e veloce, ma con circa il doppio degli errori. Per telefoni lenti o con poca memoria |

- Coda in background: si può chiudere l'app. La trascrizione riprende da dove era arrivata anche dopo un riavvio del telefono.
- Notifica con percentuale e minuti elaborati; si può annullare e riprendere.
- Al primo avvio una procedura guidata in 3 passi (modello → download → cloud) misura la velocità del telefono e consiglia il modello adatto. Si può saltare e richiamare dalle impostazioni.

### Note in Markdown
Ogni registrazione produce un `note.md` con intestazione YAML e testo a paragrafi con timestamp:

```markdown
---
id: "4c10b692-…"
title: "Riunione di progetto"
date: "2026-11-04T10:30:00"
duration: "00:42:17"
duration_ms: 2537000
language: "it"
tags: []
audio_file: "4c10b692-….opus"
photos:
  - timestamp: "00:03:12"
    file: "foto_00-03-12.jpg"
---

[00:00:00] Buongiorno a tutti, partiamo dal primo punto…
```

Le foto sono elencate nell'intestazione, non inserite nel testo.

### Foto
- Scatto con la fotocamera durante la registrazione, oppure aggiunta dalla galleria anche dopo. Ogni foto è legata al momento della registrazione in cui è stata fatta.
- L'originale resta sul telefono. Sul cloud e nelle condivisioni va solo una **copia ridotta** (lato lungo 2048 px, JPEG), **senza coordinate GPS né dati del telefono**.

### Cloud: Nextcloud e WebDAV
- **Nextcloud**: accesso dal browser con il proprio login, anche con SSO aziendale. Nextcloud crea una password dedicata a PrendiNota, che si può revocare in qualsiasi momento. L'app non vede mai la password dell'utente.
- **WebDAV generico**: indirizzo, utente e password.
- Più profili, ognuno con un nome a scelta (es. «lavoro», «famiglia»), attivabili e disattivabili; uno è il predefinito.
- Al termine della trascrizione l'app crea sul cloud una cartella `data_titolo` e carica la nota e le foto ridotte.
- Le foto aggiunte dopo vengono caricate da sole, e nella nota remota si aggiorna solo l'elenco delle foto.
- **Conflitti**: se la nota è stata modificata sul cloud, l'app non la sovrascrive. Avvisa e lascia scegliere se tenere la versione del telefono o scaricare quella del cloud. In quel caso le modifiche fatte sul cloud restano, e l'elenco delle foto viene aggiornato.
- **Cambia cloud**: dalla scheda di una registrazione si può caricarla su un altro cloud, per esempio se il precedente non funziona più.
- Credenziali cifrate con l'Android Keystore e mai incluse nei backup. Solo indirizzi `https://`. Le password restano legate al server per cui sono state inserite.

### Gestione delle registrazioni
- Elenco cronologico con stato di trascrizione e di caricamento.
- Dettaglio con player (si salta al punto toccando un paragrafo), nota, griglia delle foto e anteprima di `note.md`.
- Rinomina, duplica, elimina. L'eliminazione cancella anche foto e copie condivise.
- **Condivisione** con le app Android (email, chat, …) di `note.md` e delle foto ridotte, mai degli originali.

---

## Privacy in breve

| Dato | Dove va |
|---|---|
| Audio | Solo sul telefono, mai caricato |
| Trascrizione | Fatta sul telefono, poi sul tuo cloud come `note.md` |
| Foto originali | Solo sul telefono |
| Foto ridotte (senza GPS) | Sul tuo cloud e nelle condivisioni |
| Credenziali del cloud | Cifrate nel Keystore del telefono, escluse dai backup |
| Telemetria, account, server di terze parti | Nessuno |

---

## Requisiti

- Android 10 (API 29) o successivo, ARM64 consigliato (supportati anche armeabi-v7a e x86_64 per l'emulatore).
- Per Small servono circa 700 MB liberi durante l'installazione (download del modello completo e ottimizzazione sul telefono) e 138 MB dopo.
- Distribuzione come **APK firmato**, non tramite Play Store.
