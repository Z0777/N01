# procesos01 · Pasos 10, 11 y 12 — control por control (sin YAML)

Todo en **sintaxis española**: separador `;` · encadenar `;;` · decimales con `,`.
Los formatos numéricos van con `[$-en-US]` para que el punto/coma de miles no se confunda con el separador (igual que ya usas en `btnBuildCarga`).

**Cómo leer cada bloque:**

- **Control** = nombre exacto que debes escribir en el panel de árbol (F2 sobre el control para renombrar).
- **Tipo** = qué insertar desde la cinta **Insertar**.
- **Ubicación** = dentro de qué contenedor va y en qué posición.
- Debajo, la tabla **Propiedad → Fórmula** (todas las que hay que cambiar; las que no aparecen se dejan como vienen).

---

# PASO 10 · Popup "Agregar SETs" — mismo SET varias veces

Contenedor existente: `conPopupAddSetsExport` → `Container124`.
Cada toque en una tarjeta **agrega una línea** (una caja). Abajo se ve el carrito y se pueden quitar líneas.

## 10.1 Controles existentes que cambian

### Control: `btnSelSetExp` (ya existe, dentro de `galSetsAddExp`)

| Propiedad | Fórmula |
|---|---|
| OnSelect | ver abajo |

```
Collect(colSetsExpNuevos;
    {
        Linea: Coalesce(Max(colSetsExpNuevos; Linea); 0) + 1;
        SetID: ThisItem.ID;
        SetName: ThisItem.SetName;
        Crop: ThisItem.Crop.Value;
        Program: ThisItem.Program.Value;
        Generation: ThisItem.Generation.Value;
        MCT: ThisItem.MCT.Value;
        TrialIntent: ThisItem.TrialIntent.Value;
        OperationType: ThisItem.OperationType.Value;
        VCR: ThisItem.VCR;
        Field: ThisItem.Field;
        Area: ThisItem.Area.Value;
        Season: Text(ThisItem.Season);
        Rank: ThisItem.Rank;
        Greenhouse: ThisItem.Greenhouse;
        Plantas: Coalesce(ThisItem.PlantsTransplanted; 0);
        EsManual: false
    }
);;
Notify(ThisItem.SetName & " · " & CountRows(Filter(colSetsExpNuevos; SetID = ThisItem.ID)) & " línea(s)"; NotificationType.Information; 1000)
```

### Control: `btnGuardarSetsExport` (ya existe, dentro de `Container124`)

| Propiedad | Fórmula |
|---|---|
| Text | `"Agregar " & CountRows(colSetsExpNuevos) & " línea(s)"` |
| DisplayMode | `If(CountRows(colSetsExpNuevos) = 0 Or IsBlank(varReqExportSel.Request_ID); DisplayMode.Disabled; DisplayMode.Edit)` |
| OnSelect | ver abajo |

```
ClearCollect(colGLLineas;
    ForAll(Sequence(CountRows(colSetsExpNuevos)) As N;
        With({S: Index(colSetsExpNuevos; N.Value)};
            {
                Linea: N.Value;
                AsignID: varReqExportSel.Request_ID;
                SetID: S.SetID;
                SetName: S.SetName;
                Crop: S.Crop;
                Program: S.Program;
                Generation: S.Generation;
                MCT: S.MCT;
                TrialIntent: S.TrialIntent;
                OperationType: S.OperationType;
                VCR: S.VCR;
                Field: S.Field;
                Area: S.Area;
                Season: S.Season;
                Rank: S.Rank;
                Greenhouse: S.Greenhouse;
                Plantas: S.Plantas;
                Packets: "";
                PesoNeto: "";
                Treatment: "";
                Comentario: ""
            }
        )
    )
);;
Set(varGLModo; "popup");;
Select(btnGuardarLineasExp)
```

> Este guardado ya **no** escribe `Status: "REALIZADO"`, `'Mother plants Lab report'`, `'Report type'` ni `ResponsableReporte`. Esos campos marcaban como reporte de laboratorio cualquier SET agregado a exportación y lo hacían aparecer en Plant health.

---

## 10.2 Control nuevo: contador en la tarjeta

**Control:** `lblCntSetExp`
**Tipo:** Insertar → Texto → **Etiqueta de texto**
**Ubicación:** dentro de `Container125` (la tarjeta de la galería `galSetsAddExp`), como último hijo, debajo de `TextInput16`.

| Propiedad | Fórmula |
|---|---|
| Text | `With({n: CountRows(Filter(colSetsExpNuevos; SetID = ThisItem.ID))}; If(n > 0; "✔ " & n & " línea(s)"; ""))` |
| Align | `Align.Center` |
| AlignInContainer | `AlignInContainer.Stretch` |
| Color | `RGBA(47; 158; 95; 1)` |
| FontWeight | `FontWeight.Bold` |
| Size | `10` |
| Height | `20` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |

---

## 10.3 Control nuevo: título del carrito

**Control:** `lblCarritoSetsExp`
**Tipo:** Insertar → Texto → **Etiqueta de texto**
**Ubicación:** dentro de `Container124`, **encima de** `btnGuardarSetsExport` (arrástralo en el árbol para que quede después de `galSetsAddExp`).

| Propiedad | Fórmula |
|---|---|
| Text | `"Líneas por agregar: " & CountRows(colSetsExpNuevos) & "  ·  " & CountRows(Distinct(colSetsExpNuevos; SetName)) & " SET(s) distintos"` |
| Align | `Align.Center` |
| AlignInContainer | `AlignInContainer.Stretch` |
| Color | `RGBA(12; 56; 76; 1)` |
| FontWeight | `FontWeight.Semibold` |
| Size | `11` |
| Height | `26` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |

---

## 10.4 Control nuevo: galería del carrito

**Control:** `galSetsExpCarrito`
**Tipo:** Insertar → **Galería vertical en blanco**
**Ubicación:** dentro de `Container124`, justo debajo de `lblCarritoSetsExp`.

| Propiedad | Fórmula |
|---|---|
| Items | `Sort(colSetsExpNuevos; SetName; SortOrder.Ascending)` |
| WrapCount | `3` |
| TemplateSize | `30` |
| TemplatePadding | `2` |
| Height | `110` |
| FillPortions | `0` |
| Width | `680` |
| ShowScrollbar | `true` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |

### Hijo 1 — **Control:** `lblCarritoLinea` · Tipo: **Etiqueta de texto** (dentro de la plantilla de `galSetsExpCarrito`)

| Propiedad | Fórmula |
|---|---|
| Text | `ThisItem.SetName & "  ·  línea " & ThisItem.Linea & If(ThisItem.EsManual; "  (manual)"; "")` |
| Fill | `RGBA(232; 245; 236; 1)` |
| Size | `10` |
| PaddingLeft | `8` |
| Height | `Parent.TemplateHeight` |
| Width | `Parent.TemplateWidth - 30` |
| X | `0` |
| Y | `0` |

### Hijo 2 — **Control:** `icoQuitarLinea` · Tipo: Insertar → Iconos → **Icono moderno** (dentro de la misma plantilla)

| Propiedad | Fórmula |
|---|---|
| Icon | `"Dismiss"` |
| IconColor | `RGBA(215; 58; 60; 1)` |
| OnSelect | `Remove(colSetsExpNuevos; ThisItem)` |
| Height | `Parent.TemplateHeight` |
| Width | `28` |
| X | `Parent.TemplateWidth - 30` |
| Y | `0` |

---

## 10.5 Control nuevo: alta manual (contenedor)

**Control:** `conSetManualExp`
**Tipo:** Insertar → Diseño → **Contenedor de diseño horizontal**
**Ubicación:** dentro de `Container124`, entre `galSetsExpCarrito` y `btnGuardarSetsExport`.

| Propiedad | Fórmula |
|---|---|
| Fill | `RGBA(251; 241; 220; 1)` |
| Height | `48` |
| FillPortions | `0` |
| Width | `680` |
| LayoutAlignItems | `LayoutAlignItems.Center` |
| LayoutJustifyContent | `LayoutJustifyContent.Center` |
| LayoutGap | `8` |
| PaddingTop | `6` |
| PaddingBottom | `6` |
| PaddingLeft | `8` |
| PaddingRight | `8` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |

### Hijo 1 — **Control:** `txtSetManualExp` · Tipo: Insertar → Entrada → **Entrada de texto (clásica)**

| Propiedad | Fórmula |
|---|---|
| Default | `""` |
| HintText | `"SET manual (si no aparece arriba)"` |
| FillPortions | `2` |
| Height | `32` |
| PaddingLeft | `6` |
| Size | `11` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |

### Hijo 2 — **Control:** `cmbCropManualExp` · Tipo: Insertar → Entrada → **Cuadro combinado (moderno)**

