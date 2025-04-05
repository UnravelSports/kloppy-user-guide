# Event data

Event (stream) data is **a time-coded feed which describes the key events that occur during a match**. The data is generally created by trained human annotators (assisted by computer vision) based on broadcast video of a game. Most of the events they annotate are on-the-ball player actions such as shots, passes, and dribbles. However, the data feed will typically also include other relevant events like substitutions and tactical changes. As an example, we load the [StatsBomb](https://statsbomb.com) event data of the 2002 World Cup final.

```python exec="true" source="above" session="concept-eventdata"
from kloppy import statsbomb

dataset = statsbomb.load_open_data(match_id="3869685")
```

This will create an [`EventDataset`][kloppy.domain.EventDataset] that wraps a list of [`Event`][kloppy.domain.Event] entities and implements a number of common operations to manipulate the dataset. This section explains the [`Event`][kloppy.domain.Event] entities. Other sections of the user guide will explain in-depth how to [load](user-guide/providers), [filter](user-guide/filters), [transform](user-guide/transformations) and [export](user-guide/dataframe) an event dataset.


## Kloppy's Event Data Model

Event data is sold by specialized data vendors such as StatsBomb, Stats Perform (Opta), and Wyscout. All these data vendors annotate the same games but use their own set of event types, attributes, and data formats. This can make it difficult to write software or perform data analyses that can be applied to multiple event data sources. Therefore, kloppy implements its own **vendor-independent data model** for describing events.

Below is what the kick-off event looks like in the raw data. StatsBomb uses a JSON object to describe each event.

```python exec="true" result="text" session="concept-eventdata"
print(dataset[4].raw_event)
```

For comparison, below is what the same kick-off looks like in kloppy's data model.

```python exec="true" result="text" session="concept-eventdata"
print(dataset[4])
```

Instead of JSON objects, kloppy uses [`Event`][kloppy.domain.Event] objects to represent events. This provides a number of advantages over storing the raw data in a dictionary or data frame, such as better readability, type-safety, and autocompletion in most IDEs.


## Event Types

Each event has a specific type, corresponding to passes, shots, tackles, etc. These event types are implemented as different subclasses of [`Event`][kloppy.domain.Event]. For example, a pass is implemented by the [`PassEvent`][kloppy.domain.PassEvent] subclass, while a substitution is implemented by the [`SubstitutionEvent`][kloppy.domain.SubstitutionEvent] subclass. Each subclass implements additional attributes specific to that event type. For example, a pass has a `result` (complete, incomplete, out, or offside); while a substitution has a `replacement_player` attribute.

Let's look at the opening goal of the 2002 World Cup final as an example. It is a penalty by Lionel Messi.

```python exec="true" source="tabbed-left" result="text" session="concept-eventdata"
goal_event = dataset.get_event_by_id("6d527ebc-a948-4cd8-ac82-daced35bb715")
print(goal_event)
```

In kloppy's data model, the penalty is represented by a [`ShotEvent`][kloppy.domain.ShotEvent]. Each [`ShotEvent`][kloppy.domain.ShotEvent] has a `result` attribute that contains a [`ShotResult`][kloppy.domain.ShotResult]. As the penalty was scored, the result here is `ShotResult.GOAL`.

If a particular event type is not included in kloppy's data model, it will be deserialized as a [`GenericEvent`][kloppy.domain.GenericEvent]. For example, kloppy does not (yet) have a data model for ball receival events.

```python exec="true" source="tabbed-left" result="text" session="concept-eventdata"
receival_event = dataset.get_event_by_id("0db72b17-bed3-446f-ae22-468480e33ad6")
print(receival_event)
```

For an overview of all event types and their attributes, see the [Data Model Reference](/reference/event-data/event-types).

!!! note

    Kloppy's data model covers the event types and attributes that are commonly used by multiple data vendors. Some vendors might have certain specific event types or attributes that are not implemented in kloppy's data model, but kloppy's data model can easily be [extended](/todo) to support these if needed.


## Qualifiers

In addition to event type-specific attributes each event can have one or more qualifiers attached to it. While attributes define core properties of an event such as its outcome, qualifiers provide extra context about how an event happened. They add more descriptive details that help with deeper analysis.

For Messi's penalty that we looked at above, kloppy adds a [`SetPieceQualifier`](kloppy.domain.SetPieceQualifier) and a [`BodyPartQualifier`](kloppy.domain.BodyPartQualifier).

```python exec="true" source="tabbed-left" result="text" session="concept-eventdata"
print(goal_event.qualifiers)
```

It's also possible to check if an event has a qualifier of a certain type.

```python exec="true" source="tabbed-left" result="text" session="concept-eventdata"
from kloppy.domain import SetPieceQualifier
sp_qualifiers = goal_event.get_qualifier_value(SetPieceQualifier)
print(sp_qualifiers)
```

For an overview of all qualifiers, see the [Data Model Reference](/reference/event-data/qualifiers).
