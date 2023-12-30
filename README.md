# DB-Projekt

# Praktická maturitní zkouška

**Střední průmyslová škola elektrotechnická, Praha 2, Ječná 30**
**Školní rok 2023/2024**
---
Jméno a příjimeni: Tomáš Novotný
---
Třída: C4b
---

# *Úvod*
Problém jsem se rozhodl řešit v MSSQL Server a jako návrhové prostředí jsem využil Microsoft SQL Server Management Studio.

# *E-R model*
Konceptuální (logický) model databáze se nachází v /img/Logical.png</br></br>
Relační model databáze se nachází v /img/Relational.png</br></br>

## *Konceptuální (logický) model*
![Logical](https://github.com/tomasnovotnyy/DB-Projekt/assets/84340580/c3c37806-ee89-4eb2-af96-30809def2c60)</br></br>
## *Relační model*
![Relational](https://github.com/tomasnovotnyy/DB-Projekt/assets/84340580/4715b435-089a-4103-a0fb-45dfa84a62ba)</br>

# *Entitní integrita*
Každá entita v databázi je vybavena uměle vytvořeným primárním klíčem ID, který slouží jako jedinečný identifikátor a inkrementuje se s přidáním každého nového záznamu. Tento klíč zajišťuje unikátnost a jednoznačnost každé entity, usnadňuje propojování mezi tabulkami a umožňuje efektivní správu dat.

# *Doménová integrita*
*Zakaznik*
- Jmeno: Textový řetězec, maximálně 50 znaků, povinný
- Prijmeni: Textový řetězec, maximálně 50 znaků, povinný
- Email: Textový řetězec, maximálně 100 znaků, unikátní, formát emailu
- Telefon: Textový řetězec, maximálně 20 znaků, povinný, unikátní
- Adresa: Textový řetězec, maximálně 100 znaků, povinný

*Zamestnanec*
- Jmeno: Textový řetězec, maximálně 50 znaků, povinný
- Prijmeni: Textový řetězec, maximálně 50 znaků, povinný
- Telefon: Textový řetězec, maximálně 20 znaků, povinný, unikátní
- Hodnoceni: Celé číslo, omezené mezi 1 až 5
- *Constraint CHK_Telefon_Length, který zaručuje, že délka telefonního čísla v poli 'Telefon' bude přesně 9 znaků. Toto omezení zajistí, že se mohou vkládat a aktualizovat pouze záznamy obsahující telefonní číslo o délce přesně 9 znaků.*

*Produkt*
- Nazev: Textový řetězec, maximálně 50 znaků, povinný
- EAN: Textový řetězec, 8 nebo 13 znaků, povinný
- Cena: Desetinné číslo (max. 10 míst celkem, 2 za desetinnou čárkou), povinný
- Hmotnost: Desetinné číslo (max. 10 míst celkem, 2 za desetinnou čárkou), povinný

*Detail_objednavky*
- Datum_objednani: Datum, povinný
- Dorucovaci_adresa: Textový řetězec, maximálně 100 znaků, povinný
- Poznamka: Textový řetězec, maximálně 100 znaků

*Doruceni*
- Predpokladany_cas_doruceni: Datum a čas, povinný
- Skutecny_cas_doruceni: Datum a čas
- Stav_doruceni: Textový řetězec, maximálně 30 znaků, povinný, přijímá hodnoty 'Doruceno', 'Ceka na doruceni' nebo 'Zruseno'
- *Constraint CHK_NullSkutecnyCas_Stav, který kontroluje, že pokud je stav doručení uveden jako 'Doruceno', musí existovat i skutečný čas doručení, neboli není možné mít objednávku v tomto stavu bez vyplněného skutečného času doručení.*

*Objednavka*
- Mnozstvi: Celé číslo, povinný
- Cena_celkem: Desetinné číslo (max. 10 míst celkem, 2 za desetinnou čárkou)

# *Referenční integrita*
Návrh obsahuje několik cizích klíčů, které jsou uvedeny níže.</br>

*Detail_objednavky*
- Zakaznik_ID: Cizí klíč na tabulku Zakaznik

*Doruceni*
- Detail_objednavky_ID: Cizí klíč na tabulku Detail_objednavky
- Zamestnanec_ID: Cizí klíč na tabulku Zamestnanec

*Objednavka*
- Detail_objednavky_ID: Cizí klíč na tabulku Detail_objednavky
- Produkt_ID: Cizí klíč na tabulku Produkt

# *Indexy* 
- Databáze má pro každou entitu indexy vytvořené pro primární klíče, další indexy jsou uvedeny níže.</br>

```
-- 1. Index pro sloupec Nazev v tabulce Produkt
CREATE INDEX idx_Nazev ON Produkt(Nazev);

-- 2. Index pro sloupec Stav_doruceni v tabulce Doruceni
CREATE INDEX idx_Stav_doruceni ON Doruceni(Stav_doruceni);

-- 3. Index pro sloupec Datum_objednani v tabulce Detail_objednavky
CREATE INDEX idx_Datum_objednani ON Detail_objednavky(Datum_objednani);
```

# *Pohledy*
- Návrh obsahuje pohled, který vypíše jméno a příjmení zákazníka, jeho doručovací adresu objednávky, případnou poznámku k objednávce a skutečný čas doručení.
- V případě, že byla objednávka zrušena, tak se jako skutečný čas doručení zobrazí 'Objednávka zrušena'.
- V případě, že objednávka ještě čeká na doručení, tak se jako skutečný čas doručení zobrazí 'Objednávka čeká na doručení'.
- V případě, že byla již objednávka doručena, tak se jako skutečná čas doručení zobrazí datum ve formátu YYYY-MM-DD HH:MI:SS
```
CREATE VIEW Informace_o_doruceni AS
SELECT
    Z.Jmeno AS 'Jméno zákazníka',
    Z.Prijmeni AS 'Příjmení zákazníka',
    DO.Dorucovaci_adresa AS 'Doručovací adresa',
    DO.Poznamka AS Poznámka,
    CASE
        WHEN D.Stav_doruceni = 'Doruceno' THEN CONVERT(VARCHAR(50), D.Skutecny_cas_doruceni, 120)
        WHEN D.Stav_doruceni = 'Zruseno' THEN 'Objednávka zrušena'
        WHEN D.Stav_doruceni = 'Ceka na doruceni' THEN 'Objednávka čeká na doručení'
    END AS 'Skutečný čas doručení'
FROM Zakaznik Z
JOIN Detail_objednavky DO ON Z.ID = DO.Zakaznik_ID
JOIN Doruceni D ON DO.ID = D.Detail_objednavky_ID;
```

# *Triggery*
- Databáze obsahuje trigger, nazvaný "Check_Cas_doruceni", který je implementován na tabulce Doruceni a je spouštěn po operacích INSERT a UPDATE. Jeho hlavním účelem je ověřit správnost dat v případě změn stavu doručení zásilky.
- Nejprve si ukládá stav doručení z nových či upravovaných záznamů, které byly vloženy či upraveny, a to prostřednictvím klíče "inserted". Pokud je tento stav doručení označen jako 'Doruceno', trigger kontroluje, zda skutečný čas doručení není starší než datum objednání pro doručené zásilky.
- V případě, že byl tento stav doručení označen jako 'Zruseno', vypíše se zpráva, že objednávka byla zrušena. Stejně tak, pokud je stav 'Ceka na doruceni', vypíše se informace, že objednávka stále čeká na doručení.
- Celkově tento trigger zajišťuje, že v případě doručených zásilek je kontrola provedena pro skutečný čas doručení, a to tak, aby nebyl starší než datum objednání. Pro objednávky s jinými stavy pouze informuje uživatele o stavu dané objednávky.

```
CREATE TRIGGER Check_Cas_doruceni
ON Doruceni
AFTER INSERT, UPDATE
AS
BEGIN
    DECLARE @Stav_doruceni VARCHAR(30);

    SELECT @Stav_doruceni = i.Stav_doruceni
    FROM inserted i
    INNER JOIN Detail_objednavky do ON i.Detail_objednavky_ID = do.ID;

    IF @Stav_doruceni = 'Doruceno'
    BEGIN
        IF EXISTS (
            SELECT 1
            FROM inserted i
            INNER JOIN Detail_objednavky do ON i.Detail_objednavky_ID = do.ID
            WHERE i.Skutecny_cas_doruceni < do.Datum_objednani
        )
        BEGIN
            RAISERROR ('Skutečný čas doručení nemůže být dříve než datum objednání pro doručené zásilky!', 16, 1);
            ROLLBACK TRANSACTION;
        END
    END
    ELSE
    BEGIN
        IF @Stav_doruceni = 'Zruseno'
        BEGIN
            PRINT 'Objednávka je zrušena.';
        END
        ELSE IF @Stav_doruceni = 'Ceka na doruceni'
        BEGIN
            PRINT 'Objednávka čeká na doručení.';
        END
    END
END;
```

# *Uložené procedury*
- Databáze obsahuje proceduru s názvem "Informace_o_produktu" , která je vytvořena pro získání informací o produktu na základě jeho EAN kódu. Procedura vrací údaje o daném produktu včetně informací o objednávce, zákazníkovi, zaměstnanci a stavu doručení tohoto produktu.
- Interně procedura provádí několik JOIN operací mezi tabulkami Produkt, Objednavka, Detail_objednavky, Doruceni, Zamestnanec a Zakaznik na základě identifikátorů produktu a souvisejících entit. Výsledný výběr obsahuje sloupce pro ID produktu, název produktu, ID objednávky, jméno zákazníka, jméno zaměstnance, telefon zaměstnance a stav doručení produktu.
- Po spuštění procedury s daným EAN kódem se vrátí relevantní informace, které lze využít k identifikaci produktu a jeho stavu v rámci objednávek a doručení.

```
CREATE PROCEDURE Informace_o_produktu(@EAN VARCHAR(13))
AS
BEGIN
    SELECT 
        P.ID AS 'ID produktu',
        P.Nazev AS 'Název produktu',
        O.ID AS 'ID objednávky',
        CONCAT(ZK.Jmeno, ' ', ZK.Prijmeni) AS Zákazník,
		CONCAT(ZJ.Jmeno, ' ', ZJ.Prijmeni) AS Zaměstnanec,
        CONCAT('(+420)', SUBSTRING(ZJ.Telefon, 1, 3), '-', SUBSTRING(ZJ.Telefon, 4, 3), '-', SUBSTRING(ZJ.Telefon, 7, 3)) AS 'Telefon zaměstnance',
        D.Stav_doruceni AS 'Stav doručení'
    FROM Produkt P
    JOIN Objednavka O ON P.ID = O.Produkt_ID
    JOIN Detail_objednavky DO ON O.Detail_objednavky_ID = DO.ID
    JOIN Doruceni D ON DO.ID = D.Detail_objednavky_ID
    JOIN Zamestnanec ZJ ON D.Zamestnanec_ID = ZJ.ID
    JOIN Zakaznik ZK ON DO.Zakaznik_ID = ZK.ID
    WHERE P.EAN = @EAN;
END;

EXECUTE Informace_o_produktu @EAN = '1278567890123';
```

## Přístupové údaje do databáze
př:
- Přístupové údaje jsou volně konfigurovatelné v souboru /config/... .doc
pro vývoj byly použity tyto:
host		: localhost
uživatel	: sa
heslo		: student
databáze	: ...

## Import struktury databáze a dat od zadavatele
př:
Nejprve je nutno si vytvořit novou databázi, čistou, bez jakýchkoliv dat...
Poté do této databáze nahrát soubor, který se nachází v /sql/structure.sql ...
Pokud si přejete načíst do databáze testovací data, je nutno nahrát ještě soubor /sql/data.sql ...

# *Klientská aplikace*
- Databáze neobsahuje klientskou aplikaci.

## Požadavky na spuštění
př:
- Oracle DataModeler, rok vydání 2014 a více ... 
- MSSQL Server, rok vydání 2014 a více ... 
- připojení k internetu alespoň 2Mb/s ...
...

## Návod na instalaci a ovládání aplikace
př:
Uživatel by si měl vytvořit databázi a nahrát do ní strukturu, dle kroku "Import struktury databáze 
a dat od zadavatele" ...
Poté se přihlásit předdefinovaným uživatelem, nebo si vytvořit vlastního pomocí SQL příkazů ...
Měl by upravit konfigurační soubor klientské aplikace, aby odpovídal jeho podmínkám ...
Dále nahrát obsah složky src na server a navštívit adresu serveru ... 
Přihlásit se a může začít pracovat ... 

## Závěr
př:
Tento systém by po menších úpravách mohl být převeden na jiný databázový systém, 
klientská aplikace není zabezpečená, 
počítá se s tím, že klient byl proškolen o používání této aplikace ...
Nepodařilo se dořešit ...
Pro další vývoj aplikace by bylo vhodné ...
