# Monte Carlo Production tools
* [Introduction of tool](https://cms-pdmv.gitbook.io/)
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
