# Power World PW030 v Home Assistantu

YAML konfigurace pro vestavěnou integraci Modbus v Home Assistantu. Čtení bylo ověřené na PW030-DKZLRS-A s deskou PW58286 (`2022-05-25L`); mapa registrů vychází z protokolu PW58332 V1.01. Kompatibilita s dalšími variantami není ověřená.

Konfigurace byla ověřena s Home Assistant Core **2026.11.2**. Zápis byl ověřen proti simulátoru; všechny servisní parametry nebyly vyzkoušené na skutečném čerpadle.

Konfigurace používá Modbus adresu čerpadla `1`. Ověřené parametry linky RS‑485 jsou `9600 8N1`. Seznam registrů je společný pro USB adaptér RS‑485 i síťový převodník.

## Proč jsou dva YAML soubory

`pw030_modbus_sensors.yaml` definuje čtení registrů čerpadla a vytváří Modbus senzory. Patří pod `sensors:` příslušného hubu a funguje samostatně.

Volitelný balíček `pw030.yaml` z těchto senzorů vytváří pojmenované servisní hodnoty, binární stavy (například chod kompresoru a odmrazování) a ovladače pro zápis. Používá již načtené hodnoty, takže nepřidává druhé periodické čtení stejných registrů. Při zápisu si vyžádá aktuální hodnotu a následně výsledek zkontroluje dalším čtením.

Oddělení odpovídá struktuře konfigurace HA: seznam Modbus senzorů patří pod hub, zatímco balíček sdružuje šablony, ovladače a skripty pod `homeassistant.packages`. Připojení k čerpadlu je tak nastavené na jednom místě a ovládání lze přidat samostatně.

## Čtení

| Soubor v repozitáři | Umístění v HA |
| --- | --- |
| [pw030_modbus_sensors.yaml](pw030_modbus_sensors.yaml) | `/config/pw030_modbus_sensors.yaml` |

V `configuration.yaml` je v hlavní úrovni:

```yaml
modbus: !include modbus.yaml
```

Soubor `modbus.yaml` obsahuje hub s názvem například `tepelne_cerpadlo`. Nastavení připojení závisí na adaptéru:

