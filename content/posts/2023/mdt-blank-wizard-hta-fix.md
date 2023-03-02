---
title: "How to fix a blank Wizard.hta error in Microsoft Deployment Toolkit"
date: 2023-03-01T20:03:51-06:00
draft: false
description: "A guide on how to resolve an issue with the Microsoft Deployment Toolkit where the deployment fails with a blank Wizard.hta screen"
tags:
  - windows
  - microsoft deployment toolkit
---

At work we use Microsoft Deployment Toolkit to automate the process of imaging
laptops and joining them to our Active Directory domain. This has always worked
quite smoothly with every single model of computer we have purchased, with the
exception of the Dell Latitude 7430 laptops we recently acquired. After
injecting drivers during the imaging process, the install would suddenly fail
after a reboot. It provided absolutely no error message on screen either; it
simply presented me with a blank Wizard.hta window.

After doing a little research on this issue, I found that Windows was replacing 
some of the the drivers included with the Dell Command Deploy driver pack with a
more recent driver, causing the laptop to lose network access during
deployments.

I was able to resolve this issue by adding two additional steps to my task
sequence: one that prevents Windows from checking Windows Update for drivers,
and another rule that reverts the change once the OS is installed.

To fix this, the first thing you will need to do is make sure that you are 
*only* injecting drivers for the specfic model of computer you have. In the
**Inject Drivers** step, you should set the Selection Profile to `Nothing`, and
make sure `Install all drivers from the selection profile` is selected, like so:

![MDT Inject Drivers step](/img/mdt_injectdrivers.png)

This of course assumes that you have set the `DriverGroup001` variable. Here is
a guide on [Spiceworks][0] that walks you through setting this up.

Next, you'll want to add a Run Command Line step in the **PostInstall** stage
right before the Next Phase and Restart Computer steps. The command you will
need to run makes a change in the Windows registry that prevents Windows from
using Windows Update to look for drivers. (Source: [admx.help][1])

    reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\DriverSearching" /v "SearchOrderConfig" /t REG_DWORD /d 0 /f

![MDT Prevent Driver replacements](/img/mdt_nodriverreplace.png)

Finally, add one more step in the State Restore stage that reverts the previous
registry change back to the default of 1.

    reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\DriverSearching" /v "SearchOrderConfig" /t REG_DWORD /d 1 /f

Once done, save your task sequence. If you added any drivers, be sure to rebuild
your deployment share, and replace the boot image in WDS if you PXE boot.

You should no longer be getting any failed deployments that result in a cryptic,
blank Wizard.hta window. Hope this helps!

[0]:https://community.spiceworks.com/how_to/116865-add-drivers-to-mdt-all-versions-total-control-method
[1]:https://admx.help/?Category=Windows_10_2016&Policy=Microsoft.Policies.DeviceSoftwareSetup::DriverSearchPlaces_SearchOrderConfiguration
