# Note tecniche — mirkobruni-landing

## Sviluppo locale

FontLoader usa XHR — non funziona su `file://`. Avviare sempre un server locale:

```bash
cd mirkobruni-landing
python3 -m http.server 8080
# poi aprire http://localhost:8080
```

## Deploy

Ogni `git push origin main` triggera il deploy automatico via Cloudflare (GitHub integration).

Il sito è servito da un **Cloudflare Worker** (non Pages puro) — configurato in `wrangler.jsonc`.

URL di produzione:
- https://mirkobruni.it
- https://www.mirkobruni.it
- https://mirkobruni-landing.fareyus.workers.dev (URL worker diretto)

## Parametri grafici

Tutti i valori del materiale Three.js sono stati calibrati visivamente per ottenere il nero quasi assoluto con riflessi appena percettibili:

| Parametro | Valore | Note |
|-----------|--------|------|
| `toneMappingExposure` | 0.12 | Chiave principale per l'oscurità globale |
| `envMapIntensity` | 0.02 | Environment map quasi azzerata |
| `clearcoat` | 1.0 | Lacca dieletrica (non metallica) |
| `clearcoatRoughness` | 0.25 | Riflessi leggermente diffusi |
| `roughness` | 0.05 | Superficie quasi speculare |
| `AmbientLight` | 0.01 | Quasi nulla |
| `PointLight main` | 0.25 | Da (-3, 3, 4) |
| `PointLight fill` | 0.05 | Da (3, -2, 3) |
| Torus tube radius | 0.04 | Ridotto da 0.06 per meno riflessioni di taglio |

## Struttura importmap

Three.js r165 da jsDelivr richiede l'importmap per risolvere i bare specifiers interni ai moduli `examples/jsm/`:

```html
<script type="importmap">
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.165.0/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.165.0/examples/jsm/"
  }
}
</script>
```

Senza questo blocco: `TypeError: Failed to resolve module specifier "three"`.

## DNS mirkobruni.it (Cloudflare)

| Record | Tipo | Target | Note |
|--------|------|--------|------|
| `@` | CNAME | `mirkobruni-landing.pages.dev` | Proxied — intercettato dal Worker |
| `www` | CNAME | `mirkobruni-landing.fareyus.workers.dev` | Proxied |
| `file.`, `film.`, `ha.`, `libri.` | Tunnel | DietPi / homeassistant | Non toccare |
| MX | — | iCloud Mail | Non toccare |

## Link placeholder

Il link `·` in basso (`#link` nel CSS) è un placeholder — URL da definire.
Per aggiornarlo: modificare `href="#"` in `index.html` e `git push`.
