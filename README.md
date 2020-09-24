Wellspring
===========

A fork of paper with some tweaks to allow for additional content, modification of internal Minecraft features and better access to the Minecraft Registry for plugins.
No breaking changes should be made to existing features. Wellspring's sole aim is to provide more accessibilty.

Maven Information
-----
Repository information:
```xml
<repository>
    <id>pan-repo</id>
    <name>Pandaemonium Repository</name>
    <url>https://gitlab.com/api/v4/projects/18568066/packages/maven</url>
</repository>
```

Dependency information:
```xml
<dependency>
    <groupId>mx.kenzie.wellspring</groupId>
    <artifactId>wellspring-api</artifactId>
    <version>1.16.3-R0.1-SNAPSHOT</version>
    <scope>provided</scope>
</dependency>
```

As a lot of Wellspring's additions are in the `Server` layer (which cannot be distributed) you may want to build from the source yourself.
Instructions for this are available on Paper's repository.


Current additional features:
-----
 * NBT is now accessible without requiring version-specific methods, unsafe utilities or reflection.
   * See [here](https://github.com/Moderocky/Wellspring/blob/master/guides/nbt.md) for a proper guide to NBT.
 * [Attachment](https://github.com/Moderocky/Wellspring/blob/master/guides/attachments.md) system to allow for custom fields, methods and utilities to be added to entities and other 'Attachable' objects.
   * Attachments allow data and code to be attached to entities (and certain other things.)
   * They function as an actually-useful replacement for both entity metadata and persistent data containers.
   * Attachments have a normal class structure, but have a hook to allow data to be saved within NBT.
   * See [here](https://github.com/Moderocky/Wellspring/blob/master/guides/attachments.md) for more information.
 * Custom potion effect type [builder](https://github.com/Moderocky/Wellspring/blob/master/guides/potions.md).
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
 * Interface layer with Bukkit for a simple packet-sending system.

Planned future features:
-----
 * Interface layer with Bukkit for creating attributes, goals, custom entities, potions and enchantments.
 * Interface layer with Bukkit for creating customised worlds, biomes and structure types.
 * Replacement of all of the idiotic Bukkit enums - allowing for modded content, materials, blocks, items, APIs, etc. to be added easily.
   * This will (eventually) include materials, though that will take a long time to complete.
   * With any hope, this will allow for the reintroduction of projects like "forgebukkit" and similar.
 * Interface layer with Bukkit for managing basic client-side entities.
 * A proper way to create custom NBT-tag implementations, such as to allow for better storage or to store new types.
 * Packet listening/events system.

#### How does the version cycle work?
Wellspring has a very simple versioning system.
It takes into account the Minecraft version, the testing scope, and the build number.

For example: `1.16.3-alpha-3`
`1.16.3` -> Minecraft version
`alpha` -> release channel
`3` -> the build number for this Minecraft version

Build numbers increment with every release, no matter the release channel. This means that `alpha-5` is newer than `beta-4`, for example.

The release channels reference how extensively the release has been tested.
`alpha` -> means it has built successfully, starts and functions successfully, and has compiled into KenzieServer properly. This means it is being used by at least one server with no known issues, however it has not been extensively tested.
`beta` -> it has been tested on multiple servers and environments, with no issues found.
`stable` -> it has been widely used with no known problems, and has been available for long enough that any major issues should have been discovered.
`perfect` -> the final release for this Minecraft version cycle, with no new features planned, no bugs found, and no changes anticipated.

Alpha versions will rarely have proper documentation as they are made available for the purposes of testing/reviewing and can undergo major revisions. This is a good time to recommend changes, debate the implementation of a feature or suggest a better way of doing something.

Beta releases should have extensive documentation, and are generally the "dress rehearsal" for the stable release. Method erasure should remain the same.

#### Will this be maintained?
Fairly. It ought to be kept up-to-date with Paper upstream. New major Minecraft versions require me to rewrite a large chunk manually, so those updates might take a little longer.

#### Is it safe to use in production?
I believe so. Wellspring forms a part of KenzieServer (the platform used by my servers) so it is very likely that any bugs will be found by me.
That said-- I cannot possibly check every potential use case by myself, so I would definitely advise caution and prior testing before using the more advanced features.



This project forms part of the base for KenzieServer. Additions may be inconsistent, but it will generally be maintained as it is used in live production servers.

See [Paper](https://github.com/PaperMC/Paper) for more information about the parent fork, and for compiling instructions.
