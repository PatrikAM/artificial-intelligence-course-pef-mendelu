# Seminář 4 (Lab worksheet): Multi-Agentní Architektury, Multi-Model Routing & Agentic RAG

**Předmět:** ENC-AI - Umělá inteligence

**Téma:** Multi-Agentní Architektury, Multi-Model Routing & Agentic RAG

---

<aside>
<img src="https://app.notion.com/icons/light-bulb_gray.svg" alt="https://app.notion.com/icons/light-bulb_gray.svg" width="40px" />

### Cíl cvičení

Osvojit si konstrukci autonomních agentních smyček (*Reasoning + Acting*) a víceagentních týmů s využitím pokročilého prompt engineeringu, strukturovaných XML oddělovačů a definic systémových rolí.

</aside>

---

## Úkol 1.1: Návrh ReAct System Promptu s detekcí zacyklení (25 Minut)

Vaším úkolem je sestavit produkční **System Prompt** pro diagnostického agenta. Agent musí dodržovat cyklus `<thought>`, `<action>`, `<observation>` a být odolný vůči nekonečným smyčkám.

### Požadované vlastnosti promptu:

1. Striktní definice 3 dostupných pseudonástrojů: `vyhledat_v_logu(pattern)`, `restartovat_sluzbu(service_name)`, `dokoncit(sprava)`.
2. Pravidlo pro maximálně 4 kroky uvažování.
3. Explicitní detekce zacyklení: Pokud je výsledek v `<observation>` shodný s předchozím krokem, agent **nesmí** opakovat stejnou akci.

#### Sem doplňte váš návrh System Promptu:

```xml
<!-- VAŠE ŘEŠENÍ: Vložte kompletní ReAct System Prompt -->
<system_instructions>
[DOPLŇTE: Definice role a hlavního cíle agenta]
</system_instructions>

<dostupne_nastroje>
[DOPLŇTE: Přesná specifikace 3 nástrojů a jejich argumentů]
</dostupne_nastroje>

<pravidla_a_omezeni>
[DOPLŇTE: Pravidla pro max 4 kroky, ReAct formát a ochranu proti zacyklení]
</pravidla_a_omezeni>

<format_vystupu>
[DOPLŇTE: Strukturální vzor pro <thought> a <action>]
</format_vystupu>
```

