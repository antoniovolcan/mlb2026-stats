# MLB 2026 Stats — CONTEXT

## Lo esencial

| | |
|---|---|
| **Archivo único** | `C:\Users\anton\OneDrive\Escritorio\mlb2026-stats\index.html` |
| **Stack** | HTML + CSS + Vanilla JS. Sin frameworks. Todo en un solo archivo. |
| **Repo** | `https://github.com/antoniovolcan/mlb2026-stats.git` |
| **Deploy** | Vercel, auto-deploy desde `main` |
| **Git** | Antonio Volcan / soyvolcom@gmail.com |
| **Regla** | Cada cambio termina con `git add index.html && git commit && git push` |
| **APIs** | `statsapi.mlb.com` · `bdfed.stitch.mlbinfra.com` |
| **Fuentes** | Bebas Neue, Azeret Mono (números/stats), Sora (texto) — Google Fonts |

---

## Design System

```css
--bg:#0a0c0f  --bg2:#111318  --bg3:#181c22  --bg4:#1e2330
--border:#252b38  --border2:#2e3647
--text:#e8ecf4  --text2:#8892a4  --text3:#4a5568
--accent:#c8a84b  --accent2:#e8c96a
--red:#e05252  --green:#4caf82  --blue:#4a90d9
--radius:8px  --radius2:12px
```

---

## 4 Secciones (tabs en header sticky)

| Tab | HTML ID | Qué hace |
|---|---|---|
| Equipos | `sec-equipos` | Standings por división + últimos 10 juegos con pills W/L y tooltip. **Tab por defecto al cargar.** |
| Bateadores | `sec-bateadores` | Roster con stats inline, gamelog expandible, desglose D/Z por play-by-play |
| Pitcher vs Equipo | `sec-versus` | Pitcher seleccionado vs bateadores de un equipo rival — historial bateador a bateador |
| Pitcher vs Pitcher | `sec-pvs` | Dos pitchers lado a lado — stats de toda su carrera por año, gamelog expandible por temporada, box score por partido |

> **Nota:** el antiguo tab "Pitchers" fue eliminado.

---

## Columnas de Stats

### BAT_INLINE — Bateadores (11 cols)
```
AB · H · HR · SO · BB · R · RBI · AVG · OBP · SLG · OPS
```
Orden de filas: descendente por AB.

### PVS_COLS — Pitcher vs Pitcher, stats por temporada (10 cols)
```
IP · K · H · R · HR · ERA · WHIP · AVG · OBP · OPS
```

### PVS_GL_MAP — Pitcher vs Pitcher, gamelog por partido
```
IP · K · H · R · HR · ERA · BB (slot WHIP) · — · — · —
```

---

## Estado y Caché JS

```js
// Bateadores
let currentBatTeamId = 'all';
let currentBatSplit = 'general';
let batRosterCache = {};
let batStatsCache = {};
let gameLogCache = {};
let gameSplitCache = {};
let openGameLogs = {};
let openGameSplits = {};

// Versus (Pitcher vs Equipo)
let vsPitchersList = [];        // cache global de todos los pitchers (también usado por pvs)
let vsSelectedPitcherId = null;
let vsSelectedPitcherName = '';

// Pitcher vs Pitcher
let pvsA = null;                // { id, name, teamId, team }
let pvsB = null;
let pvsCareerCache = {};        // { [pid]: yearByYear splits[] }
let pvsGameLogCache = {};       // { `${pid}_${year}`: games[] }
let pvsOpenSeasons = {};        // { `${side}_${pid}_${year}`: bool }
let pvsBoxScoreCache = {};      // { gamePk: linescoreData }
```

---

## Funciones JS Clave

```js
// Bateadores
selectAllBatters()
selectBatTeam(id)
renderBatRoster(roster)
batterHTML(p, rank)
toggleGameLog(pid, name)
renderGameLog(pid, games)
toggleGameSplit(splitId, batterId, gamePk)
filterBatters(q)

// Versus — Pitcher vs Equipo
vsEnsurePitchers()             // precarga vsPitchersList al iniciar
vsPitcherSearch(q)
vsSelectPitcher(id, name, teamId)
vsOnOppTeamSelect()
vsLoadPitcherVsTeam(pitId, oppTeamId)
renderVsTeamCard(pit, oppTeam, oppTeamId, results)

// Pitcher vs Pitcher
pvsSearch(side, q)
pvsSelect(side, id, name, teamId, team)
pvsLoadPanel(side, pid, name, teamId, team)   // fetch yearByYear + auto-expande año actual
pvsBuildPanel(side, pid, name, hand, currentTeam, splits)
pvsToggleSeason(side, pid, year)              // inserta <tr> reales en la tabla
pvsToggleBoxScore(gamePk, gameRow, gameData)
pvsRenderBoxScore(ls, gameData)

// Standings
loadStandings()                // carga al inicio y al volver a Equipos
buildLast10(teamId, teamGames)
showGameTip(e, pill) / hideGameTip()

// Logos
logoOutlineClass(teamId)
const SOFT_OUTLINE_TEAMS = new Set([158,135,145,112,134,143,108])

// Tooltips de stats
showStatTip(e, text) / hideStatTip() / positionTip(e, tip)

// Middle-click
// Global mousedown listener: abre bateadores/pitcher en mlb.com/player/{id},
// tabs con #hash en nueva pestaña
```

---

## Clases CSS Importantes

