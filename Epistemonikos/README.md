Epistemonikos
==============================================================================

Issues with the Epistemonikos search platform  that I experienced:

* The Epistemonikos website handles query strings entered in unpredictable ways. Sometimes unwanted field codes are added to the search strings. This may corrupt the search but possibly go unnoticed. Various forms of a search string may be shown in the search forms. It is not always obvious what is actually searched.

* The URL shown in the browser is not a reliable reflection of what is going on in the web page or what is being searched and may be misleading.

* For title-and-abstract searches the search form acts differently from what is usually done in other interfaces. This may deviate from the intention of a searcher.

We try to tackle these with the help of OpenRefine.


## Title/Abstract searches

Entering a search string `block_1 AND block_2` in the _Title/Abstract_ field of the advanced search form is translated into a search string `(title:(block_1 AND block_2) OR abstract:(block_1 AND block_2))`. This will not find records where _block_1_ terms occur in the title field and _block_2_ terms in the abstract field only. And vice versa. This may come unexpected.

Therefore, the following structure of the search strategy may be preferable as it is more sensitive but it is not created by Epistemonikos' search form:

```
(title:(block_1) OR abstract:(block_1))
AND 
(title:(block_2) OR abstract:(block_2))
```

To give a concrete example, we search for:

* Population: cancer
* Intervention: surgery

When entering a (not particularly good) search string as a single line we get the following in the Epistemonikos advanced search form:

![Screenshot of advanced search with title/abstract search](media/Epistemonikos_screenshot_TIAB_search.png)

* String entered in Titel/Abstract field: `(cancer OR carcinoma) AND (surgery OR surgical)`
* String shown in query box: `(title:((cancer OR carcinoma) AND (surgery OR surgical)) OR abstract:((cancer OR carcinoma) AND (surgery OR surgical)))`

