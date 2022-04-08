# The TLDR Guide of Fixing Shutdown/Restart

This guide is a simplified (if somewhat expanded) version of the section [Fixing Shutdown/Restart](https://dortania.github.io/OpenCore-Post-Install/usb/misc/shutdown.html#fixing-shutdown-restart) found in [Dortania's OpenCore Post-Install Guide](https://dortania.github.io/OpenCore-Post-Install/).  
[[images/Multiple_USB_controllers_DM_example.png]|Example]
I wrote this guide  because I could not understand the steps in the original guide as they where written. It took me a lot of time, frusrtation, and trial-and-error to finally figure it out. 
 
This is a combination of personal notes (in case I ever need to do this procedure again) and a guide for others who have the same difficulty I had in understanding the original guide.

## The problem:
When you select *shutdown* on your Hackintosh, the computer restarts instead.


## The solution:
[According to the original guide](https://dortania.github.io/OpenCore-Post-Install/usb/misc/shutdown.html#fixing-shutdown-restart) the problem is that the USB controller is not getting the correct command to shut down. So we need to create a patch to send the correct command to the USB controller.

This patch is implemented in two parts:  
1. an SSDT file named [FixShutdown-USB-SSDT.dsl](https://github.com/dortania/OpenCore-Post-Install/blob/master/extra-files/FixShutdown-USB-SSDT.dsl) that contains the correct shutdown command for our USB controller.  
2. [an ACPI patch](https://github.com/dortania/OpenCore-Post-Install/blob/master/extra-files/FixShutdown-Patch.plist) in the OpenCore config.plist file which will enable the use of the SSDT file.

You may also need to **disable Power Nap** in macOS System Preferences. (Actually, try this before adding this patch to Open Core. If you're lucky you may not need the patch).

### Editing the SSDT file
The SSDT file must be edited with the correct "name" for our USB controller. So the first thing we need to do is figure out what that name is. (This "name" is properly called an **ACPI path**.)  

**Here's how to find the ACPI path of your USB controller:**  

1. Boot your Hackintosh PC in Windows.  
2. Press the Windows Key and X (win+X) and from the menu select **Device Manager**.   
3. In the device manager, navigate to **Universal Serial Bus Controllers**.  
4. In that category, select the entry that has "controller" in its name. Right-click on it and select **Properties**.  
5. In the properties window, go to the **Location Paths** section and copy the data there (usually its the second line).

An example of what you'll probably see: `ACPI(_SB_)#ACPI(PCI0)#ACPI(XHC_)`  
	
The ACPI path is the text in parenthesis, so by leaving everything else out we get: `_SB_PCI0XHC_` and we add full stops to separate the three elements: `_SB_.PCI0.XHC_`   
This is the ACPI path we need for the SSDT file.  

**What to do if you have multiple USB controllers.**  
If in Device Manager -> Universal Serial Bus Controllers you see more than one entry with "controller" in its name, you will need to copy the ACPI paths for each one.

**Here's how to edit your SSDT file:** 

Note: to edit (and compile) the SSDT file [FixShutdown-USB-SSDT.dsl](https://github.com/dortania/OpenCore-Post-Install/blob/master/extra-files/FixShutdown-USB-SSDT.dsl) you will need to use the application [MaciASL](https://github.com/acidanthera/MaciASL/releases/) 

1. Open [FixShutdown-USB-SSDT.dsl](https://github.com/dortania/OpenCore-Post-Install/blob/master/extra-files/FixShutdown-USB-SSDT.dsl) in [MaciASL](https://github.com/acidanthera/MaciASL) 
2. Look for `External (_SB_.PCI0.XHC_.PMEE` in the code and replace it with your USB controller's ACPI path (in this example with `External (_SB_.PCI0.XHC_`).
3. Look for `\_SB.PCI0.XHC.PMEE = Zero` and replace it your USB controller's ACPI path (in this example with `\_SB.PCI0.XHC = Zero`)

Note: Youl'll notice that the ACPI path in step 3 has some underscores (_) missing. I don't know the [ACPI programming language](https://acpica.org/sites/acpica/files/acpica-reference_19.pdf) so I'm not certain why the second instance of ACPI path differs from the first, but by mimicking the formatting of the original ACPI path the patch worked for me. If it does not work for you, try it with and without the underscores.

**What to do if you have multiple USB controllers.**  
You will need to copy the entire code from and duplicate it inside the file, replacing the ACPI path for each USB controller you have.


After you finish editing the file, you need to select **Save As...** and save the file with the format **ACPI Machine Language Binary**. (It has the name extension .aml instead of .dsl).

Then you are ready to move the newly edited .aml file into your **ACPI folder**. I'm going to assume you [know where that folder is located](https://dortania.github.io/Getting-Started-With-ACPI/ssdt-methods/ssdt-easy.html#adding-to-opencore).

Afterwards, you'll need to add to the FixShutdown-USB-SSDT.aml entry into the ACPI -> Add section of the config.plist file so that Open Core knows its there.

### Editing the OpenCore config.plist file

Fortunately, this is much simpler than the SSDT file. You don't need to adjust the ACPI patch. Just apply it as is.

Open your config.plist file in a [plist editor](https://github.com/corpnewt/ProperTree) and copy the contents of [the FixShutdown-Patch.plist file](https://github.com/dortania/OpenCore-Post-Install/blob/master/extra-files/FixShutdown-Patch.plist) into it.

While you have the OpenCore config.plist file open, don't forget to add the FixShutdown-USB-SSDT.aml in the ACPI -> Add section.

When finished, the relevant sections of your config.plist file should look something like this:


### macOS power settings

Finally, for the patch to work you may need to change some power settings in System Preferences. The one that seems to affect the system shutdown is **Power Nap**. If your computer still restarts at shutdown, try disabling Power Nap in both the **Battery** and the **Power Adapter** sections.

Note: you can also disable Power Nap from the terminal with the command `sudo pmset powernap 0` 

## Further help
If you did all of the above but still haven't managed to solve the issue, I'm afraid I can't be of much help. I have no real technical knowledge of ACPI. I suggest you ask for help from the [Hackintosh SubReddit](https://www.reddit.com/r/hackintosh/) community.
