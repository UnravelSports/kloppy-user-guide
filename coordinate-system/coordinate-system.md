# Kloppy Coordinate Systems Guide

Kloppy provides support for multiple coordinate systems for event and tracking data, each with different origins and axis directions, and provides built-in conversions to ensure accurate analysis and visualization across various data providers.

- [Kloppy Coordinate Systems Guide](#kloppy-coordinate-systems-guide)
  - [Supported Providers](#supported-providers)
  - [Glossary](#glossary)
    - [Origin](#origin)
    - [Vertical Orientation](#vertical-orientation)
    - [Pitch Diagrams](#pitch-diagrams)
  - [Converting Between Systems](#converting-between-systems)

## Supported Providers
| Provider    | Coordinate System | Origin        | Vertical Orientation |Pitch Length|Pitch Width|
|-------------|-------------------|---------------|----------------------|------------|-----------|
|Kloppy | [KloppyCoordinateSystem](#origin-top-left--vertical-orientation-top-to-bottom) | Top Left| Top to Bottom|1|1
Metrica | [MetricaCoordinateSystem](#origin-top-left--vertical-orientation-top-to-bottom) | Top Left       | Top to Bottom |1|1
HawkEye | [HawkEyeCoordinateSystem](#origin-center--vertical-orientation-bottom-to-top) | Center       | Bottom to Top |105**|68|
PFF | [PFFCoordinateSystem](#origin-center--vertical-orientation-bottom-to-top) | Center       | Bottom to Top |105*|68
SecondSpectrum | [SecondSpectrumCoordinateSystem](#origin-center--vertical-orientation-bottom-to-top) | Center| Bottom to Top | 105*|68
SkillCorner | [SkillCornerCoordinateSystem](#origin-center--vertical-orientation-bottom-to-top) | Center       | Bottom to Top | 105*|68
Sportec Event Data | SportecEventDataCoordinateSystem |?|?|?|?
Sportec Tracking Data | [SportecTrackingDataCoordinateSystem](#origin-center--vertical-orientation-bottom-to-top) | Center     | Bottom to Top | 105* | 68 
Opta | [OptaCoordinateSystem](#origin-bottom-left--vertical-orientation-bottom-to-top) | Bottom Left     | Bottom to Top |100|100
Tracab | [TracabCoordinateSystem](#origin-center--vertical-orientation-bottom-to-top) | Center     | Bottom to Top | 105* | 68 
DataFactory | [DatafactoryCoordinateSystem](#origin-center--vertical-orientation-top-to-bottom) | Center     | Top to Bottom |?|?
StatsBomb | [StatsBombCoordinateSystem](#origin-top-left--vertical-orientation-top-to-bottom) | Top Left     | Top to Bottom |120|80 
WyScout | [WyScoutCoordinateSystem](#origin-top-left--vertical-orientation-top-to-bottom) | Top Left     | Top to Bottom |100|100 
SportVU | [SportVUCoordinateSystem](#origin-top-left--vertical-orientation-top-to-bottom) | Top Left     | Top to Bottom |?|? 

Apart from the above data providers, Kloppy also provides support to create your custom coordinate system using `CustomCoordinateSystem`.

> $*$  This provider provides measured pitch dimensions and includes them in the metadata. This information is used to correctly set set the coordinate system and may deviate from 105 by 68.

> ** HawkEye does not always provide measured pitch dimensions and meta data. When no meta data is provided pitch dimensions default to 105 by 68. 

## Glossary  
### Origin  
- **Top Left** → Origin at the top-left corner of the field.  
- **Bottom Left** → Origin at the bottom-left corner of the field.  
- **Center** → Origin at the center of the field. 
- 
### Vertical Orientation  
- **Top to Bottom** → The Y-axis increases as you move from the top to the bottom of the pitch.  
- **Bottom to Top** → The Y-axis decreases as you move from the top to the bottom of the pitch.  



### Pitch Diagrams
<table>
  <tr>
    <td align="center">
      <strong>Origin: Top Left <br> Vertical Orientation: Top to Bottom</strong><br>
      <img src="imgs/origin_top_left.png" alt="Origin Top Left" width="400"/>
    </td>
    <td align="center">
      <strong>Origin: Bottom Left <br> Vertical Orientation: Bottom to Top</strong><br>
      <img src="imgs/origin_bottom_left.png" alt="Origin Bottom Left" width="400"/>
    </td>
  </tr>
  <tr>
    <td align="center">
      <strong>Origin: Center <br> Vertical Orientation: Bottom to Top</strong><br>
      <img src="imgs/origin_center_bottom_to_top.png" alt="Origin Center Bottom to Top" width="400"/>
    </td>
    <td align="center">
      <strong>Origin: Center <br> Vertical Orientation: Top to Bottom</strong><br>
      <img src="imgs/origin_center_top_to_bottom.png" alt="Origin Center Top to Bottom" width="400"/>
    </td>
  </tr>
</table>

## Converting Between Systems
Kloppy provides built-in functions to seamlessly convert between coordinate systems.

For example, the following snippet allows you to convert data from StatsBombCoordinateSystem to MetricaCoordinateSystem.
```python
# Load the libraries
from kloppy import statsbomb
from kloppy.domain import MetricaCoordinateSystem

# Load StatsBomb open event data
dataset = statsbomb.load_open_data(
    match_id=15946,
    # Optional arguments
    coordinates="statsbomb",
)

# Transform StatsBomb's coordinate system to SportVUCoordinateSystem
dataset_transformed = dataset.transform(
    to_coordinate_system=MetricaCoordinateSystem()
)
```

