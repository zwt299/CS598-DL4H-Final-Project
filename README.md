# CS598-DL4H-Final-Project

Author: Zachary Thurston (zwt3@illinois.edu)

To reproduce this paper please follow below steps:

1. Please take a look at https://github.com/ratschlab/mmugl?tab=readme-ov-file

2. Obtain access to MIMIC III through physionet and UMLS by requesting a liscense from NIH (National Library of Medicine)

3. Upon obtaining access to MIMIC III AND UMLS please download according to each section.
    MIMIC III - download the below files
    - ADMISSIONS.csv
    - DIAGNOSIS_ICD.csv
    - NOTEEVENTS.csv
    - PRESCRIPTIONS.csv

    Please place them all under a folder - say MIMIC_III/

    UMLS - From the UMLS Knowledge Sources: File Downloads Page download the
    20XXYY UMLS Metathesaurus Full Subset. For this project I used 2025AA, and even though the original paper used an older version of UMLS, I had no issues running most of the scripts.
    
    From the UMLS full subset we are concerned with the following files:
    - MRCONSO.RRF
    - MRDEF.RRF
    - MRHIER.RRF
    - MRREL.RRF
    - MRSAT.RRF
    - MRSTY.RRF

4. Please also download a zipfile of the repository for MMUGL - as the scripts will be used to run concept extraction and training. Extract the zipfile and setup a conda environment using 

`mamba env create -f conda_linux_x86_environment.yml`

This should be done on a linux device that supports x86. 

One note is that this may cause issues on initial environment create command due to some deprecated packages.
I was able to use the LLM to identify any suitable replacement package.

In particular there should be a line in the conda environment file
` - torchvision==0.2.2=py_3`
Which is no longer supported. The LLM suggested to remove that line, install the environment and activate it.
Then run:
`conda install pytorch=1.9.0 torchvision=0.10.0 cpuonly -c pytorch` which should work.

Followed by:
`pip install -e ./kg` to install the custom knowledge graph code package.


5. After activating your environment there is still some setup that needs to be done.
Download the files from GAMENet: https://github.com/sjy1203/GAMENet/tree/master/data
`ndc2atc_level4.csv` and `ndc2rxnorm_mapping.txt` and create a code_mappings folder to located these files within the data folder from the file hierarchy used by MMUGL.

```
> tree ./data
├── cgl_mimic3_disease_codes.txt
├── cgl_patient_splits.pkl
├── code_mappings
│   ├── ndc2atc_level4.csv
│   └── ndc2rxnorm_mapping.txt
├── disease_codes_all.txt
└── med_codes_all.txt
```

6. After downloading all the files, it is necessary to setup QuickUMLS which is supposed to do the concept extraction from the UMLS data. https://github.com/Georgetown-IR-Lab/QuickUMLS - follow the steps listed verbatim and set the destination folder to be something like Quick_UMLS, as this will be a parameter to be used in the invokation of training and extraction scripts. This takes somewhere around 10-30 minutes to run.

7. A manual fix is needed to the file under `~/miniconda3/envs/mmugl/lib/python3.8/site-packages/quickumls/core.py`
where around line 150 there is a line `self.nlp = spacy.load(spacy_lang)` that should be replaced with 
```
if spacy_lang == "en":
    spacy_lang = "en_core_web_sm"
self.nlp = spacy.load(spacy_lang)
```

8. Another manual fix is needed to fix an outdated API for quickumls.
In `kg/kg/data/quickumls.py` update to the line 
`matches = self.quickumls._match(doc, best_match=True, ignore_syntax=False)`
To be 
`matches = self.quickumls.match(doc.text, best_match=True, ignore_syntax=False)`

9. In `kg/kg/data/vocabulary.py` change the line 86 from 
`return list(map(lambda token: self.vocabulary.word2idx[token], tokens))`
to
`return list(map(lambda token: self.vocabulary.word2idx.get(token, self.vocabulary.word2idx.get("<unk>", 0)), tokens))`
This prevents exceptions from being thrown in the case that you don't use the full UMLS dataset (which I did not due to time constraints).

10. Congrats! Those are the manual fixes that need to be done. Please defer to looking at `training-results/{size}/commands_run.txt` to see the commands and corresponding parameters that I used to invoke `text_concept_extraction.py`, `umls_graph_extraction.py`, and `run_training_concepts.py`. If looking to replicate, please run the commands in the order provided in the textfile.

Running the first two scripts does some mandatory preprocessing steps to perform concept extraction and graph extraction. Data preprocessing is done within the (third) main training script. Evaluation metrics are also computed within the last script and are dumped in the terminal.

I have attached a file under train-results subfolders called `training_and_eval_log.txt` which shows what a run for the third script looks like.


NEW FUNCTIONALITY:
One can run the file in similar format to the commands_run for `run_training_concepts.py` but instead use `run_training_concepts_focal.py` with param `--loss focal` to try a new loss function implementation.

I have included results from a sample run with the same configs as the fifty thousand notes run, but using the new script and loss param to set focal loss. Those results are under `training_and_eval_log_focal.txt`


---------------------------------

LLM Usage: 
In summary the project wasn't too difficult nor too easy to setup.
ChatGPT 4o was able to help me expedite changes that I needed to make to patch old files, as well as a small update that was causing exceptions when running the training script. So rather than utilize LLMs to do preprocessing of data, I utilized them to fix any issues I encountered in running the already provided data preprocessing scripts.

--------------------------------

Note: For the repository I will be excluding the data files due to non-distribute concerns. Likewise I will not be including code from the original authors Multi-modal Graph Learning over UMLS Knowledge Graphs so I do not infringe on their intellectual property/research. Files that are included are mainly results achieved from running training scripts.



-----------
Citation to original paper:

@InProceedings{pmlr-v225-burger23a,
  title = 	 {Multi-modal Graph Learning over UMLS Knowledge Graphs},
  author =       {Burger, Manuel and R\"atsch, Gunnar and Kuznetsova, Rita},
  booktitle = 	 {Proceedings of the 3rd Machine Learning for Health Symposium},
  pages = 	 {52--81},
  year = 	 {2023},
  editor = 	 {Hegselmann, Stefan and Parziale, Antonio and Shanmugam, Divya and Tang, Shengpu and Asiedu, Mercy Nyamewaa and Chang, Serina and Hartvigsen, Tom and Singh, Harvineet},
  volume = 	 {225},
  series = 	 {Proceedings of Machine Learning Research},
  month = 	 {10 Dec},
  publisher =    {PMLR},
  pdf = 	 {https://proceedings.mlr.press/v225/burger23a/burger23a.pdf},
  url = 	 {https://proceedings.mlr.press/v225/burger23a.html}
}