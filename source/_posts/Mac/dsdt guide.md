---
title: DSDT GUIDE
date: 2017-05-05 13:09:40
tags:
  -MacOS
categories:
  -MacOS
---

Overview

In order to make many OS X features work well on a laptop, you will always need a properly patched DSDT (and SSDTs). The purpose of this guide is to provide a foundation for proper patching of your OEM DSDT/SSDTs.

Advanced users may wish to implement hotpatching via Clover. See guide here: http://www.tonymacx86.com/threads/guide-using-clover-to-hotpatch-acpi.200137/

Although you may be tempted to use a DSDT from another computer, it will almost always end in failure. You simply cannot be certain it is valid to use ACPI files from another computer. Even minor differences in hardware configuration (BIOS version, amount of memory installed, BIOS options selected, and other hardware differences such as which WiFi card is installed) can make for differences that cause instability and weird bugs if you use foreign ACPI files. Differences such as BIOS version, amount of memory installed, BIOS options selected, and other hardware differences such as which WiFi card is installed, can make various OperationRegion addresses different, which makes a patched DSDT for one system incompatible with another. It is also not uncommon for the same laptop model to be produced in different runs with different motherboards, and potentially incompatible ACPI files.
<!--more-->
The process of patching involves several steps:
- extracting native files
- disassembling the native files
- analyzing and filtering the native files
- patching
- saving (compiling) and installing


Extracting native ACPI files

All BIOS implementations provide ACPI files to the OS. So, on any OS, you can extract them for patching later. Extraction can therefore be done on Linux, OS X, Windows, or even in the Clover bootloader. Native files extracted are generally identical, although because of the software used to extract, they may be named differently.

This guide will focus on three methods of extraction: Using patchmatic in OS X, using F4 with Clover, or using Linux.

Extracting with 'patchmatic':

If you've already installed OS X, provided you're not currently booting with any patched ACPI files, you can extract your native DSDT/SSDT with patchmatic. Download the patchmatic binary here: https://github.com/RehabMan/OS-X-MaciASL-patchmatic (be sure to read the README as the download location is linked from it). For ease of use in Terminal, you should copy the binary (inside the ZIP) to /usr/bin.

After installing patchmatic, you can invoke it in Terminal, as such:

Code (Text):

cd ~/Desktop
mkdir extract
cd extract
patchmatic -extract

The patchmatic binary will extract all loaded ACPI files and place them in the current directory. If you're using any options in the bootloader that affect the injected DSDT/SSDT, you will not get native ACPI files, so be sure you're not using those options. For example, if you're using (Chameleon) DropSSDT=Yes, or (Clover) DropOem=true, the native SSDTs are being dropped before OS X can load them, so you'll be missing them in the patchmatic output. Same goes for any Clover DSDT "Fixes" -- those fixes are patching the native DSDT, and to do your own DSDT patching, you don't want that. Options such as GeneratePStates/GenerateCStates=Yes, or with Clover /ACPI/SSDT/Generate/CStates /ACPI/SSDT/Generate/PStates will inject extra SSDTs which can cause complications with disassembly.

It is for all those reasons, it is often easier to extract via Linux or using Clover F4.

Note: Using 'patchmatic -extract' to confirm you're patching DSDT/SSDTs as you expect is a useful diagnostic tool.

Extracting with Clover F4 (recommended):

Extracting with Clover F4 is recommended, due to ease of extraction, and due to ease of comparison between ACPI/origin and ACPI/patched (for troubleshooting).

At the main Clover bootloader screen, you can press F4 and Clover will dump the native ACPI files to EFI/Clover/ACPI/origin. You can then access them after you boot OS X to disassemble them and patch. Note that some BIOS implementations reverse the function of Fn+F4 with F4, so when in doubt, press both Fn+F4 and F4. There is no feedback during or after the dump, just a slight delay as the files are written. The delay is more noticeable if they are being written to USB, as would be the case when booting from a Clover USB.

Sometimes, Clover F4 will write duplicate SSDTs. These duplicates will cause problems during disassembly. If you run into issues (duplicate definitions) during disassembly, you will need to analyse all SSDTs to eliminate the files which are duplicate. It is easy to see which are duplicates by looking at the file sizes. Files with equal size are likely duplicates.

You can see file sizes in bytes of all SSDTs in Terminal:
Code (Text):

ls -l SSDT*.aml

Extracting with Linux:

