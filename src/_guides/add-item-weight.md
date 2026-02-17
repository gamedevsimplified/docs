---
order: 700
title: Add Item weight
---

!!!
This setup uses the `Basic` template as an example. If you're using another template, apply the same steps to your version instead.
!!!

If your game uses an *Encumbrance* system where item weight matters, you can extend a template to support it.

### 1. Extend ItemBase

Add a `Weight` field to `ItemBase`.

```cs #4 ItemBase.cs
public class ItemBase : GDS.Core.ItemBase {
    public Class Class = Class.NoClass;
    public static readonly NoBase NoBase = new();
    public int Weight = 0; /*✚*/
}
```

Expose a `Weight` property in `BasicItem` that returns the base item's weight.

```cs #5 Item.cs
public record BasicItem : Item {
    public Rarity Rarity = Rarity.NoRarity;
    public string CustomName { get; init; }
    public string CustomIcon { get; init; }
    public int Weight => (Base as ItemBase).Weight; /*✚*/
}
```

---
### 2. Set Weights for Item Bases

For each item base that should have weight, assign an appropriate value to its `Weight` field.

```cs # Bases.cs
public static readonly ArmorBase WarriorHelmet = new() { Id = "WarriorHelmet", /*...*/ Weight = 10 /*✚*/};
public static readonly ArmorBase LeatherGloves = new() { Id = "LeatherGloves", /*...*/ Weight = 8 /*✚*/};
public static readonly ArmorBase LeatherArmor = new() { Id = "LeatherArmor", /*...*/ Weight = 7 /*✚*/};
public static readonly ArmorBase SteelBoots = new() { Id = "SteelBoots", /*...*/ Weight = 10 /*✚*/};

public static readonly WeaponBase Axe = new() { Id = "Axe", /*...*/ Weight = 13 /*✚*/};
public static readonly WeaponBase LongSword = new() { Id = "LongSword", /*...*/ Weight = 12 /*✚*/};
public static readonly WeaponBase ShortSword = new() { Id = "ShortSword", /*...*/ Weight = 11 /*✚*/};
```

---
### 3. Show Weight in Tooltips

To display item weight in the tooltip, update the **Tooltip** view.

This example uses a label but you can use an icon instead.

```cs #7,15,22,26 Tooltip.cs
VisualElement WeaponTooltip(WeaponItem item, string cost) => Div(
    NameLabel(item),
    RarityLabel(item),
    AffixLabel("Attack", item.Attack),
    AffixLabel("Attack Speed", item.AttackSpeed),
    AffixLabel("DPS", item.DPS),
    WeightLabel(item), /*✚*/
    CostLabel(cost)
);

VisualElement ArmorTooltip(ArmorItem item, string cost) => Div(
    NameLabel(item),
    RarityLabel(item),
    AffixLabel("Defense", item.Defense),
    WeightLabel(item), /*✚*/
    CostLabel(cost)
);

VisualElement DefaultTooltip(BasicItem item, string cost) => Div(
    NameLabel(item),
    RarityLabel(item),
    WeightLabel(item), /*✚*/
    CostLabel(cost)
);

Label WeightLabel(BasicItem item) => AffixLabel("Weight", item.Weight).SetVisible(item.Weight > 0); /*✚*/
```

![The **Weight** should show up in the **Tooltip**](/static/images/tutorials/add-weight-tooltip.jpg){.rounded-lg}

---

### 4. Compute Total Weight

In order to show the total weight in the **Character Sheet** view, we need to add it to `CharacterSheet` class and recompute on equipment change.

```cs #1,4,7,14,17 CharacterSheet.cs
public record CharacterStats(int Defense, float Damage, float AttackSpeed, int Weight /*✚*/);
public Observable<CharacterStats> Stats;
public CharacterSheet(SetBag bag) {
    Stats = new(new(0, 0, 0, 0 /*✚*/));

    bag.Data.OnChange += (slots) => {
        int defense = 0, weight = 0 /*✚*/;
        float dps = 0f, aps = 0f;

        foreach (var slot in slots.Values) {
            if (slot.Item is not BasicItem item) continue;
            if (item is ArmorItem a) defense += a.Defense;
            if (item is WeaponItem b) { aps = b.AttackSpeed; dps = b.DPS; }
            weight += item.Weight; /*✚*/
        }

        Stats.SetValue(new(defense, dps, aps, weight /*✚*/));
    };
}
```

### 5. Update Character Sheet Window

Finally, update the `CharacterSheetWindow` class to show the total weight.

As a bonus, you can add color-coded encumbrance levels based on custom ranges — tweak these thresholds as needed.

```cs #6,13,21,26-30 CharacterSheetWindow.cs 
public CharacterSheetWindow(CharacterSheet character) {

    var Defense = Label("character-line", "Defense");
    var DPS = Label("character-line", "DPS");
    var APS = Label("character-line", "Attack Speed");
    var Weight = Label("character-line", "Weight"); /*✚*/
    this.Add("window character-sheet gap-v-50",
        Components.CloseButton(character),
        Title("Character Sheet"),
        Div(Defense,
            DPS,
            APS,
            Weight /*✚*/
        )
    );

    this.Observe(character.Stats, (data) => {
        Defense.text = $"Defense: {DefenseText(data.Defense)}";
        DPS.text = $"Damage: {DamageText(data.Damage)}" + "/s".DarkGray();
        APS.text = $"Attack Speed: {data.AttackSpeed}" + "/s".DarkGray();
        Weight.text = $"Total Weight: {data.Weight} ({WeightText(data.Weight)})"; /*✚*/
    });

}

string WeightText(int value) => value switch {  /*✚*/
    >= 40 => "Heavy".Red(),                     /*✚*/
    >= 25 => "Medium".Yellow(),                 /*✚*/
    _ => "Light".Blue()                         /*✚*/
};                                              /*✚*/
```
![The **Total Weight** should show up in the **Character Sheet**](/static/images/tutorials/add-weight-char-sheet.jpg)