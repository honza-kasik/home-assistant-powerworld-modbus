# Modbus registry Power World PW030

Mapa vychází z protokolu PW58332 V1.01. U jednotky PW58286 je ověřené čtení na adrese 1. Význam hodnot a rozsah zápisu se dají ověřit porovnáním s ovládacím panelem. [Protokol PW58332 V1.01](podklady/Poolworld_PW58332_MODBUS_protocal_V1.01.doc) · [Podklady k desce PW58286](modbus-port-research.md)

## Převody hodnot

- Provozní teploty s poznámkou `Precision 0.1` nebo `Precision 0.5` jsou znaménkové s příslušným násobkem. Servisní nastavení se záporným rozsahem používá dvojkový doplněk.
- Koeficient P24 se zadává od 0,1 do 3,0. U ventilů s poznámkou o násobku zobrazení 2 se do registru zadává jeho surová hodnota.
- Proud kompresoru (`0x8025`) je na ověřené jednotce PW58286 v desetinách ampéru. Hodnota registru `29` znamená `2,9 A`. Napětí DC meziobvodu (`0x8021`) se zobrazuje s převodem 1:1.
- Ochrana a obnovení nízkého tlaku (`0x0044`, `0x0045`), teplota dovolené (`0x00F2`) a příznak časování tichého režimu (`0x00F5`) se zobrazují jako surové hodnoty. Mapa je označuje jako zapisovatelné a chybí v ní převod, rozsah nebo rozpis bitů.
- Stavové a poruchové registry se zobrazují jako číselná slova. Servisní balíček poskytuje i binární entity jednotlivých popsaných bitů.

## Zápis v Home Assistantu

Servisní výběr zobrazuje hodnotu, jednotku a povolený rozsah. Pole **PW030 Nová hodnota** přijímá desetinnou tečku i čárku. Výčty používají číselný kód a příznaky hodnoty 0 nebo 1. Ovladače se zpřístupní po zapnutí **PW030 Povolit zápis**. Skript načte aktuální hodnotu, zapíše změnu a výsledek ověří. Při změně bitu zachovává ostatní bity načteného slova. Souběžný zápis do stejného registru z panelu může mezi čtením a zápisem změnit jeho obsah. Řídicí jednotka při zápisu ukládá nastavení do paměti.

### Režim výkonu

`select.pw030_rezim_vykonu` nabízí Smart, Silent a Powerful. Volba mění současně bity 4 a 5 registru `0x0001` a zachovává ostatní bity. Smart znamená oba bity vypnuté, Powerful bit 4 a Silent bit 5. Toto mapování Smart vychází z běžného režimu bez obou příznaků; přepnutí je vhodné porovnat s panelem. Pokud jsou oba bity aktivní, výběr nehlásí žádný z těchto tří režimů.

### Jednorázový zápis bez balíčku

