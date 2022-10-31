## Agregace

Zkusme si nyní vytvořit nějaké složitější dotazy. Důležitou operací, kterou s daty často provádíme, je řazení. Pro řazení potřebujeme zařídit dvě věci.

První z nich je **index**. Index si můžeme představit jako datovou strukturu, která usnadňuje vyhledávání a další práci s hodnotami. Index vytvoříme poměrně jednoduše, a to s využitím metody `create_index()`. U indexu musíme specifikovat, pro který klíč má být vytvořen. Pokud například chceme seřadit nákupy podle klíče `"Částka v korunách"`, vytvoříme index následujícím způsobem.

```py
kolekce = databaze["nakupy"]
kolekce.create_index("Částka v korunách")
```

U větších kolekcí může vytvoření indexu trvat delší dobu. Vytváření indexu může pokračovat i chvíli poté, co skript skončí, v takovém případě je potřeba počkat na vytvoření kompletního indexu.

Dále potřebujeme vytvořit **pipeline**. To je seznam (přeneseně i doslova) kroků, které chceme provést. My do seznamu vložíme pouze jeden krok, a to řazení. Každý krok zadáváme jako slovník. Vytvoříme slovník, který má jako klíč operátor `"$sort"` (operátor pro řazení). Hodnotou klíče je opět slovník, kam vložíme klíč, podle kterého chceme řadit, a hodnotou je `pymongo.ASCENDING` nebo `pymongo.DESCENDING` podle toho, zda chceme řadit vzestupně či sestupně.

Pro větší přehlednost si krok uložíme do samostatné proměnné `krokRazeni`. Po vytvoření pipeline zavoláme metodu `aggregate`, která provede všechny kroky (v našem případě jeden) a výsledek tradičně uložíme do proměnné `vysledek`.

```py
krokRazeni = {
    "$sort": {
        "Částka v korunách": pymongo.DESCENDING
    }
}

pipeline = [
    krokRazeni
]
vysledek = kolekce.aggregate(pipeline)
for dokument in vysledek:
    print(dokument)
```

Nyní vytvoříme spojení do skupiny (`group`). U spojení více řádků do skupin potřebujeme říci, podle jakého klíče vytvoříme skupiny. Pokud bychom chtěli zjistit, kolik každý ze spolubydlících utratil za nákupy, vytvoříme skupiny podle sloupce `Jméno`. Dále vytvoříme nový klíč `Celkem utraceno`, jehož hodnota bude součet všech nákupů v rámci dané skupiny. Samotné spuštění pipeline je již stejné.

```py
kolekce = databaze["nakupy"]

krokSoucetNakladu = {
   "$group": {
         "_id": "$Jméno",
         "Celkem utraceno": { "$sum": "$Částka v korunách" }, 
   }
}

pipeline = [
   krokSoucetNakladu,
]

vysledek = kolekce.aggregate(pipeline)
for dokument in vysledek:
    print(dokument)
```

Nyní si zkusíme vytvořit pipeline se dvěma kroky. Prvním krokem je součet hodnot nákupů jednotlivých spolubydlících. V druhém kroku seřadíme spolubydlící dle celkové útraty od nejvíce po nejméně utrácejícího.

```py
kolekce = databaze["nakupy"]
krokSoucetNakladu = {
    "$group": {
        "_id": "$Jméno",
        "Celkem utraceno": {"$sum": "$Částka v korunách"},
    }
}
krokRazeni = {
    "$sort": {
        "Celkem utraceno": pymongo.DESCENDING
    }
}

pipeline = [
    krokSoucetNakladu,
    krokRazeni
]

vysledek = kolekce.aggregate(pipeline)
for dokument in vysledek:
    print(dokument)
```
