# Decisions
This collection of decisions is intended to be used for future reference. It was formed from decisions made during development and extended with design questions wich arose from the community. In a sense it could be used as a faq for design questions.

## How does the engine start the computation of systems?
The engine starts the computations via the event bus. Systems subscribe on events like `EngineEvent.Computation` or `EngineEvent.Render` which in return get called by the engine. It is the systems responsibility to subscribe to relevant event. This results in two major advantages: easy activation and deactivation of systems as well as the possibility to implementation a general purpose engine.

## Can entities share components?
If the player enters another part of the world (aka changes scenes) will he be the same entity?

If not it would be convenient for those two entities to share some components like maybe the health component, inventory etc.

On the other hand if he continues to be the same entity, the entity has to be shared between systems of different scenes. Thought this is not as much of a problem since entities are uniquely identifiable by definition.

### Conclusion
Entities cannot share components. Components are value objects like integers. And as with integers changes to copies do not apply to the original.


### Related decisions
Can scenes share systems?
Can systems handle multiple registrations lf  entity?

## Can scenes share systems?
If the player enters another part of the world (aka changes scenes) will his entity (provided it stays the same) switch instances of systems?

If systems are shared (reused) the same system would have to be able to handle multiple entries for the same entity (see "Can systems handle multiple registrations lf  entity?").

### Conclusion
Sharing systems between scenes is extremely unpractical when it comes to activation and deactivation of scenes them self and thus will not be explicitly supported nor recommended.

### Related decisions
Can entities share components?
Can systems handle multiple registrations lf  entity?

## Can systems handle multiple registrations of the same entity?
The entity is in some kind of superposition (two places at once). The two superposition share some properties (components).
The entity could register two translate components with possibly the same render component (different position, same view model) to the render system. It would have to do the same with many other system as well (damage system etc).

This could be solved with "component bags". Basically systems wouldn't act on entities anymore but on these bags of components. Further more the components in these collection do not have to belonging to the same entity.

Implementing this with two entities would ether not be supported (see "Can entities share components?") or would require some kind of superposition system which synchronizes the components.

### Conclusion
This is not possible due to serious issues encountered during implementation. Namely would entities have to be able to hold the same component twice and know which one is which. In addition it leads to a potential break of the `registerEntity` interface on systems. Further more the expected behavior of the `deregistratEntity` method would also be ambiguous.

### Related decisions
Can entities share components?
How do entities and systems connect?
Can scenes share systems?

## How do entities and systems connect?
If systems can handle multiple registrations of the same entity (see "Can systems handle multiple registrations of the same entity?") the act of registration isn't as straight forward anymore.

The base implementation of the system requests the creation of a cache (or components bag) from the registered entity. The entity is in full control of this cache and thus capable of creating differing caches for repeated registrations. However entities would struggle to identify what the expected behavior is.

### Conclusion
The currently implemented interface supports all necessary use cases. It is also capable of further expansion in future minor version updates.

### Related decisions
Can systems handle multiple registrations of the same entity?

## Should the engine control everything?
Should it be possible to access the whole game via the engine?

``` typescript
engine.addScene(scene);
engine.getActiveScenes();
engine.addEntity(scene.getId(), systemType, entity);
engine.getEntity(entityId);
```

This could be very convenient for the developers but goes against the single responsibility principal. Preferably this would be implemented in a separated structure, in its own class. This class, lets call it game, could be used as follows.

``` typescript
// this already creates an engine including event bus
let game = new Game();

// adds inter-scene entities to the game
game.addEntity(PlayerFactory.create(000));

// creates a scene
game.addScene(new DesertScene(100));
// add systems
game.addSystem(100, new RenderSystem(game.getEventBus()));
game.addSystem(100, new PlayerSystem(game.getEventBus()));
game.addSystem(100, new DoorSystem(game.getEventBus()));
// create and add entities
game.registerEntity(100, RenderSystem, 000);
game.registerEntity(100, PlayerSystem, 000);
game.addEntity(PyramidFactory.create(101, ...));
game.registerEntity(100, RenderSystem, 101);
game.addEntity(DoorFactory.create(102, ...));
game.registerEntity(100, RenderSystem, 102);
game.registerEntity(100, DoorSystem, 102);

// next scene
...

// and finally
game.activateScene(100);
game.start();

```

### Conclusion
This is officially approved.
