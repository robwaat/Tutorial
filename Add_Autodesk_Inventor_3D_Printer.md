# Adding a Printer to Autodesk Inventor

## Overview
When generating stl files for 3D printing in Autodesk Inventor it is helpful to have the print volume visualisation correspond to the printer you intend to use.
This allows you to check that the part as designed will fit on the print bed and can be manufactured.
However, the default printer selection is very limited and is unlikely to describe the 3D printer in use.

This document therefore provides instructions on adding a custom 3D printer to the list of those usable in Autodesk Inventor's 3D printing stl generation environment.


## Steps
### 0. Pre-Amble
The printers known to Autodesk Inventor are defined in an xml file named `3DPrinterDescriptions.xml`.
If you're not familiar with xml files, this is a plain text document used to store data about a number of objects in a format that is easily understood by a computer.
It can then be read in to populate the attributes of an appropriate object in memory with little effort and re-written when the program exits, allowing data to be conveniently saved after the program is closed.

Here the xml file resembles the snippet below:
```xml
<Printers3D>
    <Printer internalName="BCF2EDCD-3DB2-48D6-83B9-8A5D9AB1C661" modelName="Default Machine Workspace" manufacturer="Autodesk Inc." technology="FDM" origin="0,0,0" units="mm" depth="450" width="420" height="400" thicknessThreshold="1" defaultprinter="0" favorite="1" />
    <Printer internalName="ED5911E2-2934-4822-B0CE-3C4F6B6E0EF6" modelName="Makerbot Replicator 2" manufacturer="Stratasys" technology="Extrusion" origin="BottomCenter" units="mm" depth="148" width="285" height="150" thicknessThreshold="1" defaultprinter="0" favorite="0" />
    ...
</Printers3D>
```
Each `<Printer ... >` line corresponds to one of the printers stored in Inventor's memory. 
They have a number of attributes but only a few are of interest.
These are indicted in the steps below, but a brief description for all attributes is given here:

- **internalName** This is a hexadecimal string used to uniquely name each printer known as a [Universally Unique Identifier](https://en.wikipedia.org/wiki/Universally_unique_identifier) (UUID). 
This helps the program identify each printer's settings in the case where they have other similar attributes.
- **modelName** This key defines the name of the printer.
- **manufacturer** This details the company that manufactured the printer and is used to group printers together into categories when accessed in through the Inventor GUI.
- **technology** This tag defines the extrusion technology employed by the printer.
Typically, this will be [Fused Deposition Modelling](https://en.wikipedia.org/wiki/Fused_filament_fabrication) (FDM) whereby the part is built up in a number of layers of melted plastic.
- **origin** Sets the default position of the part within the print bed. This can be a set of 3D coordinates of the form `"x,y,z"` (where z will usually be zero to ensure the part is not printed in mid-air) or a human readable descriptor such as `"BottomLeft"`. The effects of changing this parameter are fairly limited as Inventor 2019 mostly ignores the value.
- **units** This is one of the more important parameters used to set the units of the numerical dimensions of the print volume. Values are typically defined in millimetres `mm`.
- **depth** The numerical size of the print volume along the dimension running from the front edge of the build plate to the back.
- **width** The numerical size of the print volume running along the print bed from the left to right edge.
- **height** The numerical size of the print volume in the vertical direction perpendicular to the build plate.
- **thicknessThreshold** The function of this is uncertain. It is set to `1` for all printers.
- **defaultPrinter** This is a Boolean variable identifying the data of the printer to be used as default when the 3D print environment is opened. 
Only one entry should have the value of `1` at any time.
- **favorite** This is another Boolean flag that dictates if the printer is added to the quick access drop down menu (`1`) or only present in the `other printers` dialog (`0`).

### 1. Accessing the file
The first step is to locate the xml file describing the printers currently known to Autodesk Inventor.
This file is named `3DPrinterDescriptions.xml` and can typically be found in the following location, but will depend on the options selected upon Inventor's installation location and the software version.
```
C:\Users\<USERNAME>\AppData\Roaming\Autodesk\Inventor 2019\3DPrinterDescriptions.xml
```
Note that `<USERNAME>` should be replaced with the name of the user that installed Autodesk Inventor.

If the file is not at this location, a search of the `C:` drive for the xml filename should return its path.

While Autodesk Inventor is closed, open this file with a text editor such as [Notepad++](https://notepad-plus-plus.org/) or [Visual Studio Code](https://code.visualstudio.com/).

### 2. Generating a new printer entry
Next, we have to gather the data to populate the attributes of a new printer.

Start by copying the blank template entry below to the bottom of the list of printers but ensure this is above the closing `</Printers3D>` tag.

```xml
<Printer internalName="" modelName="" manufacturer="" technology="FDM" origin="0,0,0" units="mm" depth="" width="" height="" thicknessThreshold="1" defaultprinter="0" favorite="1" />
```
Working from left to right, create a unique identifier for your printer using a program such as [UUID Generator] (https://www.uuidgenerator.net/) and copying the identifier string into the region between the quotation marks after the `internalName` parameter.
You may wish to shift this to upper case to retain a consistent formatting.

Next add a name for you printer between the quote marks after the `modelName` key and its manufacturer after the `manufacturer` tag.
These are how you will identify your printer so it is important to use the correct values so it can be easily picked out later.
Pay particular attention to the `manufacturer` value as this dictates the group your printer will be added to when accessing it from within Inventor.

The default `technology` and `origin` values are suitable in the vast majority of cases so should not need to be adjusted.

Then, add the `depth`, `width` and `height` data.
This is typically found on a website listing your printer's technical specifications, but you can also use measured values to the nearest millimetre.

Follwong that, you can make this new printer your default selection by changing the `defaultprinter` tag to `1`.
Note that you can only have one printer be default at a time, so will have to examine the other printer entries to find the current default and change its `defaultprinter` tag to `0`.

Finally, if you want this new printer to appear in the quick access dropdown list, set `favorite` to `1`. 

### 3. Finishing up
The `3DPrinterDescriptions.xml` document should now resemble the snippet below, with our new printer added at the end, albeit without the ellipsis (i.e. the three dots, `...`) representing the other default printers. 
As an example, this snippet has added data for a Prusa i3 MK2 and a Prusa i3 MK3S 3D printer.
```xml
<Printers3D>
    <Printer internalName="BCF2EDCD-3DB2-48D6-83B9-8A5D9AB1C661" modelName="Default Machine Workspace" manufacturer="Autodesk Inc." technology="FDM" origin="0,0,0" units="mm" depth="450" width="420" height="400" thicknessThreshold="1" defaultprinter="0" favorite="1" />
    <Printer internalName="ED5911E2-2934-4822-B0CE-3C4F6B6E0EF6" modelName="Makerbot Replicator 2" manufacturer="Stratasys" technology="Extrusion" origin="BottomCenter" units="mm" depth="148" width="285" height="150" thicknessThreshold="1" defaultprinter="0" favorite="0" />
    ...
    <Printer internalName="207C34AA-3152-4184-8778-BB813B045149" modelName="i3 MK2" manufacturer="Prusa Research s.r.o." technology="FDM" origin="0,0,0" units="mm" depth="210" width="250" height="200" thicknessThreshold="1" defaultprinter="0" favorite="1" />
    <Printer internalName="D854798C-AF19-47E5-8806-B707AC333A9F" modelName="i3 MK3S" manufacturer="Prusa Research s.r.o." technology="FDM" origin="0,0,0" units="mm" depth="210" width="250" height="210" thicknessThreshold="1" defaultprinter="1" favorite="1" />
</Printers3D>
```

You can now save and close the xml document.

To test if our new printer has been added successfully, then open Autodesk inventor and load a part file before accessing the `3D printing` app from the `Environments` tab at the top of the screen.

If you set the new printer definition as default, it should be automatically loaded.

Otherwise, click on the drop-down list to the left of the `Print Options` button and select `Other Printers`. 
You can find your newly added printer by clicking the `+` box to expand the item corresponding to your printer's manufacturer.
Select it for use by ticking the `Use in current document` box and pressing the `OK` button.

Assuming everything has gone according to plan, the print volume visualisation will match the dimensions of your new printer and be labelled with the `modelName` value at the front of the print volume cuboid.

The process of adding a new printer is now complete. Congratulations!
