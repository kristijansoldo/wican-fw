# PRV-LINK v2.0 - JLCPCB Turnkey PCBA (200 komada)

## Pregled dizajna

- **3 IC-a** + 1 konektor + 20 pasivnih = **24 komponente ukupno**
- **Svi SMT dijelovi na LCSC-u** (Basic ili Extended) - nema Global Sourcing
- **1 THT dio** (OBD2 konektor) - JLCPCB katalog C9900123312
- **Procjena: ~$8.60/kom za 200 komada (~$1,720 ukupno)**

---

## Sto trebas pripremiti

### 1. KiCad dizajn fajlovi
Potrebno jos napraviti PCB layout na temelju `prv-link.kicad_sch`:

- [ ] PCB layout u KiCad-u (routing, placement)
- [ ] **Gerber fajlovi** (.gbr + .drl) - File > Plot
- [ ] **BOM** u JLCPCB formatu (.csv)
- [ ] **CPL** (Component Placement List) (.pos fajl)

### 2. BOM format za JLCPCB
Pretvori `PRV-LINK_BOM.csv` u JLCPCB format:

```csv
Comment,Designator,Footprint,LCSC Part Number
MP2359DJ-LF-Z,U1,SOT-23-6,C14259
AMS1117-3.3,U2,SOT-223,C6186
ESP32-S3-MINI-1-N8,U3,ESP32-S3-MINI-1,C2913204
TJA1043T/118,U4,SOIC-8,C95335
OBD-II-16P-Male,J1,TH-16P,C9900123312
SMBJ16A,D1,SMB,C113632
SS34,D2,SMA,C8678
SS14,D3,SMA,C2480
PESD2CAN,D4,SOT-23,C101402
Green LED,D5,0603,C72043
10uH/1.5A,L1,3030,C339747
22uF/25V,C1,0805,C45783
22uF/10V,C2,0805,C45783
10uF,C3,0805,C15850
22uF,C4,0805,C45783
100nF,C5,0402,C1525
10uF,C6,0805,C15850
100nF,C7,0402,C1525
100nF,C8,0402,C1525
49.9K,R1,0402,C25129
15K,R2,0402,C25808
10K,R3,0402,C25744
0R,R4,0402,C17168
0R,R5,0402,C17168
1K,R6,0402,C11702
```

### 3. CPL format za JLCPCB
```csv
Designator,Mid X,Mid Y,Layer,Rotation
U1,xx.xx,yy.yy,T,0
U2,xx.xx,yy.yy,T,0
...
```
(Generira se iz KiCad-a: File > Fabrication Outputs > Component Placement)

---

## Dostupnost dijelova

### LCSC Basic (na lageru, najjeftinija montaza) - 22 od 24 dijelova
| Ref | Dio | LCSC# | Status |
|-----|-----|-------|--------|
| U1 | MP2359DJ-LF-Z | C14259 | Basic |
| U2 | AMS1117-3.3 | C6186 | Basic |
| U4 | TJA1043T,118 | C95335 | Basic |
| D1 | SMBJ16A | C113632 | Basic |
| D2 | SS34 | C8678 | Basic |
| D3 | SS14 | C2480 | Basic |
| D4 | PESD2CAN | C101402 | Basic |
| D5 | Green LED | C72043 | Basic |
| L1 | 10uH | C339747 | Basic |
| C1-C8 | Caps | razni | Basic |
| R1-R6 | Resistors | razni | Basic |

### LCSC Extended - 1 dio
| Ref | Dio | LCSC# | Napomena |
|-----|-----|-------|---------|
| U3 | ESP32-S3-MINI-1-N8 | C2913204 | Extended - visa cijena montaze |

### JLCPCB Assembly katalog - 1 dio (THT)
| Ref | Dio | JLCPCB# | Napomena |
|-----|-----|---------|---------|
| J1 | OBD2 16P Male | C9900123312 | Wave soldering, THT |

**Nema Global Sourcing / Consigned dijelova!** Sve je dostupno kroz JLCPCB/LCSC.

---

## PCB specifikacije

| Parametar | Vrijednost |
|-----------|-----------|
| Layers | 2 |
| Dimenzije | ~40 x 25 mm (cilj) |
| Debljina | 1.6 mm |
| Materijal | FR-4 TG155 |
| Solder mask | Crna (PRV-LINK branding) |
| Surface finish | HASL lead-free |
| Min trace/space | 6/6 mil (0.15mm) |
| Min via | 0.3mm drill |
| Copper weight | 1 oz |

Napomena: HASL je dovoljan - nema fine-pitch QFN-a (STN1110 uklonjen).
ESP32-S3-MINI-1 je castellated modul, HASL je OK.

