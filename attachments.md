Attachments
-----

Wellspring's attachment system is designed to provide plugin creators with a fast and simple method of linking plugin-provided data to entities.

### Goals of Attachments
 * To provide a simpler and faster system than Spigot's metadata and persistent data containers
 * To follow OOP principles and not force users to rely on singleton maps and manager classes
 * To give developers more freedom about what data to store and how to store it
 * To avoid superfluous null-checks at every access

### Non-goals of Attachments
 * This is not a place to add custom behaviour and tick-actions to entities - please use pathfinder goals instead
 * This is not a place to try and perform illegal NBT edits during entity saving/loading
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
@Override
public void onEnable() {
    Bukkit.registerAttachment(this, PlayerData::new, Player.class);
    Bukkit.registerAttachment(this, PlayerTag::new, Player.class);
}
```

Since both are an `Attachment<Player>` we can use `Player.class` for the AttachableObject type.
If they were `Attachment<Entity>` we would instead use `Entity.class` as this is what our constructor requires.

Because the constructor was a simple `new PlayerData(player)`, the direct method reference for this `PlayerData::new` fits the `Function<? extends T, Attachment<?>> creatorFunction` parameter.

If you have created a more complex constructor (for example `new PlayerData(int i, Player player)` you would need to use an explicit function. For example: `player -> new PlayerTag(10, player)`

Wellspring also includes a way to retroactively register an attachment.

This means that, supposing you /reload the server with a new plugin on (which is probably a bad idea) and the plugin registers an attachment for `Entity.class` - the server can retroactively add the attachment to all existing entities.
This uses an internal weak cache. You will, however, have missed the initial loading phase for the entities, so any NBT changes will not be loaded.


