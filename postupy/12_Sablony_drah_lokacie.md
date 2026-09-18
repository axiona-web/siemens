# Postup 12 — Šablóny dráh: pridávanie a úprava lokácií

**Cieľ:** automaticky pridávať nábehové a odbehové lokácie, posúvať ich,
zarovnávať, premenúvať a kopírovať konfiguráciu — bez ručného klikania.

**Kam súbor patrí:** rovnako ako postup 11 —
```
<PS installation>\eMPower\Robotics\Olp\<controller>\PathTemplateConfiguration\*.xml
```

Tento postup pokrýva elementy vnútri `<Action>`: `AddLoc`, `MoveLoc`,
`Relocate`, `DynamicAlignment`, `Rename`, `CopyConfiguration`, `ApplyOn`.

---

## `AddLoc` — vytvorenie novej lokácie

```xml
<Action Name="AddApproachLocation" LocRange="First">
  <AddLoc Name="approachViaLoc" RefLoc="First" Placed="After"
          CopyAttachment="true" RelX="-100.05" RelZ="100.05">
    <!-- platí len pre novú via lokáciu -->
    <Param Name="Speed Data" Dynamic="true" Value="z1000"/>
  </AddLoc>
  <!-- platí pre výstup filtra akcie -->
  <Param Name="RRS_MOTION_TYPE" Value="2"/>
</Action>
```

⭐ **Dôležitý princíp:** čo je **vnútri** `<AddLoc>`, platí len pre novú lokáciu.
Čo je **mimo** `<AddLoc>`, sa aplikuje na výstup filtra akcie — a systém filter
prepočíta znovu, už aj s novou lokáciou.

### Atribúty `AddLoc`

| Atribút | Význam |
|---|---|
| `Name` | Názov novej lokácie |
| `NameSuffix` | Názov `RefLoc` + táto prípona |
| `NameSuffixIndex` | Názov `RefLoc` + podtržník + toto číslo |
| `RefLoc` (povinný) | Referenčná lokácia pre súradnice: `First`, `Last`, `Itself`, `PreviousLoc`, `NextLoc` |
| `Placed` (povinný) | `Before`, `After`, alebo `Automatic` (len pre seam — vloží sa na správne miesto pozdĺž MFG) |
| `LocType` | `via` (predvolené), `seam`, `pick`, `place` |
| `CopyAttachment` | `true` — skopíruje pripojenie (attachment) z lokácie v `LocRange` |
| `CopyExternals` | `true` — skopíruje externé osi z referenčnej lokácie |
| `CopyConfiguration` | `true` — skopíruje konfiguráciu ramena zo zdrojovej lokácie |
| `RelX`, `RelY`, `RelZ` | Prírastok k súradnici `RefLoc` |
| `RelRX`, `RelRY`, `RelRZ` | Prírastok k rotácii `RefLoc` (v stupňoch, double) |
| `UseWorldCoordinates` | `true` — súradnice sú absolútne, voči nulovému bodu |
| `RefreshFilter` | `true` — po vytvorení lokácie prepočítať filter akcie a až potom pokračovať |
| `Distance` | Vzdialenosť pozdĺž MFG od `RefLoc` (**len pre seam**) |
| `IgnoreRefLocAlignment` | `true` — ignoruje zarovnanie podľa `RefLoc`; výsledok je rovnaký ako pri *Insert Location Inside Seam* |
| `PoseName` | Názov pózy pre novú lokáciu |

⚠ Ak filtrom prejde viacero lokácií, každá vstupujúca lokácia môže zmeniť to,
na čo ukazuje `RefLoc`. Pri reťazení akcií to je najčastejší zdroj prekvapení —
použi `RefreshFilter="true"`, ak chceš mať poradie pod kontrolou.

⚠ Pri `Placed="Automatic"` musíš v tej istej akcii uviesť aj `LocType="seam"`.

### Príklad 1 — odbehová lokácia s príponou

```xml
<Action Name="AddDepartLocation">
  <AddLoc NameSuffix="_1" Placed="After" RefLoc="Itself"/>
</Action>
```

