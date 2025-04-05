# Analýza projektu: Simulácia križovatky so semaformi

## 1. Základná charakteristika systému

Simulácia sa zameriava na riadenie dopravy na klasickej križovatke v tvare **+ (4 cesty)**. Zohľadňuje len **osobné motorové vozidlá**, bez chodcov, cyklistov, priechodov atď. Uvažujeme bez otáčania vozidiel v križovatke.

Simulácia počíta so štyrmi smermi (vetvami): **Sever, Juh, Východ, Západ**. Všetky štyri smery majú rovnaký počet jazdných pruhov – tento počet zadáva používateľ a musí byť v rozmedzí **1 až 3**:

- **1 pruh**: univerzálny – priamo, vpravo, vľavo.
- **2 pruhy**: ľavý pruh – iba vľavo, pravý pruh – priamo a vpravo.
```plaintext
______________________________________
|                 |                  |
|                 |        Ʌ         |
|     <--¬        |        |-->      |
|        |        |        |         |
|        |        |        |         |
|        |        |        |         |
|                 |                  |
```

- **3 pruhy**: ľavý pruh – vľavo, stredný – priamo, pravý – vpravo.
```plaintext
_________________________________________________________
|                 |                  |                  |
|     <--¬        |        Ʌ         |         Г-->     |
|        |        |        |         |         |        |
|        |        |        |         |         |        |
|        |        |        |         |         |        |
|                 |                  |                  |
```

## 2. Typy semaforov

Používajú sa **dva typy semaforov**, pričom pre celú križovatku je vždy zvolený rovnaký typ. Používateľ si ich môže vybrať **len v prípade, že sú nastavené 2 pruhy**.

### a) Semafory s plnými signálmi
- Môžu byť použité len pri **1 alebo 2 pruhoch**.
- **Zelená znamená voľno pre všetky smery (priamo, vľavo, vpravo)**.
- **Existujú kolízne situácie** – vozidlá odbočujúce vľavo musia **dávať prednosť** protiidúcim vozidlám idúcim priamo alebo vpravo.
- Vozidlá pri odbočení vľavo **vchádzajú do križovatky a čakajú**, kým bude možné odbočiť.
- **Riadiaca logika**:
  - Zelenú majú vždy **naprotivné smery** (napr. Sever a Juh).
  - Zvyšné smery majú zároveň červenú.
  - **Oranžová** trvá vždy **3 sekundy** (rovnaké platí aj pre smerové signály).
- Používateľ môže nastaviť **trvanie zelenej** pre každý pár naprotivných smerov, ktoré musia byť **rovnaké** (napr. ak Sever má 20 s, Juh musí mať tiež 20 s).

### b) Semafory so smerovými signálmi
- Môžu byť použité pri **2 alebo 3 pruhoch**.
- Vozidlá majú samostatné signály pre **ľavé, priame a pravé odbočenie**.
- **Zelená šípka znamená voľno bez nutnosti dávať prednosť**.
- Pri 2 pruhoch:
  - Ľavý pruh má signál iba **vľavo**.
  - Pravý pruh má signál pre **priamo + vpravo**.

#### Povolené kombinácie (fázy):

1. **Jeden smer (napr. Sever)** má signál **Voľno** pre všetky smery (vľavo, priamo, vpravo).  
   → Ostatné smery majú **červenú**.

2. **Naprotivné smery (napr. Sever a Juh)** majú **Voľno** iba pre **ľavé odbočenie**.  
   → **Ak sú 3 pruhy**, smery **do ktorých sa odbočuje vľavo** (napr. Východ a Západ) majú súčasne signál **voľno vpravo** – nedochádza ku križovaniu vozidiel.  
   → Všetky ostatné smery majú **červenú**.

3. **Naprotivné smery (napr. Východ a Západ)** majú Voľno pre **priamy smer a pravé odbočenie**.  
   → Ľavé odbočenie z týchto smerov má v tejto fáze **červenú**.  
   → Ostatné smery majú **červenú**.

Používateľ môže:
- Nastaviť **poradie** týchto kombinácií.
- Zvoliť **trvanie (v sekundách)** pre každú kombináciu samostatne.

> **Dôležité obmedzenie:**  
Nie je možné kombinovať rôzne typy semaforov na jednej križovatke.  
Ak je v jednom smere použitý **semafor s plnými signálmi**, potom **musí byť rovnaký typ** použitý aj v ostatných smeroch.  
Rovnako to platí pre **smerové signály**.

## 3. Zadanie dopravnej záťaže

Používateľ môže pri simulácii definovať **dopravnú záťaž** pre každý zo štyroch smerov (Sever, Juh, Východ, Západ) ako:
- **Počet áut za minútu pre každý pruh** (napr. 10 áut/min) – napr. pri 3 pruhoch to znamená, že do každého pruhu bude prichádzať 10 áut za minútu.

## 4. Frontend a ovládanie

Frontend obsahuje dve hlavné časti:

### a) Ovládacia stránka (nastavenia)
- Výber počtu pruhov (1 až 3).
- Výber typu semaforov (ak sú 2 pruhy – používateľ môže vybrať).
- Zadanie trvania zelenej pre každý smer (pri plných signáloch).
- Definovanie a zoradenie fáz + ich trvanie (pri smerových signáloch).
- Zadanie dopravnej záťaže (autá/min pre každý smer).
- Tlačidlo na **spustenie simulácie**.

### b) Simulačná stránka
- Vizualizácia križovatky s jednoduchými animovanými semaformi.
- Vizualizácia áut prichádzajúcich zo všetkých smerov podľa zadanej záťaže.
- Semafory zobrazujú aktuálne farby (červená, oranžová, zelená).
- Autá reagujú na signály: zastavujú, rozbiehajú sa, odbočujú podľa typu semaforu.

## 5. Komunikácia frontend – backend – simulácia

```plaintext
+-----------------+        +------------------+        +------------------------+
|  Ovládacia UI   | <----> |  Backend logika  | <----> |  Simulácia (zobrazenie)|
+-----------------+        +------------------+        +------------------------+
        |                         |                             |
        | Nastavenia používateľa  |  Riadiace cykly semaforov   |
        | (časy, počet pruhov,    |  + časovanie + intenzita    |
        | typ, fázy, záťaž)       |                             |     
