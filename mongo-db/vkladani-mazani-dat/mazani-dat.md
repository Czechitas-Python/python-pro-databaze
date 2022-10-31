## Mazání dat

Často se nám stane, že potřebujeme nějaký záznam smazat, například když omylem vložíme stejnou informaci do kolekce dvakrát. K mazání záznamů můžeme používat funkce `delete_one()` a `delete_many()`. Při volání funkcí vždy použijeme dotaz, který určí, který záznam (nebo které záznamy) chceme smazat.

Funkce `delete_one()` smaže jeden záznam, i kdyby konkrétnímu dotazu vyhovovalo záznamů více. Pokud žádný vyhovující záznam nenajde, neprovede žádnou operaci. Následující kód tedy způsobí, že Petr bude mít v kolekci o jeden nákup méně.

```py
dotaz = {"Jméno": "Petr"}
kolekce.delete_one(dotaz)
```

Následující kód je radikálnější a smaže všechny Petrovy nákupy.

```py
dotaz = {"Jméno": "Petr"}
kolekce.delete_many(dotaz)
```

Pokud chceme smazat jeden konkrétní záznam, často to provádíme na základě jeho jednoznačného identifikátoru `_id`, které každému záznamu přiřadí databáze automaticky. Známe-li `_id`, můžeme ho vložit do dotazu stejně jako jakýkoli jiný klíč a použít ke smazání záznamu. Protože každý dokument má své unikátní `_id`, je logické využít funkci `delete_one()`.

```py
from bson import ObjectId

dotaz = {"_id": ObjectId("62248970d034741099592733")}
kolekce.delete_one(dotaz)
```
