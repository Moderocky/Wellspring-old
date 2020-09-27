Attributes
-----

Wellspring contains a system for creating and registering custom attributes.
These can be placed on entities and items, and use the vanilla modifier system, allowing for additive, multiplicative and scalar modifiers.

Custom attributes are entirely compatible with the vanilla system - Minecraft's internal API allows for them to be registered.

Previously, Bukkit has made it impossible to use custom attributes due to its over-reliance on enums and immutable maps.

#### Goals of Attributes
 * To allow plugins to register their own attributes
 * To allow plugins to use attributes for skill-based systems, without needing to write their own system

#### Non-goals of Attributes
 * This is not designed for editing, overwriting or removing vanilla attributes
 * This is not designed for storing statistics or data


How do Attributes Work?
----

An attribute has a base value. This is the value used in calculations if no modifiers are present.

Entity-types can have a different baseline value, which will be the "base" present on all entities of that type.
For example, players have a base maximum health of 20.

On top of this entity baseline, modifiers can be used to alter the value.

A modifier can do one of the following: (taken from wiki)
 * add: Increment X by Amount
 * multiply_base: Increment Y by X * Amount
 * multiply: Y = Y * (1 + Amount) (equivalent to Increment Y by Y * Amount).

These operations are always processed in that order.
This means that the result will always be the same.

Entities can have modifiers directly added to them, or can get them applied by an equipped item or from a potion effect.

Item modifiers may only have an effect while in the "equipment" slots - armour and hands.


Creating an Attribute
----

Using Wellspring, creating a new attribute is as simple as:
```java 
Attribute attribute = Bukkit.registerAttribute(NamespacedKey.minecraft("blob"), 10);
```

*NOTE: the "minecraft" namespace is used here for example purposes! Please use your own in production.*

This should be done during the STARTUP phase so that it is accessible afterwards.
The attribute can then be obtained using its instance, or by simply re-creating it with `new Attribute(NamespacedKey)`.


Currently, our newly-registered attribute is practically useless.
No entities have a provider for it, so any modifiers we make will be ineffective.

To account for this, we have to set a baseline value.
See below for an example.
```java 
Attribute attribute = Bukkit.registerAttribute(NamespacedKey.minecraft("blob"), 10);

EntityType.BAT.setBaseValue(attribute, 10);
EntityType.PLAYER.setBaseValue(attribute, 20);
```

This example would add the `minecraft:blob` attribute baseline to both bats and players.

Any modifiers would now use this as the baseline.