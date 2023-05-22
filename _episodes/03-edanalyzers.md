---
title: "EDAnalyzers"
teaching: 7
exercises: 1
questions:
- "What is an EDAnalyzer and what does it contain?"
- "What files are relevant in an EDAnalyzer?"
objectives:
- "Learn what an EDAnalyzer is and how it is structured"
- "Learn what c++, python and xml files are relevant."

keypoints:
- "An EDAnalyzer is a an edm class that generates a template for any analysis code using CMSSW."
- "There are essentially three important files in an EDAnalyzer package, the source code in c++, the python config file and a Buildfile for tracking dependencies."
---

## Structure

First, make sure you start up your container as discussed in the previous episode.

EDAnalyzers are modules that allow read-only access to the Event. They are useful to produce histograms, reports, statistics, etc.  Take a look at the `DemoAnalyzer` package that we created while validating our CMSSW working environment; it is an example of an EDAnalyzer.  

Go to your `CMSSW_7_6_7/src` area, for example:

~~~
cd /code/CMSSW_7_6_7/src
~~~
{: .language-bash}

Let's explore the DemoAnalyzer package:

~~~
ls Demo/DemoAnalyzer/
~~~
{: .language-bash}

~~~
doc  plugins  python  test
~~~
{: .output}

~~~
ls Demo/DemoAnalyzer/plugins/
~~~
{: .language-bash}

~~~
BuildFile.xml  DemoAnalyzer.cc
~~~
{: .output}

~~~
ls Demo/DemoAnalyzer/python/
~~~
{: .language-bash}

~~~
CfiFile_cfi.py  CfiFile_cfi.pyc  ConfFile_cfg.py  ConfFile_cfg.pyc  __init__.py  __init__.pyc
~~~
{: .output}

Note that it has a similar structure as any of the CMSSW packages we mentioned [before](../01-introduction/index.html#structure-and-architecture).  In this sense, our `DemoAnalyzer` is just one more CMSSW package that we created privately.  However, the headers and implementation of our simple DemoAnalyzer are coded in one single file under the `plugins` directory.  The file was automatically named `DemoAnalyzer.cc` when we created the package.

> CMSSW could be very picky about the structure of its packages.  Most of the time, scripts or other tools expect to have a `Package/Sub-Package` structure, just like our `Demo/DemoAnalyzer` example.
{: .testimonial}

We also notice we have a python configuration file called `ConfFile_cfg` inside the `python` directory.  This is the default configurator for the `DemoAnalyzer.cc` code.

Finally, there is a `BuildFile.xml`, inside the `plugins` directory, where we can include any dependencies if needed so our code can compile without problems.

All EDAnalyzers are created equal; of course, if made with the same `mkedanlzr`, they will look identical.  The `DemoAnalyzer.cc` is a skeleton, written in C++, that contains all the basic ingredients to use CMSSW libraries.  So, in order to perform a physics analysis, and extract information from our CMS open data, we just need to understand what to add to this code and how to configure it.  This is, one way or another, the first and unskippable step when using CMS open data.


{% include links.md %}
