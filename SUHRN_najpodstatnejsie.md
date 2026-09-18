# Robotics Customized UI — výťah najpodstatnejších informácií

Zdroj: *Robotics Customized UI*, Siemens Digital Industries Software, 2022, 148 strán
(Process Simulate / RobotExpert, modul Tecnomatix Robotics).

---

## O čom celá príručka je

Príručka popisuje **jediný mechanizmus**: ako cez XML súbory rozšíriť robotické UI
v Process Simulate — pridať vlastné OLP príkazy, vlastné pohybové príkazy, vlastné
dátové typy, šablóny dráh a šablóny downloadu — tak, aby výstup presne zodpovedal
syntaxi konkrétneho robotického radiča (ABB, KUKA, Fanuc, Comau…).

**Žiadne programovanie. Všetko sú XML súbory** uložené na konkrétnych miestach
v inštalácii. Aplikácia ich načíta pri štarte do cache podľa radiča a verzie.

---

## Kľúčový koncept: tri vrstvy jedného príkazu

Toto je jadro celého systému. Každý vlastný príkaz existuje v troch podobách:

| Vrstva | Na čo slúži | Obmedzenie |
|---|---|---|
| `UILayer` | Čo vidí človek v Teach Pendante a Path Editore | **iba jeden riadok** |
| `SimulationLayer` | Čo sa vykoná pri simulácii (pohyby, signály, čakania) | viac riadkov |
| `DownloadLayer` | Čo sa zapíše do reálneho programového súboru robota | viac riadkov |

Ten istý parameter môže mať v každej vrstve inú reprezentáciu — napr. v UI „Open“,
v download súbore „2“ (atribút `DownloadRepresentation`).

**Download musí byť reverzibilný.** Čo sa dá zapísať (download), musí sa dať aj
spätne načítať (upload) — a práve na tom väčšina konfigurácií v praxi zlyháva.

---

## Kde sa čo ukladá

```
<PS installation>\eMPower\Robotics\Olp\<controller>\
    ├── OlpConfiguration\*.xml            → vlastné OLP príkazy
    ├── MotionConfiguration\*.xml         → vlastné pohybové príkazy
    ├── DataConfiguration\*.xml           → vlastné dátové typy (FDAT, PDAT, LDAT, XDAT)
    ├── PathTemplateConfiguration\*.xml   → šablóny robotických dráh
    └── DownloadTemplatesConfiguration\*.xml → šablóny download súborov

\Robotics\Olp\CustomizedIcons\      → ikony dialógov (spoločné pre všetky radiče)
\Robotics\Olp\CustomizedPictures\   → obrázky v rozbaľovacích zoznamoch
\Robotics\Olp\CustomizedHelp\       → online pomocník k dialógom
```

Názvy XML súborov sú ľubovoľné — načítajú sa všetky v priečinku.
Centralizácia pre celý tím: atribút `CustomizedPath` v `rrs.xml` presmeruje čítanie
na sieťový zdieľaný priečinok.

---

## Päť konfiguračných systémov

1. **OlpConfiguration** — vlastné procesné príkazy na lokáciách (otvor kliešte, pošli signál, zapni lepenie).
2. **MotionConfiguration** — kombinácia pohybu + procesu. Rieši sa cez dvojicu
   *typ lokácie* (VIA / WELD / Seam Start / Middle / End / Pick / Place) × *typ procesu*.
   Každá kombinácia má vlastný dialóg a vlastnú download vrstvu.
3. **DataConfiguration** — generovanie dátových deklarácií (KUKA `.DAT` vs `.SRC`).
   `RoboticParams` a `Aliases` definované tu sú **zdieľané** medzi OLP a Motion.
4. **PathTemplateConfiguration** — automatizácia celých dráh jedným kliknutím:
   pridaj nábeh/odbeh, nastav parametre, pridaj OLP príkazy, premenuj, zafarbi.
5. **DownloadTemplatesConfiguration** — hlavičky a štruktúra výsledného programového
   súboru + vlastné kľúčové slová (okrem pevných ako `<ProgName>`, `<Date>`, `<Body>`).

---

## Najdôležitejšie praktické pravidlá

### Jedinečnosť názvov
Názov každého robotického parametra aj každého OLP príkazu **musí byť jedinečný
naprieč všetkými XML súbormi daného radiča a verzie** — všetko ide do jednej cache.

### Nikdy nemeň názvy po nasadení
Zmena `Title` dialógu alebo názvu príkazu **rozbije väzbu na už vytvorené príkazy
v databáze** — nedajú sa ďalej editovať a simulačná/download vrstva sa stratí.
Riešenie: pridaj atribút `Id` so *starým* názvom a potom meň `Title`.