| Propiedad | Fórmula |
|---|---|
| Items | `["Tomato"; "Pepper"; "Melon"; "Watermelon"; "Cucumber"]` |
| ItemDisplayText | `ThisItem.Value` |
| InputTextPlaceholder | `"Crop"` |
| SelectMultiple | `false` |
| FillPortions | `1` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |

### Hijo 3 — **Control:** `btnAddManualExp` · Tipo: Insertar → **Botón (clásico)**

| Propiedad | Fórmula |
|---|---|
| Text | `"+ Agregar manual"` |
| Fill | `RGBA(232; 169; 60; 1)` |
| Color | `RGBA(255; 255; 255; 1)` |
| Size | `11` |
| Height | `32` |
| DisplayMode | `If(IsBlank(Trim(txtSetManualExp.Text)); DisplayMode.Disabled; DisplayMode.Edit)` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |
| OnSelect | ver abajo |

```
With(
    {_n: Trim(txtSetManualExp.Text); _d: LookUp(DATASUMMARY1; SetName = Trim(txtSetManualExp.Text))};
    If(
        IsBlank(_d.ID) And IsBlank(cmbCropManualExp.Selected.Value);
        Notify("Ese SET no está en DATASUMMARY1: selecciona el Crop."; NotificationType.Warning);

        Collect(colSetsExpNuevos;
            {
                Linea: Coalesce(Max(colSetsExpNuevos; Linea); 0) + 1;
                SetID: Coalesce(_d.ID; -1 * (CountRows(colSetsExpNuevos) + 1));
                SetName: _n;
                Crop: Coalesce(_d.Crop.Value; cmbCropManualExp.Selected.Value);
                Program: Coalesce(_d.Program.Value; varReqExportSel.Program; "NA");
                Generation: Coalesce(_d.Generation.Value; "NA");
                MCT: Coalesce(_d.MCT.Value; "NA");
                TrialIntent: Coalesce(_d.TrialIntent.Value; varReqExportSel.Trial_Intent; "NA");
                OperationType: Coalesce(_d.OperationType.Value; "NA");
                VCR: Coalesce(_d.VCR; "");
                Field: Coalesce(_d.Field; "");
                Area: Coalesce(_d.Area.Value; varReqExportSel.Area; "SSD");
                Season: Coalesce(Text(_d.Season); "");
                Rank: Coalesce(_d.Rank; "");
                Greenhouse: Coalesce(_d.Greenhouse; "");
                Plantas: Coalesce(_d.PlantsTransplanted; 0);
                EsManual: IsBlank(_d.ID)
            }
        );;
        Reset(txtSetManualExp);;
        Reset(cmbCropManualExp);;
        Notify(_n & " agregado"; NotificationType.Information; 1000)
    )
)
```

---

# PASO 11 · Análisis de exportaciones (rediseño)

## 11.0 Antes de empezar

1. En el árbol, **elimina el contenedor `Kpis Exports` completo**. Con él se van `btnAnaCargar`, `btnAnaNormalizar`, `btnAnaFiltrar`, `btnAnaGraficas` y `btnAnaGuardarDias`. Es a propósito: los volvemos a crear con los nombres correctos. Si no lo borras primero, Studio renombrará los nuevos a `btnAnaCargar_1` y el botón del menú dejará de funcionar.
2. Trabaja en este orden: primero el contenedor, luego **los 4 botones ocultos (11.7 a 11.10)**, y al final los visuales. Así las fórmulas no quedan en rojo mientras construyes.

### Control: `conAnaExport`
**Tipo:** Insertar → Diseño → **Contenedor de diseño vertical**
**Ubicación:** hijo directo de `Container73` (al mismo nivel que `vTablero`, `vCarga`, `vExport`…), como último hijo.

| Propiedad | Fórmula |
|---|---|
| Visible | `varSeccionActiva = "ExportAnalyst"` |
| Fill | `RGBA(244; 247; 250; 1)` |
| Height | `Parent.Height` |
| FillPortions | `1` |
| LayoutDirection | `LayoutDirection.Vertical` |
| LayoutAlignItems | `LayoutAlignItems.Stretch` |
| LayoutGap | `14` |
| LayoutOverflowY | `LayoutOverflow.Scroll` |
| PaddingTop | `16` |
| PaddingBottom | `24` |
| PaddingLeft | `20` |
| PaddingRight | `20` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |

> **La barra de desplazamiento sí se activa pero no se ve:** `LayoutOverflowY` debe quedar en `Scroll` (si no, no baja), y para ocultar la barra pon la propiedad **ShowScrollbar** de `conAnaExport` en `false`. Si tu versión del contenedor no muestra esa propiedad, escríbela igual en la barra de fórmulas con el control seleccionado.

---

## 11.1 Control: `htmlAnaTitulo`
**Tipo:** Insertar → Multimedia → **Visor de HTML**
**Ubicación:** primer hijo de `conAnaExport`.

| Propiedad | Fórmula |
|---|---|
| AutoHeight | `true` |
| FillPortions | `0` |
| PaddingTop | `0` |
| PaddingBottom | `0` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |
| HtmlText | ver abajo |

```
"<div style='font-family:Segoe UI;display:flex;justify-content:space-between;align-items:flex-end;flex-wrap:wrap;gap:8px'>" &
"<div><div style='font-size:11px;letter-spacing:1.2px;color:#88BC30;font-weight:700'>SEED PROCESSING AREA · EXPORT ANALYST</div>" &
"<div style='font-size:22px;font-weight:700;color:#0C384C'>Análisis de exportaciones</div></div>" &
"<div style='font-size:12px;color:#5a6b7b;text-align:right'>Fecha base: <b>" & Coalesce(varAnaBase; "Pack/treat start") & "</b><br>" &
If(IsBlank(varAnaDesde); ""; Text(varAnaDesde; "[$-en-US]dd/mm/yyyy") & " – " & Text(DateAdd(varAnaHasta; -1; TimeUnit.Days); "[$-en-US]dd/mm/yyyy")) &
"  ·  " & CountRows(colAnaFilt) & " líneas · " & CountRows(colAnaEnvios) & " envíos</div></div>"
```

---

## 11.2 Barra de filtros

### Control: `conAnaFiltros`
**Tipo:** Insertar → Diseño → **Contenedor de diseño horizontal**
**Ubicación:** segundo hijo de `conAnaExport`.

| Propiedad | Fórmula |
|---|---|
| Fill | `RGBA(255; 255; 255; 1)` |
| Height | `56` |
| FillPortions | `0` |
| LayoutAlignItems | `LayoutAlignItems.Center` |
| LayoutGap | `8` |
| PaddingTop | `8` |
| PaddingBottom | `8` |
| PaddingLeft | `10` |
| PaddingRight | `10` |
| RadiusTopLeft / RadiusTopRight / RadiusBottomLeft / RadiusBottomRight | `10` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |

Dentro de `conAnaFiltros`, en este orden:

### Control: `DatePicker2` · Tipo: Insertar → Entrada → **Selector de fecha (moderno)**

| Propiedad | Fórmula |
|---|---|
| DefaultDate | `DateAdd(Today(); -365; TimeUnit.Days)` |
| Format | `"dd/mm/yyyy"` |
| Placeholder | `"Desde"` |
| Width | `135` |
| OnChange | `Select(btnAnaCargar)` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |

### Control: `DatePicker4` · Tipo: **Selector de fecha (moderno)**

| Propiedad | Fórmula |
|---|---|
| DefaultDate | `Today()` |
| Format | `"dd/mm/yyyy"` |
| Placeholder | `"Hasta"` |
| Width | `135` |
| OnChange | `Select(btnAnaCargar)` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |

### Control: `cmbAnaFechaBase` · Tipo: **Cuadro combinado (moderno)**

| Propiedad | Fórmula |
|---|---|
| Items | `["Pack/treat start"; "CreatedDate"; "Shipping Date"; "Delivery Date"]` |
| DefaultSelectedItems | `["Pack/treat start"]` |
| ItemDisplayText | `ThisItem.Value` |
| InputTextPlaceholder | `"Fecha base"` |
| SelectMultiple | `false` |
| FillPortions | `1` |
| OnChange | `Select(btnAnaCargar)` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |

### Control: `cmbAnaTipo` · Tipo: **Cuadro combinado (moderno)**

| Propiedad | Fórmula |
|---|---|
| Items | `["Todos"; "Seed"; "Tissue"]` |
| DefaultSelectedItems | `["Todos"]` |
| ItemDisplayText | `ThisItem.Value` |
| InputTextPlaceholder | `"Tipo"` |
| SelectMultiple | `false` |
| FillPortions | `1` |
| OnChange | `Select(btnAnaFiltrar)` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |

