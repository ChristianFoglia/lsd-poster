# LSD · Neon Poster — Riepilogo Progetto

## Descrizione
Singola pagina HTML autosufficiente che genera un poster interattivo psichedelico neon con p5.js + Tone.js. Canvas 880×640 landscape, estetica acid/psychedelic, reattivo al suono (mic o oscillator di fallback).

## Architettura (3 Layer + Extra)

### 1. Sfondo — Ciclo HSB ipnotico
- `bgHue` cicla a velocità `2.2°/frame`
- Saturazione oscilla: `70 ± 15`
- Luminosità oscilla: `95 ± 5`
- **Flash audio-reattivi**: picchi di audio innescano flash con intensità proporzionale, luminosità al 100%, saturazione collassa, hue si sposta (`hueShift = intensità² × 200`)

### 2. Stella — DuplicatingStar + Buffer Feedback
- **Classe `DuplicatingStar`**: spawn ogni 55 frame, scala cresce linearmente (`+0.018/frame`), max scale 7.0
- **Vertici elastici**: ogni vertice è una particella con spring verso home + random walk audio-reattivo (`shatterStr = audio × 2.5`) — la forma si deforma col suono
- **Detriti (`StarParticle`)**: emessi dai vertici quando `audioSmoothed > 0.06`, si muovono con moto browniano, boundati (`maxDist: 80–140px`), con alone + nucleo colorato
- **Buffer feedback**: `starBuffer` (offscreen Graphics), fade alfa `3 + (1 - audio) × 20`, renderizzato su canvas con `blendMode(ADD)` e multi-pass (`passes = 1 + floor(audio × 5)`) creando glow/bloom
- **CurveTightness audio-reattivo**: `-0.25 - audio × 0.35` per arrotondamento variabile
- Jitter vertici extra: `amp = audio × 35` con angolo `i × 1.618 + age × 0.025`

### 3. Testo LSD — TextParticle
- **Campionamento**: Path2D dalle coordinate SVG (L/S/D), fallback a `text('LSD')` via p5
- **Particelle**: ogni pixel campionato è una `TextParticle` con posizione home
- **Interazione mouse**: repulsione (`strength 6.0`) + orbita (`strength 2.5`) entro soglia 150px
- **Spring+damping**: ritorno elastico (`0.035`, damping `0.88`)
- **Trail**: buffer di 8 posizioni, fade alpha quadratico

### 4. Extra — BgParticle + Bocca

#### BgParticle
- 60 particelle di sfondo, quadrate (`rect` con `rotate`), 20–80px
- Random walk caotico con sway, rotazione indipendente
- Audio-reattive: alpha incrementa, size cresce, caos aumenta con l'audio
- Colori oscillanti (±80° da bgHue), due quadrati per particella

#### Bocca (elemento HTML)
- `<img id="bocca" src="bocca.webp">` — posizionato con CSS fixed centrato
- Larghezza 408.8px (1022×0.4, stesso scale 0.4 del SVG)
- `mix-blend-mode: multiply` per trasparenza moltiplicativa
- `pointer-events: none`, `z-index: 5`

## Config (in `CONFIG`)
| Parametro | Valore | Descrizione |
|-----------|--------|-------------|
| `canvasW/H` | 880×640 | Dimensioni canvas |
| `starSpawnInterval` | 55 | Frame tra una stella e l'altra |
| `starGrowthRate` | 0.018 | Incremento scala per frame |
| `starMaxScale` | 7.0 | Scala massima prima della rimozione |
| `bgCycleSpeed` | 2.2 | Velocità ciclo tonalità sfondo |
| `mouseThreshold` | 150 | Distanza attivazione mouse |
| `trailLength` | 8 | Lunghezza trail particelle testo |
| `springStiffness` | 0.035 | Ritorno elastico |
| `damping` | 0.88 | Damping movimento |
| `audioSmooth` | 0.25 | Fattore smoothing audio |

## Audio (Tone.js)
- Attivato al click su overlay start
- `Tone.UserMedia` (mic) con fallback `Tone.Oscillator(120, 'sawtooth')`
- `Tone.FFT(256)` — analisi spettrale
- Conversione dB → 0-1: `(val + 90) / 90` se `raw[0] < -10`
- Smoothing: `audioSmoothed += (avg - audioSmoothed) × 0.25`
- In assenza audio: `audioSmoothed *= 0.985`

## Struttura File
```
Es 1 Workshop/
├── index.html      # Singolo file: HTML + CSS + p5.js + Tone.js + SVG inline
├── LSD.svg         # SVG sorgente (riferimento visivo, ruotato landscape)
├── bocca.webp      # Immagine bocca (elemento HTML overlay)
└── PROJECT_SUMMARY.md  # Questo documento
```

## Dipendenze CDN
- p5.js 1.9.0
- Tone.js 14.7.77

## Note Tecniche
- Funziona via `file://` — nessun server richiesto
- ColorMode: HSB `(360, 100, 100, 100)` — tutti i fill/stroke in HSB
- `starBuffer` ha `colorMode(HSB, 360, 100, 100, 100)` — stesso spazio colore del canvas principale
- Le stelle sono renderizzate su buffer separato per l'effetto feedback
- Le particelle di testo sono indipendenti dal buffer stella

## Flusso draw()
1. Calcola bgHue, sat, bri (ciclo HSB)
2. Gestione flash audio-reattivo
3. `background(bgHue, sat, bri)`
4. Update + display `BgParticle` (×60)
5. Lettura FFT → aggiorna `audioSmoothed`
6. Fade `starBuffer` → update stelle → render buffer su canvas (ADD multi-pass)
7. Update + display `TextParticle`
