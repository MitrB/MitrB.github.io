---
layout: post
title:  "How to implement data driven abilities: exploring SpacetimeDB"
date:   2025-10-19 20:00:43 +0100
categories: game design 
---

When [first](https://www.youtube.com/watch?v=yctM7oTLurA) hearing about [SpacetimeDB](https://spacetimedb.com/) I was very intrigued. I didn't have any experience creating backends for MMOs but I did have experience creating smaller multiplayer applications. The promise of efficiently running application logic on the database seemed too good to be true. Most importantly, I had a thought many will relate to "That seems easy to do! I should try it!".

First, I will shortly explain what SpacetimeDB does for you as a developer and designer in a very practical way. Then I will go over how I worked with it and the systems I developed that interact with it, focusing on the abilities system. Oh yeah... I should mention that this is about how I created data-driven abilities for a MOBA in SpacetimeDB.

## SpacetimeDB's design

Apart from being convenient to set up, let's look at what we're actually doing. SpacetimeDB can be seen as essentially a database that runs your game logic. The main idea behind Spacetime's design is that all persistent game data is defined in the backend as database tables. The client subscribes to these tables to receive updates and represent the game state. When a player takes an action, the client calls a reducer (similar to an API endpoint), which runs game logic and potentially modifies the backend data. Since all clients are subscribed to the relevant tables, any changes are automatically synced to every connected client in real-time.

This creates a simple design architecture. The client is just representing the data and the possible actions the user can take, while the backend does all the calculations and game logic.

## Why MOBA

For people who are not familiar with the term [MOBA](https://en.wikipedia.org/wiki/Multiplayer_online_battle_arena) (Multiplayer Online Battle Arena), think games like League of Legends, Dota... To keep it short, a MOBA is a game where usually 2 teams of 5 fight until one comes out victorious. Usually there are player controlled characters with unique **abilities** that play a unique role. And that is what we are going to focus on for this blog. The **abilities**.

I wanted to create a MOBA game with the idea that my friends and I could play it and iterate on it. Most importantly, I wanted to be able to quickly tweak character abilities and stats. If I had to rewrite parts of an ability, that would become annoying very quickly. So I began to search for solutions and tried some ideas that I had initially. I looked at what the industry standard was and found this interesting Dota [blog](https://developer.valvesoftware.com/wiki/Dota_2_Workshop_Tools/Scripting/Abilities_Data_Driven). This post greatly inspired my data-driven abilities system.

## Data-Driven Abilities

The idea behind data-driven abilities is that your ability entity is defined as a data object, which doesn't execute any logic. You create component scripts which handle specific parts of what an ability should do. This sounds like an [Entity Component System](https://en.wikipedia.org/wiki/Entity_component_system)! I ended up creating a system that feels similar to an ECS, but definitely limited to handling abilities.

An ability might be a projectile that does magic damage. Okay, we can define the ability as an entity with a projectile data component. On the backend, we also provide a script that handles the projectile's movement and damage. But what if we want the projectile to create an explosion on impact? What if we want it to split into more projectiles? Do we create another component? An explosive projectile component? A split-projectile component? We can see that this leads us to bad design.

What I decided to do is to define abilities as directional graphs. The nodes are components that should be executed and the edges are conditionals that may or may not spawn entities which they are pointing to. Let's consider an ability that fires a projectile, then if it hits something, spawns 3 more projectiles. We can define it in a graph as follows:

![ProjectileSplit](/assets/DataDrivenAbilities/projectilesplit.png)

You can understand that if we want to change the behavior of this ability, it is very easy to do so. We can adjust the number of projectiles, or maybe we don't want it to split into three. Or maybe we want the split to happen regardless when the first projectile is done with an OnEnd trigger.

Abilities in this system have a common data definition that holds information such as mana cost, ability name, etc. The nodes are entity-component definitions, both holding data and, by virtue of being a node of a certain type, defining its behavior (which is scripted in the backend).

Now that you understand the concept, let's see how this translates to actual implementation in SpacetimeDB.

## SpacetimeDB specifics

### Table definitions

Let's look at how I implemented this for my game. First, we need to declare our abilities. I wrote the backend in C# because I was yet unfamiliar with Rust (which has changed). So I'll keep to C#-like code blocks. Let's look at the table definitions for the abilities system. I also added comments to explain some fields.

```C#
[Table(Name = "ability_definition", Public = true)]
public partial struct AbilityDefinition
{
    [PrimaryKey]
    public string ability_id; // unique identifier (SpacetimeDB supported)
    public string ability_name;
    public string description;
    public float mana_cost;
    public float cast_time;
    public float cooldown;
    public float range; // range within you can cast this ability
}
```

Here is the node definition. The *SpacetimeDB.Index.BTree* definition is for an external key for easy lookup. 

```C#
[Table(Name = "node_definition")]
[SpacetimeDB.Index.BTree(Name = "AbilityNode", Columns = [nameof(ability_id), nameof(node_id)])]
public partial struct NodeDefinition
{
    [PrimaryKey, AutoInc]
    public uint id;
    public string ability_id;
    public uint node_id;
    public string component_type; // this defines what script is used to execute this node
    public string component_id; // specific id for data
}
```

To represent the edges I define "Watchers". Think of watchers as conditional connections between nodes. A watcher sits on a node and waits for a specific event (like "OnHit" or "OnEnd"). When that event occurs, the watcher spawns the next node in the graph. This is how we create ability chains like when a projectile hits something, triggers its "OnHit" watcher, which spawns an explosion node.

```C#
[Table(Name = "watcher_definition")]
[SpacetimeDB.Index.BTree(Name = "AbilitySource", Columns = [nameof(ability_id), nameof(source_node_id)])]
public partial struct WatcherDefinition
{
    [PrimaryKey, AutoInc]
    public uint id;
    public string condition; // Conditions that trigger this watcher to spawn a node. [OnHit, OnStart, OnEnd]
    public string ability_id;
    public uint source_node_id;
    public uint target_node_id;
}
```

Let's also look at a simplified projectile table definition.

```C#
[Table(Name = "projectile_component")]
public partial struct ProjectileComponent
{
    [PrimaryKey]
    public string id;
    public float value;
    public float speed;
    public float radius;
    public float duration;          // max lifetime
    public string movement_type;    // follow_terrain, from_to
    public string spawn_type;       // on_caster, on_target_position
    public bool destroy_on_hit;
    public float spawn_offset;
}
```
### Scripting

To give a quick overview: all abilities have a virtual node with index 0. This is the root node of the graph. When casting an ability, "OnStart" is triggered on the root node, which triggers any watchers that are looking for this condition. This means that in reality we will be defining abilities as such:

![RootShow](/assets/DataDrivenAbilities/RootShow.png)

Let's take a look at pseudocode that represents the reducer that the client would call if it wanted to cast an ability:

```
function AbilityCast(player, entity_id, ability_id, target_position):
    ability = database.find_ability(ability_id)
    caster = database.find_entity(entity_id)
    caster_stats = get_character_stats(entity_id)

    if (conditions not valid):
        return error("Invalid ability")

    caster.mana -= ability.mana_cost
    database.add_cooldown(entity_id, ability_id, ability.cooldown)

    active_ability = GenerateWatchers(ability_id, entity_id, target_position)

    TriggerWatchers(active_ability, 0, "OnStart")
```

Most important to note is that we generate all watchers here and then call the trigger on the root node with condition "OnStart". Here you can see pseudocode for the GenerateWatchers and TriggerWatchers methods.

```
function GenerateWatchers(ability_id, active_ability_id, source_node_id):
    watchers = database.find_watchers(ability_id, source_node_id)

    for each watcher_definition in watchers:
        watcher = create_watcher(
            condition: watcher_definition.condition,
            ability_id: active_ability_id,
            source_node_id: source_node_id,
            target_node_id: watcher_definition.target_node_id
        )
        add_to_active_watchers(active_ability_id, watcher)
```

```
function TriggerWatchers(ability_id, source_node_id, condition):
    if ability has no active watchers:
        return

    active_ability = get_active_ability(ability_id)
    caster = database.find_entity(active_ability.caster_id)

    if caster does not exist:
        return

    for each watcher in active_watchers:
        if watcher matches (ability_id, source_node_id, condition):
            target_nodes = database.find_nodes(ability_id, watcher.target_node_id)

            for each node in target_nodes:
                component = create_component_from_node(node, active_ability, caster)

                if component created successfully:
                    add_active_component(ability_id, component)
                    generate_watchers(ability_id, node.node_id)
```

The rest of the system are methods and game loops that track active abilities and their nodes. If a node exists, its function will be executed by the corresponding script. Each component type has its own update function that gets called every frame. Here's an example of a projectile component's update logic:

```
function ProjectileComponent.Update(delta_time, active_ability):
    hit_volume = database.find_entity(this.hit_volume_id)

    if hit_volume does not exist OR duration expired:
        delete_hit_volume()
        return

    current_position = hit_volume.position
    target_position = active_ability.target_position

    if close_to_target AND end_on_target:
        delete_hit_volume()
        TriggerWatchers(active_ability.id, this.node_id, "OnHit")
        return

    duration -= delta_time

    direction = normalize(target_position - current_position)
    new_position = current_position + direction * speed * delta_time

    if movement_type == "follow_terrain":
        new_position = snap_to_terrain(new_position)

    hit_volume.position = new_position
    database.update_entity(hit_volume)
```

With the core system in place, the next question is: how do we actually create these abilities without writing code?

## Actually creating abilities (aka writing JSON files)

To define abilities, we just need a data format to represent them. I chose JSON because, as you will see in the next part, I was using an admin client to update the game. This way I can change ability behavior and data mid-game by rearranging and changing nodes.

I soon realized I needed some kind of editor. So this application was born:

![EditAppCore](/assets/DataDrivenAbilities/editAppCore.png)

It supports defining new node/component types by specifying their name and fields. This way I can easily create a new ability type. Of course, I will need to add a script that can handle it too.

![Editor](/assets/DataDrivenAbilities/definingTypes.png)

In this part I can glue the ability together. You can see the graph being reflected on the right side.

![Editor](/assets/DataDrivenAbilities/EditApp.png)


## Updating SpacetimeDB (aka reading JSON files) Even if it's not perfect.

I wrote reducers for admin features. One of them is, of course, updating the abilities. It's pretty simple: the JSON file gets sent to SpacetimeDB and it creates records for Abilities, Nodes, and Watchers.

## Conclusion

I quite liked my first experience with SpacetimeDB. I'm still finishing the project and there are other design aspects I could write about. But this is one system that I am proud of. Even if it's not perfect.