### Pravidlá pre upload (najčastejší zdroj problémov)
- Za každým parametrom typu `string`, `dynamic`, `object` musí nasledovať **konštanta**,
  alebo musí byť **posledný v riadku**.
- Pri `int`/`double`: prvý znak za hodnotou nesmie byť číslica.
- Ak nemáš konštantu, použi `UploadRegex="\w+"`.
- Jeden OLP príkaz = jedna inštrukcia (jeden fold pri KUKA, jeden `MOVE…ENDMOVE` pri Comau).
- Na konci riadka nesmie byť medzera.
- **Kuka Vkrc upload nie je podporovaný.**

### Desatinné čísla
V XML **vždy bodka**. V UI sa zobrazí podľa regionálneho nastavenia Windows (u nás čiarka).
V download a simulačnej vrstve vždy bodka.

---

## Čo väčšina ľudí prehliadne, a ušetrí to najviac práce

| Funkcia | Čo rieši |
|---|---|
| `Aliases` | Opakujúci sa blok XML sa definuje raz a odkazuje sa naň. Zásadne zmenší súbor. |
| `TemplateOlpCommands` | Jedna šablóna príkazu + argument namiesto 20 skoro rovnakých príkazov. |
| `DownloadRepresentation` | Nahradí dlhé switch/case bloky jediným atribútom. |
| `Hide` + `DynamicValue` | Dialóg sa sám prispôsobí — skryje nerelevantné polia, zmení rozsahy. |
| `Disabled` | Deaktivuje parameter podľa hodnoty iného parametra. |
| `Picture` v `ElmDef` | Pri výbere klieští/nástroja sa zobrazí ich fotka — výrazne menej omylov. |
| `Help` v `Dialog` | Vlastný online pomocník (HTML, video, URL) priamo v dialógu. |
| Path Template | Desiatky ručných úkonov na dráhe → jedno kliknutie. |

---

## Ladenie — tri nástroje, ktoré treba poznať

| Nástroj | Kde | Čo ukáže |
|---|---|---|
| **XML Checker** | ikona v PS/RobotExpert | Validita všetkých piatich konfiguračných sekcií |
| **Show Layers** | pravý klik v TP / stĺpec Customized Debug | UI, simulačnú a download podobu príkazu |
| **Upload Checker** | to isté menu | Aktuálne vs. načítané hodnoty každého parametra |
| **Upload Debugger** | to isté menu | Vloží sa problémový riadok programu → červeným sa označí, čo zlyhalo a prečo |
| **Upload and Download** | v Upload Debuggeri | Načíta a znova zapíše — odhalí „tichú“ nezhodu, keď upload prejde, ale obsah je iný |

---

## Ochrana know-how

- **Encrypt** v XML Checkeri → súbory s príponou `.XMLC`, bezpečne odovzdateľné dodávateľom.
  (Nikdy nenechávať v priečinku XML aj XMLC súčasne.)
- **Report** → zoznam verzií radičov, ku ktorým zašifrované súbory patria.
- `ExpirationDate` + `ExpirationMsg` v `<RobotController>` → XML po dátume prestane platiť.

---

## Simulačná vrstva — špeciálne príkazy

Nie sú dostupné z Teach Pendantu, fungujú vo všetkých simuláciách radičov:

- `# StartMove` — spustí pohyb a pokračuje **paralelne** s ním
- `# WaitReached` — počká na dokončenie pohybu (pozor: nie je to potvrdenie príchodu do bodu)
- `# Move` — `StartMove` + `WaitReached`
- `# ForceFullArrival` — vynúti zónu fine a úplný príchod (mení trajektóriu aj čas cyklu)

**Dôležité:** pri **vlastnej** zvarovej lokácii sa pohyb klieští **nesimuluje automaticky**
(na rozdiel od štandardnej). Ak ho chceš, musíš do simulačnej vrstvy dať `# Weld`.

---

## Limity, o ktoré sa človek potkne

- `UILayer` = **iba jeden riadok**.
- `Switch` nesmie byť vnorený — vždy len jedna úroveň.
- Vnorené `If` nie je povolené (viacero `ElseIf` áno).
- V zdieľanom dialógu nesmú byť dva OLP príkazy s rovnakým názvom parametra —
  a **Checker to nevie odhaliť**.
- `AdditionalDownloadLayer` funguje iba pre Kuka-Vkrc.
- V `UploadAdditionalDynamicParameters` nefungujú prepínače (switch).
- Pri Path Template na zloženej operácii musia mať všetky podoperácie **ten istý robot**.
- Parameter na úrovni operácie (`Level="operation"`) sa nedá nastaviť z UI.
