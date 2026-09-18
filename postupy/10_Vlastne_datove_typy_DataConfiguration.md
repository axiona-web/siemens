# Postup 10 — Vlastné dátové typy (DataConfiguration)

**Cieľ:** riadiť názvy a obsah dátových deklarácií (FDAT, PDAT, LDAT, XDAT)
v programovom súbore robota — alebo vytvoriť vlastné dátové typy.

**Kam súbor patrí:**
```
<PS installation>\eMPower\Robotics\Olp\<controller>\DataConfiguration\*.xml
```
Priečinok sa vytvorí pri inštalácii radiča (ak ho radič podporuje).

---

## Prečo to existuje — na príklade KUKA

Program KUKA má dva súbory:
- `.SRC` — logika (inštrukcie)
- `.DAT` — definície dát

Sú prepojené. V `.DAT`:
```
DECL E6POS Xlo4 ={X 1086.01,Y -936,Z 2125.74,A 180,B 36.222,C 0,S 6,T 26}
```
a v `.SRC` sa `Xlo4` použije:
```
LIN Xlo4 C_DIS
LDAT_ACT= LLlo4
FDAT_ACT= Flo4
```

**MotionConfiguration XML vie prispôsobiť iba `.SRC`.** Názvy a hodnoty
dátových typov generuje download radiča automaticky z názvu lokácie.
DataConfiguration tento zámok otvára.

---

## Syntax

```xml
<RobotController Name="Kuka-Krc" Version="krc5.3_r01">
<RoboticParams>
  <Param Name="PL_ACC" ValueType="double" MaxVal="100.5" MinVal="0"/>
  <Param Name="T11" ValueType="int" MaxVal="100" MinVal="50"/>
</RoboticParams>

<DataDef>
  <Data Type="DAT_PTP">
    <Name>
      <Item Type="const">DAT_PTP</Item>
      <Item Type="dynamicParameter">Loc Id</Item>
    </Name>
    <DownloadLayer>
      <Line>
        <Item Type="const">DECL DAT_PTP</Item>
        <Item Type="dataName">DAT_PTP</Item>
        <Item Type="const">={PL_ACC</Item>
        <Item Type="parameter">PL_ACC</Item>
        <Item Type="const">}</Item>
      </Line>
    </DownloadLayer>
  </Data>

  <Data Type="TQM_TQDAT_T">
    <Name>
      <Item Type="const">TM</Item>
      <Item Type="dynamicParameter">Loc Id</Item>
    </Name>
    <DownloadLayer>
      <Line>
        <Item Type="const">DECL TQM_TQDAT_T</Item>
        <Item Type="dataName">TQM_TQDAT_T</Item>
        <Item Type="const">={T11</Item>
        <Item Type="parameter">T11</Item>
        <Item Type="const">}</Item>
      </Line>
    </DownloadLayer>
  </Data>
</DataDef>
</RobotController>
```

Dve časti každého `<Data>`:
- `<Name>` — ako sa bude dátový typ volať (konštanty + parametre + dynamické parametre)
- `<DownloadLayer>` — celý riadok deklarácie. `<Item Type="dataName">` vloží
  názov definovaný vyššie.

---

## Použitie v OLP XML

```xml
<Command Name="MountGun">
  <RoboticParamRef>
    <Param>OperationTask</Param>
    <Param>Gun</Param>
  </RoboticParamRef>

  <DataTypeRef>
    <Data>TQM_TQDAT_T</Data>
    <Data>DAT_PTP</Data>
  </DataTypeRef>

  <DataDef>
    <!-- voliteľný prepis názvu iba pre tento príkaz -->
    <Data Type="DAT_PTP">
      <Name>
        <Item Type="const">DAT_PTP</Item>
        <Item Type="const">_</Item>
        <Item Type="dynamicParameter">Loc Id</Item>
        <Item Type="const">_</Item>
        <Item Type="parameter">PL_ACC</Item>
      </Name>
    </Data>
  </DataDef>
</Command>
```

## Použitie v Motion XML

Rovnaká logika, `<DataTypeRef>` a `<DataDef>` sa dávajú do `<Process>`:

```xml
<Location Type="WELD">
  <Process Type="Spot Off">
    <DataTypeRef>
      <Data>TQM_TQDAT_T</Data>
      <Data>DAT_PTP</Data>
    </DataTypeRef>
    <DataDef>
      <Data Type="DAT_PTP">
        <Name>…</Name>
      </Data>
    </DataDef>
    <Dialog>…</Dialog>
```

V oboch prípadoch sa názov dátového typu vypíše cez
`<Item Type="dataName">XXX</Item>`.

---

## Zdieľanie medzi OLP a Motion ⭐

`<RoboticParams>` a `<Aliases>` definované v **DataConfiguration** sú **spoločné**
pre OLP aj Motion konfiguráciu.

Definuj ich teda **raz tu** a odkazuj sa na ne z oboch strán — namiesto
duplikovania v každom súbore.

---

## Upload dátových typov

Služba načíta hodnotu aj názov dátového typu a pokúsi sa z neho vyextrahovať
parametre (funguje v OLP a Motion, **nie** pri uploade samotného `dataDef`).

Ak sa názov nezhoduje s XML, riadok sa preskočí. Aby fungoval, potrebuje
**koncovú konštantu**:
```
DECL MYDATATYPE <dataName> ={<Definition>}
CONST MYDATATYPE <dataName> := [<Definition>]
```
