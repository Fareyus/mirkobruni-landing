# mirkobruni.it — Landing Page Design Spec

**Data:** 2026-06-27  
**Stato:** Approvato

---

## Obiettivo

Landing page ultraleggera per il dominio personale `mirkobruni.it`. Nessun contenuto informativo, nessuna navigazione — solo un logo animato in 3D. Effetto: lettera M in un cerchio, neri su nero assoluto, visibili esclusivamente grazie ai riflessi del materiale lucido durante la rotazione.

---

## Architettura

**File:** un singolo `index.html` (~20KB).  
**Dipendenze esterne (CDN):**
- Three.js r165: `https://cdn.jsdelivr.net/npm/three@0.165.0/build/three.module.js`
- Font JSON: `https://cdn.jsdelivr.net/npm/three@0.165.0/examples/fonts/optimer_bold.typeface.json`

Nessun bundler, nessun npm, nessun build step. La pagina è un file HTML statico servito direttamente.

**Struttura repo:**
```
mirkobruni-landing/
├── index.html
└── docs/
    └── superpowers/specs/
        └── 2026-06-27-mirkobruni-landing-design.md
```

---

## Hosting: Cloudflare Pages (piano gratuito)

1. Repo GitHub: `Fareyus/mirkobruni-landing` (pubblico)
2. Cloudflare Pages → Connect to Git → zero configurazione build
3. Custom domain `mirkobruni.it` aggiunto dalle impostazioni Pages
4. DNS gestito automaticamente da Cloudflare (già nameserver del dominio)
5. HTTPS automatico via Universal SSL

---

## Scena Three.js

### Camera
- `PerspectiveCamera`, FOV 45°, posizione `(0, 0, 5)`
- Resize listener per adattarsi alla viewport

### Geometrie
| Oggetto | Tipo Three.js | Parametri chiave |
|---------|---------------|-----------------|
| Cerchio | `TorusGeometry` | radius: 2, tube: 0.06, radialSeg: 16, tubSeg: 100 |
| Lettera M | `TextGeometry` (font: optimer_bold) | size: 2.0, depth: 0.15, bevel: abilitato sottile |
| Gruppo | `Group` | contiene anello + M, ruota come unità |

La M viene centrata nel cerchio via `geometry.computeBoundingBox()` dopo la creazione.

### Materiale
Identico per anello e M:
```js
MeshStandardMaterial({
  color: 0x080808,
  roughness: 0.05,
  metalness: 0.95,
})
```
Environment map neutra generata con `PMREMGenerator` per amplificare il gloss senza riflessi estranei.

### Luci
| Luce | Tipo | Posizione | Intensità |
|------|------|-----------|-----------|
| Ambient | `AmbientLight` bianca | — | 0.08 |
| Principale | `PointLight` bianca | (-3, 3, 4) | 3.0 |
| Fill | `PointLight` bianca | (3, -2, 3) | 0.5 |

### Animazione
- Rotazione continua sull'asse Y
- Velocità: `2π / 8` rad/s (un giro ogni 8 secondi)
- Clock Three.js → `delta` tra frame per velocità costante a qualsiasi FPS
- `renderer.setAnimationLoop(render)` — loop nativo ottimizzato

---

## Layout HTML

```
┌─────────────────────────────────┐
│                                 │
│                                 │
│            ╭─────╮              │
│           │   M   │             │
│            ╰─────╯              │
│         (occupa ~85% viewport)  │
│                                 │
│          ·                      │  ← link placeholder, centrato in basso
└─────────────────────────────────┘
```

- `<canvas>` fullscreen, `position: fixed`, background `#000000`
- Link overlay: `position: absolute`, bottom 24px, center, `color: rgba(255,255,255,0.12)`, font 11px system sans-serif
- Il link è un placeholder (`href="#"`) — l'URL reale viene aggiunto in seguito

---

## Comportamento responsive

Il canvas ridimensiona la scena via `renderer.setSize()` + aggiornamento aspect ratio camera al resize della finestra. Il logo occupa circa l'85% dell'asse più corto della viewport su qualsiasi dispositivo.

---

## Meta e SEO

- `<title>Mirko Bruni</title>`
- `<meta name="description">` assente (pagina non indicizzabile per scelta)
- `robots: noindex, nofollow` — pagina personale, non indicizzabile
- `viewport` meta corretto per mobile

---

## Decisioni aperte

- **URL del link discreto:** da definire (email, LinkedIn, GitHub). Placeholder `·` per ora.
- **Deploy:** richiede Token API Cloudflare o operazione manuale via dashboard.
