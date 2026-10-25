# DAX measures, calculated columns and Power Query

All measures live in the table `'RAW DATA(JAN-SEPT)'`. Exported from the report file.

## Contents

- [Core counts](#core-counts)
- [Rates](#rates)
- [KPI card sub-labels](#kpi-card-sub-labels)
- [Campaign insight cards](#campaign-insight-cards)
- [Formatting helpers](#formatting-helpers)
- [SVG visuals (Data category = Image URL, shown in a Table or Matrix)](#svg-visuals-data-category--image-url-shown-in-a-table-or-matrix)
- [Calculated columns](#calculated-columns)
- [Power Query](#power-query)

## Core counts

### Uploaded

Format: `#,0`

```dax
Uploaded =
COUNTROWS ( 'RAW DATA(JAN-SEPT)' )
```

### Attempted

Format: `0`

```dax
Attempted =
CALCULATE(
    COUNTROWS('RAW DATA(JAN-SEPT)'),
    'RAW DATA(JAN-SEPT)'[callstatus] = "Attempted"
)
```

### Not Attempted

Format: `#,0`

```dax
Not Attempted =
CALCULATE (
    COUNTROWS ( 'RAW DATA(JAN-SEPT)' ),
    'RAW DATA(JAN-SEPT)'[callstatus] = "Notattempted"
)
```

### Quote_sent

Format: `0`

```dax
Quote_sent =
CALCULATE(COUNTROWS('RAW DATA(JAN-SEPT)'),'RAW DATA(JAN-SEPT)'[quote_generated]="YES")
```

### Link_sent_count

Format: `0`

```dax
Link_sent_count =
CALCULATE(
    COUNTROWS('RAW DATA(JAN-SEPT)'),
    'RAW DATA(JAN-SEPT)'[Link_sent] = "YES"
)
```

### Renewed

Format: `#,0`

```dax
Renewed =
CALCULATE(
    COUNTROWS('RAW DATA(JAN-SEPT)'),
    'RAW DATA(JAN-SEPT)'[autopulse_renewal] = "YES"
)
```

### Whatsapp_renewal_count

Format: `0`

```dax
Whatsapp_renewal_count =
CALCULATE(
    COUNTROWS('RAW DATA(JAN-SEPT)'),
    'RAW DATA(JAN-SEPT)'[whatsapp_renewal] = "YES"
)
```

### WhatsApp Renewals

Format: `#,0`

```dax
WhatsApp Renewals =
CALCULATE (
    COUNTROWS ( 'RAW DATA(JAN-SEPT)' ),
    'RAW DATA(JAN-SEPT)'[whatsapp_renewal] = "YES"
)
```

### Not Interested

Format: `0`

```dax
Not Interested =
CALCULATE (
    COUNTROWS('RAW DATA(JAN-SEPT)'),
    'RAW DATA(JAN-SEPT)'[lastcalldisposition] = "Renewal Not Required",
    'RAW DATA(JAN-SEPT)'[tertiary_disposition] IN {
        "High Renewal Premium",
        "Distance from Dealer Location",
        "Dissatisfied with Claim",
        "Dissatisfied with previous service",
        "Dissatisfied with Sales",
        "Other Service"
    }
)
```

### Follow-ups

Format: `0`

```dax
Follow-ups =
CALCULATE(COUNT ( 'RAW DATA(JAN-SEPT)'[followupdate]),'RAW DATA(JAN-SEPT)'[attempt_count]>5)
```

### Total Premium

Format: `"₹"\ #,0;#,0\ -"₹";"₹"\ #,0`

```dax
Total Premium =
CALCULATE( SUM('RAW DATA(JAN-SEPT)'[premium_value]))
```

### Avg Premium

Format: `#,0`

```dax
Avg Premium =
CALCULATE (
    AVERAGE ( 'RAW DATA(JAN-SEPT)'[premium_value]),
    'RAW DATA(JAN-SEPT)'[autopulse_renewal] = "Yes"
)
```

### Avg Attempts

```dax
Avg Attempts =
CALCULATE (
    AVERAGE ( 'RAW DATA(JAN-SEPT)'[attempt_count] ),
    'RAW DATA(JAN-SEPT)'[callstatus] = "Attempted"
)
```

## Rates

### Attempt %

Format: `0.0%;-0.0%;0.0%`

```dax
Attempt % =
DIVIDE ( [Attempted], COUNTROWS ( 'RAW DATA(JAN-SEPT)' ) )
```

### Quote %

Format: `0.00%;-0.00%;0.00%`

```dax
Quote % =
DIVIDE ( [Quote_sent], COUNTROWS('RAW DATA(JAN-SEPT)' ))
```

### Link to Renewal %

Format: `0.0%;-0.0%;0.0%`

```dax
Link to Renewal % =
DIVIDE ( [Renewed], [Link_sent_count] )
```

### Conversion %

Format: `0.00%;-0.00%;0.00%`

```dax
Conversion % =
DIVIDE ( [Renewed], COUNTROWS ( 'RAW DATA(JAN-SEPT)' ) )
```

### Conversion % (Matrix)

Format: `0.0%;-0.0%;0.0%`

```dax
Conversion % (Matrix) =
VAR _Att = [Attempted]
RETURN
    IF ( _Att < 10, BLANK (), DIVIDE ( [Renewed], _Att ) )
```

### Lead Share %

Format: `0.0%;-0.0%;0.0%`

```dax
Lead Share % =
DIVIDE (
    COUNTROWS ( 'RAW DATA(JAN-SEPT)' ),
    CALCULATE ( COUNTROWS ( 'RAW DATA(JAN-SEPT)' ), REMOVEFILTERS ( 'RAW DATA(JAN-SEPT)'[channel] ) )
)
```

### Outcome Share

Format: `0.0%;-0.0%;0.0%`

```dax
Outcome Share =
DIVIDE (
    CALCULATE (
        COUNTROWS ( 'RAW DATA(JAN-SEPT)' ),
        'RAW DATA(JAN-SEPT)'[callstatus] = "Attempted"
    ),
    CALCULATE (
        COUNTROWS ( 'RAW DATA(JAN-SEPT)' ),
        'RAW DATA(JAN-SEPT)'[callstatus] = "Attempted",
        REMOVEFILTERS ( 'RAW DATA(JAN-SEPT)'[Outcome Group], 'RAW DATA(JAN-SEPT)'[lastcalldisposition] )
    )
)
```

## KPI card sub-labels

### Uploaded Sub

```dax
Uploaded Sub =
"Total leads"
```

### Attempted Sub

```dax
Attempted Sub =
FORMAT ( DIVIDE ( [Attempted], COUNTROWS ( 'RAW DATA(JAN-SEPT)' ) ), "0.0%" ) & " of uploaded"
```

### Quote Sub

```dax
Quote Sub =
FORMAT ( DIVIDE ( [Quote_sent], [Attempted] ), "0.0%" ) & " of attempted"
```

### Link Sub

```dax
Link Sub =
FORMAT ( DIVIDE ( [Link_sent_count], [Attempted] ), "0.0%" ) & " of attempted"
```

### Renewed Sub

```dax
Renewed Sub =
FORMAT ( DIVIDE ( [Renewed], [Attempted] ), "0.00%" ) & " of attempted"
```

### WhatsApp Sub

```dax
WhatsApp Sub =
"₹" &
FORMAT(
    CALCULATE(
        SUM('RAW DATA(JAN-SEPT)'[premium_value]),
        'RAW DATA(JAN-SEPT)'[whatsapp_renewal] = "YES"
    ) / 1000000,
    "0.0"
) & "M premium"
```

### Not Interested Sub

```dax
Not Interested Sub =
FORMAT ( DIVIDE ( [Not Interested], [Attempted] ), "0.0%" ) & " of attempted"
```

### Premium Sub

```dax
Premium Sub =
"₹" & FORMAT ( DIVIDE ( SUM ( 'RAW DATA(JAN-SEPT)'[premium_value] ), [Renewed] ), "#,##0" ) & " avg / policy"
```

## Campaign insight cards

### L2R Gap

```dax
L2R Gap =
VAR _Ren =
    CALCULATE ( DIVIDE ( [Renewed], [Link_sent_count] ), 'RAW DATA(JAN-SEPT)'[campignname] = "renewal" )
VAR _Roll =
    CALCULATE ( DIVIDE ( [Renewed], [Link_sent_count] ), 'RAW DATA(JAN-SEPT)'[campignname] = "rollover" )
RETURN
    DIVIDE ( _Ren, _Roll )
```

### L2R Gap Sub

```dax
L2R Gap Sub =
"Renewal links vs rollover"
```

### Rollover Not Called

Format: `0`

```dax
Rollover Not Called =
CALCULATE (
    COUNTROWS ( 'RAW DATA(JAN-SEPT)' ),
    'RAW DATA(JAN-SEPT)'[campignname] = "rollover",
    'RAW DATA(JAN-SEPT)'[callstatus] = "Notattempted"
)
```

### Rollover Not Called Sub

```dax
Rollover Not Called Sub =
VAR _RollLeads =
    CALCULATE ( COUNTROWS ( 'RAW DATA(JAN-SEPT)' ), 'RAW DATA(JAN-SEPT)'[campignname] = "rollover" )
RETURN
    FORMAT ( DIVIDE ( [Rollover Not Called], _RollLeads ), "0.0%" ) & " of rollover leads"
```

### Rollover Premium Share

```dax
Rollover Premium Share =
VAR _Roll =
    CALCULATE ( SUM ( 'RAW DATA(JAN-SEPT)'[premium_value] ), 'RAW DATA(JAN-SEPT)'[campignname] = "rollover" )
VAR _All =
    CALCULATE ( SUM ( 'RAW DATA(JAN-SEPT)'[premium_value] ), REMOVEFILTERS ( 'RAW DATA(JAN-SEPT)'[campignname] ) )
RETURN
    DIVIDE ( _Roll, _All )
```

### Rollover Premium Sub

```dax
Rollover Premium Sub =
VAR _Roll =
    CALCULATE ( SUM ( 'RAW DATA(JAN-SEPT)'[premium_value] ), 'RAW DATA(JAN-SEPT)'[campignname] = "rollover" )
VAR _All =
    CALCULATE ( SUM ( 'RAW DATA(JAN-SEPT)'[premium_value] ), REMOVEFILTERS ( 'RAW DATA(JAN-SEPT)'[campignname] ) )
RETURN
    "₹" & FORMAT ( _Roll / 1000000, "0.0" ) & "M of ₹" & FORMAT ( _All / 1000000, "0.0" ) & "M"
```

## Formatting helpers

### Attempt Label Colour

```dax
Attempt Label Colour =
IF ( [Attempt %] < 0.4, "#A35A00", "#2B3040" )
```

### Backlog Bar Colour

```dax
Backlog Bar Colour =
VAR _Rank =
    RANKX (
        ALLSELECTED ( 'RAW DATA(JAN-SEPT)'[DueMonth], 'RAW DATA(JAN-SEPT)'[Month No] ),
        [Not Attempted]
    )
RETURN
    IF ( _Rank <= 2, "#1A237E", "#A9CBF4" )
```

### Backlog Note

```dax
Backlog Note =
VAR _Total = [Not Attempted]
VAR _Months =
    ADDCOLUMNS (
        VALUES ( 'RAW DATA(JAN-SEPT)'[DueMonth] ),
        "@NA", [Not Attempted],
        "@No", CALCULATE ( MAX ( 'RAW DATA(JAN-SEPT)'[Month No] ) )
    )
VAR _Top2   = TOPN ( 2, _Months, [@NA], DESC )
VAR _Top2NA = SUMX ( _Top2, [@NA] )
VAR _Names  =
    CONCATENATEX ( _Top2, LEFT ( 'RAW DATA(JAN-SEPT)'[DueMonth], 3 ), " + ", [@No], ASC )
RETURN
    IF (
        COUNTROWS ( _Months ) > 2,
        _Names & " = " & FORMAT ( _Top2NA / 1000, "0" ) & "K of "
            & FORMAT ( _Total / 1000, "0" ) & "K uncalled leads",
        FORMAT ( _Total, "#,##0" ) & " uncalled leads"
    )
```

## SVG visuals (Data category = Image URL, shown in a Table or Matrix)

### Funnel SVG

Data category: `ImageUrl`

```dax
Funnel SVG =
-- ===== DATA: keep the 5 lines from your working version =====
VAR _Up  = COUNTROWS('RAW DATA(JAN-SEPT)')
VAR _Att = 'RAW DATA(JAN-SEPT)'[Attempted]
VAR _Q   = 'RAW DATA(JAN-SEPT)'[Quote_sent]
VAR _L   = 'RAW DATA(JAN-SEPT)'[Link_sent_count]
VAR _R   = 'RAW DATA(JAN-SEPT)'[Renewed]
-- ===== SIZES in px (1 pt = 1.33 px) =====
VAR _W       = 460                 -- Image size › Width
VAR _Top     = 40                  -- space for the title
VAR _Step    = 120                  -- height of one stage
VAR _H       = _Top + 5 * _Step    -- 440 = Image size › Height
VAR _TitlePx = 19                  -- 14 pt
VAR _LabelPx = 15                  -- 11 pt
VAR _ValuePx = 19                  -- 12 pt
VAR _BarH    = 28
VAR _MaxBar  = 320
-- ===== STAGES =====
VAR _Stages =
    UNION (
        ROW ( "i", 0, "Stage", "Uploaded",   "Val", _Up,  "Sub", "100%" ),
        ROW ( "i", 1, "Stage", "Attempted",  "Val", _Att, "Sub", FORMAT ( DIVIDE ( _Att, _Up ), "0.0%" ) & " of uploaded" ),
        ROW ( "i", 2, "Stage", "Quote Sent", "Val", _Q,   "Sub", FORMAT ( DIVIDE ( _Q, _Att ), "0.0%" ) & " of attempted" ),
        ROW ( "i", 3, "Stage", "Link Sent",  "Val", _L,   "Sub", FORMAT ( DIVIDE ( _L, _Q ), "0.0%" ) & " of quotes" ),
        ROW ( "i", 4, "Stage", "Renewed",    "Val", _R,   "Sub", FORMAT ( DIVIDE ( _R, _L ), "0.0%" ) & " of links" )
    )
VAR _Body =
    CONCATENATEX (
        _Stages,
        VAR _y    = _Top + [i] * _Step
        VAR _bw   = ROUND ( MAX ( 6, DIVIDE ( [Val], _Up ) * _MaxBar ), 0 )
        VAR _x    = ROUND ( ( _W - _bw ) / 2, 0 )
        VAR _fill = IF ( [i] = 4, "rgb(26,35,126)", "rgb(22,131,232)" )
        RETURN
            "<text x='" & _W / 2 & "' y='" & _y + _LabelPx & "' text-anchor='middle' font-size='" & _LabelPx & "' font-weight='600' fill='rgb(74,81,104)'>" & [Stage] & "</text>"
                & "<rect x='" & _x & "' y='" & _y + _LabelPx + 7 & "' width='" & _bw & "' height='" & _BarH & "' rx='4' fill='" & _fill & "'/>"
                & "<text x='" & _W / 2 & "' y='" & _y + _LabelPx + 7 + _BarH + _ValuePx + 4 & "' text-anchor='middle' font-size='" & _ValuePx & "' fill='rgb(43,48,64)'>"
                & "<tspan font-weight='700'>" & FORMAT ( [Val], "#,##0" ) & "</tspan> &#183; " & [Sub] & "</text>",
        "",
        [i], ASC
    )
VAR _Svg =
    "<svg xmlns='http://www.w3.org/2000/svg' width='" & _W & "' height='" & _H & "' viewBox='0 0 " & _W & " " & _H & "' font-family='Segoe UI, sans-serif'>"
        & "<rect width='" & _W & "' height='" & _H & "' fill='white'/>"
        & "<text x='" & _W / 2 & "' y='26' text-anchor='middle' font-size='" & _TitlePx & "' fill='rgb(37,36,35)'>Renewal Funnel</text>"
        & _Body
        & "</svg>"
RETURN
    "data:image/svg+xml;utf8," & SUBSTITUTE ( SUBSTITUTE ( _Svg, "%", "%25" ), "#", "%23" )
```

### Channel Funnel SVG

Data category: `ImageUrl`

```dax
Channel Funnel SVG =
VAR _Ch   = SELECTEDVALUE ( 'RAW DATA(JAN-SEPT)'[channel], "All" )
VAR _Up   = COUNTROWS ( 'RAW DATA(JAN-SEPT)' )
VAR _Att  = [Attempted]
VAR _Q    = [Quote_sent]
VAR _L    = [Link_sent_count]
VAR _R    = [Renewed]
VAR _L2R  = DIVIDE ( _R, _L )
-- ===== FONT SIZES in px : change these =====
VAR _NamePx      = 18      -- channel name (OEM, Agency, HOM)
VAR _LabelPx     = 17      -- Uploaded, Attempted, Quote Sent ...
VAR _ValuePx     = 17      -- 465,866 ...
VAR _FootLabelPx = 16      -- "Link -> Renewal"
VAR _FootValuePx = 26      -- 35.0%
-- ===== IMAGE SIZE in px : must match matrix Image size =====
VAR _W = 350
VAR _H = 250
-- ===== layout : calculated automatically =====
VAR _LabelW = ROUND ( _LabelPx * 6, 0 )
VAR _ValW   = ROUND ( _ValuePx * 4.3, 0 )
VAR _BarX   = _LabelW
VAR _MaxBar = _W - _LabelW - _ValW - 8
VAR _ValX   = _BarX + _MaxBar + 8
VAR _HeadH  = _NamePx + 16
VAR _FootH  = _FootValuePx + 24
VAR _RowH   = ROUND ( ( _H - _HeadH - _FootH ) / 5, 0 )
VAR _BarH   = ROUND ( _LabelPx * 1.1, 0 )
VAR _LineY  = _HeadH + 5 * _RowH + 4
VAR _FootY  = _H - 10
VAR _NameY  = _NamePx + 2
VAR _DotR   = ROUND ( _NamePx * 0.38, 0 )
VAR _DotY   = ROUND ( _NameY - _NamePx * 0.35, 0 )
-- ===== colours =====
VAR _Color =
    SWITCH ( _Ch,
        "OEM",    "rgb(30,111,217)",
        "Agency", "rgb(232,137,12)",
        "HOM",    "rgb(20,153,138)",
        "rgb(22,131,232)" )
VAR _L2RColor = IF ( _L2R < 0.1, "rgb(163,90,0)", "rgb(26,35,126)" )
-- ===== stages =====
VAR _Stages =
    UNION (
        ROW ( "i", 0, "Stage", "Uploaded",   "Val", _Up ),
        ROW ( "i", 1, "Stage", "Attempted",  "Val", _Att ),
        ROW ( "i", 2, "Stage", "Quote Sent", "Val", _Q ),
        ROW ( "i", 3, "Stage", "Link Sent",  "Val", _L ),
        ROW ( "i", 4, "Stage", "Renewed",    "Val", _R )
    )
VAR _Rows =
    CONCATENATEX (
        _Stages,
        VAR _BarY  = _HeadH + [i] * _RowH + ROUND ( ( _RowH - _BarH ) / 2, 0 )
        VAR _TextY = _BarY + ROUND ( _BarH / 2 + _LabelPx * 0.35, 0 )
        VAR _bw    = IF ( [Val] > 0, ROUND ( MAX ( 2, DIVIDE ( [Val], _Up ) * _MaxBar ), 0 ), 0 )
        RETURN
            "<text x='0' y='" & _TextY & "' font-size='" & _LabelPx & "' fill='rgb(74,81,104)'>" & [Stage] & "</text>"
                & "<rect x='" & _BarX & "' y='" & _BarY & "' width='" & _bw & "' height='" & _BarH & "' rx='3' fill='" & _Color & "'/>"
                & "<text x='" & _ValX & "' y='" & _TextY & "' font-size='" & _ValuePx & "' fill='rgb(43,48,64)'>" & FORMAT ( [Val], "#,##0" ) & "</text>",
        "",
        [i], ASC
    )
VAR _Svg =
    "<svg xmlns='http://www.w3.org/2000/svg' width='" & _W & "' height='" & _H & "' viewBox='0 0 " & _W & " " & _H & "' font-family='Segoe UI, sans-serif'>"
        & "<rect width='" & _W & "' height='" & _H & "' fill='white'/>"
        & "<circle cx='" & _DotR & "' cy='" & _DotY & "' r='" & _DotR & "' fill='" & _Color & "'/>"
        & "<text x='" & 2 * _DotR + 6 & "' y='" & _NameY & "' font-size='" & _NamePx & "' font-weight='700' fill='rgb(31,36,51)'>" & _Ch & "</text>"
        & _Rows
        & "<line x1='0' y1='" & _LineY & "' x2='" & _W & "' y2='" & _LineY & "' stroke='rgb(225,231,240)' stroke-width='1'/>"
        & "<text x='0' y='" & _FootY & "' font-size='" & _FootLabelPx & "' fill='rgb(74,81,104)'>Link &#8594; Renewal</text>"
        & "<text x='" & _W & "' y='" & _FootY & "' text-anchor='end' font-size='" & _FootValuePx & "' font-weight='700' fill='" & _L2RColor & "'>" & FORMAT ( _L2R, "0.0%" ) & "</text>"
        & "</svg>"
RETURN
    "data:image/svg+xml;utf8," & SUBSTITUTE ( SUBSTITUTE ( _Svg, "%", "%25" ), "#", "%23" )
```

### Zone Tile SVG

Data category: `ImageUrl`

```dax
Zone Tile SVG =
VAR _Zone    = SELECTEDVALUE ( 'RAW DATA(JAN-SEPT)'[dealerzone], "ALL" )
VAR _Leads   = COUNTROWS ( 'RAW DATA(JAN-SEPT)' )
VAR _Att     = [Attempted]
VAR _Ren     = [Renewed]
VAR _Prem    = SUM ( 'RAW DATA(JAN-SEPT)'[premium_value] )
VAR _AttPct  = DIVIDE ( _Att, _Leads )
VAR _ConvPct = DIVIDE ( _Ren, _Att )
-- ===== IMAGE SIZE in px : must match matrix Image size =====
VAR _W   = 455          -- Image size › Width
VAR _H   = 165          -- Image size › Height
VAR _Gap = 0           -- space between tiles (same as your KPI cards)
-- ===== FONT SIZES in px =====
VAR _NamePx  = 24       -- WEST, SOUTH ...
VAR _LeadsPx = 25       -- 486,717 leads
VAR _LabelPx = 17      -- Attempt %, Renewed ...
VAR _ValuePx = 30       -- 33.7%, 4,557 ...
VAR _BarH    = 10       -- progress bar thickness
-- ===== layout : calculated =====
VAR _CW     = _W - _Gap
VAR _Pad    = 24
VAR _ColW   = ( _CW - 2 * _Pad ) / 4
VAR _NameY  = _Pad + ROUND ( _NamePx * 0.75, 0 )
VAR _BarY   = _H - _Pad - _BarH
VAR _ValueY = _BarY - 18
VAR _LabelY = _ValueY - _ValuePx - 4
-- ===== colours =====
VAR _Navy      = "rgb(26,35,126)"
VAR _Amber     = "rgb(163,90,0)"
VAR _Grey      = "rgb(91,98,117)"
VAR _AttColor  = IF ( _AttPct < 0.4, _Amber, _Navy )
VAR _ConvColor = IF ( _ConvPct < 0.01, _Amber, _Navy )
VAR _BarColor  = IF ( _AttPct < 0.4, "rgb(232,137,12)", "rgb(22,131,232)" )
VAR _BarW      = ROUND ( ( _CW - 2 * _Pad ) * _AttPct, 0 )
-- ===== the 4 numbers =====
VAR _Metrics =
    UNION (
        ROW ( "i", 0, "Lbl", "Attempt %", "Txt", FORMAT ( _AttPct, "0.0%" ),   "Col", _AttColor ),
        ROW ( "i", 1, "Lbl", "Renewed",   "Txt", FORMAT ( _Ren, "#,##0" ),     "Col", _Navy ),
        ROW ( "i", 2, "Lbl", "Conv %",    "Txt", FORMAT ( _ConvPct, "0.00%" ), "Col", _ConvColor ),
        ROW ( "i", 3, "Lbl", "Premium",   "Txt", "&#8377;" & FORMAT ( _Prem / 1000000, "0.0" ) & "M", "Col", _Navy )
    )
VAR _Cols =
    CONCATENATEX (
        _Metrics,
        VAR _x = ROUND ( _Pad + [i] * _ColW, 0 )
        RETURN
            "<text x='" & _x & "' y='" & _LabelY & "' font-size='" & _LabelPx & "' fill='" & _Grey & "'>" & [Lbl] & "</text>"
                & "<text x='" & _x & "' y='" & _ValueY & "' font-size='" & _ValuePx & "' font-weight='600' fill='" & [Col] & "'>" & [Txt] & "</text>",
        "",
        [i], ASC
    )
VAR _Svg =
    "<svg xmlns='http://www.w3.org/2000/svg' width='" & _W & "' height='" & _H & "' viewBox='0 0 " & _W & " " & _H & "' font-family='Segoe UI, sans-serif'>"
        & "<rect x='0' y='0' width='" & _CW & "' height='" & _H & "' rx='18' fill='white'/>"
        & "<text x='" & _Pad & "' y='" & _NameY & "' font-size='" & _NamePx & "' font-weight='700' fill='rgb(22,26,92)'>" & _Zone & "</text>"
        & "<text x='" & _CW - _Pad & "' y='" & _NameY & "' text-anchor='end' font-size='" & _LeadsPx & "' fill='rgb(74,81,104)'>" & FORMAT ( _Leads, "#,##0" ) & " leads</text>"
        & _Cols
        & "<rect x='" & _Pad & "' y='" & _BarY & "' width='" & _CW - 2 * _Pad & "' height='" & _BarH & "' rx='" & _BarH / 2 & "' fill='rgb(227,235,246)'/>"
        & "<rect x='" & _Pad & "' y='" & _BarY & "' width='" & _BarW & "' height='" & _BarH & "' rx='" & _BarH / 2 & "' fill='" & _BarColor & "'/>"
        & "</svg>"
RETURN
    "data:image/svg+xml;utf8," & SUBSTITUTE ( SUBSTITUTE ( _Svg, "%", "%25" ), "#", "%23" )
```

### Campaign Funnel SVG

Data category: `ImageUrl`

```dax
Campaign Funnel SVG =
VAR _Camp  = SELECTEDVALUE ( 'RAW DATA(JAN-SEPT)'[campignname], "All" )
VAR _Title = UPPER ( LEFT ( _Camp, 1 ) ) & MID ( _Camp, 2, 50 ) & " Campaign Funnel"
VAR _Up    = COUNTROWS ( 'RAW DATA(JAN-SEPT)' )
VAR _Att   = [Attempted]
VAR _Q     = [Quote_sent]
VAR _L     = [Link_sent_count]
VAR _R     = [Renewed]
VAR _Prem  = SUM ( 'RAW DATA(JAN-SEPT)'[premium_value] )
VAR _L2R   = DIVIDE ( _R, _L )
-- same scale for both cards: the larger campaign's Uploaded = full bar
VAR _MaxUp =
    MAXX (
        ALL ( 'RAW DATA(JAN-SEPT)'[campignname] ),
        CALCULATE ( COUNTROWS ( 'RAW DATA(JAN-SEPT)' ) )
    )
-- ===== IMAGE SIZE in px : Image size in BOTH tables must be exactly this =====
VAR _W = 696
VAR _H = 490
-- ===== FONT SIZES in px =====
VAR _TitlePx     = 29
VAR _LabelPx     = 27      -- Uploaded, Attempted ...
VAR _ValuePx     = 27      -- 557,190 ...
VAR _SubPx       = 22      -- 56.7% of uploaded ...
VAR _FootLabelPx = 27      -- "Premium"
VAR _FootValuePx = 33      -- ₹16.0M
-- ===== layout : calculated, no need to touch =====
VAR _Pad    = 28
VAR _LabelW = ROUND ( _LabelPx * 6.2, 0 )
VAR _ValW   = ROUND ( _SubPx * 10, 0 )
VAR _BarX   = _Pad + _LabelW
VAR _MaxBar = _W - _BarX - 20 - _ValW - _Pad
VAR _ValX   = _BarX + _MaxBar + 20
VAR _TitleY = _Pad + _TitlePx
VAR _Top    = _TitleY + 24
VAR _FootY  = _H - _Pad + 4
VAR _LineY  = _FootY - _FootValuePx - 16
VAR _RowH   = ROUND ( ( _LineY - 8 - _Top ) / 5, 0 )
VAR _BarH   = ROUND ( _LabelPx * 1.6, 0 )
-- ===== colours =====
VAR _Blue  = "rgb(22,131,232)"
VAR _Navy  = "rgb(26,35,126)"
VAR _Amber = "rgb(163,90,0)"
VAR _Grey  = "rgb(91,98,117)"
-- ===== stages =====
VAR _Stages =
    UNION (
        ROW ( "i", 0, "Stage", "Uploaded",   "Val", _Up,  "Sub", "",                                                   "SubCol", _Grey ),
        ROW ( "i", 1, "Stage", "Attempted",  "Val", _Att, "Sub", FORMAT ( DIVIDE ( _Att, _Up ), "0.0%" ) & " of uploaded", "SubCol", _Grey ),
        ROW ( "i", 2, "Stage", "Quote Sent", "Val", _Q,   "Sub", FORMAT ( DIVIDE ( _Q, _Att ), "0.0%" ) & " of attempted", "SubCol", _Grey ),
        ROW ( "i", 3, "Stage", "Link Sent",  "Val", _L,   "Sub", FORMAT ( DIVIDE ( _L, _Q ), "0.0%" ) & " of quotes",      "SubCol", _Grey ),
        ROW ( "i", 4, "Stage", "Renewed",    "Val", _R,   "Sub", FORMAT ( _L2R, "0.0%" ) & " of links",                   "SubCol", IF ( _L2R < 0.1, _Amber, _Grey ) )
    )
VAR _Rows =
    CONCATENATEX (
        _Stages,
        VAR _RowY   = _Top + [i] * _RowH
        VAR _ValY   = _RowY + _ValuePx
        VAR _SubY   = _ValY + _SubPx + 6
        VAR _BarY   = ROUND ( ( _RowY + _SubY ) / 2 - _BarH / 2 + 2, 0 )
        VAR _LabelY = _BarY + ROUND ( _BarH / 2 + _LabelPx * 0.35, 0 )
        VAR _bw     = IF ( [Val] > 0, ROUND ( MAX ( 3, DIVIDE ( [Val], _MaxUp ) * _MaxBar ), 0 ), 0 )
        VAR _Fill   = IF ( [i] = 4, _Navy, _Blue )
        VAR _SubW   = IF ( [i] = 4 && _L2R < 0.1, "700", "400" )
        RETURN
            "<text x='" & _Pad & "' y='" & _LabelY & "' font-size='" & _LabelPx & "' font-weight='600' fill='rgb(74,81,104)'>" & [Stage] & "</text>"
                & "<rect x='" & _BarX & "' y='" & _BarY & "' width='" & _bw & "' height='" & _BarH & "' rx='5' fill='" & _Fill & "'/>"
                & "<text x='" & _ValX & "' y='" & _ValY & "' font-size='" & _ValuePx & "' font-weight='700' fill='rgb(31,36,51)'>" & FORMAT ( [Val], "#,##0" ) & "</text>"
                & "<text x='" & _ValX & "' y='" & _SubY & "' font-size='" & _SubPx & "' font-weight='" & _SubW & "' fill='" & [SubCol] & "'>" & [Sub] & "</text>",
        "",
        [i], ASC
    )
VAR _Svg =
    "<svg xmlns='http://www.w3.org/2000/svg' width='" & _W & "' height='" & _H & "' viewBox='0 0 " & _W & " " & _H & "' font-family='Segoe UI, sans-serif'>"
        & "<rect x='0' y='0' width='" & _W & "' height='" & _H & "' rx='18' fill='white'/>"
        & "<text x='" & _W / 2 & "' y='" & _TitleY & "' text-anchor='middle' font-size='" & _TitlePx & "' font-weight='600' fill='rgb(43,48,64)'>" & _Title & "</text>"
        & _Rows
        & "<line x1='" & _Pad & "' y1='" & _LineY & "' x2='" & _W - _Pad & "' y2='" & _LineY & "' stroke='rgb(225,231,240)' stroke-width='1'/>"
        & "<text x='" & _Pad & "' y='" & _FootY & "' font-size='" & _FootLabelPx & "' font-weight='600' fill='rgb(74,81,104)'>Premium</text>"
        & "<text x='" & _W - _Pad & "' y='" & _FootY & "' text-anchor='end' font-size='" & _FootValuePx & "' font-weight='700' fill='" & _Navy & "'>&#8377;" & FORMAT ( _Prem / 1000000, "0.0" ) & "M</text>"
        & "</svg>"
RETURN
    "data:image/svg+xml;utf8," & SUBSTITUTE ( SUBSTITUTE ( _Svg, "%", "%25" ), "#", "%23" )
```

### TP Bars SVG

Data category: `ImageUrl`

```dax
TP Bars SVG =
VAR _NeedTP =
    CALCULATE ( COUNTROWS ( 'RAW DATA(JAN-SEPT)' ),
        'RAW DATA(JAN-SEPT)'[tertiary_disposition] = "Need TP Insurance Only" )
VAR _TPCov =
    CALCULATE ( COUNTROWS ( 'RAW DATA(JAN-SEPT)' ),
        'RAW DATA(JAN-SEPT)'[tertiary_disposition] = "TP Already Covered" )
VAR _SATP =
    CALCULATE ( COUNTROWS ( 'RAW DATA(JAN-SEPT)' ),
        'RAW DATA(JAN-SEPT)'[renewal_type] = "SATP" )
VAR _Max = MAX ( _NeedTP, MAX ( _TPCov, _SATP ) )
-- ===== size (Image size must match) =====
VAR _W      = 480
VAR _H      = 250
VAR _BarX   = 190
VAR _MaxBar = _W - _BarX - 70
-- ===== colours =====
VAR _Grey = "rgb(122,131,150)"
VAR _Teal = "rgb(20,153,138)"
VAR _Ink  = "rgb(43,48,64)"
-- ===== bars =====
VAR _Bars =
    UNION (
        ROW ( "i", 0, "Lbl", "Need TP Insurance Only", "Val", _NeedTP, "Col", _Grey, "Wt", "400" ),
        ROW ( "i", 1, "Lbl", "TP Already Covered",     "Val", _TPCov,  "Col", _Grey, "Wt", "400" ),
        ROW ( "i", 2, "Lbl", "SATP leads uploaded",    "Val", _SATP,   "Col", _Teal, "Wt", "700" )
    )
VAR _BarSvg =
    CONCATENATEX (
        _Bars,
        VAR _top = 8 + [i] * 42
        VAR _bw  = IF ( [Val] > 0, ROUND ( MAX ( 2, DIVIDE ( [Val], _Max ) * _MaxBar ), 0 ), 0 )
        RETURN
            "<text x='0' y='" & _top + 17 & "' font-size='15' font-weight='" & [Wt] & "' fill='" & _Ink & "'>" & [Lbl] & "</text>"
                & "<rect x='" & _BarX & "' y='" & _top + 2 & "' width='" & _bw & "' height='22' rx='4' fill='" & [Col] & "'/>"
                & "<text x='" & _BarX + _bw + 8 & "' y='" & _top + 18 & "' font-size='15' font-weight='" & [Wt] & "' fill='" & _Ink & "'>" & FORMAT ( [Val], "#,##0" ) & "</text>",
        "",
        [i], ASC
    )
-- ===== orange note (3 lines) =====
VAR _Callout =
    "<rect x='0' y='140' width='" & _W & "' height='96' rx='10' fill='rgb(255,244,229)'/>"
        & "<text x='14' y='168' font-size='15' fill='" & _Ink & "'><tspan font-weight='700'>" & FORMAT ( _NeedTP, "#,##0" ) & "</tspan> customers asked for third-party cover only,</text>"
        & "<text x='14' y='192' font-size='15' fill='" & _Ink & "'>but just <tspan font-weight='700'>" & FORMAT ( _SATP, "#,##0" ) & "</tspan> SATP leads were uploaded &#8212;</text>"
        & "<text x='14' y='216' font-size='15' fill='" & _Ink & "'>a ready SATP cross-sell list.</text>"
VAR _Svg =
    "<svg xmlns='http://www.w3.org/2000/svg' width='" & _W & "' height='" & _H & "' viewBox='0 0 " & _W & " " & _H & "' font-family='Segoe UI, sans-serif'>"
        & "<rect width='" & _W & "' height='" & _H & "' fill='white'/>"
        & _BarSvg & _Callout & "</svg>"
RETURN
    "data:image/svg+xml;utf8," & SUBSTITUTE ( SUBSTITUTE ( _Svg, "%", "%25" ), "#", "%23" )
```

### TP Reasons SVG

```dax
TP Reasons SVG =
VAR _W     = 480
VAR _H     = 250
VAR _ColW  = 240
VAR _Ink   = "rgb(43,48,64)"
VAR _Muted = "rgb(74,81,104)"
VAR _Navy  = "rgb(26,35,126)"
VAR _Types = { ( "SAOD", 0 ), ( "PACKAGE", 1 ) }
VAR _Blocks =
    CONCATENATEX (
        _Types,
        VAR _t = [Value1]
        VAR _x = [Value2] * _ColW
        VAR _R =
            ADDCOLUMNS (
                FILTER (
                    CALCULATETABLE (
                        VALUES ( 'RAW DATA(JAN-SEPT)'[tertiary_disposition] ),
                        'RAW DATA(JAN-SEPT)'[renewal_type] = _t
                    ),
                    'RAW DATA(JAN-SEPT)'[tertiary_disposition] <> ""
                        && NOT ( LEFT ( 'RAW DATA(JAN-SEPT)'[tertiary_disposition], 1 )
                                 IN { "0", "1", "2", "3", "4", "5", "6", "7", "8", "9" } )
                ),
                "n", CALCULATE ( COUNTROWS ( 'RAW DATA(JAN-SEPT)' ), 'RAW DATA(JAN-SEPT)'[renewal_type] = _t )
            )
        VAR _Items =
            CONCATENATEX (
                TOPN ( 3, _R, [n], DESC ),
                VAR _cur = [n]
                VAR _rk  = COUNTROWS ( FILTER ( _R, [n] > _cur ) ) + 1
                VAR _y   = 100 + ( _rk - 1 ) * 38
                RETURN
                    "<text x='" & _x & "' y='" & _y & "' font-size='14' fill='" & _Ink & "'>" & _rk & ". "
                        & 'RAW DATA(JAN-SEPT)'[tertiary_disposition]
                        & " <tspan font-weight='700'>" & FORMAT ( _cur, "#,##0" ) & "</tspan></text>",
                ""
            )
        RETURN
            "<text x='" & _x & "' y='64' font-size='15' font-weight='700' fill='" & _Navy & "'>" & _t & "</text>" & _Items,
        "",
        [Value2], ASC
    )
VAR _Svg =
    "<svg xmlns='http://www.w3.org/2000/svg' width='" & _W & "' height='" & _H & "' viewBox='0 0 " & _W & " " & _H & "' font-family='Segoe UI, sans-serif'>"
        & "<rect width='" & _W & "' height='" & _H & "' fill='white'/>"
        & "<text x='0' y='22' font-size='15' font-weight='700' fill='" & _Muted & "'>Top drop reasons by policy type</text>"
        & "<line x1='0' y1='34' x2='" & _W & "' y2='34' stroke='rgb(199,210,227)' stroke-width='1'/>"
        & _Blocks & "</svg>"
RETURN
    "data:image/svg+xml;utf8," & SUBSTITUTE ( SUBSTITUTE ( _Svg, "%", "%25" ), "#", "%23" )
```

## Calculated columns

### Outcome Group

```dax
Outcome Group =
VAR _d = 'RAW DATA(JAN-SEPT)'[lastcalldisposition]
RETURN
    SWITCH (
        TRUE (),
        _d IN { "Confirmed", "Book Appointment" }, "Positive",
        _d = "Call Me Later", "Follow-up",
        _d IN { "Ringing No response", "Busy", "Switched Off / Unreachable", "Invalid Number", "Wrong Number" }, "Not reachable",
        _d = "Renewal Not Required", "Not required",
        _d IN { "Cancelled", "NoOther" }, "Other",
        BLANK ()
    )
```

### Outcome Order

```dax
Outcome Order =
VAR _d = 'RAW DATA(JAN-SEPT)'[lastcalldisposition]
RETURN
    SWITCH (
        TRUE (),
        _d IN { "Confirmed", "Book Appointment" }, 1,
        _d = "Call Me Later", 2,
        _d IN { "Ringing No response", "Busy", "Switched Off / Unreachable", "Invalid Number", "Wrong Number" }, 3,
        _d = "Renewal Not Required", 4,
        _d IN { "Cancelled", "NoOther" }, 5,
        BLANK ()
    )
```

Set **Outcome Group → Sort by column → Outcome Order** so the outcome legend keeps its order.

## Power Query

Change the file path in `Source` to where your CSV is saved.

### RAW DATA(JAN-SEPT)

```powerquery
let
    Source = Csv.Document(File.Contents("D:\POWER BI\RAW DATA(JAN-SEPT).csv"),[Delimiter=",", Columns=38, Encoding=1252, QuoteStyle=QuoteStyle.None]),
    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{{"SourceFile", type text}, {"DueMonth", type text}, {"assigned_id", Int64.Type}, {"assigneddate", type date}, {"vehicle_id", Int64.Type}, {"policyduedate", type date}, {"cp_code", Int64.Type}, {"is_dropped", type text}, {"attempt_count", Int64.Type}, {"firstcalldate", type date}, {"lastcalldate", type date}, {"firstcalldisposition", type text}, {"lastcalldisposition", type text}, {"calldispositiondata_id", Int64.Type}, {"username", type text}, {"dealercode", Int64.Type}, {"DisplayName", type text}, {"dealerstate", type text}, {"dealerzone", type text}, {"dealercity", type text}, {"callstatus", type text}, {"is_renewed", type text}, {"Link_sent", type text}, {"autopulse_renewal", type text}, {"quote_generated", type text}, {"whatsapp_renewal", type text}, {"campignname", type text}, {"renewal_type", type text}, {"followupdate", type date}, {"tertiary_disposition", type text}, {"channel", type text}, {"premium_value", Int64.Type}, {"whatsapp_premium", type text}, {"ZoneManager", type text}, {"StateManager", type text}, {"AreaManager", type text}, {"ChannelManager", type text}, {"ChannelHead", type text}}),
    #"Removed Columns" = Table.RemoveColumns(#"Changed Type",{"SourceFile"}),
    #"Changed Type1" = Table.TransformColumnTypes(#"Removed Columns",{{"assigneddate", type date}}),
    #"Added Custom" = Table.AddColumn(#"Changed Type1", "Month No", each List.PositionOf({"January","February","March","April","May","June","July","August","September","October","November","December"}, [DueMonth]) + 1),
    #"Changed Type2" = Table.TransformColumnTypes(#"Added Custom",{{"Month No", Int64.Type}}),
    Custom1 = Table.ReplaceValue(
    #"Changed Type2",
    each [autopulse_renewal],
    each if [premium_value] <> null and Text.Upper([autopulse_renewal]) = "NO"
         then "YES"
         else [autopulse_renewal],
    Replacer.ReplaceValue,
    {"autopulse_renewal"}
),
    #"Added Custom1" = Table.AddColumn(Custom1, "Custom", each if [lastcalldisposition] = null or [lastcalldisposition] = "" then null
else if List.Contains({"Confirmed", "Book Appointment"}, [lastcalldisposition]) then "Positive"
else if [lastcalldisposition] = "Call Me Later" then "Follow-up"
else if [lastcalldisposition] = "Renewal Not Required" then "Not required"
else if List.Contains({"Ringing No response", "Busy", "Switched Off / Unreachable", "Invalid Number", "Wrong Number"}, [lastcalldisposition]) then "Not reachable"
else "Other"),
    #"Renamed Columns" = Table.RenameColumns(#"Added Custom1",{{"Custom", "Outcome Group"}}),
    #"Added Custom2" = Table.AddColumn(#"Renamed Columns", "Outcome Order", each if [Outcome Group] = "Positive" then 1
else if [Outcome Group] = "Follow-up" then 2
else if [Outcome Group] = "Not reachable" then 3
else if [Outcome Group] = "Not required" then 4
else if [Outcome Group] = "Other" then 5
else null),
    #"Changed Type3" = Table.TransformColumnTypes(#"Added Custom2",{{"Outcome Order", Int64.Type}}),
    #"Removed Columns1" = Table.RemoveColumns(#"Changed Type3",{"Outcome Group", "Outcome Order"})
in
    #"Removed Columns1"
```
