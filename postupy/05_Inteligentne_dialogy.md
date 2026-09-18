# Postup 05 — Inteligentné dialógy: Hide, DynamicValue, Disabled

**Cieľ:** dialóg sa má sám prispôsobovať tomu, čo užívateľ vyberie —
nerelevantné polia zmiznú, rozsahy sa zmenia.

---

## `Disabled` — zošedivenie parametra

Deaktivovaný parameter sa pri kliknutí na OK **neberie do úvahy**.

```xml
<Command Name="PLC_Release">
<RoboticParamRef>
  <Param Optional="true" Name="order1">
    <Disabled>
      <!-- Deaktivuj order1, keď order3 nie je vyplnený -->
      <![CDATA[('order3' == NULL)]]>
    </Disabled>
  </Param>
  <Param>SDelay | Sync</Param>
  <Param>order2</Param>
  <Param Optional="true" Name="order3">
    <Disabled>
      <![CDATA[('SDelay | Sync'==SDelay) && ('order4' >5)]]>
    </Disabled>
  </Param>
  <Param>order4</Param>
</RoboticParamRef>
```

Sekcia `<Disabled>` sa píše **v rámci `RoboticParamRef`** konkrétneho príkazu,
nie v globálnej definícii parametra.

---

## `Hide` — úplné skrytie parametra

Sekcia `<Hide>` sa píše **v globálnej definícii parametra** v `<RoboticParams>`.

```xml
<Param Name="Angle" ValueType="string">
  <Hide><![CDATA[('ActSpeed' == 0)]]></Hide>
  ...
</Param>
```

---

## `DynamicValue` — hodnoty a rozsahy podľa kontextu

**Pre rozbaľovací zoznam** — ktoré položky sa vôbec ponúknu:

```xml
<Param Name="Angle" ValueType="string">
  <Hide><![CDATA[('ActSpeed' == 0)]]></Hide>
  <ComboDef>
    <DynamicValue Value="30"><![CDATA[('ActSpeed' == 1)]]></DynamicValue>
    <DynamicValue Value="-30"><![CDATA[('ActSpeed' == 1)]]></DynamicValue>
    <DynamicValue Value="60"><![CDATA[('ActSpeed' == 2)]]></DynamicValue>
    <DynamicValue Value="-60"><![CDATA[('ActSpeed' == 2)]]></DynamicValue>
  </ComboDef>
</Param>
```

**Pre číselné pole** — aký rozsah a predvolená hodnota platí:

```xml
<Param Name="FeedNo" ValueType="int" MaxVal="4" MinVal="0">
  <Hide><![CDATA[('Angle' == "")]]></Hide>
  <DynamicValue MaxVal="200" MinVal="20" Default="100">
    <![CDATA[('Angle'> 0)]]>
  </DynamicValue>
  <DynamicValue MaxVal="-20" MinVal="-200" Default="-100">
    <![CDATA[('Angle' <0)]]>
  </DynamicValue>
</Param>
```

---

## Reťazenie podmienok

Podmienky sa dajú vrstviť — výsledok je dialóg, ktorý sa postupne odkrýva:

```xml
<Param Name="Part" ValueType="double" MaxVal="100.00" MinVal="0.00">
  <Hide><![CDATA[(('FeedNo'=="")) || ('Angle'> 0)]]></Hide>
</Param>
```

Typický priebeh:
1. `ActSpeed = 0` → viditeľný je iba jeden prvok
2. `ActSpeed = 1` → objaví sa `Angle` s hodnotami 30 / -30
3. kladný `Angle` → objaví sa `FeedNo` s rozsahom [20, 200]
4. záporný `Angle` → rozsahy sa obrátia, pribudnú `Part`, `EqualPr`, `Advance_Gun_Time`

---

## Obmedzenia ⚠

1. **Všetky parametre spomenuté v `Disabled` musia byť v tom istom dialógu.**
2. **V zdieľanom dialógu nesmú byť dva OLP príkazy s rovnakým názvom parametra.**
   XML Checker toto **nevie** odhaliť — prejaví sa to až zvláštnym správaním dialógu.
