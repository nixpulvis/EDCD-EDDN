# EDDN Commodity Schema

## Introduction
Here we document how to take data from an ED `Market` Journal Event and
properly structure it for sending to EDDN.

Please consult [EDDN Schemas README](./README-EDDN-schemas.md) for general
documentation for a schema such as this.

## Senders
The primary data source for this schema is the ED Journal event `Market`, 
and the additional file, `Market.json`, that it signals the writing of.

So, look for the `Market` event, and when it occurs open and read the 
`Market.json` file which contains the actual data.  Treat *that* data as 
the event.

It *is* also possible to construct this data from a CAPI `/market` query.  
See [Using CAPI data](#using-capi-data) below.

### Statusflags
`statusFlags` is an optional augmentation that can be used to identify
`Producer`, `Consumer`, and `Rare` goods from the output of `Market.json`.  
So, as per the schema, do include it if available.

### Key Renames
Many of the key names have a different case defined in this schema, make 
sure you are renaming them as appropriate.

#### StarSystem to systemName
Rename the `StarSystem` key name to `systemName`.

### Elisions
#### Remove _Localised key/values
All keys whose name ends with `_Localised`, i.e. the `Name_Localised`
key/values in Items.

#### Other Elisions
You MUST remove the following key/value pairs from the data:

- `StationType` key/value.
- `Producer` key/value pair in Items.
- `Rare` key/value pair in Items.
- `id` key/value pair in Items.

In the list of commodites:

- Skip commodities with `"categoryname": "NonMarketable"` (i.e.
  Limpets - not purchasable in station market) or a *non-empty*`"legality":` 
  string (not normally traded at this station market).

#### Item Category
Remove not only the `Category_Localised` key/value, as above, but also the
`Category` key/value pair from each Item.

### Augmentations
#### horizons flag
You SHOULD add this key/value pair, using the value from the `LoadGame` event.

#### odyssey flag
You SHOULD add this key/value pair, using the value from the `LoadGame` event.

### Using CAPI data
It is *not* recommended to use CAPI data as the source as it's fraught with 
additional issues.  EDMarketConnector does so in order to facilitate 
obtaining data without the player needing to open the commodities screen.

Please read
[the guidance on checking for CAPI lag](README-EDDN-schemas.md#detecting-capi-data-lag)
before utilising CAPI data for EDDN messages.

Note that CAPI `/market` data will sometimes have the `StatusFlasg` per 
item, which are defined as optional in this schema (because they're not in 
the Market.json data).  You SHOULD include this data in your message if 
using CAPI as the source.

Now you will need to construct the necessary additional fields:

#### CAPI horizons flag
If your application can be certain that the game client is still running, 
and logged into the game (not just run to the main menu), then you can 
simply use the value from the `LoadGame` journal event.

Otherwise, you MUST check if any of the economies from the `/market` 
data have a `name` of `Colony`, if so, set this flag true.

Additionally, you should retrieve the CAPI `/shipyard` endpoint and check if 
any of the listed ships or modules have a `sku` value of 
`ELITE_HORIZONS_V_PLANETARY_LANDINGS`.  If so, set this flag true.

#### CAPI odyssey flag
Unfortunately there is no method to be *certain* of this from CAPI data, so 
you will have to trust in the system/station name check and use the value 
from the Journal `LoadGame` event.