### Control: `cmbAnaCountry` · Tipo: **Cuadro combinado (moderno)**

| Propiedad | Fórmula |
|---|---|
| DefaultSelectedItems | `["Todos"]` |
| ItemDisplayText | `ThisItem.Value` |
| InputTextPlaceholder | `"País"` |
| SelectMultiple | `false` |
| FillPortions | `1` |
| OnChange | `Select(btnAnaFiltrar)` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |
| Items | ver abajo |

```
Ungroup(
    Table(
        {Op: Table({Value: "Todos"})};
        {Op: Sort(Distinct(colAna; SHIP_TO_COUNTRY); Value; SortOrder.Ascending)}
    );
    Op
)
```

### Control: `txtAnaBuscar` · Tipo: **Entrada de texto (clásica)**

| Propiedad | Fórmula |
|---|---|
| Default | `""` |
| DelayOutput | `true` |
| HintText | `"Request ID, SET, PEA, locación, programa, cultivo"` |
| FillPortions | `2` |
| Height | `34` |
| PaddingLeft | `8` |
| Size | `11` |
| OnChange | `Select(btnAnaFiltrar)` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |

### Control: `icoAnaLimpiar` · Tipo: **Icono moderno**

| Propiedad | Fórmula |
|---|---|
| Icon | `"Dismiss"` |
| IconColor | `RGBA(215; 58; 60; 1)` |
| Tooltip | `"Limpiar filtros"` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |
| OnSelect | ver abajo |

```
Clear(colAnaCropSel);;
Reset(txtAnaBuscar);;
Reset(cmbAnaTipo);;
Reset(cmbAnaCountry);;
Reset(cmbAnaFechaBase);;
Reset(DatePicker2);;
Reset(DatePicker4);;
Select(btnAnaCargar)
```

---

## 11.3 Control: `htmlAnaKpis` (tarjetas KPI)
**Tipo:** **Visor de HTML**
**Ubicación:** tercer hijo de `conAnaExport`.

| Propiedad | Fórmula |
|---|---|
| AutoHeight | `true` |
| FillPortions | `0` |
| PaddingTop / PaddingBottom / PaddingLeft / PaddingRight | `0` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |
| HtmlText | ver abajo |

```
"<div style='font-family:Segoe UI;display:flex;flex-wrap:wrap;gap:10px'>" &
Concat(
    Table(
        {t: "Envíos (IDs)"; v: Text(Coalesce(varAnaEnvios; 0); "[$-en-US]#,##0"); s: Text(Coalesce(varAnaEntregados; 0)) & " entregados"; c: "#0C384C"};
        {t: "SETs enviados"; v: Text(Coalesce(varAnaSets; 0); "[$-en-US]#,##0"); s: "únicos por ID"; c: "#88BC30"};
        {t: "Packets"; v: Text(Coalesce(varAnaPackets; 0); "[$-en-US]#,##0"); s: "suma de líneas"; c: "#2e9fd6"};
        {t: "Peso neto"; v: Text(Coalesce(varAnaNeto; 0); "[$-en-US]#,##0.0") & " kg"; s: "bruto " & Text(Coalesce(varAnaBruto; 0); "[$-en-US]#,##0.0") & " kg"; c: "#7E3A66"};
        {t: "Cajas"; v: Text(Coalesce(varAnaCajas; 0); "[$-en-US]#,##0"); s: "No. de caja distinto por ID"; c: "#C97E17"};
        {t: "Lead solicitado"; v: Text(Coalesce(varAnaLeadReq; 0); "[$-en-US]0.0") & " d"; s: "Created → Requested ship"; c: "#17456B"};
        {t: "Solicitud → envío"; v: Text(Coalesce(varAnaSolEnv; 0); "[$-en-US]0.0") & " d"; s: "Created → Shipping"; c: "#63756B"};
        {t: "Ciclo total"; v: Text(Coalesce(varAnaCiclo; 0); "[$-en-US]0.0") & " d"; s: "Created → Delivery"; c: "#10241C"};
        {t: "Envíos a tiempo"; v: Text(Coalesce(varAnaOnTime; 0); "[$-en-US]0.0") & "%"; s: Text(Coalesce(varAnaConFecha; 0)) & " con fecha de envío"; c: If(Coalesce(varAnaOnTime; 0) >= 85; "#2f9e5f"; "#d35442")}
    ) As K;
    "<div style='flex:1 1 150px;min-width:140px;background:#fff;border:1px solid #e3eaf1;border-left:4px solid " & K.c & ";border-radius:10px;padding:10px 12px'>" &
    "<div style='font-size:10px;color:#5a6b7b;text-transform:uppercase;letter-spacing:.5px'>" & K.t & "</div>" &
    "<div style='font-size:24px;font-weight:700;color:#16273a;margin:2px 0'>" & K.v & "</div>" &
    "<div style='font-size:11px;color:#93a4b4'>" & K.s & "</div></div>"
) & "</div>"
```

---

## 11.4 Tarjetas por cultivo

### Control: `lblAnaSecCultivo` · Tipo: **Etiqueta de texto** · Ubicación: cuarto hijo de `conAnaExport`

| Propiedad | Fórmula |
|---|---|
| Text | `"Por cultivo  ·  toca una tarjeta para filtrar" & If(IsEmpty(colAnaCropSel); ""; "  (filtro: " & Concat(colAnaCropSel; CROP; ", ") & ")")` |
| Color | `RGBA(12; 56; 76; 1)` |
| FontWeight | `FontWeight.Bold` |
| Size | `12` |
| Height | `24` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |

### Control: `galAnaCrop`
**Tipo:** Insertar → **Galería horizontal en blanco**
**Ubicación:** quinto hijo de `conAnaExport`.

| Propiedad | Fórmula |
|---|---|
| Items | `Filter(colAnaCrop; CROP <> "Otros" Or ENVIOS > 0)` |
| Height | `170` |
| FillPortions | `0` |
| TemplateSize | `220` |
| TemplatePadding | `6` |
| ShowScrollbar | `false` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |

### Hijo 1 — **Control:** `htmlAnaCropCard` · Tipo: **Visor de HTML**

| Propiedad | Fórmula |
|---|---|
| Height | `Parent.TemplateHeight` |
| Width | `Parent.TemplateWidth` |
| PaddingTop / PaddingBottom / PaddingLeft / PaddingRight | `0` |
| X | `0` |
| Y | `0` |
| HtmlText | ver abajo |

```
"<div style='font-family:Segoe UI;box-sizing:border-box;height:150px;background:" & If(ThisItem.SEL; "#F3F9E9"; "#fff") &
";border:1px solid " & If(ThisItem.SEL; ThisItem.COLOR; "#e3eaf1") & ";border-top:5px solid " & ThisItem.COLOR & ";border-radius:12px;padding:10px 12px'>" &
"<div style='display:flex;justify-content:space-between;align-items:center'><b style='font-size:14px;color:#16273a'>" & ThisItem.CROP & "</b>" &
"<span style='font-size:10px;color:#93a4b4'>" & ThisItem.ENVIOS & " envíos</span></div>" &
"<div style='font-size:28px;font-weight:700;color:" & ThisItem.COLOR & ";line-height:1.2'>" & ThisItem.SETS_UNICOS & "<span style='font-size:11px;color:#5a6b7b;font-weight:400'> SETs</span></div>" &
"<div style='font-size:11px;color:#5a6b7b'>" & Text(ThisItem.PACKETS; "[$-en-US]#,##0") & " packets · " & ThisItem.CAJAS & " cajas</div>" &
"<div style='font-size:11px;color:#5a6b7b'>" & Text(ThisItem.NET_WEIGHT; "[$-en-US]#,##0.0") & " kg neto · " & Text(ThisItem.DIAS_CICLO; "[$-en-US]0.0") & " d ciclo</div>" &
"<div style='font-size:9px;color:#93a4b4;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;margin-top:2px'>" & ThisItem.NOMBRES & "</div></div>"
```

### Hijo 2 — **Control:** `btnAnaCropSel` · Tipo: **Botón (clásico)** (transparente, encima de la tarjeta)

| Propiedad | Fórmula |
|---|---|
| Text | `""` |
| Fill | `RGBA(0; 0; 0; 0)` |
| Color | `RGBA(0; 0; 0; 0)` |
| BorderColor | `RGBA(0; 0; 0; 0)` |
| HoverFill | `RGBA(0; 0; 0; 0,03)` |
| HoverColor | `RGBA(0; 0; 0; 0)` |
| HoverBorderColor | `RGBA(0; 0; 0; 0)` |
| PressedFill | `RGBA(0; 0; 0; 0)` |
| PressedColor | `RGBA(0; 0; 0; 0)` |
| PressedBorderColor | `RGBA(0; 0; 0; 0)` |
| Height | `Parent.TemplateHeight` |
| Width | `Parent.TemplateWidth` |
| X | `0` |
| Y | `0` |
| OnSelect | ver abajo |

