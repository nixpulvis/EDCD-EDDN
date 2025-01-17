# EDDN CodexEntry Schema

## Introduction
Here we document how to take data from an ED `CodexEntry` Journal Event and
properly structure it for sending to EDDN.

Please consult [EDDN Schemas README](./README-EDDN-schemas.md) for general 
documentation for a schema such as this.

## Senders
The primary data source for this schema is the ED Journal event `CodexEntry`.

### Elisions
You MUST remove any key where the key name ends in 
`_Localised`.

You MUST remove the two keys `IsNewEntry` and `NewTraitsDiscovered`.

### Augmentations
#### horizons flag
You SHOULD add this key/value pair, using the value from the `LoadGame` event.

#### odyssey flag
You SHOULD add this key/value pair, using the value from the `LoadGame` event.

#### StarPos
You MUST **add** a `StarPos` key with value of type `array` containing the 
galaxy co-ordinates of the system.  You will need to have obtained these 
from prior event(s) upon the player arriving, or logging into, the system.

e.g. if the system is `Alpha Centauri`:
```json
    "StarPos": [3.03125, -0.09375, 3.15625]
```

#### BodyID and BodyName
You SHOULD attempt to track the BodyName and BodyID where the player is 
and add keys/values for these.

You MUST track `BodyName` both from Status.json *and* also from some
[Journal](./README-EDDN-schemas.md#journal-files)
events in order to cross-check it before using the `BodyID` from 
[Journal](./README-EDDN-schemas.md#journal-files) events.

The following is correct as of game version 4.0.0.801 (Odyssey initial 
release, Update 7, plus one patch).

1. Record `journal_body_name` and `journal_body_id` from the 
   `BodyName` and `BodyID` values in `ApproachBody` events.

   This will occur when the player flies below Orbital Cruise altitude 
   around a body.
2. Also record these from `Location` events to cover logging in already there.
3. Unset both `journal_body_name` and `journal_body_id` on `LeaveBody` and
   `FSDJump` events.
   Do NOT do so for `SupercruiseEntry`, because a player can enter supercruise
   below max Orbital Cruise altitude and then come back down without a new 
   `ApproachBody` event occurring.
4. If Status.json has `BodyName` present, record that as `status_body_name`.  

   This key and its value will be present whenever the player comes close 
   enough to a body for the Orbital Cruise/Glide HUD elements to appear.
   It will disappear again when they fly back above that altitude, or jump 
   away.
5. If Status.json does **not** have `BodyName` then clear `status_body_name`.
6. For a `CodexEntry` event:
    1. Check that `status_body_name` is set. If it is not, exit.
        1. Set the EDDN `codexentry` schema message `BodyName` to this value.
        2. Check if it matches the `journal_body_name` value, and 
           ONLY if they match, set `BodyID` in the EDDN `codexentry`
           schema message to the value of `journal_body_id`.
    
   If `status_body_name` is not set then you MUST NOT include `BodyName` or
   `BodyID` keys/values in the EDDN message.

   If `status_body_name` is set, but does not match with 
   `journal_body_name` then you MUST NOT include a `BodyID` key+value in the
   EDDN message.

   For emphasis, in both of these cases you MUST NOT include the keys with a 
   `null`, `''`, or otherwise 'empty' value.  Do not include the key(s) at all.

One possible issue is binary bodies where you might get an `ApproachBody` for
one before descending towards the other, without an additional `ApproachBody`
to correct things.

An example of this is `Baliscii 7 a` and `Baliscii 7 b`.  Approaching one 
and going below Orbital Cruise altitude will set `journal_body_name` and 
`journal_body_id` to it, but you can then turn and approach the other 
without a new `ApproachBody` event, but `status_body_name` will change to 
the other when you are close enough.

In this case due to `status_body_name` and `journal_body_name` not matching 
the `codexentry` message MUST be sent **without**  `BodyID`, but SHOULD be 
sent with the `status_body_name` value on the `BodyName` key.

e.g. for `Bestia A 2 a`
```json
    "BodyName": "Bestia A 2 a",
    "BodyID": 15,
```

If you cannot properly obtain the values for `BodyName` or `BodyID` then 
you MUST NOT include them.

## Receivers

As per ['BodyID and BodyName'](#bodyid-and-bodyname) above be aware that 
you are not guaranteed to receive these values for any given event.  Some 
codex entries will be in space, and thus they aren't even relevant.  In 
other cases it may not have been possible to properly determine both of them.

So you might receive any of:

1. Neither `BodyName` nor `BodyID` present in the message, not even the 
   key names.  This SHOULD indicate a codex entry object which is not on a 
   body  surface.
2. `BodyName` key present with a value, but no `BodyID` key.  This SHOULD
   indicate a codex entry object which is on a body surface, but probably 
   where there is a close-orbiting binary companion which has confused things.
3. Both `BodyName` and `BodyID` keys present, with values.  This SHOULD 
   indicate a codex entry object which is on a body surface.

Adjust your local processing accordingly.