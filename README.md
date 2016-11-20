# inkscape-embroidery: An Inkscape plugin for designing machine embroidery patterns
## Introduction
**Want to design embroidery pattern files (PES, DST, etc) using free, open source software?  Hate all the other options?  Try this one.**

I received a really wonderful christmas gift for a geeky programmer hacker: an [embroidery machine](http://www.brother-usa.com/homesewing/ModelDetail.aspx?ProductID=SE400).  It's pretty much a CNC thread-bot... I just had to figure out how to design programs for it.  The problem is, **all free embroidery design software seems to be terrible**, especially when you add in the requirement of being able to run in Linux, my OS of choice.

So I wrote one.

Okay, not really.  I'm pretty terrible at GUIs, but I found this nifty inkscape extension that was created and hacked on by a couple of other folks.  It was pretty rudimentary, but it got the job done, and more importantly, it was super hackable.  I hacked the hell out of it, and at this point **inkscape-embroidery is a viable entry-level machine embroidery design tool**.


## Setup

To use this tool, you're going to need to set it up.  It's an inkscape extension written as a Python file.  Once you get it working, you'll need to learn how to design vectors in the way that inkscape-embroidery expects, and then you can generate your design files.

### Inkscape
First, install Inkscape if you don't have it.  I highly recommend the **development version**, which has a really key feature: the Objects panel.  This gives you a heirarchical list of objects in your SVG file, listed in their stacking order.  This is really important because the stacking order dictates the order that the shapes will be sewn in.

I've had success running version `0.91.0+devel+14591+61`.  Installation instructions are [here](https://inkscape.org/da/release/trunk/).

### Python Dependencies
Make sure you have the `shapely` python module installed.  The `appdirs` python module is also useful but is not required.  On Ubuntu:

```
apt-get install python-shapely python-appdirs
```

### Extension installation
1. Clone the extension source: `git clone https://github.com/lexelby/inkscape-embroidery`
2. Install it as directed [here](https://inkscape.org/da/gallery/%3Dextension/)

I prefer to symbolically link into my git clone, which allows me to hack on the code.  Changes to the Python code take effect the next time the extension is run.  Changes to the extension description files (`*.inx`) take effect the next time Inkscape is restarted

### Optional: conversion program
The extension can output machine embroidery design files directly in Melco format.  I don't even know what that is, so I don't use it.  I prefer to use the **CSV output format** which can be consumed by another awesome open source project: [Embroidermodder2](https://github.com/Embroidermodder/Embroidermodder).  In theory, this project was going to be exactly what I wanted.  In practice, it never got funded on Kickstarter and it's largely incomplete.

However, it contains a really awesome core library that knows pretty much every machine embroidery format and how to convert between them.  I use it to convert the CSV files that inkscape-embroidery outputs into the PES files that my SE400 uses.

Grab the source: `git clone https://github.com/Embroidermodder/Embroidermodder`.  Build just `libembroidery-convert` using the instructions in "3)" in the [Embroidermodder build docs](https://github.com/Embroidermodder/Embroidermodder/wiki/Compiling-parts-of-the-project). You can then use it like this: `./libembroidery-convert your-file.csv your-file.pes`.

Since the CSV + libembroidery-convert method is the only method I use, it's the one I'll assume from here on.  I'm not even sure if the other output formats from inkscape-embroidery still work (or ever worked).

## Usage
### Basic Usage
First things first: I'm going to assume you know a few embroidery terms like "fill stitch" and "satin".  Look those up if you're mentally 404ing, then come back here.  I'm *not* going to assume you know some of the more advanced terms, because I had to learn all that when I started this project, so I might as well teach you too.

1. Open up Inkscape and create a rectangle.
2. Make sure it has both a stroke and a fill.
3. Convert it to a path using **Path -> Object to Path** (because inkscape-embroidery doesn't understand rectangles, circles, and the like, and ignores them).
4. Run **Extensions -> Embroidery -> Embroider**.  Use the default settings.

The rectangle you made will disappear and be replaced with some stripes and zig-zags.  inkscape-embroidery has hidden all of your layers and created a new one called Embroidery, in which it has palced a visual representation of the stitch plan it created.  It has interpreted your shape as two instructions: Fill and Stroke.  Fill is implemented using fill stitching, and Stroke is implemented by running satin stitching along the outline.

Select the horizontal lines using the "Edit Paths by Nodes" tool.  Zoom in a bit and you'll see that the lines are actually made up of lots of points.  Each point represents one stitch -- one needle penetration and interlocking of the top thread with the bobbin thread.  Notice how the points all line up nicely in diagonals.  This will give the fill stitching a nice, orderly visual appearance.

Now look at the zig-zags.  These are the satin stitches.  Note that the corners look pretty ugly.  This is because satin stitches generated from a shape's stroke are pretty rudimentary and aren't implemented intelligently.  You can exert much greater control over satin stitching using a Satin Column, described later.

The stitching preview you're looking at just now isn't intended to be permanent.  I usually immediately undo it (ctrl-Z) after I've looked at the stitches.  The actual work that inkscape-embroidery does is to output a design file.

### Stitching Out the Design
Where'd the design file go?  One of the parameters you were able to specify in the filter settings dialog was the output directory.  By default, the directory used is the place where you installed the extension's Python files.  I output mine to `~/Documents/embroidery/output`.

inkscape-embroidery will create a file named `something.csv`, where `something` is the name of your svg file (e.g. `something.svg`).  If `something.csv` already existed, it will be renamed to `something.csv.1`, and `something.csv.1` will be renamed to `something.csv.2`, etc, up to 5 backup copies.  When you've got the design the way you like it, save off a copy of `something.csv`.

Next, convert it to your machine's format using `libembroidery-convert` (as described above).  Send it to your machine in whatever way one does that for your machine, and try stitching it out!

### Ordering

Copy your rectangle and paste it elsewhere on your canvas.  Deselect any shapes (**Edit -> Deselect**), re-run the extension, and look at the output.  You'll see that both regions have been stitched, and there will be a line connecting them.  That's a jump-stitch, where the machine will move a long distance between stitching the sections.

If you're like me, your machine can't automatically cut the thread between stitching sections, so you'll need to minimize jump stitches as much as possible through careful planning of your stitch path.  If your machine *can* do thread cuts, congratulations!  But you'll need to modify inkscape-embroidery to allow you to specify a thread cut, because there's no way to do that right now.

However, note that inkscape-embroidery pays attention to the colors you use for objects.  If you change colors from one object to the next, inkscape-embroidery will include a color-change instruction using the color you've set for the object.  My machine cuts the thread and waits for me to switch to the new thread color.

#### Reordering

Use the Objects panel to view the stacking order of the objects in your SVG file.  Inkscape-embroidery will stitch them in their stacking order, from lowest to highest.  You can reorder them in the normal way in inkscape to affect the stitching order.

You can also use the Reorder extension.  Hold shift and select the objects you'd like to reorder, one at a time, in the order you'd like them to end up in (lowest to highest).  Run **Embroidery -> Reorder**.  This extension will pull all of the selected objects out of wherever they were in the stacking order and insert them in order at the same place as the *first* object you selected.  This can save you a ton of time.

### Seeing the stitch plan for selected objects

If you have one or more objects selected when you run the **Embroider** extension, only those objects will be embroidered.  This can be useful to help you fine-tune just one small section of your design.

### Embroidery Parameters
When you run **Embroider**, you'll have the option to specify a few parameters like stitch length, fill stitch row spacing, etc.  These are used as defaults for all objects in your design.  You can override these parameters and set many more using the **Embroidery -> Params** extension.

This extension gives you an interface to control many aspects of the stitching of each object individually.  To use it, first select one or more objects.  Parameters will be applied to them all as a group.  If the selected objects already have parameters set, these settings will be pre-loaded into the interface.

Parameters are stored in your SVG file as additional attributes on the XML objects.  You can view these attributes using Inkscape's built-in XML editor panel, but you shouldn't actually need to do this during normal usage.  Inkscape ignores attributes that it doesn't know, so these attributes will be saved right along with your SVG file.  Note that other SVG programs may *not* retain these attributes, so be careful!

I recommend avoiding dependence on the default settings specified in the **Embroider** extension's settings window.  In fact, I bypass it entirely by binding a keystroke (ctrl+e) to "Embroider (no preferences)" in Inkscape's settings.  This way, I can quickly see the stitch plan just by pressing the keystroke.  I also bind a keystroke to **Params** so that I can quickly view and change settings for each object.

### Sidenote on extensions
**Params** is a bit weird, in that the dialog is produced by an entirely separate program (the extension) rather than Inkscape itself.  This is due to the way Inkscape structures extensions.  I wish inkscape-embroidery could have deeper integration into Inkscape's user interface, but it's currently not possible.  This is the price we pay for not having to write an entire vector graphics editor program :)

Another issue is that Inkscape has a memory leak related to extensions.  The more times you run an extension, the more memory Inkscape uses and the slower it gets.  I periodically save my SVG file, close Inkscape, and restart it to work around this issue.  See above re: putting up with this kind of hassle so as not to have a to implement an entire vector graphics editor.  Hopefully they'll fix this bug soon.

### AutoFill

AutoFill is the default method for generating fill stitching.  To use it, create a closed path in Inskcape and add a fill color.

inkscape-embroidery will break the shape up into sections that it can embroider at once using back-and-forth rows of stitches.  It then adds straight-stitching between sections until it's filled in the entire design.  The staggered pattern of stitches is continued seamlessly between sections, so the end result doesn't appear to have any breaks.  When moving from one section to the next, it generates running stitching along the outside edge of the shape.

This algorithm works great for simple shapes, convex or concave.  However, it doesn't work for shapes with holes, because the stitching could get "stuck" on the edge of a hole and be unable to reach any remaining section.  For this reason, AutoFill rejects regions with holes in them.

So what do you do if your shape does have holes?  You have two choices: use manually-routed fill (described below), or break the shape up into one or more shapes without holes.

Here's an example of converting a region with a hole into a region without:
![breaking auto-fill regions with holes](images/autofill_with_holes.png)

An SVG version is available in `images/autofill_hole_example.svg` for you to test out.

Note the thin line drawn from the hole to the edge.  In fact, this is a very thin strip missing from the shape -- thinner than the spacing between the rows of stitches.  This allows the autofill system to travel into and out of the center of the shape if necessary to get from section to section.

Note that I've drawn the gap at exactly the same angle as the fill.  When the autofill system sees that it is traveling in the same direction as the fill, **it places stitches correctly to match the fill pattern**.  This means that the gap will be virtually undetectable because the travel stitches will be hidden in the fill.  This may double- or triple-up one of the fill rows, but it's really hard to tell unless you look very closely.

#### AutoFill parameters

Using the **Params** extension, you can set these parameters:

* **angle**: The angle of the rows of stitches, in degrees.  0 is horizontal, and the angle increases in a counter-clockwise direction.  Negative angles are allowed.
* **row spacing**: distance between rows of stitches
* **maximum stitch length**: the length of each stitch in a row.  "Max" is because a shorter stitch may be used at the start or end of a row.
* **running stitch length**: length of stitches around the outline of the fill region used when moving from section to section
* **staggers**: stitches are staggered so that neighboring rows of stitches don't all fall in the same column (which would create a distracting valley effect).  Setting this dictates how many rows apart the stitches will be before they fall in the same column position.
