# MLB 2026 Stats — CONTEXT (para nueva sesión)

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
| Equipos | `sec-equipos` | Standings por división, pills W/L últimos 10 juegos con tooltip |
| Bateadores | `sec-bateadores` | Roster con stats inline, gamelog expandible, desglose por mano |
| Pitchers | `sec-pitchers` | Roster con stats inline, gamelog expandible |
| Versus | `sec-versus` | Pitcher vs equipo rival — historial bateador a bateador |

---

## Columnas de Stats

### PITCH_INLINE — Pitchers (13 cols, sin POS ni QS)
```
IP · K · H · R · HR · ERA · WHIP · W · L · G · GS · AVG · OBP · OPS
```
Orden de filas: descendente por IP.

### BAT_INLINE — Bateadores (11 cols, sin PA ni POS)
```
AB · H · HR · SO · BB · R · RBI · AVG · OBP · SLG · OPS
```
Orden de filas: descendente por AB.

---

## Estado y Caché JS

```js
// Pitchers
let currentTeamId = null;          // 'all' | number
let rosterCache = {};              // { 'all': [...], [id]: [...] }

// Bateadores
let currentBatTeamId = 'all';      // 'all' | number
let currentBatSplit = 'general';   // 'general' | 'vsLeft' | 'vsRight' (sin UI, interno)
let batRosterCache = {};           // { 'all': [...], [id]: [...] }
let batStatsCache = {};            // { `${pid}_${split}`: statData }

// Gamelogs
let gameLogCache = {};             // { pid: games[] }
let gameSplitCache = {};           // { splitId: { R:{ab,h,hr,bb,so,rbi,avg}, L:{...} } }
let openGameLogs = {};             // { pid: bool }
let openGameSplits = {};           // { splitId: bool }

// Versus
let vsPitchersList = [];           // cache global de todos los pitchers
let vsSelectedPitcherId = null;
let vsSelectedPitcherName = '';
```

---

## Funciones JS Clave

```js
// Pitchers
selectAllPitchers()              // API bdfed, sortStat=inningsPitched, carga TODOS
selectTeam(id)                   // carga pitchers de un equipo por teamId
renderRoster(roster)             // renderiza tabla de pitchers en #roster-area
pitcherHTML(p, rank)             // HTML de una fila de pitcher
togglePitcherLog(pid, name)      // abre/cierra gamelog de un pitcher
renderPitcherLog(pid, games)     // renderiza las filas del gamelog
filterPitchers(q)                // filtra [id^="prow-wrap-"] por .bat-name

// Bateadores
selectAllBatters()               // API bdfed, sortStat=atBats, carga TODOS
selectBatTeam(id)                // carga bateadores de un equipo
renderBatRoster(roster)          // renderiza tabla en #bat-roster-area
batterHTML(p, rank)              // HTML de una fila de bateador
toggleGameLog(pid, name)         // abre/cierra gamelog de un bateador
renderGameLog(pid, games)        // renderiza filas del gamelog
toggleGameSplit(splitId, batterId, gamePk)  // expande desglose D/Z por juego
parseGameHandSplit(allPlays, batterId)      // parsea play-by-play → { R:{...}, L:{...} }
renderGameSplit(splitId, sp)     // renderiza el panel D/Z (omite lados sin AB)
filterBatters(q)                 // filtra [id^="bat-wrap-"] por .bat-name

// Versus
vsEnsurePitchers()               // carga lista global para autocomplete
vsPitcherSearch(q)               // filtra dropdown de hasta 12 resultados
vsSelectPitcher(id, name, teamId)
vsOnOppTeamSelect()
vsLoadPitcherVsTeam(pitId, oppTeamId)  // fetch historial + render tabla
renderVsTeamCard(pit, oppTeam, oppTeamId, results)

// Standings
buildLast10(teamId, teamGames)   // genera pills W/L con tooltip de juego
showGameTip(e, pill)             // tooltip flotante de una pill
hideGameTip()

// Logos
logoOutlineClass(teamId)         // → 'logo-outline-full' | 'logo-outline-soft'
const SOFT_OUTLINE_TEAMS = new Set([158,135,145,112,134,143,108])
// MIL=158, SD=135, CWS=145, CHC=112, PIT=134, PHI=143, LAA=108

// Tooltips de stats
showStatTip(e, text)             // reutiliza #global-tip con texto plano
hideStatTip()
positionTip(e, tip)              // posiciona #global-tip cerca del cursor
```

