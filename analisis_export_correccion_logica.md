# Análisis Export (`Kpis Exports`) — corrección de lógica sobre el diseño existente

**Criterio:** se conserva toda la estructura actual (`Container139`, `Container140`, `Container132`, `Container132_1`, `Container141` + `Gallery2`, *Resumen por cultivo* + `Gallery3`, `conAnaTiempos` + `Gallery4`/`Gallery4_2`, `Container148` + `Gallery5`…`Gallery5_3` y los botones ocultos). **No se borra ni se recrea nada.**
Solo se cambian fórmulas, se agregan 3 tarjetas KPI (copiando una existente) y 5 botones transparentes opcionales.

Sintaxis española: `;` separador · `;;` encadenar · `,` decimal. Formatos numéricos con `[$-en-US]` para que el punto de miles no choque con el separador.

**Requisito previo:** en `App.Formulas` deben existir `fxCultivo` y `fxHexCultivo` (paso 1 del documento anterior). Sin ellas, `btnAnaNormalizar` marca error.

---

## Resumen de qué corrige cada cambio

| Problema | Dónde se corrige |
|---|---|
| "Operación no válida: división entre cero" | `HtmlText6`, `HtmlText6_1`, `HtmlText8`…`HtmlText8_3`, `colAnaCiclo` — el divisor se calcula dentro del propio HTML con `Max(1; …)` |
| El rango de fechas no filtra | `btnAnaCargar` — `!IsBlank(Request_ID)` rompía la delegación; ahora se filtra solo por fecha y el vacío se quita después |
| Falta la fecha base Start/treat | `btnAnaCargar` + `cmbAnaFechaBase` (nueva opción *Pack/treat start*, queda por defecto) |
| Primero días, después fechas (al revés) | `btnAnaNormalizar` — ahora `Coalesce(fecha; díaGuardado; -1)` |
| Falta el tiempo Created → Requested ship | `btnAnaNormalizar` (`DIAS_REQUERIDO`), `btnAnaFiltrar` (`varAnaLeadReq`), `btnAnaGraficas` (barra comparativa), `cmbAnaMetrica` |
| Cajas mal contadas | `btnAnaFiltrar` — número de caja **distinto por ID**, no filas ni suma |
| SETs mal contados | `btnAnaFiltrar` — únicos por `Request_ID & "|" & SET`; el mismo SET repetido en varias cajas cuenta una vez |
| "Tomato Fresh / Process" como cultivos aparte | `btnAnaNormalizar` con `fxCultivo` |
| Solo salían los 8 primeros | `Gallery4_2`, `Gallery5`, `Gallery5_1`, `Gallery5_2`, `Gallery5_3` — se quita `FirstN` |
| Promedios inflados por líneas | `btnAnaFiltrar` — nueva `colAnaEnvios`: se promedia por envío (ID), no por línea |
| Botón "Guardar días" visible | `btnAnaGuardarDiasVisible` → `Visible = false`; el guardado corre solo y solo donde cambió |

---

# 1. Filtros (`Container140`)

### `cmbAnaFechaBase`

| Propiedad | Fórmula |
|---|---|
| Items | `["Pack/treat start"; "CreatedDate"; "Shipping Date"; "Delivery Date"]` |
| DefaultSelectedItems | `["Pack/treat start"]` |
| InputTextPlaceholder | `"Fecha base"` |
| OnChange | `Select(btnAnaCargar)` |

### `DatePicker2`

| Propiedad | Fórmula |
|---|---|
| Placeholder | `"Desde"` |
| OnChange | `Select(btnAnaCargar)` |

### `DatePicker4`

| Propiedad | Fórmula |
|---|---|
| Placeholder | `"Hasta"` |
| OnChange | `Select(btnAnaCargar)` |

### `cmbAnaCountry`

| Propiedad | Fórmula |
|---|---|
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

### `txtAnaBuscar`

| Propiedad | Fórmula |
|---|---|
| HintText | `"Request ID, SET, PEA, locación, programa, cultivo"` |

### `btnAnaGuardarDiasVisible`

| Propiedad | Fórmula |
|---|---|
| Visible | `false` |

---

# 2. Botones ocultos — el motor

### `btnAnaCargar` · OnSelect

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

---

### `btnAnaNormalizar` · OnSelect

Primero fecha, luego día guardado. `CROP` queda normalizado y `CROP_ORIG` conserva el nombre original para mostrarlo.

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

---

### `btnAnaFiltrar` · OnSelect