In Linux, the native ACPI files are available directly from the file system. You can find them at /sys/firmware/acpi/tables and /sys/firmware/acpi/tables/dynamic. It is possible to copy the entire set with a single command in Terminal.

It is not necessary to install Linux. Simply run it from USB: http://www.ubuntu.com/download/desktop/create-a-usb-stick-on-windows.

In Linux Terminal:
Code (Text):

# substitute DEST with the mountpoint of a FAT32 formatted USB stick
sudo cp -R /sys/firmware/acpi/tables DEST

You should copy the files to a FAT32 formatted USB. Using FAT32 avoids permissions issues as FAT32 has no file permissions. The value of DEST for an auto-mounted USB will depend on the version of Linux you're using and how you booted it. You can see the mount-point by typing 'mount' in Terminal, or hover your mouse over the volume name in the Linux file explorer.


Preparing tools for disassembly

To properly disassemble your extracted files, you need the iasl compiler, which is run from Terminal.

You will need a recent build of iasl to disassemble them properly. There is an appropriate version available here: https://bitbucket.org/RehabMan/acpica/downloads/. It is a good idea to copy the iasl binary to your path (eg. /usr/bin), so it is easily accessed from Terminal.

For example, if you downloaded it to ~/Downloads/iasl.zip, you can extract and copy it in Terminal:
Code (Text):

cd ~/Downloads
unzip iasl.zip
sudo cp iasl /usr/bin


Building the latest iasl from github

You can also build the latest version of my iasl from my github. Assuming you have Xcode installed:
Code (Text):

mkdir ~/Projects && cd ~/Projects
git clone https://github.com/RehabMan/Intel-iasl.git iasl.git
cd iasl.git

The latest version always tends to have experimental and not well tested code. For example, the new code Intel added to disassemble If/Else blocks as Switch/Case is full of bugs (I have not had time to report them to Intel).

I recommend using the version prior to that change:
Code (Text):

git checkout b9c6c2b

Note: The b9c6c2b version is the one currently available from bitbucket.

Then build it:
Code (Text):

make

At that point, you can install it with:
Code (Text):

sudo make install

And assuming you have MaciASL.app installed to /Applications, you can use the new version (that you just built and installed to /usr/bin) in MaciASL as well:
Code (Text):

sudo cp /usr/bin/iasl /Applications/MaciASL.app/Contents/MacOS/iasl61

The newer version of iasl will eventually be available at the bitbucket link, but for those who want to be on the "bleeding edge"...


Disassembling ACPI files

Although the extracted native files can be opened directly in MaciASL, it is not recommended. Opening an AML file directly in MaciASL will cause MaciASL to disassemble the file (with iasl) standalone, and if the AML has complex references to other AMLs, it will not disassemble it correctly. You'll be left with many hard to fix errors.

