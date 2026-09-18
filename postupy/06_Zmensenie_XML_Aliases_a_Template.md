# Postup 06 — Zmenšenie XML: Aliases a Template príkazy

**Cieľ:** prestať kopírovať tie isté bloky. Oprava potom prebehne na jednom mieste.

---

## Aliases — opakovane použiteľný blok

Definuj raz:

```xml
<Aliases>
  <Alias Name="BitOrder">
    <Switch ParamName="ordre1">
      <Case Value="0"></Case>
      <Case Value="1"><Item Type="const">\O1</Item></Case>
    </Switch>
    <Switch ParamName="ordre2">
      <Case Value="0"></Case>
      <Case Value="1"><Item Type="const">\O2</Item></Case>
    </Switch>
    <!-- ordre3 … ordre6 rovnako -->
  </Alias>
</Aliases>
```

Použi kdekoľvek v ľubovoľnej vrstve:

```xml
<Command Name="PLC_Release">
  <RoboticParamRef>...</RoboticParamRef>
  <Layers>
    <UILayer>
      <Line><Alias>BitOrder</Alias></Line>
    </UILayer>
    <DownloadLayer>
      <Line>
        <Item Type="const">ORDER</Item>
        <Alias>BitOrder</Alias>
        <Item Type="const">;</Item>
      </Line>
    </DownloadLayer>
  </Layers>
</Command>
```

Sekcia `<Aliases>` sa umiestňuje za `<RoboticParams>`.

> **Tip:** aliasy (aj `RoboticParams`) definované v **DataConfiguration** XML sú
> zdieľané medzi OLP aj Motion konfiguráciou — definuj ich tam raz pre celý radič
> (→ postup 10).

---

## TemplateCommand — jeden príkaz, veľa variantov

Namiesto 20 skoro rovnakých príkazov `SET_SEGMENT_1`, `SET_SEGMENT_2`, …
napíš jeden s argumentom:

```xml
<TemplateOlpCommands>
  <TemplateCommand Name="SET_SEGMENT">
    <RoboticParamRef>
      <Param>ProgNr1</Param>
    </RoboticParamRef>

    <CommandName>
      <Item Type="const">SET_SEGMENT_</Item>
      <Item Type="const" Arg="SegmentNumber"/>
    </CommandName>

    <Layers>
      <UILayer>
        <Line>
          <Item Type="const">SET SEGMENT(</Item>
          <Item Type="const" Arg="SegmentNumber"></Item>
          <Item Type="const">);</Item>
        </Line>
      </UILayer>
      <SimulationLayer></SimulationLayer>
      <DownloadLayer>
        <Line>
          <Item Type="const">SET SEGMENT(</Item>
          <Item Type="const" Arg="SegmentNumber"></Item>
          <Item Type="const">(</Item>
          <Item Type="parameter">ProgNr1</Item>
          <Item Type="const">);</Item>
        </Line>
      </DownloadLayer>
    </Layers>
  </TemplateCommand>
</TemplateOlpCommands>
```

Hodnotu argumentu dodá dialóg:

```xml
<OlpDialogs>
  <Dialog Title="SET_SEGMENT|SET_SEGMENT1">
    <OlpCommandRef><OlpCommand SegmentNumber="1">SET_SEGMENT</OlpCommand></OlpCommandRef>
  </Dialog>
  <Dialog Title="SET_SEGMENT|SET_SEGMENT2">
    <OlpCommandRef><OlpCommand SegmentNumber="2">SET_SEGMENT</OlpCommand></OlpCommandRef>
  </Dialog>
  <Dialog Title="SET_SEGMENT|SET_SEGMENT3">
    <OlpCommandRef><OlpCommand SegmentNumber="3">SET_SEGMENT</OlpCommand></OlpCommandRef>
  </Dialog>
</OlpDialogs>
```

**Kľúčové časti:**
- `<CommandName>` — z konštánt a argumentov sa poskladá **jedinečný názov** príkazu.
- `<Item Type="const" Arg="…"/>` — miesto, kam sa argument dosadí.
- Hodnota argumentu sa uvádza ako atribút na `<OlpCommand>` v dialógu.
