Databáze je systém, který slouží k ukládání dat. Standardní relační databáze ukládají data do předem definovaných tabulek, které jsou provázány vztahy (odtud pojem "relační"). Nejznámější zdarma dostupné databáze jsou PostgreSQL, MySQL a SQLite, z placených poté Microsoft SQL Server nebo Oracle Database. Pro práci s databázemi používáme jazyk SQL (Structured Query Language).

Hlavní motivace, proč používat databáze, jsou:

- umožňují efektivně uložit velké množství dat,
- poskytují způsob, jak snadno a rychle najít požadovanou informaci,
- snadno můžeme přidávat, upravovat, řadit a mazat záznamy,
- umožňují propojení s aplikacemi (např. weby),
- řeší přístup více uživatelů najednou,
- zajišťují zabezpečení dat.

## Čtení dat

NoSQL databáze jsou poměrně široký pojem a zahrnuje různé typy databází, které používají jiný způsob ukládání dat než tabulky provázané relacemi. Konkrétně existuje několik typů NoSQL databází.

- Key-value databáze ukládají data do dvojic, kde jeden prvek (klíč) identifikuje hodnotu (value). Na stejném principu fungují například slovníky v Pythonu.
- Grafové databáze vycházejí z teorie grafů. To je součást matematiky, která se zabývá strukturami složenými z bodů (vrcholů) a spojnic mezi nimi (hranami). Pomocí teorie grafů lze řešit například dopravní úlohy. V grafu pak vrcholy symbolizují města a hrany vzdálenosti mezi nimi. Můžeme pak např. spočítat nejkratší trasu pro návštěvu několika měst. (Speciální případ, kdy navštěvujeme všechna města, se na nazývá *problém obchodního cestujícího*.)
- Dokumentové databáze slouží k ukládání dokumentů, nejčastěji ve formátu JSON, XML nebo YAML.

My se budeme zabývat databází MongoDB, což je dokumentová databáze využívající formát JSON.

Kromě NoSQL databází se v poslední době objevil blockchain, což je způsob ukládání dat, kdy již vložené záznamy nelze nijak upravit.

### Formát JSON

Pojďme se nyní vrátit k našemu příkladu se spolubydlícími. Uvažujeme, že si chtějí svoje výdaje evidovat v databázi. Protože jsou ale poněkud neukáznění, ke každému nákupu si zapisuje různé věci. Např. někdo zapisuje datum nákupu, jiný poznámku, značku výrobku, jestli byl produkt v akci atd. Kvůli jejich kreativitě se rozhodli, že využijí NoSQL databázi, která jim dá větší volnost než klasická databáze. Domluvili se ale, že je nutné vždy uvést jméno kupujícího a cenu. Jednotlivé nákupy pak máme zapsané ve formátu JSON a naším úkolem je uložit je do databáze.

Níže jsou informace o nákupu, jak je zaprotokoloval Petr.

- jméno: Petr,
- věc: Prací prášek,
- částka v korunách: 399,
- datum: 2020-03-04,
- značka: Persil,
- hmotnost: 7.8.

Formát JSON taktéž připomíná slovníky v Pythonu. Data jsou uspořádána do dvojic - klíče a hodnoty. Níže vidíš, jak vypadají data zapsaná ve formátu JSON.

```
nakup = {
    "Jméno": "Petr",
    "Cena": 399,
    "Věc": "Prací prášek",
    "Datum": "2020-03-04",
    "Značka": "Persil",
    "Hmotnost": 7.8
}
```

Ve světě relačních databází používáme ke čtení příkaz `SELECT`. Většinou nechceme načíst všechna data v tabulce (kolekci), ale pouze jejich část. Nyní si ukážeme, jak nastavit, jaká data chceme získat.

### Test načtení dat

K načtení jednoho záznamu použijeme funkci `find_one()`. Pokud funkci nedáme žádný parametr, vrátí nám první záznam z kolekce.

```py
vysledek = kolekce.find_one()
print(vysledek)
```

Funkce vrátí první vložený záznam, který obsahuje námi zadané hodnoty a vygenerované ID.

### Sestavení dotazu

