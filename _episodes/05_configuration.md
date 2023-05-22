---
title: "The Configuration"
teaching: 20
exercises: 30
questions:
- "What are the key elements of a CMSSW configuration file?"
- "What kind of elements can I control at the configuration level?"
objectives:
- "Learn the basic structure of the Python implementation of a CMSSW configuration file."
- "Learn how to modify a config file in order to change parameters and/or run additional code."
keypoints:
- "The Python implementation of the configuration of CMSSW is fundamentally modular."
- "The CMSSW configuration allows for the usage of already available code and/or make changes to yours without having to recompile."
---

## CMSSW configuration Framework

The CMS software framework uses a “software bus” model.  A single executable, `cmsRun`, is used, and the modules are loaded at runtime.  A configuration file, fully written in Python, defines which modules are loaded, in which order they are run, and with which configurable parameters they are run. Note that this is not an interactive system. The entire configuration is defined once, at the beginning of the job, and cannot be changed during running.  This is the file that you "feed" `cmRun` when it is executed.

## Playing with the ConfFile_cfg.py file

In the case of our `DemoAnalyzer` we have been working with, its configuration is the `ConfFile_cfg.py` file, which resides in the `Demo/DemoAnalyzer/python` directory of your CMSSW `Demo` package.

If we explore what is in the `Demo/DemoAnalyzer/python` directory:

~~~
ls Demo/DemoAnalyzer/python
~~~
{: .language-bash}

we will get:

~~~
CfiFile_cfi.py  CfiFile_cfi.pyc  ConfFile_cfg.py  ConfFile_cfg.pyc  __init__.py  __init__.pyc
~~~
{: .output}

