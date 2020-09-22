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
`Bukkit.getNBTFactory().newCompound()`
or:
`NBTCompound.create()`

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
    void test() {
        NBTCompound compound = NBTCompound.create();
    }
}
```

There are two ways to add values to our compound. Either we can add the primitives directly using special methods, or we can pre-wrap them using the NBT base converter.

```java
class Example {
    void test() {
        NBTCompound compound = NBTCompound.create();
        simple: { // This wraps the int internally
            compound.setInt("Number", 10); // This is the best method to use
        }
        simple: { // This wraps the int internally
            compound.set("Number", 10);
            // Slightly slower than the first method
            // always better to use an explicit type
        }
        complex: {
            NBT value = NBT.convert(10, NBT.Type.INT);
            compound.set("Number", value);
        }
        complex: { 
            NBT value = NBT.convert(10);
            compound.set("Number", value); 
        }
    }
}
```
The outcome of all methods will produce the compound `{Number:10}`.
As you can see, Wellspring provides multiple ways of converting raw primitive types to NBT bases to be used.

It is almost always best to explicitly provide the type where possible. For un-typed conversions, Wellspring will simply try to find a matching type for it.

If you pass something such as a List, Map or array to the converter it will break it down into the individual types and construct a new NBT List, attempting to recursively convert the contents.

When it comes to retrieving a value from an NBT compound, there are three possible approaches.

```java
class Example {
    void test() {
        NBTCompound compound = NBTCompound.create();
        compound.setInt("Number", 10);
        simple: {
            int i = compound.getInt("Number");
            // The best option, returns an explicit type.
        }
        complex: {
            int i = compound.get("Number", NBT.Type.INT);
            // The second best option.
            // Java will attempt to automatically cast to your variable type.
            // If you need the value explicitly, you can use the following:
            compound.<Integer>get("Number", NBT.Type.INT);
        }
        complex: {
            int i = compound.get("Number").getAsObject();
            // Wellspring will attempt to automatically convert the NBT base to an object.
            // Java will attempt to automatically cast to your variable type.
            // If you need the value explicitly, you can use the following:
            compound.get("Number").<Integer>getAsObject();
        }
    }
}
```

As with setting the value, Wellspring provides a utility for automatic conversion of unknown types. This is not foolproof, however, and can cause unexpected behaviour. It is always better to use an explicit type.

#### Further Examples

Below are some more examples of manipulating NBT maps. 
Please note that these are not actual implementations, but simply examples of what can be done.


##### Example 1.

```java
class Example {
    void test() {
        NBTCompound compound = NBTCompound.create();
        compound.setInt("Number", 10);
        compound.setList("MyList", NBTList.create(1, 2, 3));
        compound.setList("MySecondList", NBTList.create("hello", "there"));
        compound.set("MyThirdList", Arrays.asList("general", "kenobi"));
        // Automatically converts the list to an NBT list, recursively!
        compound.set("AnotherList", new String[] {"hi"});
        // Automatically converts the array to an NBT list, recursively!

        compound.setBoolean("Boolean", true);
        compound.set("Nested", NBTCompound.create());

        compound.set("Spooky", (Object) "hello"); // Unknown types are fine!

        Map<String, Object> map = new HashMap<>();
        map.put("Blob", 10.0D);
        map.put("Foo", "Bar");

        compound.set("Manual", NBTCompound.create(map));
        compound.set("Automatic", map); // Internally converted via NBTCompound#create(Map)

        int ten = compound.getInt("Number"); // 10
        List<Integer> list1 = compound.getList("MyList").unwrap(); // Automatic unwrapping
        NBTList list2 = compound.getList("MySecondList"); // Automatic unwrapping
        if (compound.containsKey("Boolean", NBT.Type.BOOLEAN))
            // Do something?
            ;

        Map<String, Object> unwrapped = compound.unwrap();
        // Returns the compound's contents to a map of Java types
    }
}
```

The resulting NBT:
`{Boolean:1b,MySecondList:["hello","there"],Spooky:"hello",Manual:{Blob:10.0d,Foo:"Bar"},AnotherList:["hi"],MyList:[1,2,3],Number:10,MyThirdList:["general","kenobi"],Automatic:{Blob:10.0d,Foo:"Bar"},Nested:{}}`

Note: serialised values are not in any particular order.


##### Example 2.

```java
class Example {
    void test() {
        Player player = Bukkit.getOnlinePlayers().iterator().next(); // Any player

        NBTCompound compound = player.getNBT();
        
        boolean invulnerable = compound.getBoolean("Invulnerable");
        compound.set("Glowing", true);
        compound.setDouble("Health", 20.0);
        
        player.mergeNBT(compound); // Really bad idea - don't mess with player NBT
        
        Map<String, Object> map = compound.unwrap();
        // Recursively converts the object into Java types

        NBTCompound remade = NBTCompound.create(map);
        // Wraps and converts the map back into an NBT compound
        // different from the original but the values are equal
    }
}
```

##### Example 3.

```java
class Example {
    void test() {
        Map<String, Object> map = new HashMap<>();
        map.put("Name", "Terrence");
        map.put("Surname", "Blob");
        map.put("Age", 25);
        map.put("BirthDate", "03/02/01");
        map.put("LuckyNumbers", new int[] {10, 5, 7, 23, 14});
        map.put("StarSign", "Leo");
        map.put("Strength", 20.5D);
        map.put("Words", Arrays.asList("car", "park", "box", "pasta"));

        NBTCompound compound = NBTCompound.create(map);

        String serialised = compound.toString();
        NBTCompound parsed = NBTCompound.create(serialised);

        Map<String, Object> recreation = parsed.unwrap();
        // This will contain the same values as the first
        // The order may have changed
    }
}
```