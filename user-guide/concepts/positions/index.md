# Positions
!!! example "TODO"

    This page should explain positions and how they are implemented by kloppy.

In soccer, while players are not bound by strict rules about where they must be positioned on the field, there are traditionally recognized positions that players tend to stay closest to during a match. These positions are commonly tracked by various providers, though each provider may use different naming conventions and classifications. To offer consistency across different data sources, kloppy implements a standardized [PositionType][kloppy.domain.PositionType] with support for the following positions:

<div style="font-family: monospace; white-space: pre; font-size: 0.6rem">
PositionType.Unknown (UNK)
PositionType.Goalkeeper (GK)
PositionType.Defender (DEF)
├── PositionType.FullBack (FB)
│ ├── PositionType.LeftBack (LB)
│ └── PositionType.RightBack (RB)
├── PositionType.CenterBack (CB)
│ ├── PositionType.LeftCenterBack (LCB)
│ └── PositionType.RightCenterBack (RCB)
└── PositionType.WingBack (abstract, no link)
│ ├── PositionType.LeftWingBack (LWB)
│ └── PositionType.RightWingBack (RWB)
PositionType.Midfielder (MID)
├── PositionType.DefensiveMidfield (DM)
│ ├── PositionType.LeftDefensiveMidfield (LDM)
│ ├── PositionType.CenterDefensiveMidfield (CDM)
│ └── PositionType.RightDefensiveMidfield (RDM)
├── PositionType.CentralMidfield (CM)
│ ├── PositionType.LeftCentralMidfield (LCM)
│ ├── PositionType.CenterMidfield (CM)
│ └── PositionType.RightCentralMidfield (RCM)
├── PositionType.AttackingMidfield (AM)
│ ├── PositionType.LeftAttackingMidfield (LAM)
│ ├── PositionType.CenterAttackingMidfield (CAM)
│ └── PositionType.RightAttackingMidfield (RAM)
└── PositionType.WideMidfield (WM)
│ ├── PositionType.LeftWing (LW)
│ ├── PositionType.RightWing (RW)
│ ├── PositionType.LeftMidfield (LM)
│ └── PositionType.RightMidfield (RM)
PositionType.Attacker (ATT)
├── PositionType.LeftForward (LF)
├── PositionType.Striker (ST)
└── PositionType.RightForward (RF)
</div>

As you might have noticed, these positions are ordered hierarchically. This allows you to work on different levels of granularity, from broader categories like "Defender" or "Midfielder" to more specific positions like "Left Back" or "Center Midfield."

```python
>>> from kloppy.domain import PositionType

>>> print(PositionType.LeftBack.parent) 
FullBack

>>> print(PositionType.Defender.is_subtype_of(PositionType.Defender))
True
>>> print(PositionType.LeftCenterBack.is_subtype_of(PositionType.Defender))
True
>>> print(PositionType.LeftBack.is_subtype_of(PositionType.Midfielder))
False
```

Each [`PositionType`][kloppy.domain.PositionType] has a unique name and code.

```python
>>> print(PositionType.LeftCenterBack) 
Left Center Back
>>> print(PositionType.LeftCenterBack.code) 
LCB
```

A player's position can change throughout a game.

```python exec="true" result="text" source="tabbed-left" session="concepts-positions"

from kloppy import statsbomb

event_dataset = statsbomb.load_open_data(match_id="15946")
player = event_dataset.metadata.teams[0].get_player_by_jersey_number(5)
print(player)
print(player.starting_position)
for start_time, end_time, position in player.positions.ranges():
    print(f"{start_time}:{end_time} - {position.code if position is not None else 'SUB'}")
```

You can conveniently retrieve a player's starting position or a player's position at a certain time in the match.

```python exec="true" result="text" source="tabbed-left" session="concepts-positions"
print(player.starting_position)
```


```python exec="true" result="text" source="tabbed-left" session="concepts-positions"
print(player.position_at())
```

Note that a player's position will be `None` if not on the pitch.