V **Nástroje pro vývojáře → Akce → Modbus: Write register** se používá název hubu čerpadla z `modbus.yaml` a jeho Modbus adresa, v této konfiguraci `1`. R30 pro tiché vytápění používá adresu `239`, R32 pro tiché chlazení adresu `241`. Mapa u obou uvádí hodnoty od 30 do 120 Hz. Limit se uplatní při zapnutém tichém režimu. Po zápisu lze hodnotu zkontrolovat na panelu. [Dokumentace akce Home Assistantu](https://www.home-assistant.io/actions/modbus.write_register/)

### Karta se servisním výběrem

```yaml
type: entities
title: PW030 servis
entities:
  - input_boolean.pw030_povolit_zapis
  - number.pw030_tichy_limit_vytapeni
  - number.pw030_tichy_limit_chlazeni
  - input_select.pw030_servisni_parametr
  - sensor.pw030_aktualni_servisni_hodnota
  - type: attribute
    entity: sensor.pw030_aktualni_servisni_hodnota
    attribute: povoleny_zapis
    name: Povolený zápis
  - type: attribute
    entity: sensor.pw030_aktualni_servisni_hodnota
    attribute: jednotka
    name: Jednotka
  - type: attribute
    entity: sensor.pw030_aktualni_servisni_hodnota
    attribute: popis
    name: Popis
  - input_text.pw030_nova_hodnota
  - button.pw030_zapsat_servisni_parametr
```

### Tovární reset

Zápis `1` funkcí 06 do registru `0x03E8` (`1000`) obnoví tovární nastavení a registr se následně sám vynuluje. Akce `modbus.write_register` používá hub a Modbus adresu čerpadla, adresu registru `1000` a hodnotu `1`. Před resetem má smysl uložit dosavadní nastavení. [Protokol PW58332 V1.01](podklady/Poolworld_PW58332_MODBUS_protocal_V1.01.doc)

## Mapa registrů

Sloupec **Zápis v balíčku** uvádí hodnoty zadávané v servisním UI. Pomlčka značí čtení nebo rezervu. U řídicích slov jsou zapisovatelné jednotlivé popsané bity.

| Adresa | Parametr | Protokol | Jednotka | Zápis v balíčku |
| --- | --- | --- | --- | --- |
| `0x0000` | Řídicí slovo 1 | RW | — | Viz popsané bity |
| `0x0001` | Řídicí slovo 2 | RW | — | Viz popsané bity |
| `0x0002` | Řídicí slovo 3 | RW | — | Viz popsané bity |
| `0x0003` | Počet kompresorů | RW | — | 1 |
| `0x0004` | Provozní režim | RW | — | 0 = Ohřev TUV, 1 = Vytápění, 2 = Chlazení, 3 = Ohřev TUV + vytápění, 4 = Ohřev TUV + chlazení |
| `0x0005` | P09 Frekvence odmrazování | RW | Hz | 30 až 120 |
| `0x0006` | P10 Perioda odmrazování | RW | min | 20 až 90 |
| `0x0007` | P12 Doba odmrazování | RW | min | 5 až 20 |
| `0x0008` | A01 Perioda regulace hlavního expanzního ventilu | RW | s | 20 až 90 |
| `0x0009` | A14 Počáteční otevření hlavního ventilu při vytápění 1 | RW | — | 0 až 240 |
| `0x000A` | A15 Počáteční otevření hlavního ventilu při vytápění 2 | RW | — | 0 až 240 |
| `0x000B` | A16 Počáteční otevření hlavního ventilu při vytápění 3 | RW | — | 0 až 240 |
| `0x000C` | A17 Počáteční otevření hlavního ventilu při vytápění 4 | RW | — | 0 až 240 |
| `0x000D` | A18 Počáteční otevření hlavního ventilu při vytápění 5 | RW | — | 0 až 240 |
| `0x000E` | A19 Počáteční otevření hlavního ventilu při vytápění 6 | RW | — | 0 až 240 |
| `0x000F` | A20 Počáteční otevření hlavního ventilu při vytápění 7 | RW | — | 0 až 240 |
| `0x0010` | A21 Počáteční otevření hlavního ventilu při vytápění 8 | RW | — | 0 až 240 |
| `0x0011` | A22 Počáteční otevření hlavního ventilu při chlazení 1 | RW | — | 0 až 240 |
| `0x0012` | A23 Počáteční otevření hlavního ventilu při chlazení 2 | RW | — | 0 až 240 |
| `0x0013` | A24 Počáteční otevření hlavního ventilu při chlazení 3 | RW | — | 0 až 240 |
| `0x0014` | A25 Počáteční otevření hlavního ventilu při chlazení 4 | RW | — | 0 až 240 |
| `0x0015` | A26 Počáteční otevření hlavního ventilu při ohřevu TUV 1 | RW | — | 0 až 240 |
| `0x0016` | A27 Počáteční otevření hlavního ventilu při ohřevu TUV 2 | RW | — | 0 až 240 |
| `0x0017` | A28 Počáteční otevření hlavního ventilu při ohřevu TUV 3 | RW | — | 0 až 240 |
| `0x0018` | A29 Počáteční otevření hlavního ventilu při ohřevu TUV 4 | RW | — | 0 až 240 |
| `0x0019` | A30 Minimum hlavního ventilu v automatice při vytápění 1 | RW | — | 0 až 240 |
| `0x001A` | A31 Minimum hlavního ventilu v automatice při vytápění 2 | RW | — | 0 až 240 |
| `0x001B` | A32 Minimum hlavního ventilu v automatice při vytápění 3 | RW | — | 0 až 240 |
| `0x001C` | A33 Minimum hlavního ventilu v automatice při vytápění 4 | RW | — | 0 až 240 |
| `0x001D` | A34 Minimum hlavního ventilu v automatice při vytápění 5 | RW | — | 0 až 240 |
| `0x001E` | A35 Minimum hlavního ventilu v automatice při vytápění 6 | RW | — | 0 až 240 |
| `0x001F` | A36 Minimum hlavního ventilu v automatice při vytápění 7 | RW | — | 0 až 240 |
| `0x0020` | A37 Minimum hlavního ventilu v automatice při vytápění 8 | RW | — | 0 až 240 |
| `0x0021` | A39 Otevření hlavního expanzního ventilu při odmrazování | RW | — | 10 až 225 |
| `0x0022` | A40 Minimum otevření hlavního expanzního ventilu při ohřevu TUV | RW | — | 25 až 75 |
| `0x0023` | A42 Ruční počet kroků hlavního expanzního ventilu | RW | — | 10 až 225 |
| `0x0024` | A43 Proporcionální koeficient hlavního ventilu podle přehřátí | RW | — | 1 až 20 |
| `0x0025` | A44 Diferenční koeficient hlavního ventilu podle přehřátí | RW | — | 1 až 180 |
| `0x0026` | B02 Ruční počet kroků pomocného expanzního ventilu | RW | — | 10 až 225 |
| `0x0027` | B04 Proporcionální koeficient pomocného ventilu podle výtlaku | RW | — | 1 až 20 |
| `0x0028` | B05 Diferenční koeficient pomocného ventilu podle výtlaku | RW | — | 0 až 180 |
| `0x0029` | B06 Proporcionální koeficient pomocného ventilu podle přehřátí | RW | — | 1 až 20 |
| `0x002A` | B07 Diferenční koeficient pomocného ventilu podle přehřátí | RW | — | 0 až 180 |
| `0x002B` | B08 Perioda regulace pomocného expanzního ventilu | RW | — | 10 až 20 |
| `0x002C` | B19 Počáteční otevření pomocného ventilu při vytápění 1 | RW | — | 0 až 240 |
| `0x002D` | B20 Počáteční otevření pomocného ventilu při vytápění 2 | RW | — | 0 až 240 |
| `0x002E` | B21 Počáteční otevření pomocného ventilu při vytápění 3 | RW | — | 0 až 240 |
| `0x002F` | B22 Počáteční otevření pomocného ventilu při vytápění 4 | RW | — | 0 až 240 |
| `0x0030` | B23 Počáteční otevření pomocného ventilu při vytápění 5 | RW | — | 0 až 240 |
| `0x0031` | B24 Počáteční otevření pomocného ventilu při vytápění 6 | RW | — | 0 až 240 |
| `0x0032` | B25 Počáteční otevření pomocného ventilu při vytápění 7 | RW | — | 0 až 240 |
| `0x0033` | B26 Počáteční otevření pomocného ventilu při vytápění 8 | RW | — | 0 až 240 |
| `0x0034` | B27 Počáteční otevření pomocného ventilu při ohřevu TUV 1 | RW | — | 0 až 240 |
| `0x0035` | B28 Počáteční otevření pomocného ventilu při ohřevu TUV 2 | RW | — | 0 až 240 |
| `0x0036` | B29 Počáteční otevření pomocného ventilu při ohřevu TUV 3 | RW | — | 0 až 240 |
| `0x0037` | B30 Počáteční otevření pomocného ventilu při ohřevu TUV 4 | RW | — | 0 až 240 |
| `0x0038` | B31 Minimum pomocného ventilu v automatice při vytápění 1 | RW | — | 0 až 240 |
| `0x0039` | B32 Minimum pomocného ventilu v automatice při vytápění 2 | RW | — | 0 až 240 |
| `0x003A` | B33 Minimum pomocného ventilu v automatice při vytápění 3 | RW | — | 0 až 240 |
| `0x003B` | B34 Minimum pomocného ventilu v automatice při vytápění 4 | RW | — | 0 až 240 |
| `0x003C` | B35 Minimum pomocného ventilu v automatice při vytápění 5 | RW | — | 0 až 240 |
| `0x003D` | B36 Minimum pomocného ventilu v automatice při vytápění 6 | RW | — | 0 až 240 |
| `0x003E` | B37 Minimum pomocného ventilu v automatice při vytápění 7 | RW | — | 0 až 240 |
| `0x003F` | B38 Minimum pomocného ventilu v automatice při vytápění 8 | RW | — | 0 až 240 |
| `0x0040` | B39 Otevření pomocného expanzního ventilu při odmrazování | RW | — | 0 až 240 |
| `0x0041` | B40 Otevření pomocného expanzního ventilu při chlazení | RW | — | 0 až 240 |
| `0x0042` | Rezerva (C03 High pressure protection value(reserve)) | RW | — | — |
| `0x0043` | Rezerva (C04 High pressure recovery value(reserve)) | RW | — | — |
| `0x0044` | C05 Mez ochrany nízkého tlaku | RW | bar (převod neuveden) | — |
| `0x0045` | C06 Mez obnovení po nízkém tlaku | RW | bar (převod neuveden) | — |
| `0x0046` | D03 Otáčky ventilátoru při vytápění 1 | RW | — | 0 až 100 |
| `0x0047` | D04 Otáčky ventilátoru při vytápění 2 | RW | — | 0 až 100 |
| `0x0048` | D05 Otáčky ventilátoru při vytápění 3 | RW | — | 0 až 100 |
| `0x0049` | D06 Otáčky ventilátoru při vytápění 4 | RW | — | 0 až 100 |
| `0x004A` | D07 Otáčky ventilátoru při vytápění 5 | RW | — | 0 až 100 |
| `0x004B` | D08 Otáčky ventilátoru při vytápění 6 | RW | — | 0 až 100 |
| `0x004C` | Rezerva | RW | — | — |
| `0x004D` | Rezerva | RW | — | — |
| `0x004E` | D11 Teplota výměníku pro otáčky ventilátoru při vytápění 1 | RW | — | -30 až 30 |
| `0x004F` | D12 Teplota výměníku pro otáčky ventilátoru při vytápění 2 | RW | — | -30 až 30 |
| `0x0050` | D13 Teplota výměníku pro otáčky ventilátoru při vytápění 3 | RW | — | -30 až 30 |
| `0x0051` | D14 Teplota výměníku pro otáčky ventilátoru při vytápění 4 | RW | — | -30 až 30 |
| `0x0052` | D15 Teplota výměníku pro otáčky ventilátoru při vytápění 5 | RW | — | -30 až 30 |
| `0x0053` | D16 Teplota výměníku pro otáčky ventilátoru při vytápění 6 | RW | — | -30 až 30 |
| `0x0054` | Rezerva (D17reserve) | RW | — | — |
| `0x0055` | Rezerva (D18reserve) | RW | — | — |
| `0x0056` | D19 Perioda regulace DC ventilátoru | RW | s | 10 až 180 |
| `0x0057` | D20 Změna otáček ventilátoru za periodu | RW | rpm | 0 až 100 |
| `0x0058` | D21 Otáčky ventilátoru při ohřevu TUV 1 | RW | — | 0 až 1000 |
| `0x0059` | D22 Otáčky ventilátoru při ohřevu TUV 2 | RW | — | 0 až 1000 |
| `0x005A` | D23 Otáčky ventilátoru při ohřevu TUV 3 | RW | — | 0 až 1000 |
| `0x005B` | D24 Otáčky ventilátoru při ohřevu TUV 4 | RW | — | 0 až 1000 |
| `0x005C` | D25 Teplota výměníku pro otáčky ventilátoru při ohřevu TUV 1 | RW | — | -30 až 30 |
| `0x005D` | D26 Teplota výměníku pro otáčky ventilátoru při ohřevu TUV 2 | RW | — | -30 až 30 |
| `0x005E` | D27 Teplota výměníku pro otáčky ventilátoru při ohřevu TUV 3 | RW | — | -30 až 30 |
| `0x005F` | D28 Teplota výměníku pro otáčky ventilátoru při ohřevu TUV 4 | RW | — | -30 až 30 |
| `0x0060` | D29 Maximum otáček ventilátoru při chlazení 1 | RW | — | 0 až 1000 |
| `0x0061` | D30 Maximum otáček ventilátoru při chlazení 2 | RW | — | 0 až 1000 |
| `0x0062` | D31 Maximum otáček ventilátoru při chlazení 3 | RW | — | 0 až 1000 |
| `0x0063` | D32 Maximum otáček ventilátoru při chlazení 4 | RW | — | 0 až 1000 |
| `0x0064` | B41 Minimum pomocného ventilu v automatice při ohřevu TUV 1 | RW | — | 0 až 240 |
| `0x0065` | B42 Minimum pomocného ventilu v automatice při ohřevu TUV 2 | RW | — | 0 až 240 |
| `0x0066` | B43 Minimum pomocného ventilu v automatice při ohřevu TUV 3 | RW | — | 0 až 240 |
| `0x0067` | B44 Minimum pomocného ventilu v automatice při ohřevu TUV 4 | RW | — | 0 až 240 |
| `0x0068` | B45 Teplota výtlaku pro otevření EVI při vytápění 1 | RW | °C | 50 až 125 |
| `0x0069` | B46 Teplota výtlaku pro otevření EVI při vytápění 2 | RW | °C | 50 až 125 |
| `0x006A` | B47 Teplota výtlaku pro otevření EVI při vytápění 3 | RW | °C | 50 až 125 |
| `0x006B` | B48 Teplota výtlaku pro otevření EVI při vytápění 4 | RW | °C | 50 až 125 |
| `0x006C` | B49 Teplota výtlaku pro otevření EVI při vytápění 5 | RW | °C | 50 až 125 |
| `0x006D` | B50 Teplota výtlaku pro otevření EVI při vytápění 6 | RW | °C | 50 až 125 |
| `0x006E` | B51 Teplota výtlaku pro otevření EVI při vytápění 7 | RW | °C | 50 až 125 |
| `0x006F` | B52 Teplota výtlaku pro otevření EVI při vytápění 8 | RW | °C | 50 až 125 |
| `0x0070` | B53 Teplota výtlaku pro otevření EVI při ohřevu TUV 1 | RW | °C | 50 až 125 |
| `0x0071` | B54 Teplota výtlaku pro otevření EVI při ohřevu TUV 2 | RW | °C | 50 až 125 |
| `0x0072` | B55 Teplota výtlaku pro otevření EVI při ohřevu TUV 3 | RW | °C | 50 až 125 |
| `0x0073` | B56 Teplota výtlaku pro otevření EVI při ohřevu TUV 4 | RW | °C | 50 až 125 |
| `0x0074` | B57 Teplota výtlaku pro otevření EVI při chlazení 1 | RW | °C | 50 až 125 |
| `0x0075` | B58 Teplota výtlaku pro otevření EVI při chlazení 2 | RW | °C | 50 až 125 |
| `0x0076` | B59 Teplota výtlaku pro otevření EVI při chlazení 3 | RW | °C | 50 až 125 |
| `0x0077` | B60 Teplota výtlaku pro otevření EVI při chlazení 4 | RW | °C | 50 až 125 |
| `0x0078` | B61 Zpoždění otevření ventilu EVI | RW | s | 0 až 180 |
| `0x0079` | B63 Hystereze výtlaku pro zavření EVI | RW | — | 0 až 30 |
| `0x007A` | B64 Rozdíl teploty výtlaku pro pomocný ventil při vytápění 1 | RW | °C | 0 až 125 |
| `0x007B` | B65 Rozdíl teploty výtlaku pro pomocný ventil při vytápění 2 | RW | °C | 0 až 125 |
| `0x007C` | B66 Rozdíl teploty výtlaku pro pomocný ventil při vytápění 3 | RW | °C | 0 až 125 |
| `0x007D` | B67 Rozdíl teploty výtlaku pro pomocný ventil při vytápění 4 | RW | °C | 0 až 125 |
| `0x007E` | B68 Rozdíl teploty výtlaku pro pomocný ventil při vytápění 5 | RW | °C | 0 až 125 |
| `0x007F` | B69 Rozdíl teploty výtlaku pro pomocný ventil při vytápění 6 | RW | °C | 0 až 125 |
| `0x0080` | B70 Rozdíl teploty výtlaku pro pomocný ventil při vytápění 7 | RW | °C | 0 až 125 |
| `0x0081` | B71 Rozdíl teploty výtlaku pro pomocný ventil při vytápění 8 | RW | °C | 0 až 125 |
| `0x0082` | B72 Rozdíl teploty výtlaku pro pomocný ventil při ohřevu TUV 1 | RW | °C | 0 až 125 |
| `0x0083` | B73 Rozdíl teploty výtlaku pro pomocný ventil při ohřevu TUV 2 | RW | °C | 0 až 125 |
| `0x0084` | B74 Rozdíl teploty výtlaku pro pomocný ventil při ohřevu TUV 3 | RW | °C | 0 až 125 |
| `0x0085` | B75 Rozdíl teploty výtlaku pro pomocný ventil při ohřevu TUV 4 | RW | °C | 0 až 125 |
| `0x0086` | B77 Korekce cílového přehřátí 1 | RW | °C | -30 až 30 |
| `0x0087` | B78 Korekce cílového přehřátí 2 | RW | °C | -30 až 30 |
| `0x0088` | B79 Korekce cílového přehřátí 3 | RW | °C | -30 až 30 |
| `0x0089` | B80 Korekce cílového přehřátí 4 | RW | °C | -30 až 30 |
| `0x008A` | B81 Korekce cílového přehřátí 5 | RW | °C | -30 až 30 |
| `0x008B` | B82 Korekce cílového přehřátí 6 | RW | °C | -30 až 30 |
| `0x008C` | B83 Korekce cílového přehřátí 7 | RW | °C | -30 až 30 |
| `0x008D` | P01 Rozdíl vratné vody a cílové teploty chlazení | RW | °C | 2 až 18 |
| `0x008E` | P02 Rozdíl vratné vody a cílové teploty TUV | RW | °C | 2 až 18 |
| `0x008F` | P03 Cílová teplota TUV | RW | °C | 28 až 60 |
| `0x0090` | P04 Cílová teplota chlazení | RW | °C | 7 až 30 |
| `0x0091` | P05 Cílová teplota vytápění | RW | °C | 15 až 50 |
| `0x0092` | P08 Korekce teploty vody | RW | °C | -5 až 15 |
| `0x0093` | P11 Teplota zahájení odmrazování | RW | °C | -15 až -1 |
| `0x0094` | P13 Teplota ukončení odmrazování | RW | °C | 1 až 40 |
| `0x0095` | P14 Rozdíl teplot okolí a výparníku pro odmrazování 1 | RW | °C | 0 až 15 |
| `0x0096` | P15 Rozdíl teplot okolí a výparníku pro odmrazování 2 | RW | °C | 0 až 15 |
| `0x0097` | P16 Venkovní teplota pro odmrazování | RW | °C | 0 až 20 |
| `0x0098` | A02 Cílové přehřátí hlavního ventilu při vytápění 1 | RW | °C | -5 až 10 |
| `0x0099` | A03 Cílové přehřátí hlavního ventilu při vytápění 2 | RW | °C | -5 až 10 |
| `0x009A` | A04 Cílové přehřátí hlavního ventilu při vytápění 3 | RW | °C | -5 až 10 |
| `0x009B` | A05 Cílové přehřátí hlavního ventilu při vytápění 4 | RW | °C | -5 až 10 |
| `0x009C` | A06 Cílové přehřátí hlavního ventilu při vytápění 5 | RW | °C | -5 až 10 |
| `0x009D` | A07 Cílové přehřátí hlavního ventilu při vytápění 6 | RW | °C | -5 až 10 |
| `0x009E` | A08 Cílové přehřátí hlavního ventilu při vytápění 7 | RW | °C | -5 až 10 |
| `0x009F` | A09 Cílové přehřátí hlavního ventilu při vytápění 8 | RW | °C | -5 až 10 |
| `0x00A0` | A10 Cílové přehřátí hlavního ventilu při chlazení 1 | RW | °C | -5 až 10 |
| `0x00A1` | A11 Cílové přehřátí hlavního ventilu při chlazení 2 | RW | °C | -5 až 10 |
| `0x00A2` | A12 Cílové přehřátí hlavního ventilu při chlazení 3 | RW | °C | -5 až 10 |
| `0x00A3` | A13 Cílové přehřátí hlavního ventilu při chlazení 4 | RW | °C | -5 až 10 |
| `0x00A4` | A38 Teplota výtlaku pro regulaci hlavního ventilu | RW | °C | 70 až 125 |
| `0x00A5` | B03 Venkovní teplota pro otevření elektromagnetického ventilu EVI | RW | °C | 11 až 45 |
| `0x00A6` | B09 Cílová teplota výtlaku pomocného ventilu | RW | — | 70 až 120 |
| `0x00A7` | B10 Teplota výtlaku pro zavření pomocného ventilu | RW | — | 40 až 70 |
| `0x00A8` | B11 Cílové přehřátí pomocného ventilu při vytápění 1 | RW | — | -10 až 10 |
| `0x00A9` | B12 Cílové přehřátí pomocného ventilu při vytápění 2 | RW | — | -10 až 10 |
| `0x00AA` | B13 Cílové přehřátí pomocného ventilu při vytápění 3 | RW | — | -10 až 10 |
| `0x00AB` | B14 Cílové přehřátí pomocného ventilu při vytápění 4 | RW | — | -10 až 10 |
| `0x00AC` | B15 Cílové přehřátí pomocného ventilu při vytápění 5 | RW | — | -10 až 10 |
| `0x00AD` | B16 Cílové přehřátí pomocného ventilu při vytápění 6 | RW | — | -10 až 10 |
| `0x00AE` | B17 Cílové přehřátí pomocného ventilu při vytápění 7 | RW | — | -10 až 10 |
| `0x00AF` | B18 Cílové přehřátí pomocného ventilu při vytápění 8 | RW | — | -10 až 10 |
| `0x00B0` | Rezerva (D01 AC fan speed switch ambient temp (reserve)) | RW | — | — |
| `0x00B1` | Rezerva (D02 AC fan speed switch ambient temp (reserve)) | RW | — | — |
| `0x00B2` | R16 Teplota výtlaku TP 0 | RW | °C | 50 až 125 |
| `0x00B3` | R17 Teplota výtlaku TP 1 | RW | °C | 50 až 125 |
| `0x00B4` | R18 Teplota výtlaku TP 2 | RW | °C | 50 až 125 |
| `0x00B5` | R19 Teplota výtlaku TP 3 | RW | °C | 50 až 125 |
| `0x00B6` | R20 Teplota výtlaku TP 4 | RW | °C | 50 až 125 |
| `0x00B7` | Ruční frekvence kompresoru | RW | — | 15 až 120 |
| `0x00B8` | R00 Provozní frekvence kompresoru 1 | RW | Hz | 30 až 120 |
| `0x00B9` | R01 Provozní frekvence kompresoru 2 | RW | Hz | 30 až 120 |
| `0x00BA` | R02 Provozní frekvence kompresoru 3 | RW | Hz | 30 až 120 |
| `0x00BB` | R03 Provozní frekvence kompresoru 4 | RW | Hz | 30 až 120 |
| `0x00BC` | R04 Provozní frekvence kompresoru 5 | RW | Hz | 30 až 120 |
| `0x00BD` | R05 Provozní frekvence kompresoru 6 | RW | Hz | 30 až 120 |
| `0x00BE` | R06 Provozní frekvence kompresoru 7 | RW | Hz | 30 až 120 |
| `0x00BF` | R07 Provozní frekvence kompresoru 8 | RW | Hz | 30 až 120 |
| `0x00C0` | R08 Provozní frekvence kompresoru 9 | RW | Hz | 30 až 120 |
| `0x00C1` | R09 Provozní frekvence kompresoru 10 | RW | Hz | 30 až 120 |
| `0x00C2` | R10 Provozní frekvence kompresoru 11 | RW | Hz | 30 až 120 |
| `0x00C3` | R11 Provozní frekvence kompresoru 12 | RW | Hz | 30 až 120 |
| `0x00C4` | R12 Minimální frekvence kompresoru po dosažení cíle | RW | Hz | 30 až 120 |
| `0x00C5` | R13 Maximální frekvence kompresoru po dosažení cíle | RW | Hz | 30 až 120 |
| `0x00C6` | Rezerva | RW | — | — |
| `0x00C7` | Rezerva | RW | — | — |
| `0x00C8` | R21 Bod omezení minimální frekvence 1 | RW | Hz | 0 až 125 |
| `0x00C9` | R22 Bod omezení minimální frekvence 2 | RW | Hz | 0 až 125 |
| `0x00CA` | R23 Bod omezení minimální frekvence 3 | RW | Hz | 0 až 125 |
| `0x00CB` | R24 Bod omezení minimální frekvence 4 | RW | Hz | 0 až 125 |
| `0x00CC` | R25 Bod omezení maximální frekvence 1 | RW | Hz | 0 až 125 |
| `0x00CD` | R26 Bod omezení maximální frekvence 2 | RW | Hz | 0 až 125 |
| `0x00CE` | R27 Bod omezení maximální frekvence 3 | RW | Hz | 0 až 125 |
| `0x00CF` | R28 Bod omezení maximální frekvence 4 | RW | Hz | 0 až 125 |
| `0x00D0` | Povolené časovače | RW | — | Viz popsané bity |
| `0x00D1` | Hodina zapnutí časovače 1 | RW | h | 0 až 23 |
| `0x00D2` | Minuta zapnutí časovače 1 | RW | min | 0 až 59 |
| `0x00D3` | Hodina vypnutí časovače 1 | RW | h | 0 až 23 |
| `0x00D4` | Minuta vypnutí časovače 1 | RW | min | 0 až 59 |
| `0x00D5` | Hodina zapnutí časovače 2 | RW | h | 0 až 23 |
| `0x00D6` | Minuta zapnutí časovače 2 | RW | min | 0 až 59 |
| `0x00D7` | Hodina vypnutí časovače 2 | RW | h | 0 až 23 |
| `0x00D8` | Minuta vypnutí časovače 2 | RW | min | 0 až 59 |
| `0x00D9` | Hodina zapnutí časovače 3 | RW | h | 0 až 23 |
| `0x00DA` | Minuta zapnutí časovače 3 | RW | min | 0 až 59 |
| `0x00DB` | Hodina vypnutí časovače 3 | RW | h | 0 až 23 |
| `0x00DC` | Minuta vypnutí časovače 3 | RW | min | 0 až 59 |
| `0x00DD` | Hodina zapnutí časovače 4 | RW | h | 0 až 23 |
| `0x00DE` | Minuta zapnutí časovače 4 | RW | min | 0 až 59 |
| `0x00DF` | Hodina vypnutí časovače 4 | RW | h | 0 až 23 |
| `0x00E0` | Minuta vypnutí časovače 4 | RW | min | 0 až 59 |
| `0x00E1` | Hodina zapnutí časovače 5 | RW | h | 0 až 23 |
| `0x00E2` | Minuta zapnutí časovače 5 | RW | min | 0 až 59 |
| `0x00E3` | Hodina vypnutí časovače 5 | RW | h | 0 až 23 |
| `0x00E4` | Minuta vypnutí časovače 5 | RW | min | 0 až 59 |
| `0x00E5` | Rezerva | RW | — | — |
| `0x00E6` | P17 Interval vysokoteplotní dezinfekce | RW | d | 0 až 30 |
| `0x00E7` | P18 Hodina začátku dezinfekce | RW | h | 0 až 23 |
| `0x00E8` | P19 Doba udržování dezinfekční teploty | RW | min | 0 až 90 |
| `0x00E9` | P20 Cílová dezinfekční teplota | RW | °C | 0 až 90 |
| `0x00EA` | P21 Cílová teplota čerpadla při dezinfekci | RW | °C | 40 až 60 |
| `0x00EB` | P23 Venkovní teplota pro ekvitermní kompenzaci | RW | — | 0 až 40 |
| `0x00EC` | P24 Koeficient kompenzace cílové teploty | RW | — | 0.1 až 3.0 |
| `0x00ED` | P26 Venkovní teplota pro spuštění elektrického dohřevu | RW | °C | -20 až 20 |
| `0x00EE` | R29 Zvýšení frekvence při výkonném vytápění | RW | Hz | -30 až 30 |
| `0x00EF` | R30 Maximální frekvence při tichém vytápění | RW | Hz | 30 až 120 |
| `0x00F0` | R31 Zvýšení frekvence při výkonném chlazení | RW | Hz | -30 až 30 |
| `0x00F1` | R32 Maximální frekvence při tichém chlazení | RW | Hz | 30 až 120 |
| `0x00F2` | Cílová teplota v režimu dovolené | RW | — | — |
| `0x00F3` | D08 Maximální otáčky ventilátoru při tichém vytápění nebo ohřevu TUV | RW | — | 0 až 1000 |
| `0x00F4` | D09 Maximální otáčky ventilátoru při tichém chlazení | RW | — | 0 až 1000 |
| `0x00F5` | Povolení časování tichého režimu | RW | — | — |
| `0x00F6` | Hodina zapnutí tichého režimu 1 | RW | h | 0 až 23 |
| `0x00F7` | Minuta zapnutí tichého režimu 1 | RW | min | 0 až 59 |
| `0x00F8` | Hodina vypnutí tichého režimu 1 | RW | h | 0 až 23 |
| `0x00F9` | Minuta vypnutí tichého režimu 1 | RW | min | 0 až 59 |
| `0x00FA` | Hodina zapnutí tichého režimu 2 | RW | h | 0 až 23 |
| `0x00FB` | Minuta zapnutí tichého režimu 2 | RW | min | 0 až 59 |
| `0x00FC` | Hodina vypnutí tichého režimu 2 | RW | h | 0 až 23 |
| `0x00FD` | Minuta vypnutí tichého režimu 2 | RW | min | 0 až 59 |
| `0x00FE` | P27 Zpoždění elektrického ohřevu zásobníku | RW | — | 0 až 60 |
| `0x00FF` | R14 Korekce frekvence kompresoru při ohřevu TUV | RW | Hz | -50 až 20 |
| `0x0100` | F1 Dostupné provozní režimy | RW | — | 1 = Vytápění, 2 = Vytápění + chlazení, 3 = Vytápění + ohřev TUV, 4 = Vytápění + chlazení + ohřev TUV |
| `0x0101` | F2 Režim oběhového čerpadla po dosažení cíle | RW | — | 0 = Přerušovaný chod, 1 = Trvalý chod, 2 = Zastavení |
| `0x0102` | Perioda oběhového čerpadla po dosažení cíle | RW | min | 1 až 120 |
| `0x0103` | D1 Osazený DC ventilátor 1 | RW | s | 0 = Ne, 1 = Ano |
| `0x0104` | D2 Osazený DC ventilátor 2 | RW | s | 0 = Ne, 1 = Ano |
| `0x0105` | Režim DC oběhového čerpadla | RW | — | 0 = Vypnutý, 1 = Automatický, 2 = Ruční |
| `0x0106` | Perioda regulace DC oběhového čerpadla | RW | s | 10 až 120 |
| `0x0107` | Ruční rychlost DC oběhového čerpadla | RW | % | 10 až 100 |
| `0x0108` | Maximum rychlosti DC oběhového čerpadla | RW | % | 10 až 100 |
| `0x0109` | Minimum rychlosti DC oběhového čerpadla | RW | % | 10 až 100 |
| `0x010A` | Krok regulace DC oběhového čerpadla | RW | % | 0 až 50 |
| `0x010B` | F10 Rozdíl vstupní a výstupní teploty pro regulaci oběhu | RW | °C | 2 až 30 |
| `0x010C` | F11 Model DC oběhového čerpadla | RW | Hz | 0 = 1000 Hz, 1 = 100 Hz |
| `0x03E8` | Tovární reset | RW | — | Samostatný servisní postup |
| `0x8000` | Rezerva | R | — | — |
| `0x8001` | Rezerva | R | — | — |
| `0x8002` | Stav jednotky | R | — | — |
| `0x8003` | Příznaky výbavy a odmrazování | R | s | — |
| `0x8004` | Výstupní slovo 1 | R | — | — |
| `0x8005` | Výstupní slovo 2 | R | — | — |
| `0x8006` | Výstupní slovo 3 | R | — | — |
| `0x8007` | Poruchové slovo 1 | R | — | — |
| `0x8008` | Poruchové slovo 2 | R | — | — |
| `0x8009` | Poruchové slovo 3 | R | — | — |
| `0x800A` | Poruchové slovo 4 | R | — | — |
| `0x800B` | Poruchové slovo 5 | R | — | — |
| `0x800C` | Poruchové slovo 6 | R | — | — |
| `0x800D` | Poruchové slovo 7 | R | — | — |
| `0x800E` | Teplota vody na vstupu | R | °C | — |
| `0x800F` | Teplota zásobníku TUV | R | °C | — |
| `0x8010` | Rezerva | R | — | — |
| `0x8011` | Venkovní teplota | R | °C | — |
| `0x8012` | Teplota vody na výstupu | R | °C | — |
| `0x8013` | Rezerva (Economical Inlet water temperature(reserve)) | R | — | — |
| `0x8014` | Rezerva (Economical Outlet water temperature(reserve)) | R | — | — |
| `0x8015` | Teplota sání | R | °C | — |
| `0x8016` | Teplota venkovního výměníku | R | °C | — |
| `0x8017` | Rezerva | R | — | — |
| `0x8018` | Rezerva | R | — | — |
| `0x8019` | Rezerva | R | — | — |
| `0x801A` | Teplota výměníku při chlazení | R | °C | — |
| `0x801B` | Teplota výtlaku | R | °C | — |
| `0x801C` | Otevření hlavního expanzního ventilu | R | — | — |
| `0x801D` | Otevření pomocného expanzního ventilu | R | — | — |
| `0x801E` | Skutečná frekvence kompresoru | R | Hz | — |
| `0x801F` | Porucha měniče – dolní bajt | R | — | — |
| `0x8020` | Porucha měniče – horní bajt | R | — | — |
| `0x8021` | Napětí DC meziobvodu | R | V | — |
| `0x8022` | Rezerva | R | — | — |
| `0x8023` | Rezerva | R | — | — |
| `0x8024` | Cílová frekvence kompresoru | R | Hz | — |
| `0x8025` | Proud kompresoru | R | A | — |
| `0x8026` | Otáčky DC ventilátoru 1 | R | — | — |
| `0x8027` | Rezerva | R | — | — |
| `0x8028` | Teplota přepočtená z nízkého tlaku | R | — | — |
| `0x8029` | Rezerva | R | — | — |
| `0x802A` | Skutečná rychlost DC oběhového čerpadla | R | — | — |

### Popsané bity

| Adresa.bit | Příznak | Přístup v balíčku |
| --- | --- | --- |
| `0x0000.0` | Zapnutí čerpadla | Čtení a zápis |
| `0x0000.1` | Ruční režim hlavního expanzního ventilu | Čtení a zápis |
| `0x0000.2` | Ruční režim frekvence kompresoru | Čtení a zápis |
| `0x0000.4` | Trvalý chod čerpadla v režimu konstantní teploty | Čtení a zápis |
| `0x0000.5` | Ruční režim pomocného expanzního ventilu | Čtení a zápis |
| `0x0000.6` | Nastavitelné počáteční otevření hlavního ventilu | Čtení a zápis |
| `0x0001.0` | Výbava pro konstantní teplotu | Čtení a zápis |
| `0x0001.1` | Aktivní tlakové čidlo | Čtení a zápis |
| `0x0001.2` | Zákaz pomocného okruhu chlazení | Čtení a zápis |
| `0x0001.3` | Regulace pomocného ventilu podle přehřátí výtlaku | Čtení a zápis |
| `0x0001.4` | Výkonný režim | Čtení a zápis |
| `0x0001.5` | Tichý režim | Čtení a zápis |
| `0x0001.6` | Automatická korekce cílové teploty vytápění | Čtení a zápis |
| `0x0002.1` | Režim dovolené | Čtení a zápis |
| `0x00D0.0` | Povolení časovače 1 | Čtení a zápis |
| `0x00D0.1` | Povolení časovače 2 | Čtení a zápis |
| `0x00D0.2` | Povolení časovače 3 | Čtení a zápis |
| `0x00D0.3` | Povolení časovače 4 | Čtení a zápis |
| `0x00D0.4` | Povolení časovače 5 | Čtení a zápis |
| `0x8003.0` | Podpora ohřevu TUV | Čtení |
| `0x8003.2` | Podpora vytápění | Čtení |
| `0x8003.3` | Podpora chlazení | Čtení |
| `0x8003.4` | DC ventilátor 1 ve výbavě | Čtení |
| `0x8003.5` | DC ventilátor 2 ve výbavě | Čtení |
| `0x8003.7` | Odmrazování | Čtení |
| `0x8004.0` | Kompresor | Čtení |
| `0x8004.5` | Ventilátor | Čtení |
| `0x8004.6` | Čtyřcestný ventil | Čtení |
| `0x8005.0` | Ohřev vany | Čtení |
| `0x8005.5` | Pomocný elektrický ohřev | Čtení |
| `0x8005.6` | Třícestný ventil | Čtení |
| `0x8005.7` | Elektrický ohřev zásobníku | Čtení |
| `0x8006.0` | Oběhové čerpadlo | Čtení |
| `0x8006.1` | Ohřev klikové skříně | Čtení |
| `0x8007.0` | Er 14 Čidlo zásobníku | Čtení |
| `0x8007.1` | Er 21 Čidlo venkovní teploty | Čtení |
| `0x8007.2` | Er 16 Čidlo venkovního výměníku | Čtení |
| `0x8007.4` | Er 27 Čidlo výstupní vody | Čtení |
| `0x8007.5` | Er 05 Vysoký tlak | Čtení |
| `0x8007.6` | Er 06 Nízký tlak | Čtení |
| `0x8008.0` | Er 03 Průtok vody | Čtení |
| `0x8008.2` | Er 32 Příliš teplá výstupní voda při vytápění | Čtení |
| `0x8009.6` | Er 18 Čidlo teploty výtlaku | Čtení |
| `0x800A.0` | Er 15 Čidlo vstupní vody | Čtení |
| `0x800A.1` | Er 12 Příliš vysoká teplota výtlaku | Čtení |
| `0x800A.5` | Er 23 Příliš studená výstupní voda při chlazení | Čtení |
| `0x800A.6` | Er 29 Čidlo teploty sání | Čtení |
| `0x800B.0` | Er 69 Ochrana proti nízkému tlaku | Čtení |
| `0x800B.2` | Er 33 Příliš vysoká teplota výměníku | Čtení |
| `0x800B.3` | Er 42 Čidlo výměníku chlazení | Čtení |
| `0x800B.7` | Er 67 Nízkotlaké čidlo | Čtení |
| `0x800C.4` | Protimrazová ochrana 2. stupně | Čtení |
| `0x800C.5` | Protimrazová ochrana 1. stupně | Čtení |
| `0x800D.5` | Er 66 DC ventilátor 2 | Čtení |
| `0x800D.6` | Er 64 DC ventilátor 1 | Čtení |
