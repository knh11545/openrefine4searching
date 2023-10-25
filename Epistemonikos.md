Epistemonikos
==============================================================================

Issues:

* Website handles query strings entered in unpredictable ways. Sometimes unwanted field codes are added to the search strings. Various forms of a search string may be shown in the search forms. It is not always obvious what is actually searched. The URL shown in the browser is not a reliable reflection of what is going on in the web page.

* For title-and-abstract searches the search form acts differently from what is usually done in other interfaces. This may deviate from the intention of a searcher:

Entering a search string `block_1 AND block_2` in the _Title/Abstract_ field of the advanced search form is translated into a search string `(title:(block_1 AND block_2) OR abstract:(block_1 AND block_2))`. This will not find records where _block_2_ occurs in the title field and _block_2_ in the abstract field only. And vice versa. This may come unexpected.

Therefore, the following structure of the search strategy may be preferable but is not created by Epistemonikos' search form:

```
(title:(block_1) OR abstract:(block_1))
AND 
(title:(block_2) OR abstract:(block_2))
```

A concrete example (but not a particularly sensitive search string):

```
(title:(cancer OR carcinoma) OR abstract:(cancer OR carcinoma))
AND 
(title:(surgery OR surgical) OR abstract:(surgery OR surgical))
```

TODO: Add link to API documentation: <https://api.epistemonikos.org>

API used by the Advanced search form:

