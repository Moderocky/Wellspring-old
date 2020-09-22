NBT Format (Named Binary Tag)
-----

Wellspring exposes Minecraft's NBT data system for plugins using an interface layer. This allows plugins to create and manipulate NBT easily.

#### Terminology
 * NBT Base -> Something (usually a value) wrapped in an NBT serialiser object. All NBT types are NBT bases.
 * NBT Compound -> a string (key) - nbt base (value) "map" stored in a {Key:Value} format.
 * NBT List -> a list of multiple nbt bases.

#### Goals of Wellspring's NBT Implementation
 * To provide plugins with a simple and safe way to use Minecraft's storage format.
 * To allow plugins to access this data on entities and other objects for the purposes of saving and loading it.
 * To allow plugins to use the NBT format for their own purposes without requiring version-dependent code or external libraries.

#### Non-goals of Wellspring's NBT Implementation
 * This is not an excuse to use NBT to modify entities - methods exist for this.
 * This is not to allow for illegal edits to structures or data.
 * This is not for use as a runtime storage format, Java already provides lists and maps.

Different NBT Types
----

There are approximately ten distinct types of NBT Base. Most of these are wrappers for primitive number types (byte, int, short, long, float, double.)
NBT also allows for a faux boolean type using a 1/0 byte. This is internally a byte, but Wellspring allows you to treat it as a boolean.
There is a non-primitive String type.
There are three (existing) number-array types: byte array, int array and long array. There can potentially be other number array types, but these are not present in the Minecraft code explicitly. The number-arrays are technically NBT Lists internally, but have a fixed length and dataset.
There are two complex types: NBT Compounds and NBT Lists. 

There are two additional internal types: Catch-all primitive number and Null terminator. Neither of these should **ever** be touched by a plugin. They are used (respectively) for blanket checks and for serialisation.


Creating New NBT Objects
----

Wellspring structures the visible NBT types using an interface layer. This means that you cannot directly create a `new NBTCompound()` using a constructor.

Instead, you can use the NBT Factory (which provides creation and conversion utilities.)
This can be obtained using: `Bukkit.getNBTFactory()`

To create an NBT Compound (map) use either:
```java
Bukkit.getNBTFactory().newCompound();
```
or:
```java
NBTCompound.create();
```

Either of these will create a new `NBTTagCompound` (the internal Minecraft type) and return it to you as a NBT Compound.

A similar process can be used for lists (`NBTList.create()`) and bases (`NBT.convert("Hello there!")`.)
With the latter, you will notice that we are converting (wrapping) the existing object to an NBT Base, rather than directly creating a new NBT Base.


Using NBT Compounds and Lists
----

Compounds function similarly to Java's HashMaps (and, in fact, are internally a map implementation.)
They use a simple Key - Value system, where the key is a String and the value is an NBT Base.

##### Key Caveats
The keys in an NBT compound must follow a simple regex pattern `[A-Za-z0-9._+-]+` - which does not allow for spaces.
Almost all keys used by Minecraft will be UpperCamelCased words, whereas Bukkit, Spigot and Paper will use `.` and `-` characters as well.
Generally speaking, your keys ought to be `FormattedLikeThis`.

Compounds can hold any NBT type inside them, including other compounds. You may nest compounds up to 512 deep.
The only other limit on NBT compounds is that they are often sent via packets, and there is a limit on how large the packet data may be. Over-sized packets can cause client crashes or errors (e.g. the old book exploit.)

To start our example, we can create a new NBT compound. This will be effectively empty `{}` to start with.
This is effectively a map. It can store data, and can be cloned and merged into other compounds.
```java
class Example {
    public void test() {
        NBTCompound compound = NBTCompound.create();
    }
}
```

There are two ways to add values to our compound. Either we can add the primitives directly using special methods, or we can pre-wrap them using the NBT base converter.

```java
class Example {
    public void test() {
        NBTCompound compound = NBTCompound.create();
        
        

    }
}
```
