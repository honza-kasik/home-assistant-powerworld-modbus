# Modbus RTU pro Power World PW030-DKZLRS-A

Jednotka má štítek `PW58286` s označením `2022-05-25L`. Přes převodník s nastavením `9600 8N1` při živém testu odpověděla na adrese `1`. Funkce `03` na registru `0x800E` vrátila `284`, což podle převodu z protokolu PW58332 odpovídá `28,4 °C`. Ve veřejných podkladech zůstává nedohledaná přesná mapa registrů pro `PW58286`.

## Konektory a přepínače

Na [fotografii ověřené instalace](podklady/pw58286-modbus-svorkovnice.png) je modrá třípólová svorkovnice u spodního okraje desky. V této orientaci jsou svorky zleva **GND – A – B**. GND je nezapojená, oranžový vodič je na A a bílý na B. Barvy popisují zapojení této konkrétní instalace.

- Schéma jednofázové jednotky PW030/040-DKZLRS-A uvádí desku `CC1010B`, výkres `803004-2978`. Na výkresu jsou zleva doprava piny `CN485` v pořadí `GND`, `A`, `B` a svorky `CN9` v pořadí `A`, `B`. Kabel ovládacího panelu vede do `CN6`. Účel `CN485` a `CN9` pro externí Modbus a sériové nastavení příručka neupřesňuje. [Instalační příručka Power World, str. 22](https://svt.sfzp.cz/stahnout-soubor/YKNZmQipmY7wNSHYRv0Pb8GfpORv6iQgelFnOHEy#page=25)
- Tabulka na stejném schématu přiřazuje `SW1` konfiguraci výbavy. Pozice 1 povoluje `DC 1`, pozice 2 je rezervovaná, pozice 3 nastavuje dostupnost ohřevu TUV a pozice 4 chlazení. Tabulka adresových přepínačů z protokolu `PW58332` se proto na `SW1` této desky nevztahuje. [Instalační příručka Power World, str. 22](https://svt.sfzp.cz/stahnout-soubor/YKNZmQipmY7wNSHYRv0Pb8GfpORv6iQgelFnOHEy#page=25) · [PW58332 V1.01](podklady/Poolworld_PW58332_MODBUS_protocal_V1.01.doc)
- Funkční externí připojení přes `CN485` popisuje autor integrace pro Power World **R290**. Jeho zkušenost podporuje volbu `CN485` jako prvního kandidáta na `PW58286`. Zapojení jeho konkrétního kabelu platí pro jeho jednotku. [Záznam autora integrace pro R290](https://github.com/Mike19700/domoticz-PowerWorld-heat-pump#readme)

## Příbuzné protokoly

| Dokument | Parametry uvedené v dokumentu |
| --- | --- |
| [PW58321 V1.0, Easylife bez kaskády](https://www.photovoltaikforum.com/core/file-download/521505/) | RTU `9600 8N1`, pevná adresa `1`, funkce `03`, teplota vratné vody na `0x800E`. Produktová stránka uvádí PW030-DKZLRS-A v řadě [Easylife R32](https://powerworld-e.nl/products/residentiele-warmtepompen/easylife-series-2). Protokol má označení `PW58321`. |
| [PW58332-2, revize z května 2023](https://www.photovoltaikforum.com/core/file-download/521506/) | RTU `9600 8N1`, adresy `1–16` podle DIP, funkce `03`. Konfigurace vychází z [PW58332 V1.01](podklady/Poolworld_PW58332_MODBUS_protocal_V1.01.doc). |
| [PW58329 V1.0, CC1010B, R290](https://www.photovoltaikforum.com/core/file-download/521507/) | RTU `9600 8N1`, adresy `1–8`, funkce `03`, teplota vstupní vody na `0x000E`. Specifikace uvádí jako master počítač nebo kabelový ovladač. |

Rozdílné adresy teploty ukazují, proč shoda rodiny výrobku nebo označení desky sama o sobě nestačí k výběru celé mapy registrů. Živý test potvrzuje odpověď `PW58286` na `0x800E`. Význam ostatních registrů a jejich zápis ještě vyžadují ověření. [PW58321](https://www.photovoltaikforum.com/core/file-download/521505/) · [PW58329](https://www.photovoltaikforum.com/core/file-download/521507/)

## Rozsah ověření

Připojení na adrese `1` při `9600 8N1` je ověřené živým čtením. Přesná mapa registrů pro firmware `PW58286`, způsob změny adresy a souběh externího řízení s ovládacím panelem nejsou v dostupných podkladech potvrzené. Konfigurace vychází z dostupného protokolu `PW58332 V1.01`; ověřené převody jsou uvedené v [mapě registrů](modbus-registers.md).