As a result, it is better to disassemble all files as a group using iasl in Terminal. To prepare, place all DSDT and SSDT files in a single directory (DO NOT copy ACPI files that don't begin with DSDT or SSDT), and change the names such that they have an .aml extension.

Then disassemble in OS X Terminal:
Code (Text):

cd "to directory where you placed all SSDT/DSDT"
iasl -da -dl DSDT.aml SSDT*.aml

Note: Do NOT attempt to disassemble other ACPI files with the -da option. It will not work.

Note: Also read the section below regarding refs.txt. Using refs.txt takes little effort, but can eliminate many common errors.

From this point onward, you will work exclusively with the resulting *.dsl files using MaciASL. Of course, to use them you must save as "ACPI Machine Language Binary" with an extension .aml and place them where they will be loaded by the bootloader. But keep your patched .dsl files in case you need to apply more patches in the future.

Let me state it quite simply (because this comes up a lot): If you are opening an AML file directly in MaciASL and clicking Compile, you are doing it WRONG. Let that soak into the gray matter between your ears for a minute.

Note: The new tools with ACPI 6.1 are much more robust when dealing with AML files that have been compiled with the new version of iasl. ACPI 6.1 adds a feature to the compiler where opcodes for External references are added to the AML binary. ACPI interpreters ignore this data, but the data is useful to the disassembler (also only ACPI 6.1 version of iasl) to create a better disassembly from a standalone AML. As a result, you might find that AML files that have been recompiled with the latest tools may open directly more reliably. Of course, existing OEM ACPI DSDT and SSDTs are not using the new tools at this point, so you still must disassemble initially with all DSDT/SSDT with option -da, as described in this guide.

Note regarding Snow Leopard ACPI implementation: Unfortunately, the 10.6.8 ACPI is old enough that it chokes on AMLs with the external opcode. If you are planning to use your ACPI files with Snow Leopard use the undocumented "-oe" switch to iasl when you compile your AML files. This option is not set when you compile (Save As) from MaciASL, so you will need to compile your files in Terminal. The "-oe" option disables generation of the external opcode in the output AML files.


Disassembly with refs.txt

Sometimes there are additional unresolved externals (symbols not defined in any file). The iasl disassembler will attempt to guess the number of arguments, but often it guesses poorly. You can correct it, by providing the External declarations in a text file. Some common unresolved symbols are SGPO, ECRD, ECWT, and MMTB.

The following refs.txt content has some common (and not so common) missing symbols (as reported by users in this thread) that the disassembler tends to be confused by.

First create refs.txt in the directory where your DSDT/SSDT files are:
Code (Text):

External(MDBG, MethodObj, 1)
External(_GPE.MMTB, MethodObj, 0)
External(_SB.PCI0.LPCB.H_EC.ECWT, MethodObj, 2)
External(_SB.PCI0.LPCB.H_EC.ECRD, MethodObj, 1)
External(_SB.PCI0.LPCB.H_EC.ECMD, MethodObj, 1)
External(_SB.PCI0.PEG0.PEGP.SGPO, MethodObj, 2)
External(_SB.PCI0.GFX0.DD02._BCM, MethodObj, 1)
External(_SB.PCI0.SAT0.SDSM, MethodObj, 4)
External(_GPE.VHOV, MethodObj, 3)
External(_SB.PCI0.XHC.RHUB.TPLD, MethodObj, 2)

Note: A handy way to create refs.txt is to use pbpaste in Terminal. Copy the text above to the clipboard (I'm assuming you know how to do that), then:
Code (Text):

pbpaste>refs.txt

That will create refs.txt in your current working directory.

Then use it during disassembly:
Code (Text):

iasl -da -dl -fe refs.txt DSDT.aml SSDT*.aml

Older versions of the iasl disassembler will place these External declarations before all the other External declarations. This, also, is a poor choice. Most of the time, you'll need to move them so they follow the other External declarations instead of preceding them. It will be obvious as you'll get errors from the External declarations that follow those inserted from refs.txt. In the current iasl supporting ACPI 6.1, this bug has been fixed.


Filtering ACPI files

For older computers (Sandy Bridge and prior), the SSDTs related to CPU can cause problems. If this is the case (you already had to use the alternate DropTables, or DropOem=true, or DropSSDT=Yes), then you should not include such SSDTs in ACPI/patched.

I like to include all SSDTs, in their original order, patched appropriately, unless they are known to cause a problem. Keep in mind that an SSDT that doesn't need any patches, does not need to be recompiled. You can simply use the original AML file unmodified.

Note: The 'x' SSDTs from a Clover dump and likewise the SSDTs in the dynamic subdirectory from a Linux dump are loaded dynamically and are never included in ACPI/patched (they are SSDTs loaded on-demand with ACPI Load from SystemMemory).

After you successfully disassemble your files, look at each one in an attempt to determine the SSDT's purpose. If it is CPU related and is known to cause a problem, set it aside, and don't include it in your final set for injection via the bootloader. For the most part, SSDTs with objects declared in Scope _PR.CPUx are likely CPU related.

Here are some typical SSDTs you'll find:

- CPU related: already discussed above. Include unless known to cause a problem.
- SATA: Can be excluded or included... your choice.
- PTID: Usually this file is useless for OS X and contains many errors. In rare cases, it may provide clues for how to read fan speed, temperature, or other system status.
- IAOE: If this SSDT is present, it is usually accessed from DSDT in _PTS and _WAK. Sleep may not work properly without it.
- GFX0: Usually the SSDT with 'Device GFX0' will be for integrated graphics. This is the SSDT you patch for backlight control. With older laptops, GFX0 is usually defined in DSDT. With newer Haswell laptops, it is usually defined in SSDT (although it can also be in DSDT).
- PEGP: PEGP usually corresponds to the discrete card in a switched dual-GPU configuration. There can be more than one, and usually you will need to include all of them as a group in order to accomplish any meaningful patching. These SSDTs should be patched if you wish to disable the discrete card when running OS X.

It is a good idea to keep notes on what the purpose of each SSDT is and which ones you plan to "drop", which ones you plan to keep unmodified, and those that you plan to patch.


Fixing Errors

Even by disassembling all at once (iasl with -da option), the native files can still have errors. The disassembled files have errors due to changes in iasl over time, imperfections in iasl itself, and differences in the compilation environment between our laptops and the OEM. A common cause of errors (my theory), for example, is that some of the methods referenced are actually inside Windows (MMTB and MDBG for example). There is also clearly cases where bugs are in the code or code was written uninitentionally (sometimes hard to tell the difference).

So.. after determining which files you need, you must patch them so they compile without errors. There are many common patches for such errors in my laptop patch repository for MaciASL.

MaciASL: https://github.com/RehabMan/OS-X-MaciASL-patchmatic
Laptop Patches: https://github.com/RehabMan/Laptop-DSDT-Patch

Note: I do not test my patches with DSDT Editor. It has too many bugs and a very old version of iasl. Please do not ask me about it.

Be certain to always read the README, in order to download from the correct location and in order to setup MaciASL correctly. The patches for syntax/error problems begin with "[syn]" in the name. Common examples for older DSDTs are "Fix _PLD Buffer/Package Error", "Fix TNOT Error", and "Fix FPED Parse Error". In order to determine which patch you need, you can look at the error message coming from the iasl compiler and the code at the line the error was detected. You can also attempt to apply a patch just to see if it makes changes as shown in the Preview window in MaciASL. If you're not familiar with each type of error, it can take some experimentation and trial/error.

For some errors, you can simply remove the line of code causing the error. But, it depends on whether the line is necessary for proper operation of the code or not. For example, errors caused by 'External' declarations can generally be removed to fix the error. If you wish, you can create automated patches of your own to remove these lines.

It helps to have some experience with the ACPI spec and some programming experience.

Your goal is to get each .dsl file to compile without errors (warnings/remarks/optimizations are ok). Once you have files that compile without errors, you can move on to patching them to fix issues you may have with your OS X installation.


Common Patches

Generally, a DSDT patch should only be applied after finding a need for that specific fix. But there are several patches that are commonly needed and that have only a small chance of causing a problem. They are in my laptop repo and are listed here:
"Fix _WAK Arg0 v2"
"HPET Fix"
"SMBUS Fix"
"IRQ Fix"
"RTC Fix"
"OS Check Fix"
"Fix Mutex with non-zero SyncLevel"
"Fix PNOT/PPNT" (use only if you're dropping CPU related SSDTs)
"Add IMEI" (do not use if your DSDT or SSDTs already have IMEI/HECI/MEI device)

Note: The OS Check Fix patch you use has nothing to do with the version of Windows the laptop came with, nor with the version of Windows you're currently using.

Note: Do not use the "Fix PNOT/PPNT" patch if you're including all OEM SSDTs. It is intended only for the case you omit the OEM CPU related SSDTs.

The USB patches can be used to fix "instant wake" where the laptop will not sleep without waking up seconds after sleep begins.

Apply the patch appropriate for your hardware:
"6-series USB"
"7-series/8-series USB"

The USB3 Mutliplex patch can assist in using AppleUSBXHCI.kext instead of GenericUSBXCHI.kext. It is based on information published by Mieze. Most DSDTs will need modifications to use it. The ProBook, for example, uses a modified version of this patch. And the Lenovo u310/u410 can use it as-is:
"7-series USB3 Multiplex"

If you're using GenericUSBXHCI.kext on Yosemite, make sure you're using one built for Yosemite. Also, to avoid instant wake you may need kernel flag -gux_defer_usb2.

An alternate solution for "instant wake" using AppleUSBXHCI.kext is to use "USB _PRW 0x6D (instant wake)". You should examine your DSDT to determine what the relevant _PRW methods return to be certain the patch is appropriate for your DSDT. Also provided in the repo is "USB _PRW 0x0D (instant wake)" (0x0D and 0x6D are both common values for XHC/EHC/HDEF return from _PRW).

If you have a Haswell CPU/8-series chipset, and AppleLPC.kext is not loading, you should use this patch to inject a compatible ID that will allow it to load:
"Haswell LPC"

If you have a Skylake CPU/100-series chipset, and AppleLPC.kext is not loading, you should use this patch to inject a compatible ID that will allow it to load:
"Skylake LPC"

Note regarding renames: Renames must be "balanced." It is common to rename objects to better match what OS X expects (example "Rename GFX0 to IGPU" for proper IGPU power management). In such cases, all DSDT/SSDTs with references to that name must also be renamed.

Note regarding duplicate identifiers: You must be sure that your patched files do not contain duplicate identifiers. A common case would be adding a _DSM method to a given path in one SSDT, where the OEM has defined a _DSM at the same path in another SSDT. To avoid this problem, you can use the "Remove _DSM methods" patch as one of the first patches you do to all DSDT/SSDTs. Also, "Rename _DSM methods to XDSM" is an alternative (sometimes "Remove _DSM methods" exposes a bug in MaciASL).


Problem specific patching

Battery status: http://www.tonymacx86.com/yosemite-...de-how-patch-dsdt-working-battery-status.html

Backlight control: http://www.tonymacx86.com/yosemite-...ching-dsdt-ssdt-laptop-backlight-control.html

Disabling nVidia/Radeon discrete graphics: http://www.tonymacx86.com/yosemite-...bling-discrete-graphics-dual-gpu-laptops.html

When following a guide for a specific laptop, it may instruct you to apply a patch that is provided in the post itself. You will recognize it as the syntax used will look similar to other patches you've seen in the repository (eg. 'into_all method label FOO code_regex xxyy removeall_matched;'). These patches are intended to be pasted directly into the patch window in MaciASL so they can be applied.

If you're interested in writing your own patches, read the documentation on MaciASL patch grammar: http://sourceforge.net/p/maciasl/wiki/Patching Syntax Grammar/

Note: In many cases, DSDT patches are used in conjunction with additional kexts, patched kexts, or Clover config.plist patches that patch the system kexts as they are loaded.


Patches for using patched AppleHDA

With patched AppleHDA, there are two patches that are needed in conjunction with the kext:
"Audio Layout 12" (change the layout-id from 12 to the one used by your DSDT)
"IRQ Fix"

Note that you must have an AppleHDA that matches your codec, and must determine which layout-id was chosen. The layout-id is an arbitrary choice by the creator of the patched AppleHDA.

To determine the layout-id used by a particular patched AppleHDA: First you need to know your codec id in decimal (eg. 0x10ec0269 = 283902569). Then look in the Info.plist for AppleHDAHardwareConfigDriver.kext (at AppleHDA.kext/Contents/PlugIns/AppleHDAHardwareConfigDriver.kext/Contents/Info.plist), find your codec id under HDAConfigDefault (there may be many entries in a sloppy patched AppleHDA or only one). The LayoutID that matches your codec id is the layout id you need. It is possible that a patched AppleHDA contains more than one layout-id for a given codec. In that case, choose the one you want to use.


Saving files for loading by the bootloader

In order to use your patched DSDT/SSDTs, you must save them where the bootloader can load them. Each bootloader location is unique and has different requirements for naming. Files must be saved in "ACPI Machine Language Binary" (MaciASL->Save As). Saving a text file (dsl) with an AML extension will likely cause panic or very strange behavior in OS X.

Clover: Files should be placed on the Clover bootloader partition (usually the EFI partition), in EFI/Clover/ACPI/patched. DSDT.aml, if present, will automatically replace the OEM DSDT. For versions of Clover prior to v3062, SSDTs must be named SSDT-x or SSDT-xx, where 'x' is a digit (up to SSDT-19). Clover allows gaps in the numbering (eg. SSDT-1.aml, SSDT-5.aml, SSDT-6.aml is allowed). Clover version v3062+ will load all *.aml files present in ACPI/patched, with no restrictions on the name. Keep in mind that SSDT loading order is important. The original order of the SSDTs must be maintained in the patched set.

Note regarding Clover v3062+: A change in the way SSDTs are loaded from ACPI/patched makes the order non-deterministic. You should specify the order explicitly with config.plist/ACPI/SortedOrder. The SortedOrder option is implemented in Clover v3088+. The config.plist files linked from my Clover guide have a good default for SortedOrder: http://www.tonymacx86.com/yosemite-...de-booting-os-x-installer-laptops-clover.html

Chameleon(or Chimera): Files should be placed in /Extra on the system volume (or wherever your bootloader /Extra configuration is located). DSDT.aml, if present, will automatically replace the OEM DSDT. Chameleon requires there are no gaps in the names of SSDTs. It will load SSDT.aml, SSDT-1.aml, SSDT-2.aml, SSDT-3.aml, and so-on from /Extra until it does not find the file in the sequence. So, if you have SSDT.aml, SSDT-1.aml, SSDT-4.aml, SSDT-5.aml, only SSDT.aml and SSDT-1.aml will be loaded. SSDT-4.aml and SSDT-5.aml will be ignored.

Finally, keep in mind you cannot provide patched SSDTs without dropping the OEM SSDTs that they replace. The easiest way is to use DropSSDT=Yes (Chameleon) or ACPI/SSDT/DropOem=true (Clover) to drop all the SSDTs, then provide the patched (and unpatched) files for loading by the bootloader.


Floating regions

In ACPI, an OperationRegion can define a MMIO region, SystemMemory region, EmbeddedControl region, etc. These regions usually have fixed addresses dependent only on the machine configuration, BIOS version, or BIOS options. Sometimes, these regions can change randomly or unexpectedly. This is referred to as "floating regions".

Since by patching DSDT and/or SSDTs, we are providing a snapshot of these addresses at a given point in time, they may not match up should the BIOS decide to place such regions at a different address. If this is the case, you may notice that certain features are intermittently working, or other stability issues that appear to be random.

As a result, it is a good idea to use Clover and its FixRegions feature. You can find the details in the Clover Wiki. All of the config.plist files in the Clover laptop guide use this feature. Note: Only floating regions in DSDT can be fixed by FixRegions. Floating regions in SSDTs are problematic and there is no good solution other than to not provide patched SSDTs for SSDTs subject to randomly floating regions. Working around floating regions in patched SSDTs is beyond the scope of this guide.


Resources

MaciASL (RehabMan fork): https://github.com/RehabMan/OS-X-MaciASL-patchmatic
patchmatic: https://github.com/RehabMan/OS-X-MaciASL-patchmatic
iasl (RehabMan fork): https://bitbucket.org/RehabMan/acpica/downloads
ACPI spec: http://acpi.info/spec.htm

RehabMan github: https://github.com/RehabMan?tab=repositories

Clover laptop guide: http://www.tonymacx86.com/yosemite-...oting-os-x-installer-laptops-clover-uefi.html
Clover config.plist files for laptops: https://github.com/RehabMan/OS-X-Clover-Laptop-Config

Clover thread: http://www.insanelymac.com/forum/topic/284656-clover-general-discussion/
Clover changes: http://www.insanelymac.com/forum/topic/304530-clover-change-explanations/


Providing Feedback

Do not treat this thread as your private troubleshooting thread. If you have a specific problem with your specific laptop, open a separate thread. If you see something here that is in error, or wish to make a contribution, please reply to this thread.

Problem Reporting

Please read above "Providing Feedback". It is best to open a separate thread.

In that separate thread, describe you problem clearly. And provide relevant data...

Download patchmatic: https://bitbucket.org/RehabMan/os-x-maciasl-patchmatic/downloads/RehabMan-patchmatic-2015-0107.zip
Extract the 'patchmatic' binary from the ZIP. Copy it to /usr/bin, such that you have the binary at /usr/bin/patchmatic.

In terminal,
Code (Text):

if [ -d ~/Downloads/RehabMan ]; then rm -R ~/Downloads/RehabMan; fi
mkdir ~/Downloads/RehabMan
cd ~/Downloads/RehabMan
patchmatic -extract

Note: It is easier if you use copy/paste instead of typing the commands manually.

Attach contents of Downloads/RehabMan directory as ZIP.

Attach ioreg as ZIP: http://www.tonymacx86.com/audio/58368-guide-how-make-copy-ioreg.html. Please, use the IORegistryExplorer v2.1 attached to the post! DO NOT reply with an ioreg from any other version of IORegistryExplorer.app.

Provide output (in Terminal):
Code (Text):

kextstat|grep -y acpiplat
kextstat|grep -y appleintelcpu
kextstat|grep -y applelpc
kextstat|grep -y applehda

Attach EFI/Clover folder as ZIP (press F4 at main Clover screen before collecting). Please eliminate 'themes' directory. Provide only EFI/Clover, not the entire EFI folder.

Attach output of (in Terminal):
Code (Text):

sudo touch /System/Library/Extensions && sudo kextcache -u /

Compress all files as ZIP. Do not use external links. Attach all files using site attachments only.

If you cannot boot, attach photo of panic/hang and the EFI/Clover that causes it (as ZIP, without themes).
