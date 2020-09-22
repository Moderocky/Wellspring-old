Wellspring
===========

A fork of paper with some tweaks to allow for additional content, modification of internal Minecraft features and better access to the Minecraft Registry for plugins.
No breaking changes should be made to existing features. Wellspring's sole aim is to provide more accessibilty.

Current additional features:
-----
 * NBT is now accessible without requiring version-specific methods, unsafe utilities or reflection.
 * Attachment system to allow for custom fields, methods and utilities to be added to entities and other 'Attachable' objects.
   * Attachments allow data and code to be attached to entities (and certain other things.)
   * They function as an actually-useful replacement for both entity metadata and persistent data containers.
   * Attachments have a normal class structure, but have a hook to allow data to be saved within NBT.
 * Custom potion effect type builder.
   * Persist over restarts.
   * Work with existing bukkit effects.
   * Allow for custom attribute attachments and on-tick behaviour.
 * Custom entities can now be saved properly over restarts, and displayed correctly to the client.
   * The entity type sent in packets can be separated from the internal type.
 * Conversion of attributes from an enum to a field to allow for custom attribute types to work properly with Bukkit.
   * Custom attributes can be created and added to entities, items, potion effects, and other things.
 * Entity default attribute map is now mutable, allowing for more freedom to customise and edit entity types, and to add new ones.
   * This still *currently* has to be accessed through NMS, but allows custom entity types to have their own attributes.
   * This also allows you to add your custom attributes to entities by default.
 * Code changes to allow for the proper creation, storage and viewing of custom entity types.
   * No more limits on the entity type registry.
   * The existing attribute mappings are now mutable.

Current testing/in-progress features:
-----
 * Packet factory and outgoing packet builders. (Testing)

Planned future features:
-----
 * Interface layer with Bukkit for creating attributes, goals, custom entities, potions and enchantments.
 * Interface layer with Bukkit for creating worlds, biomes and structure types.
 * Replacement of all of the idiotic Bukkit enums - allowing for mod content, materials, blocks, APIs, etc. to be added easily.
 * Interface layer with Bukkit for a simple packet-sending system.
 * Interface layer with Bukkit for managing basic client-side entities.
 * A proper way to create custom NBT-tag implementations, such as to allow for better storage or to store new types.
 * Packet listening/events system.

##### Will this be maintained?
Fairly. It ought to be kept up-to-date with Paper upstream. New major Minecraft versions require me to rewrite a large chunk manually, so those updates might take a little longer.

##### Is it safe to use in production?
I believe so. Wellspring forms a part of KenzieServer (the platform used by my servers) so it is very likely that any bugs will be found by me.
That said-- I cannot possibly check every potential use case by myself, so I would definitely advise caution and prior testing before using the more advanced features.

##### Is there a Maven repository?
Not currently. As a lot of Wellspring's additions are in the `Server` layer (which cannot be distributed) you ought to build from source yourself. Instructions for this are available on Paper's repository.


This project forms part of the base for KenzieServer. Additions may be inconsistent, but it will generally be maintained as it is used in live production servers.

See [Paper](https://github.com/PaperMC/Paper) for more information about the parent fork, and for compiling instructions.
