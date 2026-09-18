# Postup 11 — Šablóny dráh: parametre a OLP príkazy

**Cieľ:** jedným kliknutím nastaviť celú dráhu — rýchlosti, zóny, nástroje,
OLP príkazy, farby — namiesto ručného klikania po jednotlivých lokáciách.

**Kam súbor patrí:**
```
<PS installation>\eMPower\Robotics\Olp\<controller>\PathTemplateConfiguration\*.xml
```
Každý radič má vlastný priečinok (vytvorí sa pri inštalácii radiča).
**Na názvoch súborov nezáleží** — berú sa do úvahy všetky XML v priečinku.

---

## Načo to je

Typický príklad — lepenie. Pre každú lepiacu dráhu treba nastaviť rovnaké veci:
štart lepenia, stop lepenia, rýchlosť robota, nábehové a odbehové lokácie.
Šablóna dráhy si tieto akcie zapamätá a aplikuje ich naraz na viacero operácií.

Aplikuje sa cez dialóg **Apply Path Template Action** v Process Simulate.

---

## Kostra súboru

```xml
<RobotController Name="Abb-Rapid" Version="4.0.91.s4cplus, 5.07.01.irc5">
  <ActionList>
    <Action Name="SetAllLocations" LocRange="All"
            LocationTypes="Via,Weld"
            ProcessTypes="Polishing,Glue-on"
            MotionTypes="1,2"
            Description="nastaví všetky lokácie s procesom Polishing: \n rýchlosť[20] \n zóna[z50]">
      <!-- čo sa má spraviť -->
    </Action>
  </ActionList>
</RobotController>
```

**`<RobotController>`**
- `Name` (povinný) — názov radiča
- `Version` (voliteľný) — verzie radiča, pre ktoré XML platí; verzia sa definuje
  v `rrs.xml` cez aplikáciu Robot Properties. Ak chýba, platí `All`.

---

## Atribúty `<Action>`

| Atribút | Význam |
|---|---|
| `Name` (povinný) | Názov akcie — **musí byť jedinečný** |
| `LocRange` | Na ktoré lokácie akciu použiť |
| `SeamRange` | Ktoré seam operácie pod zloženou operáciou |
| `LocationTypes` | Filter podľa typu lokácie |
| `ProcessTypes` | Filter podľa typu procesu (stĺpec Process Type v Path Editore) |
| `MotionTypes` | Filter podľa typu pohybu |
| `Description` | Popis zobrazený v dialógu Apply Path Template Action |
| `Hide` | `true` — akciu skryť zo zoznamu (pozri nižšie) |

⚠ Výsledný filter je **prienik** všetkých filtrov:
`SeamRange × LocRange × LocationTypes × ProcessTypes × MotionTypes`.
Všetky indexy musia byť väčšie ako nula.

### Menu a oddeľovače v názve

Znak `|` vytvorí podmenu, `-` na konci vytvorí oddeľovač:

```xml
<Action Name="Paint|Correct TCP"> … </Action>
<Action Name="Paint|-"></Action>
<Action Name="Paint|CleanAll"> … </Action>
```

### LocationTypes — možné hodnoty

`Via`, `Weld`, `Seam Start`, `Seam End`, `Seam Middle`, `Pick`, `Place`

```xml
<Action Name="AddLocAction" LocationTypes="Pick,Place">
  <Color Value="RED"/>
</Action>
```

### LocRange a SeamRange — možné hodnoty

`All` (predvolené), `First`, `Last`, `n1`, `Last - n1`, `"n1-n2"`, `"n1,n2,n3"`,
`Operation`, `AllWithOperation`

Podporený je aj zápis `<od>-<do>:<krok>`:

| Zápis | Význam |
|---|---|
| `"4-8:2"` | od 4. po 8., každú 2. lokáciu |
| `"4-Last:2"` | od 4. po poslednú, každú 2. |
| `"4-(Last-5):2"` | od 4. po (poslednú − 5), každú 2. |
| `"(Last-4)-(Last-1)"` | rozsah relatívne od konca |

⚠ Hodnota `Operation` nefunguje spolu s inými filtrami — použi ju len pre OLP
príkazy, farbu a robotické parametre **na úrovni operácie**.

