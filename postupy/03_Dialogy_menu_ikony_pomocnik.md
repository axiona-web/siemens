# Postup 03 — Dialógy: menu, ikony, popisy, pomocník

**Cieľ:** usporiadať vlastné príkazy do prehľadného menu a spraviť ich
zrozumiteľné aj pre človeka, ktorý ich nepísal.

---

## Základný dialóg

```xml
<OlpDialogs>
  <Dialog Title="MyTest|Customized|ChooseLocation"
          Description="Please select a location"
          Icon="gripperOp.ico"
          Help="MyTest.html">
    <OlpCommandRef>
      <OlpCommand>ChooseLocation</OlpCommand>
    </OlpCommandRef>
  </Dialog>
</OlpDialogs>
```

| Atribút | Význam |
|---|---|
| `Title` (povinný) | Cesta v menu Teach Pendantu. Každý `\|` = nová úroveň podmenu. |
| `Description` | Krátky text v hlavičke dialógu |
| `Icon` | Ikona dialógu, relatívna cesta od `\Robotics\Olp\CustomizedIcons\` |
| `Help` | Súbor pomocníka, relatívna cesta od `\Robotics\Olp\CustomizedHelp\` |
| `Id` | Skryté ID pre databázu — umožňuje premenovanie (viď nižšie) |

---

## Ikony

Vytvor priečinok `\Robotics\Olp\CustomizedIcons\` (slúži **všetkým radičom**,
pre OLP aj pohybové dialógy). Vnútri si môžeš robiť podpriečinky:

```
\Robotics\Olp\CustomizedIcons\weld\myIcon.ico   →   Icon="weld\myIcon.ico"
```

---

## Online pomocník v dialógu

Pridaním `Help="…"` sa v ľavom dolnom rohu dialógu automaticky objaví tlačidlo
**Help**. Podporuje čokoľvek — HTML, PDF, video, audio.

**Odkaz na URL:** vlož do priečinka zástupcu (`.url`) a v XML uveď jeho názov
s príponou `.url`. Takto sa dá všetko držať na jednom serveri a v XML mať
len odkazy.

Rieši to reálny problém: dialógy píše pár ľudí, používajú ich desiatky.

---

## Oddeľovač v menu

```xml
<Dialog Title="PL2Glue|-">
  <OlpCommandRef/>
</Dialog>
```
Pomlčka na konci názvu = vodorovná čiara v menu. Ten istý názov sa môže
použiť viackrát.

---

## Viac príkazov v jednom dialógu

```xml
<Dialog Title="Glue">
  <OlpCommandRef>
    <OlpCommand>OLP_1</OlpCommand>
    <OlpCommand>OLP_2</OlpCommand>
    <OlpCommand>OLP_3</OlpCommand>
    <OlpCommand>OLP_1</OlpCommand>
  </OlpCommandRef>
</Dialog>
```

Užívateľ otvorí jeden dialóg, vyplní všetko naraz, po OK vzniknú **zoskupené**
príkazy. Rozdeliť ich vie cez kontextové menu → **Ungroup**.

**Podmienka:** žiadny parameter sa nesmie v dialógu objaviť dvakrát.
Checker toto neoverí — musíš si dať pozor sám.

---

## Premenovanie dialógu bez straty väzby ⚠

Toto je najčastejšia „tichá“ chyba pri údržbe.

Zmena `Title` **rozbije väzbu** na už vytvorené príkazy v databáze — nedajú sa
ďalej editovať a download/simulačné vrstvy sa stratia.

**Správny postup:** pri prvom premenovaní pridaj `Id` so *starým* názvom:

```xml
<Dialog Id="Customized|Supplier1|Robot Job1"
        Title="Customized|Supplier1|Robot|Job1">
```

Pri ďalších premenovaniach už meň **iba `Title`**, `Id` nechaj tak:

```xml
<Dialog Id="Customized|Supplier1|Robot Job1"
        Title="Customized|Supplier1|Robot|Job2">
```

Ak `Id` nie je uvedené, ako `Id` sa použije `Title`.

> Hodnota `Id` sa používa aj v šablónach dráh ako `CustomizedDialogName` (→ postup 11).
