# Responsive Poster Design

## Goal
Rendere l'LSD Neon Poster responsive a qualsiasi viewport (mobile, tablet, desktop) con centratura e scaling proporzionale.

## Approccio: CSS Scaling
Il canvas p5.js rimane a risoluzione fissa 880×640. Il responsive è gestito interamente via CSS, senza modificare la logica di rendering o le coordinate interne.

## Modifiche

### CSS
- **Body**: display flex, centratura orizzontale e verticale, overflow hidden
- **`#p5canvas`**: contenitore ad aspect-ratio 880:640, dimensioni calcolate con viewport units (`width: min(100vw, calc(100vh * 880/640))` e viceversa per height)
- **Canvas `<canvas>`**: override degli stili inline di p5 con `!important` per adattarsi al contenitore
- **`#bocca`**: da pixel fissi (408.8px) a percentuale relativa al viewport
- **`#start-overlay`**: già full-viewport, nessuna modifica

### p5.js (nessuna modifica al codice)
- `createCanvas(880, 640)` invariato — le coordinate interne restano 880×640
- `mouseX`/`mouseY` corretti automaticamente da p5.js quando il canvas è CSS-scalato
- `windowResized()` opzionale per gestire resize dinamici del viewport

### File interessati
- `index.html` (solo sezione `<style>`)
