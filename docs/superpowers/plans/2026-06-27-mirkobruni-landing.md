# mirkobruni.it Landing Page — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Creare un singolo file `index.html` con una M serif in un cerchio, nero su nero assoluto, visibile solo grazie ai riflessi 3D durante la rotazione sull'asse Y, e deployarlo su Cloudflare Pages alla root di `mirkobruni.it`.

**Architecture:** Singolo file HTML statico con Three.js caricato da CDN jsDelivr. Nessun bundler, nessun npm. La M è generata come `TextGeometry` con font `optimer_bold`; il cerchio è un `TorusGeometry`. Entrambi condividono un `MeshStandardMaterial` nero lucido (roughness 0.05, metalness 0.95) con environment map neutra via `PMREMGenerator`. La rotazione Y è animata a 8s/giro via `setAnimationLoop` con delta time.

**Tech Stack:** Three.js r165 (CDN), FontLoader + TextGeometry (jsm/CDN), HTML/CSS puro, Cloudflare Pages (free tier), GitHub (`Fareyus/mirkobruni-landing`).

---

## File che verranno creati/modificati

| File | Azione | Responsabilità |
|------|--------|----------------|
| `index.html` | Create | Intera pagina: scena Three.js, materiale, luci, animazione, overlay link, meta tag |
| `.gitignore` | Create | Esclude `.superpowers/`, `.DS_Store` |

---

## Task 1: Git init + HTML skeleton

**Files:**
- Create: `index.html`
- Create: `.gitignore`

- [ ] **Step 1.1: Inizializza repo git**

```bash
cd /Users/mirko/Desktop/CLAUDE/mirkobruni-landing
git init
```

Expected output: `Initialized empty Git repository in .../mirkobruni-landing/.git/`

- [ ] **Step 1.2: Crea `.gitignore`**

```
.DS_Store
.superpowers/
```

- [ ] **Step 1.3: Crea `index.html` con skeleton minimo**

```html
<!DOCTYPE html>
<html lang="it">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Mirko Bruni</title>
  <meta name="robots" content="noindex, nofollow, noarchive, nosnippet">
  <meta name="googlebot" content="noindex, nofollow">
  <meta name="anthropic-ai" content="noindex">
  <meta name="gptbot" content="noindex">
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    html, body { width: 100%; height: 100%; overflow: hidden; background: #000; }
    canvas { display: block; }
    #link {
      position: fixed;
      bottom: 24px;
      left: 50%;
      transform: translateX(-50%);
      color: rgba(255, 255, 255, 0.12);
      font: 11px/1 system-ui, -apple-system, sans-serif;
      text-decoration: none;
      letter-spacing: 0.15em;
      transition: color 0.4s;
    }
    #link:hover { color: rgba(255, 255, 255, 0.35); }
  </style>
</head>
<body>
  <a id="link" href="#">·</a>
</body>
</html>
```

- [ ] **Step 1.4: Verifica apertura nel browser**

Apri `index.html` nel browser. Expected: pagina completamente nera con un `·` appena visibile in basso al centro.

- [ ] **Step 1.5: Commit**

```bash
git add index.html .gitignore docs/
git commit -m "chore: project skeleton + spec docs"
```

---

## Task 2: Three.js renderer + camera

**Files:**
- Modify: `index.html` — aggiunge `<script type="module">` con renderer e camera

- [ ] **Step 2.1: Aggiungi il blocco script Three.js dopo `</style>` e prima di `</body>`**

Aggiungi questo `<script>` subito prima di `</body>`:

```html
<script type="module">
  import * as THREE from 'https://cdn.jsdelivr.net/npm/three@0.165.0/build/three.module.js';

  // ── Renderer ──────────────────────────────────────────────────────────────
  const renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.toneMapping = THREE.ACESFilmicToneMapping;
  renderer.toneMappingExposure = 1.0;
  document.body.appendChild(renderer.domElement);

  // ── Scene + camera ────────────────────────────────────────────────────────
  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 100);
  camera.position.z = 5;

  // ── Render loop placeholder ───────────────────────────────────────────────
  renderer.setAnimationLoop(() => {
    renderer.render(scene, camera);
  });

  // ── Resize ────────────────────────────────────────────────────────────────
  window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
  });
</script>
```

- [ ] **Step 2.2: Verifica nel browser**

