# w3ts-w3mmd
A w3mmd library for WC3 TypeScript maps.

## Installation
Install as a codependency (not devDependency) of `w3ts`:
```
npm i -S w3ts w3ts-w3mmd
```

## Usage
### setPlayerFlag
Sets a flag for a player.
```ts
import { setPlayerFlag } from "w3ts-w3mmd";

setPlayerFlag(Player(0), "winner");
setPlayerFlag(Player(1), "loser");
```

### defineEvent
Defines a w3mmd event. The second argument is the format, where the nth
argument (0-indexed) is formatted `{n}`.

Returns a callback used to log an instance of the event.
```ts
import { defineEvent } from "w3ts-w3mmd";
import { Trigger } from "w3ts";

const logKill = defineEvent("kill", "{0} killed {1}", ["killer", "victim"]);

new Trigger()
  .registerAnyUnitEvent(EVENT_PLAYER_UNIT_DEATH)
  .addAction(() =>
    logKill(
      GetOwningPlayer(GetKillingUnit()),
      GetOwningPlayer(GetDyingUnit()),
    ),
  );
```

### defineStringValue
Defines a string value. Values can be updated throughout the game.

Returns a callback to set the value.
```ts
import { defineStringValue } from "w3ts-w3mmd";

const setHero = defineStringValue("hero");

setHero(Player(0), "Archmage");
setHero(Player(1), "Mountain King");
```

### defineNumberValue
Defines a numeric value. Values can be updated throughout the game. Numeric
values default to integers, but can also be reals (floats).

Returns a callback to set (default), increase, or decrease the value.
```ts
import { defineNumberValue } from "w3ts-w3mmd";
import { Trigger } from "w3ts";

const incKills = defineNumberValue("kills", "high");
const setBonus = defineNumberValue("bonus", "none", "none", "real");

// assuming Player( 0 ) wins; we'd want to reduce bonus if they lost
setBonus(Player(0), 2 - GetPlayerHandicap(Player(0)));

new Trigger()
  .registerAnyUnitEvent(EVENT_PLAYER_UNIT_DEATH)
  .addAction(() => incKills(GetOwningPlayer(GetKillingUnit()), 1, "add"));
```

### emitCustom
Emits some custom w3mmd data. This would be used by a specialized parser.
```ts
import { emitCustom } from "w3ts-w3mmd";

emitCustom(
  "build",
  compiletime(() => new Date().toISOString()),
);
```

## Initialization

By default, `w3ts-w3mmd` will automatically emit w3mmd init messages, including
the w3mmd version and player names. This behavior can be prevented by calling
`stopInit`. This will prevent all emits until `init` is called.
```ts
import { stopInit, init } from "w3ts-w3mmd";

stopInit();

const afterPlayerNamesNormalized = () => {
  init();
};
```

Explicit handling of init emits can be done by using `emit`, `pack`, and
`flagReady`:
```ts
import { emit, flagReady, pack, stopInit } from "w3ts-w3mmd";

stopInit();

const customInit = () => {
  emit("init version 1 1");

  for (let i = 0; i < bj_MAX_PLAYERS; i++) {
    if (
      GetPlayerSlotState(Player(i)) === PLAYER_SLOT_STATE_PLAYING &&
      GetPlayerController(Player(i)) === MAP_CONTROL_USER
    ) {
      emit(`init pid ${i} ${pack(anonymize(Player(i)))}`);
    }
  }
};
```
