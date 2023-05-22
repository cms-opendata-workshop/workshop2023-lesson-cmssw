---
title: "Introduction"
teaching: 7
exercises: 0
questions:
- "What is CMSSW?"
- "How is CMSSW structured?"
objectives:
- "Understand what CMSSW is and how it is organized"
keypoints:
- "The CMS SoftWare (CMSSW) is the software used by the CMS experiment for acquiring, producing, processing and analyzing its data."
- "CMSSW is built in a modular fashion around a main Framework."
- "CMSSW is needed to decode data formats like AOD and miniAOD"
---

## Overview

The CMS Software (CMSSW) is a collection of software libraries that the CMS experiment uses in order to acquire, produce, process and even analyze its data.  The program is written in C++ but its configuration is manipulated using the Python language.  

CMSSW is built around a [Framework](https://github.com/cms-sw/cmssw/tree/CMSSW_7_6_X/FWCore), an [Event Data Model](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookCMSSWFramework#InTro) (EDM), and Services needed by the simulation, calibration and alignment, and reconstruction modules that process event data so that physicists can perform analysis. The primary goal of the Framework and EDM is to facilitate the development and deployment of reconstruction and analysis software.

The [CMSSW repository](https://github.com/cms-sw/cmssw) is on Github. You can browse this huge amount of code, search through it using the [CMSSW Software Cross Reference](https://cmssdt.cern.ch/lxr/) or explore the documentation [here](http://cms-sw.github.io/).

> CMSSW is a continuously-evolving project.  Historically, there has been many releases, which are handled on Github using branches.  Be aware that in this workshop we will use release CMSSW_7_6_7 (the official release for 2015 open data), which can be found in the [CMSSW_7_6_X](https://github.com/cms-sw/cmssw/tree/CMSSW_7_6_X) branch of the repository.  This branch may differ a little or a lot compared to the bleeding-edge one in the [master](https://github.com/cms-sw/cmssw/tree/master) branch, so make sure you are always referencing to the historical one.
{: .testimonial}

## Structure and architecture

As it was mentioned above, the CMSSW software is used for almost all computing activities in CMS.  From data acquisition to data analysis, using different pieces of CMSSW is very intuitive.  Different modules (or plugins) have different functionalities. Some, for instance, are in charge of setting up certain services like the magnetic field configuration (we call this type of code Setup), while others help you create some object that was not there before (EDProducers) or analyze final data (EDAnalyzers).  You can find details [here](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookCMSSWFramework#InTro).  In this workshop, we are only going to look at EDAnalyzers.  In fact, while setting up your environment using Docker, you already [created a simple instance](https://cms-opendata-workshop.github.io/workshop2022-lesson-docker/04-validation/index.html#run-a-simple-demo-for-testing-and-validating) of such an analyzer.

As an example of modularity, take a look at the [package](https://github.com/cms-sw/cmssw/tree/CMSSW_7_6_X/RecoTracker) used for reconstructing tracks.  It has many sub-packages that put in evidence the many bits involved in making a track from detector sensor information.  One of those sub-packages is the [TrackProducer](https://github.com/cms-sw/cmssw/tree/CMSSW_7_6_X/RecoTracker/TrackProducer), which is in charge of putting (recording) the track information in the event.  Note the structure of this sub-package:

![](../fig/trackerproducer.png)

It has the usual look of a C++ repository.  Commonly, you can find a `src` directory with the  bulk of the C++ programming (\*.cc files), an `interface` directory, with mostly the *header* files (\*.h files) matching the code in the `src`, and a `python` directory, where configuration files, written in Python, are stored.  Some other *accesories* are in other directories.  Of course, this is not a standard rule, and many times the structure follows a different logic.  There is also a `Buildfile`, which controls the package dependencies at compile time.

All these packages are, in a sense, plugins to the main [Framework](https://github.com/cms-sw/cmssw/tree/CMSSW_7_6_X/FWCore), which is also a package by itself.

The event data architecture is modular, just as the framework is. Different data layers (using different data formats) can be configured, and a given application can use any layer or layers. The following diagram illustrates this concept if one thinks about how the information from tracks of charged particles is organized:

![](https://twiki.cern.ch/twiki/pub/CMSPublic/WorkBookCMSSWFramework/modular_event_products.gif)

All the information regarding the physics of a collision is stored in the **Event**.  Computationally, one can think of the Event as an object from which you can pull all the information you need from the collision.  

CMS uses different data formats, which are arranged in [tiers](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookDataFormats#EvenT).  Currently CMS open data comes only in the AOD (Run 1 data from 2010-2012) and miniAOD (Run 2 data from 2015) format.  In this workshop we will use the latter format, i.e., [miniAOD](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookMiniAOD) as we will learn to analyze 2015 data.

{% include links.md %}