`AllWithOperation` = akcia sa vykoná na všetkých lokáciách **aj** na samotnej
operácii.

---

## `<Param>` — nastavenie robotických parametrov

```xml
<Action Name="SetAllLocations" LocRange="All">
  <Param RemoveAll="true"/>
  <Param Name="RRS_MOTION_TYPE" Value="2"/>
  <Param Name="Speed Data"/>
  <Param Name="Wobj Data" Dynamic="true" Value="wobj0"/>
  <Param Name="Zone Data" Dynamic="true" Remove="true"/>
</Action>
```

| Atribút | Význam |
|---|---|
| `Name` | Názov parametra — plne definovaný v Motion XML. Povinný, ak nepoužívaš `RemoveAll` |
| `Value` | Nová hodnota. **Ak chýba, PS pri aplikovaní zobrazí dialóg** a užívateľ hodnotu doplní |
| `Dynamic="true"` | Pre komplexné parametre. Platné len pre tie so zápisom v `…\Process Simulate\<verzia>\Robotics\PathEditor\AvailableColumns\*.xml` |
| `Remove="true"` | Odstráni konkrétny parameter z lokácie |
| `RemoveAll="true"` | Odstráni všetky robotické parametre z lokácie |

⚠ Pre komplexné a DynamicCombo parametre je `Value` **povinná**, pokiaľ
nepoužívaš `Remove` alebo `RemoveAll`.

### Tlačidlo Set — hodnoty za behu ⭐

Tlačidlo **Set** v dialógu Apply Path Template otvorí vygenerovaný dialóg
(vyzerá rovnako ako vlastný OLP/motion dialóg), kde sa doplnia:
- všetky robotické parametre bez hodnoty
- všetky vlastné OLP príkazy, ktoré majú aspoň jeden parameter bez predvolenej hodnoty

Staré hodnoty z posledného nastavenia sa dajú znova upraviť.

Dynamické parametre sa dajú nastaviť za behu takto:

```xml
<Action Name="Set Dynamic Parameters">
  <Param Name="Gun Wait" Dynamic="true"/>
  <Param Name="Speed"    Dynamic="true"/>
  <Param Name="Zone"     Dynamic="true"/>
</Action>
```

### Hodnota počítaná výrazom

```xml
<Action Name="AddLocAction">
  <Param Name="Speed"   Dynamic="True" Expression="![CDATA[('A15_Speed m/min'/60)]]"/>
  <Param Name="MySpeed"               Expression="![CDATA[('A15_Speed m/min'/60)]]"/>
</Action>
```

### Kopírovanie parametra zo susednej lokácie

```xml
<Action Name="CopyParameters" LocRange="last">
  <Param Name="SpindleRPM_D" CopyFrom="-2" CopyParamName="SpindleRPM_S"/>
  <Param Name="Zone" Dynamic="true" CopyFrom="-2" ApplyOn="-1"/>
</Action>
```
`CopyFrom` berie celé číslo (posun oproti aktuálnej lokácii),
`CopyParamName` umožní kopírovať z parametra s iným názvom.
Funguje aj cez hranice operácií — na všetkých seam a via lokáciách pod
nadradenou continuous operáciou.

---

## `<Olp>` — pridávanie a mazanie OLP príkazov

```xml
<Action Name="SetOLPCommands" LocRange="3-8">
  <Olp RemoveAll="true"/>
  <Olp Name="Check Grp 2 State= OPN at Start Delay= 2 ms"/>
  <Olp Name="# GunToState" Remove="true"/>
  <Olp Name="PULSE 2 'sig1' State= TRUE Time= 3 sec"/>
  <Olp CustomizedDialogName="OrdreArret"/>
</Action>
```

| Atribút | Význam |
|---|---|
| `Name` | Text OLP príkazu. Pre štandardný/špecifický radič vytvorí *free text* príkaz, pre vlastný radič *composite* príkaz. Ak už rovnaký príkaz na lokácii je, pridá sa ďalší |
| `Remove="true"` | Odstráni všetky príkazy s rovnakým celým textom |
| `RemoveAll="true"` | Odstráni všetky OLP príkazy z lokácie |
| `CustomizedDialogName` | Názov vlastného dialógu z OLP XML |
| `CreationIndex` | Poradie vloženia do zoznamu OLP príkazov na lokácii. Celé číslo > 0, inak sa zobrazí varovanie |

