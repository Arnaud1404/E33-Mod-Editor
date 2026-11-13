# Running Expedition Infinity with the coop mod

Notes on playing an Infinity seed together with Kouzukii's
[Expedition 33 coop mod](https://github.com/Kouzukii/expedition33-coop)
(3 local players). Checked against Infinity 1.3.0 packs; the 1.3.3+ sources
here patch the same assets, but that combination has not been played.

## Where the two mods meet

Coop hooks 8 game blueprints with `ue-hook-blueprints`: the original function
`X` is renamed `orig_X` and the hook takes the name `X`. Only one of them is
also shipped by Infinity:

| asset | Infinity changes | coop hooks |
|---|---|---|
| `jRPGTemplate/Blueprints/Basics/BP_jRPG_Character_Battle_Base` | `GetBattleSpotIndexLeftToRight_(WhenFacingCharacters)` (player table `[1,0,2]` to `[1,0,2,2,1]`) | 8 other functions (turn ownership, defense inputs) |

Whichever pack mounts last wins the whole asset. Coop builds with the
`infinity-compat` changes ship that asset with a 5-entry player table
(`[1,0,2,0,1]`) and mount after Infinity (`expedition33_coop_9_P`), so both
packs can sit in `~mods` together. Older coop releases (`expedition33_coop_P`)
lose either Infinity's table or coop's hooks.

## Party slots and battle spots

Coop maps players to characters by battle spot. Up to 0.9.6, its `ModLibrary`
(`MapBattleSpotToUser`, `MapUserToBattleSpots`) hardcodes the vanilla
party-slot to battle-spot table `[2,0,1]`. With the Party of Five table
`[2,0,3,1,4]` from `RemapPartyIndexToBattleSpotIndex`, the third member stands
on spot 3, which that table does not contain, and the fourth member stands on
spot 1, which coop gives to the third member's player.

The `infinity-compat` changes read the table from
`RemapPartyIndexToBattleSpotIndex` instead and add assignment settings for
party slots 4 and 5 (default: player 1).

## Load order

- The coop pack must mount after the Infinity pack. A pack name ending in
  `_9_P` gets a higher mount order than Infinity's `_P` packs.
- Other party-size mods (`PartyMax`, `0000_Party_Of_Four_P`) ship their own
  copies of the battle blueprint and the function library and cannot be
  combined with this setup.

## Known limits

- Spots 3 and 4 read `0` and `1` from coop's player table where Infinity uses
  `2` and `1`, so the 4th member gets the rightmost free-aim limits instead of
  the leftmost ones.
- Parry and dodge windows: coop's per-player defense window multiplies
  `ImmuneDuration`, so Infinity's window changes still apply.
- 4 or more human players: not supported by coop (3 local players).
