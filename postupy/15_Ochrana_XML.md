# Postup 15 — Ochrana XML: šifrovanie, expirácia, zdieľanie

**Cieľ:** poslať konfigurácie dodávateľovi alebo integrátorovi bez toho, aby
si s nimi odovzdal aj celé know-how — a mať nad ich používaním kontrolu v čase.

---

## 1. Šifrovanie XML súborov

V dialógu **Customized Commands XML Checker** stlač **Encrypt** a vyber XML
súbory na zašifrovanie.

- Vzniknú súbory s príponou **`.XMLC`**.
- Tie sa dajú bezpečne posielať dodávateľom a subdodávateľom.
- Process Simulate / RobotExpert pracuje s XML aj XMLC **transparentne** —
  nie je potrebné nič prepínať.

⚠ **Nikdy nenechávaj v priečinku XML aj XMLC verziu toho istého súboru naraz.**
Manuál to výslovne zakazuje — správanie je nepredvídateľné a veľmi ťažko sa to
ladí (upravuješ jeden súbor a systém číta druhý).

Praktický postup: zdrojové XML si drž mimo inštalácie (napr. v repozitári alebo
zdieľanom priečinku) a do inštalácie nasadzuj len XMLC.

---

## 2. Report — ku ktorým verziám radičov XMLC patria

Po zašifrovaní už do súboru nenahliadneš. Tlačidlo **Report** v XML Checkeri
zobrazí zoznam verzií radičov, ktorých sa zašifrované XML týkajú.

Užitočné, keď máš na disku desiatky XMLC od rôznych dodávateľov a potrebuješ
zistiť, ktorý patrí kam.

---

## 3. Dátum expirácie

Ochrana proti neobmedzenému používaniu ukradnutých alebo „zapožičaných“
konfiguračných súborov. Do `<RobotController>` sa pridajú dva atribúty:

```xml
<RobotController Name="Abb-Rapid"
                 ExpirationDate="15/03/2015"
                 ExpirationMsg="The XML file has expired.">
```

| Atribút | Význam |
|---|---|
| `ExpirationDate` | Dátum, po ktorom XML prestane platiť (formát DD/MM/RRRR) |
| `ExpirationMsg` | Text chybovej správy, ktorá sa zobrazí po expirácii |

Po tomto dátume je XML neplatné a zobrazí sa chybová správa.

⭐ Typické použitie: pilotný projekt, skúšobná prevádzka u zákazníka, alebo
konfigurácia viazaná na trvanie zmluvy.

⚠ Nezabudni na vlastnú dokumentáciu — expirovaná konfigurácia sa v praxi
prejaví ako „zrazu nič nefunguje“ a bez poznámky sa príčina hľadá dlho.

---

## 4. Centrálne umiestnenie konfigurácií — `CustomizedPath`

Predvolene sa XML čítajú z inštalačného priečinka:
```
\Robotics\Olp\<controller name>\OlpConfiguration\*.xml
\Robotics\Olp\<controller name>\MotionConfiguration\*.xml
…
```

To znamená, že pri zmene konfigurácie treba obísť všetky stanice. Atribút
`CustomizedPath` v súbore **`rrs.xml`** presmeruje čítanie na sieťový priečinok:

```xml
<Controller Name="Kuka-Krc">
  <InstalledVersions>
    <Version Name="krc5.3_r01" CustomizedPath="\\jlhzsomebody\Kuka-Krc\Supplier_1">
      <ModuleName>C:\rrs_bin\rcs_krc1\krc5.3_r01\bin\rcskrc1_tune.exe</ModuleName>
    </Version>
  </InstalledVersions>
</Controller>
```

V tomto príklade robot s radičom Kuka-Krc verzie `krc5.3_r01` číta svoje XML
z uvedeného sieťového umiestnenia.

⭐ **Toto je najpraktickejšia vec z celej kapitoly**: konfigurácia sa udržiava na
jednom mieste, aktualizuje sa raz a platí pre všetkých užívateľov. Navyše sa dá
každému dodávateľovi vyhradiť vlastný podpriečinok (`Supplier_1`, `Supplier_2`…).

---

## 5. Sprievodné priečinky

Ikony, obrázky a pomocníka, na ktoré sa XML odkazuje, treba nasadiť spolu s ním:

| Priečinok | Obsah |
|---|---|
| `CustomizedIcons` | Ikony príkazov (atribút `Icon`) |
| `CustomizedPictures` | Obrázky v dialógoch (element `Picture`) |
| `CustomizedHelp` | Súbory pomocníka (element `Help`) |

⚠ Pri presune konfigurácie cez `CustomizedPath` alebo pri odovzdaní XMLC
dodávateľovi sa na tieto priečinky ľahko zabudne — príkazy potom fungujú, ale
dialógy sú „holé“ a bez pomocníka.

---

## Kontrolný zoznam pred odovzdaním konfigurácie

- [ ] XML prešlo cez **Check** bez chýb
- [ ] Príkazy overené cez **Show Layers**, simuláciu, download aj upload
- [ ] Nastavený `ExpirationDate` a `ExpirationMsg` (ak je to na mieste)
- [ ] Súbory zašifrované cez **Encrypt**
- [ ] Zdrojové `.XML` **odstránené** z cieľového priečinka (zostáva len `.XMLC`)
- [ ] Priložené `CustomizedIcons`, `CustomizedPictures`, `CustomizedHelp`
- [ ] Overené cez **Report**, že XMLC sedí na správnu verziu radiča
- [ ] Zdrojové XML archivované u seba — z XMLC sa už späť nedostaneš