⚠ Ak je v OLP XML pre dialóg použitý atribút `Id`, do `CustomizedDialogName`
sa uvádza **hodnota `Id`**, nie názov dialógu.

Ak majú všetky parametre príkazu predvolené hodnoty, príkaz sa vytvorí
automaticky. Inak PS zobrazí dialóg na doplnenie.

Po aplikovaní akcie musí reálny radič zvládnuť: **editáciu (TP), simuláciu,
download aj upload** takto vytvoreného príkazu.

---

## `Replace` — výmena starého príkazu za nový

```xml
<Action Name="ReplaceOldOlp" Description="Nahradí starý OLP novým">
  <Olp Name="PULSE 1 'sig1' State= TRUE Time= 1 sec" ReplaceOLPStartWith="PULSE 2"/>
  <Olp CustomizedDialogName="Machining | Change RPM"
       ReplaceCustomizedDialogName="Machining | Stop Motor"/>
  <Olp Name="Bingo" ReplaceOLPStartWith="StopMotor"/>
</Action>
```
- `ReplaceOLPStartWith` — nahradí každý OLP príkaz, ktorý **začína** touto hodnotou
- `ReplaceCustomizedDialogName` — nahradí každý vlastný OLP vytvorený daným dialógom

---

## `Update` — prepočítanie OLP parametra

Aktualizuje parametre OLP príkazu podľa hodnoty robotického parametra.
Funguje **len** pre OLP parametre definované ako `UseLocationParamVal`.

```xml
<Update Name="DialogName.ParameterName"/>
```

---

## `<Color>` — farba lokácie

```xml
<Action Name="SetColor" LocRange="First">
  <Color Value="YELLOW"/>
</Action>
```
Možné hodnoty: Red, Green, Yellow, Blue, Magenta, Cyan, Orange, White, Pink,
Gray, Brown, Wood, Dark green, Dark red, Dark brown, Light blue, Black.

---

## `ActionRef` — skladanie akcií z akcií ⭐

```xml
<Action Name="Polishing" LocRange="First">
  <ActionRef>AddApproachLocation</ActionRef>
  <ActionRef>SetFirstLocation</ActionRef>
  <Param Name="SW_TIME_ON_PT" Value="0.8"/>
</Action>
```
Toto je hlavný nástroj proti duplicite — malé akcie sa definujú raz a skladajú
do väčších.

### `Hide` — skrytie pomocných akcií

Aby sa v zozname pod tlačidlom **Select** neukazovali čiastkové akcie:

```xml
<Action Name="RootAction" LocRange="AllWithOperation">
  <ActionRef>AddParametersToOperation</ActionRef>
  <ActionRef>AddLocationsWithParamters</ActionRef>
</Action>

<Action Name="AddParametersToOperation" LocRange="Operation" Hide="true"> … </Action>
<Action Name="AddLocationsWithParamters" LocRange="All"       Hide="true"> … </Action>
```

---

## `ActionFilter` — ktoré akcie pre ktoré operácie

```xml
<ActionFilter>
  <Operation Types="Weld,Continuous">
    <ActionRef>Polishing</ActionRef>
    <ActionRef>SetOLPCommands</ActionRef>
  </Operation>
  <Operation Types="Continuous">
    <ActionRef>SetOLPCommands</ActionRef>
    <ActionRef>SetFirstLocation</ActionRef>
  </Operation>
</ActionFilter>
```
`Types` môže byť: `Weld`, `PickAndPlace`, `Continuous`, `Seam`.

⚠ Dialóg Apply Path Template Action ukáže akcie podľa **prieniku** všetkých
filtrov. Akcia bez filtra sa zobrazí pre všetky operácie.

---

## `If-ElseIf-Else` — rozhodovanie podľa parametrov