Na MongoDB a modulu `pymongo` je sympatické, že pro dotazy používáme slovník. Dotazy tedy píšeme stejně, jako když připravujeme data pro vložení. Zkusme třeba napsat dotaz na nákupy, které provedl Libor. Dotaz ve formě slovníku předáme funkce `find_one()`.

```py
dotaz = {"Jméno": "Libor"}
vysledek = kolekce.find_one(dotaz)
print(vysledek)
```

Většinou chceme vrátit všechny řádky, které odpovídají našemu dotazu. K tomu slouží funkce `find()`. Ta nám vrátí všechny dokumenty, které odpovídají našemu dotazu, jako seznam. Seznam poté můžeme projít pomocí cyklu.

```py
dotaz = {"Jméno": "Petr"}
vysledek = kolekce.find(dotaz)
for dokument in vysledek:
    print(dokument)
```

Pokud chceme požadovaný dokument vybrat na základě více klíčů, jednoduše z těchto klíčů sestavíme slovník.

```py
dotaz = {"Jméno": "Petr", "Věc": "Toaletní papír"}
vysledek = kolekce.find(dotaz)
for dokument in vysledek:
    print(dokument)
```

Pokud bychom chtěli vypsat jenom nějakou konkrétní informaci (např. cenu), vložíme požadovaný klíč do hranatých závorek za proměnnou `dokument`. Je to podobné, jako kdybychom četli nějakou hodnotu ze seznamu, pouze nepoužíváme číselný index. Pokud znáš slovníky, je ti zápis určitě povedomý.

```py
vysledek = kolekce.find(dotaz)
for dokument in vysledek:
    print(dokument["Částka v korunách"])
```

Určitým nebezpečím je, že tento zápis skončí chybou, pokud by v některém dokumentu nebyl přítomen daný klíč. Pokud máme obavu, že nějaká informace může chybět, můžeme využít metodu `get()`, které můžeme vložit výchozí hodnotu, kterou vrátí, nenalezne-li požadovaný klíč.

Další věc, na kterou si musíme dát pozor, je, že na rozdíl od seznamu nebo slovníku nemůžeme výsledek dotazu automaticky projít cyklem dvakrát. Pokud bychom chtěli s výsledkem stejného dotazu pracovat opakovaně, musíme použít metodu `rewind()` nebo položit dotaz znovu. Proměnná `vysledek` je totiž ve skutečnosti jakási forma záložky (kurzor) v knize. Pokud knihu dočteme, zůstane záložka na konci. Chceme-li knihu přečíst znovu, musíme nejprve záložku přesunout na začátek.

```py
vysledek.rewind()
for dokument in vysledek:
    print(dokument.get("Datum", "Datum není zadáno"))
```

### Větší než, menší než...

U číselných hodnot a dat chceme často formulovat dotaz obsahující nerovnost. Mohli bychom například chtít vypsat všechny nákupy v hodnotě větší než 100 Kč. MongoDB nepoužívá symboly `>` a `<`, ale jejich anglické zkratky. Například porovnání **větší než** zapisujeme jako `$gt`, což vychází z anglického "greater than". Dolar přidáváme, aby si MongoDB zkratku nespletlo s názvem sloupce. Kompletní přehled operátorů najdeš v tabulce níže.

| Význam           | Zápis v Pythonu | Zápis v MongoDB |
| ---------------- | :-------------: | :-------------: |
| Větší než        |       `>`       |      `$gt`      |
| Menší než        |       `<`       |      `$lt`      |
| Větší nebo rovno |      `>=`       |     `$gte`      |
| Menší nebo rovno |      `<=`       |     `$lte`      |

Operátor a hodnotu, se kterou chceme porovnávat, píšeme jako slovník, kde operátor je klíč `{"$gt": 100}`. To pak vložíme do dalšího slovníku, kterým určíme, pro jaký sloupec naše podmínka platí `{"Částka v korunách": {"$gt": 100}}`. Celý zápis tedy vypadá takto:

```py
dotaz = {"Částka v korunách": {"$gt": 100}}
vysledek = kolekce.find(dotaz)
for dokument in vysledek:
    print(dokument)
```

### Výběr hodnoty ze seznamu

