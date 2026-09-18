# Siemens Tecnomatix — Robotics Customized UI (SK)

Slovenský preklad a spracovanie manuálu **Robotics Customized UI** (Siemens
Digital Industries Software, 2022, 148 strán) pre Process Simulate / RobotExpert.

Manuál popisuje prispôsobenie robotického UI cez XML: vlastné OLP príkazy,
pohybové príkazy, dátové typy, šablóny dráh a šablóny download súborov.

## Obsah repozitára

| Súbor / priečinok | Čo to je |
|---|---|
| `index.html` | **Celý web v jednom súbore** — preklad, súhrn aj všetkých 15 postupov, obrázky vložené priamo vnútri. Funguje aj offline, aj ako GitHub Pages. |
| `Robotics_Customized_UI_SK.html` | Kompletný preklad všetkých 9 kapitol vrátane 94 obrázkov. Otvor v prehliadači. |
| `SUHRN_najpodstatnejsie.md` | Výťah kľúčových informácií — koncept troch vrstiev, štruktúra priečinkov, prehliadané funkcie, limity. |
| `postupy/` | 15 samostatných postupov, každý čitateľný nezávisle. Index v `00_PREHLAD_postupov.md`. |
| `obrazky/` | 94 obrázkov vyextrahovaných z originálu, pomenované podľa strany (`strNNN_M.png`). |

## Postupy

| # | Postup |
|---|---|
| 01 | Vytvorenie vlastného OLP príkazu od nuly |
| 02 | Typy parametrov a ovládacích prvkov |
| 03 | Dialógy: menu, ikony, popisy, pomocník |
| 04 | Logika: switch/case, if-elseif-else, výrazy |
| 05 | Inteligentné dialógy: Hide, DynamicValue, Disabled |
| 06 | Zmenšenie XML: Aliases a Template príkazy |
| 07 | Vlastný pohybový príkaz (MotionConfiguration) |
| 08 | Simulačná vrstva |
| 09 | Upload: pravidlá a riešenie problémov |
| 10 | Vlastné dátové typy (DataConfiguration) |
| 11 | Šablóny dráh: parametre a OLP príkazy |
| 12 | Šablóny dráh: pridávanie a úprava lokácií |
| 13 | Šablóny download súborov a kľúčové slová |
| 14 | Ladenie a kontrola |
| 15 | Ochrana XML: šifrovanie, expirácia, zdieľanie |

## Poznámka k prekladu

XML kód (názvy elementov, atribútov, hodnôt) je ponechaný **v origináli** —
prekladať sa nesmie, inak konfigurácia prestane fungovať. Preložený je
vysvetľujúci text, komentáre a popisy.

## Zdroj

Siemens Digital Industries Software, *Robotics Customized UI*, © 2022 Siemens.
Preklad je pracovná pomôcka; v prípade nezrovnalostí platí originálny manuál.

## Publikovanie ako web (GitHub Pages)

Settings → Pages → Source: *Deploy from a branch* → branch `main`, folder `/ (root)` → Save.
Stránka nabehne na `https://<meno>.github.io/siemens/` — `index.html` je samostatný,
takže žiadne ďalšie nastavovanie netreba.

Pozor: GitHub Pages na privátnom repozitári vyžaduje platený plán.
