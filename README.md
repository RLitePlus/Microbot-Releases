# Microbot releases

Downloadable releases for Efficient Walker, an independent pathfinding plugin
for Microbot.

## Efficient Walker

Efficient Walker plans and executes its own routes. It uses Microbot's client
and input APIs, but it does not delegate routing, movement, doors, or transports
to the Microbot walker. Routes come from its bundled collision map, transport
data, and the live RuneLite scene.

### What it can do

- Walk local or long-distance routes and chain supported transitions across
  multiple floors. Scene changes rebuild the remaining route from the player's
  current position instead of restarting the request.
- Detect live doors and gates with `Open` or `Pass`, open them before crossing,
  and exclude a refused boundary after three attempts so an alternate route can
  be used when one exists.
- Use stairs, ladders, trapdoors, climbing ropes, passages, tunnels, cave
  entrances, portals, lift platforms, gangplanks, bridges, rocks, cracks,
  chasms, ropeswings, rubble, shelves, and other proven object transports.
- Use eligible agility shortcuts only when their current skill, item,
  equipment, varbit, and varp requirements pass.
- Preview routes from scene tiles or the world map with `Dry-run walk`, execute
  them with `Test walk`, display per-floor overlays, or accept a destination
  through `EfficientWalker#walkTo(WorldPoint)`.

Transport support is live-tested by action and object family. The bundled data
contains more individual transport rows than can reasonably be visited one by
one, so a listed family means representative fixtures have passed rather than
every location having been manually tested.

### Known limitations

- Global routing is limited to the bundled collision map and supported
  transitions. Unmapped areas and global or multi-plane instances are rejected.
- Teleports, boats and ferries, NPC transports, payment transports, and general
  dialogue transports are not supported. Stronghold security-door dialogue is
  the explicit exception.
- Ordinary quest-, skill-, and item-gated transports remain excluded. Eligible
  agility shortcuts and Stronghold portals have their own verified checks.
- Boundary handling supports loaded wall objects with `Open` or `Pass`. Other
  boundary actions require explicit support.
- Unsupported same-plane transports fail closed. Calling `walkTo` reports that
  a request was accepted; movement continues asynchronously and is not proof of
  arrival by itself.

### Use Efficient Walker from a plugin

#### 1. Add the release JAR

Download `efficient-walker-v1.4.2-obf.jar` into your plugin project's `libs`
directory, then add it as a runtime dependency:

```groovy
dependencies {
    compileOnly files('libs/microbot.jar')
    implementation files('libs/efficient-walker-v1.4.2-obf.jar')
}
```

The Efficient Walker JAR must be included in the plugin JAR you distribute.
If your build does not already create a fat JAR, Gradle's standard `jar` task
can merge the runtime dependency:

```groovy
jar {
    duplicatesStrategy = DuplicatesStrategy.EXCLUDE
    from {
        configurations.runtimeClasspath.collect { zipTree(it) }
    }
}
```

#### 2. Declare and inject the dependency

```java
@PluginDependency(EfficientWalkerPlugin.class)
public class MyPlugin extends Plugin
{
    @Inject
    private EfficientWalker walker;
}
```

`@PluginDependency` makes RuneLite load Efficient Walker before the consuming
plugin and lets Guice provide the same `EfficientWalker` instance.

#### 3. Walk

```java
walker.walkTo(new WorldPoint(3206, 3229, 2));
```

`walkTo` returns whether the request was accepted. Walking then progresses on
game ticks until arrival or failure. Efficient Walker must remain enabled while
the route is running.

Do not install multiple plugin JARs that each bundle Efficient Walker at the
same time; each would register its own Efficient Walker plugin.

### Standalone installation

The Efficient Walker release JAR can also run by itself. Place it in
`~/.runelite/microbot-plugins/` and restart Microbot outside safe mode. Do not
sideload it when the client already bundles Efficient Walker.

### Release files

- `releases/efficient-walker/v1.4.2/efficient-walker-v1.4.2-obf.jar`

The JAR has a neighboring `.sha256` checksum file.