---

## Clases CSS Importantes

```
.bat-row-wrap        → wrapper completo de una fila (incluye gamelog panel)
.bat-row             → fila clickeable del jugador
.bat-info            → celda sticky izquierda (nombre, mano)
.bat-stats           → celda de stats (derecha, scrollable)
.bat-stat            → un stat (label + valor)
.bat-stat-lbl        → etiqueta del stat (10px, texto3)
.bat-stat-val        → valor del stat (Azeret Mono)
.bat-name            → span del nombre del jugador ← usado por filterPitchers/filterBatters
.hand-tag.hand-L     → etiqueta Z (zurdo, azul)
.hand-tag.hand-R     → etiqueta D (derecho, dorado)
.hand-tag.hand-S     → etiqueta A (switch, verde)
.gamelog-panel       → panel expandible del gamelog
.gl-game-wrap        → wrapper de una fila de juego (incluye split panel)
.gl-split-panel      → panel D/Z expandible dentro de un juego
.result-pill         → pill W/L (30×30px, font 14px)
.pill-w              → fondo verde, texto blanco
.pill-l              → fondo rojo, texto blanco
.logo-outline-full   → filter drop-shadow blanco 70% (logos oscuros)
.logo-outline-soft   → filter drop-shadow blanco 22% (logos con blanco)
[data-stat-tip]      → cursor:help, dispara showStatTip en mouseenter
```

---

## IDs HTML Clave

```
#roster-area         → contenedor tabla pitchers
#bat-roster-area     → contenedor tabla bateadores
#player-search       → input búsqueda pitchers
#bat-player-search   → input búsqueda bateadores
#pitcher-team-select → dropdown equipo pitchers
#bat-team-select     → dropdown equipo bateadores
#vs-pitcher-search   → input autocomplete pitcher (Versus)
#vs-pitcher-results  → dropdown resultados autocomplete
#vs-opp-team         → select equipo rival (Versus)
#vs-result           → div donde se renderiza la tabla VS
#global-tip          → div flotante para tooltips (pills y stats)
#standings-area      → contenedor standings
```

---

## Features Implementados — Estado Actual

### Equipos
- Standings por división, todas las divisiones en una sola vista
- Pills W/L: 30×30px, blancas, 14px, con tooltip flotante (logo, score, fecha, resultado)
- Logo de cada equipo con outline blanco siguiendo el contorno del SVG
- Fuente tabla: 14px datos, 15px nombre equipo, 12px headers

### Bateadores
- Dropdown: TODOS — MLB o por equipo
- Buscador en tiempo real por nombre
- Sin botones de split (eliminados de la UI; `currentBatSplit` siempre 'general')
- Sin columna POS
- Mano: Z / D / A
- Gamelog: últimos 15 juegos, click en fila → desglose D/Z por play-by-play
  - Si no hubo AB vs un lado → esa fila no aparece
- Tooltips en headers: AVG, OBP, SLG, OPS

### Pitchers
- Dropdown: TODOS — MLB o por equipo
- Buscador en tiempo real por nombre
- Sin columna POS ni QS
- Mano: Z / D
- Gamelog: últimos 15 juegos (sin score, sin colores)
- Tooltips en headers: ERA, WHIP, AVG, OBP, OPS

### Versus
- Autocomplete pitcher (busca en lista global)
- Select equipo rival (30 equipos)
- Sin previews superiores (eliminados)
- Preview: foto pitcher circular VS logo equipo grande + nombre
- Tabla: Bateador · Mano · AB · H · HR · BB · SO · AVG · OPS
  - Columnas POS y OBP eliminadas
  - Sin historial → atenuados (opacity .45) al final, ordenados por AB
- Logos con el mismo sistema de outline que standings

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
| 1 | **Desglose D/Z en gamelog de Pitchers** | Mismo feature que bateadores. Filtrar `matchup.pitcher.id` en play-by-play |
| 2 | **Mejoras VS** | Historial de AB individuales por bateador |