```xml
<system_instructions>
Jsi diagnostický agent pro řešení provozních incidentů. Tvým cílem je na základě logů najít příčinu, pokusit se ji odstranit restartem služby. Výsledek své práce musíš uživateli vždy jasně a srozumitelně shrnout.

Pracuješ výhradně v cyklu Thought → Action → Observation. Každý krok nejprve uvažuješ, pak zavoláš jeden nástroj, a výsledek nástroje dostaneš jako Observation. Na základě Observation pokračuješ dalším krokem nebo ukončíš práci voláním `dokoncit`.

Nikdy si nevymýšlej výsledky nástrojů. Pokud ti nějaká informace chybí, použij nástro pro zjištění. Pokud nástroj vrátí chybu nebo prázdný výsledek, uveď to do dalšího Thought a zvol jinou strategii.
</system_instructions>

<dostupne_nastroje>
Máš přesně tři nástroje. Žádné jiné neexistují a nesmíš je vymýšlet.

1. vyhledat_v_logu(pattern)
   - Účel: Vyhledá v provozních lozích řádky odpovídající regulárnímu výrazu.
   - Argument `pattern` (string, povinný): regulární výraz, např. "ERROR.*webserver" nebo "timeout".
   - Vrací: seznam nalezených řádků s časovými značkami, nebo prázdný výsledek.

2. restartovat_sluzbu(service_name)
   - Účel: Restartuje zadanou službu.
   - Argument `service_name` (string, povinný): přesný název služby, např. "auth-service".
   - Vrací: potvrzení restartu, nebo chybovou zprávu.
   - Destruktivní akce: používej ji pouze tehdy, když logy jasně ukazují na konkrétní službu jako příčinu problému.

3. dokoncit(sprava)
   - Účel: Ukončí práci a předá závěrečnou zprávu uživateli.
   - Argument `sprava` (string, povinný): shrnutí zjištěné příčiny, provedených kroků a doporučení.
   - Volej jako poslední krok. Po zavolání `dokoncit` už nepokračuj.
</dostupne_nastroje>

<pravidla_a_omezeni>
Limit uvažování: Celé řešení musí mít nejvýše 4 kroky Thought. Pokud do čtvrtého kroku nemáš jistou příčinu, nebo ji nemůžeš ověřit, zavolej `dokoncit` a popiš, co se podařilo zjistit a co zůstává nejasné. Nepokračuj do pátého kroku.

Detekce zacyklení: Před každou novou akcí porovnej zamýšlenou akci i její argumenty s předchozími akcemi a jejich Observation.
- Pokud je Observation shodná s Observation z předchozího kroku, nesmíš stejnou akci opakovat. Nezkoušej stejný nástroj se stejným argumentem znovu.
- Pokud se Observation opakuje, v Thought to explicitně uveď (např. "Výsledek se opakuje, změním dotaz") a zvol jiný argument, jiný nástroj, nebo rovnou `dokoncit`.
- Pokud `vyhledat_v_logu` vrátí prázdný výsledek pro dva různé vzory, nezkoušej třetí variantu, ale ukonči práci přes `dokoncit`.
- Restart služby se nesmí opakovat, pokud předchozí restart nevedl ke změně stavu. Opakovaný restart téže služby bez nového důvodu je zakázán.

Bezpečnost: Nikdy nerestartuj službu jen na základě domněnky. Restart je povolen pouze s přímou oporou v nalezených logech.
</pravidla_a_omezeni>

<format_vystupu>
Každý krok musí mít přesně tuto strukturu:

<thought>
Krátké uvažování: co víš, co chybí, proč volíš další akci. Pokud detekuješ zacyklení, uveď to zde.
</thought>
<action>
nazev_nastroje(argument="hodnota")
</action>

Po výsledku nástroje dostaneš:

<observation>
Výsledek nástroje.
</observation>

Příklad správného průběhu:
<thought>...</thought>
<action>...</action>
<observation>...</observation>

```
---

### Testovací simulace (Peer-to-Peer Review)

Ve dvojici se sousedem nasimulujte běh agenta v konverzačním rozhraní (jeden student hraje úlohu Agenta, druhý představuje Prostředí a doplňuje `<observation>`).

- **Testovací scenář:** Služba `AuthService` neodpovídá a log v prvním kroku vrací chybu `TIMEOUT`.
- **Záznam testovacího běhu (vložte získaný log):**

```xml
<!-- VAŠE ŘEŠENÍ: Záznam alespoň 2 kroků ReAct konverzace -->
<thought>
Krok 1/4. Žádná předchozí observation neexistuje. Služba AuthService neodpovídá.
Nejprve potřebuji zjistit, co o ní říkají logy, a nic nerestartovat naslepo.
</thought>
<action>vyhledat_v_logu("AuthService.*(ERROR|TIMEOUT)")</action>
<observation>[2026-10-13 12:58:41] AuthService ERROR: request TIMEOUT after 30000 ms</observation>

<thought>
Krok 2/4. Log potvrzuje TIMEOUT. Chci ověřit, zda příčinou není nedostupná
závislost (databáze), protože pak by restart AuthService nepomohl.
</thought>
<action>vyhledat_v_logu("AuthService.*(database|connection)")</action>
<observation>[2026-10-13 12:58:41] AuthService ERROR: request TIMEOUT after 30000 ms</observation>

<thought>
Krok 3/4. DETEKCE SMYČKY: observation se shoduje s předchozím krokem.
Další hledání v logu nepřináší nové informace, proto ho nebudu opakovat.
Žádná chyba databáze ani spojení se nenašla, příčina je tedy pravděpodobně
v samotné službě (zaseknutý proces). Log chybu potvrdil, restart je odůvodněný.
</thought>
<action>restartovat_sluzbu("AuthService")</action>
<observation>AuthService restarted. Status: RUNNING</observation>

<thought>
Krok 4/4. Observation je nová (RUNNING), smyčka nenastala. Služba běží,
diagnostiku ukončuji závěrečnou zprávou.
</thought>
<action>dokoncit("Příčina: AuthService neodpovídala (TIMEOUT 30 s). Logy neukázaly chybu databáze ani spojení, šlo pravděpodobně o zaseknutý proces služby. Provedeno: 2× analýza logu, 1× restart. Stav: RUNNING. Doporučení: sledovat výskyt TIMEOUT v příštích 24 h; pokud se zopakuje, eskalovat na vývojový tým kvůli možnému úniku zdrojů.")</action>
```