```
If(
    !IsBlank(LookUp(colAnaCropSel; CROP = ThisItem.CROP; CROP));
    RemoveIf(colAnaCropSel; CROP = ThisItem.CROP);
    Collect(colAnaCropSel; {CROP: ThisItem.CROP})
);;
Select(btnAnaFiltrar)
```

---

## 11.5 Control: `htmlAnaCropTabla` (resumen por cultivo)
**Tipo:** **Visor de HTML** · **Ubicación:** sexto hijo de `conAnaExport`.

| Propiedad | Fórmula |
|---|---|
| AutoHeight | `true` |
| FillPortions | `0` |
| PaddingTop / PaddingBottom / PaddingLeft / PaddingRight | `0` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |
| HtmlText | ver abajo |

```
"<div style='font-family:Segoe UI;background:#fff;border:1px solid #e3eaf1;border-radius:12px;padding:12px 14px'>" &
"<div style='font-size:14px;font-weight:700;color:#0C384C;margin-bottom:8px'>Resumen por cultivo</div>" &
"<table style='width:100%;border-collapse:collapse;font-size:12px'><tr style='background:#f4f8fc;color:#5a6b7b'>" &
Concat(
    Table({h: "Cultivo"}; {h: "SETs"}; {h: "Envíos"}; {h: "Packets"}; {h: "Neto kg"}; {h: "Bruto kg"}; {h: "Cajas"}; {h: "Ciclo"}) As _h;
    "<th style='text-align:" & If(_h.h = "Cultivo"; "left"; "right") & ";padding:7px 8px;font-weight:600'>" & _h.h & "</th>"
) & "</tr>" &
Concat(
    Sort(Filter(colAnaCrop; ENVIOS > 0 And (IsEmpty(colAnaCropSel) Or SEL)); PACKETS; SortOrder.Descending) As _c;
    "<tr style='border-bottom:1px solid #eef2f6'><td style='padding:7px 8px'>" &
    "<span style='display:inline-block;width:9px;height:9px;border-radius:2px;background:" & _c.COLOR & ";margin-right:6px'></span><b>" & _c.CROP & "</b>" &
    "<div style='font-size:10px;color:#93a4b4'>" & _c.NOMBRES & "</div></td>" &
    "<td style='text-align:right;padding:7px 8px'>" & Text(_c.SETS_UNICOS; "[$-en-US]#,##0") & "</td>" &
    "<td style='text-align:right;padding:7px 8px'>" & Text(_c.ENVIOS; "[$-en-US]#,##0") & "</td>" &
    "<td style='text-align:right;padding:7px 8px'>" & Text(_c.PACKETS; "[$-en-US]#,##0") & "</td>" &
    "<td style='text-align:right;padding:7px 8px'>" & Text(_c.NET_WEIGHT; "[$-en-US]#,##0.0") & "</td>" &
    "<td style='text-align:right;padding:7px 8px'>" & Text(_c.GROSS_WEIGHT; "[$-en-US]#,##0.0") & "</td>" &
    "<td style='text-align:right;padding:7px 8px'>" & Text(_c.CAJAS; "[$-en-US]#,##0") & "</td>" &
    "<td style='text-align:right;padding:7px 8px'>" & Text(_c.DIAS_CICLO; "[$-en-US]0.0") & " d</td></tr>"
) &
"<tr style='background:#F0F5EE;font-weight:700'><td style='padding:7px 8px'>Total</td>" &
"<td style='text-align:right;padding:7px 8px'>" & Text(Coalesce(varAnaSets; 0); "[$-en-US]#,##0") & "</td>" &
"<td style='text-align:right;padding:7px 8px'>" & Text(Coalesce(varAnaEnvios; 0); "[$-en-US]#,##0") & "</td>" &
"<td style='text-align:right;padding:7px 8px'>" & Text(Coalesce(varAnaPackets; 0); "[$-en-US]#,##0") & "</td>" &
"<td style='text-align:right;padding:7px 8px'>" & Text(Coalesce(varAnaNeto; 0); "[$-en-US]#,##0.0") & "</td>" &
"<td style='text-align:right;padding:7px 8px'>" & Text(Coalesce(varAnaBruto; 0); "[$-en-US]#,##0.0") & "</td>" &
"<td style='text-align:right;padding:7px 8px'>" & Text(Coalesce(varAnaCajas; 0); "[$-en-US]#,##0") & "</td>" &
"<td style='text-align:right;padding:7px 8px'>" & Text(Coalesce(varAnaCiclo; 0); "[$-en-US]0.0") & " d</td></tr></table></div>"
```

---

## 11.6 Control: `htmlAnaEtapas` (dónde se va el tiempo)
**Tipo:** **Visor de HTML** · **Ubicación:** séptimo hijo de `conAnaExport`.

| Propiedad | Fórmula |
|---|---|
| AutoHeight | `true` |
| FillPortions | `0` |
| PaddingTop / PaddingBottom / PaddingLeft / PaddingRight | `0` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |
| HtmlText | ver abajo |

> Aquí es donde antes salía **"Operación no válida: división entre cero"**. Los `Max(1; …)` del `With` inicial lo evitan.

```
With(
    {
        _tot: Max(1; Sum(colAnaEtapas; DIAS));
        _mx: Max(1; Max(colAnaEtapas; DIAS));
        _mr: Max(1; Coalesce(varAnaLeadReq; 0); Coalesce(varAnaSolEnv; 0))
    };
    "<div style='font-family:Segoe UI;background:#fff;border:1px solid #e3eaf1;border-radius:12px;padding:14px 16px'>" &
    "<div style='display:flex;justify-content:space-between;align-items:flex-end;flex-wrap:wrap;gap:8px'>" &
    "<div><div style='font-size:14px;font-weight:700;color:#0C384C'>Ciclo del envío · dónde se va el tiempo</div>" &
    "<div style='font-size:11px;color:#93a4b4'>Promedio por envío (ID). Se calcula con las fechas; si faltan, con los días guardados.</div></div>" &
    "<div style='font-size:26px;font-weight:700;color:#10241C'>" & Text(Coalesce(varAnaCiclo; 0); "[$-en-US]0.0") & " d <span style='font-size:11px;color:#5a6b7b;font-weight:400'>solicitud → entrega</span></div></div>" &
    "<div style='display:flex;height:26px;border-radius:6px;overflow:hidden;margin:12px 0 10px;background:#eef2f6'>" &
    Concat(
        Sort(colAnaEtapas; ORDEN; SortOrder.Ascending) As _e;
        If(_e.DIAS > 0; "<div style='background:" & _e.COLOR & ";width:" & Text(_e.DIAS / _tot * 100; "[$-en-US]0.##") & "%'></div>"; "")
    ) & "</div>" &
    Concat(
        Sort(colAnaEtapas; ORDEN; SortOrder.Ascending) As _e;
        "<div style='display:flex;align-items:center;gap:10px;font-size:12px;padding:3px 0'>" &
        "<div style='width:160px;color:#27322B'><span style='display:inline-block;width:9px;height:9px;border-radius:2px;background:" & _e.COLOR & ";margin-right:6px'></span>" & _e.ETAPA & "</div>" &
        "<div style='flex:1;background:#eef2f6;border-radius:4px;height:14px;overflow:hidden'><div style='height:14px;border-radius:4px;background:" & _e.COLOR & ";width:" & Text(_e.DIAS / _mx * 100; "[$-en-US]0.##") & "%'></div></div>" &
        "<div style='width:130px;text-align:right;color:#16273a'><b>" & Text(_e.DIAS; "[$-en-US]0.0") & " d</b><span style='color:#93a4b4'> · " & _e.N & " env.</span></div></div>"
    ) &
    "<div style='border-top:1px dashed #cfdae5;margin-top:10px;padding-top:8px;font-size:11px;color:#5a6b7b'>Plazo solicitado vs real</div>" &
    Concat(
        Table(
            {n: "Lead solicitado (Created → Requested ship)"; d: Coalesce(varAnaLeadReq; 0); c: "#17456B"};
            {n: "Real (Created → Shipping)"; d: Coalesce(varAnaSolEnv; 0); c: If(Coalesce(varAnaSolEnv; 0) > Coalesce(varAnaLeadReq; 0); "#d35442"; "#2f9e5f")}
        ) As _r;
        "<div style='display:flex;align-items:center;gap:10px;font-size:12px;padding:3px 0'>" &
        "<div style='width:260px;color:#27322B'>" & _r.n & "</div>" &
        "<div style='flex:1;background:#eef2f6;border-radius:4px;height:12px;overflow:hidden'><div style='height:12px;border-radius:4px;background:" & _r.c & ";width:" & Text(_r.d / _mr * 100; "[$-en-US]0.##") & "%'></div></div>" &
        "<div style='width:80px;text-align:right'><b>" & Text(_r.d; "[$-en-US]0.0") & " d</b></div></div>"
    ) & "</div>"
)
```

