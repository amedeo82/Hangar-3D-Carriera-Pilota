# 🚁 Hangar 3D — Carriera Pilota

Stato dell'arte: **un unico file HTML autonomo** (Three.js r160 via CDN, nessun build step) che contiene un simulatore-gioco di elicottero passeggeri guidabile da tastiera, con modelli .glb reali scaricati in tempo reale, fisica aerodinamica didattica, missioni con punteggio e mondo ambientale completo.

---

## 1. Pipeline modelli 3D (.glb)

- **Download live con fallback a cascata**: il gioco prova in ordine Quaternius *Helicopter* (CDN market.pmnd.rs, stile videogioco) → `helicopter.glb` da GitHub (srcejon/sdrangel-3d-models) → **H135 procedurale costruito in codice** (fusoliera estrusa da profilo bezier, cupolino in vetro, fenestron a 9 pale, 4 pale con profilo alare, swashplate, pattini, porte incernierate, luci di navigazione, beacon).
- **Normalizzazione automatica** di scala, appoggio al suolo e **prua** (il naso viene allineato a −X usando la posizione del rotore rispetto al baricentro: evita comandi invertiti su qualsiasi file).
- **Rilevamento automatico dei rotori** (per nome nodo o per geometria "sottile e larga", con risalita al pivot che raggruppa le pale) per agganciare rotazione, dischi di sfocatura e ombre.
- **Clip animate incorporate** nei .glb gestite con `AnimationMixer` la cui velocità è legata al regime motore (`timeScale = rpm`): motore spento = animazioni ferme.
- Supporto DRACO, ombre proiettate, materiali PBR con environment map.

## 2. Modello di volo e aerodinamica (Fase B)

- **Canale verticale fisico**: portanza = f(regime², collettivo) vs peso, con integrazione della velocità verticale (V/S reale, niente quote target).
- **Effetto suolo (IGE)**: +22% portanza sotto ~4,5 m; uscita dal cuscino d'aria con affondamento didattico.
- **Translational lift (ETL)**: +16% portanza sopra ~5 m/s.
- **Vortex Ring State (VRS)**: affondata con perdita di portanza fino al 60% se scendi rapido a bassa velocità, con vibrazione forte, allarme e recupero reale (guadagnare velocità).
- **Stallo della pala retrocedente** oltre ~18 m/s con vibrazioni e perdita di portanza.
- **HOLD QUOTA** (SPACE): autopilota di quota con trim automatico a autorità limitata e discesa automatica rate-limitata (~1,3 m/s) per touchdown morbidi; in VRS profondo non compensa (resta la tua emergenza).
- **Volo manuale** completo: ciclico longitudinale/laterale con assetto concorde (naso giù in accelerazione, rollio in deriva), pedali con imbardata smorzata, collettivo, auto-centraggio del ciclico per stabilità assistita.
- **Regime motore**: spool-up/spool-down esponenziali, blur disc dei rotori ad alto regime, **autorotazione** se spegni il motore in volo.
- **Ombra rotore solo vicino al suolo** (<1,5 m), disattivata in volo.
- **Vibrazioni con firma per stato** (regime base, VRS, stallo) su camera e fusoliera.
- Riga HUD aerodinamica live: `IGE ● / ETL ● / VRS ⚠ / STALL ⚠ / COLL REQ % / SET quota`.

## 3. Comandi (tastiera + mouse + UI)

| Input | Funzione |
|---|---|
| `W/S` o `↑/↓` | Ciclico longitudinale (avanti/indietro) |
| `A/D` o `←/→` | Ciclico laterale (sinistra/destra) |
| `Q/E` | Pedali (imbardata sinistra/destra) |
| `R/F` | Collettivo (manual) o quota set (in HOLD) |
| `SPAZIO` | Decollo / HOLD QUOTA on-off |
| `M` | Avvio/spegnimento motore |
| `P` | Porte |
| `C` | Cycle camera (4 preset, 4ª = inseguimento) |
| `U` | Audio sintetico on/off |
| `1/2/3` | Preset giorno / tramonto / notte |
| `O` | Ciclo giorno/notte automatico on/off |
| `ESC` | Menu |
| Mouse | Orbita, zoom, pan; slider collettivo/ciclici |

