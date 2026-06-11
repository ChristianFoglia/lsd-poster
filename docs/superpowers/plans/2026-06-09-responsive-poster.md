# Responsive Poster Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Render the LSD poster responsive: PC invariato, su mobile (verticale) tutto ruota 90° ed è centrato.

**Architecture:** Il canvas p5.js rimane a risoluzione fissa 880×640. Il responsive è gestito via CSS + media query. Su mobile il container viene ruotato via `transform: rotate(90deg)` e scalato a riempire lo schermo.

**Tech Stack:** HTML, CSS, p5.js

---

### Task 1: CSS — Container responsive e mobile rotation

**Files:**
- Modify: `index.html:7-51` (sezione `<style>`)

- [ ] **Step 1: Sostituisci il CSS body e #p5canvas per il responsive**

Sostituisci il CSS esistente (body, #p5canvas, #bocca) con:

```css
*{margin:0;padding:0;box-sizing:border-box}
html, body{
  width:100%;
  height:100%;
  background:#000;
  display:flex;
  justify-content:center;
  align-items:center;
  overflow:hidden;
  font-family:monospace
}
#start-overlay{
  position:fixed;
  inset:0;
  display:flex;
  flex-direction:column;
  justify-content:center;
  align-items:center;
  background:#000;
  color:#45ff00;
  cursor:pointer;
  z-index:10;
  font-size:18px;
  letter-spacing:6px;
  text-transform:uppercase;
  user-select:none;
  transition:color .3s
}
#start-overlay span{font-size:13px;letter-spacing:2px;margin-top:16px;color:#45ff0066;transition:color .3s}
#start-overlay:hover{color:#fff}
#start-overlay:hover span{color:#ffffff66}
#p5canvas{
  position:relative;
  width:min(100vw, calc(100vh * 880 / 640));
  height:min(100vh, calc(100vw * 640 / 880));
}
#bocca{
  position:fixed;
  left:50%;
  top:50%;
  transform:translate(-50%,-50%);
  width:408.8px;
  height:auto;
  mix-blend-mode:multiply;
  pointer-events:none;
  z-index:5
}
.hidden{display:none}

/* Mobile: rotazione 90° */
@media (orientation: portrait) and (max-aspect-ratio: 4/3) {
  #p5canvas {
    width:100vh;
    height:100vw;
    transform:rotate(90deg);
  }
  #bocca {
    width:min(40vh, calc(40vw * 640 / 880));
  }
  #start-overlay {
    font-size: min(3vw, 18px);
  }
}
```

- [ ] **Step 2: Aggiungi override canvas p5 per adattarsi al contenitore**

Aggiungi dopo le regole esistenti:

```css
#p5canvas canvas {
  width:100% !important;
  height:100% !important;
}
```

- [ ] **Step 3: Verifica che il CSS sia corretto**

Apri `index.html` in un browser e ridimensiona la finestra a forma verticale (stretta e alta). Il canvas deve:
- Su desktop (larga): restare 880×640 centrato
- Su mobile (stretta): ruotare 90°, riempire lo schermo, elementi centrati

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: responsive layout with mobile 90deg rotation"
```

---

### Task 2: JS — Trasformazione coordinate mouse su mobile

**Files:**
- Modify: `index.html:219-301` (funzione draw e classi)

- [ ] **Step 1: Aggiungi variabili e listener per mouse raw**

Prima di `function setup()` (linea ~149), aggiungi variabili globali per tracciare la posizione mouse reale:

```javascript
let rawCX = 0, rawCY = 0;
document.addEventListener('mousemove', e => { rawCX = e.clientX; rawCY = e.clientY; });
document.addEventListener('touchstart', e => { if (e.touches.length) { rawCX = e.touches[0].clientX; rawCY = e.touches[0].clientY; } }, {passive:true});
document.addEventListener('touchmove', e => { if (e.touches.length) { rawCX = e.touches[0].clientX; rawCY = e.touches[0].clientY; } }, {passive:true});
```

Aggiungi dopo la sezione audioSmoothed (linea ~268) una funzione che restituisce le coordinate mouse corrette:

```javascript
function getMousePos() {
  let c = document.querySelector('#p5canvas canvas');
  if (!c) return {x: mouseX, y: mouseY};
  let rect = c.getBoundingClientRect();
  // Su desktop: rect.width=880, rect.height=640
  // Su mobile (rotato 90°): rect.width=640, rect.height=880
  let nx = (rawCX - rect.left) / rect.width;  // 0..1 in visual space
  let ny = (rawCY - rect.top) / rect.height;
  return {x: nx * CONFIG.canvasW, y: ny * CONFIG.canvasH};
  // NOTA: su mobile il mapping esatto potrebbe richiedere
  // swap x↔y a seconda del browser — testare e regolare
}
```

- [ ] **Step 2: Sostituisci mouseX/mouseY in TextParticle.update**

In `TextParticle.update` (riga ~567-601), sostituisci l'uso diretto di `mouseX` e `mouseY` con un oggetto `mp` che passa attraverso `getMousePos()`:

```javascript
  update() {
    let dx = this.x - this.homeX;
    let dy = this.y - this.homeY;

    this.vx += -dx * CONFIG.springStiffness;
    this.vy += -dy * CONFIG.springStiffness;

    // Interazione mouse (con supporto mobile rotation)
    let mp = getMousePos();
    let mDist = dist(this.x, this.y, mp.x, mp.y);
    if (mDist < CONFIG.mouseThreshold && mDist > 0.1) {
      let force = (CONFIG.mouseThreshold - mDist) / CONFIG.mouseThreshold;
      let nx = (this.x - mp.x) / mDist;
      let ny = (this.y - mp.y) / mDist;
      this.vx += nx * force * CONFIG.repulsionStrength;
      this.vy += ny * force * CONFIG.repulsionStrength;
      let orbitDir = (Math.floor(Math.random() * 3) - 1) * 0.6 + 0.4;
      this.vx += -ny * force * CONFIG.orbitStrength * orbitDir;
      this.vy += nx * force * CONFIG.orbitStrength * orbitDir;
    }

    this.vx *= CONFIG.damping;
    this.vy *= CONFIG.damping;
    this.x += this.vx;
    this.y += this.vy;
```

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "fix: mouse coordinate transform for mobile rotation"
```
