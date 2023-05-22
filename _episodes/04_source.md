---
title: "The Source"
teaching: 10
exercises: 30
questions:
- "What are the elements of the source code of an EDAnalyzer?"
- "How do I modify the source to get additional information?"
objectives:
- "Learn the basic structure of the C++ implementation of an EDAnalyzer."
- "Learn the basics on how to modify the source in order to extract physics information good for analysis."
keypoints:
- "The C++ source code file of an EDAnalyzer is taylored for particle physics analysis under the CMSSW Framework."
- "This source file needs to be modified according to the analyzer needs"
---

## Playing with the DemoAnalyzer.cc file

The `DemoAnalyzer.cc` file in the `Demo/DemoAnalyzer/plugins/` directory is the main file of our EDAnalyzer.  As it was mentioned, the default structure is always the same.  Have a look at what is inside it using your favourite editor of your local computer. Remember that you will find the `Demo` area under the `cms_open_data_work/CMSSW_7_6_7/src` directory on your local computer. As the working directory has been mounted into the container, all changes take effect there as well.

The first thing that you will see is a set of *includes*:

~~~
// system include files
#include <memory>

// user include files
#include "FWCore/Framework/interface/Frameworkfwd.h"
#include "FWCore/Framework/interface/EDAnalyzer.h"

#include "FWCore/Framework/interface/Event.h"
#include "FWCore/Framework/interface/MakerMacros.h"

#include "FWCore/ParameterSet/interface/ParameterSet.h"
~~~
{: .language-cpp}

These are the most basic *Framework* classes that are needed to mobilize the CMSSW machinery.  In particular, notice the `Event.h` class.  This class contains essentially all the *accessors* that are needed to extract information from *the Event*, i.e., from the particle collision.  Another important class is the `ParameterSet.h`.  This one will allow us to extract configuration parameters, which can be manipulated using the `Demo/DemoAnalyzer/python/ConfFile_cfg.py` python file.

