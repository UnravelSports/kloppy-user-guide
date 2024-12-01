# Time

Soccer data is fundamentally tied to time. Seasons, matches within a season, and events within a match are all ordered. Hence, each event needs to carry information about *when* it happened.

However, the representation of time varies significantly across data providers. Time may be represented using absolute timestamps (e.g., a specific UTC time like 2024-12-01T15:45:13Z) or through a match-specific game clock (e.g., 75:13, as seen on a scoreboard). Even when using a game clock, there are further variations, such as how extra time is handled or whether the clock resets at the start of each period.

To address these inconsistencies, kloppy introduces a standardized approach to managing time. In short, the game is split up in *periods* (i.e., the two 45-minute halves) which have absolute timestamps to denote their start and end timestamps. Withing each period, time is then expressed relatively with respect to the start of the period using a game clock.

!!! note

    We use the built-in `datetime` objects to handle absolute timestamps and `timedelta` objects to handle relative timestamps. Absolute timestamps always include timezone information.


## Periods

[`Period`][kloppy.domain.Period] entities are used to split up a game into periods.

```python exec="true" source="tabbed-left" result="text" session="getting-started"
from kloppy.domain import Period
from datetime import datetime, timezone

periods = [
    Period(
        id=1,
        start_timestamp=datetime(2024, 12, 1, 15, 0, 0, tzinfo=timezone.utc),
        end_timestamp=datetime(2024, 12, 1, 15, 45, 10, tzinfo=timezone.utc),
    ),
    Period(
        id=2,
        start_timestamp=datetime(2024, 12, 1, 16, 00, 0, tzinfo=timezone.utc),
        end_timestamp=datetime(2024, 12, 1, 16, 48, 30, tzinfo=timezone.utc),
    ),
]
```

Ideally, the `start_timestamp` and `end_timestamp` values are expressed as absolute time-zone aware `datetime`, with the `start_timestamp` marking the exact time of the period's kick-off and the `end_timestamp` marking the time of the final whistle. This alows users to link and sync different datasets (e.g., tracking data with video).

However, when absolute times are not available, kloppy falls back to using offsets. In this case, the `start_timestamp` is defined as the offset between the start of the data feed for the period and the kick-off of the period, while the `end_timestamp` is defined as the offset between the start of the data feed and the final whistle of the period. This ensures that even in the absence of absolute time data, a relative timeline is maintained.

```python exec="true" source="tabbed-left" result="text" session="getting-started"
from kloppy.domain import Period
from datetime import timedelta

periods = [
    Period(
        id=1,
        start_timestamp=timedelta(seconds=0),
        end_timestamp=timedelta(minutes=45),
    ),
    Period(
        id=2,
        start_timestamp=timedelta(minutes=45, seconds=10),
        end_timestamp=timedelta(minutes=90),
    ),
]
```

Each period also has an ID. Therfore, kloppy uses the following convention.

- `1`: First half
- `2`: Second half
- `3`: First half of overtime
- `4`: Second half of overtime
- `5`: Penalty shootout

## Game Clock

Within a period, timings are relative. Each record (i.e., event, frame or code) has a `time` attribute that measures the elapsed time since the period started. This implies that timestamps are reset to zero for each period and that records before the kick-off event get a negative timestamp.

```python exec="true" source="tabbed-left" result="text" session="getting-started"
time = goal_event.time
print(time)
```

A [`Time`][kloppy.domain.Time] entity consist of two parts: a reference to a period and a timestamp relative to the kick-off in that period.

```python exec="true" source="tabbed-left" result="text" session="getting-started"
print(time.period)
print(time.timestamp)
```

The `timestamp` represents the time elapsed since the start of the period. The absolute time in the match can be obtained by combining both the `period` and `timestamp`.

```python exec="true" source="tabbed-left" result="text" session="getting-started"
abs_time = time.period.start_time + time.timestamp
print(abs_time)
```


TODO: mathematical operations on `Time` objects.


# TimeContainer

to handle properties that change during a match, like Player Position, or Team Formation
