# kloppy: standardizing soccer tracking and event data <a href='https://kloppy.pysport.org'><img style="width: 120px; height: 139px" src="https://github.com/PySport/kloppy/raw/master/docs/logo.png" align="right" /></a>

[![PyPI Latest Release](https://img.shields.io/pypi/v/kloppy.svg)](https://pypi.org/project/kloppy/)
[![Downloads](https://pepy.tech/badge/kloppy/month)](https://pepy.tech/project/kloppy/month)
![](https://img.shields.io/github/license/PySport/kloppy)
![](https://img.shields.io/pypi/pyversions/kloppy)
[![Powered by PySport](https://img.shields.io/badge/powered%20by-PySport-orange.svg?style=flat&colorA=104467&colorB=007D8A)](https://pysport.org)

## What is it?

Each vendor of soccer data uses its own unique format to describe the course of a game. Hence, software written to analyze this data has to be tailored to a specific vendor and cannot be used without modifications to analyze data from other vendors. Kloppy is a Python package that addresses the challenges posed by the variety of data formats and aims to be the fundamental building block for processing soccer tracking and event data. It provides (de)serializers, standardized data models, filters, and transformers which make working with tracking and event data from different vendors a breeze.

☕ **Kloppy** is powered by [PySport](https://pysport.org/)  (non-profit, RSIN: 866294211). Consider [contributing](#contributing-to-kloppy) or [donating](https://pysport.org/) to ensure its longevity!

## Supported Providers
| Provider | Event | Tracking | Public Data | Example(s) |
|----------|:-------:|:----------:|:---------------------:| ---------------------|
| Hawkeye (2D) |  | ✅ |  |  |
| Metrica | ✅ | ✅ | ✅ |  |
| PFF | 🟠 | ✅ | ✅ | |
| SecondSpectrum | [PR #437](https://github.com/PySport/kloppy/pull/437) | ✅ |  |  |
| SkillCorner |  | ✅ | ✅ |  |
| Sportec | ✅ | ✅ | ✅ |  |
| StatsPerform / Opta | ✅ | ✅ |  |  |
| Tracab |  | ✅ |  |  |
| DataFactory | ✅ |  |  |  |
| StatsBomb | ✅ $^1$ |  | ✅ $^1$ |  |
| WyScout | ✅ |  | ✅ |  |

✅ Implemented  🟠 Not yet implemented 

$^1$ Includes 360 freeze frame data support

More information on the supported events per provider can be found [here](broken).

## Main features
### Loading Data

#### ▶️ Standardized Data
Loading event or tracking dataw ith **kloppy** creates a standardized dataset and standardized meta data.

#### ▶️ Load Public Datasets

There are multiple providers that offer public / free data. Loading this data is directly supported within kloppy. 

For example [An integrated dataset of spatiotemporal and event data in elite soccer (Bassek et al. 2025)](https://www.nature.com/articles/s41597-025-04505-y) offers 7 matches of Deutsche Fußball Liga event and tracking data.
```python
from kloppy import sportec

event_dataset = sportec.load_open_event_data(match_id="J03WMX")
tracking_dataset = sportec.load_open_tracking_data(match_id="J03WMX")
```

#### ▶️ Load Local Datasets
```python
from kloppy import statsperform

dataset = statsperform.load_tracking(
    ma1_data="../../kloppy/tests/files/statsperform_tracking_ma1.xml",
    ma25_data="../../kloppy/tests/files/statsperform_tracking_ma25.txt",
)
```

#### ▶️ Load Remote Datasets
```python
from kloppy import statsperform

dataset = statsperform.load_tracking(
    ma1_data="s3://some_bucket/statsperform_tracking_ma1.xml",
    ma25_data="3://some_bucket/statsperform_tracking_ma25.txt",
)
```

### Processing data

Load some (open) event data: 
```python
from kloppy import sportec
dataset = sportec.load_open_event_data()
```

#### ▶️ Standarized Meta Data
Each `EventDataset` and `TrackingDataset` comes with standardized meta data. See [Meta data documentation](broken) for more information.
```python
home_team, away_team = dataset.metadata.teams
f"{home_team} - {away_team}"
>>> 'Fortuna Düsseldorf - 1. FC Nürnberg'
```
#### ▶️ Filtering
Filter for certain events. See [EventTypes documentation](broken) for more information on available events.
```python
shots = dataset.filter("shot")    
```
#### ▶️ To DataFrame
**Kloppy** supports directly outputting `EventDataset` or `TrackingDataset` to a [Polars](https://pola.rs/) or [Pandas](https://pandas.pydata.org/) DataFrame with `.to_df()`

Change `engine="pandas"` for a Pandas DataFrame.

```python
shots.to_df(
    "player_id",
    lambda event: {
        "player_name": str(event.player),
        "is_goal": event.result.is_success if event.result is not None else None
    },
    "coordinates_*",
    prev_pass_player=lambda event: str(event.prev("pass").player),
    engine="polars"
).head(3)
```

| player_id | player_name | is_goal | coordinates_x | coordinates_y | prev_pass_player |
| --- | --- | :---: | :---: | :---: | --- |
| DFL-OBJ-0028BZ | C. Daferner | False | 0.899 | 0.573 | Kwadwo Duah |
| DFL-OBJ-0000F8 | F. Klaus | False | 0.0916 | 0.429 | M. Karbownik |
| DFL-OBJ-0028BZ | C. Daferner | False | 0.876 | 0.426 | J. Castrop |

#### ▶️ Aggregation
`EventDataset` has a built in `"minutes_played"` aggregation.

```python
data = dataset.aggregate("minutes_played")

for item in data:
    print(f"{item.player} - {item.duration.total_seconds() / 60:.1f} minutes played")
```
```text
R. Hennings - 33.9 minutes played
Matthias Zimmermann - 97.2 minutes played
F. Klaus - 76.7 minutes played
etc..
```

Combining this with some data frame operations allows us to compute "per 90" stats, see [this example](broken).

#### ▶️ Transformations
**Kloppy** supports multiple different [coordinate systems](broken) and different [orientations](broken).
Here we transform from the default coordinate system to `SecondSpectrumCoordinateSystem`.

```python
shots_transformed = shots.transform(
    to_coordinate_system=SecondSpectrumCoordinateSystem(
        pitch_length=105,
        pitch_width=68
    ),
    to_orientation="STATIC_HOME_AWAY"
)
```

| <br>team_name | <br>ground | shots<br>coordinates_x | shots<br>coordinates_y | shots_transformed<br>coordinates_x | shots_transformed<br>coordinates_y |
| --- | --- | :---: | :---: | :---: | :---: |
| 1. FC Nürnberg | away | 0.90 | 0.57 | -41.98 | 5.01 |
| Fortuna Düsseldorf | home | 0.09 | 0.43 | 42.88 | -4.81 |
| 1. FC Nürnberg | away | 0.88 | 0.43 | -39.48 | -5.01 |

#### ▶️ Custom Events

<details>
    <summary> <b><i> Expand for a short explanations on Custom Events</i></b> </summary>
<div style="display: flex; align-items: flex-start;">
<div style="flex: 1; padding-right: 20px;">

<br>
You can create custom events with the `EventFactory`. In this example we'll add xG to our Sportec `EventDataset`.


```python
from kloppy.domain import EventFactory, create_event, ShotEvent
from dataclasses import dataclass
from typing import Optional

@dataclass(repr=False)
class SportecShotEvent(ShotEvent):
    xg: Optional[float] = None

    @property
    def inverted_xg(self) -> Optional[float]:
        if self.xg is not None:
            return 1 - self.xg
        else:
            return None

    def __str__(self):
        return (
            f"<Shot event_id='{self.event_id}' time='{self.time}' player='{self.player}' xg={self.xg} result='{self.result}'>"
        )

class SportecEventFactory(EventFactory):
    def build_shot(self, raw_event, **kwargs) -> ShotEvent:
        xg = raw_event['xG']
        
        if xg is not None:
            xg = float(xg)

        return create_event(
            SportecShotEvent,
            xg=xg,
            raw_event=raw_event,
            **kwargs,
        )

dataset = sportec.load_open_event_data(
    event_factory=SportecEventFactory()  # Use our custom EventFactory
)

shots = dataset.filter("shot")

for shot in shots[:3]:
    print(shot)

>>> <Shot event_id='18237400000125' time='P1T07:40' player='C. Daferner' xg=0.5062 result='SAVED'>
>>> <Shot event_id='18237400000209' time='P1T12:27' player='F. Klaus' xg=0.1432 result='BLOCKED'>
>>> <Shot event_id='18237400000279' time='P1T16:22' player='C. Daferner' xg=0.0934 result='BLOCKED'>
```
</div>
</div>
</details>

#### ▶️ Pattern matching
<details>
    <summary> <b><i> Expand for a short explanations on Pattern matching</i></b> </summary>
<div style="display: flex; align-items: flex-start;">
<div style="flex: 1; padding-right: 20px;">

<br>
You can create custom events with the `EventFactory`. In this example we'll add xG to our Sportec `EventDataset`.


```python
recover_ball_within_10_seconds = (
    # 1. Search for a pass. When we found one, lets capture it for later usage.
    pm.match_pass(capture="last_pass_of_team_a")
    +
    # 2. We want to find ball losses. This means the team changes. In this case we
    #    want to match 1 or more passes from team B ("not same as team A"). The
    #    'slice(1, None)' means "1 or more"
    pm.match_pass(team=pm.not_same_as("last_pass_of_team_a.team"))
    * slice(1, None)
    +
    # 3. We create a group of events. The groups makes it possible to: i) match all
    #    of its children, or none and ii) capture it.
    #
    #    The pattern within the group matches when there is a successful pass of
    #    team A within 10 seconds after "last_pass_of_team_a" and it's followed by
    #    a successful pass OR a shot
    #
    #    The 'slice(0, 1)' means the subpattern should match zero or once times.
    #    When the subpattern is not found there is not capture. We use this
    #    further on the calculate percentage.
    pm.group(
        pm.match_pass(
            success=True,
            team=pm.same_as("last_pass_of_team_a.team"),
            timestamp=pm.function(
                lambda timestamp, last_pass_of_team_a_timestamp: timestamp
                - last_pass_of_team_a_timestamp
                < timedelta(seconds=15)
            ),
            capture="recover",
        )
        + (
            # resulted in possession after 5 seconds
            pm.group(
                pm.match_pass(
                    success=True,
                    team=pm.same_as("recover.team"),
                    timestamp=pm.function(
                        lambda timestamp, recover_timestamp, **kwargs: timestamp
                        - recover_timestamp
                        < timedelta(seconds=5)
                    ),
                )
                * slice(None, None)
                + pm.match_pass(
                    success=True,
                    team=pm.same_as("recover.team"),
                    timestamp=pm.function(
                        lambda timestamp, recover_timestamp, **kwargs: timestamp
                        - recover_timestamp
                        > timedelta(seconds=5)
                    ),
                )
            )
            | pm.group(
                pm.match_pass(
                    success=True, team=pm.same_as("recover.team")
                )
                * slice(None, None)
                + pm.match_shot(team=pm.same_as("recover.team"))
            )
        ),
        capture="success",
    )
    * slice(0, 1)
)
matched_events = pm.search(dataset, pattern=recover_ball_within_10_seconds)
matched_events[0]

>>> Match(events=(<PassEvent event_id='18237400000009' time='P1T00:06' player='Dawid Kownacki' result='INCOMPLETE'>, <PassEvent event_id='18237400000010' time='P1T00:12' player='C. Mathenia' result='COMPLETE'>, <PassEvent event_id='18237400000011' time='P1T00:15' player='James Lawrence' result='COMPLETE'>, <PassEvent event_id='18237400000012' time='P1T00:18' player='F. Nürnberger' result='COMPLETE'>, <PassEvent event_id='18237400000013' time='P1T00:23' player='James Lawrence' result='INCOMPLETE'>), captures={'last_pass_of_team_a': <PassEvent event_id='18237400000009' time='P1T00:06' player='Dawid Kownacki' result='INCOMPLETE'>})
```

Count the number of matched events

```python
counter = Counter()

for m in matched_events:
    team = m.captures["last_pass_of_team_a"].team
    success = "success" in m.captures

    counter.update(
        {
            f"{team.ground}_total": 1,
            f"{team.ground}_success": 1 if success else 0,
        }
    )

counter

>>> Counter({'away_total': 29,
         'home_total': 27,
         'home_success': 3,
         'away_success': 1})
```
</div>
</div>
</details>


## Installation

The source code is currently hosted on GitHub at: [https://github.com/PySport/kloppy](https://github.com/PySport/kloppy).

Installers for the latest released version are available at the [Python package index](https://pypi.org/project/kloppy).

```sh
pip install kloppy
```

Install from github (dev version)

```sh
pip install git+https://github.com/PySport/kloppy.git
```

## Documentation

The official documentation is hosted on pysport.org: [https://kloppy.pysport.org](https://kloppy.pysport.org). 


## Contributing to kloppy

All contributions, bug reports, bug fixes, documentation improvements, enhancements, and ideas are welcome.

An overview on how to contribute can be found in the **[contributing guide](https://kloppy.pysport.org/contributing)**.

If you are simply looking to start working with the kloppy codebase, navigate to the GitHub "issues" tab and start looking through interesting issues.


## Current contributors

<a href="https://github.com/PySport/kloppy/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=PySport/kloppy" />
</a>

Made with [contrib.rocks](https://contrib.rocks).
