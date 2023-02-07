# Table of Contents
1. [Executing the Case Studies](#executing-the-case-studies)
2. [How does the AEAD library work?](#aead-library)
4. [Modifying the Case Studies](#case-study-modification)

----------------------


## Executing the Case Studies

The case studies in this folder are different models for the following protocols:

- Whatsapp Groups
- Facebook Message Franking mechanism (2 versions)
- SFrame
- YubiHSM
- Webpush
- GPG
- Scuttlebot
- SaltPack

For more details on the protocols themselves, refer to the original paper [Anonymized]

It further contains the `AEADlibrary.splib` file with all AEAD models
presented in the paper.

To execute all case studies with all AEAD models please use

`python3 tamarin_wrapper.py -f case_studies.tamjson`

which will create a results folder with the results as .csv files.

Depending on the machine used, this can take several hours.

#### Dependencies

To install all dependencies on Ubuntu for the `tamarin_wrapper.py` file, run

```
apt-get install python3
apt-get install python3-pip
pip3 install tabulate matplotlib
```


-----------------

## AEAD Library

To ease applicability of our new aead models, we
use a library file
`AEADlibrary.splib`
.

This file needs to be included into the model file
of the case study. To do this add 
`#include "AEADlibrary.splib"` to your file.

To choose the different aead and threat models,
we use the tamarin preprocesser flags.

This is done by adding 
`-D=FLAG`
to you Tamarin query for each model you want to look at.

As an example

```tamarin-prover --prove test.spthy -D=n_reuse_1 -D=reveal_nad```

adds the capability to the attacker to compute the key if 
the same key and nonce were used to encrypt some message.
Additionally, the attacker is able to extract the header
and nonce from the ciphertext.

A full list of flags can be found here compared to their name
in the paper:

### Flags:

**Collision Resistance**

collkeys : (`KeysColl`)

collkey : (`KeyColl`)

collkeymax : (`FullKeyColl`)

colln : (`nColl`)

collnmax : (`Full-nColl`)

collmmax : (`Full-mColl`)

colladmax : (`Full-adColl`)

**Tags (seperate cipher texts and MAC/tags)**

tag : activate tags function symbol. It is needed to use the other tag models.

tagkeys : (`KeysTag`)

tagkey : (`KeyTag`)

tagkeymax : (`FullKeyTag`)

tagn : (`nTag`)

tagnmax : (`Full-nTag`)

tagmmax : (`Full-mTag`)

tagadmax : (`Full-adTag`)

**Nonce reuse**

n_reuse_1 :  (`k-NR`)

n_reuse_2 :  (`m-NR`) 

**Commiting AEADs**

commit : (`Com`)

**Weak decryption**

forge: (`Forge`)

**Leakage**

keyleak : leaks encryption key & is mainly for testing purposes (not in the Paper)

reveal_nad : ciphertext leaks nonce & header (over-approximation of `Leak`)


---------


## Case Study Modification

There are several ways to modify the case studies or run only parts of it:

- changing the tamjson file to run another set of case studies
- changing the `flags.json` file to change the set of possible AEAD models used on the case studies
- modifying the `AEADlibrary.splib` by adding new preprocesser flags or new models in general. See Tamarin documentation for details on syntax and semantics.

### Tamjson

With the .tamjson file we define what models under which conditions should be executed.
It is a .json file with specfic keys.

- "executable":  Define the executable of your Tamarin instance as a string. Default: `tamarin-prover`
- "timeout": Define the timeout for a single Tamarin query as an integer
- "silent": Boolean. Decides if tamarin runs should be shwon in the shell while running
- "cores": Define the number of cores to use as an int
- "tamcommand": string of tamarin specific flags like "--auto-sources" or heuristics
- PROTOCOL_NAME: gets another dict as an input with the optional keys "lemmas" and "fixed_flags"
- "lemmas": List of lemmas to execute. if empty (or non-existend), all lemmas will be executed
- "fixed_flags": preprocesser flags that need to be set on each run of the specific protocol. Not needed if empty.
- "flags": Path to the flag file used. See [Flags](#flags)

Example:

```json
{
    "timeout": 60,
    "silent": true,
    "cores": 4,
    "models" : ["whatsapp.spthy",
                "webpush_noserver.spthy",
                "sframe.spthy"],
    "whatsapp.spthy" : {
      "lemmas" : ["consistency"],
      "fixed_flags" : ["Dishonest"],
      "flags" : "flags.json"
      },
    "sframe.spthy" : {
      "lemmas" : ["Authentication"],
      "fixed_flags" : ["tag"],
      "flags" : "flags.json"
      } 
}
```

### Flags

- "restrictions": list of lists. Should include all flags/models that should be considered in the analysis. Unary lists are standard. If models are mutally execlusive  they are put in the same list.
- "priority": List of priority model combinations that should be explored first. Can help the pruning strategy to execute less total model combinations.
- "orders": list of lists of implications. Proofs using the left side of inner list imply proofs of the right side.

```json
{
"restrictions": [["collkeys","collkey","collkeymax"],["reveal_nad"]],
"priority": [["collkeymax"],["collkey","reveal_nad"]],
"orders": [["collkeymax","collkey"],["collkey","collkeys"],["collkeymax","collkeys"]]
}
```
