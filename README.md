# XyGrib — DrCalambre fork

Companion fork of [frfa/XyGrib](https://github.com/frfa/XyGrib).

## Background: keeping XyGrib usable after the opengribs server went down

When the opengribs server went down in September 2026, XyGrib could no longer fetch
GFS and WW3 wave data through it. During the following weeks, the workaround was a
separate Bash script,
[GFS_NOAA_NOMADS_Downloader_for_XyGrib-72h_GRIB](https://github.com/DrCalambre/GFS_NOAA_NOMADS_Downloader_for_XyGrib-72h_GRIB2_Forecast),
                                                 
which fetched the GRIB2 files with `filter_gfswave.pl` and respected NOMADS' request
pacing by hand. That script kept the wave forecast usable in XyGrib until this fork's
native provider took over.

## What this fork adds

- **Native NOMADS GFS/GFS-Wave provider:** downloads GFS + WW3 wave files directly from
  NOMADS, doing internally what the external script used to do. No wrapper needed.
- **NOMADS pacing:** enforces the 10 s between fetches requested by NOMADS (SCN 21-32),
  with a visible countdown in the download dialog.
- **Swell (surfaceType1=241):** makes WW3 swell height/period/direction readable, so
  "Swell" shows up in the MeteoTable instead of being discarded.
- **CSNOW / CFRZR:** downloads CSNOW (categorical snow) and CFRZR (categorical freezing
  rain) in the same NOMADS request; `sde` (snow depth) is displayed.

## Upstream status

- **PR #4** (native NOMADS provider) and **PR #5** (CSNOW/CFRZR) are open against
  `frfa/XyGrib`, as single-commit branches cherry-picked onto `frfa/main` (no unrelated
  commits).
- The **pacing + countdown** work stays local, pending #4.

## What about the script?

The script is **not deprecated**. It remains useful for automated, unattended tasks —
for example a cron (or anacron) job that refreshes a 72 h GRIB file on a schedule,
independent of the GUI. The native provider and the script now cover different needs:
interactive use inside XyGrib, and scheduled/headless fetching outside it.

## Build

The fork does not bundle `aecunpack.c`. Building against a clean `frfa/main` requires
that file locally (see PR #4 for details). Windows: see `BUILD-WINDOWS.md`.

---

## Upstream README (inherited, unchanged)

# XyGrib - Weather Forecast Visualization
Fork of zyGrib-8.0.1 under a new name and a new home https://opengribs.org

Changes in version 1.2.8
- Chinese translation added. Thanks to 张 树奎

Changes in version 1.2.6
- Macbook touchpad zooming of map issue fixed
- Belgium NMC opendata gribs can now be read by XyGrib
- GFS reflectivity added
- ECMWF ERA5 reanalysis data can now be read
- NOADD GFS Ensemble model can now be read
- Improvements in wave display
- utf-8 characters in file paths should be readable
- Many behind the scenes improvements and bug fixes

Changes in version 1.2.4
- New feature - Arome 0.025 deg regional model has been added covering all of France and neigbouring areas
- Grib grid only shown if there is weather display selected and if zoom factor allows it (no more black out)
- Many code improvements thanks to Didier
- Bug fixes including crashes on closing POI Editor or opening MeteoTable


Changes in version 1.2.2 :
- New feature - Gust color map that can show absolute gust velocity values or relative (delta) values above the wind speed. Absolute gust velocity is the default. This can be changed in the weather options sub-menu.
- Transparency added to lowest color values for Precipitation, CAPE, Reflectivity, Gust (delta) and Snow. This improves the color overlay displays.
- Functionality for viewing non-global model coverage is improved and now zooms on the shaded area.
- Cloud display problems with ICON gribs is fixed.
- Enforcement of minimum area for grib requests is fixed. "Bad Request" errors are no longer returned on very small area requests.
- Several other bugs were fixed and several code stability improvements were included.


Changes in version 1.2.0 :
- Major enhancement to Grib Server and to XyGrib with the addition of large area regional models. These include NAM (CONUS, Central America & Carribean, Pacific & Hawai), ICON-EU and Arpege-EU.
- ECMWF added to global models albeit a limited free public release of the model including some surface and altitude data at 0.5 deg resolution and 24hr temporal resolution.
- Bug fixes
- Many maintenance improvements on stability and speed (Thanks to Didier)


Changes in version 1.1.2 :
- Mainly bug fixes regarding the saving of settings and a few others
- New installer system applied now to Mac OS
- Native menu system now used on Mac OS (Thanks to Theeko)


Changes in version 1.1.1 :
- Build system changed from qmake to cmake (Thanks to Pavel Kalian)
- Major clean up and improvements to code and stability (Thanks to Didier)
- Jpeg2000 compression in Grib files is now working properly. Wave data on the grib server will included this compression soon. (Thanks to Didier)
- Application and data have been separated. The app goes to where application should be installed and static data goes to where applicaiton data needs should be located.
- New installer system for binaries. The installer downloads app and static data from an online repository. Installation includes optional install of high resolution maps. It also installs a maintenance tool that can be used to install updates, add/remove components or uninstall all.


Changes in version 1.1.0 :
- Interactive GRIB downloads is now from the OpenGribs' on-the-fly grib server with
  multiple models to choose from. These include: GFS, ICON and Arpege Global model
  and WW3, GWAM and EWAM wave models.
- Opened the filter logic of accepting only gribs from recognized sources. 
  Now any valid grib file can be read.
- Improvements in the use of alternate parameters for the same result such as
  Total Precipitation and Precipitation Rate.
- A startup check for new versions is now build-in. This can also be performed
  from the help menu group.
- Significant cleaning up of the code
- Hebrew language was added
- Select/Pan toggle (Thanks to Andrew Tseng)


Changes in version 1.0.1 :
- Addition of new meteorological parameter 'Composite Simulated Reflectivity'. This supports visualisation of severe convection
- Inclusion of DWD ICON gribs
- Addition of Grib2 handling for wave data
- Enabling of Gusts at both surface or 10 meters
- Name change of the application to XyGrib
- Inclusion of Norwegian Meteorological Institute and NRK Gribs
- Inclusion of German Hydrographic Office current gribs
- Switchable dark/light skin (in Options menu)
- Help points to Wiki (WIP)

