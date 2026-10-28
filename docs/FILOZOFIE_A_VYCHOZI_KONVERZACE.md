# Access Guardian — filozofický základ a záznam výchozí konverzace

Datum: 28. října 2026  
Stav: výchozí zadání před technickým auditem a výběrem implementace.

Tento dokument zachycuje obsah a vývoj úvodní konverzace o projektu Access Guardian. Je to redakčně uspořádaný záznam požadavků, argumentů a upřesnění, nikoliv doslovný přepis. Rozlišuje závazné směrování projektu od návrhů, které je teprve nutné ověřit. Neprokazuje bezpečnostní stav konkrétní firmy a neznamená, že je popsané řešení již implementované.

## 1. Proč projekt vzniká

Výchozím problémem jsou historicky příliš široká oprávnění k firemním datům, zejména přístupy přes Everyone a obdobně široké skupiny. S příchodem AI agentů roste význam těchto starých nedostatků: nástroj jednající pod účtem zaměstnance může rychle prohledávat, číst, měnit a mazat vše, co daná identita technicky dovoluje.

Běžný zaměstnanec chce především ulehčit práci. Nemusí chápat rozdíl mezi chatbotem, lokálním agentem, konektorem a delegovaným oprávněním. Může ochotně potvrdit propojení, poskytnout přihlašovací prostředek nebo spustit nástroj ve své již přihlášené relaci.

Cílem IT je chránit firmu i při takovém jednání. Ochrana nesmí záviset pouze na odbornosti uživatele, školení, dobré vůli ani instrukci zapsané do promptu.

Access Guardian má pomoci odpovědět:

> Co může běžný zaměstnanec zpřístupnit nebo nechat provést nástrojem, který si spustí či připojí, které technické kontroly jej omezí a zda se o rizikovém jednání dozvíme?

## 2. Základní rozhodnutí uživatele

1. První fáze je hluboký audit současného nastavení, který dodá argumenty pro další postup.
2. Na první užitečný výsledek se nesmí čekat tři měsíce.
3. Maximálně využít existující nástroje, standardní rozhraní a zavedené procesy.
4. Vlastní vývoj přidávat pouze pro ověřené mezery; nestavět automaticky vlastní SIEM, databázi logů a model strojového učení.
5. Výsledný model přístupů má používat skupiny a role, nikoliv rozsáhlé seznamy přímých oprávnění jednotlivých uživatelů.
6. Projekt zahrnuje souborové servery, identity, databáze i relevantní části klientských stanic.
7. Prostředí zahrnuje přibližně 200 klientských stanic. Tento počet je vstupní kontext, nikoliv měření objemu událostí.
8. Platforma je otevřená: Windows i Linux podle výhod a potřeb zvolených nástrojů.
9. SQL Server Express není zamýšlená varianta.
10. Telemetrie, simulace změn a behaviorální analýza jsou navazující možnosti, nikoliv podmínka prvního auditu.

## 3. Vývoj úvah v konverzaci

### 3.1 Od Everyone k oprávněním podloženým provozem

Původní úvaha byla několik měsíců sledovat skutečné přístupy k souborům a složkám, poté odstranit široká oprávnění a zachovat potřebné přístupy.

Inspirací je přístup používaný u Business Central telemetry: pozorovat činnost a podle ní navrhovat oprávnění k tabulkám tak, aby zpřísnění neomezilo práci.

Upřesnění: pozorované použití není automaticky oprávněná pracovní potřeba. Nepozorované použití není důkaz nepotřebnosti. Zohlednit uzávěrky, zastupování, havarijní situace, servisní účty a vlastníka dat.

Před změnou se má návrh porovnat se zaznamenaným provozem: které operace by byl zablokoval? Tato simulace dokládá pouze pokrytí sledovaných operací, nikoliv všech budoucích situací.

### 3.2 Skupiny a role

Na souborových serverech přidělovat práva skupinám AD. Oddělit například čtení od změn a mazání. Členství může vycházet z pracovních rolí a vnořených skupin.

Databáze mají vlastní role a oprávnění k objektům a operacím. Aplikační oprávnění Business Central jsou další samostatná vrstva; nesmí se mechanicky zaměňovat s databázovými právy.

Pokud aplikace používá společný databázový účet, samotný databázový audit neodliší jednotlivé zaměstnance. Potřebujeme korelaci s aplikační telemetrií.

### 3.3 Riziko nepřímých cest

V konverzaci zazněl uživatelský popis incidentu: AI nasazující opravenou betaverzi měla zasáhnout i primární data a hledat další přihlašovací prostředky. Tento popis nebyl ověřen forenzními záznamy; je motivací a scénářem hrozby, ne doloženým nálezem.