- USB adaptér RS‑485 používá `type: serial` a `method: rtu`, cestu k sériovému portu a parametry `9600 8N1`. [Sériové připojení v HA](https://www.home-assistant.io/integrations/modbus/#configuring-a-serial-connection)
- Síťová brána s převodem Modbus TCP → RTU používá `type: tcp`, adresu a port brány. [Modbus TCP v HA](https://www.home-assistant.io/integrations/modbus/#configuring-a-modbus-tcp-connection)
- Transparentní síťový převodník přenášející RTU používá `type: rtuovertcp`, adresu a port převodníku. Parametry sériové linky se nastavují v převodníku. [RTU over TCP v HA](https://www.home-assistant.io/integrations/modbus/#configuring-a-modbus-rtu-over-tcp-connection)

Pod zvolený hub patří následující položky společné pro všechny způsoby připojení:

```yaml
  message_wait_milliseconds: 250
  sensors: !include pw030_modbus_sensors.yaml
```

Pokud hub pro tuto RS‑485 linku už existuje, senzory čerpadla patří do něj. Hub má jediný klíč `sensors:`; druhý stejnojmenný klíč by původní seznam přepsal. Pro sloučení lze použít adresář, například `/config/modbus_sensors/`, se soubory `stávající.yaml` a `pw030_modbus_sensors.yaml`. Oba obsahují seznam položek začínajících `- name:`. Pod existujícím hubem pak místo původního `sensors:` patří:

```yaml
  sensors: !include_dir_merge_list modbus_sensors/
```

V této variantě je soubor čerpadla pouze v uvedeném adresáři. Adresář obsahuje senzory daného hubu; senzory ostatních hubů mají vlastní seznamy. Kompletní příklady připojení jsou v [dokumentaci Modbus v HA](https://www.home-assistant.io/integrations/modbus/#configuring-modbus-communication).

Po kontrole konfigurace a restartu jsou v **Nástroje pro vývojáře → Stavy** například `sensor.pw030_teplota_vody_na_vstupu`, `sensor.pw030_venkovni_teplota` a `sensor.pw030_skutecna_frekvence_kompresoru`. Tento jediný seznam čte provozní i servisní hodnoty.

## Ovládání

| Soubor v repozitáři | Umístění v HA |
| --- | --- |
| [pw030.yaml](pw030.yaml) | `/config/packages/pw030.yaml` |

Balíček se načítá přes `homeassistant.packages` v `configuration.yaml`. Při nastavení `packages: !include_dir_named packages` se soubor v adresáři `packages/` načte automaticky. [Balíčky Home Assistantu](https://www.home-assistant.io/docs/configuration/packages/)

```yaml
homeassistant:
  packages:
    pw030: !include packages/pw030.yaml
```

Hodnota `modbus_hub` ve skriptu `pw030_zapsat_registr` odpovídá názvu hubu v `modbus.yaml`. Přednastavený název v balíčku je `tepelne_cerpadlo`, název lze přizpůsobit existujícímu hubu.

Balíček poskytuje:

- zapnutí a vypnutí čerpadla;
- výběr provozního režimu;
- volbu výkonu Smart / Silent / Powerful (`select.pw030_rezim_vykonu`);
- nastavení cílové teploty TUV, vytápění a chlazení;
- limity frekvence pro tiché vytápění (R30) a chlazení (R32), s krokem 1 Hz a rozsahem 30–120 Hz;
- servisní výběr parametru, zobrazení jeho hodnoty a zadání nové hodnoty;
- binární stavy včetně chodu kompresoru, oběhového čerpadla a odmrazování.

Zápis se zpřístupní zapnutím **Povolit zápis**. Šablony a ovladače používají hodnoty ze společného seznamu Modbus. Dostupné parametry, rozsahy a příklad servisního ovládání jsou v [mapě registrů](docs/modbus-registers.md). Před prvním zápisem pomůže porovnání s panelem čerpadla, protože dostupná mapa je pro PW58332.

## Společné ovládání čerpadla

Balíček vytváří `climate.pw030` pro standardní kartu termostatu. Nabízí vypnutí, vytápění a chlazení, nastavení cílové teploty a předvolby Smart / Silent / Powerful. Zpřístupní se zapnutím **PW030 Povolit zápis**. Používá stejná čtení a skript zápisu jako ostatní ovladače.

```yaml
type: thermostat
entity: climate.pw030
name: Tepelné čerpadlo
features:
  - type: climate-hvac-modes
  - type: climate-preset-modes
    style: dropdown
```

Zobrazená aktuální teplota je teplota vratné vody, nikoli místnosti. V režimu pouze TUV se zobrazuje teplota zásobníku a nastavuje jeho cílová teplota; climate tento režim zobrazuje jako vytápění. Plný výběr včetně TUV a kombinovaných režimů poskytuje `select.pw030_nastavit_rezim`. Přepnutí heat/cool v climate zachová TUV u kombinovaných režimů; z režimu pouze TUV přejde na samostatné vytápění nebo chlazení.

Karta má společný rozsah 7–60 °C. Skript zápisu kontroluje rozsah podle režimu: vytápění 15–50 °C, chlazení 7–30 °C, TUV 28–60 °C. Hodnota mimo rozsah daného režimu se nezapíše. V kombinovaném režimu karta nastavuje teplotu vytápění nebo chlazení; teplota TUV má vlastní ovladač. Vypnutí přes climate vypíná celé čerpadlo včetně ohřevu TUV.

## Reference

- [Mapa registrů, převody a ruční zápis](docs/modbus-registers.md)
- [Podklady k desce a zapojení](docs/modbus-port-research.md)
- [Zdroje a původní podklady](docs/podklady/README.md)

## Licence

Vlastní YAML konfigurace a autorský text dokumentace jsou licencované pod [GNU GPL v3.0](LICENSE) (`GPL-3.0-only`). Licence umožňuje i komerční použití; při distribuci upravených verzí platí podmínky GPL včetně poskytnutí odpovídajících zdrojů příjemcům.

GPL tohoto projektu se nevztahuje na podklady třetích stran: návody výrobce, původní protokoly, schémata ani obrázky. Práva k nim náleží jejich příslušným držitelům. Původní podklady jsou přiložené v `docs/podklady/`. Vlastní přehled zdrojů `docs/podklady/README.md` je součástí dokumentace projektu pod GPL.
