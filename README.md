tornado-risk-map
================

Builds the monthly "Average Tornado Risk Area" graphic from the Storm
Prediction Center's tornado database. A single-file MapLibre GL application
that downloads every US tornado on record, turns a chosen month into a
three-tier climatology and exports a finished PNG in the Live Storm Chasers
style.

Copyright (c) 2026 Live Storm Chasers Network LLC. All rights reserved.
See LICENSE.


Shape
-----

    index.html      the application. Data loading, climatology, contouring,
                    map rendering, overlay drawing and export all live here.

There is no build step, no bundler and no dependency graph: the deployed
artefact is the source. Libraries load from jsDelivr at pinned versions.
Deploy by pushing to the branch Cloudflare Pages serves; the output directory
is the repository root.

Opening the file inside an app's preview pane does not work. Previews block
outside downloads, so the tiles, outlines and SPC data never arrive. The page
detects this and says so. Open it in Chrome or serve it.


Versioning
----------

The version string lives in `APP_VERSION` near the top of the script and is
shown at the foot of the control panel with the build date in `APP_BUILD`. Both
move together. The commit title carries the version:

    Version 1.0.0.0 — short description


How the numbers are made
------------------------

    source      spc.noaa.gov/wcm/data/1950-YYYY_actual_tornadoes.csv,
                newest year tried first. SPC serves it with open CORS, so
                the browser fetches it directly. Cached in IndexedDB for
                30 days.
    grid        0.2 degrees, 127W to 64W, 22N to 52N
    hit         a cell scores a year if any tornado passed within 25 statute
                miles of it during the month. Tracks are sampled every 10 km
                from start to end point, so a long-track tornado counts along
                its whole path, not just where it touched down.
    value       years with a hit divided by years in the period. A value of
                0.33 means one year in three.
    period      the full record, 1950 to the last complete year, by default.
                The last 30 years, 1991-2020 normals and the last 20 years
                are offered. A year that ends before December
                in the file is treated as incomplete and left out.
    halves      days 1-15 and 16-end are binned separately, so "Early April"
                is its own climatology, not half of April's.
    smoothing   Gaussian, 110 miles by default, run as a normalised
                convolution against a land mask so coastal values are not
                dragged down by ocean zeros
    tiers       "TWC style" by default, calibrated against all twelve of
                The Weather Channel's 2025 monthly maps:
                  Possible  fixed 2 percent (1 year in 50)
                  Greater   49 percent of the month's peak
                  Greatest  78 percent of the month's peak
                Greater is drawn in Feb-Sep, Nov and Dec; Greatest in
                Mar-Jun, the months TWC shows them. "Same all year" (fixed
                chances) and "Relative" (all three tiers every month) remain
                as options. The panel shows the real percentages behind each.
    clipping    contours are intersected with the contiguous US outline and
                the Great Lakes are cut out with Natural Earth 1:50m lakes

The 25-mile radius matches the definition SPC uses in its outlooks, so the
numbers read the same way a forecaster's would.


Things that cost time and are worth carrying
--------------------------------------------

The SPC CSV is not sorted. The last row in the file is not the last tornado
of the year, so completeness has to be judged from the latest month present,
not the final line.

Natural Earth's land and country polygons include the Great Lakes as land.
Clipping risk areas to them paints Lake Michigan red. The lakes have to be
subtracted separately, and drawn above the state lines so the borders that run
through the water disappear.

Which tiers appear in which month cannot be derived from the data alone. In
the climatology January and February peak within a point of each other, and
so do March and July, yet TWC gives February a Greater area and January none,
and March a Greatest area and July none. No single rule, fixed or relative,
reproduces all twelve maps, so the TWC months are written down as a table
(`TWC_SEASON`). The extents are data-driven; only which tiers exist is fixed.

The period matters as much as the cutoffs. Matched against TWC's maps, the
full 1950-present record fits far better than the last 30 years: November's
Possible area reaches Kansas City and leaves the Appalachian notch, as TWC's
does, only on the full record.

The PNG is composed in three passes: starfield, map canvas, overlay. The map
canvas is grabbed inside a `render` callback after `triggerRepaint`, while the
frame is still in the drawing buffer. Reading it at any other moment can return
a blank map unless `preserveDrawingBuffer` is honoured, and that is not worth
depending on.


Not done
--------

Alaska, Hawaii and Puerto Rico are filtered out of the data and the clip.

The climatology counts presence, not frequency. Ten tornadoes in one year and
one tornado score the same for that year, which is the intended reading but
not the only possible one.

There is no batch export of half-month periods. "Save all 12 months" exports
whole months, or the chosen half of each month.


Attribution
-----------

Tornado reports come from the NOAA Storm Prediction Center, satellite imagery
from NASA GIBS (Blue Marble Shaded Relief and Bathymetry), state outlines from
the US Census Bureau via us-atlas, and country outlines, coastlines and lakes
from Natural Earth. All are in the public domain. Place search uses
OpenStreetMap Nominatim; its results are (c) OpenStreetMap contributors under
the Open Database Licence. This repository contains no weather data.

Libraries loaded at run time, each under its own licence: MapLibre GL JS 5.6.0
(BSD 3-Clause), d3-array 3.2.4 and d3-contour 4.0.2 (ISC), topojson-client
3.1.0 (ISC), polygon-clipping 0.15.7 (MIT).


Licence
-------

All rights reserved. See LICENSE. The libraries above keep their own licences
and are unaffected.