---

## 11.7 Días por locación (selector + gráfica)

### Control: `conAnaMetrica` · Tipo: **Contenedor de diseño horizontal** · Ubicación: octavo hijo de `conAnaExport`

| Propiedad | Fórmula |
|---|---|
| Height | `44` |
| FillPortions | `0` |
| LayoutAlignItems | `LayoutAlignItems.Center` |
| LayoutGap | `10` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |

### Hijo 1 — **Control:** `lblAnaMetrica` · Tipo: **Etiqueta de texto**

| Propiedad | Fórmula |
|---|---|
| Text | `"Días promedio por locación"` |
| Color | `RGBA(12; 56; 76; 1)` |
| FontWeight | `FontWeight.Bold` |
| Size | `12` |
| FillPortions | `1` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |

### Hijo 2 — **Control:** `cmbAnaMetrica` · Tipo: **Cuadro combinado (moderno)**

| Propiedad | Fórmula |
|---|---|
| Items | `["Ciclo total"; "Lead solicitado"; "Solicitud a envio"; "Laboratorio"; "Empaque"; "IP"; "GL"; "Transito"; "Aduana a entrega"]` |
| DefaultSelectedItems | `["Ciclo total"]` |
| ItemDisplayText | `ThisItem.Value` |
| SelectMultiple | `false` |
| Width | `240` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |

> No lleva `OnChange`: la gráfica de abajo se recalcula sola al cambiar la selección.

### Control: `htmlAnaLocDias` · Tipo: **Visor de HTML** · Ubicación: noveno hijo de `conAnaExport`

| Propiedad | Fórmula |
|---|---|
| AutoHeight | `true` |
| FillPortions | `0` |
| PaddingTop / PaddingBottom / PaddingLeft / PaddingRight | `0` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |
| HtmlText | ver abajo |

```
With(
    {_m: Coalesce(cmbAnaMetrica.Selected.Value; "Ciclo total")};
    With(
        {
            _rows: ForAll(
                Distinct(colAnaEnvios; SHIP_TO_LOCATION) As _l;
                With(
                    {
                        _v: ForAll(Filter(colAnaEnvios; SHIP_TO_LOCATION = _l.Value) As _e;
                            {
                                D: Switch(_m;
                                    "Lead solicitado"; _e.DIAS_REQUERIDO;
                                    "Solicitud a envio"; _e.DIAS_SOLICITUD_ENVIO;
                                    "Laboratorio"; _e.DIAS_LAB;
                                    "Empaque"; _e.DIAS_EMPAQUE;
                                    "IP"; _e.DIAS_IP;
                                    "GL"; _e.DIAS_GL;
                                    "Transito"; _e.DIAS_TRANSITO;
                                    "Aduana a entrega"; _e.DIAS_ADUANA;
                                    _e.DIAS_CICLO
                                )
                            }
                        )
                    };
                    With({_ok: Filter(_v; D >= 0)};
                        {L: _l.Value; N: CountRows(_ok); V: If(IsEmpty(_ok); 0; Round(Average(_ok; D); 1))}
                    )
                )
            );
            _col: Switch(_m;
                "Laboratorio"; "#17456B";
                "Empaque"; "#88BC30";
                "IP"; "#C97E17";
                "GL"; "#7E3A66";
                "Transito"; "#B23A32";
                "Aduana a entrega"; "#63756B";
                "Lead solicitado"; "#2e9fd6";
                "#10241C"
            )
        };
        With({_mx: Max(1; Max(_rows; V))};
            "<div style='font-family:Segoe UI;background:#fff;border:1px solid #e3eaf1;border-radius:12px;padding:12px 14px'>" &
            "<div style='font-size:11px;color:#93a4b4;margin-bottom:6px'>Métrica: <b style='color:#16273a'>" & _m & "</b> · barra = días promedio · n = envíos con dato</div>" &
            Concat(
                Sort(_rows; V; SortOrder.Descending) As _r;
                "<div style='display:flex;align-items:center;gap:10px;font-size:12px;padding:3px 0'>" &
                "<div style='width:180px;color:#27322B;white-space:nowrap;overflow:hidden;text-overflow:ellipsis'>" & _r.L & "</div>" &
                "<div style='flex:1;background:#eef2f6;border-radius:4px;height:14px;overflow:hidden'><div style='height:14px;border-radius:4px;background:" & _col & ";width:" & Text(_r.V / _mx * 100; "[$-en-US]0.##") & "%'></div></div>" &
                "<div style='width:110px;text-align:right;color:#16273a'><b>" & Text(_r.V; "[$-en-US]0.0") & " d</b><span style='color:#93a4b4'> · n " & _r.N & "</span></div></div>"
            ) &
            If(IsEmpty(_rows); "<div style='font-size:12px;color:#93a4b4'>Sin datos para los filtros actuales</div>"; "") & "</div>"
        )
    )
)
```

---

## 11.8 Control: `htmlAnaRanking` (locación, PEA, país, tipo de caja — completos)
**Tipo:** **Visor de HTML** · **Ubicación:** décimo hijo de `conAnaExport`.

| Propiedad | Fórmula |
|---|---|
| AutoHeight | `true` |
| FillPortions | `0` |
| PaddingTop / PaddingBottom / PaddingLeft / PaddingRight | `0` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |
| HtmlText | ver abajo |

