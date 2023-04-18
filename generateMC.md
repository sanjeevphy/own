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
  - Crab job submission sample file:
  - ```python
	from CRABClient.UserUtilities import config
	config = config()

	#config.section_('General')
	config.General.requestName = 'bsTo4Mu_RunIISummer20UL18GENSIMDIGIRAW_v1'
	config.General.workArea = 'crab_projects'
	config.General.transferOutputs = True
	config.General.transferLogs = True

	#config.section_('JobType')
	config.JobType.pluginName = 'PrivateMC'
	config.JobType.psetName = 'BPH-Run3Summer22EEGS-00017_test.py'
	config.Data.outputPrimaryDataset = 'BsTo4Mu-pythia8-evtgen'

	config.JobType.allowUndistributedCMSSW = True
	config.JobType.maxMemoryMB = 4000
	config.Data.splitting = 'EventBased'
	config.Data.unitsPerJob = 60000
	NJOBS = 2000
	config.Data.totalUnits = config.Data.unitsPerJob * NJOBS

	config.Data.outLFNDirBase = '/store/user/sakumar/bs4mu/privateSamples/Run3/'
	config.Site.storageSite   = 'T2_IN_TIFR'
	config.Data.publication   = True
    ```
  - Output can be found on DBS page by searching `/*/username*/USER`
## DIGI,DATAMIX,L1,DIGI2RAW,HLT,RAW2DIGI,L1Reco,RECO,RECOSIM
[Commands can be found at mcm page.](https://cms-pdmv.cern.ch/mcm/public/restapi/requests/get_setup/BPH-Run3Summer22EEDRPremix-00015)  

### Get the cms configuration (DIGI,DATAMIX,L1,DIGI2RAW,HLT)
  - Configuration file
  - ```bash
    cmsDriver.py  \
        --python_filename BPH-Run3Summer22EEDRPremix-00015_1_cfg.py \
        --eventcontent PREMIXRAW \
        --customise Configuration/DataProcessing/Utils.addMonitoring \
        --datatier GEN-SIM-RAW \
        --fileout file:BPH-Run3Summer22EEDRPremix-00015_0.root \
        --pileup_input "dbs:/Neutrino_E-10_gun/Run3Summer21PrePremix-Summer22_124X_mcRun3_2022_realistic_v11-v2/PREMIX" \
        --conditions 124X_mcRun3_2022_realistic_postEE_v1 \
        --step DIGI,DATAMIX,L1,DIGI2RAW,HLT:2022v14 \
        --procModifiers premix_stage2,siPixelQualityRawToDigi \
        --geometry DB:Extended \
        --filein file:BPH-Run3Summer22EEGS-00017.root \
        --datamix PreMix \
        --era Run3 \
        --no_exec \
        --mc -n $EVENTS
    ```

### Get the cms configuration (RAW2DIGI,L1Reco,RECO,RECOSIM)
  - Configuration file
  - ```bash
    cmsDriver.py  \
        --python_filename BPH-Run3Summer22EEDRPremix-00015_2_cfg.py \
        --eventcontent AODSIM \
        --customise Configuration/DataProcessing/Utils.addMonitoring \
        --datatier AODSIM \
        --fileout file:BPH-Run3Summer22EEDRPremix-00015.root \
        --conditions 124X_mcRun3_2022_realistic_postEE_v1 \
        --step RAW2DIGI,L1Reco,RECO,RECOSIM \
        --procModifiers siPixelQualityRawToDigi \
        --geometry DB:Extended \
        --filein file:BPH-Run3Summer22EEDRPremix-00015_0.root \
        --era Run3 \
        --no_exec \
        --mc -n $EVENTS
    ```

### Get the cms configuration file for AODSIM (mixing of above two)
  - ```bash
    cmsDriver.py  \
        --python_filename BPH-Run3Summer22EE_genSimToAOD_noPU_cfg.py \
        --eventcontent AODSIM \
        --customise Configuration/DataProcessing/Utils.addMonitoring \
        --datatier AODSIM \
        --fileout file:BPH-Run3Summer22EE-00015_0.root \
        --conditions 124X_mcRun3_2022_realistic_postEE_v1 \
        --step DIGI,L1,DIGI2RAW,HLT:2022v14,RAW2DIGI,L1Reco,RECO,RECOSIM \
        --procModifiers siPixelQualityRawToDigi \
        --geometry DB:Extended \
        --filein file:BPH-Run3Summer22EEGS-00017.root \
        --era Run3 \
        --no_exec \
        --mc -n 100
    ```
  - Sample crab submission file:
  - ```python
	from CRABClient.UserUtilities import config
	config = config()

	#config.section_('General')
	config.General.requestName = 'bsTo4Mu_RunIISummer22AOD_v1_15Apr23'
	config.General.workArea = 'crab_projects'
	config.General.transferOutputs = True
	config.General.transferLogs = True

	#config.section_('JobType')
	config.JobType.pluginName = 'Analysis'
	config.JobType.psetName = 'BPH-Run3Summer22EE_genSimToAOD_cfg.py'

	config.Data.inputDBS = 'phys03'
	config.JobType.allowUndistributedCMSSW = True
	config.JobType.maxMemoryMB = 3000
	#config.JobType.numCores = 8
	config.Data.inputDataset ='/BsTo4Mu-pythia8-evtgen/sakumar-crab_bsTo4Mu_RunIISummer20UL18GENSIMDIGIRAW_v1-a5933a4d0222a7203b437841532fd2cd/USER'
	config.Data.splitting = 'FileBased'
	config.Data.unitsPerJob = 1

	config.Data.outLFNDirBase = '/store/user/sakumar/bs4mu/privateSamples/Run3/'
	config.Site.storageSite   = 'T2_IN_TIFR'
	config.Data.publication   = True
    ```

## MiniAOD
### Get the cms configuration file for miniAODSIM
  - [McM page for MiniAOD](https://cms-pdmv.cern.ch/mcm/public/restapi/requests/get_setup/BPH-Run3Summer22EEMiniAODv3-00044)  
  - ```bash
    cmsDriver.py  \
        --python_filename BPH-Run3Summer22EEMiniAODv3-00044_1_cfg.py \
        --eventcontent MINIAODSIM \
        --customise Configuration/DataProcessing/Utils.addMonitoring \
        --datatier MINIAODSIM \
        --fileout file:BPH-Run3Summer22EEMiniAODv3-00044.root \
        --conditions 124X_mcRun3_2022_realistic_postEE_v1 \
        --step PAT \
        --geometry DB:Extended \
        --filein "dbs:/BsTo4Mu-pythia8-evtgen/sakumar-crab_bsTo4Mu_RunIISummer22AOD_v1_15Apr23-d816354374bd25c19c12b7ad61fa46ff/USER" \
        --era Run3 \
        --no_exec \
        --mc -n 100
    ```
  - For interactive run, we may need to add input file manually. For submitting crab jobs, this file is fine and you will find output in database.
  - Crab jobs can be submitted by writting crab file as it is written in previous steps.
## FWlite code or EdAnalyzer need to be written for further steps.