```
// Bateadores / Pitchers (gamelog compartido)
.bat-row-wrap / .bat-row / .bat-info / .bat-stats
.bat-stat / .bat-stat-lbl / .bat-stat-val
.bat-name    ← usado por filterBatters
.hand-tag.hand-L/R/S  → Z / D / A
.gamelog-panel / .gl-game-wrap / .gl-split-panel
.result-pill / .pill-w / .pill-l
.logo-outline-full / .logo-outline-soft

// Pitcher vs Pitcher
.pvs-builder / .pvs-side / .pvs-panels / .pvs-panel
.pvs-panel-header / .pvs-photo / .pvs-player-info
.pvs-table-wrap / .pvs-table / .pvs-season-row (clickeable)
.pvs-season-group (tbody) / .pvs-arrow
.pvs-gl-subheader / .pvs-game-row (clickeable → box score)
.pvs-game-row-active / .pvs-boxscore-row / .pvs-boxscore
.pvs-bs-table / .pvs-bs-team / .pvs-bs-scored / .pvs-bs-zero
.pvs-bs-sep / .pvs-bs-rhe / .pvs-dec-w / .pvs-dec-l
.pvs-totals

// Post-it
#postit / #postit-bar / #postit-body / #postit-text
#postit-toggle-fab / .postit-btn
#postit.minimized
```

---

## IDs HTML Clave

```
#bat-roster-area       → tabla bateadores
#bat-player-search     → input búsqueda bateadores
#bat-team-select       → dropdown equipo bateadores

#vs-pitcher-search     → autocomplete pitcher (Pitcher vs Equipo)
#vs-pitcher-results    → dropdown resultados
#vs-opp-team           → select equipo rival
#vs-result             → div resultado tabla VS

#pvs-search-a/b        → autocomplete pitcher A y B
#pvs-results-a/b       → dropdown resultados A y B
#pvs-panel-a/b         → paneles de stats carrera

#standings-area        → contenedor standings
#global-tip            → tooltip flotante global
#postit                → nota flotante draggable
#postit-toggle-fab     → botón para reabrir el postit
```

---

## Features Implementados

### Equipos
- Standings por división, todas en una sola vista
- Pills W/L 30×30px con tooltip flotante (logo, score, fecha, resultado)
- `endDate` del schedule es dinámico → siempre trae el día de hoy
- Logo con outline blanco contorneando el SVG

### Bateadores
- Dropdown TODOS — MLB o por equipo
- Buscador en tiempo real
- Mano: Z / D / A · sin POS
- Gamelog: últimos 15 juegos · click en fila → desglose D/Z por play-by-play
- Tooltips en headers: AVG, OBP, SLG, OPS

### Pitcher vs Equipo (`sec-versus`)
- Autocomplete pitcher (lista global `vsPitchersList`)
- Select equipo rival (30 equipos)
- Preview: foto pitcher circular VS logo equipo
- Tabla: Bateador · Mano · AB · H · HR · BB · SO · AVG · OPS
- Sin historial → atenuados al final, ordenados por AB

### Pitcher vs Pitcher (`sec-pvs`)
- Dos buscadores de pitcher (A y B) con autocomplete
- Stats de **toda la carrera** separadas por año via `yearByYear`
- Columnas: AÑO · EQUIPO · IP · K · H · R · HR · ERA · WHIP · AVG · OBP · OPS
- Temporadas ordenadas descendente (más reciente arriba)
- **Temporada actual (2026) se auto-expande** al cargar el pitcher
- Click en temporada → gamelog de esa temporada (más reciente primero)
  - Gamelog como `<tr>` reales → alineación perfecta con columnas de la tabla
  - Columnas gamelog: FECHA · OPP · IP · K · H · R · HR · ERA · BB
  - W en verde, L en rojo
- Click en partido del gamelog → **box score inning a inning** (1–9 + R H E)
  - Runs anotadas resaltadas en dorado
- Fila TOTAL al final de cada panel

### Post-it flotante
- `position: fixed` → visible en todos los tabs
- Color azul marino (`#1a3f6e`) con header `#1a5fa8`
- Draggable desde la barra de título (mouse y touch)
- **Resize en ambas direcciones** (handle esquina inferior derecha)
- Minimizable, cerrable (reabre con botón 📌 abajo derecha)
- Guarda texto, posición y tamaño en `localStorage`

### Actualizaciones automáticas
- **Fetch interceptor**: añade `?_v=YYYY-MM-DD` a todas las llamadas a APIs de MLB → evita caché HTTP del día anterior
- **Auto-reload**: cada hora comprueba si cambió la fecha; si sí, recarga la página
- **Badge de fecha** en el header

### Responsive (mobile)
- Nav tabs: scroll horizontal
- Bateadores: tabla con sticky left + scroll horizontal
- Pitcher vs Equipo: builder apilado, vs-side full width
- Pitcher vs Pitcher: paneles apilados, tabla con scroll horizontal, box score con scroll
- Post-it: max-width limitado, resize deshabilitado en touch

### Middle-click
- Tab del nav → abre sección en nueva pestaña (`#hash`)
- Fila de bateador → abre `mlb.com/player/{id}` en nueva pestaña
- Hash en URL al cargar → abre la sección correspondiente

---

## Archivos del Repo

```
mlb2026-stats/
├── index.html     ← único archivo de código
├── CONTEXT.md     ← este archivo
├── vercel.json    ← { "cleanUrls": true, "trailingSlash": false }
└── .gitignore
```

---

## Pendientes

| # | Feature | Notas |
|---|---|---|
| 1 | **Desglose D/Z en gamelog de Bateadores** | Ya implementado |
| 2 | **Mejoras VS** | Historial de AB individuales por bateador (pendiente) |