Ricarica `index.html`. Expected: pagina nera uniforme (canvas Three.js sovrapposto), `·` visibile in basso. Nessun errore in console.

- [ ] **Step 2.3: Commit**

```bash
git add index.html
git commit -m "feat: Three.js renderer + camera"
```

---

## Task 3: Material nero lucido + environment map + luci

**Files:**
- Modify: `index.html` — aggiunge import `RoomEnvironment`, material, PMREMGenerator, luci

- [ ] **Step 3.1: Aggiungi import `RoomEnvironment` in cima al blocco script**

Dopo la riga `import * as THREE from ...` aggiungi:

```js
import { RoomEnvironment } from 'https://cdn.jsdelivr.net/npm/three@0.165.0/examples/jsm/environments/RoomEnvironment.js';
```

- [ ] **Step 3.2: Aggiungi environment map e material dopo `camera.position.z = 5;`**

```js
// ── Environment map (neutra, per il gloss) ────────────────────────────────
const pmremGenerator = new THREE.PMREMGenerator(renderer);
scene.environment = pmremGenerator.fromScene(new RoomEnvironment()).texture;
pmremGenerator.dispose();

// ── Materiale nero lucido (condiviso tra anello e M) ─────────────────────
const material = new THREE.MeshStandardMaterial({
  color: 0x080808,
  roughness: 0.05,
  metalness: 0.95,
});

// ── Luci ──────────────────────────────────────────────────────────────────
scene.add(new THREE.AmbientLight(0xffffff, 0.08));

const mainLight = new THREE.PointLight(0xffffff, 3.0, 0);
mainLight.position.set(-3, 3, 4);
scene.add(mainLight);

const fillLight = new THREE.PointLight(0xffffff, 0.5, 0);
fillLight.position.set(3, -2, 3);
scene.add(fillLight);
```

- [ ] **Step 3.3: Verifica nel browser**

Ricarica. Expected: ancora pagina nera, nessun errore console. Il materiale è pronto ma non ancora applicato a nessun mesh.

- [ ] **Step 3.4: Commit**

```bash
git add index.html
git commit -m "feat: black lacquer material + env map + lights"
```

---

## Task 4: TorusGeometry (cerchio) + Group

**Files:**
- Modify: `index.html` — aggiunge torus e group alla scena

- [ ] **Step 4.1: Aggiungi group e torus dopo la sezione luci**

```js
// ── Group (contiene anello + M, ruota insieme) ────────────────────────────
const group = new THREE.Group();
scene.add(group);

// ── Cerchio (torus) ───────────────────────────────────────────────────────
const torus = new THREE.Mesh(
  new THREE.TorusGeometry(2, 0.06, 16, 100),
  material
);
group.add(torus);
```

- [ ] **Step 4.2: Verifica nel browser**

Ricarica. Expected: un cerchio sottile quasi invisibile (nero su nero) appare al centro. Potrebbe essere necessario cercare con attenzione — il gloss lo rende appena visibile con la luce principale. Nessun errore console.

- [ ] **Step 4.3: Commit**

```bash
git add index.html
git commit -m "feat: torus ring geometry"
```

---

## Task 5: FontLoader + TextGeometry per la M

**Files:**
- Modify: `index.html` — aggiunge import FontLoader + TextGeometry, carica font, crea mesh M

- [ ] **Step 5.1: Aggiungi import FontLoader e TextGeometry dopo gli import esistenti**

```js
import { FontLoader } from 'https://cdn.jsdelivr.net/npm/three@0.165.0/examples/jsm/loaders/FontLoader.js';
import { TextGeometry } from 'https://cdn.jsdelivr.net/npm/three@0.165.0/examples/jsm/geometries/TextGeometry.js';
```

- [ ] **Step 5.2: Aggiungi funzione di scaling e caricamento font dopo il torus**

