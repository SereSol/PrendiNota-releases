# PrendiNota

App Android per **registrare riunioni, trascriverle sul telefono e salvare le note su un cloud privato** (Nextcloud o WebDAV).

L'audio **non lascia mai il telefono**: la trascrizione avviene sul dispositivo con [whisper.cpp](https://github.com/ggml-org/whisper.cpp), e sul cloud vanno solo la nota in Markdown e le foto ridotte. Niente account, niente server centrale, niente telemetria.

## Installazione

Scarica `PrendiNota-*.apk` dall'[ultima release](https://github.com/SereSol/PrendiNota-releases/releases/latest) e aprilo sul telefono (Android chiede di consentire l'installazione da quella app).

Per ricevere gli aggiornamenti in automatico usa [Obtainium](https://github.com/ImranR98/Obtainium): «Aggiungi app» → incolla `https://github.com/SereSol/PrendiNota-releases`.

Ogni APK è firmato con la stessa chiave. Impronta SHA-256 del certificato:
`45:ed:fc:20:26:6a:ec:d2:31:03:ec:c0:a1:62:db:ab:52:b7:d1:52:9b:1a:40:d0:09:09:b5:e6:9e:81:12:31`

> Questo repository contiene solo gli APK. Il codice sorgente è in un repository privato: i link alla documentazione qui sotto funzionano solo per chi vi ha accesso.

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

---

## Compilare

Servono JDK 21 e Android SDK (compileSdk 37). whisper.cpp e opus sono sottomoduli git:

```bash
git clone --recurse-submodules https://github.com/SereSol/PrendiNota.git
cd PrendiNota
./gradlew assembleDebug
```

Le dipendenze sono controllate con la [verifica di Gradle](https://docs.gradle.org/current/userguide/dependency_verification.html) in modalità stretta (`gradle/verification-metadata.xml`).

### Build di release
Per firmare, crea `keystore.properties` nella cartella principale (è escluso da git):

```properties
storeFile=percorso/della/chiave.jks
storePassword=…
keyAlias=…
keyPassword=…
```

Poi:

```bash
./gradlew assembleRelease
```

Senza `keystore.properties` l'APK di release esce **non firmato**, così non si può distribuire per sbaglio. Per una prova locale con la chiave di debug serve chiederlo esplicitamente con `-PdebugSignedRelease`: la versione diventa `…-debugkey` e quell'APK non va distribuito.

### Test

```bash
./gradlew testDebugUnitTest :core:domain:test lintDebug
./gradlew connectedDebugAndroidTest   # con un dispositivo collegato
```

---

## Struttura del progetto

| Modulo | Contenuto |
|---|---|
| `app` | Activity, navigazione, avvio |
| `core:domain` | Modelli, regole di sincronizzazione, formato della nota (JVM puro, testabile) |
| `core:data` | Room, repository, cifratura delle credenziali |
| `core:audio` | Encoder e decoder Opus via JNI, file Ogg |
| `core:ui` | Tema «Carta & Inchiostro», componenti, icone |
| `feature:recordings` | Lista, registrazione, dettaglio, foto, condivisione |
| `feature:transcription` | whisper.cpp via JNI, coda di trascrizione, modelli |
| `feature:cloud` | Client WebDAV su OkHttp, login Nextcloud, sincronizzazione |
| `feature:settings` | Impostazioni, profili cloud, procedura guidata |

Stack: Kotlin, Jetpack Compose (Material 3), Hilt, Room, Coroutines/Flow, WorkManager, OkHttp, whisper.cpp e libopus via JNI.

## Documentazione

- [PRD.md](https://github.com/SereSol/PrendiNota/blob/main/PRD.md): requisiti di prodotto
- [PLAN.md](https://github.com/SereSol/PrendiNota/blob/main/PLAN.md): piano e stato di avanzamento
- [STACK.md](https://github.com/SereSol/PrendiNota/blob/main/STACK.md): scelte tecniche e verifiche
- [design.md](https://github.com/SereSol/PrendiNota/blob/main/design.md): linee guida visive
- [TODO.md](https://github.com/SereSol/PrendiNota/blob/main/TODO.md): attività aperte
- [bench/README.md](https://github.com/SereSol/PrendiNota/blob/main/bench/README.md): benchmark di accuratezza e velocità della trascrizione
