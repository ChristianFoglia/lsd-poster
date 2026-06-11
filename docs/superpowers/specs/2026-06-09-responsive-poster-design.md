# Responsive Poster Design

## Goal
Rendere l'LSD Neon Poster responsive: su PC invariato, su mobile (verticale) tutto ruota di 90° ed è centrato.

## Approccio
Il canvas p5.js rimane a risoluzione fissa 880×640. Il responsive è gestito via CSS + media query, senza modificare la logica di rendering o le coordinate interne.

## Desktop (invariato)
- Canvas 880×640 centrato orizzontalmente e verticalmente nel viewport
- `#bocca`: centrata, dimensioni originali (408.8px)
- Stella, parola, sfondo: nessuna modifica

## Mobile (rilevato da `@media (orientation: portrait)` o `aspect-ratio < 1`)
- **Rotazione 90°**: il container `#p5canvas` viene ruotato via CSS `transform: rotate(90deg)`, effettivamente portando il canvas 880×640 in orientamento portrait (640×880)
- **Scaling**: il canvas ruotato viene scalato proporzionalmente per riempire il viewport (`width: 100vh`, `height: 100vw` dopo rotazione)
- **Centratura**: tutti gli elementi (stella, bocca, parola) sono già centrati nelle coordinate 880×640 originali — dopo rotazione 90° restano centrati
- **`#bocca`**: dimensioni scalate con `calc()` in base al viewport

## Elementi interessati
- `index.html` — sezione `<style>` (media query, rotazione, scaling)
- Nessuna modifica al codice p5.js

## Rilevamento mobile
```css
/* Portrait / mobile: aspect-ratio verticale */
@media (orientation: portrait) and (max-aspect-ratio: 4/3) {
  #p5canvas {
    transform: rotate(90deg);
    /* scaling per riempire lo schermo */
  }
}
```
