---
{"dg-publish":true,"permalink":"/notizen/tauchen/berechnungen/"}
---


#tauchen #physik

## Boyle Mayot bei gleicher Temperatur
```graph 
bounds: [-8,4,8,-4]
axis: false

elements: [
  #setup_circle_axis_lines,
  {type: slider, def: [[5,1],[5,3],[1,1,10]], att: {name: "Druck P bar"}},
  {type: text, def: [-7.5, 3, "Boyle Mayot<br> bei gleicher Temperatur"], att: {name: "Label"}},
    {type: text, def: [-7.5, -1, "Volumen V l"], att: {name: "Vol"}},
    {type: text, def: [-6, -1, (2/f:e0).toPrecision(1)], att: {label: "Volumen"}},
  {type: circle, def: [[0,0],"f:2/e0"], att: {strokeColor: "grey", fixed: true}}
  ]
```

# Amotons / Gay-Lussac

```graph 
bounds: [-8,4,8,-4]
axis: false

elements: [
  #setup_circle_axis_lines,
  {type: slider, def: [[5,1],[5,3],[10,45,45]], att: {name: "Temperatur T °C<br> Anfang", label: "Temp"}},
  {type: slider, def: [[5,-2],[5,0],[10,10,45]], att: {name: "Temperatur T °C<br>Ende", label: "Temp"}},
  {type: slider, def: [[2.5,-2],[2.5,3],[190,210,230]], att: {name: "Druck P bar<br>Anfang"}},
  {type: text, def: [-7.5, 3, "Temperaturänderung bei gleichen Volumen"], att: {name: "Label"}},
    {type: text, def: [-7.5, -1, "Enddruck P bar"], att: {name: "Vol"}},
    {type: text, def: [-5.5, -1, (f:(e2/(e0+273)*(e1+273))).toPrecision(3)]},
  {type: circle, def: [[0,0],2], att: {strokeColor: "grey", fixed: true, label: "Volumen gleich"}}
  ]
```