Podstatná otázka je, zda pracovní prostředí zpřístupňuje cestu do produkce. Beta účet nemusí mít přímý zápis, ale může číst konfiguraci s jinou identitou, která zapisovat smí.

Audit proto nemá končit seznamem ACL. Má posuzovat dosažitelné kombinace identit, dostupných přihlašovacích prostředků, síťových cest, nástrojů a připojených úložišť.

### 3.4 Neúspěšné i úspěšné operace

Odlišit:

- chybné přihlášení;
- úspěšné přihlášení následované zamítnutím přístupu;
- technicky povolenou, ale neobvyklou nebo nebezpečnou operaci.

Zajímají nás opakovaná selhání, střídání účtů a cílů, pomalé zkoušení během delší doby, úspěch po sérii chyb, hromadné čtení a mazání i přístup z bety do produkce.

Nemusíme určit, že jednala AI. Potřebujeme detekovat rizikové jednání člověka, programu, malware či agenta. Ke spolehlivému určení procesu může být nutná telemetrie zdrojové stanice.

### 3.5 Využít Microsoft a existující platformy

Před vlastním vývojem prověřit doménové politiky, nativní audit, Windows Event Forwarding, dostupné licence a nasazené produkty Microsoftu.

AD Account Lockout Policy již poskytuje práh chybných přihlášení, reset počítadla a dobu zamčení. Zamčení není totéž jako administrativní deaktivace účtu. Neřeší samo zamítnuté otevření složky, všechny formy rozptýlených útoků ani automatické ukončení již existujících relací.

MFA je užitečná vrstva, ale neomezuje samo rozsah činností v již autorizované relaci ani všechny cesty přes klíče a tokeny.

Defender for Identity, Sentinel a Wazuh byly probírány jako kandidáti, nikoliv jako schválená sestava. Je nutné ověřit licence, pokrytí, provozní náročnost a náklady.

### 3.6 Strojové učení

ML může doplnit pravidla o profil chování účtů, zařízení a pracovních skupin. Příklady: neobvyklý počet souborů, nové cíle, změna denní doby, nový proces nebo neobvyklé zápisy.

Obvyklé chování nemusí být bezpečné a odchylka nemusí být útok. Model nemá automaticky přidělovat oprávnění podle používání. Začít vysvětlitelnými upozorněními a ověřovat falešné poplachy.

První audit ML nepotřebuje. Nejdříve prověřit již dostupnou behaviorální analytiku existujících platforem.

### 3.7 Databáze a platforma jsou otevřené

Diskutovány byly MSSQL Standard, PostgreSQL, Linux, Python a samostatná auditní VM. Žádná z těchto variant není konečné rozhodnutí.

Poslední a určující upřesnění uživatele: nejdříve zvolit existující nástroje a využít jejich standardní sběr, úložiště, pravidla a analytiku. Vlastní databáze a Python nejsou povinné součásti.

## 4. První fáze: audit aktuálního stavu

### Rozsah

| Oblast | Kontroly |
|---|---|
| AD a identity | Účty, aktivita a stav, skupiny a vnoření, delegace, administrátorské a servisní identity, přihlašovací politiky |
| Souborové servery | SMB i NTFS oprávnění, dědičnost, široké skupiny, přímé výjimky, nepřečtené části a efektivní přístupy |
| Databáze a aplikace | Role, oprávnění, společné identity, aplikační versus databázový kontext |
| Prostředí a přihlašovací prostředky | Cesty beta–produkce–zálohy, přístupné konfigurace, zacházení s klíči a tokeny bez jejich zveřejnění |
| Monitoring | Zapnuté auditní politiky, rozsah sběru, retence, časová synchronizace, upozornění a mezery |
| Stanice | Základní přehled ochrany, aktualizací, lokálních administrátorů, spouštěných nástrojů a možností připojení AI |
| Obnova | Oddělení přístupů k zálohám, možnost jejich poškození společně s produkcí a dostupné důkazy o obnově |

Rozsah skutečně provedených kontrol závisí na dostupných zdrojích a oprávněních. Audit z konfigurace nesmí tvrdit, že otestoval obnovu nebo vynucení pravidla, pokud takový test neproběhl.

### Způsob provedení

Výchozí sběr je pouze pro čtení. Neprovádí zkoušení hesel, automatickou změnu ACL, mazání ani neřízené bezpečnostní testy. Ověření detekcí a vynucení má mít zvlášť vymezený rozsah a řízené testy.

Každá kontrola rozlišuje: splněno, nesplněno, neověřeno, případně nevztahuje se s odůvodněním. Nepřístupná složka nebo chybějící logy nejsou důkaz bezpečnosti.

### Výstup

Každý nález má obsahovat:

