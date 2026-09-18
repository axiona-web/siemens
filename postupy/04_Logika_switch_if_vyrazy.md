# Postup 04 — Logika: switch/case, if-elseif-else, výrazy

**Cieľ:** výstup príkazu má závisieť od toho, čo užívateľ vybral.

---

## Voľba 1: `DownloadRepresentation` — najjednoduchšie

Ak ide len o to, že v UI má byť „Started“ a v programe „1“, **nepotrebuješ switch**:

```xml
<Param Name="A01_JobMode" ValueType="string" Default="Started">
  <ComboDef>
    <ElmDef DownloadRepresentation="1">Started</ElmDef>
    <ElmDef DownloadRepresentation="2">Done</ElmDef>
  </ComboDef>
</Param>
```

```xml
<DownloadLayer>
  <Line>
    <Item Type="parameter" Context="download">A01_JobMode</Item>
  </Line>
</DownloadLayer>
```

Nahradí desiatky riadkov switch/case.

---

## Voľba 2: `Switch` vnútri riadka

```xml
<Line>
  <Switch ParamName="Location">
    <Case Value="via1">
      <Item Type="const">The Location is: via1</Item>
    </Case>
    <Case Value="via2">
      <Item Type="const">The Location is:</Item>
      <Item Type="parameter">Location</Item>
    </Case>
    <Case>
      <Default>
        <Item Type="const">The Location is: Default</Item>
      </Default>
    </Case>
  </Switch>
</Line>
```

**Podľa dynamického parametra:**
```xml
<Switch ParamName="Attr:RRS_ZONE_NAME" DynamicParameter="true">
  <Case Value="fine"><Item Type="const">G60</Item></Case>
  <Case Value="nodecel"><Item Type="const">G64</Item></Case>
</Switch>
```

**Prázdna hodnota = nevypíš nič:**
```xml
<Switch ParamName="Load Data">
  <Case Value=""/>
  <Case>
    <Default>
      <Item Type="const">\PartLoad:=</Item>
      <Item Type="parameter">Load Data</Item>
    </Default>
  </Case>
</Switch>
```

## Voľba 3: `Switch` mimo riadka

Keď jednotlivé vetvy generujú **rôzny počet riadkov**:

```xml
<Switch ParamName="Location">
  <Case Value="via1">
    <Line><Item Type="const">The location is:</Item>
          <Item Type="parameter">Location</Item></Line>
    <Line><Item Type="const">The Speed is:</Item>
          <Item Type="parameter">Speed</Item></Line>
  </Case>
  <Case>
    <Default>
      <Line><Item Type="const">Default value</Item></Line>
    </Default>
  </Case>
</Switch>
```

**Obmedzenie:** switch nesmie byť vnorený. Vždy len jedna úroveň.

---

## Voľba 4: `If / ElseIf / Else` — zložené podmienky

Keď podmienka závisí od **viacerých** parametrov alebo od signálov.

```xml
<If>
  <![CDATA[('Speed1'<10) && ('Speed2'<=10) && ('Zone'!=NULL) && ('Fclo'==FALSE)]]>
  <Line>
    <Item Type="const">SLOW</Item>
  </Line>
</If>
<ElseIf>
  <![CDATA[('Speed1'>10) && ('Speed2'>10) && ('Zone'==fine)]]>
  <Line>
    <Item Type="const">FAST</Item>
  </Line>
</ElseIf>
<Else>
  <Line>
    <Item Type="const">DEFAULT</Item>
  </Line>
</Else>
```

**Dve pravidlá zápisu:**
1. Podmienku obaľ do `<![CDATA[ … ]]>` — inak XML rozbijú znaky `<`, `>`, `&`.
2. Každý parameter obaľ apostrofmi: `'Speed1'`.

**Podporované operátory:**
`And`/`and`/`&` · `Or`/`or`/`|` · `Xor`/`xor` · `Not`/`not`/`!` ·
`==` · `<>`/`!=` · `<` `<=` `>` `>=` · `+` `-` `*` `/` ·
`TRUE`/`FALSE` · signály (v simulačnej vrstve) · zátvorky ·
`NULL` (iba pre dynamické parametre)

**Obmedzenia:**
- Vnorené `If` nie je povolené.
- Viacero `ElseIf` povolené je.
- `If` sa dá dať dovnútra riadka aj okolo riadkov.

---

## Výrazy (`expression`)

```xml
<Item Type="expression"><![CDATA['IntNum' * 60]]></Item>
```

Typické použitie: v UI zobraziť m/min, v download súbore mm/s.

**Výpočet nad download hodnotami:**
```xml
<Item Type="expression" Context="download"><![CDATA[('ParamA'+'ParamB')]]></Item>
```

**Zaokrúhlenie výsledku:**
```xml
<Item Type="expression" DigitsAfterPoint="3"><![CDATA[('WeldPointX'/25.4)]]></Item>
```

---

## Cielenie na konkrétnu vrstvu

Atribút `Representation` (`UI` / `simulation` / `download`) funguje v `Item`,
`If` aj `Switch`:

```xml
<Item Type="dynamicParameter" Representation="UI">Tool Nr</Item>

<If Representation="download">
  <![CDATA[(('Ipo Fr' == #BASE) && ('Tool Nr' != 90))]]>
  <Item Type="const">G80</Item>
</If>
```

---

## Špeciálne znaky v XML

| Zápis | Znak |
|---|---|
| `&lt;` | `<` |
| `&gt;` | `>` |
| `&amp;` | `&` |
| `&quot;` | `"` |
| `&apos;` | `'` |

Namiesto nečitateľného:
```xml
<Item Type="const">(&apos;Speed&apos; &gt;50) &amp;&amp; (&apos;Acceleration&apos; &lt;=30)</Item>
```
napíš:
```xml
<Item Type="const"><![CDATA[('Speed' >50) && ('Acceleration' <=30)]]></Item>
```