Something important to take into account is that you can learn a lot about the kind of information you have access to by exploring the code in the CMSSW repository on Github.  For instance, you can look at the [Event.h](https://github.com/cms-sw/cmssw/blob/CMSSW_7_6_X/FWCore/Framework/interface/Event.h) header and check all the available methods. You will notice, for instance, the presence of the `getByToken` accessors; we will be using one these to access physics objects.

> When exploring CMSSW code on Github, remember to choose the CMSSW_7_6_X branch to match the release we are using for this workshop.
{: .testimonial}

> If you need to look at CMS Run 1 open data, please look at [previous tutorials](https://cms-opendata-workshop.github.io/2021-07-19-cms-open-data-workshop/).  The essential ideas are similar, but due to the different dataformats (miniAOD in Run 2 as opposed to AOD in Run 1), accessing the information is a bit different.  This workshop focuses on Run 2 (miniAOD format) data.
{: .testimonial}

> ## Including muon headers
>
> Let's pretend that we are interested in extracting the energy of all the muons in the event.  We would need to add the appropriate classes for this.  After quickly reviewing [this chapter](https://cms-opendata-guide.web.cern.ch/analysis/selection/objects/objects/) (make sure you pick the `Run 2` data tabs) of the CMS Open Data Guide (which is still under construction), we conclude that we need to add this header line to our analyzer:
>
> ```
> //classes to extract Muon information
> #include "DataFormats/PatCandidates/interface/Muon.h"
> ```
>
> Let's add it at the end of the header section together with the standard vector C++ library:
>
> ```
> #include<vector>
> ```
>
> So the header section of the `Demo/DemoAnalyzer/plugins/DemoAnalyzer.cc` source file becomes:
>
> ~~~
> // system include files
> #include <memory>
>
> // user include files
> #include "FWCore/Framework/interface/Frameworkfwd.h"
> #include "FWCore/Framework/interface/one/EDAnalyzer.h"
>
> #include "FWCore/Framework/interface/Event.h"
> #include "FWCore/Framework/interface/MakerMacros.h"
> 
> #include "FWCore/ParameterSet/interface/ParameterSet.h"
> 
> //class to extract muon info
> #include "DataFormats/PatCandidates/interface/Muon.h"
> 
> #include<vector>
> ~~~
> {: .language-cpp}
{: .challenge}

Next, you will see the class declaration:

~~~
//
// class declaration
//

// If the analyzer does not use TFileService, please remove
// the template argument to the base class so the class inherits
// from  edm::one::EDAnalyzer<> and also remove the line from
// constructor "usesResource("TFileService");"
// This will improve performance in multithreaded jobs.

class DemoAnalyzer : public edm::one::EDAnalyzer<edm::one::SharedResources>  {
   public:
      explicit DemoAnalyzer(const edm::ParameterSet&);
      ~DemoAnalyzer();

      static void fillDescriptions(edm::ConfigurationDescriptions& descriptions);


   private:
      virtual void beginJob() override;
      virtual void analyze(const edm::Event&, const edm::EventSetup&) override;
      virtual void endJob() override;

      // ----------member data ---------------------------
};
~~~
{: .language-cpp}

The first thing one notices is that our class inherits from the `edm::EDAnalyzer` class.  It follows the same structure as any class in C++.  The declaration of the methods reflect the functionality needed for particle physics analysis.  Their implementation is further below in the same file.

> ## Declaring info containers
>
> Let's add the declaration of a vector for our energy values:  
>
> ```
> std::vector<float> muon_e;
> ```
>
> and a *token*, which is needed for [registering for data access](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideEDMGetDataFromEvent#Registering_for_data_access).
>
> This section becomes:
>
> ~~~
> //
> // class declaration
> //
> 
> // If the analyzer does not use TFileService, please remove
> // the template argument to the base class so the class inherits
> // from  edm::one::EDAnalyzer<> and also remove the line from
> // constructor "usesResource("TFileService");"
> // This will improve performance in multithreaded jobs.
> 
> class DemoAnalyzer : public edm::one::EDAnalyzer<edm::one::SharedResources>  {
>    public:
>       explicit DemoAnalyzer(const edm::ParameterSet&);
>       ~DemoAnalyzer();
> 
>       static void fillDescriptions(edm::ConfigurationDescriptions& descriptions);
> 
> 
>    private:
>       virtual void beginJob() override;
>       virtual void analyze(const edm::Event&, const edm::EventSetup&) override;
>       virtual void endJob() override;
> 
>       // ----------member data ---------------------------
>     
>      std::vector<float> muon_e; //energy values for muons in the event
>      edm::EDGetTokenT<pat::MuonCollection> muonToken_;
> };
> ~~~
> {: .language-cpp}
{: .challenge}


Next, we can see the constructor and destructor of our DemoAnalyzer class:

~~~
//
// constructors and destructor
//
DemoAnalyzer::DemoAnalyzer(const edm::ParameterSet& iConfig)

{
   //now do what ever initialization is needed
   usesResource("TFileService");

}


DemoAnalyzer::~DemoAnalyzer()
{

   // do anything here that needs to be done at desctruction time
   // (e.g. close files, deallocate resources etc.)

}
~~~
{: .language-cpp}

Note that a `ParameterSet` object is passed to the constructor.  This is then the place where we will read any configuration we might end up implementing through our `Demo/DemoAnalyzer/python/ConfFile_cfg.py` python configuration file.

> ## Registering for data access
> Before a module, or a class helper delegated by a module, can access data it must have registered with the framework ahead of the time that it will be making a
> data access request. This registration must happen in the module's constructor. Registration is accomplished by changing the constructor to
>
> ~~~
> //
> DemoAnalyzer::DemoAnalyzer(const edm::ParameterSet& iConfig)
> 
> {
>    //now do what ever initialization is needed
>    usesResource("TFileService");
> 
>    muonToken_ = consumes<pat::MuonCollection>(edm::InputTag("slimmedMuons"));
> 
> 
> }
> ~~~
> {: .language-cpp}
{: .challenge}

Do not mind the rather complicated way in which the token objects are handled.  The good thing is that the same structure repeats for almost 
any object you need from the event.  The `slimmedMuons` tag refers to the collection stored inside the miniAOD ROOT files.  There could be more than one collection for a given physical object as it can be seen in this [table](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookMiniAOD2015#High_level_physics_objects).  This will be important to remember when we make this piece configurable.

The heart of the source file is the `analyze` method:

~~~
// ------------ method called for each event  ------------
void
DemoAnalyzer::analyze(const edm::Event& iEvent, const edm::EventSetup& iSetup)
{
   using namespace edm;



#ifdef THIS_IS_AN_EVENT_EXAMPLE
   Handle<ExampleData> pIn;
   iEvent.getByLabel("example",pIn);
#endif

#ifdef THIS_IS_AN_EVENTSETUP_EXAMPLE
   ESHandle<SetupData> pSetup;
   iSetup.get<SetupRecord>().get(pSetup);
#endif
}

~~~
{: .language-cpp}

Anything that goes inside this routine will loop over all available events.  The CMSSW Framework will take care of that, so you do not really have to write a `for` loop to go over all events.  Note that an `edm::Event` object and a `edm::EventSetup` object are passed by default.  While from the Event we can extract information like physics objects, from the EventSetup we can get information like trigger prescales.

> ## Get the muons energy
>
> Now let's add a few lines in the analyzer so we can retrieve the energy of all the muons in each event.  We will print out this information as an example. Again, after checking out [this guide](https://cms-opendata-guide.web.cern.ch/analysis/selection/objects/objects/#access-methods), the analyze method becomes:
>
> ~~~
> // ------------ method called for each event  ------------
> void
> DemoAnalyzer::analyze(const edm::Event& iEvent, const edm::EventSetup& iSetup)
> {
>    using namespace edm;
> 
>    //clean the container
>    muon_e.clear();
> 
>    //define the handler, a token and get the information by token
>    Handle<pat::MuonCollection> mymuons;
>    iEvent.getByToken(muonToken_, mymuons);
> 
>    //if collection is valid, loop over muons in event
>    if(mymuons.isValid()){
>       for (const pat::Muon &itmuon : *mymuons){
>           muon_e.push_back(itmuon.energy());
>       }
>    }
> 
>    //print the vector of muons for each event
>    for(unsigned int i=0; i < muon_e.size(); i++){
>       std::cout <<"Muon # "<<i<<" with E = "<<muon_e.at(i)<<" GeV."<<std::endl;
>    }
> 
> #ifdef THIS_IS_AN_EVENT_EXAMPLE
>    Handle<ExampleData> pIn;
>    iEvent.getByLabel("example",pIn);
> #endif
> 
> #ifdef THIS_IS_AN_EVENTSETUP_EXAMPLE
>    ESHandle<SetupData> pSetup;
>    iSetup.get<SetupRecord>().get(pSetup);
> #endif
> }
>
> ~~~
> {: .language-cpp}
{: .challenge}



The other methods are designed to execute instructions according to their own name description.  
~~~
// ------------ method called once each job just before starting event loop  ------------
void
DemoAnalyzer::beginJob()
{
}

// ------------ method called once each job just after ending the event loop  ------------
void
DemoAnalyzer::endJob()
{
}

~~~
{: .language-cpp}

For instance, any instructions placed inside the `beginJob` or `endJob` routine will be executed every time the Framework starts or ends a 
computing job, respectively.  One may also use the `beginRun` and `endRun` routines to, for example, execute code that needs to be run at the
beginning of each CMS Run, like the trigger routines.

Now, it is time to compile. Remember that while editing is done on your local computer, compiling and running must be done in the container. So let's move to the container shell and compile (heads-up: it will fail; see below)

~~~
scram b
~~~
{: .language-bash}

> ## Work assignment
>
> The compilation will invariably fail.  This is because the Muon class we added introduced some dependencies that need to be taken care of in the `BuildFile.xml`.  We will deal with this in a moment.  Now, however, it is a good time to submit your assignment for this lesson.  One of the assignments is to copy the compilation error message you got and paste it to the corresponding section in our [assignment form](https://forms.gle/7YYRv6ZCTfRYiocr7); remember you must sign in and <strong style="color: red;">click on the submit button</strong> in order to save your work.  You can go back to edit the form at any time.
{: .challenge}


<!--
> ## Let's compile
>  
> ~~~
> scram b
> ~~~
> {: .language-bash}
>
> > ## Look at the output
> >
> > Well, it fails.
> >
> > ~~~
> > >> Local Products Rules ..... started
> > >> Local Products Rules ..... done
> > >> Building CMSSW version CMSSW_5_3_32 ----
> > >> Entering Package Demo/DemoAnalyzer
> > >> Creating project symlinks
> >   src/Demo/DemoAnalyzer/python -> python/Demo/DemoAnalyzer
> > >> Compiling edm plugin /home/cmsusr/CMSSW_5_3_32/src/Demo/DemoAnalyzer/src/DemoAnalyzer.cc
> > In file included from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/TrackingRecHit/interface/TrackingRecHit.h:4:0,
> >                  from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/TrackingRecHit/interface/RecSegment.h:17,
> >                  from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/DTRecHit/interface/DTRecSegment4D.h:16,
> >                  from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/DTRecHit/interface/DTRecSegment4DCollection.h:20,
> >                  from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/MuonReco/interface/MuonSegmentMatch.h:6,
> >                  from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/MuonReco/interface/MuonChamberMatch.h:5,
> >                  from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/MuonReco/interface/Muon.h:17,
> >                  from /home/cmsusr/CMSSW_5_3_32/src/Demo/DemoAnalyzer/src/DemoAnalyzer.cc:34:
> > /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/CLHEP/interface/AlgebraicObjects.h:8:33: fatal error: CLHEP/Matrix/Vector.h: No such file or directory
> > compilation terminated.
> > In file included from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/TrackingRecHit/interface/TrackingRecHit.h:4:0,
> >                  from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/TrackingRecHit/interface/RecSegment.h:17,
> >                  from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/DTRecHit/interface/DTRecSegment4D.h:16,
> >                  from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/DTRecHit/interface/DTRecSegment4DCollection.h:20,
> >                  from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/MuonReco/interface/MuonSegmentMatch.h:6,
> >                  from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/MuonReco/interface/MuonChamberMatch.h:5,
> >                  from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/MuonReco/interface/Muon.h:17,
> >                  from /home/cmsusr/CMSSW_5_3_32/src/Demo/DemoAnalyzer/src/DemoAnalyzer.cc:34:
> > /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/CLHEP/interface/AlgebraicObjects.h:8:33: fatal error: CLHEP/Matrix/Vector.h: No such file or directory
> > compilation terminated.
> > gmake: *** [tmp/slc6_amd64_gcc472/src/Demo/DemoAnalyzer/src/DemoDemoAnalyzer/DemoAnalyzer.o] Error 1
> > gmake: *** [There are compilation/build errors. Please see the detail log above.] Error 2
> > ~~~
> > {: .error}
> > This is because the Muon classes we added introduced some dependencies that need to be taken care of in the `BuildFile.xml`
> {: .solution}
{: .challenge}
-->

So let's modify the `Demo/DemoAnalyzer/BuildFile.xml` to include `DataFormats/PatCandidates` dependencies.  It should look like:

~~~
<use name="FWCore/Framework"/>
<use name="FWCore/PluginManager"/>
<use name="FWCore/ParameterSet"/>
<use name="DataFormats/PatCandidates"/>
<flags EDM_PLUGIN="1"/>
~~~
{: .language-xml}

Now, if you **compile again**, it should work.  Then, we can run with the `cmsRun` executable:

~~~
cmsRun Demo/DemoAnalyzer/python/ConfFile_cfg.py > mylog.log 2>&1 &
~~~
{: .language-bash}

Let's check the log file:

~~~
cat mylog.log
~~~
{: .language-bash}

~~~
14-Jul-2022 17:22:35 CEST  Successfully opened file root://eospublic.cern.ch//eos/opendata/cms/Run2015D/SingleElectron/MINIAOD/08Jun2016-v1/10000/001A703B-B52E-E611-BA13-0025905A60B6.root
(/code/CMSSW_7_6_7/src) cat mylog.log 
14-Jul-2022 17:22:32 CEST  Initiating request to open file root://eospublic.cern.ch//eos/opendata/cms/Run2015D/SingleElectron/MINIAOD/08Jun2016-v1/10000/001A703B-B52E-E611-BA13-0025905A60B6.root
%MSG-w XrdAdaptor:  file_open 14-Jul-2022 17:22:33 CEST pre-events
Data is served from cern.ch instead of original site eospublic
%MSG
14-Jul-2022 17:22:35 CEST  Successfully opened file root://eospublic.cern.ch//eos/opendata/cms/Run2015D/SingleElectron/MINIAOD/08Jun2016-v1/10000/001A703B-B52E-E611-BA13-0025905A60B6.root
Begin processing the 1st record. Run 257645, Event 1184198851, LumiSection 776 at 14-Jul-2022 17:22:37.137 CEST
Begin processing the 2nd record. Run 257645, Event 1184202760, LumiSection 776 at 14-Jul-2022 17:22:39.003 CEST
Begin processing the 3rd record. Run 257645, Event 1183968519, LumiSection 776 at 14-Jul-2022 17:22:39.003 CEST
Muon # 0 with E = 15.2976 GeV.
Begin processing the 4th record. Run 257645, Event 1183964627, LumiSection 776 at 14-Jul-2022 17:22:39.014 CEST
Muon # 0 with E = 6.0168 GeV.
Begin processing the 5th record. Run 257645, Event 1184761030, LumiSection 776 at 14-Jul-2022 17:22:39.014 CEST
Begin processing the 6th record. Run 257645, Event 1184269130, LumiSection 776 at 14-Jul-2022 17:22:39.014 CEST
Muon # 0 with E = 8.94318 GeV.
Begin processing the 7th record. Run 257645, Event 1184358918, LumiSection 776 at 14-Jul-2022 17:22:39.014 CEST
Muon # 0 with E = 4.05406 GeV.
Begin processing the 8th record. Run 257645, Event 1183874827, LumiSection 776 at 14-Jul-2022 17:22:39.015 CEST
Muon # 0 with E = 4.2305 GeV.
Begin processing the 9th record. Run 257645, Event 1184415529, LumiSection 776 at 14-Jul-2022 17:22:39.015 CEST
Muon # 0 with E = 8.99778 GeV.
Begin processing the 10th record. Run 257645, Event 1184425291, LumiSection 776 at 14-Jul-2022 17:22:39.015 CEST
Muon # 0 with E = 8.0897 GeV.
Muon # 1 with E = 4.36828 GeV.
14-Jul-2022 17:22:39 CEST  Closed file root://eospublic.cern.ch//eos/opendata/cms/Run2015D/SingleElectron/MINIAOD/08Jun2016-v1/10000/001A703B-B52E-E611-BA13-0025905A60B6.root

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

Interestingly, as one can see, there are muons in a `SingleElectron` dataset.  This is, of course, expected.

{% include links.md %}