## 4. Sistema gioco (Fase A)

- **Menu carriera** con 3 missioni progressive + **volo libero**; best score per missione in `localStorage`.
  1. *Primo volo* — circuito di 3 anelli + atterraggio su H.
  2. *Rotta panoramica* — 4 anelli alti sopra la valle.
  3. *LZ Bravo — precisione* — trasferimento con atterraggio centrato e V/S < 1 m/s su secondo eliporto dedicato.
- **Anelli-checkpoint sequenziali** con beam luminoso, styling attivo/passato/futuro, bonus precisione (100 + fino a 50 pt).
- **Atterraggio valutato**: bonus centro LZ (offset), bonus touchdown morbido, penalità contatto duro; rilevamento V/S al contatto.
- **Punteggio** = anelli + atterraggio + bonus tempo sul "par"; **stelle 1-3** e schermata risultati con dettaglio punti, Riprova/Prossima/Menu.
- **HUD di missione**: minimappa (eliporti, pista, anelli, velivolo orientato, nord), **bussola a nastro** con punti cardinali italiani, **freccia-guida 3D** sul muso, riga obiettivo con distanza, timer, telemetria (RPM, quota, V/S, IAS, prua, stato).
- **Radio TORRE/ISTRUTTORE/ALLARME**: messaggi contestuali (briefing, anelli, rifiuti di decollo, coaching aerodinamico una-tantum, esiti) con fade automatico.

## 5. Ambiente e atmosfera

- **Terreno procedurale** collinare con vertex colors, zone spianate per eliporti e corridoio pista.
- **Due eliporti** (H e LZ Bravo) con texture "H", pali faro; **pista in asfalto** con segnaletica e luci di bordo; taxiway; **hangar** con tetto curvo; barili carburante; **manica a vento animata** che reagisce anche al downwash quando le passi vicino basso.
- **Ciclo giorno/notte completo**: sole che percorre il cielo (giorno in 10 min), cielo shader notte→tramonto→giorno, nebbia coordinata, **stelle e luna**, luce lunare fredda, luci artificiali che si accendono al buio, orologio e stato cielo in HUD.
- **Catena montuosa** con cime innevate, **150 alberi istanziati** (2 draw call), **stormo di uccelli** con flap, **nuvole sprite** alla deriva.
- **Rotor wash / downwash**: sistema particellare GPU (shader custom con alpha e size per particella) che solleva polvere radiale sotto ~8 m.
- Ombre PCF soft, tone mapping ACES, environment PBR.

## 6. Audio

- **Motore sintetico WebAudio**: noise browniano filtrato + saw, chop modulato alla frequenza pale, legato a RPM; si "sporca" in VRS. Avviabile solo su gesto utente (policy browser).

## 7. Robustezza e UX

- Pannello log con esito di ogni download e delle normalizzazioni applicate.
- Gestione graceful di rete/CORS assenti (fallback procedurale).
- Prevenzione scroll su Space/frecce, blur automatico degli slider alla pressione tasti, resize handling, camera shake compatibile con OrbitControls.

---

## Riepilogo in una frase per area

| Area | Stato |
|---|---|
| Modelli .glb + fallback procedurale | ✅ Completo |
| Fisica elicotteristica didattica (IGE/ETL/VRS/stallo) | ✅ Completo |
| Comandi tastiera/mouse + camera chase | ✅ Completo |
| Missioni, punteggio, stelle, salvataggi | ✅ Completo |
| HUD, minimappa, bussola, radio | ✅ Completo |
| Giorno/notte, meteo base, paesaggio vivo | ✅ Completo |
| Rotor wash polveroso + vibrazioni | ✅ Completo |
| Verricello/gancio, HEMS, antincendio, meteo avanzato, vegetazione animata | ⏳ Fase C disponibile |

Se vuoi, il prossimo passo naturale è la **Fase C**: gancio baricentrico/verricello con carichi esterni e missioni HEMS o benna antincendio, erba/chiome animate dal vento e meteo dinamico (pioggia/neve/nebbia). Dimmi solo "procedi con la fase C" — oppure chiedimi un file unico aggiornato, una variante di bilanciamento, o documentazione tecnica separata.
