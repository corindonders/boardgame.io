# Plugins

The Plugin API allows you to provide off-the-shelf objects
that add custom functionality to [boardgame.io](https://boardgame.io/).
You can create wrappers around moves, provide custom interfaces
in `G` and much more.

#### Creating a Plugin

A plugin is an object that contains the following fields.

```js
{
  // Optional.
  // Function that accepts a move / trigger function
  // and returns another function that wraps it. This
  // wrapper can modify G before passing it down to
  // the wrapped function. It is a good practice to
  // undo the change at the end of the call.
  fnWrap: (fn) => (G, ctx, ...args) => {
    G = preprocess(G);
    G = fn(G, ctx, ...args);
    G = postprocess(G);
    return G;
  },

  // Optional.
  // Called during setup in order to add state to G.
  setupG: (G, ctx) => G,

  // Optional.
  // Called during setup in order to add state to ctx.
  setupCtx: (ctx) => ctx,
}
```

#### Adding Plugins to Games

The list of plugins is specified in the game spec.

```js
import { PluginA, PluginB } from 'boardgame.io/plugins';

Game({
  name: 'my-game',

  moves: {
    ...
  },

  plugins: [PluginA, PluginB],
})
```

!> Plugins are applied one after the other in the order
that they are specified (from left to right).

#### Available Plugins

**PluginPlayer**

```js
import { PluginPlayer } from 'boardgame.io/plugins';

function PlayerState(playerID) {
  return { ... };
}

Game({
  plugins: [PluginPlayer(PlayerState)],
})
```

`PluginPlayer` makes it easy to manage player state.
It creates an object `G.players` that
stores state for individual players:

```
G: {
  players: {
    '0': { ... },
    '1': { ... },
    '2': { ... },
    ...
  }
}
```

The initial values of these states are determined by the parameter
`PlayerState`, which is a function that returns the state for a
particular `playerID`.

Before each move, the plugin makes the state associated with the
current player available at `G.player`. If this is a 2 player game,
then it also adds `G.opponent`. These fields can be modified and the
corresponding entries in `G.players` will be updated at the end of the move.

```
// The current player is '0'.

G: {
  player: { ... }  // copy of players['0']
  opponent: { ... }  // copy of players['1']

  players: {
    '0': { ... },
    '1': { ... },
  }
}
```