Někdy potřebujeme do jednoho dotazu vložit více možných hodnot pro jeden klíč. V MongoDB používáme operátor `in`. Operátor `in` známe i z Pythonu a jeho funkce je zde obdobná: ptáme se, jestli je hodnota klíče pro daný řádek přítomna v námi zadaném seznamu. Syntaxi najdeš na příkladu níže.

```py
dotaz = {"Jméno": {"$in": ["Libor", "Míša"]}}
vysledek = kolekce.find(dotaz)
for dokument in vysledek:
    print(dokument)
```

### Počet hodnot

Pokud by nás zajímal počet dokumentů, které vyhovují našemu dotazu, můžeme použít metodu `count_documents()`, která pro danou kolekci vrátí počet dokumentů, které vyhovují danému dotazu.

```py
dotaz = {"Jméno": {"$in": ["Libor", "Míša"]}}
vysledek = kolekce.find(dotaz)
print(kolekce.count_documents(dotaz))
```

### Dotaz typu "nebo"

Uvažujme nyní, že bychom chtěli vyhledat nákupy, které provedl Petr *nebo* šlo o nákupy toaletního papíru. V tomto případě nemůžeme vložit oba klíče a hodnoty vedle sebe, protože bychom získali nákupy, které provedl Petr *a současně* šlo o nákupy toaletního papíru. K získání dokumentů, které vyhovují alespoň jedné z podmínek, můžeme použít operátor `$or`. Klíče a hodnoty, které hledáme, pak vložíme do seznamu.

```py
dotaz = {"$or": [{"Jméno": "Petr"}, {"Věc": "Toaletní papír"}]}
vysledek = kolekce.find(dotaz)
for dokument in vysledek:
    print(dokument)
```

### Použití regulárních výrazů

V MongoDB můžeme k vyhledávání používat i regulární výrazy. Pokud si například nejsme jisti, jak jsou v kolekci zapsané pytle na odpadky (můžou být v jednotném i množném čísle), můžeme použít žolíky na poslední dvě písmena, která se pro jednotné a množné číslo liší. Při použití regulárních výrazů používáme operátor `$regex`.

```py
dotaz = {"Věc": {"$regex": "Pyt.. na odpadky"}}
vysledek = kolekce.find(dotaz)
for polozka in vysledek:
    print(polozka)
```

Pro lepší kontrolu můžeme použít i skupiny.

```py
dotaz = {"Věc": {"$regex": "Pyt(le|el) na odpadky"}}
vysledek = kolekce.find(dotaz)
for polozka in vysledek:
    print(polozka)
```

### Čtení na doma: Velikost písmen

Pokud bychom chtěli u **hodnot** provádět vyhledávání bez ohledu na velikost písmen, můžeme k tomu využít metodu `collation`. Té jako parametr zadáme dvojici hodnot - `locale` a `strength`. Locale je označení jazyka, podle kterého se má velikost písmen posuzovat, v našem případě zvolíme `cs` (češtinu), pro angličtinu bychom například zadali `en`. Druhý parametr označuje, zda má být velikost písmen ignorována. Pokud zadáme `2`, je při vyhledávání ignorována, pokud `1`, musí být velikost písmen u hodnoty dokumentu stejná jako v našem dotazu.

Pozor, toto nastavení ovlivní pouze hodnoty, velikost písmen u klíče musí být stejná v našem dotazu i dokumentu.

```py
dotaz = {"Věc": "pytel na odpadky"}
vysledek = kolekce.find(dotaz).collation({"locale": "cs", "strength": 2})
for polozka in vysledek:
    print(polozka)
```

Alternativně můžeme použít regulární výrazy, kde nastavíme operátoru `$options` hodnotu `i`, což znamená, že je u regulárních výrazů ignorována velikost písmen (další možnosti, které je možné nastavit, jsou k dispozici v [dokumentaci](https://www.mongodb.com/docs/manual/reference/operator/query/regex/)).

```py
dotaz = {"Věc": {"$regex": "pyt(le|el) na odpadky", "$options": 'i'}}
vysledek = kolekce.find(dotaz)
for polozka in vysledek:
    print(polozka)
```
