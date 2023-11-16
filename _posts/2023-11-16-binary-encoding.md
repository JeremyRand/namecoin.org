---
layout: post
title: "Binary Encoding Support and Testing Enhancements"
author: Robert Nganga
tags: [News]
---


Electrum-NMC has recently undergone significant improvements aimed at refining the user experience and expanding functionality. The most prominent changes involve the introduction of three distinct tabs within the graphical user interface (GUI): Binary, Domain, and ASCII. The Binary tab now serves as a dedicated space for users to interact with the hexadecimal representations of names, providing a standardized and transparent view of binary data. The Domain tab focuses on simplifying interactions by allowing users to employ a simpler format like "name.bit" instead of the original "d/name" format. Meanwhile, the ASCII tab preserves the original format of registration, ensuring compatibility for users who prefer the original format. 

## Graceful Handling of Invalid ASCII

In the event of invalid ASCII input, the system gracefully handles the situation. This allows valid hexadecimal data that is not a valid ASCII representation to be managed without disrupting the user experience.

## AppImage Binaries Testing

A dedicated testing pipeline has been introduced for AppImage binaries. The regression tests run on these binaries to verify their functionality and identify any potential issues. This meticulous approach ensures that users working with AppImage binaries can trust in the stability and performance of Electrum-NMC.

## MacOS Tarball Binaries Testing

For MacOS, a specific testing environment has been created to run regression tests on the tarball binaries. This testing setup not only verifies the integrity of the binaries but also ensures that Electrum-NMC performs seamlessly in a MacOS environment.


These testing procedures cover a range of scenarios and use cases, providing a robust framework to identify and address any potential issues before they impact end-users. The commitment to thorough testing underscores the project's dedication to delivering a reliable and high-performance cryptocurrency wallet.


This work was funded by NLnet Foundation’s NGI Assure Fund.