You will note that there is also a `CfiFile_cfi.py` in there.  We will not pay attention to this file now, but it instructive to point out that the `_cfg` and `_cfi` descriptors are meaningful.  While the former one defines a top level configuration, the latter works more like a *module initialization* file.  There are also `_cff` files which bear pieces of configuration and so they are dubbed *config fragments*.  You can read a bit more about it in [this subsection](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookConfigFileIntro#PythonConfigExamples) of the Workbook.

Ok, so the file we will play around with is just the top level `Demo/DemoAnalyzer/python/ConfFile_cfg.py`. Open it in the editor of your local computer from your working directory in `cms_open_data_work/CMSSW_7_6_7/src`.

The first instructions that you will find in all the top level CMSSW config files are the lines

~~~
import FWCore.ParameterSet.Config as cms

process = cms.Process("Demo")
~~~
{: .language-python}

The first line imports our CMS-specific Python classes and functions, and the second one creates a **process object**.  This refers to a CMSSW process (the one we will be configuring, of course).  Essentially, the main idea is that we will be *feeding* to our process all the tasks that we need executed by the CMSSW software or its plugins.  The process needs always a name. It could be any short word, but it is usually chosen so it is meaningful.  For instance, if the main task would be to process the high level trigger information, an adequate name will be "HLT"; if the process is actually the full reconstruction of the data, then it is most likely assigned the name "RECO".  For our demo, we will leave our creativity aside and just call it "Demo" (you can, of course, change it to your linking).

Then, you will notice a line that *loads* something:

~~~
process.load("FWCore.MessageService.MessageLogger_cfi")
~~~
{: .language-python}

Actually, because of the `_cfi` tag, we know it is presumably a piece of Python code that initializes some module.  Indeed, it is the *MessageLogger* service.  As the name describes, it controls how the message logging is handled during the job execution.  The string ``"FWCore.MessageService.MessageLogger_cfi"`` tells you exactly where to look for it on [Github](https://github.com/cms-sw/cmssw/blob/CMSSW_7_6_X/FWCore/MessageService/python/MessageLogger_cfi.py) if you ever need it.  Note the structure matches the repository's, except that the `python` directory name is always omitted when loading modules this way (this is why it is often important to put config files in the python directory).

There is a whole [Workbook section](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideMessageLogger) regarding this module, but let's just look at a simple example.

> ## Changing the logging frequency in our CMSSW job
>
> Suppose you want the Framework to report every 5 events instead of each event.  Then one can simply add this line
>
> `process.MessageLogger.cerr.FwkReport.reportEvery = 5`
>
> right below the *load* line (actually the place where you copy it hardly matters, but it helps to keep things organized):
>
> ~~~
process.load("FWCore.MessageService.MessageLogger_cfi")
process.MessageLogger.cerr.FwkReport.reportEvery = 5
> ~~~
> {: .language-python}
{: .challenge}

Note that all we are doing is loading the MessageLogger module and changing just one parameter, the one in [this line](https://github.com/cms-sw/cmssw/blob/CMSSW_7_6_X/FWCore/MessageService/python/MessageLogger_cfi.py#L29), instead of going with the default value, which is one.

For the next line

~~~
process.maxEvents = cms.untracked.PSet( input = cms.untracked.int32(10) )
~~~
{: .language-python}

it is easy to guess that it controls the number of events that are going to be processed in our CMSSW job.  It is worth noting that `maxEvents` is an *untracked* variable within the Framework.  In general, the system keeps track of what parameters are used to create each data item in the Event and saves this information in the output files. This can be used later to help understand how the data was made. However, sometimes a parameter will have no effect on the final objects created. Such parameters are declared *untracked*.  More information can be found in the [Workbook](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideAboutPythonConfigFile#Parameters).

Let's change the number of events to `100`:

~~~
process.maxEvents = cms.untracked.PSet( input = cms.untracked.int32(100) )
~~~
{: .language-python}

Next, there is the first module (also an object by itself) we are attaching to our process object:

~~~
process.source = cms.Source("PoolSource",
    # replace 'myfile.root' with the source file you want to use
    fileNames = cms.untracked.vstring(
        #'file:myfile.root'
        'root://eospublic.cern.ch//eos/opendata/cms/Run2015D/SingleElectron/MINIAOD/08Jun2016-v1/10000/001A703B-B52E-E611-BA13-0025905A60B6.root'
    )
)

~~~
{: .language-python}


Inside the process object there must be exactly one object assigned that has Python type `Source` and is used for data input. There may be zero or more objects for each of many other Python types. In the official production configurations there can be hundreds or even thousands of objects attached to the process. Your job is configured by your choice of objects to construct and attach to the process, and by the configuration of each object. (This may be done via [import statements](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideAboutPythonConfigFile#TheImportStatment) or calls to the load function, instead of or in addition to object construction.) Some of the Python types that may be used to create these objects are listed in the [Workbook](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideAboutPythonConfigFile#Attribute_Declarations_for_the_P).


> ## Explore the PoolSource C++
>
> Generally, all modules in our python configuration are associated with its corresponding C++ code.  By searching the [CMSSW Github repository](https://github.com/cms-sw/cmssw), would you be able to point exactly to the line of C++ code where the `fileNames` parameter is read?
>
> > ## solution
> >
> > You will find that the label first appearing in a Python CMSSW module is actually the name of the C++ code.  So, we would expect that there be a class `PoolSource.h` (and perhaps its implementation as `PoolSource.C`) associated with the "PoolSource" label.  Let's then go to the [Github CMSSW repository](https://github.com/cms-sw/cmssw) and simply search for `PoolSource.C` using the search field of that page.  Immediately, in the [search results](https://github.com/cms-sw/cmssw/search?q=PoolSource), we notice there is a `IOPool/Input/src/PoolSource.cc` file that we can browse.  After looking for the variable `fileNames`, we find that this parameter is read in [this line](https://github.com/cms-sw/cmssw/blob/abc9134327fff8fbb378b3c81a56c5850195a9bb/IOPool/Input/src/PoolSource.cc#L68).  Note that it is in the constructor of the object where the `ParameterSet` objects are read.
> {: .solution}
{: .challenge}

Note also that the `fileNames` variable is a `vstring`, i.e., a vector of strings in the C++ sense.  In Python, it is a list, so you can very well input a comma separated list of files.  There is a drawback, though.  In general, our open datasets will contain more than 255 files, which is the limit for the number of arguments a Python function can take, so very long vstrings cannot be created in one step.  There are [various alternatives](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuidePythonTips#Running_on_more_than_255_files) to circumvent this problem.  To run over massive amounts of ROOT files, one will usually use the [FileUtils](https://github.com/cms-sw/cmssw/blob/master/FWCore/Utilities/python/FileUtils.py) module to load index files instead of individual ROOT files.


## Configure our DemoAnalyzer

The second to last line in our configuration
~~~
process.demo = cms.EDAnalyzer('DemoAnalyzer'
)
~~~
{: .language-python}

has to do with our recently created `DemoAnalyzer`.  This module is now just a *declaration of existance* because it is empty.  Let's put it to work:

> ## Making our EDAnalyzer configurable
>
> Recall that, previously, we [registered data access](../04_source/index.html#registering-for-data-access) for *slimmedMuons*.  However as, we saw, there could be some objects which have more than one collection, like it can be seen in this [table](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookMiniAOD2015#High_level_physics_objects).  Therefore we would like to make the selection of this tag (slimmedMuons) configurable.  Would you be able to implement that?
>
> > ## Solution
> >
> > Since we are going to make our DemoAnalyzer configurable, the first thing we need to do is to modify the C++ source of our analyzer in order to accommodate configurability. Let's modify then the `Demo/DemoAnalyzer/plugins/DemoAnalyzer.cc` file.  Again, following the logic in the [Physics Ojects guide](https://cms-opendata-guide.web.cern.ch/analysis/selection/objects/objects/#access-methods) and using an editor, we should read the token from the *ParameterSet*.
> >
> >
> > We will have to read this *InputTag* from the configuration.  As it was noted above, this is done in the constructor.  One option is to read and assign the value to the relevant variable directly.  The constructor will need to be modified like:
> >
> > ~~~
> > DemoAnalyzer::DemoAnalyzer(const edm::ParameterSet& iConfig): muonToken_(consumes<pat::MuonCollection>(iConfig.getParameter<edm::InputTag>("muons")))
> >
> > {
> >    //now do what ever initialization is needed
> >    usesResource("TFileService");
> >
> >
> > }
> >
> > ~~~
> > {: .language-cpp}
> >
> > Here we will be reading the `Collection` variable from configuration (which is of type `edm::InputTag`, which is [essentially a string](https://github.com/cms-sw/cmssw/blob/52ef6482b221be8c1516bcc6eab63015d4e1fb72/FWCore/Utilities/interface/InputTag.h#L15)) and will store it in the `muonToken_` variable.  Note the parameter read is called *muons*
> >
> > In this way, the string tag *slimmedMuons* is not harcoded any more; it can be changed if needed at configuration time.
> >
> > Finally, let's change the `Demo/DemoAnalyzer/python/ConfFile_cfg.py` by replacing our empty module statement:
> >
> > `process.demo = cms.EDAnalyzer('DemoAnalyzer')`
> >
> > with
> >
> > ~~~
> > process.demo = cms.EDAnalyzer('DemoAnalyzer',
> >        muons = cms.InputTag("slimmedMuons")
> > )
> > ~~~
> > {: .language-python}
> >
> > In this way, we are now able to enter "slimmedMuons" or the token of any other collection, depending on our needs.
> {: .solution}
{: .challenge}

Now, before re-compiling our code, let's check that our python configuration is ok.  Go to the container shell. We can validate the syntax of your configuration using python:

~~~
python Demo/DemoAnalyzer/python/ConfFile_cfg.py
~~~
{: .language-bash}

If there are no errors, you are good to go.

Since we modified the source code, let's compile the code again, with `scram ` in the container:

~~~
scram b
~~~
{: .language-bash}

If everything goes well, you should see something like:

~~~
>> Local Products Rules ..... started
>> Local Products Rules ..... done
>> Building CMSSW version CMSSW_7_6_7 ----
>> Entering Package Demo/DemoAnalyzer
>> Creating project symlinks
  src/Demo/DemoAnalyzer/python -> python/Demo/DemoAnalyzer
>> Compiling edm plugin /code/CMSSW_7_6_7/src/Demo/DemoAnalyzer/plugins/DemoAnalyzer.cc
>> Building edm plugin tmp/slc6_amd64_gcc493/src/Demo/DemoAnalyzer/plugins/DemoDemoAnalyzerAuto/libDemoDemoAnalyzerAuto.so
Leaving library rule at src/Demo/DemoAnalyzer/plugins
@@@@ Running edmWriteConfigs for DemoDemoAnalyzerAuto
--- Registered EDM Plugin: DemoDemoAnalyzerAuto
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
@@@@ Refreshing Plugins:edmPluginRefresh
>> Pluging of all type refreshed.
>> Done generating edm plugin poisoned information
gmake[1]: Leaving directory `/code/CMSSW_7_6_7'
~~~
{: .output}

Finally, let's run the CMSSW job:

~~~
cmsRun Demo/DemoAnalyzer/python/ConfFile_cfg.py > mylog.log 2>&1 &
~~~
{: language-bash}

If you check the development of the job with

~~~
tail -f mylog.log
~~~

Eventually, as the job progresses, you will see something like:

~~~
14-Jul-2022 18:37:29 CEST  Successfully opened file root://eospublic.cern.ch//eos/opendata/cms/Run2015D/SingleElectron/MINIAOD/08Jun2016-v1/10000/001A703B-B52E-E611-BA13-0025905A60B6.root
Begin processing the 1st record. Run 257645, Event 1184198851, LumiSection 776 at 14-Jul-2022 18:37:31.600 CEST
Muon # 0 with E = 15.2976 GeV.
Muon # 0 with E = 6.0168 GeV.
Begin processing the 6th record. Run 257645, Event 1184269130, LumiSection 776 at 14-Jul-2022 18:37:33.710 CEST
Muon # 0 with E = 8.94318 GeV.
Muon # 0 with E = 4.05406 GeV.
Muon # 0 with E = 4.2305 GeV.
Muon # 0 with E = 8.99778 GeV.
Muon # 0 with E = 8.0897 GeV.
Muon # 1 with E = 4.36828 GeV.
14-Jul-2022 18:37:33 CEST  Closed file root://eospublic.cern.ch//eos/opendata/cms/Run2015D/SingleElectron/MINIAOD/08Jun2016-v1/10000/001A703B-B52E-E611-BA13-0025905A60B6.root

=============================================

MessageLogger Summary

 type     category        sev    module        subroutine        count    total
 ---- -------------------- -- ---------------- ----------------  -----    -----
    1 XrdAdaptor           -w file_open                              1        1
    2 fileAction           -s file_close                             1        1
    3 fileAction           -s file_open                              2        2

 type    category    Examples: run/evt        run/evt          run/evt
 ---- -------------------- ---------------- ---------------- ----------------
    1 XrdAdaptor           pre-events                        
    2 fileAction           PostEndRun                        
    3 fileAction           pre-events       pre-events       

Severity    # Occurrences   Total Occurrences
--------    -------------   -----------------
Warning                 1                   1
System                  3                   3

~~~
{: .output}

> ## Change the InputTag
>
> Now, change the name of the `InputColletion` from "slimmedMuons" to "slimmedElectrons" in your configuration and run again **without** re-compiling the code. Do you see any difference?
>
> > ## Solution
> >
> > After changing the tag to "slimmedElectrons" this is the output we get:
> > 
> > ~~~
> > 14-Jul-2022 18:31:47 CEST  Initiating request to open file root://eospublic.cern.ch//eos/opendata/cms/Run2015D/SingleElectron/MINIAOD/08Jun2016-v1/10000/001A703B-B52E-E611-BA13-0025905A60B6.root
> > 220714 18:31:47 5623 secgsi_InitProxy: cannot access private key file: /home/cmsusr/.globus/userkey.pem
> > %MSG-w XrdAdaptor:  file_open 14-Jul-2022 18:31:47 CEST pre-events
> > Data is served from cern.ch instead of original site eospublic
> > %MSG
> > 14-Jul-2022 18:31:49 CEST  Successfully opened file root://eospublic.cern.ch//eos/opendata/cms/Run2015D/SingleElectron/MINIAOD/08Jun2016-v1/10000/001A703B-B52E-E611-BA13-0025905A60B6.root
> > Begin processing the 1st record. Run 257645, Event 1184198851, LumiSection 776 at 14-Jul-2022 18:31:51.574 CEST
> > Begin processing the 6th record. Run 257645, Event 1184269130, LumiSection 776 at 14-Jul-2022 18:31:51.576 CEST
> > 14-Jul-2022 18:31:51 CEST  Closed file root://eospublic.cern.ch//eos/opendata/cms/Run2015D/SingleElectron/MINIAOD/08Jun2016-v1/10000/001A703B-B52E-E611-BA13-0025905A60B6.root
> > 
> > =============================================
> > 
> > MessageLogger Summary
> > 
> >  type     category        sev    module        subroutine        count    total
> >  ---- -------------------- -- ---------------- ----------------  -----    -----
> >     1 XrdAdaptor           -w file_open                              1        1
> >     2 fileAction           -s file_close                             1        1
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
> > ~~~
> > {: .output}
> >
> > Of course, we are not going to see anything because there is no slimmedElectrons tag for muon objects.
> {: .solution}
{: .challenge}



## Running some already-available CMSSW code

The last line in our `Demo/DemoAnalyzer/python/ConfFile_cfg.py` config file is

~~~
process.p = cms.Path(process.demo)
~~~
{: .language-python}

The “software bus” model that was mentioned in the introduction of this episode can be made evident in this line.  CMSSW executes its code using *Paths* (which in turn could be arranged in [Schedules](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideAboutPythonConfigFile#Schedule_statements)).  Each Path can execute a series of modules (or [Sequences](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideAboutPythonConfigFile#Module_sequences) of modules).  In our example we have just one Path named `p` that executes the `demo` process, which corresponds to our DemoAnalyzer.

In general, however, we could add more modules.  For instance, the Path line could look like

~~~
process.mypath = cms.Path (process.m1+process.m2+process.s1+process.m3)
~~~
{: .language-python}

, where `m1`, `m2`, `m3` could be CMSSW modules (individual EDAnalyzers, EDFilters, EDProducers, etc.) and `s1` could even be a [modules Sequence](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideAboutPythonConfigFile#Module_sequences).

> ## Adding a Trigger Filter
>
> In CMSSW, there are other types of code one can execute.  Some of these are known as EDFilters.  As the name implies, they can be used to filter events.  For instance, one could use the [HLTHighLevel](https://github.com/cms-sw/cmssw/blob/CMSSW_7_6_X/HLTrigger/HLTfilters/interface/HLTHighLevel.h) filter class to only run over events that have passed a certain kind of trigger.
>
> We can get a hint of its usage by exploring the [constructor](https://github.com/cms-sw/cmssw/blob/fb9777b1d76e3896aff70a926799eb3ed514f168/HLTrigger/HLTfilters/src/HLTHighLevel.cc#L39) of that code (remember that the configuration parameters are passed there). After some careful inspection we conclude that what we need to add to our configuration is the following module:
>
> ~~~
> process.hltHighLevel = cms.EDFilter("HLTHighLevel",
>    TriggerResultsTag = cms.InputTag("TriggerResults","","HLT"),
>    HLTPaths = cms.vstring('HLT_Ele27_WPLoose_Gsf_v*'),           # provide list of HLT paths (or patterns) you want
>    eventSetupPathsKey = cms.string(''), # not empty => use read paths from AlCaRecoTriggerBitsRcd via this key
>    andOr = cms.bool(True),             # how to deal with multiple triggers: True (OR) accept if ANY is true, False (AND) accept if ALL are true
>    throw = cms.bool(True)    # throw exception on unknown path names
> )
> ~~~
> {: .language-python}
>
> Of course, we need to add this module to the CMSSW path for execution. When running first, it will filter events that do not have electrons firing
> a version of the `HLT_Ele27_WPLoose_Gsf` trigger.  You will learn more about triggers during the workshop.
>
> > ## The full config file
> >
> > **Do not forget to switch back to "slimmedMuons"** for the InputTag.  The full config file will then look like:
> >
> > ~~~
> > import FWCore.ParameterSet.Config as cms
> > 
> > process = cms.Process("Demo")
> > 
> > process.load("FWCore.MessageService.MessageLogger_cfi")
> > process.MessageLogger.cerr.FwkReport.reportEvery = 5
> > 
> > process.maxEvents = cms.untracked.PSet( input = cms.untracked.int32(10) )
> > 
> > process.source = cms.Source("PoolSource",
> >     # replace 'myfile.root' with the source file you want to use
> >     fileNames = cms.untracked.vstring(
> >         #'file:myfile.root'
> >         'root://eospublic.cern.ch//eos/opendata/cms/Run2015D/SingleElectron/MINIAOD/08Jun2016-v1/10000/001A703B-B52E-E611-BA13-0025905A60B6.root'
> >     )
> > )
> > 
> > process.demo = cms.EDAnalyzer('DemoAnalyzer',
> >         muons = cms.InputTag("slimmedMuons")
> > )
> > 
> > process.hltHighLevel = cms.EDFilter("HLTHighLevel",
> >     TriggerResultsTag = cms.InputTag("TriggerResults","","HLT"),
> >     HLTPaths = cms.vstring('HLT_Ele27_WPLoose_Gsf_v*'),           # provide list of HLT paths (or patterns) you want
> >     eventSetupPathsKey = cms.string(''), # not empty => use read paths from AlCaRecoTriggerBitsRcd via this key
> >     andOr = cms.bool(True),             # how to deal with multiple triggers: True (OR) accept if ANY is true, False (AND) accept if ALL are true
> >     throw = cms.bool(True)    # throw exception on unknown path names
> > )
> > 
> > process.p = cms.Path(process.hltHighLevel+process.demo)
> > 
> > ~~~
> > {: .language-python}
> {: .solution}
>
> Without even having to compile again, the execution of the trigger path will stop if the `hltHighLevel` filter module throws a `False` result. The output becomes
>
> ~~~
> Begin processing the 1st record. Run 257645, Event 1184198851, LumiSection 776 at 14-Jul-2022 21:02:00.208 CEST
> Begin processing the 6th record. Run 257645, Event 1184269130, LumiSection 776 at 14-Jul-2022 21:02:02.132 CEST
> Muon # 0 with E = 8.94318 GeV.
> Muon # 0 with E = 4.05406 GeV.
> Muon # 0 with E = 4.2305 GeV.
> Muon # 0 with E = 8.99778 GeV.
> 14-Jul-2022 21:02:02 CEST  Closed file root://eospublic.cern.ch//eos/opendata/cms/Run2015D/SingleElectron/MINIAOD/08Jun2016-v1/10000/> 001A703B-B52E-E611-BA13-0025905A60B6.root
> 
> =============================================
> 
> MessageLogger Summary
> 
>  type     category        sev    module        subroutine        count    total
>  ---- -------------------- -- ---------------- ----------------  -----    -----
>     1 XrdAdaptor           -w file_open                              1        1
>     2 fileAction           -s file_close                             1        1
>     3 fileAction           -s file_open                              2        2
> 
>  type    category    Examples: run/evt        run/evt          run/evt
>  ---- -------------------- ---------------- ---------------- ----------------
>     1 XrdAdaptor           pre-events                        
>     2 fileAction           PostEndRun                        
>     3 fileAction           pre-events       pre-events       
> 
> Severity    # Occurrences   Total Occurrences
> --------    -------------   -----------------
> Warning                 1                   1
> System                  3                   3
> 
> ~~~
> {: .output}
>
>  It is obvious that the first event (at least) was filtered out by by the trigger filter we added.
{: .challenge}


Congratulations!!, you have made it to the end the lesson.


{% include links.md %}
