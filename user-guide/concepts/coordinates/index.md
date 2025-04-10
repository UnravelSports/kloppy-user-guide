# Coordinates

There are notable differences in how various data providers manage the spatial aspect of the data. They all rely on Cartesian coordinate systems to indicate locations on the pitch. However, they differ in the location of the origin, the orientation of the axes, and the scaling of the axes. In this section, we will first examine a few common coordinate systems, discussing their advantages and disadvantages. Next, we explain how these coordinate systems are implemented in kloppy.

## Intro to spatial reference systems
A coordinate system is defined by its **origin**, as well as the **orientation of its axes**. On a soccer pitch, the simplest way to establish a coordinate system is by placing the origin at the center of the field and aligning the y-axis vertically upward. With this setup, the center mark of the pitch corresponds to the coordinate (0, 0). Visually, it might look something like this:

![A coordinate system with the origin placed at the center of the pitch](./images/coordinates-01.png)

In this coordinate system, interpreting a player's position is intuitive. For example, if a player's coordinates at some point are (22, 14) you know that they are located 22 units right and 14 units above the center of the pitch. However, knowing the **unit of measurement** (meters, centimeters, or yards) is essential to be able to interpret these coordinates, as different providers may use different units.

Another common approach is to align the origin with the top-left or bottom-left corner of the pitch. This ensures all coordinates are positive, unless the player moves outside the pitch boundaries. Here is an example:

![A coordinate system with the origin placed at the bottom left of the pitch](./images/coordinates-02.png)

This highlights a second important aspect: **pitch dimensions**. The coordinate system itself does not specify whether a player is in or out of bounds. To clarify, imagine a player is standing at (120,0). Unless you know the actual length of the pitch (i.e., the max value along the x-dimension),[^1] you can’t tell if the player is inside or outside the pitch.

Things become more complex when analyzing data across matches played on pitches of varying dimensions. For instance, if you wish to analyze actions within predefined zones, those zones must be adjusted for each pitch format. To address this issue, many providers, particularly those offering event data, opt to **standardize pitch dimensions**. In standardized systems, all pitch data is projected onto a field of fixed dimensions. For example, kloppy's default pitch is represented as 1 unit long by 1 unit wide. No unit of measurement really applies here, but one can interpret the values as percentages.

