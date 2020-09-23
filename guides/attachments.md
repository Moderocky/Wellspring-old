Attachments
-----

Wellspring's attachment system is designed to provide plugin creators with a fast and simple method of linking plugin-provided data to entities.

#### Goals of Attachments
 * To provide a simpler and faster system than Spigot's metadata and persistent data containers
 * To follow OOP principles and not force users to rely on singleton maps and manager classes
 * To give developers more freedom about what data to store and how to store it
 * To avoid superfluous null-checks at every access
 * To offer a new way of providing public APIs without needing singleton API classes

#### Non-goals of Attachments
 * This is not a place to add custom behaviour and tick-actions to entities - please use pathfinder goals instead
 * This is not a place to try to perform illegal NBT edits during entity saving/loading
 * This is not a place to store static code and global utility functions


How do Attachments Work?
----

Any plugin can provide its own implementation of AttachableObject. As such, this information is true only for the implementation provided for entities (worlds and tile entities coming soon.)

When an attachable object (such as a CraftEntity) is created, any attachments linked to the object's class (or a superclass thereof) will be instantiated and linked to the object.
This means that if an attachment has been registered for `Monster.class`, and an entity is spawned that implements Monster, the entity will have that attachment added to it.

An AttachableObject can have any number of attachments added to it.

Whenever the entity's NBT data is saved or loaded (e.g. during original creation, world-saving, `/data get` command, plugin NBT access, metadata packets, etc.) any attachments will be given the opportunity to access the NBT compound map. During data-load, this will be read-only access (from a copy of the compound) and during saving this will be read-write access (the edits will be merged with the original.)

This means that the attachment can choose to read/write data to the NBT and have Minecraft handle the saving of it, so long as it can be stored in the NBT format.

Note: As plugins accessing NBT (e.g. `Entity#getNBT()`) will also trigger the saving hook, methods like this should **NEVER** be used within the hooks, since they would recurse infinitely. Generally speaking, the attachable object should **not** be edited or accessed at all during the saving process. During the loading process, access should be fine.

The only strong copy of an attachment is held directly on the attachable object. This means that if the entity object is garbage-collected, so too will be the attachments. In most cases, the attachment is notified pre-destruction but this is not fail-safe. It should not be necessary to listen to this though, as the saving method will be called if appropriate.


Designing an Attachment
----

When creating an attachment, please be mindful that a copy of it will exist for every object that it can be attached to.
Attachments should avoid static code where possible (except for necessary constant values.)

Below is an example attachment for players. It has no function.

```java
public class PlayerData extends Attachment<Player> { // The generic here tells us what needs to be provided in the constructor.

    // We can put our own fields here
    
    public PlayerData(@NotNull Player subject) {
        super(subject);
        // We could add our own code here!
    }
    
    // We can put our own methods here

}
```

The important part is the player-accepting constructor, which will make registering the annotation a lot simpler later on.


This code below is an example from a 'combat tag' plugin. It has been annotated for demonstration purposes.
The original code was designed for multi-thread access, so the `volatile` and `synchronised` keywords can be ignored.

```java
public class PlayerTag extends Attachment<Player> {

    // Presumably volatile for cross-thread access
    private volatile int combatCooldown = 0;
    private volatile boolean inCombat = false;

    public PlayerTag(@NotNull Player subject) {
        super(subject);
    }

    public boolean isInCombat() {
        return inCombat;
    }

    public int getCombatCooldown() {
        return combatCooldown;
    }

    // These are presumably synchronised for cross-thread access
 
    public synchronized void setCombatCooldown(int cooldown) {
        this.combatCooldown = cooldown;
    }

    public synchronized void addCombatCooldown(int cooldown) {
        this.combatCooldown += cooldown;
    }

    public synchronized void setInCombat(boolean inCombat) {
        this.inCombat = inCombat;
    }

}
```


How to Register an Attachment
----

Attachments should generally be registered during a plugin's `onEnable()` phase. If you are using your attachment to store/load data within entity NBT, it is advisable to set your plugin to load during `STARTUP` as this will make sure the attachments are registered before any tiles, worlds or entities are loaded.

The simplest method for registering an attachment is in the `org.bukkit.Bukkit` class.
`Bukkit.registerAttachment(@NotNull Plugin plugin, @NotNull Function<? extends T, Attachment<?>> creatorFunction, @NotNull Class<T> target)`

This requires:
 * An instance of your plugin (you can use `this` in your onEnable method)
 * A function to create an instance of the attachment, provided with the AttachableObject
 * The class of the AttachableObject

