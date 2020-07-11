---
title: Using crossref API to parse citations
date: 2020-05-15 00:00:00 -0700
categories: [Tutorial]
tags: [crossref]
toc: true
---

## Convert DOI to any citation format using the crossref API

* Python script to convert a DOI into specific citation style:

```python
from yaml import load, dump
import urllib.request
from urllib.error import HTTPError
import time

def build_formatted_citation(doi_url, style='mla', get_json=False, verbose=True):
  BASE_URL = 'https://doi.org/'
  bibtex = None

  if len(doi_url) == 0:
    return None

  if not doi_url.startswith('http'):
    doi_url = BASE_URL + doi_url 
  
  if BASE_URL not in doi_url:
    raise ValueError("Incorrect doi_url: {}! It might starts with {}".format(
        doi_url, BASE_URL))

  req = urllib.request.Request(doi_url)

  # This is the latex style
  #args = 'application/x-bibtex'
  
  if get_json:
    args = "application/citeproc+json"
  else:
    args = "text/bibliography; style={style}".format(style=style)

  req.add_header('Accept', args)
  
  try:
      with urllib.request.urlopen(req) as f:
          bibtex = f.read().decode('utf-8')      
  except HTTPError as e:
      if e.code == 404:
          print('DOI not found: {}'.format(doil_url))
      else:
          print('Service unavailable.')

  if verbose:
    print(bibtex)
  
  return bibtex
 
```

**1. Retrieve citation in MLA style**

For more citation styles, see http://api.crossref.org/styles

```python
doi_url = "https://doi.org/10.1073/pnas.1811971115"
refs = build_formatted_citation(doi_url, style='mla', get_json=False, verbose=False)
```

This returns

```
Brunk, Elizabeth et al. “Characterizing Posttranslational Modifications in Prokaryotic Metabolism Using a Multiscale Workflow.” Proceedings of the National Academy of Sciences 115.43 (2018): 11096–11101.
```

**2. Retrieve citation in JSON/dictionary format**
```python
import json
doi_url = "https://doi.org/10.1073/pnas.1811971115"
refs = build_formatted_citation(doi_url, style='', get_json=True, verbose=False)
data = json.loads(refs)
```

This returns a large dictionary of all the information about the paper. A subset of that is shown below:
```
{'URL': 'http://dx.doi.org/10.1073/pnas.1811971115',
 'author': [{'ORCID': 'http://orcid.org/0000-0001-8578-8658',
   'affiliation': [],
   'authenticated-orcid': False,
   'family': 'Brunk',
   'given': 'Elizabeth',
   'sequence': 'first'},
  {'affiliation': [],
   'family': 'Chang',
   'given': 'Roger L.',
   'sequence': 'additional'},
  {'affiliation': [],
   'family': 'Xia',
   'given': 'Jing',
   'sequence': 'additional'},
  {'affiliation': [],
   'family': 'Hefzi',
   'given': 'Hooman',
   'sequence': 'additional'},
  {'ORCID': 'http://orcid.org/0000-0002-9403-509X',
   'affiliation': [],
   'authenticated-orcid': False,
   'family': 'Yurkovich',
   'given': 'James T.',
   'sequence': 'additional'},
  {'affiliation': [],
   'family': 'Kim',
   'given': 'Donghyuk',
   'sequence': 'additional'},
  {'affiliation': [],
   'family': 'Buckmiller',
   'given': 'Evan',
   'sequence': 'additional'},
  {'ORCID': 'http://orcid.org/0000-0003-2164-4318',
   'affiliation': [],
   'authenticated-orcid': False,
   'family': 'Wang',
   'given': 'Harris H.',
   'sequence': 'additional'},
  {'affiliation': [],
   'family': 'Cho',
   'given': 'Byung-Kwan',
   'sequence': 'additional'},
  {'affiliation': [],
   'family': 'Yang',
   'given': 'Chen',
   'sequence': 'additional'},
  {'affiliation': [],
   'family': 'Palsson',
   'given': 'Bernhard O.',
   'sequence': 'additional'},
  {'affiliation': [],
   'family': 'Church',
   'given': 'George M.',
   'sequence': 'additional'},
  {'affiliation': [],
   'family': 'Lewis',
   'given': 'Nathan E.',
   'sequence': 'additional'}],
 'issue': '43',
 'publisher': 'Proceedings of the National Academy of Sciences',
 'subject': ['Multidisciplinary'],
 'title': 'Characterizing posttranslational modifications in prokaryotic metabolism using a multiscale workflow'}
```
## Create a list of citations from DOI
```python
list_of_doi = [
    'https://doi.org/10.1038/s41929-018-0212-4',
    'https://doi.org/10.1016/j.tibtech.2019.01.003',
    'https://doi.org/10.1186/s12859-019-2872-8',
    'https://doi.org/10.1093/bib/bby024'
]
for i, doi in enumerate(list_of_doi, 1):
  print(i, 
      build_formatted_citation(doi, style='bioinformatics', get_json=False, verbose=False))
```

Output:

