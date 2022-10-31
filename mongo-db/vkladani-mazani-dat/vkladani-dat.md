## Vkládání dat

### Vložení jednoho záznamu

Pojďme tento nákup zapsat do naší databáze. V jazyce SQL obvykle používáme příkaz `INSERT`. Názvem příkazu je inspirován i název funkce, kterou budeme používat my.

Ještě malá poznámka ke struktuře. Každá databázový server může obsahovat několik **databází** (např. máme různé databáze pro různé zákazníky, programy, oddělení atd.). V MongoDB se každá databáze skládá z **kolekcí** (v relačních databázích se skládá z tabulek) a každá kolekce se skládá z **dokumentů** (v relačních databázích se tabulka skládá z řádků). Chceme-li něco ukládat, musíme nejprve vytvořit nebo zvolit databázi a kolekci, aby MongoDB vědělo, kam dokument vložit.

K připojení k databázi použijeme modul `pymongo`, který je potřeba importovat příkazem `import`. Následně musíme zadat adresu serveru, uživatelské jméno a heslo. Tyto informace zjistíš během kurzu. Po přihlášení vytvoříme klienta, který bude prostředníkem mezi námi a MongoDB serverem. Následně vybereme konkrétní databázi a kolekci.

```py
import pymongo
nakup = {
    "Jméno": "Petr",
    "Věc": "Prací prášek",
    "Částka v korunách": 399,
    "Datum": "2020-03-04",
    "Značka": "Persil",
    "Hmotnost": 7.8
}

kolekce = databaze["nakupy"]
id = kolekce.insert_one(nakup)
print(id.inserted_id)
```

Ke vložení dokumentu do kolekce použijeme funkce `insert_one()`, která slouží ke vložení jednoho dokumentu. Funkce vrací hodnotu `id`, což je jednoznačný identifikátor našeho záznamu. Samotné ID nám toho moc neřekne, jeho hodnota je například `5fda6f16e6aeccec0ef40b87`.

### Vložení více záznamů

Nyní zkusme vložit více záznamů najednou. Zbývající nákupy máme v proměnné, která se jmenuje `zbyvajici_nakupy`. Tato proměnná obsahuje seznam, což poznáme podle hranatých závorek na začátku a na konci. Seznam se skládá ze slovníků, které mají různé klíče podle toho, co kdo považoval za důležité.

```py
zbyvajici_nakupy = [
    {
        "Jméno": "Petr",
        "Věc": "Toaletní papír",
        "Částka v korunách": 35,
        "Počet rolí": 3
    },
    {
        "Jméno": "Libor",
        "Věc": "Jahodová marmeláda",
        "Částka v korunách": 50,
        "Datum": "2020-03-15"
    },
    {
        "Jméno": "Petr",
        "Věc": "Pytel na odpadky",
        "Částka v korunách": 90,
        "Objem pytle": 15
    },
    {
        "Jméno": "Míša",
        "Věc": "Fólie na potraviny",
        "Částka v korunách": 30,
        "Datum": "2020-03-25"
    },
    {
        "Jméno": "Míša",
        "Věc": "Paralen",
        "Částka v korunách": 130
    },
    {
        "Jméno": "Gumové rukavice",
        "Věc": "Savo",
        "Částka v korunách": 200,
    }
]

kolekce.insert_many(zbyvajici_nakupy)
```

Více záznamů vložíme pomocí funkce `insert_many()`, které předáme náš seznam.