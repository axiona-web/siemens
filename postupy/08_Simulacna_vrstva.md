# Postup 08 — Simulačná vrstva

**Cieľ:** aby sa vlastný príkaz nielen zapísal do programu, ale aj korektne
odsimuloval — pohyby, signály, čakania, časy cyklu.

Keď radič počas simulácie narazí na vlastný príkaz, **nahradí ho jeho simulačnou
vrstvou** a interpretuje tú.

---

## Čo sa dá do simulačnej vrstvy napísať

Jeden príkaz na `<Line>`. Voľne sa dá miešať:

1. **Príkazy „default controllera“** — rozumejú im všetky radiče:
   `# SendSignal sig1`, `# WaitSignal …`, `# WaitTime 2`, `# Display …`,
   `# Blank …`, `# Weld`, `# GunToState` …
2. **Natívna syntax radiča** — `IF var1 = 3 THEN`, volanie makier, volanie
   procedúry modulu robota… (čo presne, nájdeš v manuáli konkrétneho radiča)

---

## Štyri príkazy riadenia pohybu ⚠

Nie sú dostupné z Teach Pendantu, ale fungujú vo všetkých simuláciách:

| Príkaz | Čo robí |
|---|---|
| `# StartMove` | Spustí pohyb do lokácie a **hneď** pokračuje ďalším príkazom (paralelne s pohybom) |
| `# WaitReached` | Počká na dokončenie pohybu. **Pozor:** nie je to potvrdenie príchodu do bodu — znamená to, že RCS vrátil stav 2 (posledný krok, rýchlosť 0) alebo stav 1 (treba viac dát). Pri flyby môže byť robot ešte kus od bodu. |
| `# Move` | `StartMove` + `WaitReached` |
| `# ForceFullArrival` | Vynúti zónu **fine** a úplný príchod do cieľa. Zaručí, že ďalšie príkazy sa vykonajú naozaj v bode. **Mení trajektóriu aj celkový čas cyklu.** |

**Ak neuvedieš nič,** správa sa to, akoby prvý príkaz bol `# Move` —
t. j. všetko sa vykoná až po pohybe.

---

## Zvarová lokácia — dôležitý rozdiel ⚠

- **Štandardná** zvarová lokácia: simulácia si `# Weld` a `# GunToState`
  doplní sama, kliešte sa vždy zatvoria a otvoria.
- **Vlastná** (customized) zvarová lokácia: **nič sa nedopĺňa.**
  Ak chceš vidieť pohyb klieští, musíš `# Weld` do simulačnej vrstvy napísať.

Dôvod: niektorí užívatelia chcú zváranie simulovať posielaním signálov
inteligentným zariadeniam, nie pohybom klieští.

---

## Príklad 1: jednoduchá logika (striekacia pištoľ)

```xml
<SimulationLayer>
  <Line>
    <Item Type="const">IF</Item>
    <Item Type="parameter">Status</Item>
    <Item Type="const">THEN</Item>
  </Line>
  <Line>
    <Item Type="const"># Display ${Robot}gun</Item>
    <Item Type="parameter">GunNum</Item>
  </Line>
  <Line>
    <Item Type="const">ELSE</Item>
  </Line>
  <Line>
    <Item Type="const"># Blank ${Robot}gun</Item>
    <Item Type="parameter">GunNum</Item>
  </Line>
  <Line>
    <Item Type="const">ENDIF</Item>
  </Line>
</SimulationLayer>
```

`${Robot}` sa nahradí názvom robota. Pri pomenovaní komponentov `r1_gun1`,
`r1_gun2` sa tak automaticky vyberie správny vejár.

---

## Príklad 2: servo kliešte cez signály PLC

Scenár: pneumaticko-servo kliešte s mnohými stavmi otvorenia (nie len
OPEN/CLOSE/SEMI). Zariadenie dostáva a posiela cez PLC:

- `TargetPoseGun<GunNum>` [robot → PLC → zariadenie] — cieľové otvorenie v mm
- `ActualPoseGun<GunNum>` [zariadenie → PLC → robot] — skutočné otvorenie v mm

Postupnosť: príď do bodu → zavri kliešte → počkaj na zavretie →
odčakaj zvárací čas → otvor na `GunPose` → počkaj na dosiahnutie.

```xml
<SimulationLayer>
  <Line><Item Type="const"># Move</Item></Line>
  <Line><Item Type="const"># ForceFullArrival</Item></Line>

  <Line>
    <Item Type="const"># SendSignal TargetPoseGun</Item>
    <Item Type="parameter">GunNum</Item>
    <Item Type="const">0</Item>
  </Line>
  <Line>
    <Item Type="const"># Wait Signal ActualPoseGun</Item>
    <Item Type="parameter">GunNum</Item>
    <Item Type="const">0</Item>
  </Line>

  <Line>
    <Item Type="const"># WaitTime</Item>
    <Item Type="dynamicParameter">Weld Time</Item>
  </Line>

  <Line>
    <Item Type="const"># SendSignal TargetPoseGun</Item>
    <Item Type="parameter">GunNum</Item>
    <Item Type="const"> </Item>
    <Item Type="parameter">GunPose</Item>
  </Line>
  <Line>
    <Item Type="const"># WaitSignal TargetPoseGun</Item>
    <Item Type="parameter">GunNum</Item>
    <Item Type="const"> </Item>
    <Item Type="parameter">GunPose</Item>
  </Line>
</SimulationLayer>
```

---

## Overenie

V Teach Pendante pravý klik na príkaz → **Show Layers**.
Uvidíš vedľa seba UI, simulačnú aj download podobu (→ postup 14).
