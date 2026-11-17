# KRL simulator – KUKA robot za risanje

Simulator v brskalniku za učenje programiranja robota KUKA v jeziku **KRL**.
Program napišeš enako kot na pravem robotu, simulator pa v 3D pokaže, kaj bi robot naredil: kako se premika, kaj nariše in kje bi prišlo do napake (trk, zlom svinčnika, točka izven dosega …).

Namenjen je šolski uporabi – vadbi pred delom na pravem robotu.

---

## Zagon

1. Prenesi datoteko `krl-simulator.html`.
2. Odpri jo z dvoklikom (Chrome ali Edge).
3. Ob prvem zagonu je potrebna internetna povezava (3D knjižnica three.js se naloži s spleta).

Namestitev ni potrebna.

---

## Zaslon

| Del | Opis |
|---|---|
| **Levo – urejevalnik** | Pišeš KRL kodo. Napake se pokažejo že med tipkanjem (rdeča vrstica). |
| **Sredina – 3D pogled** | Robot, miza, deska s papirjem in stojalce za kemik. |
| **Levo zgoraj – stanje** | Koti osi A1–A6, lega TCP v `$BASE`, sila na pisalo, stanje prijemala. |
| **Desno zgoraj** | Izbira orodja, kamera in nastavitev deske (base). |
| **Desno spodaj – papir** | Pogled na papir od zgoraj z mrežo v mm. |
| **Spodaj levo – dnevnik** | Sporočila programa. Klik na sporočilo skoči na vrstico. |

### Gumbi in tipke

| Gumb / tipka | Dejanje |
|---|---|
| **▶ Zaženi** / `Ctrl + Enter` / **preslednica** | Zažene program |
| **⏭ Korak** | Izvede en gib |
| **⏸ Pavza** / **preslednica** | Ustavi |
| **⏹ Ponastavi** | Robot na začetek, papir počiščen |
| **Puščice** | Premik kamere (Shift + ↑↓ = gor/dol) |
| **🔒 Ptičja perspektiva** | Pogled od zgoraj (kolesce = zoom) |
| **⛶ Koda / ⛶ Robot** | Okno čez cel zaslon (`Esc` = nazaj) |
| **Primeri…** | Naloži primer programa |
| **Odpri… / Shrani .src** | Odpre ali shrani program |

Velikost oken spremeniš z vlečenjem meje med kodo in robotom ter kota okna s papirjem.

---

## Podprti ukazi KRL

```krl
PTP HOME
PTP {x 30, y 50, z 10}          ; manjkajoče vrednosti ostanejo iz prejšnje točke
LIN {x 0, y 50}
CIRC {x 27.5, y 37.5} {x 15, y 25}
CIRC {…} {…} CA 360             ; cel krog

$BASE = {x 506.75, y -43.91, z 44.64, a 58.84, b 0.74, c 1.69}
$BASE = BASE_DATA[2] : {a -90}  ; zasuk v ravnini mize
$TOOL = TOOL_DATA[2]            ; kemik v prijemalu

DECL INT i
DECL REAL s = 20
DECL E6POS TPO = {x 306.3, y -400.1, z 39.9, a 105, b -1.7, c 179.5}

FOR i = 1 TO 5 … ENDFOR
IF i == 2 THEN … ELSE … ENDIF
WHILE … ENDWHILE

DEF kvadrat(x0:IN, s:IN)        ; podprogram s parametri
WAIT SEC 1
$OUT[17] = TRUE
WAIT FOR $IN[17]
```

Podprte so tudi funkcije `SIN`, `COS`, `TAN`, `SQRT`, `ABS`, `ATAN2` (v stopinjah) in operatorji `==`, `<>`, `<`, `>`, `AND`, `OR`, `NOT`.

---

## Orodja

| Orodje | Opis |
|---|---|
| `TOOL_DATA[1]` – **svinčnik** | V vzmetnem držalu (hod 5 mm). Večja sila nariše temnejšo črto. |
| `TOOL_DATA[2]` – **kemik** | V pnevmatskem prijemalu. Ob začetku je vedno v stojalcu – program ga mora **najprej pobrati**. |

### Prijemalo

| Signal | Pomen |
|---|---|
| `$OUT[18] = TRUE` | odpri |
| `$OUT[17] = TRUE` | zapri |
| `$IN[18]` | prijemalo je odprto |
| `$IN[17]` | prijemalo je zaprto |

Primer pobiranja kemika:

```krl
PTP TNO                 ; nad stojalcem
$OUT[17]=FALSE
$OUT[18]=TRUE           ; odpri
WAIT FOR $IN[18]
LIN TPO                 ; navpično dol do kemika
$OUT[18]=FALSE
$OUT[17]=TRUE           ; zapri = primi
WAIT FOR $IN[17]
LIN TNO                 ; gor
```

---

## Base (deska)

Deska s papirjem je **base robota** (`BASE_DATA[1]` = `BASE_DATA[2]` = »meho miza«).
Lego in naklon deske spremeniš z drsniki desno zgoraj in potrdiš z **Set base**.
Program brez `$BASE` riše neposredno na desko.

---

## Sporočila

| Znak | Pomen |
|---|---|
| `✔` | program se je izvedel |
| `⚠` | program se izvede, a se nekaj zgodi |
| `✖` | program se ustavi |
| `ℹ` | opomba |

Primeri:

```
⚠ vrstica 3: svinčnik se zlomi
⚠ vrstica 4: kemik se zlomi
⚠ vrstica 31: kemik pade
✖ vrstica 1: izven dosega
✖ vrstica 12: trk: kemik zadene stojalo
✖ vrstica 1: $BASE1 ne obstaja
ℹ vrstica 4: kemik ni v prijemalu – najprej ga poberi
```

---

## Primeri

V meniju **Primeri…**:

- **Črka R 2×** – podprogram in premik base
- **Lik 1 SK** – `LIN` in `CIRC`
- **FOR + IF: vrsta kvadratov** – zanke, pogoji, parametri
- **Naloga 3** – kemik s stojalca, črka R v dveh bazah, kemik nazaj
- **Prazen program**

---

## Omejitve

Simulator je učni pripomoček in **ne nadomešča preizkusa na pravem robotu**.

- Mere robota so ocenjene s fotografije (starejši mali KUKA).
- Sile, vzmet in meje zloma so ocene; spremeniš jih v **Nastavitvah**.
- Zaobljanje vogalov (`C_DIS`, `C_PTP`) ni simulirano – robot gre točno v točko.
- `LOOP`, `SWITCH` in `EXIT` niso podprti.
- Koda in nastavitve se shranijo v brskalnik; program trajno shraniš z **Shrani .src**.

---

## Datoteke

| Datoteka | Vsebina |
|---|---|
| `krl-simulator.html` | celoten simulator |
| `reports/` | poročilo o delovanju KUKA in sili na svinčnik |
| `research_notes/` | zapiski raziskave |
