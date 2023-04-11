# Monte Carlo Production tools
* [Introduction of tool](https://cms-pdmv.gitbook.io/ "detailed website")
* [Group Analysis Samples Page: GrASP](https://cms-pdmv.gitbook.io/project/group-analysis-samples-page-grasp)
## Documentations available
- [CMS Twiki : Thirty-Minute Introduction to Generation and Simulation ](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookGenIntro)
- [Mcm Page : CMS Driver Commands ](https://cms-pdmv.gitbook.io/project/cmsdriver-argument-and-meaning)
	- Has brief information of different steps in MC production
	- Has info on nomenclature used in the workflow
## Seeting up the fragments for the decay
 - For this step, we have to go to [mcm grasp page](https://cms-pdmv.cern.ch/grasp/)
 - Click on required Campaigns, here I am clicking on [BPH Run3Summer22*GS](https://cms-pdmv.cern.ch/grasp/samples?campaign=Run3Summer22*GS&pwgs=BPH)
 - New page will open, in the table, click on suitable root request. I am clicking on [BdToK0starEE McM pMp page](https://cms-pdmv.cern.ch/pmp/historical?r=BPH-Run3Summer22EEGS-00017).
 - Fragments are the
 - ```bash
   export SCRAM_ARCH=el8_amd64_gcc10
   cmsrel CMSSW_12_4_11_patch3
   cd CMSSW_12_4_11_patch3/src
   cmsenv
   mkdir -p Configuration/GenProduction/python
   mkdir -p Configuration/GenProduction/data
   cp <YOUR DECAY FILE> Configuration/GenProduction/data
   
   # Download a fragment template
   curl -s -k https://cms-pdmv.cern.ch/mcm/public/restapi/requests/get_fragment/BPH-Run3Summer22EEGS-00017 \
           --retry 3 -o fragment.py
   # Edit the fragment to add your decay file and
   cp fragment.py Configuration/GenProduction/python
   scram b
   ```
 - Parts to be edited in the fragment
 - ```py
   ExternalDecays = cms.PSet(
       list_forced_decays = cms.vstring('MyB_s0', 'Myanti-B_s0'),
       operates_on_particles = cms.vint32(),
       convertPythiaCodes = cms.untracked.bool(False),
       user_decay_embedded= cms.vstring(
       'Alias      MyB_s0        B_s0',
       'Alias      Myanti-B_s0   anti-B_s0',
       'ChargeConj MyB_s0        Myanti-B_s0',
       '#',
       'Decay MyB_s0',
       '  1.000        mu+     mu-    mu+   mu-      PHOTOS PHSP',
       'Enddecay',
       'Decay Myanti-B0',
       'End',
   ),
   bfilter = cms.EDFilter("PythiaFilter",
       MaxEta = cms.untracked.double(9999.0),
       MinEta = cms.untracked.double(-9999.0),
       ParticleID = cms.untracked.int32(531)
   )
   decayfilter = cms.EDFilter("PythiaDauVFilter",
       DaughterIDs = cms.untracked.vint32(-13, 13, -13, 13),
       MaxEta = cms.untracked.vdouble( 2.5,  2.5,  2.5, 2.5),
       MinEta = cms.untracked.vdouble(-2.5, -2.5, -2.5, 2.5),
       MinPt = cms.untracked.vdouble(3.0, 3.0, 3.0, 3.0),
       NumberDaughters = cms.untracked.int32(4),
       ParticleID = cms.untracked.int32(531),
       verbose = cms.untracked.int32(1)
   )
   ```
  - Remove `kstarfilter`
## GEN-SIM Step
### Obtain the configuration file for Production
  - This command can produce a configuration file for _cmsRun_  that could be used for producing the MC samples
  - ```bash
    cmsDriver.py Configuration/GenProduction/python/fragment.py \
   		--python_filename BPH-Run3Summer22EEGS-00017_1_cfg.py \
		--eventcontent RAWSIM \
		--customise Configuration/DataProcessing/Utils.addMonitoring \
		--datatier GEN-SIM \
		--fileout file:BPH-Run3Summer22EEGS-00017.root \
		--conditions 124X_mcRun3_2022_realistic_postEE_v1 \
		--beamspot Realistic25ns13p6TeVEarly2022Collision \
		--step GEN,SIM \
		--geometry DB:Extended \
		--era Run3 \
		--no_exec --mc -n 100
    ```
    
