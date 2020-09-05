Wellspring
===========

A fork of paper with some minor tweaks to allow for additional content, modification of internal Minecraft features and better access to the Minecraft Registry for plugins.
No breaking changes will be made to existing features. Wellspring's sole aim is to provide more accessibilty.

Current additional features:
-----
 * Entity attachment system to allow for custom fields and methods to be added to entities.
 * Extended potion registry allowing for custom potion types to be registered.
 * Conversion of attributes from an enum to a field to allow for custom attribute types to work properly with Bukkit.
 * Entity default attribute map is now mutable, allowing for more freedom to customise and edit entity types, and to add new ones.

Planned future features:
-----
 * Split off the Entity#getType() method to allow custom entities to be registered and saved under their own true type, but displayed as an existing valid type to the client.
 * Interface layer with Bukkit for creating attributes, goals, custom entities, potions and enchantments.
 * Replacement of all of the idiotic Bukkit enums - allowing for mod content, items, APIs, etc. to be added easily.
 * Interface layer with Bukkit for a simple packet-sending system.
 * Interface layer with Bukkit for managing basic client-side entities.

This project forms part of the base for KenzieServer. Additions may be inconsistent, but it will generally be maintained as it is used in live production servers.

See [Paper](https://github.com/PaperMC/Paper) for more information about the parent fork, and for compiling instructions.