```js
// ── Scala il gruppo per riempire ~85% dell'asse più corto della viewport ──
function scaleGroupToViewport() {
  const vFOV = THREE.MathUtils.degToRad(camera.fov);
  const visibleHeight = 2 * Math.tan(vFOV / 2) * camera.position.z;
  const visibleWidth = visibleHeight * camera.aspect;
  const shorter = Math.min(visibleHeight, visibleWidth);
  // Il torus ha diametro 4 unità (raggio 2)
  group.scale.setScalar((shorter * 0.85) / 4);
}

// ── Font loader + M geometry ───────────────────────────────────────────────
const fontLoader = new FontLoader();
fontLoader.load(
  'https://cdn.jsdelivr.net/npm/three@0.165.0/examples/fonts/optimer_bold.typeface.json',
  (font) => {
    const textGeo = new TextGeometry('M', {
      font,
      size: 1.8,
      depth: 0.15,
      curveSegments: 12,
      bevelEnabled: true,
      bevelThickness: 0.02,
      bevelSize: 0.01,
      bevelSegments: 5,
    });

    // Centra la M nell'origine
    textGeo.computeBoundingBox();
    const bb = textGeo.boundingBox;
    textGeo.translate(
      -(bb.max.x + bb.min.x) / 2,
      -(bb.max.y + bb.min.y) / 2,
      -(bb.max.z + bb.min.z) / 2
    );

    const mMesh = new THREE.Mesh(textGeo, material);
    group.add(mMesh);

    // Applica scaling ora che la M è nel gruppo
    scaleGroupToViewport();
  }
);

// Chiama anche prima che il font carichi (per il torus)
scaleGroupToViewport();
```

- [ ] **Step 5.3: Aggiorna il resize handler per richiamare `scaleGroupToViewport()`**

Il resize handler esistente diventa:

```js
window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
  scaleGroupToViewport();
});
```

- [ ] **Step 5.4: Verifica nel browser**

Ricarica. Expected: dopo ~1s (caricamento font), compare la M serif dentro al cerchio. Entrambi quasi invisibili (nero su nero), ma percepibili con il gloss. Il logo occupa circa l'85% della viewport. Nessun errore console.

- [ ] **Step 5.5: Commit**

```bash
git add index.html
git commit -m "feat: M TextGeometry + font loader + viewport scaling"
```

---

## Task 6: Animazione rotazione Y

**Files:**
- Modify: `index.html` — sostituisce il render loop placeholder con animazione delta-based

- [ ] **Step 6.1: Aggiungi il clock dopo la sezione font loader**

```js
// ── Clock per animazione delta-based ─────────────────────────────────────
const clock = new THREE.Clock();
const ROTATION_PERIOD = 8; // secondi per giro completo
```

- [ ] **Step 6.2: Sostituisci il render loop placeholder con quello animato**

Trova e sostituisci:
```js
renderer.setAnimationLoop(() => {
  renderer.render(scene, camera);
});
```

Con:
```js
renderer.setAnimationLoop(() => {
  const delta = clock.getDelta();
  group.rotation.y += (Math.PI * 2 / ROTATION_PERIOD) * delta;
  renderer.render(scene, camera);
});
```

- [ ] **Step 6.3: Verifica nel browser**

Ricarica. Expected: la M e il cerchio ruotano lentamente sull'asse Y a ~8s/giro. Quando la M è di taglio diventa quasi invisibile; quando è frontale i riflessi la rendono leggibile. Rotazione fluida senza stuttering. Nessun errore console.

- [ ] **Step 6.4: Commit**

```bash
git add index.html
git commit -m "feat: Y-axis rotation animation at 8s/rev"
```

---

## Task 7: Review visiva finale + polish

**Files:**
- Modify: `index.html` — aggiustamenti fini se necessari

- [ ] **Step 7.1: Verifica su mobile (DevTools → device toolbar)**

Apri DevTools → Toggle Device Toolbar → seleziona iPhone 12 (390×844).
Expected: logo centrato, occupa ~85% della larghezza. Link `·` visibile in basso. Nessun overflow.

- [ ] **Step 7.2: Verifica su viewport molto larga (es. 2560px)**

Ridimensiona la finestra a schermo intero su monitor largo.
Expected: logo scala correttamente, non deborda.

- [ ] **Step 7.3: (Se necessario) Aggiusta `size` della M**

Se la M appare troppo grande o piccola rispetto al cerchio (diametro interno del torus = ~3.88 unità, la M idealmente occupa ~80% di quello), modifica il parametro `size` in `TextGeometry` tra 1.6 e 2.0 fino a ottenere la proporzione voluta.

- [ ] **Step 7.4: Commit polish**

```bash
git add index.html
git commit -m "polish: visual review adjustments"
```

---

## Task 8: GitHub repo + push

**Files:**
- Nessuna modifica al codice

- [ ] **Step 8.1: Verifica che `gh` CLI sia autenticato**