```
"<div style='font-family:Segoe UI;display:flex;flex-wrap:wrap;gap:14px'>" &

"<div style='flex:1 1 360px;min-width:320px;background:#fff;border:1px solid #e3eaf1;border-radius:12px;padding:12px 14px'>" &
"<div style='display:flex;justify-content:space-between'><b style='font-size:13px;color:#0C384C'>Envíos por locación</b><span style='font-size:11px;color:#93a4b4'>" & CountRows(colAnaLoc) & " locaciones</span></div>" &
"<div style='font-size:10px;color:#93a4b4;margin:2px 0 8px'>envíos · packets · SETs · ciclo prom.</div>" &
With({_mx: Max(1; Max(colAnaLoc; ENVIOS))};
    Concat(Sort(colAnaLoc; ENVIOS; SortOrder.Descending) As _r;
        "<div style='display:flex;align-items:center;gap:8px;font-size:11px;padding:3px 0'>" &
        "<div style='width:120px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis'>" & _r.NOMBRE & "</div>" &
        "<div style='flex:1;background:#eef2f6;border-radius:4px;height:12px;overflow:hidden'><div style='height:12px;border-radius:4px;background:#88BC30;width:" & Text(_r.ENVIOS / _mx * 100; "[$-en-US]0.##") & "%'></div></div>" &
        "<div style='width:170px;text-align:right;color:#16273a'><b>" & _r.ENVIOS & "</b><span style='color:#93a4b4'> · " & Text(_r.PACKETS; "[$-en-US]#,##0") & " pk · " & _r.SETS & " SET · " & Text(_r.CICLO; "[$-en-US]0.0") & " d</span></div></div>"
    )
) & "</div>" &

"<div style='flex:1 1 360px;min-width:320px;background:#fff;border:1px solid #e3eaf1;border-radius:12px;padding:12px 14px'>" &
"<div style='display:flex;justify-content:space-between'><b style='font-size:13px;color:#0C384C'>Packets por PEA</b><span style='font-size:11px;color:#93a4b4'>" & CountRows(colAnaPEA) & " PEA</span></div>" &
"<div style='font-size:10px;color:#93a4b4;margin:2px 0 8px'>barra = packets · entregados · envíos · SETs</div>" &
With({_mx: Max(1; Max(colAnaPEA; PACKETS))};
    Concat(Sort(colAnaPEA; PACKETS; SortOrder.Descending) As _r;
        "<div style='display:flex;align-items:center;gap:8px;font-size:11px;padding:3px 0'>" &
        "<div style='width:130px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis'>" & _r.NOMBRE & "</div>" &
        "<div style='flex:1;background:#eef2f6;border-radius:4px;height:12px;overflow:hidden'><div style='height:12px;border-radius:4px;background:#C97E17;width:" & Text(_r.PACKETS / _mx * 100; "[$-en-US]0.##") & "%'></div></div>" &
        "<div style='width:190px;text-align:right;color:#16273a'><b>" & Text(_r.PACKETS; "[$-en-US]#,##0") & "</b><span style='color:#93a4b4'> · " & Text(_r.ENTREGADOS; "[$-en-US]#,##0") & " entr. · " & _r.ENVIOS & " env · " & _r.SETS & " SET</span></div></div>"
    )
) & "</div>" &

"<div style='flex:1 1 360px;min-width:320px;background:#fff;border:1px solid #e3eaf1;border-radius:12px;padding:12px 14px'>" &
"<div style='display:flex;justify-content:space-between'><b style='font-size:13px;color:#0C384C'>Envíos por país</b><span style='font-size:11px;color:#93a4b4'>" & CountRows(colAnaPais) & " países</span></div>" &
"<div style='font-size:10px;color:#93a4b4;margin:2px 0 8px'>envíos · packets · ciclo prom.</div>" &
With({_mx: Max(1; Max(colAnaPais; ENVIOS))};
    Concat(Sort(colAnaPais; ENVIOS; SortOrder.Descending) As _r;
        "<div style='display:flex;align-items:center;gap:8px;font-size:11px;padding:3px 0'>" &
        "<div style='width:120px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis'>" & _r.NOMBRE & "</div>" &
        "<div style='flex:1;background:#eef2f6;border-radius:4px;height:12px;overflow:hidden'><div style='height:12px;border-radius:4px;background:#17456B;width:" & Text(_r.ENVIOS / _mx * 100; "[$-en-US]0.##") & "%'></div></div>" &
        "<div style='width:150px;text-align:right;color:#16273a'><b>" & _r.ENVIOS & "</b><span style='color:#93a4b4'> · " & Text(_r.PACKETS; "[$-en-US]#,##0") & " pk · " & Text(_r.CICLO; "[$-en-US]0.0") & " d</span></div></div>"
    )
) & "</div>" &

"<div style='flex:1 1 360px;min-width:320px;background:#fff;border:1px solid #e3eaf1;border-radius:12px;padding:12px 14px'>" &
"<div style='display:flex;justify-content:space-between'><b style='font-size:13px;color:#0C384C'>Tipo de caja / paquete</b><span style='font-size:11px;color:#93a4b4'>" & CountRows(colAnaPack) & " tipos</span></div>" &
"<div style='font-size:10px;color:#93a4b4;margin:2px 0 8px'>barra = cajas · líneas · packets</div>" &
With({_mx: Max(1; Max(colAnaPack; CAJAS))};
    Concat(Sort(colAnaPack; CAJAS; SortOrder.Descending) As _r;
        "<div style='display:flex;align-items:center;gap:8px;font-size:11px;padding:3px 0'>" &
        "<div style='width:130px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis'>" & _r.NOMBRE & "</div>" &
        "<div style='flex:1;background:#eef2f6;border-radius:4px;height:12px;overflow:hidden'><div style='height:12px;border-radius:4px;background:#7E3A66;width:" & Text(_r.CAJAS / _mx * 100; "[$-en-US]0.##") & "%'></div></div>" &
        "<div style='width:160px;text-align:right;color:#16273a'><b>" & _r.CAJAS & "</b><span style='color:#93a4b4'> cajas · " & _r.LINEAS & " lín · " & Text(_r.PACKETS; "[$-en-US]#,##0") & " pk</span></div></div>"
    )
) & "</div>" &

"</div>"
```

---

## 11.9 Los 4 botones ocultos del motor

Los cuatro son **Botón (clásico)**, hijos directos de `conAnaExport` (al final del contenedor), con:

| Propiedad (los 4) | Valor |
|---|---|
| Visible | `false` |
| Height | `1` |
| Width | `1` |
| FillPortions | `0` |
| LayoutMinHeight | `16` |
| LayoutMinWidth | `16` |

### Control: `btnAnaCargar` · OnSelect
Trae de SharePoint solo por fecha (delegable) y **después** quita las filas sin `Request_ID`. Ese `!IsBlank(Request_ID)` dentro del `Filter` era lo que rompía el filtro de fechas.

```
Set(varAnaDesde; Coalesce(DatePicker2.SelectedDate; DateAdd(Today(); -365; TimeUnit.Days)));;
Set(varAnaHasta; DateAdd(Coalesce(DatePicker4.SelectedDate; Today()); 1; TimeUnit.Days));;
Set(varAnaBase; Coalesce(cmbAnaFechaBase.Selected.Value; "Pack/treat start"));;
Switch(
    varAnaBase;
    "Pack/treat start"; ClearCollect(colAnaBase; Filter('EXPORT-IMPORT'; 'Pack/treat start' >= varAnaDesde And 'Pack/treat start' < varAnaHasta));
    "Shipping Date"; ClearCollect(colAnaBase; Filter('EXPORT-IMPORT'; 'Shipping Date' >= varAnaDesde And 'Shipping Date' < varAnaHasta));
    "Delivery Date"; ClearCollect(colAnaBase; Filter('EXPORT-IMPORT'; 'Delivery Date' >= varAnaDesde And 'Delivery Date' < varAnaHasta));
    ClearCollect(colAnaBase; Filter('EXPORT-IMPORT'; CreatedDate >= varAnaDesde And CreatedDate < varAnaHasta))
);;
RemoveIf(colAnaBase; IsBlank(Request_ID));;
Select(btnAnaNormalizar)
```

### Control: `btnAnaNormalizar` · OnSelect
Calcula **primero con las fechas** (`c…`) y solo si faltan usa los días guardados (`s…`). Marca en `GUARDAR` las filas cuyo valor cambió.

```
ClearCollect(
    colAna;
    ForAll(
        colAnaBase As _r;
        With(
            {
                cLab: With({_i: Coalesce(_r.'FPPO Mother plants sampling'; _r.'Seed sample ToBRFV'); _f: Coalesce(_r.'Mother plants Lab report'; _r.'Lab Report')}; If(IsBlank(_i) Or IsBlank(_f); Blank(); DateDiff(_i; _f; TimeUnit.Days)));
                cEmp: If(IsBlank(_r.'Pack/treat start') Or IsBlank(_r.'Pack/treat completed'); Blank(); DateDiff(_r.'Pack/treat start'; _r.'Pack/treat completed'; TimeUnit.Days));
                cIP: If(IsBlank(_r.'IP Request') Or IsBlank(_r.'IP Received'); Blank(); DateDiff(_r.'IP Request'; _r.'IP Received'; TimeUnit.Days));
                cGL: If(IsBlank(_r.'GL Request') Or IsBlank(_r.'GL Received'); Blank(); DateDiff(_r.'GL Request'; _r.'GL Received'; TimeUnit.Days));
                cTra: If(IsBlank(_r.'Shipping Date') Or IsBlank(_r.'Arrival to customs'); Blank(); DateDiff(_r.'Shipping Date'; _r.'Arrival to customs'; TimeUnit.Days));
                cAdu: If(IsBlank(_r.'Arrival to customs') Or IsBlank(_r.'Delivery Date'); Blank(); DateDiff(_r.'Arrival to customs'; _r.'Delivery Date'; TimeUnit.Days));
                cSolEnv: If(IsBlank(_r.CreatedDate) Or IsBlank(_r.'Shipping Date'); Blank(); DateDiff(_r.CreatedDate; _r.'Shipping Date'; TimeUnit.Days));
                cCiclo: If(IsBlank(_r.CreatedDate) Or IsBlank(_r.'Delivery Date'); Blank(); DateDiff(_r.CreatedDate; _r.'Delivery Date'; TimeUnit.Days));
                cReq: If(IsBlank(_r.CreatedDate) Or IsBlank(_r.REQUESTED_SHIP_DATE); Blank(); DateDiff(_r.CreatedDate; _r.REQUESTED_SHIP_DATE; TimeUnit.Days));
                cDesv: If(IsBlank(_r.REQUESTED_SHIP_DATE) Or IsBlank(_r.'Shipping Date'); Blank(); DateDiff(_r.REQUESTED_SHIP_DATE; _r.'Shipping Date'; TimeUnit.Days));
                sLab: Value(Text(_r.'Testing time'));
                sEmp: Value(Text(_r.'Pack/treat Time'));
                sIP: Value(Text(_r.'IP Time'));
                sGL: Value(Text(_r.'GL Time (days)'));
                sTra: Value(Text(_r.Transit));
                sAdu: Value(Text(_r.'Clearance time'));
                sSolEnv: Value(Text(_r.'Total time ID creation ship date'));
                sCiclo: Value(Text(_r.'ID req-Delivery'));
                sReq: Value(Text(_r.'Requested Lead Time'));
                sDesv: Value(Text(_r.'Ship Deviation'))
            };
            {
                ID: _r.ID;
                Request_ID: _r.Request_ID;
                SET: Coalesce(Trim(_r.SET); "—");
                CROP_ORIG: Coalesce(_r.CROP.Value; "Sin CROP");
                CROP: fxCultivo(_r.CROP.Value);
                SHIP_TO_COUNTRY: Coalesce(_r.SHIP_TO_COUNTRY.Value; "Sin país");
                SHIP_TO_LOCATION: Coalesce(_r.SHIP_TO_LOCATION; "Sin locación");
                TYPE_OF_SHIPMENT: Coalesce(_r.TYPE_OF_SHIPMENT.Value; "Sin tipo");
                SHIPMENT_PROGRESS: Coalesce(_r.SHIPMENT_PROGRESS.Value; "Sin estatus");
                Packaging_Type: Coalesce(_r.Packaging_Type.Value; "Sin tipo de caja");
                PEA: Coalesce(_r.PEA; "Sin PEA");
                Program: Coalesce(_r.Program; "NA");
                Packets: Coalesce(Value(Text(_r.Packets)); 0);
                Net_Weight: Coalesce(Value(Text(_r.Net_Weight)); 0);
                Gross_Weight: Coalesce(Value(Text(_r.Gross_Weight)); 0);
                NumBox: Coalesce(Value(Text(_r.NumBox)); 0);
                ENTREGADO: !IsBlank(_r.'Delivery Date');
                DIAS_LAB: Coalesce(cLab; sLab; -1);
                DIAS_EMPAQUE: Coalesce(cEmp; sEmp; -1);
                DIAS_IP: Coalesce(cIP; sIP; -1);
                DIAS_GL: Coalesce(cGL; sGL; -1);
                DIAS_TRANSITO: Coalesce(cTra; sTra; -1);
                DIAS_ADUANA: Coalesce(cAdu; sAdu; -1);
                DIAS_SOLICITUD_ENVIO: Coalesce(cSolEnv; sSolEnv; -1);
                DIAS_CICLO: Coalesce(cCiclo; sCiclo; -1);
                DIAS_REQUERIDO: Coalesce(cReq; sReq; -1);
                DESVIACION: Coalesce(cDesv; sDesv; -999);
                GUARDAR:
                    (!IsBlank(cLab) And cLab <> Coalesce(sLab; -99999)) Or
                    (!IsBlank(cEmp) And cEmp <> Coalesce(sEmp; -99999)) Or
                    (!IsBlank(cIP) And cIP <> Coalesce(sIP; -99999)) Or
                    (!IsBlank(cGL) And cGL <> Coalesce(sGL; -99999)) Or
                    (!IsBlank(cTra) And cTra <> Coalesce(sTra; -99999)) Or
                    (!IsBlank(cAdu) And cAdu <> Coalesce(sAdu; -99999)) Or
                    (!IsBlank(cSolEnv) And cSolEnv <> Coalesce(sSolEnv; -99999)) Or
                    (!IsBlank(cCiclo) And cCiclo <> Coalesce(sCiclo; -99999)) Or
                    (!IsBlank(cReq) And cReq <> Coalesce(sReq; -99999)) Or
                    (!IsBlank(cDesv) And cDesv <> Coalesce(sDesv; -99999))
            }
        )
    )
);;
If(CountRows(Filter(colAna; GUARDAR)) > 0; Select(btnAnaGuardarDias));;
Select(btnAnaFiltrar)
```

