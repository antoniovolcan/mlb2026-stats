# MLB 2026 Stats — Contexto del Proyecto

## Información General

| Campo | Valor |
|---|---|
| **Proyecto** | Página de stats personales MLB 2026 |
| **Archivo** | `index.html` (single-file: HTML + CSS + JS) |
| **Stack** | HTML + CSS + Vanilla JS puro. Sin frameworks. |
| **Fuentes** | Bebas Neue, Azeret Mono, Sora (Google Fonts) |
| **API** | `statsapi.mlb.com` + `bdfed.stitch.mlbinfra.com` |
| **Repo** | `https://github.com/antoniovolcan/mlb2026-stats.git` |
| **Deploy** | Vercel (auto-deploy desde `main`) |
| **Git config** | Antonio Volcan / soyvolcom@gmail.com |
| **Archivo local** | `C:\Users\anton\OneDrive\Escritorio\mlb2026-stats\index.html` |
| **Regla de trabajo** | Todo cambio termina con `git add index.html && git commit && git push` |

---

## Estructura de la App

4 secciones navegables con header sticky:

| Tab | ID | Descripción |
|---|---|---|
| Equipos | `sec-equipos` | Tabla de posiciones (standings) con últimos 10 juegos y tooltip por juego |
| Bateadores | `sec-bateadores` | Roster de bateadores con stats inline + gamelog por jugador |
| Pitchers | `sec-pitchers` | Roster de pitchers con stats inline + gamelog por jugador |
| Versus | `sec-versus` | Pitcher vs equipo rival — tabla de todos los bateadores del equipo vs ese pitcher |

---

## Design System (CSS Variables)

```css
--bg:#0a0c0f   --bg2:#111318   --bg3:#181c22   --bg4:#1e2330
--border:#252b38   --border2:#2e3647
--text:#e8ecf4   --text2:#8892a4   --text3:#4a5568
--accent:#c8a84b   --accent2:#e8c96a
--red:#e05252   --green:#4caf82   --blue:#4a90d9
--radius:8px   --radius2:12px
```

**Fuente de números:** Azeret Mono (aplicada globalmente en todas las stats y tablas).

---

## Columnas de Stats

### Pitchers — `PITCH_INLINE` (13 columnas, sin POS ni QS)
```
IP · K · H · R · HR · ERA · WHIP · W · L · G · GS · AVG · OBP · OPS
```
Orden: descendente por IP.

### Bateadores — `BAT_INLINE` (11 columnas, sin PA ni POS)
```
AB · H · HR · SO · BB · R · RBI · AVG · OBP · SLG · OPS
```
Orden: descendente por AB.

---

## Features Implementados

### Equipos (Standings)
- Tabla por división con W, L, RS, RA, +/-, Casa, Visita, Últimos 10
- Pills W/L: 30×30px, color blanco, `font-size: 14px`
- Tooltip flotante al hover sobre cada pill: logo equipo, resultado, score, fecha
- Logo de cada equipo con outline blanco siguiendo el contorno del SVG:
  - **Outline suave** (22% opacidad): MIL (158), SD (135), CWS (145), CHC (112), PIT (134), PHI (143), LAA (108)
  - **Outline completo** (70% opacidad): los otros 23 equipos
- Fuente de tabla más grande: 14px datos, 15px nombre equipo, 12px headers

### Bateadores
- Dropdown para seleccionar equipo o cargar TODOS — MLB
- Buscador en tiempo real (filtra filas por nombre con `.bat-name`)
- Sin botones de split (General / vs Zurdos / vs Derechos — eliminados)
- Sin columna POS
- Mano del jugador: Z (zurdo) / D (derecho) / A (ambidiestro)
- Click en jugador → gamelog últimos 15 juegos
- Click en un juego del gamelog → desglose por mano del pitcher (D / Z). Solo muestra el lado en que hubo turnos al bate; si no bateó vs uno de los dos, esa fila no aparece
- Tooltips en headers AVG, OBP, SLG, OPS (tooltip JS usando `global-tip` flotante)

### Pitchers
- Dropdown para seleccionar equipo o cargar TODOS — MLB
- Buscador en tiempo real (filtra filas por nombre con `.bat-name`)
- Sin columna POS ni QS
- Mano del pitcher: Z / D
- Click en pitcher → gamelog últimos 15 juegos
- Tooltips en headers ERA, WHIP, AVG, OBP, OPS

### Versus
- Buscador de pitcher con autocomplete (dropdown de hasta 12 resultados)
- Dropdown de equipo rival con los 30 equipos
- Sin previews superiores (eliminados) — solo los dos inputs
- Preview combinado: foto circular del pitcher VS logo del equipo, con nombre y equipo
- Tabla de hasta 25 bateadores del equipo rival ordenados por AB vs ese pitcher
- Columnas: Bateador · Mano · AB · H · HR · BB · SO · AVG · OPS
- Bateadores sin historial → atenuados al final
- Logos con mismo sistema de outline que standings

