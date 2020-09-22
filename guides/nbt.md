NBT Format (Named Binary Tag)
-----

Wellspring exposes Minecraft's NBT data system for plugins using an interface layer. This allows plugins to create and manipulate NBT easily.

### Terminology
 * NBT Base -> Something (usually a value) wrapped in an NBT serialiser object. All NBT types are NBT bases.
 * NBT Compound -> a string (key) - nbt base (value) "map" stored in a {Key:Value} format.
 * NBT List -> a list of multiple nbt bases.

### Goals of Wellspring's NBT Implementation
 * To provide plugins with a simple and safe way to use Minecraft's storage format.
 * To allow plugins to access this data on entities and other objects for the purposes of saving and loading it.
 * To allow plugins to use the NBT format for their own purposes without requiring version-dependent code or external libraries.

### Non-goals of Wellspring's NBT Implementation
 * This is not an excuse to use NBT to modify entities - methods exist for this.
 * This is not to allow for illegal edits to structures or data.
 * This is not for use as a runtime storage format, Java already provides lists and maps.


