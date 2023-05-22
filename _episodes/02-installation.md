---
title: "Installation and Execution"
teaching: 0
exercises: 20
questions:
- "How do I install CMSSW?"
- "How do I compile and execute CMSSW?"
objectives:
- "Review the steps necessary to setup a CMSSW area."
- "Learn how to compile and execute CMSSW jobs."

keypoints:
- "A CMSSW area is not really installed but set up."
- "`cmsRun` is the CMSSW executable.  There are also utilitarian scripts."
- "You can compile CMSSW with `scram b`"
---

## Setting up your CMSSW area

If you completed the lessons on [Docker](https://cms-opendata-workshop.github.io/workshop2022-lesson-docker) containers you should already have a working CMSSW area.  


If you have closed the container, re-start it with:

~~~
docker start -i <theNameOfyourContainer>
~~~
{: .language-bash}

Make sure you are in the `CMSSW_7_6_7/src` area of the container. This is the default directory when you start the container, but if you have changed to another directory, come back to it with:

~~~
cd /code/CMSSW_7_6_7/src
~~~
{: .language-bash}

Note that we are not really "installing" CMSSW but setting up an environment for it.  CMSSW was already installed in the container. Every time you start the container a script runs by default to set up a few environmental variables that are needed.  

For instance, you can check where your `CMSSW_RELEASE_BASE` variable points to:

~~~
echo $CMSSW_RELEASE_BASE
~~~
{: .language-bash}

The variable points to a local CMSSW install in the Docker container:

~~~
/cvmfs/cms.cern.ch/slc6_amd64_gcc493/cms/cmssw/CMSSW_7_6_7
~~~
{: .output}

> ## Try a different variable
>
> Can you check where the variable CMSSW_BASE points to?
>
{: .challenge}



## **cmsRun**, the CMSSW executable

All the packages that comprise the CMSSW release in use have been already compiled and linked to one single **executable**, which is called `cmsRun`.  So, unless you want to create your own plugin (addition) for the software, you won't even have to re-compile.  You can actually try to execute this command by itself, but it will give you a configuration error:

~~~
cmsRun
~~~
{: .language-bash}

~~~
cmsRun: No configuration file given.
For usage and an options list, please do 'cmsRun --help'.
~~~
{: .error}

So, inevitably, the cmsRun executable needs a configuration file.  This configuration file must be written in Python.  Do not worry, we will expand on this later.  For now, let's try a simple example.

> ## Run with a configuration
>
> Now try to run, but this time with a configuration file.
>
> > ## Solution
> >
> > We could simply repeat [what we already did](https://cms-opendata-workshop.github.io/workshop2022-lesson-docker/04-validation/index.html#run-a-simple-demo-for-testing-and-validating) while setting up our Docker container: run with the `Demo/DemoAnalyzer/demoanalyzer_cfg.py` python configuration file.  
> > To run in a more efficient way, although it is not strictly necessary, we use the bash redirector `>`, the redirection of `stderr` to `stdout` (`2>&1`), and the trailing run-in-the-background control operator `&`.  This allows us to send the output to a `dummy.log` file and run the proces in the background so we can still interact with the terminal.  
> >
> > ~~~
> > cmsRun Demo/DemoAnalyzer/python/ConfFile_cfg.py > dummy.log 2>&1 &
> > ~~~
> > {: .language-bash}
> >
> > You can check the development of your job with
> >
> > ~~~
> > tail -f dummy.log
> > ~~~
> >
> > When finished, if you dumped the content of `dummy.log`, and do
> >
> > ~~~
> > cat dummy.log
> > ~~~
> >
> > you'll get an output similar to:
> >
> > ~~~
> > 14-Jul-2022 14:42:10 CEST  Initiating request to open file root://eospublic.cern.ch//eos/opendata/cms/Run2015D/SingleElectron/MINIAOD/08Jun2016-v1/10000/001A703B-B52E-E611-BA13-0025905A60B6.root
> > %MSG-w XrdAdaptor:  file_open 14-Jul-2022 14:42:11 CEST pre-events
> > Data is served from cern.ch instead of original site eospublic
> > %MSG
> > 14-Jul-2022 14:42:13 CEST  Successfully opened file root://eospublic.cern.ch//eos/opendata/cms/Run2015D/SingleElectron/MINIAOD/08Jun2016-v1/10000/001A703B-B52E-E611-BA13-0025905A60B6.root
> > (/code/CMSSW_7_6_7/src) cat dummy.log 
> > 14-Jul-2022 14:42:10 CEST  Initiating request to open file root://eospublic.cern.ch//eos/opendata/cms/Run2015D/SingleElectron/MINIAOD/08Jun2016-v1/10000/001A703B-B52E-E611-BA13-0025905A60B6.root
> > %MSG-w XrdAdaptor:  file_open 14-Jul-2022 14:42:11 CEST pre-events
> > Data is served from cern.ch instead of original site eospublic
> > %MSG
> > 14-Jul-2022 14:42:13 CEST  Successfully opened file root://eospublic.cern.ch//eos/opendata/cms/Run2015D/SingleElectron/MINIAOD/08Jun2016-v1/10000/001A703B-B52E-E611-BA13-0025905A60B6.root
> > Begin processing the 1st record. Run 257645, Event 1184198851, LumiSection 776 at 14-Jul-2022 14:42:23.233 CEST
> > Begin processing the 2nd record. Run 257645, Event 1184202760, LumiSection 776 at 14-Jul-2022 14:42:23.234 CEST
> > Begin processing the 3rd record. Run 257645, Event 1183968519, LumiSection 776 at 14-Jul-2022 14:42:23.234 CEST
> > Begin processing the 4th record. Run 257645, Event 1183964627, LumiSection 776 at 14-Jul-2022 14:42:23.235 CEST
> > Begin processing the 5th record. Run 257645, Event 1184761030, LumiSection 776 at 14-Jul-2022 14:42:23.235 CEST
> > Begin processing the 6th record. Run 257645, Event 1184269130, LumiSection 776 at 14-Jul-2022 14:42:23.235 CEST
> > Begin processing the 7th record. Run 257645, Event 1184358918, LumiSection 776 at 14-Jul-2022 14:42:23.236 CEST
> > Begin processing the 8th record. Run 257645, Event 1183874827, LumiSection 776 at 14-Jul-2022 14:42:23.236 CEST
> > Begin processing the 9th record. Run 257645, Event 1184415529, LumiSection 776 at 14-Jul-2022 14:42:23.237 CEST
> > Begin processing the 10th record. Run 257645, Event 1184425291, LumiSection 776 at 14-Jul-2022 14:42:23.237 CEST
> > 14-Jul-2022 14:42:23 CEST  Closed file root://eospublic.cern.ch//eos/opendata/cms/Run2015D/SingleElectron/MINIAOD/08Jun2016-v1/10000/001A703B-B52E-E611-BA13-0025905A60B6.root
> > 
> > =============================================
> > 
> > MessageLogger Summary
> > 
> >  type     category        sev    module        subroutine        count    total
> >  ---- -------------------- -- ---------------- ----------------  -----    -----
> >     1 XrdAdaptor           -w file_open                              1        1
> >    2 fileAction           -s file_close                             1        1
> >     3 fileAction           -s file_open                              2        2
> > 
> >  type    category    Examples: run/evt        run/evt          run/evt
> >  ---- -------------------- ---------------- ---------------- ----------------
> >     1 XrdAdaptor           pre-events                        
> >     2 fileAction           PostEndRun                        
> >     3 fileAction           pre-events       pre-events       
> > 
> > Severity    # Occurrences   Total Occurrences
> > --------    -------------   -----------------
> > Warning                 1                   1
> > System                  3                   3
> >
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

## Compilation

We use scram, the release management tool used for CMSSW, to compile (**b**uild) the code:
~~~
scram b
~~~
{: .language-bash}

~~~
Reading cached build data
>> Local Products Rules ..... started
>> Local Products Rules ..... done
>> Building CMSSW version CMSSW_7_6_7 ----
>> Entering Package Demo/DemoAnalyzer
>> Creating project symlinks
  src/Demo/DemoAnalyzer/python -> python/Demo/DemoAnalyzer
>> Leaving Package Demo/DemoAnalyzer
>> Package Demo/DemoAnalyzer built
>> Subsystem Demo built
>> Subsystem BigProducts built
>> Local Products Rules ..... started
>> Local Products Rules ..... done
gmake[1]: Entering directory `/code/CMSSW_7_6_7'
>> Creating project symlinks
  src/Demo/DemoAnalyzer/python -> python/Demo/DemoAnalyzer
>> Done python_symlink
>> Compiling python modules cfipython/slc6_amd64_gcc493
>> Compiling python modules python
>> Compiling python modules src/Demo/DemoAnalyzer/python
>> All python modules compiled
>> Pluging of all type refreshed.
>> Done generating edm plugin poisoned information
gmake[1]: Leaving directory `/code/CMSSW_7_6_7'
~~~
{: .output}

Note that scram only goes into the `Demo/DemoAnalyzer` package that we created locally to validate our setup in a previous lesson.  The rest of the packages, that live on the installation area (remember the area where `CMSSW_RELEASE_BASE` points to), were already compiled.  Since there is nothing new to compile, it finishes very quickly.  In a later episode we will modify this `DemoAnalyzer` and will need to compile again.

> Point to be made: if you compile at main `src` level, all the packages in there will be compiled.  However, if you go inside a specific package or sub-package, like our `Demo/DemoAnalyzer`, only the code in that subpackage will be compiled.
{: .testimonial}


## Additional goodies

Your CMSSW environment comes with other executable scripts/tools that can be very useful.  An example of those is the `mkedanlzr` script that we used already to create the `DemoAnalyzer` package.  This script creates skeletons for EDAnalyzers that can be later modified or expanded.  Notice that this package, `DemoAnalyzer`, has a similar structure as any of the CMSSW packages we mentioned [before](../01-introduction/index.html#structure-and-architecture).

One can find out about [other scripts](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideSkeletonCodeGenerator) like mkedanlzr by typing `mked` and hitting the <kbd>Tab</kbd> key (maybe twice):
~~~
mked + Tab + Tab
~~~
{: .language-bash}

~~~
mkedanlzr  mkedfltr   mkedlpr    mkedprod
~~~
{: .output}

In this workshop, however, we will not be using those other ones.

There are also additional scripts, like the Event Data Model(EDM) [utilities](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookEdmUtilities), the [hltGetConfiguration](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideHltGetConfiguration) trigger [dumper](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideLegacyGlobalHLT#Dumping_the_latest_configuration), or the [cmsDriver](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideCmsDriver), which can be very useful.  Now, let's check an example.

> ## Finding the EventSize of a ROOT EDM file
>
> Now, as a simple exercise, use one of the EDM utilities mentioned above to find out about the number of events in the ROOT file that is in the `Demo/DemoAnalyzer/python/ConfFile_cfg.py` config file of your analyzer package.  Try to figure it out from the documentation above before looking at the solution.
>
> > ## Solution
> >
> > After checking the documentation above, the following one-liner will work.
> >
> > ~~~
> > edmEventSize -v root://eospublic.cern.ch//eos/opendata/cms/Run2015D/SingleElectron/MINIAOD/08Jun2016-v1/10000/001A703B-B52E-E611-BA13-0025905A60B6.root | grep Events
> > ~~~
> > {: .language-bash}
> >
> > ~~~
> > File root://eospublic.cern.ch//eos/opendata/cms/Run2015D/SingleElectron/MINIAOD/08Jun2016-v1/10000/001A703B-B52E-E611-BA13-0025905A60B6.root Events 49871
> > ~~~
> > {: .output}
> >
> > So, the ROOT file has 49871 events.
> >
> {: .solution}
{: .challenge}

{% include links.md %}