- identifikátor a kontrolovanou oblast;
- důkaz, zdroj a čas sběru;
- dotčené účty, skupiny a prostředky v přiměřeném rozsahu;
- konkrétní scénář zneužití nebo chyby;
- dopad na důvěrnost, integritu a dostupnost;
- závažnost, míru jistoty a omezení ověření;
- doporučení, prioritu a možnost nápravy existujícími nástroji.

Manažerský souhrn má vysvětlit, co hrozí, jak to víme, co napravit konfigurací a kde je nutná investice nebo další měření. Jedno celkové procento bezpečnosti nesmí nahrazovat důkazy a pokrytí.

První hodnota vznikne po dokončení počátečního sběru a vyhodnocení. Délku nelze slíbit bez inventury a pilotu; tříměsíční provoz však není předpoklad.

## 5. Ochrana před neuváženým použitím AI zaměstnanci

### Dvě hlavní cesty

1. Externí aplikace nebo konektor získá delegovaný přístup. Prověřit centrální správu aplikací, souhlasů, oprávnění a životního cyklu.
2. Lokální agent pracuje v relaci uživatele. Prověřit správu stanic, spouštění programů, rozšíření, skriptů, izolaci a odchozí komunikaci.

Odebrání lokálního administrátora samo nepokrývá vše. Ani read-only přístup není bez rizika, protože umožňuje čtení a případný únik dat.

### Zásady

- Minimální potřebná oprávnění a dostupné nástroje.
- Vlastník každého schváleného agenta nebo automatizace.
- Vlastní identita nebo omezená, dohledatelná delegace; žádné nekontrolované sdílení osobních hesel.
- Oddělené běžné a privilegované pracovní kontexty.
- Technické oddělení testu, produkce a záloh.
- Vysoce dopadové operace pod nezávislou kontrolou, kterou agent nemůže sám schválit.
- Obsah dokumentu, e-mailu a výstupu nástroje je nedůvěryhodný vstup, nikoliv oprávnění měnit pravidla.
- Centrální záznam operací mimo kontrolu sledovaného subjektu.
- Ověřené odvolání přístupu včetně relevantních tokenů a relací.
- Použitelná schválená cesta práce s AI.
- Školení doplňuje technické kontroly, nenahrazuje je.

Neexistuje slib absolutní ochrany ani univerzální rozlišení člověka a AI pouze z file-serverových logů.

## 6. Navazující provozní telemetrie

Po auditu lze zahájit sledování skutečného používání. Původně diskutované tři měsíce jsou možné pozorovací období, ne pevná univerzální lhůta.

Sledovat identitu, zdroj, cíl, čas, operaci a výsledek. Rozlišovat vypsání adresáře, čtení, zápis, vytvoření, přejmenování, mazání a změny práv podle možností zdroje.

Oddělit servisní provoz, antivir, indexaci a zálohy. Zachovat i ojedinělé operace. Změny ACL a členství sledovat historicky, aby šlo interpretovat stav v čase.

Následný proces: návrh skupin a rolí → simulace vůči dostupné historii → potvrzení vlastníkem dat → postupné nasazení → sledování dopadu → možnost návratu.

## 7. Data, výkon a umístění

Preferovat centrální provoz na vyhrazené infrastruktuře. Nativní sběr nebo existující agent má přednost před vlastním doplňkem. Podrobné kontroly nejdříve na rizikových stanicích, nikoliv podmínit první výstup hlubokým skenem všech zařízení.

Kapacitu odvodit od událostí za čas, jejich velikosti, indexů, retence, archivu a záloh; samotných 200 stanic k dimenzování nestačí.

Pokud bude vlastní úložiště potřebné:

- odlišit původní události, evidence identit a objektů, historické konfigurace, souhrny a nálezy;
- dávkové a přírůstkové zpracování, deduplikace a evidence pokroku;
- vhodné časové oddíly, indexy a řízená retence podle měření;
- nezakládat samostatnou tabulku pro každou stanici;
- ML pracuje zejména s příznaky a souhrny, vyšetřování s podrobnostmi;
- nepřevádět automaticky všechny logy do jedné vlastní SQL databáze;
- sledovat úplnost sběru, zpoždění, výpadky a duplicitní doručení.

Sbírat převážně metadata. Obsahy dokumentů ani tajné hodnoty nejsou výchozí potřebou. I cesty, identity a provozní metadata jsou citlivá. Zajistit řízení přístupů, šifrování, dobu uchování a zálohy oddělené od sledovaných identit.

Do tohoto veřejného repozitáře nepatří reálné interní reporty, inventáře, logy, názvy citlivých cest, přihlašovací údaje ani produkční data.

## 8. Kandidáti k prověření, nikoliv schválený stack

