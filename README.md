# kloppy: standardizing soccer tracking and event data <a href='https://kloppy.pysport.org'><img style="width: 120px; height: 139px" src="https://github.com/PySport/kloppy/raw/master/docs/logo.png" align="right" /></a>

[![PyPI Latest Release](https://img.shields.io/pypi/v/kloppy.svg)](https://pypi.org/project/kloppy/)
[![Downloads](https://pepy.tech/badge/kloppy/month)](https://pepy.tech/project/kloppy/month)
![](https://img.shields.io/github/license/PySport/kloppy)
![](https://img.shields.io/pypi/pyversions/kloppy)
[![Powered by PySport](https://img.shields.io/badge/powered%20by-PySport-orange.svg?style=flat&colorA=104467&colorB=007D8A)](https://pysport.org)

## What is it?

Each vendor of soccer data uses its own unique format to describe the course of a game. Hence, software written to analyze this data has to be tailored to a specific vendor and cannot be used without modifications to analyze data from other vendors. Kloppy is a Python package that addresses the challenges posed by the variety of data formats and aims to be the fundamental building block for processing soccer tracking and event data. Kloppy standardizes two types of datasets, namely event data and tracking data. These datasets are generally constructed from two files: a file containing raw (event/tracking) data and a meta data file containing pitch dimensions, squad, match and player information etc.

The creation of these standardized datasets is called **"deserialization"**. 

☕ **Kloppy** is powered by [PySport](https://pysport.org/)  (non-profit, RSIN: 866294211). Consider [contributing](#contributing-to-kloppy) or [donating](https://pysport.org/) to ensure its longevity!

## Supported Providers
| Provider | Event | Tracking | Public Data | Example(s) |
|----------|:-------:|:----------:|:---------------------:| ---------------------|
| Hawkeye (2D) |  | ✅ |  |  |
| Metrica | ✅ | ✅ | ✅ |  |
| [PFF](providers/pff.ipynb) | 🟠 | ✅ | [✅](https://drive.google.com/drive/u/0/folders/1_a_q1e9CXeEPJ3GdCv_3-rNO3gPqacfa) | |
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
- [**Load and Standardize Event Data**]()
- [**Load and Standardize Tracking Data**]()
- [**Standardized Meta Data**]()
- [**Filtering**]()
- [**Transformation**]()
- [**To DataFrame** (with Polars or Pandas)]()
- [**Aggregation**]()
- [**Custom Events**]()
- [**Pattern Matching**]()

#### ▶️ Quick Start

There are multiple providers that offer public / free data. Loading this data is directly supported within kloppy. 

```python
from kloppy import sportec

# load
dataset = sportec.load_open_event_data(match_id="J03WMX")

# filter & transform
goals = (
  dataset
  .filter("shot.goal")
  .transform(
    to_coordinate_system="opta",  
    to_orientation="STATIC_HOME_AWAY"
  )
)

# export
goals.to_df(
  "player", 
  "coordinates_*", 
  assist=lambda event: event.prev("pass"),
  engine="polars"
)
```


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
