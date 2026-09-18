# Postup 07 — Vlastný pohybový príkaz (MotionConfiguration)

**Cieľ:** spojiť pohyb do bodu s procesom (zvar, lepenie, náter) tak, aby sa
v Path Editore dal nastaviť jedným dialógom a zapísal sa v presnej syntaxi radiča.

**Rozdiel oproti OLP príkazu:** OLP príkaz sa pridáva *k* lokácii.
Pohybový príkaz *je* samotný pohyb do lokácie spolu s procesom.

**Kam súbor patrí:**
```
<PS installation>\eMPower\Robotics\Olp\<controller>\MotionConfiguration\*.xml
```

---

## Ako to funguje

Konfigurácia je organizovaná dvojicou **typ lokácie × typ procesu**:

- **Typ lokácie:** `VIA`, `WELD`, `Single Seam`, `Seam Start`, `Seam Middle`,
  `Seam End`, `Pick`, `Place`
- **Typ procesu:** čokoľvek si nadefinuješ (`Spot Off`, `Glue on`, `Polishing`…)

Každá kombinácia má vlastný dialóg a môže mať vlastnú download vrstvu podľa
typu pohybu.

V Path Editore sa pridá stĺpec **Customized Motion**. Kliknutie na bunku otvorí
dialóg podľa kombinácie lokácia + proces. Po OK sa robotické parametre zapíšu
na lokáciu (ak neexistujú, vytvoria sa) a do bunky sa vypíše reprezentácia
podľa `UILayer`.

---

## Kostra súboru

```xml
<RobotController Name="Kuka-Krc">
<RoboticParams>
</RoboticParams>

<Location Type="VIA">
  <Process Type="Spot Off">

    <Force>
      <!-- automaticky vynútené hodnoty po výbere tohto procesu -->
    </Force>

    <Dialog Title="Kuka-Vkrc - VIA Spot Customization"
            Description="Please customize the location">
      <RoboticParamRef>
      </RoboticParamRef>
    </Dialog>

    <UILayer>
    </UILayer>

    <SimulationLayer>
    </SimulationLayer>

    <Motion Type="1">
      <DownloadLayer>
      </DownloadLayer>
    </Motion>

  </Process>
</Location>
</RobotController>
```

---

## `<Force>` — automatické vynútenie hodnôt

Keď užívateľ vyberie typ procesu, tieto hodnoty sa nastavia samé:

```xml
<Force>
  <!-- bežné parametre -->
  <Param Name="TypID" Type="int" NewValue="11"/>
  <Param Name="Part" Type="double" NewValue="0.0"/>
  <Param Name="Leave" Type="string" NewValue="CTRL"/>
  <!-- dynamické parametre -->
  <Param Name="Position Number" Dynamic="true" NewValue="3"/>
  <Param Name="Gun State" Dynamic="true" NewValue="Open"/>
  <Param Name="Weld Time" Dynamic="true" NewValue="5.5"/>
</Force>
```

`<Force>` sa dá umiestniť na úrovni `Process` (podľa typu procesu)
aj na úrovni `Motion` (podľa typu pohybu).

---

## `UILayer` a `DownloadLayer`

- `UILayer` — **iba jeden riadok** (ide do bunky Path Editora).
- `DownloadLayer` — musí obsahovať odkaz na dynamický parameter
  **`LocName`** alebo **`XDatName`** (pri kruhovom pohybe navyše
  `ViaLoc Name` alebo `Via XDatName`). Bez toho radič nevie, do ktorého bodu ide.

---

## Dynamické parametre v dialógu

```xml
<Dialog>
  <RoboticParamRef>
    <Param Dynamic="true">Gun State</Param>
    <Param Dynamic="true">Servo Value</Param>
    <Param Optional="true" Dynamic="true">Speed</Param>
    <Param Optional="true" Dynamic="true">Weld Time</Param>
  </RoboticParamRef>
</Dialog>
```

## Automatický výpočet ďalších dynamických parametrov

Užívateľ zadá jednu hodnotu, systém dopočíta ďalšie:

```xml
<Dialog Title="Move_JobReq">
  <RoboticParamRef>
    <Param>Job</Param>
    <Param Dynamic="true">Gun Position</Param>
  </RoboticParamRef>
  <UiAdditionalDynamicParameters>
    <AdditionalDynamicParam Name="Tool Data">
      <Item Type="const">t</Item>
      <Item Type="dynamicParameter" FormatNumber="3">Gun Position</Item>
      <Item Type="const">_onsert</Item>
    </AdditionalDynamicParam>
  </UiAdditionalDynamicParameters>
</Dialog>
```
Zadá `Gun Position = 130` → `Tool Data` sa nastaví na `t130_onsert`.

**S výpočtom:**
```xml
<AdditionalDynamicParam Name="Speed">
  <Item Type="expression"><![CDATA[('A15_Speed m/min'/60)]]></Item>
</AdditionalDynamicParam>
```

---

## Hodnota z úrovne operácie

```xml
<Item Type="parameter" Level="operation">RRS_ZONE_NAME</Item>
<Switch ParamName="RRS_ZONE_NAME" Level="operation">
```
Parameter na úrovni operácie **sa nedá nastaviť z UI** — len čítať.

## Hromadná úprava viacerých lokácií

Príkaz **Set Locations Properties** → nastav Process Type pre všetky lokácie
→ vyber riadok Customized motion → otvor dialóg. Nastavíš všetko naraz.

## Vytvorenie parametrov pri uploade

```xml
<UploadAdditionalDynamicParameters>
  <AdditionalDynamicParam Name="Loc Name">
    <Item Type="const">Reload</Item>
    <Item Type="parameter">A21_StudNo</Item>
    <Item Type="const">_</Item>
    <Item Type="parameter">A21_ReloadPos</Item>
  </AdditionalDynamicParam>
</UploadAdditionalDynamicParameters>
```
Podporuje `const`, `parameter`, `dynamicParameter`. **Switch nie.**
Pre bežné (nie dynamické) parametre existuje `<UploadAdditionalParameters>`
s rovnakou logikou.

## Typ MFG prvku

```xml
<Location Type="Seam Start"/>
<Process Type="A20_Stud_Reload" MfgType="ArcContinuousMfg"/>
```
Určuje, akú podtriedu `WeldPoint` / `ContinuousMfg` systém vytvorí
(`NutWeldPoint`, `StudWeldPoint`, `ArcContinuousMfg`, `GlueContinuousMfg`,
`PaintContinuousMfg`, `SealContinuousMfg`, `LaserWeldContinuousMfg` …).

## Predvolené hodnoty

Ak **všetky** parametre v `RoboticParamRef` dialógu majú `Default`, po výbere
typu procesu sa všetky tri vrstvy hneď zobrazia s predvolenými hodnotami
a po download+upload bude mať lokácia skutočné parametre.