Mantiene los nombres de colecciones que ya usan tus galerías (`colAnaCrop`, `colAnaLocation`, `colAnaCountry`, `colAnaPEA`, `colAnaPackaging`, `colAnaEtapas`) y agrega `colAnaEnvios` para promediar por ID.

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
            {ORDEN: 2; ETAPA: "Empaque"; COLOR: "#88BC30"; K: "EMP"};
            {ORDEN: 3; ETAPA: "IP"; COLOR: "#C97E17"; K: "IP"};
            {ORDEN: 4; ETAPA: "GL"; COLOR: "#7E3A66"; K: "GL"};
            {ORDEN: 5; ETAPA: "Transito"; COLOR: "#B23A32"; K: "TRA"};
            {ORDEN: 6; ETAPA: "Aduana a entrega"; COLOR: "#63756B"; K: "ADU"}
        ) As _s;
        With(
            {_v: Filter(ForAll(colAnaEnvios As _e; {D: Switch(_s.K; "LAB"; _e.DIAS_LAB; "EMP"; _e.DIAS_EMPAQUE; "IP"; _e.DIAS_IP; "GL"; _e.DIAS_GL; "TRA"; _e.DIAS_TRANSITO; "ADU"; _e.DIAS_ADUANA; -1)}); D >= 0)};
            {ORDEN: _s.ORDEN; ETAPA: _s.ETAPA; COLOR: _s.COLOR; N: CountRows(_v); DIAS: If(IsEmpty(_v); 0; Round(Average(_v; D); 1))}
        )
    )
);;

ClearCollect(
    colAnaLocation;
    ForAll(Distinct(colAnaEnvios; SHIP_TO_LOCATION) As _l;
        With({_g: Filter(colAnaEnvios; SHIP_TO_LOCATION = _l.Value)};
            {
                SHIP_TO_LOCATION: _l.Value;
                ENVIOS: CountRows(_g);
                SETS: Sum(_g; SETS);
                PACKETS: Sum(_g; PACKETS);
                DIAS_CICLO: With({_d: Filter(_g; DIAS_CICLO >= 0)}; If(IsEmpty(_d); 0; Round(Average(_d; DIAS_CICLO); 1)));
                DIAS_LAB: With({_d: Filter(_g; DIAS_LAB >= 0)}; If(IsEmpty(_d); 0; Round(Average(_d; DIAS_LAB); 1)));
                DIAS_EMPAQUE: With({_d: Filter(_g; DIAS_EMPAQUE >= 0)}; If(IsEmpty(_d); 0; Round(Average(_d; DIAS_EMPAQUE); 1)));
                DIAS_IP: With({_d: Filter(_g; DIAS_IP >= 0)}; If(IsEmpty(_d); 0; Round(Average(_d; DIAS_IP); 1)));
                DIAS_GL: With({_d: Filter(_g; DIAS_GL >= 0)}; If(IsEmpty(_d); 0; Round(Average(_d; DIAS_GL); 1)));
                DIAS_TRANSITO: With({_d: Filter(_g; DIAS_TRANSITO >= 0)}; If(IsEmpty(_d); 0; Round(Average(_d; DIAS_TRANSITO); 1)));
                DIAS_ADUANA: With({_d: Filter(_g; DIAS_ADUANA >= 0)}; If(IsEmpty(_d); 0; Round(Average(_d; DIAS_ADUANA); 1)));
                DIAS_SOLICITUD_ENVIO: With({_d: Filter(_g; DIAS_SOLICITUD_ENVIO >= 0)}; If(IsEmpty(_d); 0; Round(Average(_d; DIAS_SOLICITUD_ENVIO); 1)));
                DIAS_REQUERIDO: With({_d: Filter(_g; DIAS_REQUERIDO >= 0)}; If(IsEmpty(_d); 0; Round(Average(_d; DIAS_REQUERIDO); 1)))
            }
        )
    )
);;

ClearCollect(
    colAnaCountry;
    ForAll(Distinct(colAnaEnvios; SHIP_TO_COUNTRY) As _p;
        With({_g: Filter(colAnaEnvios; SHIP_TO_COUNTRY = _p.Value)};
            {
                SHIP_TO_COUNTRY: _p.Value;
                ENVIOS: CountRows(_g);
                SETS: Sum(_g; SETS);
                PACKETS: Sum(_g; PACKETS);
                DIAS_CICLO: With({_d: Filter(_g; DIAS_CICLO >= 0)}; If(IsEmpty(_d); 0; Round(Average(_d; DIAS_CICLO); 1)))
            }
        )
    )
);;

ClearCollect(
    colAnaPEA;
    ForAll(Distinct(colAnaEnvios; PEA) As _q;
        With({_g: Filter(colAnaEnvios; PEA = _q.Value)};
            {
                PEA: _q.Value;
                ENVIOS: CountRows(_g);
                SETS_UNICOS: Sum(_g; SETS);
                PACKETS: Sum(_g; PACKETS);
                ENTREGADOS: Sum(Filter(_g; ENTREGADO); PACKETS)
            }
        )
    )
);;