Referring to the examples seen above, one would register the example attachments using the following methods:
```java
public class MyPlugin implements JavaPlugin {

    @Override
    public void onEnable() {
        Bukkit.registerAttachment(this, PlayerData::new, Player.class);
        Bukkit.registerAttachment(this, PlayerTag::new, Player.class);
    }

}
```

Since both are an `Attachment<Player>` we can use `Player.class` for the AttachableObject type.
If they were `Attachment<Entity>` we would instead use `Entity.class` as this is what our constructor requires.

Because the constructor was a simple `new PlayerData(player)`, the direct method reference for this `PlayerData::new` fits the `Function<? extends T, Attachment<?>> creatorFunction` parameter.

If you have created a more complex constructor (for example `new PlayerData(int i, Player player)` you would need to use an explicit function. For example: `player -> new PlayerTag(10, player)`

Wellspring also includes a way to retroactively register an attachment.

This means that, supposing you /reload the server with a new plugin on (which is probably a bad idea) and the plugin registers an attachment for `Entity.class` - the server can retroactively add the attachment to all existing entities.
This uses an internal weak cache. You will, however, have missed the initial loading phase for the entities, so any NBT changes will not be loaded.


How to Access an Attachment
----

Unlike Bukkit's metadata and persistent data systems, Wellspring's attachments are designed to be very simple and easy to use.
They will always be present on the object by default, meaning that no null-check are required.

```java
public class JoinListener implements Listener {

    @EventHandler
    public void onJoin(PlayerJoinEvent event) {
        Player player = event.getPlayer();
        PlayerData data = player.getAttachment(PlayerData.class); // The attachment will be pre-cast to the type
        data.myBooleanField = true; // Access your (public) fields
        data.myMethod(); // You can call your custom methods
        if (data.isSomething()) { // An example if-statement
            data.setSomething(false);
        }

        if (!data.hasPlayedBefore()) { // An example of something you could do with this.
            player.sendMessage("Welcome! Here's $1000");
            data.setPlayedBefore(true);
            data.setMoney(1000L); // You could
        }
        

        // No need to re-set or save the attachment - your access was direct
    }
}
```

Below is a brief comparison of various methods of storing a boolean value on a player.

#### Using Wellspring
```java 
Player player = event.getPlayer();
PlayerData data = player.getAttachment(PlayerData.class);
data.myBoolean = true;
boolean boo = data.myBoolean; // true
```

#### Using Persistent Data Containers
```java 
Player player = event.getPlayer();
PersistentDataContainer container = player.getPersistentDataContainer();
container.set(new NamespacedKey(plugin, "myBoolean"), PersistentDataType.BYTE, (byte) 1);
boolean boo = container.getOrDefault(new NamespacedKey(plugin, "myBoolean"), PersistentDataType.BYTE, 0) == (byte) 1 ? true : false; // true
```
 
#### Using Metadata
```java 
Player player = event.getPlayer();
List<MetadataValue> list = player.getMetadata("myBoolean");
list.add(new FixedMetadataValue(plugin, true));
boolean boo = false;
for (MetadataValue value : list) {
    if (value.getOwningPlugin() == plugin)
        boo = value.asBoolean();
}
```

As you can see from those examples, attachments are a much shorter and concise option for storing data.
However, __they are not appropriate for every situation__. Since you first need to set up and register the attachment class,
it might well be preferable to use persistent data for small amounts of primitive data.


Advantages and Disadvantages of Attachments
----

#### Advantages
 * You have control over what data to save, and the format
 * You are not limited to basic primitive types
 * You can include other code (methods) within your attachment
 * You do not require a plugin instance to access the attachment
 * You do not require a singleton manager class for extra data
 * The storage in NBT can be smaller than persistent data containers (no pointless keys)
 * Your attachments can be applied to anything sharing a common interface
 * Your code can follow OOP principles (no static singleton manager classes)
 * Attachments are easy to access from other plugins or locations

#### Disadvantages
 * You are required to pre-define what data to store
 * You have to store the data yourself
 * Attachments have to be registered
 * Attachments are easy to edit from other plugins - difficult to control who can access them
 
 
Accessing Attachments from Other Plugins (API)
----

Wellspring's attachment system is designed to be useful as an API-style system.
As long as you are able to see the attachment class, you can access an attachment from any plugin.

This makes attachments a great choice for providing an API for your plugin that can be accessed by other plugins.

For example, if you are making a money/economy plugin and would like other plugins to be able to see the balance of a player:
```java 
Money money = player.getAttachment(MoneyAttachment.class);
int i = money.get();
money.add(100);
if (money.canAfford(1000)) ... ;
```

As you can see, an implementation like this is a lot simpler than the traditional way of creating a singleton API class and forcing plugins to have to obtain an instance of it.

Note: this does not mean your code needs to be put in an attachment - you may simply use the attachment as an access route.
