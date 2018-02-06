# Plugin Components
Plugin components are plugins which are not designed to be run on their own but instead used in other plugins to make development easier.

## Contents
 + [Using plugin components](#using-plugin-components)
 + [Components](#components)
    + [Player Tracker](#player-tracker)

## Using plugin components
To utilize a plugin component, the `PluginManager` class can be used. The `PluginManager` class provides some helpful functions to get managed instances of the plugin components which can then be used in other plugins. The `PluginManager` class is imported from the `plugin-module`.
```typescript
import { NrPlugin, HookPacket, Packet, PacketType, Client, PluginManager } from './../core/plugin-module';
//                                                         ^^^^^^^^^^^^^
@NrPlugin({ name: 'Hello plugin', author: 'tcrane' })
class HelloPlugin {

}
```
To use a plugin component from another plugin, that component has to be imported as well
```typescript
import { NrPlugin, HookPacket, Packet, PacketType, Client, PluginManager } from './../core/plugin-module';
import { PlayerTracker } from './player-tracker';
```
The best way to get a reference to the plugin component is to use the `PluginManager` helper methods inside of the plugin's `constructor`

```typescript
import { NrPlugin, HookPacket, Packet, PacketType, Client, PluginManager } from './../core/plugin-module';
import { PlayerTracker } from './player-tracker';

@NrPlugin({ name: 'Hello plugin', author: 'tcrane' })
class HelloPlugin {

    private playerTracker: PlayerTracker;

    constructor() {
        PluginManager.afterInit(() => {
            this.playerTracker = PluginManager.getInstanceOf(PlayerTracker);
        });
    }
}
```
Using the `PluginManager.afterInit(...)` method ensures that the component you are trying to load has been fully initialised. Trying to invoke `PluginManager.getInstanceOf(...)` without using `afterInit` could result in an error being thrown if the plugin loads before the component does.

## Components
This is a list of all currently available components.

## Player Tracker
To utilise the player tracker, first import it and get a reference to it.
```typescript
import { PlayerTracker } from './player-tracker';

@NrPlugin({ name: 'Example plugin', author: 'tcrane' })
class ExamplePlugin {
    
    private playerTracker: PlayerTracker;

    constructor() {
        PluginManager.afterInit(() => {
            this.playerTracker = PluginManager.getInstanceOf(PlayerTracker);
        });
    }
}
```
The methods available in the player tracker component are
#### `trackPlayersFor(client: Client): void`
Call this method and pass the client you want to start tracking players for. Calling this for a client which already has player tracking enabled will produce a warning but will not cause an error.

#### `getPlayersFor(client: Client): IPlayerData[] | null`
Calling this method will return an array of all players which are tracked by the client. If the client does not having tracking enabled, then `null` will be returned. If tracking is enabled, but there are no players to track, then and empty array will be returned.
### Example usage:

```typescript
import { PlayerTracker } from './player-tracker';

@NrPlugin({ name: 'Example plugin', author: 'tcrane' })
class ExamplePlugin {
    
    private playerTracker: PlayerTracker;

    constructor() {
        PluginManager.afterInit(() => {
            this.tracker = PluginManager.getInstanceOf(PlayerTracker);
            Client.on('connect', (pd, client: Client) => {
                this.tracker.trackPlayersFor(client);
            });
        });
    }

    @HookPacket(PacketType.NEWTICK)
    onNewTick(client: Client, newTick: NewTickPacket): void {
        const trackedPlayers = this.playerTracker.getPlayersFor(client);
        Log('Plugin', 'Currently tracking ' + trackedPlayers.length + ' players.');
    }
}
```