ClearCollect(
    colAnaPackaging;
    ForAll(Distinct(colAnaFilt; Packaging_Type) As _b;
        With({_g: Filter(colAnaFilt; Packaging_Type = _b.Value)};
            {
                Packaging_Type: _b.Value;
                USOS: CountRows(_g);
                CAJAS: CountRows(Distinct(Filter(_g; NumBox > 0); Request_ID & "|" & Text(NumBox)));
                PACKETS: Sum(_g; Packets)
            }
        )
    )
);;
Set(varAnaPackagingTop; First(SortByColumns(colAnaPackaging; "CAJAS"; SortOrder.Descending)).Packaging_Type);;
Select(btnAnaGraficas)
```

---

### `btnAnaGraficas` · OnSelect

Arma `colAnaLocBar` (con las dos métricas nuevas) y `colAnaCiclo`, ya sin divisiones por blanco.

```
ClearCollect(
    colAnaLocBar;
    ForAll(
        colAnaLocation As _l;
        {
            SHIP_TO_LOCATION: _l.SHIP_TO_LOCATION;
            ENVIOS: _l.ENVIOS;
            VALOR: Switch(
                Coalesce(cmbAnaMetrica.Selected.Value; "Ciclo total");
                "Laboratorio"; _l.DIAS_LAB;
                "Empaque"; _l.DIAS_EMPAQUE;
                "IP"; _l.DIAS_IP;
                "GL"; _l.DIAS_GL;
                "Transito"; _l.DIAS_TRANSITO;
                "Aduana a entrega"; _l.DIAS_ADUANA;
                "Solicitud a envio"; _l.DIAS_SOLICITUD_ENVIO;
                "Lead solicitado"; _l.DIAS_REQUERIDO;
                _l.DIAS_CICLO
            )
        }
    )
);;
Set(
    varAnaColorMetrica;
    Switch(
        Coalesce(cmbAnaMetrica.Selected.Value; "Ciclo total");
        "Laboratorio"; "#17456B";
        "Empaque"; "#88BC30";
        "IP"; "#C97E17";
        "GL"; "#7E3A66";
        "Transito"; "#B23A32";
        "Aduana a entrega"; "#63756B";
        "Lead solicitado"; "#2e9fd6";
        "#10241C"
    )
);;
ClearCollect(
    colAnaCiclo;
    [
        {
            ORDEN: 1;
            HTML:
                With(
                    {
                        _tot: Max(1; Sum(colAnaEtapas; DIAS));
                        _mr: Max(1; Coalesce(varAnaLeadReq; 0); Coalesce(varAnaSolEnv; 0))
                    };
                    "<div style='font-family:Segoe UI;padding:4px 8px'>" &
                    "<div style='display:flex;align-items:baseline;gap:10px;margin-bottom:6px'>" &
                    "<div style='font-size:26px;font-weight:700;color:#10241C'>" & Text(Coalesce(varAnaCiclo; 0); "[$-en-US]0.0") & "</div>" &
                    "<div style='font-size:12px;color:#5a6b7b'>días promedio de la solicitud a la entrega  ·  " &
                    Text(Coalesce(varAnaEntregados; 0)) & " envíos entregados</div></div>" &
                    "<div style='display:flex;height:22px;border-radius:5px;overflow:hidden;background:#eef2f6'>" &
                    Concat(
                        Sort(colAnaEtapas; ORDEN; SortOrder.Ascending) As _e;
                        If(_e.DIAS > 0; "<div title='" & _e.ETAPA & ": " & Text(_e.DIAS; "[$-en-US]0.0") & " d' style='background:" & _e.COLOR & ";width:" & Text(_e.DIAS / _tot * 100; "[$-en-US]0.##") & "%'></div>"; "")
                    ) & "</div>" &
                    "<div style='display:flex;flex-wrap:wrap;gap:12px;font-size:10px;color:#5a6b7b;margin:6px 0 8px'>" &
                    Concat(
                        Sort(colAnaEtapas; ORDEN; SortOrder.Ascending) As _e;
                        "<span><i style='display:inline-block;width:8px;height:8px;border-radius:2px;background:" & _e.COLOR &
                        ";margin-right:4px'></i>" & _e.ETAPA & " <b style='color:#16273a'>" & Text(_e.DIAS; "[$-en-US]0.0") & " d</b> <span style='color:#93a4b4'>n " & _e.N & "</span></span>"
                    ) & "</div>" &
                    Concat(
                        Table(
                            {n: "Lead solicitado (Created → Requested ship)"; d: Coalesce(varAnaLeadReq; 0); c: "#17456B"};
                            {n: "Real (Created → Shipping)"; d: Coalesce(varAnaSolEnv; 0); c: If(Coalesce(varAnaSolEnv; 0) > Coalesce(varAnaLeadReq; 0); "#d35442"; "#2f9e5f")}
                        ) As _r;
                        "<div style='display:flex;align-items:center;gap:8px;font-size:11px;padding:2px 0'>" &
                        "<div style='width:250px;color:#27322B'>" & _r.n & "</div>" &
                        "<div style='flex:1;background:#eef2f6;border-radius:4px;height:11px;overflow:hidden'><div style='height:11px;border-radius:4px;background:" & _r.c & ";width:" & Text(_r.d / _mr * 100; "[$-en-US]0.##") & "%'></div></div>" &
                        "<div style='width:70px;text-align:right'><b>" & Text(_r.d; "[$-en-US]0.0") & " d</b></div></div>"
                    ) & "</div>"
                )
        }
    ]
)
```

---

### `btnAnaGuardarDias` · OnSelect

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

---

# 3. Tarjetas por cultivo (`Container132`)

Los títulos se quedan igual. Solo cambia la línea de datos de cada uno para mostrar SETs únicos, packets y cajas.

| Control | Propiedad | Fórmula |
|---|---|---|
| Label165_2 (Tomato) | Text | `With({c: LookUp(colAnaCrop; CROP = "Tomato")}; Coalesce(c.SETS_UNICOS; 0) & " SET · " & Text(Coalesce(c.PACKETS; 0); "[$-en-US]#,##0") & " pk · " & Coalesce(c.CAJAS; 0) & " cajas")` |
| Label165_3 (Pepper) | Text | `With({c: LookUp(colAnaCrop; CROP = "Pepper")}; Coalesce(c.SETS_UNICOS; 0) & " SET · " & Text(Coalesce(c.PACKETS; 0); "[$-en-US]#,##0") & " pk · " & Coalesce(c.CAJAS; 0) & " cajas")` |
| Label165_4 (Melon) | Text | `With({c: LookUp(colAnaCrop; CROP = "Melon")}; Coalesce(c.SETS_UNICOS; 0) & " SET · " & Text(Coalesce(c.PACKETS; 0); "[$-en-US]#,##0") & " pk · " & Coalesce(c.CAJAS; 0) & " cajas")` |
| Label165_5 (Watermelon) | Text | `With({c: LookUp(colAnaCrop; CROP = "Watermelon")}; Coalesce(c.SETS_UNICOS; 0) & " SET · " & Text(Coalesce(c.PACKETS; 0); "[$-en-US]#,##0") & " pk · " & Coalesce(c.CAJAS; 0) & " cajas")` |
| Label165_6 (Cucumber) | Text | `With({c: LookUp(colAnaCrop; CROP = "Cucumber")}; Coalesce(c.SETS_UNICOS; 0) & " SET · " & Text(Coalesce(c.PACKETS; 0); "[$-en-US]#,##0") & " pk · " & Coalesce(c.CAJAS; 0) & " cajas")` |

**Color del título según selección** (opcional, para que se note el filtro): en `Label165_1`, `Label166`, `Label167`, `Label168`, `Label169` pon en **Color**, cambiando el nombre del cultivo en cada uno:

```
If(LookUp(colAnaCrop; CROP = "Tomato"; SEL); ColorValue("#B23A32"); ColorValue("#16273a"))
```

### Filtro al tocar una tarjeta (opcional pero recomendado)

Inserta un **Botón (clásico)** transparente dentro de cada contenedor de cultivo (`Container133`, `Container135`, `Container136`, `Container137`, `Container138`), nómbralos `btnCropTomato`, `btnCropPepper`, `btnCropMelon`, `btnCropWatermelon`, `btnCropCucumber`.

| Propiedad | Valor |
|---|---|
| Text | `""` |
| Fill / Color / BorderColor / HoverFill / HoverColor / HoverBorderColor / PressedFill / PressedColor / PressedBorderColor | `RGBA(0; 0; 0; 0)` |
| Height | `Parent.Height` |
| Width | `Parent.Width` |
| OnSelect | ver abajo (cambia el nombre del cultivo en cada botón) |

```
If(
    !IsBlank(LookUp(colAnaCropSel; CROP = "Tomato"; CROP));
    RemoveIf(colAnaCropSel; CROP = "Tomato");
    Collect(colAnaCropSel; {CROP: "Tomato"})
);;
Select(btnAnaFiltrar)
```

---

# 4. Tira de KPIs (`Container132_1`)

### Cajas totales — `Label165_18` (la etiqueta vacía debajo de "Box")

| Propiedad | Fórmula |
|---|---|
| Text | `Text(Coalesce(varAnaCajas; 0); "[$-en-US]#,##0") & " cajas"` |

### Cajas por cultivo (`Container132_3`) — ya funcionan, solo confirma que quedan así

| Control | Propiedad | Fórmula |
|---|---|---|
| Label165_20 | Text | `Text(Coalesce(LookUp(colAnaCrop; CROP = "Tomato"; CAJAS); 0); "[$-en-US]#,##0")` |
| Label165_21 | Text | `Text(Coalesce(LookUp(colAnaCrop; CROP = "Pepper"; CAJAS); 0); "[$-en-US]#,##0")` |
| Label165_22 | Text | `Text(Coalesce(LookUp(colAnaCrop; CROP = "Melon"; CAJAS); 0); "[$-en-US]#,##0")` |
| Label165_23 | Text | `Text(Coalesce(LookUp(colAnaCrop; CROP = "Watermelon"; CAJAS); 0); "[$-en-US]#,##0")` |
| Label165_24 | Text | `Text(Coalesce(LookUp(colAnaCrop; CROP = "Cucumber"; CAJAS); 0); "[$-en-US]#,##0")` |

### KPIs existentes — quedan más explícitos

| Control | Propiedad | Fórmula |
|---|---|---|
| Label165_7 | Text | `"Envíos (IDs)"` |
| Label165_8 | Text | `Text(Coalesce(varAnaEnvios; 0); "[$-en-US]#,##0") & "  ·  " & Coalesce(varAnaEntregados; 0) & " entregados"` |
| Label166_1 | Text | `"SETs enviados"` |
| Label165_9 | Text | `Text(Coalesce(varAnaSets; 0); "[$-en-US]#,##0") & "  (únicos por ID)"` |
| Label165_10 | Text | `Text(Coalesce(varAnaPackets; 0); "[$-en-US]#,##0")` |
| Label168_1 | Text | `"Peso (kg)"` |
| Label165_11 | Text | `Text(Coalesce(varAnaNeto; 0); "[$-en-US]#,##0.0") & " neto · " & Text(Coalesce(varAnaBruto; 0); "[$-en-US]#,##0.0") & " bruto"` |
| Label169_1 | Text | `"Entregas a tiempo"` |
| Label165_12 | Text | `Text(Coalesce(varAnaOnTime; 0); "[$-en-US]0.0") & " %  ·  " & Coalesce(varAnaConFecha; 0) & " con fecha"` |
| Label165_12 | Color | `If(Coalesce(varAnaOnTime; 0) >= 85; RGBA(92; 138; 22; 1); RGBA(201; 126; 23; 1))` |

### Nuevas tarjetas: Lead solicitado y Solicitud → envío

En el árbol, **copia `Container133_1` y pégalo dos veces** dentro de `Container132_1` (Ctrl+C / Ctrl+V sobre el contenedor). Renombra:

**Copia 1** → `conAnaKpiLead`, con sus hijos `lblAnaLeadTit` y `lblAnaLeadVal`:

| Control | Propiedad | Fórmula |
|---|---|---|
| lblAnaLeadTit | Text | `"Lead solicitado"` |
| lblAnaLeadVal | Text | `Text(Coalesce(varAnaLeadReq; 0); "[$-en-US]0.0") & " d"` |

**Copia 2** → `conAnaKpiSolEnv`, con `lblAnaSolEnvTit` y `lblAnaSolEnvVal`:

| Control | Propiedad | Fórmula |
|---|---|---|
| lblAnaSolEnvTit | Text | `"Solicitud → envío"` |
| lblAnaSolEnvVal | Text | `Text(Coalesce(varAnaSolEnv; 0); "[$-en-US]0.0") & " d"` |
| lblAnaSolEnvVal | Color | `If(Coalesce(varAnaSolEnv; 0) > Coalesce(varAnaLeadReq; 0); RGBA(211; 84; 66; 1); RGBA(47; 158; 95; 1))` |

---

# 5. Gráfica del ciclo (`Container141` + `Gallery2`)

| Control | Propiedad | Fórmula |
|---|---|---|
| lblAnaCicloTit | Text | `"Ciclo completo · dónde se va el tiempo (promedio por envío)"` |
| Gallery2 | Height | `210` |
| Gallery2 | TemplateSize | `Parent.Width` |
| HtmlViewer (dentro de `conAnaCiclo`) | HtmlText | `ThisItem.HTML` *(sin cambio)* |

El HTML ya viene armado desde `btnAnaGraficas` con los `Max(1; …)`, así que aquí no hay más que tocar.

---

# 6. Resumen por cultivo (`Gallery3`)

| Control | Propiedad | Fórmula |
|---|---|---|
| Gallery3 | Items | `SortByColumns(Filter(colAnaCrop; ENVIOS > 0); "PACKETS"; SortOrder.Descending)` |
| Gallery3 | ShowScrollbar | `false` |
| Label174 (CROP) | Text | `ThisItem.CROP` |
| Label174 (CROP) | Tooltip | `ThisItem.NOMBRES` |
| Label174 (CROP) | Color | `ColorValue(ThisItem.COLOR)` |
| Label174 (CROP) | FontWeight | `If(ThisItem.SEL; FontWeight.Bold; FontWeight.Normal)` |
| Label175 | Text | `Text(ThisItem.SETS_UNICOS; "[$-en-US]#,##0")` |
| Label176 | Text | `Text(ThisItem.ENVIOS; "[$-en-US]#,##0")` |
| Label177 | Text | `Text(ThisItem.PACKETS; "[$-en-US]#,##0")` |
| Label178 | Text | `Text(ThisItem.NET_WEIGHT; "[$-en-US]#,##0.0")` |
| Label179 | Text | `Text(ThisItem.GROSS_WEIGHT; "[$-en-US]#,##0.0")` |
| Label180 | Text | `Text(ThisItem.CAJAS; "[$-en-US]#,##0")` |
| Label181 | Text | `Text(ThisItem.DIAS_CICLO; "[$-en-US]0.0") & " d"` |

**Fila de totales (`Container144`):**

| Control | Propiedad | Fórmula |
|---|---|---|
| Label175_1 | Text | `Text(Coalesce(varAnaSets; 0); "[$-en-US]#,##0")` |
| Label176_1 | Text | `Text(Coalesce(varAnaEnvios; 0); "[$-en-US]#,##0")` |
| Label177_1 | Text | `Text(Coalesce(varAnaPackets; 0); "[$-en-US]#,##0")` |
| Label178_1 | Text | `Text(Coalesce(varAnaNeto; 0); "[$-en-US]#,##0.0")` |
| Label179_1 | Text | `Text(Coalesce(varAnaBruto; 0); "[$-en-US]#,##0.0")` |
| Label180_1 | Text | `Text(Coalesce(varAnaCajas; 0); "[$-en-US]#,##0")` |
| Label181_1 | Text | `Text(Coalesce(varAnaCiclo; 0); "[$-en-US]0.0") & " d"` |

---

# 7. `conAnaTiempos` — dónde se va el tiempo y días por locación

### `HtmlText6` (dentro de `Gallery4`, etapas) · HtmlText

Aquí estaba una de las divisiones por blanco. El divisor ahora se calcula en la propia fórmula.

```
With(
    {_mx: Max(1; Max(colAnaEtapas; DIAS))};
    "<div style='font-family:Segoe UI;display:flex;align-items:center;gap:8px;font-size:11px'>" &
    "<div style='width:110px;color:#5a6b7b'>" & ThisItem.ETAPA & "</div>" &
    "<div style='flex:1;background:#eef2f6;border-radius:4px;height:16px;overflow:hidden'>" &
    "<div style='height:16px;border-radius:4px;background:" & ThisItem.COLOR & ";width:" &
    Text(ThisItem.DIAS / _mx * 100; "[$-en-US]0.##") & "%'></div></div>" &
    "<div style='width:86px;text-align:right;font-weight:600;color:#16273a'>" &
    Text(ThisItem.DIAS; "[$-en-US]0.0") & " d <span style='color:#93a4b4;font-weight:400'>n " & ThisItem.N & "</span></div></div>"
)
```

### `Label172` · Text

```
"Dónde se va el tiempo  ·  promedio por envío (ID)"
```

### `cmbAnaMetrica`

| Propiedad | Fórmula |
|---|---|
| Items | `["Ciclo total"; "Lead solicitado"; "Solicitud a envio"; "Laboratorio"; "Empaque"; "IP"; "GL"; "Transito"; "Aduana a entrega"]` |
| DefaultSelectedItems | `["Ciclo total"]` |
| OnChange | `Select(btnAnaGraficas)` |

### `Gallery4_2` (días por locación)

| Propiedad | Fórmula |
|---|---|
| Items | `SortByColumns(colAnaLocBar; "VALOR"; SortOrder.Descending)` |
| ShowScrollbar | `true` |

### `HtmlText6_1` · HtmlText

```
With(
    {_mx: Max(1; Max(colAnaLocBar; VALOR))};
    "<div style='font-family:Segoe UI;display:flex;align-items:center;gap:8px;font-size:11px'>" &
    "<div style='width:120px;color:#5a6b7b;overflow:hidden;white-space:nowrap;text-overflow:ellipsis'>" & ThisItem.SHIP_TO_LOCATION & "</div>" &
    "<div style='flex:1;background:#eef2f6;border-radius:4px;height:16px;overflow:hidden'>" &
    "<div style='height:16px;border-radius:4px;background:" & Coalesce(varAnaColorMetrica; "#10241C") & ";width:" &
    Text(ThisItem.VALOR / _mx * 100; "[$-en-US]0.##") & "%'></div></div>" &
    "<div style='width:92px;text-align:right;color:#16273a'><b>" & Text(ThisItem.VALOR; "[$-en-US]0.0") &
    " d</b><span style='color:#93a4b4'> · " & Text(ThisItem.ENVIOS) & "</span></div></div>"
)
```

### `lblAnaMetrica` · Text

```
"Días promedio por locación  ·  " & Coalesce(cmbAnaMetrica.Selected.Value; "Ciclo total")
```

---

# 8. `Container148` — rankings completos (ya no solo 8)

## Países (`Gallery5`)

| Control | Propiedad | Fórmula |
|---|---|---|
| Gallery5 | Items | `SortByColumns(colAnaCountry; "ENVIOS"; SortOrder.Descending)` |
| Gallery5 | ShowScrollbar | `true` |
| Label173 | Text | `"Envíos por país  ·  " & CountRows(colAnaCountry) & " países"` |

**`HtmlText8` · HtmlText**

```
With(
    {_mx: Max(1; Max(colAnaCountry; ENVIOS))};
    "<div style='font-family:Segoe UI;display:flex;align-items:center;gap:8px;font-size:11px'>" &
    "<div style='width:110px;color:#5a6b7b;overflow:hidden;white-space:nowrap;text-overflow:ellipsis'>" & ThisItem.SHIP_TO_COUNTRY & "</div>" &
    "<div style='flex:1;background:#eef2f6;border-radius:4px;height:14px;overflow:hidden'>" &
    "<div style='height:14px;border-radius:4px;background:#17456B;width:" & Text(ThisItem.ENVIOS / _mx * 100; "[$-en-US]0.##") & "%'></div></div>" &
    "<div style='width:130px;text-align:right;color:#16273a'><b>" & ThisItem.ENVIOS &
    "</b><span style='color:#93a4b4'> env · " & Text(ThisItem.PACKETS; "[$-en-US]#,##0") & " pk · " & Text(ThisItem.DIAS_CICLO; "[$-en-US]0.0") & " d</span></div></div>"
)
```

## Locaciones (`Gallery5_1`)

| Control | Propiedad | Fórmula |
|---|---|---|
| Gallery5_1 | Items | `SortByColumns(colAnaLocation; "ENVIOS"; SortOrder.Descending)` |
| Gallery5_1 | ShowScrollbar | `true` |
| Label173_1 | Text | `"Envíos por locación  ·  " & CountRows(colAnaLocation) & " locaciones"` |

**`HtmlText8_1` · HtmlText**

```
With(
    {_mx: Max(1; Max(colAnaLocation; ENVIOS))};
    "<div style='font-family:Segoe UI;display:flex;align-items:center;gap:8px;font-size:11px'>" &
    "<div style='width:120px;color:#5a6b7b;overflow:hidden;white-space:nowrap;text-overflow:ellipsis'>" & ThisItem.SHIP_TO_LOCATION & "</div>" &
    "<div style='flex:1;background:#eef2f6;border-radius:4px;height:14px;overflow:hidden'>" &
    "<div style='height:14px;border-radius:4px;background:#88BC30;width:" & Text(ThisItem.ENVIOS / _mx * 100; "[$-en-US]0.##") & "%'></div></div>" &
    "<div style='width:150px;text-align:right;color:#16273a'><b>" & ThisItem.ENVIOS &
    "</b><span style='color:#93a4b4'> env · " & Text(ThisItem.PACKETS; "[$-en-US]#,##0") & " pk · " & ThisItem.SETS & " SET</span></div></div>"
)
```

## PEA (`Gallery5_2`)

| Control | Propiedad | Fórmula |
|---|---|---|
| Gallery5_2 | Items | `SortByColumns(colAnaPEA; "PACKETS"; SortOrder.Descending)` |
| Gallery5_2 | ShowScrollbar | `true` |
| Label173_2 | Text | `"Packets por PEA  ·  " & CountRows(colAnaPEA) & " PEA"` |

**`HtmlText8_2` · HtmlText**

```
With(
    {_mx: Max(1; Max(colAnaPEA; PACKETS))};
    "<div style='font-family:Segoe UI;display:flex;align-items:center;gap:8px;font-size:11px'>" &
    "<div style='width:130px;color:#5a6b7b;overflow:hidden;white-space:nowrap;text-overflow:ellipsis'>" & ThisItem.PEA & "</div>" &
    "<div style='flex:1;background:#eef2f6;border-radius:4px;height:14px;overflow:hidden'>" &
    "<div style='height:14px;border-radius:4px;background:#C97E17;width:" & Text(ThisItem.PACKETS / _mx * 100; "[$-en-US]0.##") & "%'></div></div>" &
    "<div style='width:170px;text-align:right;color:#16273a'><b>" & Text(ThisItem.PACKETS; "[$-en-US]#,##0") &
    "</b><span style='color:#93a4b4'> pk · " & Text(ThisItem.ENTREGADOS; "[$-en-US]#,##0") & " entr. · " & ThisItem.SETS_UNICOS & " SET</span></div></div>"
)
```

## Tipo de caja (`Gallery5_3`)

| Control | Propiedad | Fórmula |
|---|---|---|
| Gallery5_3 | Items | `SortByColumns(colAnaPackaging; "CAJAS"; SortOrder.Descending)` |
| Gallery5_3 | ShowScrollbar | `true` |
| Label173_3 | Text | `"Tipo de caja / paquete  ·  más usado: " & Coalesce(varAnaPackagingTop; "—")` |

**`HtmlText8_3` · HtmlText**

```
With(
    {_mx: Max(1; Max(colAnaPackaging; CAJAS))};
    "<div style='font-family:Segoe UI;display:flex;align-items:center;gap:8px;font-size:11px'>" &
    "<div style='width:130px;color:#5a6b7b;overflow:hidden;white-space:nowrap;text-overflow:ellipsis'>" & ThisItem.Packaging_Type & "</div>" &
    "<div style='flex:1;background:#eef2f6;border-radius:4px;height:14px;overflow:hidden'>" &
    "<div style='height:14px;border-radius:4px;background:#7E3A66;width:" & Text(ThisItem.CAJAS / _mx * 100; "[$-en-US]0.##") & "%'></div></div>" &
    "<div style='width:160px;text-align:right;color:#16273a'><b>" & ThisItem.CAJAS &
    "</b><span style='color:#93a4b4'> cajas · " & ThisItem.USOS & " lín · " & Text(ThisItem.PACKETS; "[$-en-US]#,##0") & " pk</span></div></div>"
)
```

---

# 9. Encabezado (`Container139`)

| Control | Propiedad | Fórmula |
|---|---|---|
| lblAnaSello | Text | `"EXPORT ANALYST"` |

**Opcional**, para ver el contexto del análisis: inserta una **Etiqueta de texto** `lblAnaContexto` dentro de `Container139`, debajo de `lblAnaSello`.

| Propiedad | Fórmula |
|---|---|
| Text | `"Base: " & Coalesce(varAnaBase; "Pack/treat start") & "  ·  " & If(IsBlank(varAnaDesde); ""; Text(varAnaDesde; "[$-en-US]dd/mm/yyyy") & " – " & Text(DateAdd(varAnaHasta; -1; TimeUnit.Days); "[$-en-US]dd/mm/yyyy")) & "  ·  " & CountRows(colAnaFilt) & " líneas · " & CountRows(colAnaEnvios) & " envíos" & If(IsEmpty(colAnaCropSel); ""; "  ·  filtro: " & Concat(colAnaCropSel; CROP; ", "))` |
| Align | `Align.Center` |
| AlignInContainer | `AlignInContainer.Stretch` |
| Color | `RGBA(90; 107; 123; 1)` |
| Size | `10` |
| Height | `20` |

---

# 10. Pruebas

1. Entra a **Export analysis**: no debe aparecer el aviso rojo de división entre cero, ni siquiera antes de que cargue.
2. Cambia **Desde / Hasta** con base *Pack/treat start*: el conteo de líneas y envíos debe moverse.
3. Cambia la base a *CreatedDate* y luego a *Delivery Date*: el conjunto cambia en cada una.
4. Busca un ID que tenga el mismo SET en 2 cajas: en *Resumen por cultivo* debe contar **1 SET** y **2 cajas** (si los números de caja son distintos).
5. Si tienes registros con CROP "Tomato Fresh" y "Tomato Process", ambos deben sumar en la fila **Tomato**; pasa el mouse sobre el nombre y el tooltip muestra los nombres originales.
6. Cambia `cmbAnaMetrica` a *Lead solicitado*: la gráfica por locación cambia de valor y de color.
7. Verifica que en locaciones, PEA, países y tipo de caja aparezcan **todos** los registros, con barra de desplazamiento.
8. Revisa en SharePoint que las columnas de días se hayan actualizado solas en las filas que tenían fechas nuevas.