---

## Koraci za narudzbu

### Korak 1: PCB Layout
1. Otvori `prv-link.kicad_sch` u KiCad 7
2. Assign footprints (vecina vec dodijeljena)
3. Prebaci u PCB editor (Update PCB from Schematic)
4. Placement prioritet:
   - J1 (OBD2 konektor) na rubu PCB-a - definira form factor
   - U3 (ESP32-S3) na suprotnom kraju - antena mora biti na rubu PCB-a!
   - U4 (CAN transceiver) izmedju ESP32 i konektora
   - U1, U2 (napajanje) blizu OBD konektora (kratki power traces)
5. Routing: 2-layer, GND plane na Bottom, signals na Top
6. DRC check - 0 errors

### Korak 2: Generiranje proizvodnih fajlova
```
KiCad > File > Plot:
  - Layers: F.Cu, B.Cu, F.Mask, B.Mask, F.Silkscreen, B.Silkscreen, Edge.Cuts
  - Format: Gerber
  - Drill: Excellon format

KiCad > File > Fabrication Outputs > Component Placement (.pos)
```

### Korak 3: Upload na JLCPCB
1. jlcpcb.com > Order Now > Upload Gerber ZIP
2. PCB parametri: 2L, 1.6mm, crni mask, HASL
3. Kolicina: **200 komada** (ili 250 za safety margin)
4. Ukljuci "SMT Assembly" > Top side
5. Upload BOM (.csv) i CPL (.pos)
6. Assembly: **Economic** (svi Basic parts osim ESP32)

### Korak 4: BOM Matching
JLCPCB prikazuje matching:
- **22 Basic** = zeleno, automatski matched
- **1 Extended** (ESP32-S3) = zuto, moze biti delay
- **1 JLCPCB Assembly** (OBD2) = C9900123312, THT wave soldering
- Sve bi trebalo biti zeleno/zuto - **nista crveno**

### Korak 5: Review i narudzba
- Provjeri 3D preview
- Provjeri orijentaciju svih komponenti
- Posebno: LED polaritet, diode polaritet, ESP32 orijentacija
- Potvrdi i plati

---

## Procjena troskova (200 komada)

| Stavka | Cijena/kom | Ukupno |
|--------|-----------|--------|
| PCB (2L, 40x25mm, crni) | ~$0.40 | ~$80 |
| SMT Assembly (23 SMT parts) | ~$1.25 | ~$250 |
| THT Assembly (1 konektor) | ~$0.50 | ~$100 |
| ESP32-S3-MINI-1-N8 (Extended) | ~$2.80 | ~$560 |
| TJA1043T + diode + pasivne | ~$1.50 | ~$300 |
| OBD2 konektor (JLCPCB) | ~$1.50 | ~$300 |
| Setup fee | - | ~$50 |
| Dostava (DHL Express) | - | ~$80 |
| **UKUPNO** | **~$8.60** | **~$1,720** |

---

## Rokovi

| Faza | Trajanje |
|------|----------|
| PCB fabricacija | 3-5 dana |
| SMT + THT Assembly | 3-7 dana |
| Quality check | 1 dan |
| Dostava DHL | 3-5 dana |
| **UKUPNO** | **~10-18 dana** |

Nema vise cekanja na Global Sourcing za STN1110!

---

## Checklist

- [ ] KiCad schematic provjeren (ERC = 0)
- [ ] PCB layout zavrsen (DRC = 0)
- [ ] Gerber fajlovi generirani
- [ ] BOM u JLCPCB formatu
- [ ] CPL generiran iz KiCad-a
- [ ] Gerber pregledan u JLCPCB viewer-u
- [ ] ESP32-S3 antena na rubu PCB-a (bez GND plane ispod antene!)
- [ ] Svi LCSC# potvrdeni i na lageru
- [ ] LED/diode polaritet provjeren
- [ ] Power trace sirine minimum 0.5mm (12V input min 0.8mm)
- [ ] Firmware kompajliran za non-PRO TWAI mode
- [ ] Test plan pripremljen (CAN loopback test, BLE pairing, OBD PID citanje)
- [ ] Kuciste odabrano i dimenzije PCB-a potvrdene

---

## Post-production: Firmware flashing

ESP32-S3-MINI-1 podrzava USB-JTAG za programiranje:
1. Spoji USB na GPIO 19 (D-) i GPIO 20 (D+) - ili koristi UART0
2. Flash WiCAN firmware (non-PRO build) sa `esptool.py`
3. Za seriju od 200: napravi jig s pogo pinovima na UART pads

Alternativa: Pre-flash ESP32-S3 module prije PCBA assembly (naruciti module s firmware-om od Espressif).