### Control: `btnAnaGuardarDias` · OnSelect
Ya no hay botón visible "Guardar días": se ejecuta solo y únicamente sobre las filas que cambiaron.

```
ForAll(
    Filter(colAna; GUARDAR) As _a;
    Patch('EXPORT-IMPORT'; LookUp('EXPORT-IMPORT'; ID = _a.ID);
        {
            'Testing time': If(_a.DIAS_LAB < 0; Blank(); _a.DIAS_LAB);
            'Pack/treat Time': If(_a.DIAS_EMPAQUE < 0; Blank(); _a.DIAS_EMPAQUE);
            'IP Time': If(_a.DIAS_IP < 0; Blank(); _a.DIAS_IP);
            'GL Time (days)': If(_a.DIAS_GL < 0; Blank(); _a.DIAS_GL);
            Transit: If(_a.DIAS_TRANSITO < 0; Blank(); _a.DIAS_TRANSITO);
            'Clearance time': If(_a.DIAS_ADUANA < 0; Blank(); _a.DIAS_ADUANA);
            'Total time ID creation ship date': If(_a.DIAS_SOLICITUD_ENVIO < 0; Blank(); _a.DIAS_SOLICITUD_ENVIO);
            'ID req-Delivery': If(_a.DIAS_CICLO < 0; Blank(); _a.DIAS_CICLO);
            'Requested Lead Time': If(_a.DIAS_REQUERIDO < 0; Blank(); _a.DIAS_REQUERIDO);
            'Ship Deviation': If(_a.DESVIACION = -999; Blank(); _a.DESVIACION);
            'On Time': If(_a.DESVIACION = -999; Blank(); If(_a.DESVIACION <= 0; "Si"; "No"))
        }
    )
);;
UpdateIf(colAna; GUARDAR; {GUARDAR: false})
```

### Control: `btnAnaFiltrar` · OnSelect
Aquí se agrupan cultivos con `fxCultivo`, se cuentan SETs únicos por ID (`Request_ID & "|" & SET`), cajas por número distinto de caja por ID, y se arma `colAnaEnvios` para promediar por envío y no por línea.

