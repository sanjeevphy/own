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
       'CDecay Myanti-B0',
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
    
----

## NEED TO BE UPDATED Parts below

----
<!--
## DIGI-RAW Step
### Obtain the configuration file for Production
	- This command can produce a configuration file for _cmsRun_  that could be used for producing the MC samples
	- ```bash
	  cmsDriver.py Configuration/GenProduction/python/fragment.py \
	  	--python_filename genSimDigiRaw_mcProd.py \
	  	--eventcontent PREMIXRAW \
	  	--customise Configuration/DataProcessing/Utils.addMonitoring \
	  	--datatier GEN-SIM-RAW \
	  	--fileout file:BsToDStarCascade.root \
	  	--pileup_input dbs:/Neutrino_E-10_gun/RunIISummer20ULPrePremix-UL18_106X_upgrade2018_realistic_v11_L1v1-v2/PREMIX \
	  	--conditions 106X_upgrade2018_realistic_v11_L1v1 \
	  	--step GEN,SIM,DIGI,DATAMIX,L1,DIGI2RAW \
	  	--procModifiers premix_stage2 \
	  	--geometry DB:Extended \
	  	--datamix PreMix \
	  	--era Run2_2018 \
	  	--runUnscheduled \
	  	--no_exec \
	  	--beamspot Realistic25ns13TeVEarly2018Collision \
	  	--mc -n 100
	  ```
	- The  genSimDigiRaw_mcProd.py file produced could be used for crab submission
	- **One should test the file locally before submitting the file **
	- **Sample crab submission file** : please edit the storage area accordingly _outLFNDirBase_
		- ```py
		  # crabSubmissionTemplate.py
		  from CRABClient.UserUtilities import config
		  config = config()
		  
		  #config.section_('General')
		  config.General.requestName = 'bsToDsStarCascade_RunIISummer20UL18GENSIMDIGIRAW_v1'
		  config.General.workArea = 'crab_projects'
		  config.General.transferOutputs = True
		  config.General.transferLogs = True
		  
		  #config.section_('JobType')
		  config.JobType.pluginName = 'PrivateMC'
		  config.JobType.psetName = 'genSimDigiRaw_mcProd.py'
		  config.Data.outputPrimaryDataset = 'BsToDsStarSemuleptonicCascade-pythia8-evtgen'
		  
		  config.JobType.allowUndistributedCMSSW = True
		  config.JobType.maxMemoryMB = 4000
		  config.Data.splitting = 'EventBased'
		  config.Data.unitsPerJob = 25000
		  NJOBS = 2000
		  config.Data.totalUnits = config.Data.unitsPerJob * NJOBS
		  
		  config.Data.outLFNDirBase = '/store/user/athachay/bs2mmg/privateSamples/UL18/'
		  config.Site.storageSite   = 'T2_IN_TIFR'
		  config.Data.publication   = True
		  ```
- After this step the published dataset could be found out at DAS by the string
- ```
  /*/<USERNAME>*/USER
  ```
- ## HLT Step
	- ```bash
	  cmsrel CMSSW_10_2_16_UL
	  cd CMSSW_10_2_16_UL/
	  cmsenv
	  mkdir hltSim
	  cd hltSim
	  cmsDriver.py -python_filename hlt.py \
	  		--eventcontent RAWSIM \
	  		--customise Configuration/DataProcessing/Utils.addMonitoring \
	  		--datatier GEN-S    IM-RAW \
	  		--fileout file:BPH-RunIISummer20UL18HLT-00125.root \
	  		--conditions 102X_upgrade2018_realistic_v15 \
	  		--customise_commands process.source.bypassVersionCheck = cms.untracked.bool(True) \
	  		--    step HLT:2018v32 \
	  		--geometry DB:Extended \
	  		--filein file:BPH-RunIISummer20UL18DIGIPremix-00125.root \
	  		--era Run2_2018 \
	  		--no_exec \
	  		--mc -n 10
	  ```
	- Now the hlt.py produced could be used to submit jobs for HLT sim from the dataset produced in the previous step
	- A sample crab submission script is as follwing
	- ```py
	  from CRABClient.UserUtilities import config
	  config = config()
	  
	  #config.section_('General')
	  config.General.requestName = 'bs2Cascade_HLTRun2UL18_v2p2'
	  config.General.workArea = 'crab_projects'
	  config.General.transferOutputs = True
	  config.General.transferLogs = True
	  
	  #config.section_('JobType')
	  config.JobType.pluginName = 'Analysis'
	  config.JobType.psetName = 'hlt.py'
	  
	  config.Data.inputDBS = 'phys03'
	  config.JobType.allowUndistributedCMSSW = True
	  config.JobType.maxMemoryMB = 3000
	  #config.JobType.numCores = 8
	  config.Data.inputDataset ='/BsToDsStarSemuleptonicCascade-pythia8-evtgen/athachay-crab_bsToDsStarCascade_RunIISummer20UL18GENSIMDIGIRAW_v1-2f8acd8ece5e5d99ce34cf583be238df/USER'
	  config.Data.splitting = 'FileBased'
	  config.Data.unitsPerJob = 1
	  
	  config.Data.outLFNDirBase = '/store/user/athachay/BsToMuMuGamma/Data/RunIISummer20UL18/'
	  config.Data.publication = True 
	  config.Site.storageSite = 'T2_IN_TIFR'
	  
	  ```
- ### RECO Step
	- ```bash
	  cmsrel CMSSW_10_6_20/
	  cd CMSSW_10_6_20/src
	  ```
	- Get the cms configuration
	- ```bash
	  cmsDriver.py --python_filename recoStepUL2018.py \
	  		--eventcontent RECOSIM \
	  		--customise Configuration/DataProcessing/Utils.addMonitoring \
	  		--datatier GEN-SIM-RECO \
	  		--fileout file:BPH-RunIISummer20UL18RECO-00125.root \
	  		--conditions 106X_upgrade2018_realistic_v11_L1v1 \
	  		--step RAW2DIGI,L1Reco,RECO,RECOSIM,EI \
	  		--geometry DB:Extended \
	  		--filein file:BPH-RunIISummer20UL18HLT-00125.root \
	  		--era Run2_2018 \
	  		--runUnscheduled \
	  		--no_exec \
	  		--mc -n 10
	  ```
- ### MiniAOD step
	- ```bash
	  cmsrel CMSSW_10_6_20
	  cd CMSSW_10_6_20
	  cmsenv
	  ```
	- Get the cms congiguration file
	- ```bash
	  cmsDriver.py --python_filename BPH-RunIISummer20UL18MiniAODv2-00122_1_cfg.py \
	  		--eventcontent MINIAODSIM \
	  		--customise Configuration/DataProcessing/Utils.addMonitoring \
	  		--datatier MINIAODSIM \
	  		--fileout file:BPH-RunIISummer20UL18MiniAODv2-00122.root \
	  		--conditions 106X_upgrade2018_realistic_v16_L1v1 \
	  		--step PAT \
	  		--procModifiers run2_miniAOD_UL \
	  		--geometry DB:Extended \
	  		--filein dbs:/BsToMuMu_SoftQCDnonD_TuneCP5_13TeV-pythia8-evtgen/RunIISummer20UL18RECO-106X_upgrade2018_realistic_v11_L1v1-v2/AODSIM \
	  		--era Run2_2018 \
	  		--runUnscheduled \
	  		--no_exec \
	  		--mc -n 10
	  ```
	-
	-
	-->