https://www.epistemonikos.org/en/api/documents?classification=all&countries=all&page=1&pmc=all&protocol=no&q=(title:((+%22dental+crown%22+OR+%22oral+crown%22+OR+%22tooth+crown%22+OR+%22dental+crowns%22+OR+%22oral+crowns%22+OR+%22tooth+crowns%22+OR+((dental+OR+oral+OR+tooth)+AND+(implant+OR+abutment))+OR+veneer+OR+%22dental+laminate%22+OR+restoration*+OR+onlay*+OR+inlay*+OR+overlay*+OR+partialcrown*+OR+tabletop+OR+%22partial+crown%22+OR+%22partial+crowns%22+OR+%22fixed+partial+denture%22+OR+%22fixed+partial+dentures%22+OR+%22fixed+dental+prosthesis%22+OR+%22fixed+dental+prostheses%22+OR+%22dental+bridge%22+OR+%22tooth+bridge%22+OR+%22fixed+bridge%22+OR+%22resin+bridge%22+OR+%22maryland+bridge%22+OR+%22rochette+bridge%22+OR+%22dental+bridges%22+OR+%22tooth+bridges%22+OR+%22fixed+bridges%22+OR+%22resin+bridges%22+OR+%22maryland+bridges%22+OR+%22rochette+bridges%22+OR+%22dental+pontic%22+OR+%22tooth+pontic%22+OR+%22fixed+pontic%22+OR+%22resin+pontic%22+OR+%22maryland+pontic%22+OR+%22rochette+pontic%22+OR+%22dental+pontics%22+OR+%22tooth+pontics%22+OR+%22fixed+pontics%22+OR+%22resin+pontics%22+OR+%22maryland+pontics%22+OR+%22rochette+pontics%22+OR+(prosthodontic+AND+%22fixed+restoration%22)+OR+(prosthodontic+AND+%22fixed+restorations%22)+OR+%22monolithic+restoration%22+OR+%22anatomic+restoration%22+OR+%22monolithic+restorations%22+OR+%22anatomic+restorations%22+OR+((FPD++OR+FPDs+OR+FDP+OR+FDPs)+AND+(dental+OR+tooth))+)+AND+(+ceramic+OR+ceramics+OR+allceramic+OR+porcelain+OR+lithia+OR+lithium+OR+silicate+OR+silicates+OR+disilicate+OR+disilicates+OR+LiSi+OR+LiAlSi+OR+Li2Si2O5+OR+metal-free+OR+metalfree+OR+non-metal+OR+nonmetal+OR+%22e-max%22+OR+emax+OR+celtra+OR+tessera+OR+suprinity+OR+zls+OR+empress2+OR+%22empress+2%22+OR+amber+OR+livento+OR+conceptpress+OR+%22concept+press%22+OR+vintage+OR+nice+OR+%22n+ce%22+))+OR+abstract:((+%22dental+crown%22+OR+%22oral+crown%22+OR+%22tooth+crown%22+OR+%22dental+crowns%22+OR+%22oral+crowns%22+OR+%22tooth+crowns%22+OR+((dental+OR+oral+OR+tooth)+AND+(implant+OR+abutment))+OR+veneer+OR+%22dental+laminate%22+OR+restoration*+OR+onlay*+OR+inlay*+OR+overlay*+OR+partialcrown*+OR+tabletop+OR+%22partial+crown%22+OR+%22partial+crowns%22+OR+%22fixed+partial+denture%22+OR+%22fixed+partial+dentures%22+OR+%22fixed+dental+prosthesis%22+OR+%22fixed+dental+prostheses%22+OR+%22dental+bridge%22+OR+%22tooth+bridge%22+OR+%22fixed+bridge%22+OR+%22resin+bridge%22+OR+%22maryland+bridge%22+OR+%22rochette+bridge%22+OR+%22dental+bridges%22+OR+%22tooth+bridges%22+OR+%22fixed+bridges%22+OR+%22resin+bridges%22+OR+%22maryland+bridges%22+OR+%22rochette+bridges%22+OR+%22dental+pontic%22+OR+%22tooth+pontic%22+OR+%22fixed+pontic%22+OR+%22resin+pontic%22+OR+%22maryland+pontic%22+OR+%22rochette+pontic%22+OR+%22dental+pontics%22+OR+%22tooth+pontics%22+OR+%22fixed+pontics%22+OR+%22resin+pontics%22+OR+%22maryland+pontics%22+OR+%22rochette+pontics%22+OR+(prosthodontic+AND+%22fixed+restoration%22)+OR+(prosthodontic+AND+%22fixed+restorations%22)+OR+%22monolithic+restoration%22+OR+%22anatomic+restoration%22+OR+%22monolithic+restorations%22+OR+%22anatomic+restorations%22+OR+((FPD++OR+FPDs+OR+FDP+OR+FDPs)+AND+(dental+OR+tooth))+)+AND+(+ceramic+OR+ceramics+OR+allceramic+OR+porcelain+OR+lithia+OR+lithium+OR+silicate+OR+silicates+OR+disilicate+OR+disilicates+OR+LiSi+OR+LiAlSi+OR+Li2Si2O5+OR+metal-free+OR+metalfree+OR+non-metal+OR+nonmetal+OR+%22e-max%22+OR+emax+OR+celtra+OR+tessera+OR+suprinity+OR+zls+OR+empress2+OR+%22empress+2%22+OR+amber+OR+livento+OR+conceptpress+OR+%22concept+press%22+OR+vintage+OR+nice+OR+%22n+ce%22+)))&query=(title:((cancer+OR+carcinoma)+AND+(surgery+OR+surgical))+OR+abstract:((cancer+OR+carcinoma)+AND+(surgery+OR+surgical)))&study_design=all&systematic_review_type=all&type_of_meta_analysis=all

The URL parameter `q` apparently has no influence on the actual search. The web app may carry over previous search strings in this parameter. We will leave it empty so as not to cause confusion.

The URL parameter `query` is the actual search string.


The other parameters are filters and for paging the result list.


## Using OpenRefine

### Goals

* We want to search Epistemonikos with a search strategy composed of two or more search blocks that are combined with AND. These blocks could be  
* We want a proper title-and-abstract search where search terms from the search blocks may occur in either title or abstract. Not the way Epistemonikos behaves when using the _Title/Abstract_ field in advanced search. See above for more information.
* We want to avoid that the Epistemonikos search forms in the browser introduce unwanted (and possibly unnoticed) modifications to our search strategy.


### Approach