The structure of this search as illustrated with [2Dsearch](https://app.2dsearch.com/):

![Epistemonikos default single-line title/abstract search](media/Epistemonikos_default_TIAB_search_2Dsearch.png)

When the term `cancer` is present only in the title and the term `surgery` only in the abstract of a document this document would not be found.

The structure of the title/abstract search "Population AND Intervention" should rather be like this:

```
(title:(cancer OR carcinoma) OR abstract:(cancer OR carcinoma))
AND 
(title:(surgery OR surgical) OR abstract:(surgery OR surgical))
```

![Structure of title/abstract search "Population AND Intervention"](media/Population_AND_Intervention.png)


## The Epistemonikos website

TODO: This paragraph is still a messy collection of things. Rework!

Epistemonikos has a [documented API](https://api.epistemonikos.org). Access requires a registration process with the people from Epistemonikos. It is not clear who will be allowed access. Is is nor documented whether searching fpr documents with this API is the same as when searching in the web forms.

However, the Advanced Search form also uses a (different) API for it to function. JavaScript code running in the browser is calling this API to do the searching. This can be seen when using the developer tools of the web browser. We can call this publicly available API with OpenRefine rather than with the JavaScripr code in the browser. Then we know exactly what search string we are sending.

API used by the Advanced search form:

https://www.epistemonikos.org/en/api/documents?classification=all&countries=all&page=1&pmc=all&protocol=no&q=(title:((cancer+OR+carcinoma)+AND+(surgery+OR+surgical))+OR+abstract:((cancer+OR+carcinoma)+AND+(surgery+OR+surgical)))&query=&study_design=all&systematic_review_type=all&type_of_meta_analysis=all

The URL parameter `q` apparently has no influence on the actual search. The web app may carry over previous search strings in this parameter. We will leave it empty so as not to cause confusion.

The URL parameter `query` is the actual search string.


The other parameters are filters and for paging the result list.


### RIS download

In an example search the following URL was used by the web page to download results in RIS format: <https://www.epistemonikos.org/en/documents/documents_ris?classification=all&countries=all&pmc=all&protocol=no&q=&query=(title:(cancer OR carcinoma) OR abstract:(cancer OR carcinoma)) AND (title:(surgery OR surgical) OR abstract:(surgery OR surgical)) AND (title:(immunology) OR abstract:(immunology))&size=100&study_design=all&systematic_review_type=all&type_of_meta_analysis=all>

Result comes in a JSON document. In the web page only up to 100 records in RIS format are offered for download without registration. More are possible after login. But the size parameter in this URL allows for larger numbers. After login the web page offers to download up to 1000, 5000 or 10000 records, although in a somwewhat different fashion. There seems to be no paging with this URL. So be careful not to send a request to this endpoint with a query that yields too many records! We do not want to put Epistemonikos under stress.

The URL such that the search string can be appended (in URL encoded form):

```
https://www.epistemonikos.org/en/documents/documents_ris?classification=all&countries=all&pmc=all&protocol=no&study_design=all&systematic_review_type=all&type_of_meta_analysis=all&q=&size=100&query=
```


## Using OpenRefine

### Goals

* We want to search Epistemonikos with a single line search strategy composed of two or more search blocks that are combined with AND. These blocks could refer to the PICO model.
* We want a proper title-and-abstract search where search terms from the search blocks may occur in either title or abstract. Not the way Epistemonikos behaves when using the _Title/Abstract_ field in advanced search. See above for more information.
* We want to avoid that the Epistemonikos search forms in the browser introduce unwanted (and possibly unnoticed) modifications to our search strategy.
* We want a proper search documentation with exactly the search string that was sent to the search engine and was used to return documents.
* We want to download the records from exactly that search as a RIS file.


### Approach

While it is possible to build the complete search string manually we here use OpenRefine to assemble the pieces. This process is described below. However, it is important to note that this assembly is optional. These steps can be skipped and a search string in its final intended form may be provided that will be sent unaltered to the Epistemonikos API.

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

#### Assemble search strategy

I found it convenient to prepare the search strings **without field codes** in an Excel file which each search block in a single cell. For _population_ and _intervention_ I used the more general terms _block\_1_ and _block\_2_: 


| block\_1            | block\_2            | block\_3    |
| ------------------- | ------------------- | ----------- |
| cancer OR carcinoma | surgery OR surgical | immunology  |

This [Excel file](data/Epistemonikos/Epistemonikos_search_blocks.xlsx) can be loaded into OpenRefine.

This is what the imported search strategy looks like in OpenRefine:

![Search strategy imported](media/search_strategy_imported.png)

In OpenRefine the search string is constructed with the following GREL expression. It is possible to have as search blocks/columns as suits the need. They will all be combined with AND in the final search string. Terms from each search block may occur in either title or abstract or both. 

```grel

forEach(
  row.columnNames, 
  v, 
  "(title:(" + cells[v].value + ") OR abstract:(" + cells[v].value + "))"
).join(" AND ")

```

We then move the column with the search string to the beginning.

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
    "index": 0,
    "description": "Move column search_string to position 0"
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
    "op": "core/column-addition",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "baseColumnName": "search_string",
    "expression": "grel:\"https://www.epistemonikos.org/en/api/documents?classification=all&countries=all&pmc=all&study_design=all&systematic_review_type=all&type_of_meta_analysis=all&protocol=no&q=\" +\n\"&page=1\" +\n\"&query=\" + escape(value, \"url\")",
    "onError": "set-to-blank",
    "newColumnName": "search_url",
    "columnInsertIndex": 1,
    "description": "Create column search_url at index 1 based on column search_string using expression grel:\"https://www.epistemonikos.org/en/api/documents?classification=all&countries=all&pmc=all&study_design=all&systematic_review_type=all&type_of_meta_analysis=all&protocol=no&q=\" +\n\"&page=1\" +\n\"&query=\" + escape(value, \"url\")"
  },
  {
    "op": "core/column-addition-by-fetching-urls",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "baseColumnName": "search_url",
    "urlExpression": "grel:value",
    "onError": "set-to-blank",
    "newColumnName": "first_search",
    "columnInsertIndex": 2,
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
    "description": "Create column first_search at index 2 by fetching URLs based on column search_url using expression grel:value"
  }
]

```


### Search result

Our search yields a JSON document with some general information about the search result and a chunk with the first records. It looks as if only 10 records at a time can be obtained from the API we are using here. If there are more than 10 records in the search result these have to be obtained by paging through the list. This we will do in the next section.

First we extract the result count from the JSON. In addition, a field _search_status_ is present which we extract, too. The exact meaning of this status is not clear to me but we probabaly want that the status is "ok".

```json