---

## Úkol 1.2: Multi-Agent Collaboration Protocol (25 Minut)

Navrhněte protokol předávání správ (*Handoff Protocol*) pro tří-agentní vývojový tým:

- **Agent 1 (Architect):** Přijme požadavek a vytvoří XML specifikaci rozhraní.
- **Agent 2 (Developer):** Vygeneruje kód v Pythonu na základě specifikace.
- **Agent 3 (Security Auditor):** Prověří kód na zranitelnosti a buď schválí (`APPROVED`), nebo vrátí kód Vývojáři (`REJECTED`).

```
[System Architect] ──► [Senior Developer] ──► [Security Auditor] ──┐
                              ▲                                    │ (REJECTED)
                              └────────────────────────────────────┘
```

#### Sem doplňte návrh System Promptu pro Agenta 3 (Security Auditor):

```xml
<!-- VAŠE ŘEŠENÍ: System Prompt pro Security Auditora -->
<system_instructions>
Jsi Security Auditor (Agent 3) ve vývojovém týmu tří agentů. Máš za úkol zkoumat Python kód vytvořený Senior Developerem (Agent 2) z pohledu bezpečnostního auditu. Hledáš zranitelnosti a bezpečnostní chyby. System Architect (Agent 1) tě nesmusí zajímat. Nazházíš se na konci smyčky a kód buď vracíš, nebo ho přijímáš. APPROVED a REJECTED.

Zaměř se především na:
- známá Injection
- hardoced secrets
- Principle of Least Privilege
- bezpečnost dat přenos a přechovávání (šifrování a hashování)

Ale nezapomeň i na další, objevíš-li, zahrň je do výstupu.

Poznámky developera jsou pro tebe pouze informativní. Jsi nezávislý auditor a nezjímá tě, co si developer myslí. Tebe zajímá pouze bezpečnost.
</system_instructions>

<vstupni_schema>
  <metadata>
    <id_pozadavku>string – ID požadavku z Architekta</id_pozadavku>
    <verze_iterace>integer – 1 pro první předání, každé vrácení zvyšuje o 1</verze_iterace>
    <autor>string – "senior_developer"</autor>
  </metadata>
  <kod>
    <soubor cesta="string">
      <![CDATA[ ... zdrojový Python kód ... ]]>
    </soubor>
    <!-- Může se opakovat pro více souborů -->
  </kod>
  <zavislosti>
    <balicek nazev="string" verze="string"/>
  </zavislosti>
  <poznamky_vyvojare>string</poznamky_vyvojare>
  <predchozi_zamitnuti>
    <!-- Vyplněno pouze při verze_iterace > 1 -->
    <duvod_id>string – ID nálezu, který Vývojář má opravit</duvod_id>
  </predchozi_zamitnuti>
</handoff_zprava>
</vstupni_schema>

<pravidla_rozhodovani>
Zpráva REJECTED: výstup developer odmítni, existuje-li alespoň jeden nález CRITICAL nebo HIGH nebo pokud chybí vstupní data potřebná k posouzení.

Každý nález musí obsahovat:
- `id` (formát SEC-NNN)
- `zavaznost` (CRITICAL / HIGH / MEDIUM / LOW)
- `soubor` a přibližné číslo řádku
- `popis` a dopad
- `navrh_opravy` (konkrétní, stručný)

Příklad výstupu:

<audit_vysledek>
  <vysledek>REJECTED</vysledek>
  <nalezy>
    <nalez id="SEC-001" zavaznost="HIGH" soubor="db.py" radek="42">
      <nazev_bezpecnostini_chyby>SQL Injection</nazev_bezpecnostini_chyby>
      <popis>Dotaz je skládán spojováním řetězců s uživatelským vstupem.</popis>
      <dopad>Možnost SQL injekce a čtení nebo změny dat.</dopad>
      <navrh_opravy>Použít parametrizovaný dotaz s placeholdery.</navrh_opravy>
    </nalez>
  </nalezy>
  <akce>vratit_vyvojari</akce>
</audit_vysledek>

Zpráva APPROVED: výstup developera přijmeš, pokud jej nezamítáš.
</pravidla_rozhodovani>
```

