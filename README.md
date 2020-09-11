Wellspring
===========

A fork of paper with some tweaks to allow for additional content, modification of internal Minecraft features and better access to the Minecraft Registry for plugins.
No breaking changes should be made to existing features. Wellspring's sole aim is to provide more accessibilty.

Current additional features:
-----
 * NBT is now accessible without requiring version-specific methods, unsafe utilities or reflection.
 * Attachment system to allow for custom fields, methods and utilities to be added to entities and other 'Attachable' objects.
 * Extended potion registry allowing for custom potion types to be registered.
 * Custom entities can now be saved properly over restarts, and displayed correctly to the client.
 * Conversion of attributes from an enum to a field to allow for custom attribute types to work properly with Bukkit.
 * Entity default attribute map is now mutable, allowing for more freedom to customise and edit entity types, and to add new ones.
 * Code changes to allow for the proper creation, storage and viewing of custom entity types.

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
Yes! Though it can only have the surface `API` layer in. As a lot of Wellspring's additions are in the `Server` layer (which cannot be distributed) you ought to build from source yourself. Instructions for this are available on Paper's repository.

**Repository Information:**
```xml
<repository>
    <id>pan-repo</id>
    <name>Pandaemonium Repository</name>
    <url>https://gitlab.com/api/v4/projects/18568066/packages/maven</url>
</repository>
```

**Artifact Information:**
```xml
<dependency>
    <groupId>mx.kenzie</groupId>
    <artifactId>wellspring-api</artifactId>
    <version>1.16.2-R0.1-SNAPSHOT</version>
    <scope>provided</scope>
</dependency>
```

This project forms part of the base for KenzieServer. Additions may be inconsistent, but it will generally be maintained as it is used in live production servers.

See [Paper](https://github.com/PaperMC/Paper) for more information about the parent fork, and for compiling instructions.