[
  {
    "op": "core/column-addition",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "baseColumnName": "first_search",
    "expression": "grel:value.parseJson().total_results",
    "onError": "set-to-blank",
    "newColumnName": "total_results",
    "columnInsertIndex": 1,
    "description": "Create column total_results at index 1 based on column first_search using expression grel:value.parseJson().total_results"
  },
  {
    "op": "core/column-addition",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "baseColumnName": "first_search",
    "expression": "grel:value.parseJson().search_status",
    "onError": "set-to-blank",
    "newColumnName": "first_search_status",
    "columnInsertIndex": 2,
    "description": "Create column first_search_status at index 2 based on column first_search using expression grel:value.parseJson().search_status"
  }
]

```

### Export search history

At this stage it is convenient to export a search history from OpenRefine, e.g. search terms and their record counts. 

Use the Custom Tabular Exporter to export to a tsv-file. Use the following option code for the _Custom tabular exporter_ to export the search strategy (columns _search_string, _total_results_, _first_search_status_ and _search_url_) as a tsv file. Paste this code into the _Option code_ tab and click on _Apply_. Then go to the _Download_ tab and click on the _Download_ button.

```json

{
  "format": "tsv",
  "separator": "\t",
  "lineSeparator": "\n",
  "encoding": "UTF-8",
  "quoteAll": true,
  "outputColumnHeaders": true,
  "outputBlankRows": false,
  "columns": [
    {
      "name": "search_string",
      "reconSettings": {
        "output": "entity-name",
        "blankUnmatchedCells": false,
        "linkToEntityPages": true
      },
      "dateSettings": {
        "format": "iso-8601",
        "useLocalTimeZone": false,
        "omitTime": false
      }
    },
    {
      "name": "total_results",
      "reconSettings": {
        "output": "entity-name",
        "blankUnmatchedCells": false,
        "linkToEntityPages": true
      },
      "dateSettings": {
        "format": "iso-8601",
        "useLocalTimeZone": false,
        "omitTime": false
      }
    },
    {
      "name": "first_search_status",
      "reconSettings": {
        "output": "entity-name",
        "blankUnmatchedCells": false,
        "linkToEntityPages": true
      },
      "dateSettings": {
        "format": "iso-8601",
        "useLocalTimeZone": false,
        "omitTime": false
      }
    },
    {
      "name": "search_url",
      "reconSettings": {
        "output": "entity-name",
        "blankUnmatchedCells": false,
        "linkToEntityPages": true
      },
      "dateSettings": {
        "format": "iso-8601",
        "useLocalTimeZone": false,
        "omitTime": false
      }
    }
  ]
}

```

The search history as exported to a [tsv-file](data/Epistemonikos/Epistemonikos-search-history.tsv).


### Fetch all records

The records are available in chunks of 10 as paged on the website. So we need to download these chunks one after the other.

We create a row for every results set page, build the URL to download and then fetch the JSON from that URL.

The URLS are assembled with this GREL code:

```grel

"https://www.epistemonikos.org/en/api/documents?classification=all&countries=all&pmc=all&study_design=all&systematic_review_type=all&type_of_meta_analysis=all&protocol=no&q=" +
"&page=" + cells["result_set_page"].value +
"&query=" + escape(cells["search_string"].value, "url")

```

The option code JSON to be applied:

```json

[
  {
    "op": "core/column-addition",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "baseColumnName": "total_results",
    "expression": "grel:forRange(1, ceil(cells[\"total_results\"].value / 10.0) + 1, 1, v, v).join(\"|\")",
    "onError": "set-to-blank",
    "newColumnName": "result_set_page",
    "columnInsertIndex": 2,
    "description": "Create column result_set_page at index 2 based on column total_results using expression grel:forRange(1, ceil(cells[\"total_results\"].value / 10.0) + 1, 1, v, v).join(\"|\")"
  },
  {
    "op": "core/multivalued-cell-split",
    "columnName": "result_set_page",
    "keyColumnName": "search_string",
    "mode": "separator",
    "separator": "|",
    "regex": false,
    "description": "Split multi-valued cells in column result_set_page"
  },
  {
    "op": "core/fill-down",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "columnName": "search_string",
    "description": "Fill down cells in column search_string"
  },
  {
    "op": "core/column-addition",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "baseColumnName": "result_set_page",
    "expression": "grel:\"https://www.epistemonikos.org/en/api/documents?classification=all&countries=all&pmc=all&study_design=all&systematic_review_type=all&type_of_meta_analysis=all&protocol=no&q=\" +\n\"&page=\" + cells[\"result_set_page\"].value +\n\"&query=\" + escape(cells[\"search_string\"].value, \"url\")",
    "onError": "set-to-blank",
    "newColumnName": "download_url",
    "columnInsertIndex": 3,
    "description": "Create column download_url at index 3 based on column result_set_page using expression grel:\"https://www.epistemonikos.org/en/api/documents?classification=all&countries=all&pmc=all&study_design=all&systematic_review_type=all&type_of_meta_analysis=all&protocol=no&q=\" +\n\"&page=\" + cells[\"result_set_page\"].value +\n\"&query=\" + escape(cells[\"search_string\"].value, \"url\")"
  }
]