```bash
gh auth status
```

Expected: `Logged in to github.com as Fareyus`

- [ ] **Step 8.2: Crea repo GitHub pubblico**

```bash
gh repo create Fareyus/mirkobruni-landing --public --description "Landing page mirkobruni.it"
```

Expected: URL del repo stampato in output.

- [ ] **Step 8.3: Aggiungi remote e pusha**

```bash
git remote add origin https://github.com/Fareyus/mirkobruni-landing.git
git branch -M main
git push -u origin main
```

Expected: `Branch 'main' set up to track remote branch 'main' from 'origin'.`

---

## Task 9: Cloudflare Pages — deploy e dominio custom

**Prerequisiti:**
- Account ID Cloudflare (visibile su https://dash.cloudflare.com → seleziona account → URL contiene `/ACCOUNT_ID/`)
- API Token con permesso `Cloudflare Pages: Edit` (crea su https://dash.cloudflare.com/profile/api-tokens)
- L'account Cloudflare deve essere già connesso a GitHub via OAuth (una-tantum: Dashboard → Pages → "Connect to Git" → autorizza GitHub)

- [ ] **Step 9.1: Crea il progetto Cloudflare Pages via API**

```bash
export CF_ACCOUNT_ID="IL_TUO_ACCOUNT_ID"
export CF_TOKEN="IL_TUO_API_TOKEN"

curl -s -X POST \
  "https://api.cloudflare.com/client/v4/accounts/${CF_ACCOUNT_ID}/pages/projects" \
  -H "Authorization: Bearer ${CF_TOKEN}" \
  -H "Content-Type: application/json" \
  --data '{
    "name": "mirkobruni-landing",
    "production_branch": "main",
    "source": {
      "type": "github",
      "config": {
        "owner": "Fareyus",
        "repo_name": "mirkobruni-landing",
        "production_branch": "main",
        "pr_comments_enabled": false,
        "deployments_enabled": true
      }
    },
    "build_config": {
      "build_command": "",
      "destination_dir": "/",
      "root_dir": ""
    }
  }' | python3 -m json.tool
```

Expected: JSON con `"success": true` e subdomain tipo `mirkobruni-landing.pages.dev`.

- [ ] **Step 9.2: Aspetta il primo deploy automatico**

Cloudflare Pages rileva il push su `main` e avvia il deploy (30-60 secondi). Controlla lo stato:

```bash
curl -s "https://api.cloudflare.com/client/v4/accounts/${CF_ACCOUNT_ID}/pages/projects/mirkobruni-landing/deployments" \
  -H "Authorization: Bearer ${CF_TOKEN}" | python3 -m json.tool | grep -E '"status"|"url"'
```

Expected: `"status": "success"` e URL tipo `https://abc123.mirkobruni-landing.pages.dev`

Verifica aprendo quell'URL nel browser — deve mostrare la landing page.

- [ ] **Step 9.3: Aggiungi dominio custom `mirkobruni.it`**

```bash
curl -s -X POST \
  "https://api.cloudflare.com/client/v4/accounts/${CF_ACCOUNT_ID}/pages/projects/mirkobruni-landing/domains" \
  -H "Authorization: Bearer ${CF_TOKEN}" \
  -H "Content-Type: application/json" \
  --data '{"name": "mirkobruni.it"}' | python3 -m json.tool
```

Expected: `"success": true`. Cloudflare crea automaticamente il record DNS (CNAME o ALIAS) perché gestisce già i nameserver di `mirkobruni.it`.

- [ ] **Step 9.4: Verifica propagazione DNS**

Aspetta 1-2 minuti, poi:

```bash
curl -s -o /dev/null -w "%{http_code}" https://mirkobruni.it
```

Expected: `200`

Apri https://mirkobruni.it nel browser. Expected: landing page con la M rotante, HTTPS attivo (lucchetto verde).

---

## Note di manutenzione

- **Aggiornare il link discreto:** modifica `href="#"` in `index.html` con l'URL reale, poi `git push` — Cloudflare Pages fa redeploy automatico in ~30s.
- **Aggiornare Three.js:** cambia il numero di versione in tutti i CDN URL (3 occorrenze: `three.module.js`, `RoomEnvironment.js`, `FontLoader.js`, `TextGeometry.js`, `optimer_bold.typeface.json`). Testa in locale prima di pushare.