| Potřeba | Kandidát |
|---|---|
| Účty, skupiny, politiky | Nativní AD/Windows rozhraní a PowerShell |
| Inventura NTFS a skupin | MAAT, File-Server-Audit |
| Pomocné operace nad NTFS | NTFSSecurity |
| Přeposílání událostí | Windows Event Forwarding / Event Collector |
| Detekce identit | Microsoft Defender for Identity |
| Centrální bezpečnostní dohled | Existující nasazená platforma, případně Wazuh nebo Microsoft Sentinel |
| Databázový audit | Nativní možnosti konkrétní databáze, například SQL Server Audit |
| Řízené přístupy automatizací | Prověřit existující identity/PAM řešení; v konverzaci zazněl také Teleport |

Rešerše byla orientační, podle veřejných dokumentací a repozitářů. Nešlo o bezpečnostní revizi kódu ani výkonový test. Licence, aktivitu projektu, podporované verze a skutečné pokrytí je nutné ověřit při výběru. Není požadováno nasadit všechny kandidáty současně.

Technické upozornění z konverzace: Windows 5145 nezachytí všechna zamítnutí NTFS; 4656 vyžaduje odpovídající audit/SACL. Událost 4663 není univerzální záznam neúspěšných přístupů. Sběr je třeba ověřit reálnými řízenými scénáři.

## 9. Podklady a zdroje

### Bezpečnostní principy

- [OWASP: AI Agent Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/AI_Agent_Security_Cheat_Sheet.html)
- [OWASP: Excessive Agency](https://genai.owasp.org/llmrisk/llm062025-excessive-agency/)
- [OWASP: AI Agent and MCP Security](https://owasp.github.io/DevSecOpsGuideline/2-Process/2-2-Develop/2-2-5-AI-Agent-and-MCP-Security/)
- [Microsoft: Least privilege for AI agents](https://learn.microsoft.com/en-us/security/zero-trust/sfi/least-privilege-for-ai-agents)
- [Microsoft: AI agent shared responsibility model](https://learn.microsoft.com/en-us/azure/security/fundamentals/shared-responsibility-ai-agent)
- [Microsoft: Token theft playbook](https://learn.microsoft.com/en-us/security/operations/token-theft-playbook)

### Nástroje a sběr

- [MAAT](https://github.com/pierrenicolasmartin/MAAT)
- [File-Server-Audit](https://github.com/sean-aura/File-Server-Audit)
- [NTFSSecurity](https://github.com/raandree/NTFSSecurity)
- [Wazuh](https://github.com/wazuh/wazuh)
- [Wazuh: monitoring přístupů ke složkám](https://wazuh.com/blog/how-to-monitor-folder-access-on-windows/) — starší návod, konfiguraci ověřit pro zvolenou verzi.
- [Windows Event Collector](https://learn.microsoft.com/en-us/windows/win32/wec/windows-event-collector)
- [Microsoft Entra Smart Lockout a kontrola on-premises politik](https://learn.microsoft.com/en-us/entra/identity/authentication/howto-password-smart-lockout)
- [SQL Server Audit](https://learn.microsoft.com/en-us/sql/relational-databases/security/auditing/sql-server-audit-database-engine)
- [Microsoft Sentinel UEBA](https://learn.microsoft.com/en-us/azure/sentinel/enable-entity-behavior-analytics)
- [Teleport](https://github.com/gravitational/teleport)

### Diskuze motivující scénáře

- [r/sysadmin: How much do you trust AI?](https://www.reddit.com/r/sysadmin/comments/1vcpu1g/how_much_do_you_trust_ai/)
- [r/sysadmin: Users and AI](https://www.reddit.com/r/sysadmin/comments/1wzttxb/users_and_ai/)
- [r/sysadmin: správa rozšíření a AI agentů](https://www.reddit.com/r/sysadmin/comments/1wbm0tp/as_admins_how_do_you_handle_vs_code_extensions/)
- [Hacker News: An AI agent deleted our production database](https://news.ycombinator.com/item?id=47911524)

Diskuze a uživatelské výpovědi jsou ilustrace scénářů, nikoliv měření četnosti incidentů nebo důkaz stavu naší sítě. Doporučení OWASP a Microsoftu nejsou samy certifikací ani automatickým prohlášením souladu s předpisy.

## 10. Nejbližší rozhodnutí

1. Získat inventuru prostředí, zdrojů logů, současných bezpečnostních nástrojů a licencí.
2. Zmapovat, co už lze zjistit bez nového vývoje.
3. Vybrat minimální sadu existujících nástrojů pro první audit.
4. Ověřit je na vymezeném pilotu: AD, file server a vybrané stanice; databázi zahrnout podle dostupnosti a rozsahu.
5. Dodat první report s důkazy, pokrytím, prioritami a nápravou.
6. Až podle zjištěných mezer rozhodnout o vlastních integracích, delším sběru a behaviorální analýze.

Základním měřítkem úspěchu není počet napsaných řádků kódu, ale doložená znalost rizik a proveditelná cesta ke snížení jejich dopadu.
