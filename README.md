# ProPlot
## Overview

This library provides helpful and versatile plotting utilities to hasten the process of crafting publication-quality graphics with `matplotlib`. I recommend importing the package with
```
import proplot as plot
```
Most of the features below derive from the **`subplots`** command, inspired by the `pyplot` command of the same name. This generates a scaffolding of axes and panels.

Next is the **`format`** method. This method is assigned to every axes generated by `subplots`, and can be used to control the look of your axes with a plethora of keyword arguments.
  
Quick overview of additional features:

  * Geometry: Specify axes aspect ratios without messing up subplot spacing. Panels and inter-axes spaces are held *fixed* in inches, while the figure width or height and individual axes sizes are allowed to change. Border space is trimmed (expanded) without messing with inter-axes spacing. *Incredibly* useful for meeting journal figure-size requirements.'
  * Colors: Provided group of perceptually distinct named colors, powerful colormap-generating tools, ability to trivially swap between "color cycles" and "colormaps". A few new, beautiful colormaps and color cycles are provided. Create colorbars from lists of lines or color strings.
  * Maps: Integration with basemap *and* cartopy. Generate arbitrary grids of map projections in one go. Switch between basemap and cartopy painlessly. Add geographical features as part of the `format` process.

## How is this different from seaborn?
There is already a great matplotlib wrapper called [seaborn](https://seaborn.pydata.org/). What makes this project different? While some of the utilties here were inspired by seaborn (in particular `colors.py` takes its inspiration from seaborn's `palettes.py`), the goals for this project are quite different. I endeavored to create a package that was *extremely* efficient at the task of **crafting beautiful, precise, publication-ready figures, and no more**.

When using ProPlot, your data should be analysed *prior* to plotting, while in seaborn the tasks of data analysis and plotting are generally wrapped into one. And as an atmospheric scientist, the datasets I use usually do not lend themselves to fitting in a simple DataFrame -- so this was not particularly useful for me. For data analysis tools I use in my physical climatology research, check out my [ClimPy](https://github.com/lukelbd/climpy`) project (still in preliminary stages).

By focusing on this one task -- creating beautiful figures -- I've created a number of **immensely powerful** features well beyond the scope of `seaborn` as a data analysis-and-visualization package. While some `rc` defaults have been improved, these tools are not intended to give you the final product on a silver platter. Instead, I expect you to tinker with your plots. This package simply makes this process much, much easier.

## Installation
This package is a work-in-progress. Currently there is no dedicated `github.io` documentation and no formal releases on PyPi. However, feel free to install directly from Github using:

```
pip install git+https://github.com/lukelbd/proplot.git#egg=proplot
```

I only push to this repo when new features are completed and working properly.

Dependencies are `matplotlib` and `numpy`. If you want to use the mapping features, you will also need `basemap` and/or `cartopy`. Note that [basemap is no longer under active development](https://matplotlib.org/basemap/users/intro.html#cartopy-new-management-and-eol-announcement) -- cartopy is integrated much more intelligently with the matplotlib API, and therefore has more room for growth. However, for the time being, basemap retains several advantages over cartopy (namely [more tools for labeling meridians/parallels](https://github.com/SciTools/cartopy/issues/881) and more available projections -- see [basemap](https://matplotlib.org/basemap/users/mapsetup.html) vs. [cartopy](https://scitools.org.uk/cartopy/docs/v0.15/crs/projections.html)). Therefore basemap may be preferred in some circumstances.

## Donation
This package took a shocking amount of time to write. If you've found it useful, feel free to buy me a cup of coffee :)

[![Donate](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=5SP6S8RZCYMQA&source=url)

# Documentation
## The `subplots` command
### Basic usage
To generate complex grids, pass a 2D array of numbers corresponding to unique subplots. Use zero to allot empty space. For example:
```
f, axs = plot.subplots(array=[[1,2],[1,3]])
```
creates a grid with one tall plot on the left, and two smaller plots on the right, while
```
f, axs = plot.subplots(array=[[1,1,1],[2,0,4]])
```
creates a grid with one long plot on top and two smaller plots on the bottom with a gap in the middle (the zero). The axes "number" is added as an axes attribute.

Pass no arguments to generate default single-axes figure (1 row, 1 column), or use `nrows` and `ncols` to generate simple subplot grids. For example:
```
f, axs = plot.subplots(nrows=2)
```
creates two rows with one column. Note that if you did not use `array`, the subplots are automatically numbered in row major order.

The subplots command returns two arguments: the figure handle `f` and a special list of axes, `axs`. Note that **this is no ordinary list** -- you can bulk-invoke methods on every multiple axes simultaneously, no for-loop necessary. For example:
```
f, axs = plot.subplots()
axs.format(xlim=(0,1)) # set to be identical everywhere
axs[:3].format(color='red') # make axis labels, spines, ticks red
axs[3:].format(color='blue') # same, but make them blue
```

Additional sizing keyword arguments are `wspace`, `hspace` (width/height spacing between axes in *inches*), `wratios`, `hratios` (width/height ratios for columns/rows), `bwidth`, `rwidth`, `lwidth` (panel widths in *inches*), and `top`, `bottom`, `left`, `right` (border widths in *inches* -- although by default, excess space will be trimmed, so these arguments have no effect in the end; see below).

### Journal sizes and units
To specify create figures that conform to journal standards, use e.g. `width='ams1'`. Some standards actually specify a width and height together -- `width='name'` will fix both dimensions. Check out `subplots.py` for the currently available ones, and **feel free to contact me with additional journal standards -- I will add them**.

Also, while the numeric sizing arguments are assumed to be in **inches**, you can specify sizes with **arbitrary units** using e.g. `width='12cm'`, `wspace='5mm'`, `hspace='3em'` (3 em squares), etc.

### A smarter subplot layout and a new GridSpec class
If you specify *either* (not both) `width` or `height`, optionally with an aspect ratio `aspect` (default is `1`), the figure **height or width will be scaled** such that the top-left subplot has aspect ratio `aspect`. The **inter-subplot spacing and panel widths are held fixed** during this scaling.

The above allowed me to create the **`smart_tight_layout`** method (which by default is **called whenever the figure is drawn**). Previously, `tight_layout` could be used to fit the figure borders over a box that perfectly encompasses all artists (i.e. text, subplots, etc.). However, because `GridSpec` spaces are relative to the subplot dimensions, changing the figure dimensions *also* changes the inter-subplot spacings. Since your font size is specified in points (i.e. a *physical* unit), *this can easily cause text to overlap with other subplots where they didn't before*. The new `smart_tight_layout` method draws a tight bounding box that **preserves inter-subplot spacing, panel widths, and subplot aspect ratios**.

To achieve this, I created a new `FlexibleGridSpec` class. The actual `wspace`, `hspace` passed to `GridSpec` are zero -- the spaces you see in your figure are `GridSpec` slots masquerading as spaces. Check out `FlexibleGridSpec.__getitem__`. Also, inter-subplot spacing is now *variable*: specify `wspace` and `hspace` as 1) a scalar constant or 2) a list of different spacings.
      
## Inner and outer "panels"
Use `[bottom|right]panel=True` to allot space for panels spanning all columns (rows) on the bottom (right). Use `[bottom|right]panels=True` to allot space for one panel per column (row). Use `[bottom|right]panels=[n1,n2,...]` to allot space for panels that can span adjacent columns (rows). These add `fig.[bottom|right]panel` attributes to the figure `fig` (access the nth panel with `fig.[bottom|right]panel[n]`).

Example: If your subplot has 3 columns, passing `bottompanels=[1,2,2]` draws one panel for the first column and a second panel spanning the next two. You can also use `[bottom|right][colorbar|legend][s]=True` to modify the panel width and spacing to be *suitable for colorbars/legends*, e.g. `rightcolorbar=True`.

Use `fig.[bottom|right]panel.[legend|colorbar]` to fill the axes with a legend or colorbar. You must supply the `legend` with a list of handles, and supply `colorbar` with a mappable, list of lines, or list of colors. Alternatively, feel free to draw on the panels with `plot`, `contourf`, etc. like normal.

## Mapping toolkit integration
For projection subplots, specify `projection='name'` with either `package='basemap'` or `package='cartopy'`. Extra arguments to `subplot` will be passed to the `basemap.Basemap` and `cartopy.crs.Projection` classes (the relevant cartopy class will be selected based on the `'name'` string).

Control which subplots are projection subplots with `projection='name'`or  `projection={1:'name1', (2,3):'name2', 4:'name3'}`, where numbers correspond to the subplot array number.

Access basemap plotting utilities directly as an axes method, thanks to the `BasemapAxes` subclass. Several plotting methods are also overridden to fix issues with the "seam" on the edge of the map (data is circularly rolled and interpolated to map edges).
   
## The `format` command
To modify an axes property (e.g. an x-axis label) with the default API, you normally have to use a bunch of one-liner `pyplot` commands (or method calls on axes/axis objects). This can get repetitive and quite verbose, resulting in lots of boilerplate code.

Now, you can pass all of these settings to `format`. Instead of having to remember the name of the function, whether it's attached to `pyplot` or an object instance, and the order/names of the arguments, you just have to remember one thing -- the name of the keyword argument. The `format` method also abstracts away some inconsistencies and redundancies in the matplotlib API -- now, There's Only One (obvious) Way To Do It.

Example usage:

```
ax.format(xlabel='time (seconds)', ylabel='temperature (K)', title='20th century sea-surface temperature')
```

Titling options:

```
suptitle, suptitle_kw
title, titlepos, title_kw,
abc, abcpos, abcformat, abc_kw
```

Axis options:
```
xgrid, ygrid,
xspineloc, yspineloc,
xtickloc, ytickloc, xtickdir, ytickdir,
xtickminor, ytickminor, xgridminor, ygridminor,
xticklabeldir, yticklabeldir,
xtickrange, ytickrange,
xlim, ylim, xreverse, yreverse, xdates, ydates,
xscale, yscale, xscale_kw, yscale_kw,
xlabel, ylabel, xlabel_kw, ylabel_kw
xlocator, xminorlocator, xlocator_kw, xminorlocator_kw
ylocator, yminorlocator, ylocator_kw, yminorlocator_kw
xformatter, yformatter, xformatter_kw, yformatter_kw
```

Mapping options:

```
latlabels, lonlabels,
latlocator, latminorlocator, lonlocator, lonminorlocator 
land, ocean, coastline
```

Axes canvas options:

```
hatch, color, facecolor, linewidth
```

## Axis scales, tick formatters, and tick locators
Added several new axis scales, which can be invoked with `[x|y]scale='name'` in `format()` calls:

* The "*inverse*" scale. Useful for, e.g., having wavenumber and wavelength on opposite sides of the same plot.
* The *sine-weighted* and *Mercator* axis scales. The former creates an area-weighted latitude axis.
* Special "*cutoff*" scales, that allow arbitrary zoom-ins/zoom-outs. This can be invoked with `[x|y]scale=('cutoff', lower, upper, scale)` where `lower` and `upper` are the boundaries within which the axis scaling is multiplied by `scale`. Use `np.inf` for a hard cutoff.

Added new default `Formatter` class for ticklabels, which renders numbers into the style you'll want 90% of the time. Also created several special formatter classes: use `ax.format([x|y]formatter='[lat|deglat|lon|deglon|deg]')` to format axis labels with cardinal direction indicators and/or degree symbols (as denoted by the names).

Added helpful tools for specifying tick locations:

* Use `ax.format([x|y]locator=N)` to tick every `N` data values.
* Use `ax.format([x|y]locator=[array])` to tick specific locations.
* Use `ax.format([x|y]locator='[string]')` to use any of the `matplotlib.ticker` locators, e.g. `locator='month'` or `locator='log'`.

Finally, use `plot.arange` for generating lists of contours, ticks, etc. -- it's like `np.arange`, but is **endpoint-inclusive**.

## Enhanced settings management
Use `plot.rc` as your one-stop shop for changing global settings. This is an instance of the new `rc_configurator` class, created on import. It can be used to change built-in `matplotlib.rcParams` settings, a few custom "`rcSpecial`" settings, and some special "global" settings that modify several other settings at once.

Quick summary of the "global" settings:
* Use `plot.rc['linewidth'] = 2` or `plot.rc.linewidth = 2` to increase the thickness of axes spines, major tick marks, and minor tick marks.
* Use `plot.rc['color'] = 'red'` or `plot.rc.color = 'red'` to make all spines, tick marks, tick labels, and axes labels red.
* Use `plot.rc['small']` and `plot.rc['large']` to control axes font sizes.
* Update any arbitrary `rcParam` with e.g. `plot.rc['legend.frameon'] = False` or `plot.rc.legend['frameon'] = False`.

Use `plot.rc.reset()` to reset everything to the initial state. By default, **this is called every time a figure is drawn** (i.e. when a figure is rendered by the matplotlib backend or saved to file).

## Colormaps, color cycles, and color names
Added **new colormap class** analagous to `LinearSegmentedColormap`, called `PerceptuallyUniformColormap`, which interpolates through hue, saturation, and luminance space (with hues allowed to vary circularly). Choose from either of 4 HSV-like colorspaces: classic HSV, perceptually uniform HCL, or HSLuv/HPLuv (which are forms of HCL adapted for convenient use with colormaps; see [this link](http://www.hsluv.org/comparison/)).

Generate perceptually uniformly varying colormaps on-the-fly by passing a **dictionary** to any plotting function that accepts the `cmap`keyword argument -- for example, `ax.contourf(..., cmap=dict(h=hues, l=lums, s=sats))`. The arguments can be lists of numbers or **color strings**, in which case the corresponding channel value (hue, saturation, or luminance) for that color will be looked up and applied.

Create single-hue colormaps on-the-fly by passing a string that looks like `cmap='name'` or `cmap='nameXX'`, where `name` is any registered color string (the corresponding hue will be looked up) and `XX` is the lightness value.

New colormaps have been added as `PerceptuallyUniformColormap` instances. New color cycles are also available (i.e. the automatic color order when drawing multiple lines). Cycle can be set with `plot.rc.cycle = 'name'` or by passing `cycle='name'` to any command that plots lines/patches (`plot`, `bar`, etc.).

The distinction between "colormap" and "color cycle" is now fluid -- all color cycles are defined as `ListedColormaps`, and cycles can be generated on the fly from `LinearSegmentedColormap`s by just specifying the colormap name as the cycler.

New colors have been added from XKCD colors, crayon colors, and Open Color web-design color palette. Colors that aren't sufficiently perceptually distinct are eliminated, so its easier to pick from the color table.

Use functions `cmap_show`, `color_show`, and `cycle_show` to visualize available colormaps, named colors, and color cycles. Functions will automatically save PDFs in the package directory.

## Revised underlying issues with contour and pcolor commands
Flipped the unnatural default used by `pcolor` and `contour` functions: that `0`th dimension of the input array is `y`-axis, `1`st dimension is `x`-axis. More intuitive to enter array with `0`th dimension on `x`-axis.

The well-documented [white-lines-between-filled-contours](https://stackoverflow.com/q/8263769/4970632)nd [white-lines-between-pcolor-rectangles](https://stackoverflow.com/q/27092991/4970632) problems are fixed by automatically changing the edgecolors when `contourf`, `pcolor`, and `pcolormesh` are called.
