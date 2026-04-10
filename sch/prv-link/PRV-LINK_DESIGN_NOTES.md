# PRV-LINK v2.0 OBD2 BLE Adapter - Design Notes

## Project Overview
Pojednostavljena verzija WiCAN OBD Pro v1.51 za Provirium platformu.
BLE-only OBD2 dongle u malom kucishtu (~60x40x20mm).
**ESP32-S3 TWAI direct - bez STN1110 OBD interpreter chipa.**
Software ELM327 emulacija na ESP32-S3 (vec implementirana u WiCAN firmware).

## Baziran na: WiCAN OBD Pro v1.51 (MeatPi Electronics)

---

## 1. ARHITEKTURA

```
[OBD2 Port 12V] --> [D2 Schottky] --> [D1 TVS] --> [MP2359 Buck 5V] --> [AMS1117 LDO 3.3V]
                                                                              |
[OBD2 CAN_H/L] --> [PESD2CAN ESD] --> [TJA1043T] <--TWAI--> [ESP32-S3-MINI-1]
                                                                    |
                                                              [BLE 5.0] --> Telefon
                                                              [GPIO42] --> [LED]
```

### Razlika od WiCAN Pro:
| WiCAN Pro | PRV-LINK v2.0 |
|-----------|---------------|
| ESP32 -> UART -> STN1110 -> CAN transceiver | ESP32 TWAI -> CAN transceiver (direktno) |
| Hardverski ELM327 (STN1110) | Softverski ELM327 (ESP32 firmware) |
| 5 IC-ova | **3 IC-a** (U1 buck, U2 LDO, U3 ESP32, U4 CAN xcvr) |
| 30+ komponenti | **24 komponente** |

---

## 2. UKLONJENE KOMPONENTE (od WiCAN Pro)

### IC-ovi uklonjeni
| Komponenta | Razlog |
|-----------|--------|
| **STN1110/STN2120** OBD interpreter | Zamijenjen softverskom emulacijom na ESP32 |
| **AW2023** RGB LED driver | Zamijenjen jednostavnom LED na GPIO42 |
| **RX8130** RTC | Vrijeme preko BLE sinkronizacija |
| **ICM-42670-P** IMU | Nepotreban za OBD citac |
| **WU3B3801** USB-C controller | Nema USB-a |
| **CH342F** USB-Serial | Nema USB-a |
| **FRG88102DLEX** USB switch | Nema USB-a |
| USB Power Output Switch | Nema USB-a |

### Sekcije uklonjene
- SD Card slot + SDMMC interface (GPIO 12,13,14,21,40,47,48)
- USB-C konektor + ESD zastita
- USB-Serial kompletna sekcija
- Battery voltage monitoring (nema baterije)
- I2C bus kompletno (nema periferinih uredaja)
- Crystal 4MHz + load caps (bio za STN1110)

### Ukupno uklonjeno: ~45 komponenti

---

## 3. ZADRZANE KOMPONENTE (PRV-LINK v2.0)

### Aktivne komponente (4 IC/modula)
| Ref | Komponenta | Funkcija | LCSC# |
|-----|-----------|----------|-------|
| U1 | MP2359DJ-LF-Z | Buck 12V->5V | C14259 |
| U2 | AMS1117-3.3 | LDO 5V->3.3V | C6186 |
| U3 | ESP32-S3-MINI-1-N8 | MCU + BLE + TWAI CAN | C2913204 |
| U4 | TJA1043T,118 | CAN transceiver | C95335 |

### Konektor
| Ref | Komponenta | JLCPCB# |
|-----|-----------|---------|
| J1 | OBD-II 16P Male THT | C9900123312 |

### Zastita (4 diode)
| Ref | Komponenta | Funkcija | LCSC# |
|-----|-----------|----------|-------|
| D1 | SMBJ16A | TVS input protection | C113632 |
| D2 | SS34 | Reverse polarity | C8678 |
| D3 | SS14 | Buck freewheeling | C2480 |
| D4 | PESD2CAN | CAN ESD protection | C101402 |

### Indikacija
| Ref | Komponenta | LCSC# |
|-----|-----------|-------|
| D5 | Green LED 0603 | C72043 |
| R6 | 1K (LED limiter) | C11702 |

### Pasivne (14 kom)
C1-C8 (8 kondenzatora), R1-R5 (5 otpornika), L1 (1 induktor)