* We start with an Excel file where the search strings of the individual search blocks are in separate cells next to each other, i.e. in separate columns in a single row. The first row in the Excel file holds the column names, i.e. _block_1_, _block_2_ and _block\_3_.
* The search string with the full structure of the search strategy in the following form is built by OpenRefine:

```
(title:(block_1) OR abstract:(block_1))
AND 
(title:(block_2) OR abstract:(block_2))
AND 
(title:(block_3) OR abstract:(block_3))
```

* OpenRefine sends the search string directly to the API that is also used by JavaScript code of the search forms of Epistemonikos. We circumvent that the search string entered in the search forms' input fields is handled (and possibly modified in an unintended way) by the JavaScript code.
* From that API we retrieve the JSON containing the search results. We need to fetch the results in chunks of 10 records as this is what the web page uses.
* We extract metadata fields from the JSON so taht we can export the records in RIS format.


### Step by step

#### Search strategy

I found it convenient to prepare the search strings **without field codes** in an Excel file which each search block in a single cell. For _population_ and _intervention_ I used the more general terms _block\_1_ and _block\_2_: 


| block\_1            | block\_2            | block\_3    |
| ------------------- | ------------------- | ----------- |
| cancer OR carcinoma | surgery OR surgical | immunology  |

This [Excel file](data/Epistemonikos/Epistemonikos_search_blocks.xlsx) can be loaded into OpenRefine.

In OpenRefine the search string is constructed with the following GREL expression:

```grel

forEach(
  row.columnNames, 
  v, 
  "(title:(" + cells[v].value + ") OR abstract:(" + cells[v].value + "))"
).join(" AND ")

```

The operation history JSON to be applied in OpenRefine:

```json

[
  {
    "op": "core/column-addition",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "baseColumnName": "block_1",
    "expression": "grel:forEach(\n  row.columnNames, \n  v, \n  \"(title:(\" + cells[v].value + \") OR abstract:(\" + cells[v].value + \"))\"\n).join(\" AND \")",
    "onError": "set-to-blank",
    "newColumnName": "search_string",
    "columnInsertIndex": 1,
    "description": "Create column search_string at index 1 based on column block_1 using expression grel:forEach(\n  row.columnNames, \n  v, \n  \"(title:(\" + cells[v].value + \") OR abstract:(\" + cells[v].value + \"))\"\n).join(\" AND \")"
  },
  {
    "op": "core/column-move",
    "columnName": "search_string",
    "index": 3,
    "description": "Move column search_string to position 3"
  }
]

```

#### Search using API

GREL code to build the URL:

```grel

"https://www.epistemonikos.org/en/api/documents?classification=all&countries=all&pmc=all&study_design=all&systematic_review_type=all&type_of_meta_analysis=all&protocol=no&q=" +
"&page=1" +
"&query=" + escape(value, "url")

```


The operation history JSON to be applied in OpenRefine:

```json

[
  {
    "op": "core/column-addition-by-fetching-urls",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "baseColumnName": "search_string",
    "urlExpression": "grel:\"https://www.epistemonikos.org/en/api/documents?classification=all&countries=all&pmc=all&study_design=all&systematic_review_type=all&type_of_meta_analysis=all&protocol=no&q=\" +\n\"&page=1\" +\n\"&query=\" + escape(value, \"url\")",
    "onError": "set-to-blank",
    "newColumnName": "first_search",
    "columnInsertIndex": 4,
    "delay": 500,
    "cacheResponses": true,
    "httpHeadersJson": [
      {
        "name": "authorization",
        "value": ""
      },
      {
        "name": "user-agent",
        "value": "OpenRefine 3.7.4 [05e9de9]"
      },
      {
        "name": "accept",
        "value": "*/*"
      }
    ],
    "description": "Create column first_search at index 4 by fetching URLs based on column search_string using expression grel:\"https://www.epistemonikos.org/en/api/documents?classification=all&countries=all&pmc=all&study_design=all&systematic_review_type=all&type_of_meta_analysis=all&protocol=no&q=\" +\n\"&page=1\" +\n\"&query=\" + escape(value, \"url\")"
  }
]

```