```
1 Lee,S.Y. et al. (2019) A comprehensive metabolic map for production of bio-based chemicals. Nature Catalysis, 2, 18–33.

2 Choi,K.R. et al. (2019) Systems Metabolic Engineering Strategies: Integrating Systems and Synthetic Biology with Metabolic Engineering. Trends in Biotechnology, 37, 817–837.

3 Yu,H. and Blair,R.H. (2019) Integration of probabilistic regulatory networks into constraint-based models of metabolism with applications to Alzheimer’s disease. BMC Bioinformatics, 20.

4 Ostaszewski,M. et al. (2018) Community-driven roadmap for integrated disease maps. Briefings in Bioinformatics, 20, 659–670.
```



## Parse a YAML file containing a list of notes about each paper into text with citations
If you ever have to store information about each paper you read, I found that the YAML format is quite nice for this task. It is human-readable and can be edited using text editor like vscode. You can add notes, comments, tags, references (in the standardized DOI format) etc. I store a separate YAML file for separate categories. If you need to convert the YAML file into Word Document or any text with specific citation formats, you can use/modify the scripts below:

* Create an input file `notes.yml` to store all your notes on each paper and the DOI. For example:

```yaml
- id: 1
  doi:
    - https://doi.org/10.1038/nbt.4072
    - https://doi.org/10.1073/pnas.1811971115
  description: >
    Brunk et al. used the toolbox to update metabolite formulae and also identify a subset of 
    10,600 reactions that were used for the final model. In another paper by Brunk 
    et al., they used the COBRA toolbox to perform Markov chain Monte Carlo sampling
    of metabolic fluxes to study post-translational modifications in E. coli.
  tags:
    - human
    - modeling

- id: 2
  doi:
    - https://doi.org/10.1016/j.celrep.2019.04.001
  description: >    
    Greenhalgh et al. performed flux simulation of the interaction of human colon 
    adenocarcinoma (COAD) cells with a model probiotic Lactobacillus rhamnosus GG 
    cells using the COBRA Toolbox 3.0. By integrating in vitro data and in silico modeling,
    this study introduced a new approach for the development of treatment for 
    colorectal cancer.
  tags:
    - disease
```

* You need the Python script from the previous section and these:

```python
def convert_data_to_formatted_citations(data, outfilepath=None):
  """Convert the data dictionary to formatted citations
  """
  # Store doi temporary to check for duplication
  doi_urls_parsed = []

  for entry in data:
    if "formatted_citations" not in entry:
      formatted_citations = []
      for doi_url in entry['doi']:
        doi_url = doi_url.strip()

        if doi_url in doi_urls_parsed:
          print("Duplicated doi found: {}".format(doi_url))
        doi_urls_parsed.append(doi_url)

        # Add a wait time of 1 sec between requests
        time.sleep(1)

        ref = build_formatted_citation(doi_url, style='mla')        
        formatted_citations.append(ref)
      entry['formatted_citations'] = formatted_citations

  if outfilepath:
    with open(outfilepath, 'w') as outfile:
      dump(data, outfile, default_flow_style=False)
  return data

def print_formatted_text(data):
  for entry in data:  
    print("\n" + entry['description'].replace("  ", " "))
    for i, row in enumerate(entry['formatted_citations']):
      print("- {}".format(row))
      
```      

* Run the scripts to parse the `notes.yml` into a text format:

```python
filepath = "notes.yml"
with open(filepath, 'r') as f:
  data = load(f)
data = convert_data_to_formatted_citations(data, outfilepath=None)
print_formatted_text(data)
```

* The output:

```
Brunk et al. used the toolbox to update metabolite formulae and also identify a subset of 10,600 reactions that were used for the final model. In another paper by Brunk et al., they used the COBRA toolbox to perform Markov chain Monte Carlo sampling of metabolic fluxes to study post-translational modifications in E. coli.

- Brunk, Elizabeth et al. “Recon3D Enables a Three-Dimensional View of Gene Variation in Human Metabolism.” Nature Biotechnology 36.3 (2018): 272–281.
- Brunk, Elizabeth et al. “Characterizing Posttranslational Modifications in Prokaryotic Metabolism Using a Multiscale Workflow.” Proceedings of the National Academy of Sciences 115.43 (2018): 11096–11101.

Greenhalgh et al. performed flux simulation of the interaction of human colon adenocarcinoma (COAD) cells with a model probiotic Lactobacillus rhamnosus GG cells using the COBRA Toolbox 3.0. By integrating in vitro data and in silico modeling, this study introduced a new approach for the development of treatment for colorectal cancer.

- Greenhalgh, Kacy et al. “Integrated In Vitro and In Silico Modeling Delineates the Molecular Effects of a Synbiotic Regimen on Colorectal-Cancer-Derived Cells.” Cell Reports 27.5 (2019): 1621–1632.e9.
```
   
# Reference:
* https://citation.crosscite.org/docs.html
* https://scipython.com/blog/doi-to-bibtex/
* https://ocefpaf.github.io/python4oceanographers/blog/2014/05/19/doi2bibtex/