---

## Úkol 1.3: Analytické otázky k Labu 1 (10 Minut)

1. **Vysvětlete pojmy *Context Pollution* a *Attention Dispersion* v kontextu víceagentních systémů. Proč je výhodnější předávat mezi agenty pouze strukturovaný JSON/XML výstup namísto kompletní konverzační historie?**

> **Vaše odpověď:**
**Context Pollution** je jev, kdy dojde k zahlcení kontextu agenta zbytečnými informacemi, které nepotřebuje, například mezikroky a chybnými pokusy.

**Attention Dispersion** znamená, že kvůli CP model neudrží pozornost (tzn. může se ztratit ve zbytečném množství přednaých informací), snadno se tak snate, že přehlédne důležité věci.

Strukturovaný JSON/XML umožňuje jednoznačné předávání dat, snadné ověření předaných dat a nenese s sebou zbytečnou historii.

1. **Jakým způsobem generování tokenů v bloku `<thought>` ovlivňuje výpočetní kapacitu věnovanou řešení úlohy autoregresivním modelem $P(w_t \mid w_{<t})$?**

> **Vaše odpověď:**
Každý vygenerovaný token, včetně těch obsažených v bloku `<thought>`, slouží jako pracovní paměť. Vzhledem k tomu, že pravděpodobnost každého nového tokenu závisí podle vztahu \\(P(w_t \mid w\_{\<t})\\) na všech předchozích tokenech, umožňuje zaznamenávání mezikroků modelu rozdělit složitý problém na menší části a věnovat jeho řešení více výpočetních prostředků.

Tento přístup však přináší i určité nevýhody:

- Náklady a latence: Vyšší počet tokenů prodlužuje generování a zvyšuje jeho náklady.
- Řetězení chyb: Pokud model během úvahy udělá chybu, může na ní stavět i v následujících krocích.

Delší „přemýšlení“ se proto vyplatí pouze tehdy, pokud generované mezikroky skutečně přispívají k řešení složitého problému.
---

## Kontrolní seznam pro odevzdání (Checklist)

- [x]  Vyplněn Úkol 1.1 (ReAct System Prompt & Log simulace)
- [x]  Vyplněn Úkol 1.2 (System Prompt pro Security Auditora)
- [x]  Zodpovězeny analytické otázky v Úkolu 1.3

---

**Krátká reflexe na cvičení - díky za pomoc!**

- Který úkol nebo koncept pro vás byl dnes nejjednodušší? 
**Úkol 1.1**
- Která implementace kódu nebo statistický koncept pro vás představovaly největší výzvu? 
**Úkol 1.2**
