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
  equipment, quest, varbit, and varp requirements pass. When a chosen route
  needs climbing boots, a crossbow, or a Mith grapple already in the inventory,
  Efficient Walker equips it before moving. Agility shortcuts are enabled by
  default, while grapple shortcuts can be enabled separately in the plugin
  settings.
- Use ordinary object transports gated by a skill level or completed quest when
  the requirement passes. These transitions remain directional, so an eligible
  entrance does not create an unrestricted reverse route.
- Use supported Standard spellbook teleports, teleport tablets, and teleport
  jewellery when their level, quest, rune, item, and destination
  requirements pass. Teleport use is enabled by default and can be disabled in
  the plugin settings.
- Preview routes from scene tiles or the world map with `Dry-run walk`, execute
  them with `Test walk`, display per-floor overlays, or accept a destination
  through `EfficientWalker#walkTo(WorldPoint)`. Integrations can cancel an
  active request and read its current status or planning failure.

### Supported teleports

Efficient Walker considers a teleport automatically when **Use teleportations**
is enabled, the teleport is available to the player, and it improves the route.
For spells, it checks the active spellbook, Magic level, required runes, and
quest unlocks. Tablets must be in the inventory.

| Teleport | Supported source | Destination and requirements |
| --- | --- | --- |
| Varrock Teleport | Spell or Varrock teleport tablet | The spell uses the configured Varrock or Grand Exchange destination. The tablet goes to Varrock. |
| Lumbridge Teleport | Spell or Lumbridge teleport tablet | Lumbridge |
| Falador Teleport | Spell or Falador teleport tablet | Falador |
| Teleport to House | Spell or Teleport to house tablet | Outside the portal for the player's current house location |
| Camelot Teleport | Spell or Camelot teleport tablet | The spell uses the configured Camelot or Seers' Village destination. The tablet goes to Camelot. |
| Kourend Castle Teleport | Spell or Kourend castle teleport tablet | Kourend Castle; requires Client of Kourend |
| Ardougne Teleport | Spell or Ardougne teleport tablet | Ardougne; requires Plague City |
| Civitas illa Fortis Teleport | Spell or Civitas illa Fortis teleport tablet | Civitas illa Fortis; requires Twilight's Promise |
| Watchtower Teleport | Spell or Watchtower teleport tablet | Uses the configured Watchtower or Yanille destination; requires Watchtower |
| Trollheim Teleport | Spell | Trollheim; requires Eadgar's Ruse |
| Ape Atoll Teleport | Spell and one banana | Ape Atoll; requires the King Awowogei subquest of Recipe for Disaster |

### Supported teleport items

Charged jewellery can be in the inventory or equipped unless noted below.
Efficient Walker checks destination unlocks before adding an option to a route.

| Item | Supported teleport options | Requirements and supported variants |
| --- | --- | --- |
| Games necklace | Burthorpe; Barbarian Outpost; Tears of Guthix; Wintertodt Camp | Tears of Guthix requires its quest. Wintertodt Camp requires having visited Zeah. Charged versions are supported. |
| Ring of dueling | Emir's Arena; Castle Wars; Fortis Colosseum | Fortis Colosseum requires Hero rank. Charged versions are supported. |
| Combat bracelet | Warriors' Guild; Champions' Guild; Edgeville Monastery; Ranging Guild | Charged versions are supported. |
| Skills necklace | Fishing Guild; Mining Guild; Crafting Guild; Cooking Guild; Woodcutting Guild; Farming Guild | The Farming Guild requires level 45 Farming. Charged versions are supported. |
| Amulet of glory | Edgeville; Karamja; Draynor Village; Al Kharid | Charged regular and trimmed amulets, plus the amulet of eternal glory, are supported. |
| Ring of wealth | Miscellania; Grand Exchange; Falador Park; Dondakan | Miscellania requires Throne of Miscellania. Dondakan requires Between a Rock. Charged regular and imbued rings are supported. |
| Slayer ring | Stronghold Slayer Cave; Slayer Tower; Fremennik Slayer Dungeon; Tarn's Lair; Dark Beasts; Wyrmscraig Cavern | Slayer Tower requires Priest in Peril, Tarn's Lair requires Haunted Mine, Dark Beasts requires Mourning's End Part II, and Wyrmscraig Cavern requires Fallen from Grace. Charged and eternal rings are supported from the inventory only. |
| Digsite pendant | Digsite; Fossil Island; Lithkren | Fossil Island requires Bone Voyage. Lithkren requires Dragon Slayer II and the pendant destination unlock. Charged versions are supported. |
| Necklace of passage | Wizards' Tower; The Outpost; Eagles' Eyrie; Wyrmscraig | Wyrmscraig requires Fallen from Grace. Charged versions are supported. |

Only the options listed above are supported. Efficient Walker ignores other
destinations offered by these items, including destinations in the Wilderness.

Transport support is live-tested by action and object family. The bundled data
contains more individual transport rows than can reasonably be visited one by
one, so a listed family means representative fixtures have passed rather than
every location having been manually tested.

### Known limitations

- Global routing is limited to the bundled collision map and supported
  transitions. Unmapped areas and global or multi-plane instances are rejected.
- Boats and ferries, NPC transports, payment transports, and general dialogue
  transports are not supported.
- Wilderness routes and teleports that land in the Wilderness are excluded.
- Ordinary item-gated transports remain excluded. Eligible agility shortcuts
  and direct skill- or quest-gated object transports have verified checks.
- Boundary handling supports loaded wall objects with `Open` or `Pass`. Other
  boundary actions require explicit support.
- Unsupported same-plane transports fail closed. Calling `walkTo` reports that
  a request was accepted; movement continues asynchronously and is not proof of
  arrival by itself.

### Use Efficient Walker from a plugin

#### 1. Add the release JAR

Download `efficient-walker-v1.8.2-obf.jar` into your plugin project's `libs`
directory, then add it as a runtime dependency:

```groovy
dependencies {
    compileOnly files('libs/microbot.jar')
    implementation files('libs/efficient-walker-v1.8.2-obf.jar')
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

Use `getStatus()` and `getPlanningFailure()` to inspect the request, or
`cancel()` to stop it.

Do not install multiple plugin JARs that each bundle Efficient Walker at the
same time; each would register its own Efficient Walker plugin.

### Standalone installation

The Efficient Walker release JAR can also run by itself. Place it in
`~/.runelite/microbot-plugins/` and restart Microbot outside safe mode. Do not
sideload it when the client already bundles Efficient Walker.

### Release files

- `releases/efficient-walker/v1.8.2/efficient-walker-v1.8.2-obf.jar`

The JAR has a neighboring `.sha256` checksum file.
