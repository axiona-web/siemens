# Postup 13 — Šablóny download súborov a kľúčové slová

**Cieľ:** aby vygenerovaný programový súbor robota mal poriadnu hlavičku a pätu —
údaje o závode, linke, štýle karosérie, verzii, autorovi — a aby sa šablóna dala
priradiť nielen robotovi, ale aj **konkrétnej operácii**.

**Kam súbory patria:**
```
<installation dir>\Robotics\Olp\<controller>\DownloadTemplatesConfiguration\*.xml
```
Definície parametrov, ktoré sa v šablóne používajú:
```
<installation dir>\Robotics\Olp\<controller>\MotionConfiguration\*.xml
```

---

## Čo to rieši

Dve veci naraz:

1. **Priradenie šablóny download súboru k robotickým operáciám** — nielen
   k robotom, ako to bolo predtým.
2. **Vloženie vlastných údajov z Process Simulate** do vygenerovaného súboru —
   nad rámec pevne zabudovaných kľúčových slov.

Užívateľ ich nastavuje v dialógu **Set Template and Keywords**, ktorý sa
generuje z XML súborov opísaných nižšie.

---

## Postup

1. V `MotionConfiguration\*.xml` **definuj parametre**, ktoré chceš vedieť
   nastavovať (napr. `Plant`, `Line`, `Style`, `Zone`, `Author`) — rovnakou
   syntaxou ako pri bežných robotických parametroch (`<Param Name="…"
   ValueType="…">`, prípadne s `<ComboDef>` pre výber zo zoznamu).
2. V `DownloadTemplatesConfiguration\*.xml` **vytvor šablónu**, ktorá tieto
   parametre používa ako kľúčové slová spolu s pevnými kľúčovými slovami.
3. V Process Simulate otvor **Set Template and Keywords**, priraď šablónu
   robotovi alebo operácii a vyplň hodnoty.
4. Spusti download — hodnoty sa dosadia do výsledného súboru.

⚠ Kľúčové slovo `<Body>` je miesto, kam sa vloží samotné telo programu.
Bez neho by šablóna vygenerovala len hlavičku.

---

## Pevne zabudované kľúčové slová

| Kľúčové slovo | Čo vloží |
|---|---|
| `<ProgName>` | Názov stiahnutého programu alebo operácie |
| `<UserName>` | Prihlasovacie meno |
| `<FileName>` | Názov vygenerovaného súboru **s** príponou |
| `<FileBaseName>` | To isté **bez** prípony |
| `<Date>` | Dátum generovania súboru |
| `<Time>` | Čas generovania súboru |
| `<Study>` | Názov štúdie |
| `<RobotName>` | Názov robota |
| `<TecnomatixSoftware>` | Napr. „Process Simulate 9.1.2.1 on eMS“ |
| `<TecnomatixVersion>` | Napr. 9.1.2.1, 10.1 |
| `<TecnomatixPlatform>` | eMS, Teamcenter, Disconnected |
| `<TecnomatixControllerVersion>` | Verzia teach pendantu |
| `<Body>` | **Hlavná časť programu** |
| `<ControllerVersion>` | Ako je zobrazené v Robot Setup |
| `<RcsVersion>` | Ako je zobrazené v Robot Setup |
| `<ManipulatorType>` | Ako je zobrazené v Robot Setup |

---

## Typické využitie

- Hlavička s dátumom, autorom a verziou softvéru → dohľadateľnosť, kto a čím
  program vygeneroval.
- Údaje o závode, linke a type karosérie → zákazník ich často vyžaduje
  normou alebo internou smernicou.
- Sledovanie verzie radiča (`<ControllerVersion>`, `<RcsVersion>`) →
  pri neskoršom hľadaní nezhôd medzi PS a reálnym radičom.
- Odlišná šablóna pre rôzne typy operácií (zváranie vs. lepenie) —
  presne preto pribudlo priradenie šablóny k operácii, nielen k robotovi.

---

## Na čo si dať pozor

1. Parametre sa definujú v **MotionConfiguration**, šablóna v
   **DownloadTemplatesConfiguration** — sú to dva rôzne priečinky a je ľahké to
   zameniť.
2. Preklep v názve kľúčového slova sa nezobrazí ako chyba — v súbore jednoducho
   zostane surový text. Po prvej zmene šablóny si vždy pozri vygenerovaný súbor.
3. Pevné kľúčové slová sa píšu presne tak, ako sú v tabuľke, vrátane veľkých
   písmen.
4. Šablóna priradená operácii má prednosť pred šablónou robota — pri ladení
   skontroluj obe priradenia v dialógu Set Template and Keywords.
5. Kontrola XML → **postup 14**.
