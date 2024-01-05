# DB-Projekt

# Praktická maturitní zkouška

**Střední průmyslová škola elektrotechnická, Praha 2, Ječná 30**
**Školní rok 2023/2024**
---
Jméno a příjmení: Tomáš Novotný
---
Třída: C4b
---

# *Úvod*
Problém jsem se rozhodl řešit v MSSQL Server a jako návrhové prostředí jsem využil Oracle DataModeler, ve kterém jsem vytvořil logické a relační schéma.

Tato dokumentace poskytuje komplexní přehled a strukturované informace o navržené a implementované relační databázi. Cílem tohoto projektu je vytvoření a správa databáze pro sledování objednávek, zákazníků, produktů a doručení v rámci podniku rohlik.cz.

Databáze obsahuje sadu propojených tabulek a vztahů mezi nimi, umožňující efektivní správu dat a snadný přístup k potřebným informacím. Struktura databáze je navržena tak, aby reflektovala reálné procesy podniku a umožňovala rychlé a přesné dotazování.

Tato dokumentace také zahrnuje detailní popis jednotlivých tabulek, vazeb, pohledů, procedur, triggerů a zabezpečení implementovaných v rámci databáze. Kromě struktury obsahuje také návody a doporučení pro instalaci, správu a zálohování databáze.

# *Zdroj zadání*
Odkaz na webovou stránku, podle které jsem databázi navrhoval a vytvořil a stručný popis toho, proč jsem si tuto webovou stránku zvolil, lze nalézt v /src/zdroj zadání (tj. odkaz na website nebo doklad).txt

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
- Cena_celkem: Desetinné číslo (max. 10 míst celkem, 2 za desetinnou čárkou), povinný

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

# *Transakce*
- Databáze obsahuje transakci, která začíná deklarováním proměnných pro potřebné informace, jako jsou ID objednávky a produktu, množství a celková cena.
- Poté se pomocí těchto informací připraví hodnoty pro vložení do tabulky Objednavka.
- Pokud jsou všechny potřebné informace definovány (nejsou NULL) a transakce obsahuje všechny potřebné informace, vloží data do tabulky 'Objednavka' a vypíše zprávu o úspěšném vložení.
- Pokud některé informace chybí, vypíše se chybová zpráva a transakce je zrušena pomocí ROLLBACK.
- V případě úspěchu se transakce potvrdí pomocí COMMIT.

```
BEGIN TRANSACTION;

DECLARE @DetailObjednavkyID INT;
DECLARE @ProduktID INT;
DECLARE @Mnozstvi INT;
DECLARE @CenaCelkem DECIMAL(10, 2);

-- Hodnoty pro vkládání
SET @DetailObjednavkyID = (SELECT ID FROM Detail_objednavky WHERE Zakaznik_ID = 2);
SET @ProduktID = (SELECT ID FROM Produkt WHERE Produkt.EAN = '1278567890123');
SET @Mnozstvi = 4;
SET @CenaCelkem = (SELECT Cena FROM Produkt WHERE ID = @ProduktID) * @Mnozstvi;

-- Vložení dat do tabulky Objednavka, pouze pokud jsou všechny informace definovány
IF (@DetailObjednavkyID IS NOT NULL AND @ProduktID IS NOT NULL AND @Mnozstvi IS NOT NULL AND @CenaCelkem IS NOT NULL)
BEGIN
    INSERT INTO Objednavka (Detail_objednavky_ID, Produkt_ID, Mnozstvi, Cena_celkem)
    VALUES (@DetailObjednavkyID, @ProduktID, @Mnozstvi, @CenaCelkem);
    PRINT 'Data byla úspěšně vložena do tabulky Objednavka.';
END
ELSE
BEGIN
    PRINT 'V průběhu transakce někde nastala chyba. Je možné, že některé informace nejsou dostupné pro vložení do tabulky Objednavka.';
    ROLLBACK TRANSACTION; -- Pokud chybí některé informace, zruší transakci
END

COMMIT TRANSACTION; -- Potvrzení transakce
```

# *Export databáze*
Export databáze se nachází v /sql/export.sql.

# *Import struktury databáze a dat od zadavatele*
- Nejprve je nutno si vytvořit novou databázi, čistou, bez jakýchkoliv dat.
- Poté je zapotřebí do této databáze nahrát soubor, který se nachází v /sql/structure.sql, který slouží pro nahrání struktury mé databáze.
- Pokud si přejete načíst do databáze testovací data, je nutno nahrát ještě soubor, který se nachází v /sql/data.sql.
- Pro nahrání všech triggerů, pohledů, procedur, transakcí a indexů atd., je zapotřebí nahrát soubor script.sql, který se nachází v /sql/script.sql.

# *Zálohování*
Protože k upravení a celkovému vytvoření mé databáze používám databázové servery Microsoft SQL Server, které provozuje škola SPŠE Ječná, tak nemohu nijak zálohovat svou databázi.
Z toho důvodu se v této části pokusím teoreticky popsat, jak bych postupoval, pokud bych měl možnost mou databázi zálohovat.

### Plná záloha databáze
Plná záloha obsahuje kompletní data a strukturu databáze v daném čase. Pro provedení plné zálohy bych mohl použít následující příkaz:

```
BACKUP DATABASE NazevDatabaze
TO DISK = 'C:\Cesta\Kam\Ulozit\Zalohu\FullZaloha.bak'
WITH INIT, STATS = 10;
```
Tento příkaz vytvoří kompletní zálohu celé databáze do souboru FullZaloha.bak na disku C:\. INIT znamená, že záloha bude přepsána, pokud již existuje soubor se stejným názvem. STATS určuje způsob zobrazení průběhu zálohování.

