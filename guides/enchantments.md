Enchantments
-----

Wellspring includes a system for creating and registering a custom enchantment type that functions properly as a default Minecraft enchantment.
Note that due to a client limitation, vanilla clients do not display custom enchantments in item lore.

#### Goals of Custom Enchantments
 * To provide an easy and simple way to create new enchantments
 * To make custom enchantments function properly with Minecraft's behaviour
 * To work flawlessly with Bukkit's existing enchantment system
 * To avoid having to use hacky third-party lore-based enchantment systems

#### Non-goals of Custom Enchantments
 * This is not designed for overriding or replacing default enchantments
 * This is not designed for editing or removing existing enchantments
 * This is not designed for storing data or statistics on an item - use persistent data containers


How do Enchantments Work?
----

Enchantments are stored within the NBT of an item.
They are designed to have a special effect when used in a certain slot, for example frost walker on boots.

Within the implementation, Wellspring provides easy ways to make your enchantments affect damage.
These come from hooks within the Minecraft internal API, and sadly only a couple exist currently.

This means that you may still have to provide some enchantment behaviour yourself using listeners, etc. 

As long as they are registered properly during the STARTUP phase, custom enchantments will persist over restarts and will be functionally indistinguishable from vanilla Minecraft enchantments, aside from the lore-display caveat.

If you do want enchantments to display within an item's lore, you can write your own system to display them.


Creating a Custom Enchantment
----

The first step in creating your own enchantment is to create a new builder instance.
This is done using the `mx.kenzie.wellspring.enchantment.EnchantmentBuilder` class.

A short example can be seen below:
```java 
EnchantmentBuilder builder = EnchantmentBuilder.create(NamespacedKey.minecraft("blob"), "Blob");
Enchantment blob = builder.create(); // A registered bukkit enchantment.
```

Please note: it is NOT recommended for you to use the `minecraft:` namespace!
In this example, you would be able to apply the enchantment using the `/enchant YourName blob` command.

Enchantments registered under a plugin namespace must use the plugin's key here. E.g. `/enchant YourName myplugin:blob`

#### Enchantment Properties
|Name         |Default Value|Description         |
|-------------|-------------|--------------------|
|Min. Level |1|The minimum (starting) level - should almost always be 1.|
|Max. Level |1|The maximum level of this enchantment. Note: values over 10 have no language entry.|
|Min. Cost |1|The minimum cost for applying this within an anvil.|
|Max. Cost |1|The maximum cost for applying this within an anvil.|
|Treasure |false|Whether this is a 'treasure' enchantment - not found in the enchanting table.|
|Curse |false|Whether this is a 'curse' enchantment - not removable by grindstones.|
|Tradable |false|Whether this can be traded by villagers.|
|Discoverable |false|Unknown. This might dictate whether the enchantment appears in an enchanting table.|
|Rarity |COMMON|This dictates the chance of finding the enchantment in various forms.|
|Target |WEARABLE|This marks what sort of items the enchantment applies to. Note: overridden by CanEnchant.|
|CanEnchant (Func.) |null|This overrides Target. Used to dictate whether an item can hold this enchantment.|
|EffectiveSlots |null|The enchantment's behaviour functions will only apply if in one of these slots.|
|DamageProtection (Func.) |-> 0|This applies a damage protection modifier based on the damage cause.|
|DamageBonus (Func.) |-> 0.0F|This applies a damage bonus based on the victim's type.|
|Compatible (Func.) |null|Alter compatibility with other enchantments. Works with CanEnchant.|
|Description |null|The description's language key. Only useful with custom resource/language packs.|
|NameByLevel (Func.) |Default|Generate the enchantment name based on the level. By default produces (name) (numeral).|
|DealDamage (Func.) |null|Set behaviour for when damage is dealt to an entity by something wielding this enchantment. Requires EffectiveSlots.|
|TakeDamage (Func.) |null|Set behaviour when the wielder takes damage. Requires EffectiveSlots.|


Examples
----

```java 
EnchantmentBuilder builder = EnchantmentBuilder.create(NamespacedKey.minecraft("blob"), "Blob"); // Please make your own key :)
builder.setMaxLevel(3);
builder.setEffectiveSlots(EquipmentSlot.values()); // Effective on all slots
builder.setTarget(Target.WEARABLE);
builder.setRarity(Rarity.VERY_RARE);
builder.setTreasure(true);
builder.setMaxCost(3);
builder.setMinCost(1);
builder.setCanEnchant(stack -> true); // Makes this applicable to ANY item

builder.onTakeDamage((holder, attacker, level) -> { // When the holder takes damage
    // attacker might be null if the holder walked into a cactus or something
    if (attacker != null)
        holder.setAbsorptionAmount((level + 1) * 2);
});

builder.onDealDamage((holder, victim, i) -> { // When the holder deals damage
    // victim should never be null here
    holder.setAbsorptionAmount(0); 
});

Enchantment blob = builder.create();
// A fully-registered Bukkit enchantment.
// Store this in a field, or something.
```