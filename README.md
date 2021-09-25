# MATLAB with Arch Build System

[中文](README_cn.md)

This PKGBUILD creates an Arch Linux package for MATLAB.

Additionally, it also builds the Python engine. They are stored in `matlab` and `python-matlabengine` packages separately.

***WARNING*** this package NEEDS you to obtain the original off-line installer, as unauthorized distributions are not permitted in user agreement.

## Tips

* You probably want to run this locally, since the package will be large. Turn off compression if this package will only be installed on the build machine.
  This cuts a fair amount of time from the build and compression is unnecessary on local machines. 
* You may also use `tar` instead of packing it into `zst` and modify the PKGBUILD when preparing the off-line installer.
* If you are managing your own repos, depending on bandwidth and size constants,
  exclude this package from your work-flow.
  (Partial install is over 8 GB, and full install about 12 GB)
* Set `MATLAB_INTEL_OVERRIDE='yes'` in your environment if hardware acceleration does not work on your Intel GPU.

## Requirements

Besides the dependencies, the source files MUST be present in the directory.
The user MUST supply:

* `matlab.fik`: installation key in plain text file 
* `matlab.lic`: License file
* `matlab.tar.zst`: off-line installer

Then run `makepkg -s` from the PKGBUILD directory to build the package. The making is a long procedure.

### File Installation Key & License File

Here is the current link to the [instructions](https://www.mathworks.com/help/install/ug/install-using-a-file-installation-key.html) by mathworks.

The file installation key identifies this specific installation of MATLAB.
The license file authorizes that this key can use the toolboxes.
Follow these steps:

* Go to [License center](https://www.mathworks.com/licensecenter) on MathWorks.
* On install and activate tab; select (or create) an appropriate license.
    * If no license is listed because you're using your organization's license and this is your first time installing MATLAB, click the link for manual activation. Insert the MAC address of a network card on your PC in the Host ID field, as for the username, it'll have to be the one of the account (on your Arch installation) that will be using MATLAB.
* Navigate to download the license file and the file installation key.
* Download the **license file**, rename it to `matlab.lic`, and put it in the source directory.
* Copy and paste the **file installation key** in a plain text file named `matlab.fik`, and put it in the source directory.

If MATLAB is distributed through your organization’s network rather than your own use, the file installation key and license file would be invalid on other computers. When matlab recognizes invalid license, it would not start at all.

It may help if you remove your license for building, i. e. uncomment the lines on the end of `build()` of PKGBUILD. When the user starts matlab for the first time, the activation program will pop out guiding you through the activation process.

It still can be activated by running `activate_matlab.sh` after the install, no matter the license is removed or not. The script will activate it and put the correct license file under `.matlab/licenses` under your home folder.

### Off-line Installer

* [Download the matlab installer](https://www.mathworks.com/downloads)
* Unpack it with its original permissions retained (for example `unzip -X -K`) and launch the installer
* After logging in and accepting the license, select
`Advanced Options > I want to download without installing`
from the top drop down menu.
* Set the download location to an empty directory called `matlab`
* Select the toolboxes you want in the `PKGBUILD`.
(Alternatively install them all)
* After downloading, from the parent directory, do
`tar -I zstd -cvf matlab.tar.zst matlab`
to create the tarball.

You may also acquire the off-line installer from your organization, but make sure the file contains correct permissions, as normal a zip could drop them, otherwise the installer will not run correctly.

### Transmitting Large files

To transmit large files in FAT32 media (most flash drives), use split and cat:
* **Split**: `split --bytes 3G --numeric-suffixes=0 matlab.tar.zst matlab.tar.zst.`
* **Concatenate**: `cat matlab.tar.zst.* > matlab.tar.zst`

### Modules

Matlab comes with a [lot of products](https://www.mathworks.com/products.html).
Most are not needed nor provided in your license, check PKGBUILD to pick and choose appropriately.