### Diferenciální záloha databáze
Diferenciální záloha obsahuje pouze změny od poslední plné zálohy. Pro provedení diferenciální zálohy bych mohl použít následující příkaz:
```
BACKUP DATABASE NazevDatabaze
TO DISK = 'C:\Cesta\Kam\Ulozit\Zalohu\DiferencialniZaloha.bak'
WITH DIFFERENTIAL, INIT, STATS = 10;
```
Tento příkaz vytvoří zálohu obsahující změny od poslední plné zálohy. DIFFERENTIAL označuje, že se jedná o diferenciální zálohu. INIT opět znamená, že záloha bude vytvořena i v případě existence již existujícího souboru se stejným názvem.

Je důležité si uvědomit, že diferenciální zálohy se mohou stávat většími s každou změnou v databázi, protože obsahují změny od poslední plné zálohy. Při obnově záloh bude potřeba nejprve obnovit poslední plnou zálohu a poté diferenciální zálohy v pořadí vytvoření.

# *Nastavení uživatelských oprávnění*
Protože k upravení a celkovému vytvoření mé databáze používám databázové servery Microsoft SQL Server, které provozuje škola SPŠE Ječná, tak nemohu nijak vytvářet LOGINY ani USERY.
Z toho důvodu se v této části pokusím teoreticky popsat, jak bych postupoval, pokud bych měl možnost vytvářet LOGINY, USERY a zároveň jak bych pracoval s uživatelskými oprávněními.

### Vytvoření LOGINu
LOGIN představuje přihlašovací údaje pro přístup do SQL Serveru. Zde je ukázka jak bych postupoval při vytváření LOGINu:
```
CREATE LOGIN JmenoLoginu
WITH PASSWORD = 'HesloLoginu';
```
Tímto příkazem bych vytvořil LOGIN s daným jménem a heslem. Samotný LOGIN však nemá přístup k žádné databázi, je to pouze identifikace pro přihlášení do serveru.

### Vytvoření USERa
USER je propojen s LOGINem a přiděluje specifická práva k databázím. Zde je příklad jak bych postupoval při vytváření USERa propojeného s LOGINem:
```
USE NazevDatabaze;
CREATE USER JmenoUsera FOR LOGIN JmenoLoginu;
```
Tímto příkazem bych vytvořil USERa v konkrétní databázi, který by byl spojen s LOGINem. USER by mohl mít různá oprávnění v dané databázi.

### Vytvoření ROLE
ROLE je skupina oprávnění, která mohou být přidělena USERům. Zde je příklad jak bych postupoval při vytváření ROLE:
```
USE NazevDatabaze;
CREATE ROLE JmenoRole;
```
ROLE může obsahovat specifická oprávnění a může být přidělena jednomu nebo více USERům. Poté bych mohl přidělit oprávnění pro tuto ROLI.

### Přidělení oprávnění pro roli
```
USE NazevDatabaze;
GRANT SELECT, INSERT, UPDATE ON SCHEMA::JmenoSchema TO JmenoRole;
```
Tímto příkazem bych přidělil roli JmenoRole určitá oprávnění (SELECT, INSERT, UPDATE) na konkrétním schématu (JmenoSchema). Oprávnění mohou být různá a mohou ovlivňovat určité tabulky, pohledy, procedury atd. ve specifickém schématu.

### Přidělení přístupu k určitým objektům přímo
```
USE NazevDatabaze;
GRANT SELECT ON NazevTabulky TO JmenoRole;
```
Tímto způsobem bych přidělil roli JmenoRole právo SELECT pro konkrétní tabulku NazevTabulky. Tento způsob přiřazování oprávnění je flexibilní a mohl bych tak přizpůsobit přístup k jednotlivým objektům dle potřeby.

# *Klientská aplikace*
- Databáze neobsahuje klientskou aplikaci.

# *Požadavky na spuštění*
- Pro běh MSSQL Serveru je nutná verze z roku 2014 a novější.
- K dispozici by mělo být připojení k internetu s rychlostí minimálně 2Mb/s.
- Pro práci s touto databází je také vhodné mít nainstalovaný prostředek pro práci s SQL -> SQL Server Management Studio pro SQL Server.
- Základní znalost SQL jazyka je pro efektivní práci s databází nezbytná.
- Pro správnou konfiguraci a práci s databází je dobré se seznámit s částí [Import struktury databáze a dat od zadavatele](https://github.com/tomasnovotnyy/DB-Projekt/blob/main/README.md#import-struktury-datab%C3%A1ze-a-dat-od-zadavatele)

# *Závěr*
Tato databáze představuje komplexní prostředí pro správu informací týkajících se objednávek, produktů, zákazníků, zaměstnanců, doručení a detailů objednávky. Je navržena tak, aby poskytovala robustní a efektivní řešení pro sledování a správu objednávek a souvisejících informací.

Při vývoji této databáze byl kladen důraz na integritu dat, správnost vztahů mezi entitami a optimalizaci pro efektivní manipulaci s informacemi. Databáze obsahuje vhodné omezení a kontrolní mechanismy, které zajišťují konzistenci a bezpečnost dat.

Tato databáze představuje základní kámen pro správu objednávek a souvisejících informací a může být v budoucnu dále rozšiřována a vylepšována.
