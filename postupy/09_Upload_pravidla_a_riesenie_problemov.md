# Postup 09 — Upload: pravidlá a riešenie problémov

**Problém:** program sa zapíše (download) v poriadku, ale pri spätnom načítaní
(upload) z reálneho robota sa vlastný príkaz nerozpozná — alebo sa načíta
s **inými hodnotami**, než sú v programe.

Download musí byť **reverzibilný**. Väčšina chýb vzniká pri návrhu download
vrstvy, nie pri uploade.

---

## Deväť pravidiel, ktoré musí download vrstva splniť

### 1. Za `string`, `dynamicParameter` a `TxObject` musí nasledovať konštanta

Parser číta hodnotu od aktuálnej pozície **po najbližšiu konštantu**.

❌ Nesprávne:
```xml
<Item Type="parameter">Robot</Item>
<Item Type="parameter">Gun</Item>
```
✅ Správne:
```xml
<Item Type="parameter">Robot</Item>
<Item Type="const">;</Item>
<Item Type="parameter">Gun</Item>
```

**Alebo** musí byť parameter **posledný v riadku** (potom sa číta do konca riadka).

### 2. Pri rozbaľovacom zozname (combo) konštanta netreba
Parser porovná text s položkami zoznamu. Ak ide o voľný reťazec bez koncovej
konštanty, použije sa najbližší **nealfanumerický znak**.

### 3. Int s rovnakým počtom číslic v Min a Max sa dá reťaziť
```xml
<Param Name="ordre1" ValueType="int" MaxVal="100" MinVal="999"/>  <!-- 3 číslice -->
<Param Name="order2" ValueType="int" MaxVal="10"  MinVal="99"/>   <!-- 2 číslice -->
<Param Name="order3" ValueType="int" MaxVal="0"   MinVal="9"/>    <!-- 1 číslica -->
```
Download: `123456` → Upload: order1=123, order2=45, order3=6.

### 4. Za `int`/`double` nesmie prvý ďalší znak byť číslica
```
Int Speed = 88, TxObject Robot = 9robot   →   download: "889robot"
```
Parser nevie, kde končí číslo. Vlož medzi ne konštantu.

### 5. Voliteľné parametre treba oddeľovať rovnako ako povinné
V žiadnom robotickom jazyku neexistujú „voliteľné“ argumenty volania:
`CALL XYZ (par1, par2, opt1, opt2)` sa nedá zapísať ako `CALL XYZ (par1, par2, opt2)`
— kompilátor hlási chýbajúci parameter. Rieši sa to buď
`CALL XYZ (par1, par2, , opt2)` alebo `CALL XYZ (par1, par2, EMPTY, opt2)`.
Rovnako to musíš oddeliť aj v XML — konštantou alebo kľúčovým slovom.

### 6. Jeden OLP príkaz = jedna inštrukcia
Jeden riadok; jeden fold pri KUKA; jeden blok `MOVE … ENDMOVE` pri Comau.

### 7. Na konci riadka nesmie byť medzera

### 8. Kuka Vkrc upload nie je podporovaný
OLP príkaz sa pri Vkrc rozpadá do troch rôznych miest programu.

### 9. Veľkosť písmen názvov objektov určuje štýl radiča

---

## Keď konštanta nie je možná: `UploadRegex`

```xml
<Item Type="dynamicParameter" UploadRegex="\w+">LPDatShortName</Item>
<Item Type="parameter" UploadRegex="\w+">ShortName</Item>
```

## Reťazec na konci switch/case

Upload zvládne reťazcový parameter na konci `switch`, ak za **celým blokom**
nasleduje konštanta — tá môže byť aj v **nasledujúcom** bloku switch:

```xml
<Switch ParamName="LPDatShortName" DynamicParameter="true">
  <Case Value="">
    <Item Type="const">no LPDatShortName[</Item>
  </Case>
  <Case>
    <Default>
      <Item Type="const">LPDatShortName:</Item>
      <Item Type="dynamicParameter">LPDatShortName</Item>
    </Default>
  </Case>
</Switch>
<Switch ParamName="Base Name" DynamicParameter="true">
  <Case Value="">
    <Item Type="const">Base[</Item>   <!-- toto je tá konštanta -->
    ...
```

---

## Medzery: `optionalSpaces`

```xml
<Item Type="optionalSpaces" xml:space="preserve">   </Item>
```
Pri zápise sa vypíše presne toľko medzier, pri načítaní sa akceptuje 0 až N.
Funguje obojsmerne — program s medzerami aj bez nich sa načíta.

**Vo väčšine prípadov to už písať nemusíš.** Parser sám:
- rozdelí konštantu podľa medzier,
- pridá voliteľné medzery okolo každého nealfanumerického znaku
  (`" ! @ $ # % . , = : - + { } ( ) ;`).

Takže stačí:
```xml
<Item Type="const">;%{PE}%MKUKATPUSER</Item>
```
a načíta sa aj `; % { PE } % MKUKATPUSER`.

**Vlastný oddeľovač** (voliteľná medzera medzi zlepenými znakmi):
```xml
<Item Type="const" Separator=".">; %.{.PE.} %.MKUKATPUSER</Item>
```
**Vypnutie automatického delenia podľa medzier:**
```xml
<Item Type="const" SpaceSeparator="false" Separator=".">; %.{.PE.} %.MKUKATPUSER</Item>
```

## Viacriadková download vrstva
Všetky radiče podporujú upload vlastných príkazov s viacerými riadkami
(viacerými foldmi pri Kuka-Krc).

---

## Postup pri ladení zlyhaného uploadu

1. Vytvor v štúdii lokáciu **rovnakého typu** (weld/via/seam start…) a
   **rovnakého typu pohybu** ako problémová.
2. Nastav na ňu problémový vlastný príkaz / typ procesu s parametrami.
3. Otvor **Upload Debugger** (pravý klik v TP alebo stĺpec Customized Debug
   v Path Editore).
4. Vlož problémový riadok z reálneho programu robota → **Re-Upload**.
5. Riadky, ktoré zlyhali, sa zvýraznia **červenou** s vysvetlením príčiny.

### Tichá chyba: upload prejde, ale obsah je iný
Stáva sa to, keď program obsahuje kombináciu, ktorú tvoja download vrstva
nedokáže vygenerovať. Upload prejde, ale hodnoty sa „prilepia“ k inej vetve.

Na to slúži tlačidlo **Upload and Download** v Upload Debuggeri: načíta a hneď
znova zapíše obsah a porovná ho s pôvodným. Rozdiel sa označí červenou.

Príklad: `GlueTechVersion` môže byť `1.2.1` alebo `2.3.2`, download vrstva
vypisuje `Yes` pre 1.2.1 a `No` inak. Ak program obsahuje `;FOLD 2.3.2 …= Yes`,
upload prejde — ale výsledok bude `1.2.1`. Bez tohto tlačidla to neodhalíš.