![Kloppy's default standardized coordinate system](./images/coordinates-03.png)

Standardized pitch dimensions are convenient when comparing data across matches. It is problematic though if you want to compute anything that involves distances. All metric information gets distorted during standardization, and all related calculations require a mapping back to actual pitch sizes to be correct.

Another aspect of standardization involves aligning **playing directions**. To simplify data visualization and analysis, some providers standardize the playing direction so that all attacking actions occur in the same direction (e.g., left-to-right). Consider plotting all shots taken by a player in a season. If the data is standardized to always represent attacks in one direction, visualizations become much easier to interpret.

![Aligning playing directions](./images/coordinates-04.png)

## Specifying a coordinate system

The above examples include the most common approaches to encode locations in soccer match data. All have pro's and con's. Therefore, kloppy supports all of them and makes transformations between these systems straightforward.

### Built-in coordinate systems

| Coordinate System | Provider               | Origin      | Vertical Orientation | Dimension x | Dimension y |
|-------------------|------------------------|-------------|----------------------|-------------|-------------|
| [`KloppyCoordinateSystem`][kloppy.domain.KloppyCoordinateSystem] | [kloppy][kloppy.domain.Provider.KLOPPY] | [top-left][kloppy.domain.Origin.TOP_LEFT] | [top-to-bottom][kloppy.domain.VerticalOrientation.TOP_TO_BOTTOM] | [0, 1] | [0, 1] |
| [`MetricaCoordinateSystem`][kloppy.domain.MetricaCoordinateSystem] | [metrica][kloppy.domain.Provider.METRICA] | [top-left][kloppy.domain.Origin.TOP_LEFT] | [top-to-bottom][kloppy.domain.VerticalOrientation.TOP_TO_BOTTOM]  | [0, 1] | [0, 1] |
| [`HawkEyeCoordinateSystem`][kloppy.domain.HawkEyeCoordinateSystem] | HawkEye | [center][kloppy.domain.Origin.CENTER] | [bottom-to-top][kloppy.domain.VerticalOrientation.BOTTOM_TO_TOP] | [0, 105] | [0, 68] |
| [`PFFCoordinateSystem`][kloppy.domain.PFFCoordinateSystem] | PFF | [center][kloppy.domain.Origin.CENTER] | [bottom-to-top][kloppy.domain.VerticalOrientation.BOTTOM_TO_TOP] | [0, 105] | [0, 68] |
| [`SecondSpectrumCoordinateSystem`][kloppy.domain.SecondSpectrumCoordinateSystem] | SecondSpectrum | [center][kloppy.domain.Origin.CENTER] | [bottom-to-top][kloppy.domain.VerticalOrientation.BOTTOM_TO_TOP] | [0, 105] | [0, 68] |
| [`SkillCornerCoordinateSystem`][kloppy.domain.SkillCornerCoordinateSystem] | SkillCorner | [center][kloppy.domain.Origin.CENTER]      | [bottom-to-top][kloppy.domain.VerticalOrientation.BOTTOM_TO_TOP] | [0, 105]     | [0, 68]      |
| [`SportecEventDataCoordinateSystem`][kloppy.domain.SportecEventDataCoordinateSystem] | Sportec Event Data | ?           | ?                    | ?           | ? |
| [`SportecTrackingDataCoordinateSystem`][kloppy.domain.SportecTrackingDataCoordinateSystem] | Sportec Tracking Data | [center][kloppy.domain.Origin.CENTER] | [bottom-to-top][kloppy.domain.VerticalOrientation.BOTTOM_TO_TOP] | [0, 105] | [0, 68]      |
| [`OptaCoordinateSystem`][kloppy.domain.OptaCoordinateSystem] | Opta | [bottom-left][kloppy.domain.Origin.BOTTOM_LEFT] | [bottom-to-top][kloppy.domain.VerticalOrientation.BOTTOM_TO_TOP] | [0, 100] | [0, 100] |
| [`TracabCoordinateSystem`][kloppy.domain.TracabCoordinateSystem] | Tracab | [center][kloppy.domain.Origin.CENTER] | [bottom-to-top][kloppy.domain.VerticalOrientation.BOTTOM_TO_TOP] | [0, 105] | [0, 68] |
| [`DatafactoryCoordinateSystem`][kloppy.domain.DatafactoryCoordinateSystem] | DataFactory | [center][kloppy.domain.Origin.CENTER] | [top-to-bottom][kloppy.domain.VerticalOrientation.TOP_TO_BOTTOM] | ? | ?           |
| [`StatsBombCoordinateSystem`][kloppy.domain.StatsBombCoordinateSystem] | StatsBomb | [top-left][kloppy.domain.Origin.TOP_LEFT] | [top-to-bottom][kloppy.domain.VerticalOrientation.TOP_TO_BOTTOM] | [0, 120]     | [0, 80]      |
| [`WyscoutCoordinateSystem`][kloppy.domain.WyscoutCoordinateSystem] | WyScout | [top-left][kloppy.domain.Origin.TOP_LEFT] | [top-to-bottom][kloppy.domain.VerticalOrientation.TOP_TO_BOTTOM] | [0, 100]     | [0, 100]     |
| [`SportVUCoordinateSystem`][kloppy.domain.SportVUCoordinateSystem] | SportVU | [top-left][kloppy.domain.Origin.TOP_LEFT] | [top-to-bottom][kloppy.domain.VerticalOrientation.TOP_TO_BOTTOM] | ?           | ?           |

> $*$  This provider provides measured pitch dimensions and includes them in the metadata. This information is used to correctly set set the coordinate system and may deviate from 105 by 68.

> ** HawkEye does not always provide measured pitch dimensions and meta data. When no meta data is provided pitch dimensions default to 105 by 68.
>

### Custom coordinate systems

Apart from the above data providers, Kloppy also provides support to create your custom coordinate system using the [`CustomCoordinateSystem`][kloppy.domain.CustomCoordinateSystem] class.

```python
from kloppy.domain import CustomCoordinateSystem, Origin, VerticalOrientation, NormalizedPitchDimensions, Dimension

my_coordinate_system = CustomCoordinateSystem(
    origin=Origin.TOP_LEFT,
    vertical_orientation=VerticalOrientation.TOP_TO_BOTTOM,
    pitch_dimensions=NormalizedPitchDimensions(
        x_dim=Dimension(0, 1),
        y_dim=Dimension(0, 1),
        pitch_length=105,
        pitch_width=68,
        standardized=True,
    ),
)
```

A coordinate system is fully defined by three components: an origin, a direction of the y-axis, and the pitch dimensions. Each of these are represented with a dedicated core object.

#### Origin
- **Top Left** → Origin at the top-left corner of the field.  
- **Bottom Left** → Origin at the bottom-left corner of the field.  
- **Center** → Origin at the center of the field.

#### VerticalOrientation

- **Top to Bottom** → The Y-axis increases as you move from the top to the bottom of the pitch.  
- **Bottom to Top** → The Y-axis decreases as you move from the top to the bottom of the pitch.

#### PitchDimensions

A [`PitchDimensions`][kloppy.domain.PitchDimensions] entity fully describes the dimensions and markings of a pitch. This is how you create a 105-by-68 meter pitch with the standard dimensions in according to the IFAB regulations:

```Python
from kloppy.domain import PitchDimensions, Dimension, Unit

pitch_dimensions = PitchDimensions(
    # pitch boundaries
    x_dim=Dimension(min=0, max=105),
    y_dim=Dimension(min=0, max=68),
    unit=Unit.METERS,
    # standardized or real dimensions
    standardized=False,
    # pitch markings
    goal_width=7.32,
    goal_height=2.44,
    six_yard_width=18.32,
    six_yard_length=5.5,
    penalty_area_width=40.32,
    penalty_area_length=16.5,
    circle_radius=9.15,
    corner_radius=1,
    penalty_spot_distance=11,
    penalty_arc_radius=9.15,
)
```

First, the `x_dim` and `y_dim` attributes specify the minimum and maximum value of coordinates inside the pitch boundaries along both axes. In case of `standardized` pitch dimensions, these are not necessarily related to the actual pitch length and width. 

Notice that you don’t technically need the actual pitch length and widths to process the data, they are purely optional. In case of the first pitch, they are easily derived from the x- and ylim properties. But not in the case of the Opta pitch.
To get non-distored calculations, the `length` and `width` of the pitch need to be specified.

Furthermore, `unit` refers to the unit of measurement. Possible values are `"METERS"`, `"CENTIMETERS"`, `"YARDS"`, `"FEET"` or `"NORMED"`. The latter is a special value to indicate that no real unit applies to be used with standardized pitch dimensions such as [0,1] or [0,100].

Most of the attributes are related to the pitch markings. These are independent of the actual size of the pitch.

For convenience we’ve included major data provider pitches

- IFAB regulations: MetricPitchDimensions vs ImperialPitchDimensions
- Provider-specific standardized pitch dimensions: Opta, Wyscout, StatsBomb

## Specifying the Attacking Direction

An [`Orientation`][kloppy.domain.Orientation] specifies the attacking direction of each team in a dataset.

| Key                  | Description                                                                                                   |
|----------------------|---------------------------------------------------------------------------------------------------------------|
| BALL_OWNING_TEAM     | The team that is currently in possession of the ball plays from left to right.                               |
| ACTION_EXECUTING_TEAM| The team that executes the action plays from left to right. Used in event stream data only. Equivalent to "BALL_OWNING_TEAM" for tracking data. |
| HOME_AWAY            | The home team plays from left to right in the first period. The away team plays from left to right in the second period. |
| AWAY_HOME            | The away team plays from left to right in the first period. The home team plays from left to right in the second period. |
| STATIC_HOME_AWAY     | The home team plays from left to right in both periods.                                                      |
| STATIC_AWAY_HOME     | The away team plays from left to right in both periods.                                                      |
| NOT_SET              | The attacking direction is not defined.                                                                     |


The orientation is at the dataset level, records have an attacking direction.




[^1]: Pitch sizes are sometimes provided as metadata by the data provider. When this information is absent, a common approach is to assume a standard pitch size of 105 by 68 meters. However, it's worth noting that actual pitch lengths can vary significantly from 90 to 120 meters and widths from 45 to 90 meters. Therefore, this naive approach can introduce significant errors. Alternatively, the dimensions of the pitch can sometimes be estimated by analyzing corner kick positions.

