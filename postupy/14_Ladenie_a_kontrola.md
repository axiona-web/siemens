# Postup 14 — Ladenie a kontrola

**Cieľ:** keď niečo nefunguje, systematicky zistiť **kde** — v XML, v simulačnej
vrstve, v download vrstve, alebo pri uploade.

---

## 1. XML Checker — prvá zastávka

V Process Simulate / RobotExpert klikni na ikonu **Customized Commands XML
Checker**. Dá sa spúšťať opakovane po každej oprave XML a **pracovať pritom
paralelne s Teach Pendantom** — netreba reštartovať PS.

| Tlačidlo | Čo robí |
|---|---|
| **Check** | Načíta XML v definovanom poradí a vypíše chyby do poľa Result |
| **Help** | Otvorí manuál Robotics Customized UI |
| **Report** | Zobrazí zoznam verzií radičov, ktorých sa šifrované XML týkajú |
| **Encrypt** | Zašifruje vybrané XML súbory (→ postup 15) |

⭐ Checker odhalí najmä **logické** problémy, niekedy aj syntaktické. Nie je to
plnohodnotný XML validátor — na hrubé syntaktické chyby použi ľubovoľný XML
editor ešte predtým.

---

## 2. Show Layers — čo vlastne príkaz generuje

**Pre OLP príkazy:**
otvor **Teach Pendant** → pravý klik na príslušný OLP príkaz → **Show Layers**.

**Pre pohybové (motion) príkazy:**
v **Path Editore** si pridaj stĺpec **Customized Debug** a klikni na príslušnú
bunku.

Otvorí sa dialóg so **simulačnou vrstvou** a **download vrstvou** vybraného
príkazu — presne tak, ako ich systém zostavil. Tu sa okamžite ukáže, či sa
napríklad `<If>` vyhodnotil inak, než si čakal, alebo či parameter zostal prázdny.

---

## 3. Upload Checker — porovnanie hodnôt

Otvorí dialóg s **aktuálnymi** a **nahratými** hodnotami každého robotického aj
dynamického parametra vybraného vlastného príkazu.

Použi vtedy, keď sa príkaz po uploade rozpozná, ale hodnoty sedia zle.

---

## 4. Upload Debugger — keď sa príkaz vôbec nerozpozná

Postup:

1. Použi existujúcu lokáciu (alebo vytvor dummy lokáciu) v štúdii.
2. Presuň na ňu problematický vlastný OLP príkaz s príslušnými parametrami.
3. Otvorí sa dialóg Debuggeru — zobrazí prípadné interné problémy
   s download/upload (vo väčšine prípadov žiadne nie sú).
4. **Vlož do dialógu problematické riadky priamo z pôvodného programu robota**
   a klikni na **Re-Upload**.
5. Riadky, ktoré spôsobili zlyhanie uploadu, sa zobrazia **červeno** aj
   s dôvodom zlyhania.

Obsah dialógu sa dá editovať a Re-Upload spustiť znova — dá sa tak metódou
pokus-omyl zistiť, ktorý znak alebo medzera príkaz rozbíja.

---

## 5. Tlačidlo Upload and Download — tichá nezhoda ⭐

Toto je ladiaci nástroj na najzákernejšiu triedu chýb: príkaz sa nahrá aj stiahne
**bez chyby**, ale výsledok nie je ten istý.

Tlačidlo vezme obsah v Upload Debuggeri, nahrá ho, znova stiahne a porovná
pôvodný obsah s výsledkom (zobrazí sa v spodnej časti dialógu).
**Každá nezhoda sa zvýrazní červeno.**

### Príklad, kde to nastane

```xml
<Param Name="GlueTechVersion" ValueType="string">
  <ComboDef>
    <ElmDef>1.2.1</ElmDef>
    <ElmDef>2.3.2</ElmDef>
  </ComboDef>
</Param>
…
<DownloadLayer>
  <Line>
    <Item Type="const">;FOLD</Item>
    <Item Type="parameter">GlueTechVersion</Item>
    <Item Type="const">Check PrePressure =</Item>
    <If>
      <![CDATA['GlueTechVersion'==1.2.1]]>
      <Item Type="const">Yes</Item>
    </If>
    <Else>
      <Item Type="const">No</Item>
    </Else>
  </Line>
  <Line>
    <Item Type="const">;ENDFOLD</Item>
  </Line>
</DownloadLayer>
```

Hodnota parametra je v download riadku zapísaná, takže upload ju prečíta
správne — ale text `Yes`/`No`, ktorý z nej vznikol podmienkou, sa pri uploade
nedá spätne overiť. Ak by download vrstva hodnotu parametra **nevypisovala**,
upload by ju nemal odkiaľ získať a po opätovnom downloade by vyšlo niečo iné.

---

## Odporúčané poradie ladenia

1. **XML Checker → Check** — chyba v XML? Oprav a spusti znova.
2. **Show Layers** — generujú sa vrstvy tak, ako čakáš?
3. **Simulácia** — správa sa príkaz v simulácii správne? (→ postup 08)
4. **Download** — je súbor taký, aký má byť?
5. **Upload Checker** — sedia hodnoty parametrov po načítaní?
6. **Upload Debugger + Re-Upload** — ktorý riadok sa nerozpozná a prečo?
7. **Upload and Download** — vráti sa po celom cykle to isté?

---

## Na čo si dať pozor

1. Checker spúšťaj po **každej** úprave XML — chyby sa reťazia a neskôr sa ťažko
   lokalizujú.
2. Ak sú v priečinku súbory XML aj XMLC naraz, správanie je nepredvídateľné
   (→ postup 15).
3. Upload Debugger potrebuje **reálne riadky z programu robota**, nie tie, ktoré
   vygenerovalo PS — inak testuješ niečo iné, než ti padá.
4. Pravidlá uploadu (koncová konštanta, jedinečnosť, poradie) → **postup 09**.