```
ClearCollect(
    colAnaFilt0;
    Filter(colAna As _f;
        (IsBlank(cmbAnaTipo.Selected.Value) Or cmbAnaTipo.Selected.Value = "Todos" Or _f.TYPE_OF_SHIPMENT = cmbAnaTipo.Selected.Value) And
        (IsBlank(cmbAnaCountry.Selected.Value) Or cmbAnaCountry.Selected.Value = "Todos" Or _f.SHIP_TO_COUNTRY = cmbAnaCountry.Selected.Value) And
        (
            IsBlank(txtAnaBuscar.Text) Or
            StartsWith(_f.Request_ID; txtAnaBuscar.Text) Or
            StartsWith(_f.SET; txtAnaBuscar.Text) Or
            StartsWith(_f.PEA; txtAnaBuscar.Text) Or
            StartsWith(_f.SHIP_TO_LOCATION; txtAnaBuscar.Text) Or
            StartsWith(_f.Program; txtAnaBuscar.Text) Or
            StartsWith(_f.CROP_ORIG; txtAnaBuscar.Text)
        )
    )
);;

ClearCollect(
    colAnaCrop;
    ForAll(
        ["Tomato"; "Pepper"; "Melon"; "Watermelon"; "Cucumber"; "Otros"] As _c;
        With({_g: Filter(colAnaFilt0; CROP = _c.Value)};
            {
                CROP: _c.Value;
                COLOR: fxHexCultivo(_c.Value);
                SEL: !IsBlank(LookUp(colAnaCropSel; CROP = _c.Value; CROP));
                ENVIOS: CountRows(Distinct(_g; Request_ID));
                SETS_UNICOS: CountRows(Distinct(_g; Request_ID & "|" & SET));
                PACKETS: Sum(_g; Packets);
                NET_WEIGHT: Round(Sum(_g; Net_Weight); 2);
                GROSS_WEIGHT: Round(Sum(_g; Gross_Weight); 2);
                CAJAS: CountRows(Distinct(Filter(_g; NumBox > 0); Request_ID & "|" & Text(NumBox)));
                DIAS_CICLO: With(
                    {_t: ForAll(Distinct(Filter(_g; DIAS_CICLO >= 0); Request_ID) As _q; {D: Max(Filter(_g; Request_ID = _q.Value); DIAS_CICLO)})};
                    If(IsEmpty(_t); 0; Round(Average(_t; D); 1))
                );
                NOMBRES: Concat(Distinct(_g; CROP_ORIG); Value; ", ")
            }
        )
    )
);;

ClearCollect(colAnaFilt; Filter(colAnaFilt0 As _f; IsEmpty(colAnaCropSel) Or !IsBlank(LookUp(colAnaCropSel; CROP = _f.CROP; CROP))));;

ClearCollect(
    colAnaEnvios;
    ForAll(
        Distinct(colAnaFilt; Request_ID) As _q;
        With({_g: Filter(colAnaFilt; Request_ID = _q.Value)};
            With({_h: First(_g)};
                {
                    Request_ID: _q.Value;
                    CROP: _h.CROP;
                    PEA: _h.PEA;
                    SHIP_TO_LOCATION: _h.SHIP_TO_LOCATION;
                    SHIP_TO_COUNTRY: _h.SHIP_TO_COUNTRY;
                    ENTREGADO: CountRows(Filter(_g; ENTREGADO)) > 0;
                    SETS: CountRows(Distinct(_g; SET));
                    PACKETS: Sum(_g; Packets);
                    NETO: Sum(_g; Net_Weight);
                    CAJAS: CountRows(Distinct(Filter(_g; NumBox > 0); NumBox));
                    DIAS_LAB: Max(_g; DIAS_LAB);
                    DIAS_EMPAQUE: Max(_g; DIAS_EMPAQUE);
                    DIAS_IP: Max(_g; DIAS_IP);
                    DIAS_GL: Max(_g; DIAS_GL);
                    DIAS_TRANSITO: Max(_g; DIAS_TRANSITO);
                    DIAS_ADUANA: Max(_g; DIAS_ADUANA);
                    DIAS_SOLICITUD_ENVIO: Max(_g; DIAS_SOLICITUD_ENVIO);
                    DIAS_CICLO: Max(_g; DIAS_CICLO);
                    DIAS_REQUERIDO: Max(_g; DIAS_REQUERIDO);
                    DESVIACION: Max(_g; DESVIACION)
                }
            )
        )
    )
);;

Set(varAnaEnvios; CountRows(colAnaEnvios));;
Set(varAnaSets; Sum(colAnaEnvios; SETS));;
Set(varAnaPackets; Sum(colAnaEnvios; PACKETS));;
Set(varAnaNeto; Round(Sum(colAnaFilt; Net_Weight); 2));;
Set(varAnaBruto; Round(Sum(colAnaFilt; Gross_Weight); 2));;
Set(varAnaCajas; Sum(colAnaEnvios; CAJAS));;
Set(varAnaLineas; CountRows(colAnaFilt));;
Set(varAnaEntregados; CountRows(Filter(colAnaEnvios; ENTREGADO)));;
Set(varAnaCiclo; With({_d: Filter(colAnaEnvios; DIAS_CICLO >= 0)}; If(IsEmpty(_d); 0; Round(Average(_d; DIAS_CICLO); 1))));;
Set(varAnaLeadReq; With({_d: Filter(colAnaEnvios; DIAS_REQUERIDO >= 0)}; If(IsEmpty(_d); 0; Round(Average(_d; DIAS_REQUERIDO); 1))));;
Set(varAnaSolEnv; With({_d: Filter(colAnaEnvios; DIAS_SOLICITUD_ENVIO >= 0)}; If(IsEmpty(_d); 0; Round(Average(_d; DIAS_SOLICITUD_ENVIO); 1))));;
Set(varAnaConFecha; CountRows(Filter(colAnaEnvios; DESVIACION > -999)));;
Set(varAnaOnTime; With({_d: Filter(colAnaEnvios; DESVIACION > -999)}; If(IsEmpty(_d); 0; Round(CountRows(Filter(_d; DESVIACION <= 0)) / CountRows(_d) * 100; 1))));;

ClearCollect(
    colAnaEtapas;
    ForAll(
        Table(
            {ORDEN: 1; ETAPA: "Laboratorio"; COLOR: "#17456B"; K: "LAB"};
            {ORDEN: 2; ETAPA: "Empaque / tratamiento"; COLOR: "#88BC30"; K: "EMP"};
            {ORDEN: 3; ETAPA: "Import permit (IP)"; COLOR: "#C97E17"; K: "IP"};
            {ORDEN: 4; ETAPA: "Green light (GL)"; COLOR: "#7E3A66"; K: "GL"};
            {ORDEN: 5; ETAPA: "Tránsito"; COLOR: "#B23A32"; K: "TRA"};
            {ORDEN: 6; ETAPA: "Aduana → entrega"; COLOR: "#63756B"; K: "ADU"}
        ) As _s;
        With(
            {_v: Filter(ForAll(colAnaEnvios As _e; {D: Switch(_s.K; "LAB"; _e.DIAS_LAB; "EMP"; _e.DIAS_EMPAQUE; "IP"; _e.DIAS_IP; "GL"; _e.DIAS_GL; "TRA"; _e.DIAS_TRANSITO; "ADU"; _e.DIAS_ADUANA; -1)}); D >= 0)};
            {ORDEN: _s.ORDEN; ETAPA: _s.ETAPA; COLOR: _s.COLOR; N: CountRows(_v); DIAS: If(IsEmpty(_v); 0; Round(Average(_v; D); 1))}
        )
    )
);;

ClearCollect(
    colAnaLoc;
    ForAll(Distinct(colAnaEnvios; SHIP_TO_LOCATION) As _l;
        With({_g: Filter(colAnaEnvios; SHIP_TO_LOCATION = _l.Value)};
            {
                NOMBRE: _l.Value;
                ENVIOS: CountRows(_g);
                SETS: Sum(_g; SETS);
                PACKETS: Sum(_g; PACKETS);
                CICLO: With({_d: Filter(_g; DIAS_CICLO >= 0)}; If(IsEmpty(_d); 0; Round(Average(_d; DIAS_CICLO); 1)))
            }
        )
    )
);;

ClearCollect(
    colAnaPais;
    ForAll(Distinct(colAnaEnvios; SHIP_TO_COUNTRY) As _p;
        With({_g: Filter(colAnaEnvios; SHIP_TO_COUNTRY = _p.Value)};
            {
                NOMBRE: _p.Value;
                ENVIOS: CountRows(_g);
                PACKETS: Sum(_g; PACKETS);
                CICLO: With({_d: Filter(_g; DIAS_CICLO >= 0)}; If(IsEmpty(_d); 0; Round(Average(_d; DIAS_CICLO); 1)))
            }
        )
    )
);;

ClearCollect(
    colAnaPEA;
    ForAll(Distinct(colAnaEnvios; PEA) As _q;
        With({_g: Filter(colAnaEnvios; PEA = _q.Value)};
            {
                NOMBRE: _q.Value;
                ENVIOS: CountRows(_g);
                SETS: Sum(_g; SETS);
                PACKETS: Sum(_g; PACKETS);
                ENTREGADOS: Sum(Filter(_g; ENTREGADO); PACKETS)
            }
        )
    )
);;

ClearCollect(
    colAnaPack;
    ForAll(Distinct(colAnaFilt; Packaging_Type) As _b;
        With({_g: Filter(colAnaFilt; Packaging_Type = _b.Value)};
            {
                NOMBRE: _b.Value;
                LINEAS: CountRows(_g);
                CAJAS: CountRows(Distinct(Filter(_g; NumBox > 0); Request_ID & "|" & Text(NumBox)));
                PACKETS: Sum(_g; Packets)
            }
        )
    )
)
```

---

## 11.10 Orden final de hijos de `conAnaExport`

1. `htmlAnaTitulo`
2. `conAnaFiltros`
3. `htmlAnaKpis`
4. `lblAnaSecCultivo`
5. `galAnaCrop`
6. `htmlAnaCropTabla`
7. `htmlAnaEtapas`
8. `conAnaMetrica`
9. `htmlAnaLocDias`
10. `htmlAnaRanking`
11. `btnAnaCargar`, `btnAnaNormalizar`, `btnAnaGuardarDias`, `btnAnaFiltrar` (ocultos, al final)

---

# PASO 12 · Pruebas (en este orden)

1. **Menú:** recorre los 9 botones. Solo el activo queda azul y en negrita.
2. **Imports:** elige R&D → cambian tarjetas, tarjetas por cultivo y lista. Luego Melon (no debe traer Watermelon) y un rango de fechas.
3. **New request → Envío de semillas:** selecciona un ID recién creado en Global Shipping (sin `Request_ID_Old`). El chip muestra el número y *Asignar ID* se habilita. Guarda 2 SETs: en EXPORT-IMPORT la fila vacía del flujo se rellena con el primer SET y el segundo se crea con país, locación, fechas y estatus de la cabecera; en SOLICITUDESPA1 el item vacío se rellena y el segundo SET se crea una sola vez.
4. **Export → Show details → icono de agregar:** toca el mismo SET dos veces. El botón debe decir "Agregar 2 línea(s)"; en EXPORT-IMPORT quedan 2 filas y en SOLICITUDESPA1 un solo item con la suma de packets.
5. Cambia **Packet Type** en una fila, cierra y vuelve a abrir el detalle: debe conservarse.
6. Haz clic en **En Proceso** del ID y luego agrega otro SET: la fila nueva hereda "In Progress" y la fecha, y en SPA queda "En proceso".
7. **Export analysis:** no debe aparecer el aviso rojo. Cambia Desde/Hasta con base *Pack/treat start* y toca la tarjeta Tomato: todo el tablero filtra solo tomate, incluidos "Tomato Fresh" y "Tomato Process".
8. Como administrador, ejecuta **icoCompletarDS** una vez para rellenar Program / Trial Intent / Generation / PEA históricos.