### Príklad 2 — číslovaná prípona

```xml
<Action Name="AddLocations">
  <AddLoc NameSuffixIndex="8" Placed="After" RefLoc="Itself"/>
</Action>
```
⚠ Systém spracúva všetky lokácie sekvenčne, bez ohľadu na to, z ktorej
operácie pochádzajú.

### Príklad 3 — dve lokácie za sebou

```xml
<Action Name="AddTwoLocations" LocRange="First">
  <AddLoc Name="firstAddVia" RefLoc="First" RefreshFilter="true"
          Placed="Before" RelX="-100.0" RelZ="100.0">
    <Param Name="Speed" Value="20"/>
  </AddLoc>
  <AddLoc Name="secondAddVia" RefLoc="First" RefreshFilter="true"
          Placed="After" RelY="-80.0" RelRZ="80.0">
    <Param Name="Speed" Value="30"/>
  </AddLoc>
  <Param Name="RRS_MOTION_TYPE" Value="2"/>
</Action>
```
Po prvom riadku je `RefLoc`/`LocRange` pôvodná prvá lokácia. Po jeho vykonaní sa
prvou lokáciou stáva `firstAddVia` — a druhý `AddLoc` už pracuje s ňou.
Posledný `<Param>` sa aplikuje na aktuálnu prvú lokáciu.

### Príklad 4 — Pick & Place

```xml
<RobotController Name="Default" Version="All">
<ActionList>
  <Action Name="AddPlace" Description="Pridá pick_and_place lokácie"
          LocationTypes="place">
    <AddLoc Name="place_2" LocType="place" Placed="after" RefLoc="first"
            PoseName="HOME" CopyAttachment="true">
      <Color Value="GREEN"/>
    </AddLoc>
  </Action>
</ActionList>
</RobotController>
```

### Hodnoty súradníc zadané až za behu ⭐

Ak namiesto čísla napíšeš `?`, hodnotu doplní užívateľ cez tlačidlo **Set**
v dialógu Apply Path Template:

```xml
<Action Name="Test1" LocRange="1">
  <AddLoc NameSuffix="_newBySetValue" LocType="via" Placed="after"
          RefLoc="itself" RelX="?">
    <Color Value="Brown"/>
  </AddLoc>
</Action>
```
Platí pre `AddLoc` aj `MoveLoc`.

---

## `MoveLoc` — posun existujúcich lokácií

```xml
<Action Name="MoveX10Z20" LocRange="All">
  <MoveLoc RelX="10" RelZ="20" IgnoreLimitations="true"/>
</Action>
```

| Atribút | Význam |
|---|---|
| `RelX`, `RelY`, `RelZ` | Prírastok k súradnici |
| `RelRX`, `RelRY`, `RelRZ` | Prírastok k rotácii |
| `IgnoreLimitations="true"` | Ignoruje obmedzenia nastavené v Options |
| `UseWorldCoordinates="true"` | Súradnice sú absolútne (voči nulovému bodu) |

⚠ Pri **zvarových lokáciách** sa nastavujú iba `Rx`, `Ry`, `Rz`. Ak vyjdú mimo
rozsah definovaný v Options, akcia **zlyhá**.

---

## `Relocate` — prepočet podľa rozdielu dvoch TCP

Typický prípad: vymenili zváraciu kliešte za iné s posunutou osou x.

Do Motion XML pridaj parametre:
```xml
<Param Name="TCPF1" ValueType="TxObject" TxValidatorType="Frame"/>
<Param Name="TCPF2" ValueType="TxObject" TxValidatorType="Frame"/>
<Param Name="ScopeType" ValueType="string" Default="All">
  <ComboDef>
    <ElmDef>All</ElmDef>
    <ElmDef>Rotation</ElmDef>
    <ElmDef>Translation</ElmDef>
  </ComboDef>
</Param>
```

Do šablóny dráhy:
```xml
<Action Name="Correct TCP">
  <Relocate FromParam="TCPF1" ToParam="TCPF2" Scope="ScopeType"
            IgnoreLimitations="true"/>
</Action>
```