```xml
<Action Name="ActionWithLogic">
  <If>
    <![CDATA['RRS_MOTION_TYPE'==1]]>
    <Color Value="RED"/>
    <Param Name="TimeOut" Value="3"/>
    <AddLoc NameSuffix="_1" Placed="before" RefLoc="itself" RelY="-50">
      <Param Name="RRS_MOTION_TYPE" Value="2"/>
      <Color Value="Yellow"/>
    </AddLoc>
  </If>
  <ElseIf>
    <![CDATA['RRS_MOTION_TYPE'==2]]>
    <Color Value="Brown"/>
    <If>
      <![CDATA['TimeOut'>8]]>
      <AddLoc NameSuffix="_1" Placed="after" RefLoc="itself" RelX="-100">
        <Param Name="RRS_MOTION_TYPE" Value="2"/>
        <Color Value="Yellow"/>
      </AddLoc>
    </If>
  </ElseIf>
  <Else>
    <Color Value="GREEN"/>
    <Param Name="RRS_MOTION_TYPE" Value="4"/>
  </Else>
</Action>
```
Podmienky sa dajú vnárať.

---

## Zložené operácie

Šablónu možno aplikovať aj na **compound operation** — do dialógu stačí vložiť
zloženú operáciu namiesto všetkých jej podoperácií.

⚠ **Obmedzenie:** všetky operácie pod zloženou operáciou musia mať priradený
**ten istý robot**.

---

## Kompletný príklad

Motion XML (definícia parametra):
```xml
<RobotController Name="Abb-Rapid" Version="All">
  <RoboticParams>
    <Param Name="RRS_MOTION_TYPE" ValueType="int"/>
  </RoboticParams>
</RobotController>
```

PathTemplateConfiguration:
```xml
<RobotController Name="Abb-Rapid" Version="All">
<ActionList>

  <Action Name="SetColorWeld" LocRange="All" LocationTypes="WELD">
    <Color Value="RED"/>
  </Action>

  <Action Name="SetColorVia" LocRange="All" LocationTypes="VIA">
    <Color Value="BLACK"/>
  </Action>

  <Action Name="Weld Op Color" LocRange="All"
          Description="Zvarové lokácie červené \nVia lokácie čierne">
    <ActionRef>SetColorWeld</ActionRef>
    <ActionRef>SetColorVia</ActionRef>
  </Action>

  <Action Name="AddViaBeforeWeld" LocRange="All" LocationTypes="WELD">
    <AddLoc NameSuffix="_01" Placed="Before" RefLoc="Itself" RelX="-5">
      <Param Name="Zone Data"  Dynamic="True" Value="z30"/>
      <Param Name="Speed Data" Dynamic="True" Value="v1000"/>
      <Param Name="RRS_MOTION_TYPE" Value="1"/>
      <Color Value="YELLOW"/>
    </AddLoc>
  </Action>

  <Action Name="AddViaAfterWeld" LocRange="All" LocationTypes="WELD">
    <AddLoc NameSuffix="_02" Placed="After" RefLoc="Itself" RelX="-5">
      <Param Name="Zone Data"  Dynamic="True" Value="z80"/>
      <Param Name="Speed Data" Dynamic="True" Value="v600"/>
      <Param Name="RRS_MOTION_TYPE" Value="1"/>
      <Color Value="BLUE"/>
    </AddLoc>
  </Action>

  <Action Name="Add Location|Add Weld Process Via" LocRange="All"
          Description="Nábeh pred zvarom (z30, v1000) \nOdbeh po zvare (z80, v600)">
    <ActionRef>AddViaBeforeWeld</ActionRef>
    <ActionRef>AddViaAfterWeld</ActionRef>
  </Action>

  <Action Name="AddDepartLocation" LocRange="Last">
    <AddLoc Name="departViaLoc" Placed="After" RefLoc="LAST" RelX="10" RelZ="150"/>
    <Color Value="YELLOW"/>
  </Action>

</ActionList>
</RobotController>
```

---

## Na čo si dať pozor

1. Názov akcie musí byť jedinečný v rámci celého priečinka.
2. Filtre sa násobia — pri príliš úzkom filtri sa akcia „nič nerobí“ a nie je
   jasné prečo. Skúšaj najprv bez filtrov.
3. Parameter bez `Value` = dialóg pri každom použití. Je to fičúra, nie chyba —
   dá sa cielene využiť.
4. Pri zloženej operácii musí byť všade rovnaký robot.
5. Práca s lokáciami (`AddLoc`, `MoveLoc`, `Rename`, `Relocate`) → **postup 12**.
