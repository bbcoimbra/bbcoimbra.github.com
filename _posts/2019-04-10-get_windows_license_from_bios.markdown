---
layout: post
title: "Get Windows 8+ License Key from BIOS"
---

On Linux Systems
-----------------

    cat /sys/firmware/acpi/tables/MSDM | hexdump -C


On Windows Systems
------------------

Open a terminal (`cmd`) and execute

    wmic path softwarelicensingservice get OA3xOriginalProductKey
