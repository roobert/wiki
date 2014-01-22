# Changing The Eee PC BIOS Logo

## Overview

The BIOS logo refers to the image displayed when the EeePC is turned on. It is possible to change this image using special tools provided by AMI.

## Warning

Flashing a machines BIOS can potentially render the machine permanently inoperable. Use these instructions at your own risk.

## Quick Guide

### Creating a BIOS Logo

 1. Create an 800x480 image in your favorite image editor

 2. Resize the image to 640x480.

 '''Note''': Resizing the image will stretch it, this is OK - in order for the image to fit into the BIOS it must be 640x480, the image dimensions will be 800x480 upon reboot.

 3. Save the image as a `BMP`, making sure to select '4 bit' colour depth.

### Creating a New BIOS ROM

 1. Aquire `OEMLOGO.EXE` which is part of the `tool_8_RC1` pack of AMIBIOS(R)8 OEM/ODM BIOS tools.

 2. Aquire either the latest or your prefered bios version from http://update.eeepc.asus.com/bios/

 3. Run OEMLOGO.EXE and browse for your BIOS .ROM file.

 4. Browse for image

 5. Run 'Swap Image'

 6. Save the new file as 701.ROM

### Flashing the BIOS

 1. Format USB Mass Storage device with FAT16 filesystem

 2. Copy BIOS ROM file to USB Mass Storage device

 3. Rename BIOS ROM file to `701.ROM` (Case Sensitive)

 4. Attach USB Mass Storage device to EeePC

 5. Reboot/Switch on EeePC

 6. Press `ALT+F2` when BIOS appears

 [Todo] Include notes on ctrl+fn+home to rescue corrupt bios, reference the eeeuser wiki noting its incorrectness. reference pdf file including relevant information

## Relevant and Useful Information

The following information may prove useful for users trying to modify their BIOS ROM, taken from `AMIBIOS ROM Utilities User Guide.pdf`.

'''OEMLOGO''' is a changing logo tool with graphical user interface. It allows you to replace the OEM Logo(Large) and OEM Logo(Small) module inside the BIOS ROM file with a new one.

'''AMIOLDOS''' is a changing logo tool with command line interface. It allows you to replace the OEM Logo(Large) and OEM Logo(Small) module inside the BIOS ROM file with a new one.


'''BIOS Requirements'''
[[br]]
The loaded BIOS ROM file should have the followings:
 * file MUST be an AMIBIOS ROM file (Core version 8.xx.xx only)
 * ROM file should be building via “8.00.08_AMITOOLS_17” label or above.
 * OEM Logo module (Module ID 0x0E) to be present
 * OEM Logo module (Module ID 0x1A) to be present
 * Boot function should be inside. It is recommended to use !DisplayLogo2 eModule “8.00.08_DISPLAYLOGO_05” label or later.


'''New Logo File Requirements'''
[[br]]
The Change OEM Logo Utility requires that the new Logo file fit the following format:
 * 16-Color Bitmap format, even width, 640*480 pixels (Maximum)
 * 256-Color Bitmap format, even width, 640*480 pixels (Maximum)
 * 256-Color PCX format, even width, 640*480 pixels (Maximum)
 * True-Color JPG format, even width, 640*480/800*600/1024*768 pixels (Maximum)
'''Note''': Small OEM Logo does support only 640*80, 16-Color Bitmap format.

The following information may prove useful for windows users wishing to update their BIOS, taken from the `AMIBIOS ROM Utilities User Guide.pdf`.

'''AFUDOS''' is an updating system BIOS utility with command line interface. It has no tedious and annoying parameters, just update your system BIOS. Hey!! Do not forget that target board MUST be AMIBIOS system.

'''AFUWIN''' is an updating system BIOS utility with command line and GUI interface. It has same parameters and behavior as AFUDOS, and further, GUI feature starting from v4.10 can provide you a friendly environment to visualize BIOS update procedure. By the way, do not forget that target board MUST be AMIBIOS system while using this utility.

'''NOTE''': '''AFULNX''' and '''AFUBSD''' have the same CLI options as '''AFUWIN''' and '''AFUDOS''' and are for use on Linux and BSD systems respectively.

'''Part 2. Graphical User Interface Mode, page 48''' and '''Chapter 1... OEMLOGO v3.xx page 49''' of the `AMIBIOS ROM Utilities User Guide-1.pdf` from the `tools_8_RC1` AMI tools collection has full documentation with step-by-step image-based tutorials on how to use `OEMLOGO.exe` and other tools referenced in this document.

# References

'''AMIBIOS8 Utility Overview''' 
 * http://www.ami.com/support/downloaddoc.cfm?DLFile=support/doc/AMIBIOS8_Utilities_2pg_PUB_08.01.23.pdf&FileID=981

'''AMI BIOS Tools'''
 * http://www.rebios.net/biosfile/tool_8_RC1.rar

