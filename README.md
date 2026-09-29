# PianoVAM v1.2 - Colab quickstart

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Alessandro1040/pianovam-quickstart/blob/main/pianovam_quickstart.ipynb)

Clicca il badge **Open in Colab** qui sopra per aprirlo direttamente in Colab (repo pubblico,
quindi il badge funziona). In alternativa: scarica il file e usalo con *File → Apri notebook →
Carica*, oppure salvalo su Google Drive e aprilo da li'.

Prova minima sul dataset multimodale [**PianoVAM v1.2**](https://huggingface.co/datasets/PianoVAM/PianoVAM_v1):
il notebook prende il dataset **gia' scaricato** (in locale o su Google Drive) e mostra
**tutte le informazioni di un singolo esempio**, senza riscaricare nulla.

Il file da aprire e' `pianovam_quickstart.ipynb`.

## Cosa stampa per ogni esempio

| Cella | Contenuto |
|---|---|
| 2 | tutti i campi di `metadata.json` per la registrazione scelta (split, brano, compositore, pianista, durata, punti di calibrazione) |
| 3 | il `TSV/`: una riga di nota **completa** (`onset`, `key_offset`, `frame_offset`, `note`, `velocity`) con nome della nota e durate calcolate, piu' statistiche |
| 4 | il `Fingering/`: la **stessa riga** con `hand` e `finger`, conteggi per mano/dito e quota di `Noinfo` (+ `Fingering_GT/` se esiste) |
| 5 | il `MIDI/`: formato, tracce, tempo, programmi, numero di note, ambito di altezza e prime 5 note (richiede `mido`) |
| 6 | lo `Handskeleton/`: numero di frame e landmark del primo frame (il file pesa ~120 MB: caricamento opzionale) |
| 7 | l'`Audio/`: sample rate, canali, bit, campioni, durata + estratto di 8 s da ascoltare in Colab |
| 8 | il `Video/`: codec, risoluzione, fps, durata (richiede `ffprobe`) |
| 9 | il riepilogo di **tutti i file** della registrazione (presente / dimensione) e la scheda riassuntiva |

Nella cella di configurazione si scelgono `DATASET_ROOT`, `RECORD_INDEX` (0..106) oppure
`RECORD_TIME` (es. `"2024-02-14_19-10-09"`).

## Uso

Apri il notebook in Colab e **esegui le celle dall'alto in basso**: non serve preparare nulla.

1. La cella **DATASET** sistema il dataset da sola e si puo' rieseguire quante volte vuoi:
   - se il dataset **non c'e'**, scarica dal Hub **una sola registrazione** (TSV, MIDI e
     `Fingering/`: pochi KB, parte in pochi secondi);
   - se il dataset **c'e' gia'** (anche su Google Drive) non scarica nulla e non usa la rete.
   Se salti questa cella non e' un problema: la cella che legge `metadata.json` scarica da
   sola la registrazione che le serve e, se non riesce, stampa il motivo e le due strade
   possibili (Drive montato oppure download dal Hub).
2. Vuoi **tutte le modalita'**? Fra i flag della cella 0 (CONFIGURAZIONE) metti
   `DOWNLOAD_MEDIA = True` (Audio ~65 MB + Video ~340 MB) e `DOWNLOAD_SKELETON = True`
   (Handskeleton ~120 MB), poi riesegui la cella DATASET: scarica solo cio' che manca.
3. Hai il dataset su **Google Drive**? Monta il Drive e cambia `DATASET_ROOT` nella cella di
   configurazione (le righe sono gia' pronte, commentate):

   ```python
   from google.colab import drive
   drive.mount("/content/drive")
   DATASET_ROOT = "/content/drive/MyDrive/PianoVAM_v1.2"
   ```

Per avere il dataset completo in locale (~45 GB con i video), una volta sola da terminale:

```bash
pip install huggingface_hub
huggingface-cli download PianoVAM/PianoVAM_v1 --repo-type dataset --local-dir ./PianoVAM_v1.2
```

## Requisiti

`pandas` e, per i dettagli del MIDI, `mido` (`pip install mido`); `ffmpeg`/`ffprobe` servono
per l'estratto audio e per le informazioni del video. In Colab `pandas` e `ffmpeg` sono gia'
presenti, `mido` no: il notebook funziona comunque e avvisa di cosa manca. La cella DATASET
usa `huggingface_hub` solo se deve scaricare (in Colab c'e' gia').

## Output atteso (estratto, registrazione `2024-02-14_19-10-09`)

```
Registrazione 2024-02-14_19-10-09 : i file necessari ci sono gia' in PianoVAM_v1.2 - non scarico nulla.
Cartella dataset : /content/PianoVAM_v1.2
Registrazioni    : 107
Split            : {'train': 53, 'test': 9, 'valid': 9, 'special(4hands)': 1, 'special(blurry)': 7, 'ext-train': 28}

Tutti i campi di questa registrazione:
  record_time       : 2024-02-14_19-10-09
  split             : train
  composer          : E. Grieg
  piece             : Piano Concerto
  performance_method: Solo
  ...
  P1_name           : Yonghyun

TSV/2024-02-14_19-10-09.tsv  ->  6935 note
UNA RIGA COMPLETA (riga 1 di 6935)
  onset        : 6.684375     s dall'inizio (il tasto scende)
  key_offset   : 6.740625     s in cui il dito lascia fisicamente il tasto
  frame_offset : 6.740625     s in cui il suono finisce (con pedale)
  note         : 105          numero MIDI  ->  A7
  velocity     : 92           forza del tasto, da 1 a 127

Fingering/2024-02-14_19-10-09.tsv  ->  6935 note
  hand         : R
  finger       : 5
Note con diteggiatura: 5938/6935 (85.6%)

MIDI/2024-02-14_19-10-09.mid  ->  55.7 KB
Note suonate     : 6935
Altezze          : A0 - B7 (MIDI 21 - 107)

Handskeleton/2024-02-14_19-10-09.json  ->  113.8 MB
Frame presenti   : 44731 (dal 0 al 44730)
```

I numeri sono coerenti fra le modalita': le 6.935 note del `TSV/` sono le 6.935 `note_on` del
MIDI (stesso ambito A0-B7, stessa prima nota), e i 44.731 frame dello scheletro a 60 fps
coprono gli stessi 745 s del MIDI.

## Cose da sapere sul dataset

- **Formato dei file**: in tutte le cartelle il nome e' il `<record_time>` di `metadata.json`.
  `metadata.json` e' un dizionario `{"0": {...}, "1": {...}, ...}` con 107 registrazioni.
  I file `TSV/` hanno l'header commentato con `#`, quelli `Fingering/` no (il notebook legge
  l'header in modo esplicito, per questo esiste la funzione `read_tsv`).
- **Diteggiatura**: `Fingering/` sono etichette automatiche, ~20% `Noinfo` (dal 2,7% all'81,4%
  per registrazione, mediana 15,5%); accuratezza 95,7% mano+dito e 99,2% solo mano sulle note
  etichettate, misurata su `Fingering_GT/`. La registrazione a 4 mani non ha `Fingering/`.
- **Snippet del dataset card**: il README su Hugging Face mostra
  `load_dataset("PianoVAM/PianoVAM_v1")` con le colonne `audio_path`, `video_path`, `midi_path`.
  Quello era il builder della v1.0: oggi la config predefinita si chiama `midi_notes` e
  contiene solo le 5 colonne delle note (527.302 righe da tutti i `TSV/`), senza percorsi.
  Per l'esempio multimodale completo servono `metadata.json` + le cartelle, come fa questo
  notebook. La `metadata.json` e' anche il file canonico degli split.
- **Privacy**: nei video della pianista "jiwoo" il busto e' sfocato; tastiera e mani no.
- **Licenza**: CC BY-NC-SA 4.0, solo uso non commerciale.

## Citazione

```bibtex
@inproceedings{kim2025pianovam,
  title={PianoVAM: A Multimodal Piano Performance Dataset},
  author={Kim, Yonghyun and Park, Junhyung and Bae, Joonhyung and Kim, Kirak and Kwon, Taegyun and Lerch, Alexander and Nam, Juhan},
  booktitle={Proceedings of the 26th International Society for Music Information Retrieval Conference (ISMIR)},
  year={2025}
}
```
