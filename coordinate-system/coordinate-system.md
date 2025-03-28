# 📌 Kloppy Coordinate Systems Guide

Kloppy provides support for multiple coordinate systems for event and tracking data, each with different origins and axis directions, and provides built-in conversions to ensure accurate analysis and visualization across various data providers.

## 🌍 Supported Providers
| Provider    | Coordinate System | Origin        | Vertical Orientation |Pitch Length|Pitch Width|
|-------------|-------------------|---------------|----------------------|------------|-----------|
|Kloppy | [KloppyCoordinateSystem](#origin-top-left--vertical-orientation-top-to-bottom) | Top Left| Top to Bottom|1|1
Metrica | [MetricaCoordinateSystem](#origin-top-left--vertical-orientation-top-to-bottom) | Top Left       | Top to Bottom |1|1
HawkEye | [HawkEyeCoordinateSystem](#origin-center--vertical-orientation-bottom-to-top) | Center       | Bottom to Top |?|?|
PFF | [PFFCoordinateSystem](#origin-center--vertical-orientation-bottom-to-top) | Center       | Bottom to Top |105|68
SecondSpectrum | [SecondSpectrumCoordinateSystem](#origin-center--vertical-orientation-bottom-to-top) | Center| Bottom to Top | 105|68
SkillCorner | [SkillCornerCoordinateSystem](#origin-center--vertical-orientation-bottom-to-top) | Center       | Bottom to Top | 105|68
Sportec Event Data | SportecEventDataCoordinateSystem |?|?|?|?
Sportec Tracking Data | [SportecTrackingDataCoordinateSystem](#origin-center--vertical-orientation-bottom-to-top) | Center     | Bottom to Top |?|? 
Opta | [OptaCoordinateSystem](#origin-bottom-left--vertical-orientation-bottom-to-top) | Bottom Left     | Bottom to Top |100|100
Tracab | [TracabCoordinateSystem](#origin-center--vertical-orientation-bottom-to-top) | Center     | Bottom to Top |?|? 
DataFactory | [DatafactoryCoordinateSystem](#origin-center--vertical-orientation-top-to-bottom) | Center     | Top to Bottom |?|?
StatsBomb | [StatsBombCoordinateSystem](#origin-top-left--vertical-orientation-top-to-bottom) | Top Left     | Top to Bottom |120|80 
WyScout | [WyScoutCoordinateSystem](#origin-top-left--vertical-orientation-top-to-bottom) | Top Left     | Top to Bottom |100|100 

Apart from the above data providers, Kloppy also provides support to create your custom coordinate system using `CustomCoordinateSystem`.


## 📖 Glossary  
### Vertical Orientation  
- **Top to Bottom** → The Y-axis increases as you move from the top to the bottom of the pitch.  
- **Bottom to Top** → The Y-axis decreases as you move from the top to the bottom of the pitch.  

### Origin  
- **Top Left** → Origin at the top-left corner of the field.  
- **Bottom Left** → Origin at the bottom-left corner of the field.  
- **Center** → Origin at the center of the field. 

### 📐 Pitch Diagrams
#### Origin: Top Left | Vertical Orientation: Top to Bottom
![Origin Top Left](imgs/origin_top_left.png)

#### Origin: Bottom Left | Vertical Orientation: Bottom to Top
![Origin Top Left](imgs/origin_bottom_left.png)

#### Origin: Center | Vertical Orientation: Bottom to Top
![Origin Top Left](imgs/origin_center_bottom_to_top.png)

#### Origin: Center | Vertical Orientation: Top to Bottom
![Origin Top Left](imgs/origin_center_top_to_bottom.png)