**UKUPNO: 24 komponente** (vs ~70 na WiCAN Pro)

---

## 4. GPIO MAPPING (PRV-LINK v2.0)

### Koristeni GPIO
| GPIO | Funkcija | Smjer |
|------|---------|-------|
| 4 | TWAI_TX (CAN transmit) | Output |
| 5 | TWAI_RX (CAN receive) | Input |
| 38 | CAN_STDBY (transceiver standby) | Output |
| 42 | LED_EN (status LED) | Output |
| EN | Chip Enable (RC reset) | - |

### Slobodni GPIO (18 pinova za buduce koristenje)
1, 2, 3, 6, 7, 8, 9, 10, 11, 12, 13, 14, 21, 39, 40, 41, 47, 48

Potencijalna korist slobodnih GPIO:
- GPIO 1/2: UART debug/programiranje
- GPIO 8: Factory reset button
- GPIO 6/7: I2C za buduce senzore
- GPIO 0: BOOT pin (firmware flash mode)

---

## 5. FIRMWARE KONFIGURACIJA

### Compile-time promjene
WiCAN firmware treba kompajlirati za **non-PRO** hardware verziju:
```c
// U sdkconfig ili menuconfig:
// NE koristiti WICAN_PRO - to ocekuje STN1110 na UART
// Koristiti WICAN_V300 ili custom HARDWARE_VER

#define HARDWARE_VER WICAN_V300  // ili novi PRV_LINK define
```

### Sto firmware vec radi (non-PRO mode):
- Softverski ELM327 AT command parser
- TWAI CAN kontroler inicijalizacija
- ISO 15765-4 ISO-TP multi-frame podrska
- Flow control (ATFCSH, ATFCSD, ATFCSM)
- Custom headers (ATSH) za UDS pristup
- BLE server za komunikaciju s telefonom
- WiFi AP/STA za web konfiguraciju

### Sto treba prilagoditi:
1. GPIO pin mapping (TWAI_TX=4, TWAI_RX=5, CAN_STDBY=38, LED=42)
2. Onemoguciti SD card, RTC, IMU, USB inicijalizaciju
3. LED kontrola: direktni GPIO umjesto AW2023 I2C
4. Dodati novi HARDWARE_VER za PRV-LINK

---

## 6. UPOZORENJA

1. **Plasticno kuciste obavezno** - metalno bi blokiralo BLE/WiFi antenu
   ESP32-S3-MINI-1. Alternativa: ESP32-S3-MINI-1U (U.FL) s eksternom antenom.

2. **TWAI GPIO izbor** - GPIO 4/5 za TWAI su sigurni za boot (nema
   strapping funkcije). Izbjegavati GPIO 0,45,46 za CAN jer utjecu na boot mode.

3. **CAN transceiver napajanje** - TJA1043T radi na 5V (VCC), ali TXD/RXD
   su 3.3V tolerantni. Nema potrebe za level shifter.

4. **OBD2 konektor velicina** - THT konektor (~30x20mm) dominira form faktorom.
   PCB mora biti dizajniran oko konektora.

5. **Firmware HARDWARE_VER** - Obavezno definirati novi hardware version
   za PRV-LINK da se izbjegne inicijalizacija nepostojecih periferinih uredaja.

---

## 7. PROCJENA TROSKOVA

### Komponente (po komadu, 200 kom serija)
| Kategorija | Cijena/kom |
|-----------|-----------|
| ESP32-S3-MINI-1-N8 | ~$2.80 |
| TJA1043T CAN transceiver | ~$0.45 |
| MP2359 + AMS1117 (napajanje) | ~$0.40 |
| Diode (D1-D4) | ~$0.35 |
| Pasivne (C,R,L,LED) | ~$0.30 |
| OBD2 konektor | ~$1.50 |
| **Komponente ukupno** | **~$5.80** |

### JLCPCB PCBA (200 komada)
| Stavka | Ukupno |
|--------|--------|
| PCB (2L, ~40x25mm) | ~$80 |
| SMT Assembly | ~$250 |
| THT Assembly (OBD konektor) | ~$100 |
| Komponente | ~$1,160 |
| Setup + Dostava | ~$130 |
| **UKUPNO** | **~$1,720** |
| **Po komadu** | **~$8.60** |

Usteda vs verzija sa STN1110: **~$6.90/kom** (bio ~$15.50)
