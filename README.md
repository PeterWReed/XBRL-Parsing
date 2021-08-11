# XBRL-Parsing
Code snips for parsing SEC Edgar database
These .py scripts are not complete programs.  They are only meant to pass along the core functionality
for parsing a company's Edgar 10-Q/K filing using BeautifulSoup.  The example below parses the XBRL LABEL LINKBASE DOCUMENT.
The result is the mapping from the XBRL tags to the common name.  The common name is used in the output, for example in Yahoo Finance
"financials".  The second snip of code demonstrates how the Python module "xmlSchema" can be used to perform the same function.

BeautifulSoup results use the default dictionary data structure.  xmlschema results are in a hierarchial dictionary data structure.

from bs4 import BeautifulSoup
import pandas as pd
import re
import requests

headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:89.0) Gecko/20100101 Firefox/89.0",
}

# XBRL TAXONOMY EXTENSION LABEL LINKBASE DOCUMENT.  Part of Apple's 10-Q filing on 06/26/2021.
# url = https://www.sec.gov/Archives/edgar/data/320193/000032019321000065/aapl-20210626_lab.xml

        urlXBRL = "https://www.sec.gov/Archives/edgar/data/320193/000032019321000065/aapl-20210626_lab.xml"
        r = requests.get(urlXBRL,headers=headers)
        soup = BeautifulSoup(r.text, "lxml")
        #
        gaapTags = defaultdict(list)
        if soup.find("link:label"):
            bs = soup.find_all("link:label", {"id": re.compile("lab_us-gaap_\w*_.")})
            path1 = True
        else:
            bs = soup.find_all("label", {"xlink:label": re.compile("us-gaap_\w*_.")})
            path1 = False
        #
        if path1:
            for label in bs:
                step1 = label.attrs["id"]
                key = step1.split("_")[2]
                gaapTags[key.lower()].append(label.text)
        else:
            for label in bs:
                step1 = label.attrs["xlink:label"]
                key = step1.split("_")[1]
                gaapTags[key.lower()].append(label.text)
                
#
import pandas as pd
import xmlschema
from xml.etree import ElementTree

#
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:89.0) Gecko/20100101 Firefox/89.0",
}
# The xml file below is the same XBRL LABEL LINKBASE DOCUMENT as used above.  The xsd url points to the XBRL SCHEMA DOCUMENT that
# is part of a company's SEC 10-Q filing. "xmlTmp_xbrl" is a dictionary containing all label mappings from xbrl terminology to
# commonly used nomenclature.
#
xml = "https://www.sec.gov/Archives/edgar/data/320193/000032019321000065/aapl-20210626_lab.xml"
xsd = "https://www.sec.gov/Archives/edgar/data/320193/000032019321000065/aapl-20210626.xsd"
#
xs = xmlschema.XMLSchema11(xsd)
xmlTmp_xbrl = xs.to_dict(xml)
#
# Print results below for "us-gaap" tags.  Note that xmlTmp_xbrl is a dictionary with all labels, not simply the labels
# for the 06/26/2021 10-Q.  The 10-Q contains historical financial results that a reader would be interested in
# but may get in the way of software algorithms and should be cleansed.

for itm in xmlTmp_xbrl.keys():
    if itm.startswith("us-gaap:"):
        print(itm)
