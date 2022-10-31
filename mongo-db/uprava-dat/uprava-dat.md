## Úprava dat

Často potřebujeme upravit již existující záznam. V jazyce SQL k tomu existuje příkaz `UPDATE`, MongoDB můžeme využít funkce `update_one()` nebo `update_many()`.

### Úprava jednoho záznamu

Při úpravách záznamů musíme vždy specifikovat, který záznam chceme upravit. Záznam, který chceme upravit, opět vybereme pomocí dotazu. Jednomu dotazu může vyhovovat více dokumentů. funkce `update_one()` však upraví pouze první vyhovující záznam, na který narazí. Úpravu hodnot specifikujeme jako slovník, do něhož vložíme dvojice klíče-hodnota stejně, jako když jsme vytvářeli nový záznam, např. takto: `{ "Poznámka": "Odečtena vrácená záloha za lahve." }`. Podobně jako u dotazů pak použijeme operátor, který bude tvořit nadřazený slovník. Tentokrát použijeme operátor `$set`. Výsledný slovník pro úpravu dokumentu tedy vypadá takto: `{ "$set": { "Poznámka": "Odečtena vrácená záloha za lahve." } }`.

Níže vidíš sestavení obou slovníků a volání funkce `update_one()`.

```py
dotaz = { "Věc": "Pivo" }
noveHodnoty = { "$set": { "Poznámka": "Odečtena vrácená záloha za lahve." } }
kolekce.update_one(dotaz, noveHodnoty)
```

### Úprava více záznamů

Pokud našemu dotazu vyhovuje více dotazů a my chceme upravit všechny, použijeme funkci `update_many()`. Zadání pro ni připravíme stejně, tj. vytvoříme jeden slovník pro dotaz a další slovník jako popis toho, co má funkce upravit. Například víme, že Petr notoricky zapomíná na dodání účtenky, tak k jeho nákupům přidáme připomenutí.

```py
dotaz = { "Jméno": "Petr" }
noveHodnoty = { "$set": { "Poznámka": "Chybí účtenka." } }
kolekce.update_many(dotaz, noveHodnoty)
```
