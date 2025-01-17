---
hide:
  - navigation
---

!!! note

    The package repository is available [here](https://github.com.mcas.ms/habitus-eu/hbGPS). For technical details, please refer to the [description](https://github.com.mcas.ms/habitus-eu/hbGPS/blob/main/documentation.md).

R package for GPS data processing and merging with accelerometer data. The package provides a pipeline that performs the following:

- Loading GPS CSV files, with the aim of making this process flexible to most common formats. 
- Taking timezone into account when interpreting timestamps. 
- Calculating signal-to-noise ratio (SNR) in GPS data. 
- Detecting and removing outliers in speed and elevation. 
- Calculating distance and speed. 
- Distinguishing between indoor and outdoor activities. 
- Detecting trips and allowing for breaks in between them. 
- Merging GGIR time series output data to work with various accelerometer brands and data formats. 
- Granting users control over crucial parameters.

## Installation

``` r
install.packages("remotes") # (1)!
remotes::install_github("wadpac/GGIR") # (2)!
remotes::install_github("habitus-eu/hbGPS") # (3)!

# Optional packages
install.packages(pkgs = c("actilifecounts", "ActCR", "GGIRread", "read.gt3x", "readxl")) # (3)!
```

1. R package for installation from remote repositories (Github).
2. Installing the GGIR package from GitHub repository.
3. Installing the hbGPS package from GitHub repository.
4. Install these packages to process accelerometer files in counts or use other GGIR functionalities.

!!! warning

    When installing packages, you may be prompted to update to more recent versions. We recommend updating all the packages.

## Usage

If you want to use hbGPS to merge accelerometer and GPS data, you need to process the accelerometer data first using the GGIR package. Once you have processed the accelerometer data, you can merge it with GPS data using hbGPS.

### 1. Processing accelerometer data with GGIR

There are two pipelines available for processing accelerometer data with GGIR, depending on whether the data is in RAW accelerometer format or already in counts. For explanation of GGIR parameters and additional information on how to use the GGIR package, see the [documentation](https://cran.r-project.org/web/packages/GGIR/vignettes/GGIR.html).

#### RAW data

``` r
library(GGIR)

GGIR(datadir = "C:/path/to/your/data/folder",
     outputdir = "C:/path/to/your/output/folder",
     mode = c(1:5),
     overwrite = TRUE,
     do.report = c(),
     windowsizes = c(5, 900, 3600),
     includedaycrit = 10,
     includenightcrit = 10,
     part5_agg2_60seconds = TRUE,
     HASPT.algo = "NotWorn",
     HASIB.algo = "NotWorn",
     HASPT.ignore.invalid = FALSE,
     threshold.mod = c(100, 120),
     boutdur.in = c(25, 30),
     ignorenonwear = FALSE,
     save_ms5rawlevels = TRUE,
     save_ms5raw_without_invalid = FALSE)
```

#### Counts data

``` r
library(GGIR)

acc_thresholds = c(100, 2500, 10000, 15000) # (1)!
acc_thresholds = acc_thresholds * c(5/60) # (2)!
acc_thresholds = round(acc_thresholds, digits = 2) # (3)!

GGIR(datadir = "C:/path/to/your/data/folder",
     outputdir = "C:/path/to/your/output/folder",
     dataFormat = "actigraph_csv",
     mode = 1:5,
     overwrite = FALSE,
     do.report = c(2),
     windowsizes = c(1, 900, 3600),
     threshold.lig = acc_thresholds[1],
     threshold.mod = acc_thresholds[2],
     threshold.vig = acc_thresholds[3],
     extEpochData_timeformat = "%Y/%m/%d %H:%M:%S",
     do.neishabouricounts = TRUE,
     acc.metric = "NeishabouriCount_x",
     HASPT.algo = "NotWorn",
     HASIB.algo = "NotWorn",
     boutdur.mvpa = 10, # (4)!
     boutdur.in = 30,
     boutdur.lig = 10,
     do.visual = TRUE,
     includedaycrit = 10,
     includenightcrit = 10,
     visualreport = FALSE,
     outliers.only = FALSE,
     save_ms5rawlevels = TRUE,
     ignorenonwear = FALSE,
     HASPT.ignore.invalid = FALSE,
     save_ms5raw_without_invalid = FALSE)
```

1. Light (2500), moderate (10000), and vigorous PA (15000) thresholds.
2. Assumes GGIR's default epoch length of 5 seconds.
3. Rounding thresholds to two decimal places.
4. Parameters *boutdur.mvpa*, *boutdur.in* and *boutdur.lig* can also be vectors.

### 2. Merge GPS and accelerometer data with hbGPS

If your accelerometer data were processed by the GGIR package, you can merge them with GPS files using the hbGPS package.

#### Configuration

| Parameter               | Description                                                                                                                                       |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| `idloc`                 | Option to find participant ID. More on methods of finding ID can be found in [GGIR documentation](https://cran.r-project.org/web/packages/GGIR/). |
| `gps_file`              | Path to input file or folder.                                                                                                                     |
| `outputDir`             | Path to output folder.                                                                                                                            |
| `GGIRpath`              | Path to GGIR output folder (ms5).                                                                                                                 |
| `outputFormat`          | indoor-outdoor-vehicle.                                                                                                                           |
| `time_format`           | Correct [date and time format](https://sparkbyexamples.com/r-programming/dates-and-times-in-r/).                                                  |
| `tz`                    | Timezone in which experiments took place ([TZ identifier list](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)).                    |
| `AccThresholds`         | Acceleration thresholds corresponding to the intensity levels.                                                                                    |
| `maxBreakLengthSeconds` | Maximum trip break duration in seconds.                                                                                                           |
| `minTripDist_m`         | Minimum trip distance in meters.                                                                                                                  |
| `minTripDur`            | Minimum trip duration seconds.                                                                                                                    |
| `threshold_snr`         | Threshold for SNR.                                                                                                                                |
| `threshold_snr_ratio`   | Threshold for SNR ratio.                                                                                                                          |

#### Example

``` r
library(hbGPS)

acc_thresholds = c(100, 2500, 10000, 15000) # (1)!
acc_thresholds = acc_thresholds * c(5/60) # (2)!
acc_thresholds = round(acc_thresholds, digits = 2) # (3)!

hbGPS(idloc = 2,
      gps_file = "C:/path_to_input_file/or/folder",
      outputDir = "C:/path_to_output_folder",
      GGIRpath = "C:/path_to_GGIR_output_folder/meta/ms5.outraw", # (4)!,
      outputFormat = "PALMS",
      time_format = "%Y/%m/%d %H:%M:%S",
      tz = "Europe/Copenhagen",
      AccThresholds = acc_thresholds, # (5)!
      maxBreakLengthSeconds = 120,
      minTripDur = 60,
      minTripDist_m = 100,
      threshold_snr = 225,
      threshold_snr_ratio = 50)
```

1. Light (2500), moderate (10000), and vigorous PA (15000) thresholds.
2. Assumes GGIR's default epoch length of 5 seconds.
3. Rounding thresholds to two decimal places.
4. The assumption is that GGIR has already been executed, and an output folder has been created.
5. Instead of specifying a thresholds for the accelerometer, you can use the *"default"* option.

## License

This project is licensed under the terms of the Apache License 2.0.