```

Now that we have the download URLs created we go and fetch the data:

```json

[
  {
    "op": "core/column-addition-by-fetching-urls",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "baseColumnName": "download_url",
    "urlExpression": "grel:value",
    "onError": "set-to-blank",
    "newColumnName": "result_chunk",
    "columnInsertIndex": 4,
    "delay": 2000,
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
    "description": "Create column result_chunk at index 4 by fetching URLs based on column download_url using expression grel:value"
  }
]

```


### Documents to cells

We now extract the records from the chunks downloaded and split so that every document (record) is in its own cell. After this, the number of rows should equal the number of total results we earlier extracted from our first search and which is documented in the search history (assuming we started with a single row of search blocks).

```json

[
  {
    "op": "core/column-addition",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "baseColumnName": "result_chunk",
    "expression": "grel:value.parseJson().documents.join(\"|||DOCUMENT_SEPARATOR|||\")",
    "onError": "set-to-blank",
    "newColumnName": "documents",
    "columnInsertIndex": 5,
    "description": "Create column documents at index 5 based on column result_chunk using expression grel:value.parseJson().documents.join(\"|||DOCUMENT_SEPARATOR|||\")"
  },
  {
    "op": "core/multivalued-cell-split",
    "columnName": "documents",
    "keyColumnName": "search_string",
    "mode": "separator",
    "separator": "|||DOCUMENT_SEPARATOR|||",
    "regex": false,
    "description": "Split multi-valued cells in column documents"
  }
]

```


### Extract metadata fields

TODO but easy

### Export records to RIS file

TODO but easy


## Discussion

Pros and cons of this approach


## Direct RIS download

### Export search history

Use the custom tabular exporter. Apply the following "Option code". Then go to the "Download" tab and click the "Download" button.

```json

{
  "format": "tsv",
  "separator": "\t",
  "lineSeparator": "\n",
  "encoding": "UTF-8",
  "quoteAll": true,
  "outputColumnHeaders": true,
  "outputBlankRows": false,
  "columns": [
    {
      "name": "search_string",
      "reconSettings": {
        "output": "entity-name",
        "blankUnmatchedCells": false,
        "linkToEntityPages": true
      },
      "dateSettings": {
        "format": "iso-8601",
        "useLocalTimeZone": false,
        "omitTime": false
      }
    },
    {
      "name": "ris_download_url",
      "reconSettings": {
        "output": "entity-name",
        "blankUnmatchedCells": false,
        "linkToEntityPages": true
      },
      "dateSettings": {
        "format": "iso-8601",
        "useLocalTimeZone": false,
        "omitTime": false
      }
    },
    {
      "name": "total_results",
      "reconSettings": {
        "output": "entity-name",
        "blankUnmatchedCells": false,
        "linkToEntityPages": true
      },
      "dateSettings": {
        "format": "iso-8601",
        "useLocalTimeZone": false,
        "omitTime": false
      }
    },
    {
      "name": "first_search_status",
      "reconSettings": {
        "output": "entity-name",
        "blankUnmatchedCells": false,
        "linkToEntityPages": true
      },
      "dateSettings": {
        "format": "iso-8601",
        "useLocalTimeZone": false,
        "omitTime": false
      }
    },
    {
      "name": "search_url",
      "reconSettings": {
        "output": "entity-name",
        "blankUnmatchedCells": false,
        "linkToEntityPages": true
      },
      "dateSettings": {
        "format": "iso-8601",
        "useLocalTimeZone": false,
        "omitTime": false
      }
    }
  ]
}

```


### Export to RIS file

Results are in the column `ris`. We export with the templatiing exporter. This is "Row template":

```
{{cells["ris"].value}}
```

"Prefix", "Row separator" and "Suffix" must be empty.