### Global
- Outline en logos: función `logoOutlineClass(teamId)` devuelve `logo-outline-full` o `logo-outline-soft`
- Tooltips JS: `showStatTip(event, text)` / `hideStatTip()` usan el div `#global-tip`
- Tooltips de juego: `showGameTip(e, pill)` / `hideGameTip()` para pills de standings
- Score del gamelog (bateadores y pitchers): eliminado completamente

---

## Arquitectura JS — Variables y Funciones Clave

### Caché y Estado
```js
rosterCache       // { 'all': [...], teamId: [...] } — pitchers
batRosterCache    // { 'all': [...], teamId: [...] } — bateadores
batStatsCache     // { pid_split: data }
gameLogCache      // { pid: games[] }
gameSplitCache    // { splitId: { R:{ab,h,hr,bb,so,rbi,avg}, L:{...} } }
openGameLogs      // { pid: bool }
openGameSplits    // { splitId: bool }
currentTeamId     // 'all' | teamId (number) — pitchers
currentBatTeamId  // 'all' | teamId (number) — bateadores
currentBatSplit   // 'general' | 'vsLeft' | 'vsRight' (bateadores, sin UI visible)
```

### Funciones Importantes
```js
selectAllPitchers()          // carga todos los pitchers MLB (sortStat=inningsPitched)
selectTeam(id)               // carga pitchers de un equipo
selectAllBatters()           // carga todos los bateadores MLB (sortStat=atBats)
selectBatTeam(id)            // carga bateadores de un equipo
renderPitchRoster(roster)    // renderiza tabla de pitchers
renderBatRoster(roster)      // renderiza tabla de bateadores
pitcherHTML(p, rank)         // HTML de una fila de pitcher
batterHTML(p, rank)          // HTML de una fila de bateador
toggleGameLog(pid, name)     // abre/cierra gamelog de bateador
togglePitcherLog(pid, name)  // abre/cierra gamelog de pitcher
toggleGameSplit(splitId, batterId, gamePk) // expande desglose D/Z por juego
filterPitchers(q)            // filtra filas de pitchers en tiempo real
filterBatters(q)             // filtra filas de bateadores en tiempo real
logoOutlineClass(teamId)     // devuelve clase CSS de outline según equipo
showStatTip(e, text)         // tooltip flotante para headers de stats
vsEnsurePitchers()           // carga lista global de pitchers para VS
vsPitcherSearch(q)           // filtra autocomplete de pitcher en VS
vsSelectPitcher(id, name, teamId) // selecciona pitcher en VS
vsOnOppTeamSelect()          // selecciona equipo rival en VS
vsLoadPitcherVsTeam(pitId, oppTeamId) // fetch historial pitcher vs equipo
renderVsTeamCard(pit, oppTeam, oppTeamId, results) // renderiza resultado VS
buildLast10(teamId, teamGames) // genera pills W/L de últimos 10 juegos
```

---

## Logos de Equipos

Se sirven desde `https://www.mlbstatic.com/team-logos/{teamId}.svg`

### Equipos con outline suave (logos claros/blancos)
| Equipo | ID |
|---|---|
| Milwaukee Brewers | 158 |
| San Diego Padres | 135 |
| Chicago White Sox | 145 |
| Chicago Cubs | 112 |
| Pittsburgh Pirates | 134 |
| Philadelphia Phillies | 143 |
| LA Angels | 108 |

---

## Archivos en el Repo

```
mlb2026-stats/
├── index.html      ← único archivo fuente (HTML + CSS + JS)
├── CONTEXT.md      ← este archivo
├── vercel.json     ← { "cleanUrls": true, "trailingSlash": false }
└── .gitignore      ← .DS_Store, Thumbs.db, .vercel
```

---

## Tooltips de Stats — Descripciones

### Bateadores
| Stat | Descripción |
|---|---|
| AVG | Promedio de bateo. Más alto = mejor contacto. Élite: .300+ |
| OBP | Qué tan seguido llega a base. Más alto = mejor. Élite: .380+ |
| SLG | Potencia del bateador. Más alto = más poder. Élite: .500+ |
| OPS | OBP + SLG combinados. Más alto = mejor bateador integral. Élite: .900+ |

### Pitchers
| Stat | Descripción |
|---|---|
| ERA | Carreras limpias por 9 entradas. Más bajo = mejor. Élite: menos de 3.00 |
| WHIP | Hits + BB permitidos por entrada. Más bajo = mejor control. Élite: menos de 1.00 |
| AVG | Promedio de bateo en contra. Más bajo = mejor para el pitcher. Élite: .220 o menos |
| OBP | Frecuencia en base del bateador vs este pitcher. Más bajo = mejor. Élite: .280 o menos |
| OPS | OBP + SLG del bateador vs este pitcher. Más bajo = mejor. Élite: .650 o menos |

---

## Pendientes / Ideas

| # | Pendiente |
|---|---|
| 1 | Desglose por mano en gamelog de **Pitchers** (misma lógica que bateadores, filtrar `matchup.pitcher.id`) |
| 2 | Mejoras VS: historial de AB individuales |