| Atribút | Význam |
|---|---|
| `From` / `FromParam` (povinný) | Východiskový rám |
| `To` / `ToParam` (povinný) | Cieľový rám |
| `Scope` (povinný) | `All`, `Rotation` alebo `Translation` |
| `IgnoreLimitations` | `true` — ignoruje obmedzenia z Options |

Po potvrdení v dialógu Apply Path Template Action sa otvorí dialóg
**Correct TCP**, kde sa vyberú TCPF rámy a typ rozdielu. Akcia sa vykoná po OK.

---

## `DynamicAlignment` — zarovnanie na RTCP

```xml
<Action Name="AlignRTCPWeldLocations">
  <DynamicAlignment/>
</Action>
```
Pre lokácie na uchytenom dielci: robot najprv skočí na lokáciu a potom ju
`DynamicAlignment` zarovná na RTCP rám.

⚠ Obmedzenie: funguje len na lokáciách **bez externých osí a bez naučených
(taught) údajov**.

---

## `Rename` — hromadné premenovanie

| Atribút | Význam |
|---|---|
| `Name` | Premenuje na túto hodnotu |
| `NamePrefix` | Pridá pred názov |
| `NameSuffix` | Pridá za názov |
| `NameSuffixIndex` | Pridá číselný index |

```xml
<Action Name="RenameApproach" LocRange="First">
  <Rename NamePrefix="app_" NameSuffixIndex="1"/>
</Action>
```

---

## `CopyConfiguration` — kopírovanie konfigurácie ramena

Pri kopírovaní lokácie:
```xml
<RobotController Name="Default">
<ActionList>
  <Action Name="AddLocWithConf" LocRange="First">
    <AddLoc NameSuffix="_sameConfiguration" Placed="after"
            RefLoc="itself" CopyConfiguration="true"/>
  </Action>

  <!-- alebo skopírovať konfiguráciu z inej lokácie -->
  <Action Name="SetConfigurationForLocations" LocRange="2-5">
    <CopyConfiguration CopyFrom="-1"/>
  </Action>
</ActionList>
</RobotController>
```

---

## `ApplyOn` — akcia na susednú lokáciu ⭐

Niekedy treba niečo spraviť nie na aktuálnej lokácii, ale na tej pred ňou alebo
za ňou:

```xml
<Action Name="ApplyOnTest" LocRange="last">
  <Olp CustomizedDialogName="Prepare Motor" ApplyOn="-1"/>
  <Olp CustomizedDialogName="Start Motor"/>
  <Olp CustomizedDialogName="Motor Working" ApplyOn="+1" BetweenOperations="true"/>
</Action>
```

- `Start Motor` → na poslednej lokácii (kvôli `LocRange="last"`)
- `Prepare Motor` → na predposlednej (`ApplyOn="-1"`)
- `Motor Working` → na **prvej lokácii nasledujúcej operácie** — to je možné len
  s `BetweenOperations="true"` (jediná platná hodnota)

⚠ Nasledujúca operácia sa určuje podľa väzieb v **Sequence Editore**. Ak väzba
na ďalšiu operáciu neexistuje alebo ich je viac, zobrazí sa chybová správa.

`ApplyOn` funguje aj na nesusediacich operáciách — na všetkých seam a via
lokáciách pod nadradenou continuous operáciou.

---

## Na čo si dať pozor

1. Rozlišuj, čo je vnútri a čo mimo `<AddLoc>` — to je najčastejšia chyba.
2. Pri reťazení viacerých `AddLoc` používaj `RefreshFilter="true"`, inak sa
   referencie posúvajú pod rukami.
3. `Distance` a `Placed="Automatic"` sú len pre seam lokácie.
4. `DynamicAlignment` neprežije externé osi ani naučené polohy.
5. `IgnoreLimitations` obchádza kontroly z Options — používaj vedome.
6. Parametre, OLP príkazy, filtre a logika → **postup 11**.
