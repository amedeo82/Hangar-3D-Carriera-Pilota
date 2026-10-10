# Hangar 3D — Simulatore-gioco di elicottero

Un simulatore di elicottero passeggeri **completamente in un singolo file HTML**, scritto in Three.js r160 via CDN. Nessun build step, nessuna installazione: si apre `index.html` in un browser moderno e si vola.

> Stato: **Fase D17** — sei missioni, gamepad, traffico taxiway, ombra in volo, LOD, lago riflettente, fanali veicoli, erba reattiva al downwash, beacon hangar, danni atterraggio, HUD vibrante in VRS, e molto altro.

---

## Indice

1. [Avvio rapido](#avvio-rapido)
2. [Screenshot](#screenshot)
3. [Controlli](#controlli)
4. [Mondo di gioco](#mondo-di-gioco)
5. [Missioni](#missioni)
6. [Fisica di volo](#fisica-di-volo)
7. [Struttura del codice](#struttura-del-codice)
8. [Roadmap delle fasi](#roadmap-delle-fasi)
9. [Cosa resta da fare](#cosa resta-da-fare)

---

## Avvio rapido

```bash
# server statico (consigliato per la cache dei modelli GLB)
python3 -m http.server 8123
# poi apri:  http://localhost:8123/index.html
```

In alternativa, doppio click su `index.html` — funziona ma il browser potrebbe bloccare i fetch CORS verso GitHub per i modelli.

**Requisiti:** browser con WebGL 2 (Chrome/Edge/Firefox/Safari recenti), Three.js r160 caricato da CDN (jsDelivr). Gamepad opzionale.

---

## Screenshot

### Menu missioni (6 missioni, progressione 2★)

![Menu missioni](docs/images/01_menu_missioni.png)

*Missioni 4 e 5 sono bloccate finché non si totalizzano 2 stelle nella missione 3. La missione 6 "Tour aeroporto" è libera.*

### Vista del campo (eliporto H, hangar, pista)

![Campo 3D](docs/images/02_campo_3d.png)

*Elicottero centrato sull'H, prua allineata all'asse, manica a vento, siepi, veicoli e pista "27" con designatore.*

### Hangar con antenne e faro rosso lampeggiante

![Hangar con antenne](docs/images/03_hangar_antenne.png)

*Dettaglio del tetto del hangar procedurale: due antenne, cella ripetitore e faro rosso che lampeggia a ~1 Hz.*

### Notte + pioggia

![Notte pioggia](docs/images/04_notte_pioggia.png)

*Cielo stellato, lampioni della pista, fanali del taxi car attivi, terreno "bagnato" con specular, HUD con indicatore allarmi.*

### Pista con lampioni

![Pista e lampioni](docs/images/05_pista_lampioni.png)

*Il runway con segnaletica "27", soglia di touchdown, e due file di lampioni che si accendono di notte.*

---

## Controlli

### Tastiera

| Tasto | Azione |
|---|---|
| `M` | Avvia / spegni motore |
| `Spazio` | HOLD quota / decollo automatico |
| `R` / `F` | Collettivo su / giù (in HOLD: setpoint quota ± 6 m/s) |
| `W` / `S` | Ciclico longitudinale (avanti / indietro) |
| `A` / `D` | Ciclico laterale (sinistra / destra) |
| `Q` / `E` | Pedali (yaw sinistra / destra) |
| `C` | Cicla camere (5 preset, 4 = cockpit) |
| `P` | Porte / sportello |
| `G` | Aggancia / sgancia carico (verricello) |
| `H` | Sgancia acqua (benna antincendio) |
| `V` / `B` | Verricello estendi / ritira |
| `1` / `2` / `3` | Ora del giorno: giorno / tramonto / notte |
| `O` | Toggle ciclo automatico giorno/notte |
| `J` | Cicla meteo (sereno, vento, pioggia, neve) |
| `U` | Audio on/off |
| `ESC` | Torna al menu |

### Gamepad

| Input | Mappato a |
|---|---|
| Stick sinistro Y | Collettivo (in HOLD: setpoint) |
| Stick destro X/Y | Ciclco longitudinale / laterale |
| Trigger LT/RT (6,7) | Pedali Q / E |
| A (0) | Avvia motore |
| B (1) | HOLD quota |
| X (2) | Porte |
| Y (3) | Camera |
| LB (4) | Audio |
| Start (9) | Menu |

### Mouse (OrbitControls)

- Trascina per orbitare la camera
- Scroll per zoom
- Click su menu/pannelli

---

## Mondo di gioco

Un eliporto alpino con:

- **Eliporto H** centrale (0, 0) — H texture 1024 px, 4 fari, palo manica a vento a 14 m
- **LZ Bravo** secondario a (60, −40) — usato come bersaglio nelle missioni cargo/HEMS
- **Hangar procedurale** (26, −22) — box + cupola cilindrica, antenne e faro rosso lampeggiante
- **Officine Kenney** (46, 24) — edificio car kit, ruotato keepOut, con tacca gialla sulla mappa
- **Pista** orientata a 270° (27), lunghezza 92 m, larghezza 12, designatore + soglia touchdown + 16 lampioni
- **Apron** 70×28, eliporto circolare h=17, con veicoli parcheggiati
- **Lago** (−40, 30), 20 m raggio, acqua riflettente con normalMap animato
- **Fiume** dal lago verso SE, TubeGeometry riflettente
- **Rogo antincendio** (50, 40) — particles shader + PointLight pulsante, usato dalla missione 5
- **Vegetazione Kenney** (alberi, cespugli, fiori) con LOD far forest r 120..200
- **Stormo di uccelli** (procedurale o .glb Flamingo CC0) — animato in cielo con scie di condensa
- **Taxi car** che looppa su 4 waypoint, con fari anteriori che si accendono di notte
- **Nuvole** sprite-based che scorrono col vento
- **Sole, luna, stelle** posizionati in base all'ora del giorno
- **Overlay bagnato** che appare con la pioggia
- **Siepi** lungo i bordi della taxiway

### Atmosfera / ciclo giorno

- 4 preset: `sereno`, `vento`, `pioggia`, `neve` (ciclabili con `J`)
- Mix colore cielo in 3 stop: notte → tramonto → giorno
- Fog color che segue il cielo
- Sole e luna con sprite
- Stellato visibile di notte
- Auto-cycle 600s/giorno

---

## Missioni

| # | Nome | Tipo | Par | Note |
|---|---|---|---|---|
| 1 | Primo volo | rings (3) | 110 s | Sempre disponibile |
| 2 | Rotta panoramica | rings (4) | 140 s | Anelli sopra la valle |
| 3 | LZ Bravo — precisione | rings (2) + landing | 160 s | Gate per missioni 4–5 |
| 4 | Cargo HEMS — gancio 🔒 | cargo | 200 s | Richiede 2★ in missione 3 |
| 5 | Antincendio — benna 🔒 | fire | 260 s | Richiede 2★ in missione 3 |
| 6 | Tour aeroporto | tour (5) | 180 s | Waypoint panoramico sempre libero |

Volo libero disponibile dal menu per esplorazione libera.

### Punteggio

- Anello attraversato: 100 pt + 50 pt bonus precisione
- Atterraggio dolce (V/S < 1 m/s): +100 pt
- Atterraggio duro (V/S > 2.5 m/s): −100 pt, "game.hard"
- Tempo entro par: +2 pt/sec residuo
- Cargo: 50 pt aggancio, 200+150 pt consegna
- Antincendio: 50 pt rifornimento, 100 pt sgancio, 200 pt rogo spento

### Stelle

- ⭐ ≥ 40% del target
- ⭐⭐ ≥ 66%
- ⭐⭐⭐ = target pieno (o superato)

Le stelle sono **persistenti** in `localStorage` (chiavi `heli3d_stars_<id>`).

---

## Fisica di volo

Modello didattico a 6 gradi di libertà ridotti:

- **Lift** = `rpm² · (collettivo) · fIGE · fETL · (1 − fVRS) · (1 − fSTALL) − load`
- **V/S** (velocità verticale) integrata con damping `exp(-0.9 dt)`
- **Forze X/Z** da ciclico + yaw, smussate esponenzialmente
- **fIGE** (In Ground Effect) = 1 + 0.22·(1 − alt/9)² entro 9 m
- **fETL** (Translational Lift) = 1 + 0.16·smooth(IAS, 3, 10)
- **fVRS** (Vortex Ring State) = 0.6·smooth(−V/S, 1.5, 5)·(1−smooth(IAS,3,8))
- **fSTALL** = 0.25·smooth(IAS, 16, 26)

In **HOLD** mode (`Spazio`), un PID mantiene `alt ≈ setpoint` regolando il trim integrale.

Allarmi radio attivi:
- IGE lasciato: "serve più collettivo"
- ETL acquisito: "puoi ridurre il collettivo"
- VRS: "VORTEX RING STATE! Guadagna velocità (W)"
- Stallo: "Stallo pala retrocedente: rallenta"

---

## Struttura del codice

`index.html` (~1830 righe) è organizzato come:

| Sezione | Righe circa | Contenuto |
|---|---|---|
| `<head>` + CSS | 1–290 | Stili pannelli, hud, menu, minimappa, bussole, gauge |
| `<body>` HTML | 291–320 | Pannelli UI (topstrip, compass, dock, log, radio, menu, results) |
| Costanti (LAKE, FIRE, HANGAR, OFFICE) | 322–335 | Posizioni mondo |
| **Terreno + laghi + fiume + wet overlay** | 335–380 | Mesh `PlaneGeometry` con vertex colors + fiume + normalMap lago |
| Hangar procedurale (con antenne/faro) | 487–510 | Box + cupola + antenne + faro rosso |
| Manica a vento | 510–520 | Palo + sock con flutter aggiuntivo |
| Montagne / picchi innevati | 512–530 | 26 coni con cappelli di neve |
| Cespugli + erba + uccelli + fuoco + splash | 517–610 | Geometrie procedurali, particles shader, Nuvole |
| `patchWind` (shader vento + downwash uniforms) | 206–240 | Modifica onBeforeCompile per InstancedMesh alberi |
| Cielo, sole, luna, stelle, nubi | 800–950 | Sprite, shader cielo a 3 colori |
| `applyTod` (ciclo giorno/notte) | 1493–1510 | Aggiorna colore cielo, nebbia, hemi, sole, GLOW; set `__nightK` |
| Heliporti (con texture H) | 335–345 | CylinderGeometry con canvas texture |
| **Pista + segnaletica + lampioni** | 349–365 | PlaneGeometry con canvas "27" + threshold + 16 lampioni |
| **Apron + veicoli + siepi** | 357–370 | Plane + Kenney vehicles + hedges esterne |
| Missioni + best score + stars | 1282–1335 | Array `MISSIONS`, `renderMenu`, gating |
| `startMission` / `endMission` / `finish*` | 1351–1395 | Game loop missioni |
| Verricello + benna + hook | 1396–1422 | Fisica del carico e dell'acqua |
| Mini-mappa, bussola, gauge | 1424–1490 | Canvas 2D |
| Input handlers (keydown, gamepad) | 1545–1642 | Mappatura tasti + gamepad poll |
| **Miglioramenti D16** (heli shadow, taxi, LOD, tour) | 1555–1600 | |
| **Miglioramenti D17** (lago, fari, downwash, beacon, danni, HUD VRS) | vari | |
| `animate()` loop | 1644–1823 | Game loop: fisica, camera, render |

---

## Roadmap delle fasi

| Fase | Contenuto |
|---|---|
| **A** | Scheletro 3D: terreno, hangar, eliporto, elicottero procedurale, camera chase |
| **B** | Modello .glb reale, mixer animazioni, rotori con blur, fisica base |
| **C** | Missioni rings, cargo, fire; verricello; benna; minimappa; bussola; ciclici collettivi; HUD |
| **D1–D10** | Iterazioni bug-fix e miglioramenti visivi (illuminazione, nebbia, allarmi radio) |
| **D11** | Kenney: nature (alberi, cespugli), vehicles (car kit), offices |
| **D12** | Nature LOD + bird flock + sky shader + auto-cycle |
| **D13** | Controlli PresetSlider, OrbitControls, Damping 3D |
| **D14** | Hangar con cupola, OFFICE ruotato, runway centerline, keepOut hangar rettangolare |
| **D15** | Minacce / waypoint indicator, OFFICE map tacca gialla |
| **D16** | Progressione missioni 2★, gamepad, missione 6 "Tour", traffico taxi, heli shadow, LOD far forest |
| **D17** | Lago riflettente, fanali notturni veicoli, erba reattiva downwash, beacon/faro hangar, vetri riflettenti, siepi, fiume, scie uccelli, danni atterraggio morbido, vibrazione HUD VRS, runway "27" + touchdown |

---

## Cosa resta da fare

Proposte D17 non ancora implementate (in ordine di complessità crescente):

### XS
- ❌ Pneumatici dei veicoli sterzati in curva
- ❌ Specchietti retrovisori + tergicristalli sui veicoli Kenney
- ❌ Ombra "soft contact" sotto ogni veicolo statico
- ❌ LOD esteso a rocce/cespugli/fiori (solo alberi hanno LOD)

### S
- ❌ Più taxi cars (2–3) con frecce lampeggianti prima delle curve
- ❌ Porte del garage officine che si aprono vicino al player
- ❌ Camion cisterna al lago per missione antincendio
- ❌ Audio "blat" in stallo (miglioramento)
- ❌ Decal "impronta" su neve dopo atterraggio

### M
- ❌ Scioglimento neve bagnata su pista (decals canvas)
- ❌ Atmosfera Rayleigh (gradient `1/cos(θ)`)
- ❌ Polvere/cenere in volo vicino al rogo

### L
- ❌ Neve cumulativa su cime e tetti con displacement
- ❌ Ombra delle nuvole proiettata sul terreno
- ❌ Animali selvatici con AI path (caprioli, conigli)
- ❌ Sistema di danni completo (contatto duro → danni rotore, fumo persistente, fine missione)
- ❌ Missione "Taxi a terra" con guida waypoint di un veicolo di terra

---

## Test

I test automatici usano **Playwright + Chromium** con `--use-gl=swiftshader` per WebGL in headless. Verificano:

- Modulo JS carica senza errori
- Mission cards renderizzate (6, con 2 e 3 locked)
- HUD telemetria aggiornata
- Spool-up motore, IGE, decollo
- Pannelli senza overlap
- Scene contains: taxi car, heli shadow, hangar beacon, fari veicoli, hedges, fiume, wet overlay, water normalMap, LOD far trees
- 0 NaN, 0 errori non-risorsa (eccetto atteso 404 su `assets/helicopter.glb`)

---

## Licenza

Modelli `Kenney.nl` / `kenney.cc`: **CC0** (dominio pubblico).
Modello elicottero `srcejon/sdrangel-3d-models`: controllare licenza del repo.
Flamingo `three.js` examples: CC0.
Three.js: MIT.

---

*Sviluppato in single-file architecture per facilitare distribuzione e ispezione. Nessun bundler, nessuna dipendenza installata localmente.*
