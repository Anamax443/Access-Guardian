# Access Guardian

Audit přístupových oprávnění a bezpečnostních konfigurací, dohled nad přístupy a vyhodnocování rizikového chování uživatelů i automatizací.

## Filozofický základ

[Filozofie projektu a záznam výchozí konverzace](docs/FILOZOFIE_A_VYCHOZI_KONVERZACE.md)

Dokument zachycuje motivaci, požadavky, jejich vývoj, rozsah první fáze, otevřené otázky a odkazy na podklady. Jde o uspořádaný záznam konverzace, nikoliv doslovný přepis.

## Určující zásady

- První výstup je hluboký audit současného stavu s důkazy a prioritami nápravy. Nevyžaduje tříměsíční čekání.
- Maximálně využít existující nástroje, standardní rozhraní a procesy. Vlastní kód pouze pro ověřené mezery.
- Chránit firmu i při neuváženém použití AI zaměstnanci; omezení musí vynucovat prostředí.
- Zkoumat přímá i nepřímá oprávnění a cesty mezi stanicemi, identitami, daty, produkcí a zálohami.
- Oddělit audit konfigurace, provozní telemetrii a případné strojové učení.
- Neověřené oblasti uvádět jako neověřené. Žádný nástroj ani databáze zatím nejsou schválenou architekturou.

## Stav

Definice projektu před pilotem a výběrem nástrojů. Tento repozitář zatím nedodává implementovaný auditní systém.

## Data

Repozitář je veřejný. Nezveřejňovat zde interní inventáře, skutečné auditní reporty a logy, přihlašovací údaje ani produkční data.
