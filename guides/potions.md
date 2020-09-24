Custom Potion Effect Types
-----

Wellspring offers a way to create custom potion effect types, with custom behaviour, attributes and colours.
These can persist over restarts and provide developers with new ways to improve their content.

Potion types should be registered by plugins during the `STARTUP` phase, so that they can persist over restarts properly.

Creating a New Type
----

To start with, you need to create a new potion builder.
This allows you to control the various properties of your new potion effect.
The method to use is `PotionBuilder.create(NamespacedKey, String)` and an example can be seen below:

```java 
PotionBuilder builder = PotionBuilder.create(new NamespacedKey(plugin, "immortality"), "Immortality");
```

The properties of the potion that can be altered are as follows:
 * The colour - seen in potion bubbles and on an item
 * Instant (e.g. health, harming) or not (e.g. poison, regeneration)
 * Type (beneficial, neutral, harmful) - whether this is a positive effect
 * Attributes - you can select attributes that will be applied to the recipient while they have this potion
   * The modifiers will be scaled based on the potion's level
   * Your modifier should have a specific and constant UUID
 * On-tick effect - you can set a consumer to be run whenever the potion ticks


A more complete example of all of these can be seen below:

```java 
PotionBuilder builder = PotionBuilder.create(new NamespacedKey(plugin, "immortality"), "Immortality");
builder.setColor(new Color(255, 181, 0));
builder.setInstant(false);
builder.setType(PotionType.BENEFICIAL);
builder.setTickConsumer((entity, level) -> { // run every tick
    entity.setNoDamageTicks(50);
    entity.setAbsorptionAmount((level + 1) * 2);
});
builder.addAttribute(Attribute.GENERIC_ARMOR, new AttributeModifier(UUID.fromString("12D3C89D-5026-4A42-869F-A136BCB64C2C"), "potion", 4.0, AttributeModifier.Operation.ADD_NUMBER));
builder.register(plugin);
```