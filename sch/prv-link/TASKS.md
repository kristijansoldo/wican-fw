# PRV-LINK - Task Lista

## Faza 1: Hardver test kit
- [x] Naruciti ESP32-C3 Super Mini dev board
- [x] Naruciti SN65HVD230 CAN transceiver modul
- [x] Naruciti OBD2 16pin male open end kabel
- [x] Naruciti Dupont jumper wire kit (M-M, M-F, F-F)
- [x] Nabaviti breadboard
- [x] Nabaviti USB-C kabel

## Faza 2: Priprema dok cekamo dostavu
- [ ] Instalirati esptool (`pip install esptool`)
- [ ] Instalirati Car Scanner app na mobitel
- [ ] Locirati OBD2 port na Golfu 6
- [ ] Buildati firmware lokalno (docker)

## Faza 3: Spajanje i flash
- [ ] Spojiti ESP32-C3 + CAN modul + LED na breadboardu
- [ ] Flashati PRV-LINK firmware na ESP32-C3
- [ ] Potvrditi da se ESP32 boota (LED, WiFi AP vidljiv)

## Faza 4: BLE test (na stolu, bez auta)
- [ ] Spojiti se na WiCAN WiFi AP (192.168.80.1)
- [ ] Spojiti Car Scanner app preko BLE
- [ ] Poslati ATZ komandu - ocekivani odgovor: ELM327 v1.5
- [ ] Poslati ATSP6 (set CAN protocol)

## Faza 5: Test na vozilu (Golf 6)
- [ ] Spojiti OBD2 kabel na auto
- [ ] Spojiti CAN modul na OBD2 kabel (pin 6, 14, 5)
- [ ] Citati standardne PID-ove (RPM, brzina, temp)
- [ ] Citati fuel trim (bank level)
- [ ] Citati DTC kodove
- [ ] Testirati UDS Service $22 (manufacturer-specific)

## Faza 6: PCB dizajn
- [ ] PCB layout u KiCad-u (ESP32-C3 + CAN + power + OBD2)
- [ ] Dodati UART test padove za flashing
- [ ] Dodati BOOT tipkalo (GPIO9 -> GND) za flash mode
- [ ] Antena keep-out zona (bez bakra ispod PCB antene)
- [ ] CAN differential pair routing (CANH/CANL paralelno)
- [ ] Buck induktor petlja minimalna (L1 blizu U1)
- [ ] Pin header za spajanje na OBD2 konektor u kucistu
- [ ] DRC provjera (0 errors)
- [ ] Generirati Gerber + BOM + CPL fajlove

## Faza 7: Prototip PCB (5 komada)
- [ ] Naruciti 5 kom PCBA na JLCPCB
- [ ] Flashati firmware na prototip
- [ ] Testirati prototip na vozilu
- [ ] Testirati fit u OBD2 kuciste

## Faza 8: Provirium app integracija
- [ ] BLE streaming podataka -> mobilna app
- [ ] Standardni OBD-II PID citanje (RPM, temp, fuel trim, DTC)
- [ ] UDS Service $22 podrska (manufacturer-specific DID-ovi)
- [ ] Vehicle profil sistem (JSON format, kao WiCAN)
- [ ] Auto-discovery: skeniranje DID-ova (22 0000-FFFF)
- [ ] AI klasifikacija nepoznatih DID odgovora
- [ ] Cloud sync profila izmedju korisnika
- [ ] Importirati postojecih 72 WiCAN profila
- [ ] OpenDBC DBC-to-JSON converter pipeline

## Faza 9: Serijska proizvodnja (200 komada)
- [ ] Finalizirati BOM i PCB dizajn
- [ ] Naruciti 200 kom JLCPCB turnkey PCBA
- [ ] Napraviti pogo pin jig za flashing
- [ ] Flashati svih 200 komada
- [ ] OTA update mehanizam testiran

## Faza 10: Kuciste i pakiranje
- [ ] Odabrati/dizajnirati OBD2 dongle kuciste
- [ ] Testirati form factor (60x40x20mm)
- [ ] Pakiranje i labeliranje
