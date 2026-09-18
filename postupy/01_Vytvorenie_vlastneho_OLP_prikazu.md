# Postup 01 — Vytvorenie vlastného OLP príkazu od nuly

**Cieľ:** pridať do Teach Pendantu príkaz, ktorý štandardný radič nepozná
(napr. `MyPaintGunCommand 2, TRUE;`).

**Kam súbor patrí:**
```
<PS installation>\eMPower\Robotics\Olp\<controller>\OLPConfiguration\*.xml
```
Názov súboru je ľubovoľný, načítajú sa všetky.

---

## Kroky

### 1. Vytvor kostru súboru

```xml
<RobotController Name="Abb-Rapid">
  <RoboticParams>
  </RoboticParams>
  <OlpCommands>
  </OlpCommands>
  <OlpDialogs>
  </OlpDialogs>
</RobotController>
```

`Name` = názov radiča. Voliteľne `Version="3.2.s4c, 5.07.01.irc5"` — bez neho platí
pre všetky verzie.

### 2. Definuj parametre v `<RoboticParams>`

```xml
<Param Name="GunNum" ValueType="int" MaxVal="4" MinVal="1" Default="1"/>
<Param Name="Status" ValueType="string" Default="OPEN">
  <ComboDef>
    <ElmDef>OPEN</ElmDef>
    <ElmDef>CLOSE</ElmDef>
  </ComboDef>
</Param>
```

Názov každého parametra **musí byť jedinečný naprieč všetkými XML** daného
radiča a verzie. Podrobne o typoch → postup 02.

### 3. Definuj príkaz v `<OlpCommands>`

```xml
<Command Name="MyPaintGun">
  <RoboticParamRef>
    <Param>GunNum</Param>
    <Param Optional="true">Status</Param>
  </RoboticParamRef>
  <Layers>
    <UILayer>
      <Line>
        <Item Type="const">Paint gun</Item>
        <Item Type="parameter">GunNum</Item>
        <Item Type="const">=</Item>
        <Item Type="parameter">Status</Item>
      </Line>
    </UILayer>
    <SimulationLayer>
    </SimulationLayer>
    <DownloadLayer>
      <Line>
        <Item Type="const">MyPaintGunCommand</Item>
        <Item Type="parameter">GunNum</Item>
        <Item Type="const">,</Item>
        <Item Type="parameter">Status</Item>
        <Item Type="const">;</Item>
      </Line>
    </DownloadLayer>
  </Layers>
</Command>
```

Tri vrstvy:
- **UILayer** — čo vidí človek. **Iba jeden riadok.**
- **SimulationLayer** — čo sa vykoná pri simulácii (→ postup 08).
- **DownloadLayer** — čo sa zapíše do programu robota.

### 4. Definuj dialóg v `<OlpDialogs>`

```xml
<Dialog Title="Customized|Paint|Paint Gun"
        Description="Vyber číslo pištole a stav"
        Icon="gripperOp.ico">
  <OlpCommandRef>
    <OlpCommand>MyPaintGun</OlpCommand>
  </OlpCommandRef>
</Dialog>
```

Znak `|` vytvára podmenu. Podrobne → postup 03.

### 5. Skontroluj a otestuj

Spusti **Customized Commands XML Checker** (ikona v PS/RobotExpert) → `Check`.
Potom otvor Teach Pendant a príkaz pridaj cez tlačidlo **Add**.

---

## Typy položiek `<Item>`

| Type | Čo vypíše |
|---|---|
| `const` | Pevný text. `xml:space="preserve"` zachová medzery. |
| `parameter` | Hodnotu parametra z `RoboticParams` |
| `dynamicParameter` | Hodnotu načítanú z radiča za behu (napr. `Interp`, `Speed`) |
| `expression` | Výsledok výpočtu, napr. `<![CDATA['IntNum' * 60]]>` |
| `dataName` | Názov dátového typu z DataConfiguration (→ postup 10) |

Podmienené vypísanie:
```xml
<Item Type="const" Conditional="true" CondParam="Robot">The Robot is:</Item>
```
`true` = vypíš len ak parameter má hodnotu, `false` = len ak nemá.
`CondParam` musí ukazovať na parameter označený `Optional="true"`.

Formátovanie čísel:
- `ConvertToInt="true"` — použi celočíselnú hodnotu z double
- `DigitsAfterPoint="2"` — zaokrúhli
- `UseFixedDecimalDigits="true"` — doplň nuly (5 → 5.00), keď to radič vyžaduje

---

## Na čo si dať pozor

- **Po nasadení už nemeň `Name` príkazu ani `Title` dialógu** — stratí sa väzba
  na už vytvorené príkazy v databáze. Ak musíš, použi atribút `Id` (→ postup 03).
- Download vrstvu navrhuj tak, aby sa dala načítať späť (→ postup 09).
  Základné pravidlo: **za reťazcovým parametrom musí byť konštanta,
  alebo musí byť posledný v riadku.**
- Povinné parametre majú v dialógu hviezdičku a blokujú OK.
  `<Param Optional="true">…</Param>` to zruší.
