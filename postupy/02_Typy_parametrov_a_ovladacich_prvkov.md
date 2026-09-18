# Postup 02 — Typy parametrov a ovládacích prvkov

**Cieľ:** navrhnúť polia dialógu tak, aby sa nedali vyplniť nesprávne.

Všetko sa definuje v sekcii `<RoboticParams>`.
Povinné atribúty každého `<Param>`: `Name`, `ValueType`. Voliteľne `Caption`
(zobrazovaný názov v dialógu).

---

## int — celé číslo

```xml
<Param Name="ProgNr" ValueType="int" MaxVal="16" MinVal="1" Default="1"/>
```

**Rozsah podľa kĺbu zariadenia:**
```xml
<Param Name="AnvilRotationVal" ValueType="double"
  MaxVal="LowerRam_G2000.Rot.Max"
  MinVal="LowerRam_G2000.Rot.Min"
  Default="LowerRam_G2000.Rot.Current"/>
```
Hodnoty sú v stupňoch alebo mm podľa typu kĺbu.

## double — desatinné číslo

```xml
<Param Name="Speed" ValueType="double" MaxVal="20.7" MinVal="10.7" Default="11.8"/>
```

**V XML vždy bodka.** V dialógu sa zobrazí podľa regionálneho nastavenia Windows
(v SK/DE čiarka). V download a simulačnej vrstve vždy bodka.

Rozsah si užívateľ zobrazí podržaním kurzora nad názvom parametra (tooltip).

## string — text

```xml
<!-- jednoriadkový -->
<Param Name="Comment" ValueType="string"/>

<!-- viacriadkový -->
<Param Name="Comment" ValueType="string" Multiline="true"/>

<!-- s obmedzením dĺžky -->
<Param Name="SeamName" ValueType="string" MaxLength="7"/>
```

## string s rozbaľovacím zoznamom

```xml
<Param Name="Gun Pose" ValueType="string">
  <ComboDef>
    <ElmDef/>            <!-- prázdna položka -->
    <ElmDef>OPEN</ElmDef>
    <ElmDef>CLOSE</ElmDef>
    <ElmDef>SEMI</ElmDef>
  </ComboDef>
</Param>
```

**Iná hodnota v download súbore než v UI:**
```xml
<ElmDef DownloadRepresentation="1">Started</ElmDef>
<ElmDef DownloadRepresentation="2">Done</ElmDef>
```
Toto nahrádza celé switch/case bloky (→ postup 04).

**S obrázkom pri každej položke:**
```xml
<ElmDef Picture="guns_tools\appzc5299229.jpg">Weld 10Am</ElmDef>
```
Cesta je relatívna k `..\Robotics\Olp\CustomizedPictures\`.
Obrázok sa v dialógu mení podľa výberu — výrazne znižuje chybovosť
pri výbere klieští a nástrojov.

## Dynamický zoznam z radiča

```xml
<Param Name="Interp" ValueType="string" DynamicCombo="true"/>
```
Načíta sa za behu. Použiteľné sú parametre označené ako „combobox“ v
`C:\ProgramData\Tecnomatix\Process Simulate\<verzia>\Robotics\PathEditor\AvailableColumns\*.xml`.

## Pózy robota, klieští, chápadla

```xml
<Param Name="RobotPose" ValueType="string">
  <ComboDef LinkTo="Robot.Poses"/>
</Param>
<Param Name="GunPose" ValueType="string">
  <ComboDef LinkTo="ActiveGun.Poses"/>
</Param>
```
`ActiveGun.Poses` vráti pózy klieští, ak nie sú, tak chápadla.

## TxObject — výber objektu zo štúdie

**Bez filtra:**
```xml
<Param Name="RobotOrGun" ValueType="TxObject"/>
```

**S filtrom podľa .NET tried:**
```xml
<Param Name="Location" ValueType="TxObject">
  <PickTypes ShowList="true">
    <PickType>TxRoboticViaLocationOperation</PickType>
    <PickType>TxWeldLocationOperation</PickType>
  </PickTypes>
</Param>
```
`ShowList="true"` zobrazí vyfiltrované objekty ako rozbaľovací zoznam
(bez neho sa musí objekt vybrať klikaním v štúdii).
Triedy nájdeš v manuáli `Tecnomatix .NET.chm`.

**S validátorom (jednoduchšie ako PickTypes):**
```xml
<Param Name="Location" ValueType="TxObject" TxValidatorType="Frame"/>
```

Hodnoty `TxValidatorType`:
`AnyLocatableObject` (predvolený), `Component`, `Group`, `Geometry`, `Frame`,
`Note`, `Device`, `Robot`, `Operation`, `RoboticLocationOperation`,
`WeldLocationOperation`, `PhysicalCollection`, `LocationOrOrderedCompoundOperation`,
`Gripper`, `LocationOperation`, `Gun`, `MfgFeature`, `PhysicalOrLogicalCollection`,
`CollisionPairItem`.

## Signály robota

```xml
<Param Name="ToSignal" ValueType="TxObject">
  <PickTypes ShowList="true">
    <PickType>TxPlcToRobotSignal</PickType>
  </PickTypes>
</Param>
```

`TxPlcToRobotSignal` = signály do robota, `TxPlcFromRobotSignal` = z robota.

**Filter podľa dátového typu signálu:**
```xml
<PickType SignalType="int,dint">TxPlcToRobotSignal</PickType>
```
Možné: `bool, byte, int, dint, word, dword, real, lreal`.

**Zobrazenie doplnkových stĺpcov vedľa názvu signálu:**
```xml
<PickType AddInfo="type,comment,IEC">TxPlcFromRobotSignal</PickType>
```
Predvolený oddeľovač je dvojbodka; zmeníš ho cez `AddInfoSeperator="-->"`.

---

## Skratky, ktoré šetria čas

**Viac parametrov s rovnakou deklaráciou naraz:**
```xml
<Param Name="Pick_1, Pick_2, Pick_3, Pick_4" ValueType="TxObject">
  <PickTypes ShowList="true">
    <PickType>TxRoboticProgram</PickType>
  </PickTypes>
</Param>
```

**Ak názov parametra sám obsahuje čiarku,** vypni toto delenie:
```xml
<Param SingleName="true" Name="SDelay time (0,1-32s)" ValueType="double"/>
```

**Prevzatie hodnoty z robotického parametra lokácie:**
```xml
<Param UseLocationParamVal="true">SW_WAIT_TIME</Param>
```
Užitočné po importe MFG atribútov do robotických parametrov lokácií.
