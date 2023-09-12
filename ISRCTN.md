Search ISRCTN registry
==============================================================================

The [ISRCTN registry](https://www.isrctn.com/) of clinical trials has a few limitations for comprehensive, systematic searches. According to the [documentation](https://www.isrctn.com/page/search-tips) there "are no synonym systems underpinning the search" and there are no wildcards that allow for truncation and masking. Therefore, plurals and spelling variants of search terms can be dealt with only by including many such variants. This means that search strings will tend to be longer than with other search interfaces. But then the length of search strings is limited due to them being part of the URLs used to call functions of the ISRCTN search interface. ISRCTN registry has no search history functions where previous search statements can be reused in a brief fashion by referencing statements numbers. All terms of a search need to go into a single line search strategy. The individual fields chosen in the only export option CSV are added to the URL and reduce the space available for a query string.

Length of the minimal useful base URL `https://www.isrctn.com/searchCsv?columns=ISRCTN&q=` for a download of only the ISRCTN IDs is 50 characters. According to the error message that appears when using longer search strings, the URLs probably have a "configured limit of 2048 characters" (but this exact number may be a limit inside the inner workings of the platform). So, query strings should be somewhat shorter than 2,000 characters after URL encoding.

But there is an [API](https://www.isrctn.com/api) that allows to search for registry entries. Records can be downloaded in several XML formats. For more information see [here](https://www.isrctn.com/page/faqs#finding-study-information) under "How do I download data from the ISRCTN registry?".

Note: Springer Nature askeed me to register in order to get access to the API documentation. But the actual [download location](https://www.isrctn.com/editorial/retrieveFile/38def0ea-8330-4eab-b288-d21115c782f3/37855) seems to be unprotected. The API documentation was a Word file that contained the XML schemas of the various download formats in a kind of MS-Specific "OLE package". After some trial and error I was able to extract the packages. Turned out they were just plain XML schema files.


## Search terms

Longer search strategies may need to be split into parts the search results of which can be combined with OR later. For a search strategy `Population AND Intervention` for example do the following. Create three serarate strategies:

```
P AND I_chunk_1
P AND I_chunk_2
P AND I_chunk_3
```

I found it convenient to do this in an Excel file which each search block in a single cell. For _population_ and _intervention_ I used the more general terms _block\_1_ and _block\_2_: 

| block\_1                              | block\_2 |
| --------------------------------------- | ------------ |
| (nasal OR nose OR sinonasal) AND cancer | surgery      |
| (nasal OR nose OR sinonasal) AND cancer | radiotherapy |
| (nasal OR nose OR sinonasal) AND cancer | imaging      |

This Excel file can be loaded into OpenRefine.



## Search using API

Here we start out by combining the partial search statements _block\_1_ and _block\_2_ with a Boolean AND. Then we build the query and take care to [URL encode](https://en.wikipedia.org/wiki/URL_encoding) the query string. As this will increase the length of the URL, we also create a cloumn _url\_length_ that allows  to create a facet to check the length of the resulting URLs:

```json

[
  {
    "op": "core/column-addition",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "baseColumnName": "block_2",
    "expression": "grel:\"(\" + cells[\"block_1\"].value + \") AND (\" + cells[\"block_2\"].value +\")\"",
    "onError": "set-to-blank",
    "newColumnName": "query",
    "columnInsertIndex": 2,
    "description": "Create column query at index 2 based on column block_2 using expression grel:\"(\" + cells[\"block_1\"].value + \") AND (\" + cells[\"block_2\"].value +\")\""
  },
  {
    "op": "core/column-addition",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "baseColumnName": "query",
    "expression": "grel:\"https://www.isrctn.com/api/query/format/default?limit=200&q=\" + value.escape(\"url\")",
    "onError": "set-to-blank",
    "newColumnName": "query_url",
    "columnInsertIndex": 3,
    "description": "Create column query_url at index 3 based on column query using expression grel:\"https://www.isrctn.com/api/query/format/default?limit=200&q=\" + value.escape(\"url\")"
  },
  {
    "op": "core/column-addition",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "baseColumnName": "query_url",
    "expression": "grel:value.length()",
    "onError": "set-to-blank",
    "newColumnName": "url_length",
    "columnInsertIndex": 4,
    "description": "Create column url_length at index 4 based on column query_url using expression grel:value.length()"
  }
]

```

The we can carry out the search by fetching the URLs and store the record set in XML format. 

Note: When building the query URL we set the maximum number of results to download in a single query to 200. Adjust as needed when you expect higher numbers. But please be considerate and adhere to the guidelines given in the API documentation.

To check what we got we extract the number of hits (attribute _totalCount_) given in the recordset and compare that to the actual number of records we retrieved with each query. You need to check _result_count_equals_trials_downloaded_ column for false using a facet.

```json

[
  {
    "op": "core/column-addition-by-fetching-urls",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "baseColumnName": "query_url",
    "urlExpression": "grel:value",
    "onError": "set-to-blank",
    "newColumnName": "xml_download",
    "columnInsertIndex": 4,
    "delay": 5000,
    "cacheResponses": true,
    "httpHeadersJson": [
      {
        "name": "authorization",
        "value": ""
      },
      {
        "name": "user-agent",
        "value": "OpenRefine 3.6.1 [5fc8883]"
      },
      {
        "name": "accept",
        "value": "*/*"
      }
    ],
    "description": "Create column xml_download at index 4 by fetching URLs based on column query_url using expression grel:value"
  },
  {
    "op": "core/column-addition",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "baseColumnName": "xml_download",
    "expression": "grel:value.parseXml().select(\"allTrials\")[0].xmlAttr(\"totalCount\")",
    "onError": "set-to-blank",
    "newColumnName": "result_count",
    "columnInsertIndex": 5,
    "description": "Create column result_count at index 5 based on column xml_download using expression grel:value.parseXml().select(\"allTrials\")[0].xmlAttr(\"totalCount\")"
  },
  {
    "op": "core/column-addition",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "baseColumnName": "xml_download",
    "expression": "grel:value.parseXml().select(\"allTrials fullTrial\").length()",
    "onError": "set-to-blank",
    "newColumnName": "records_downloaded",
    "columnInsertIndex": 5,
    "description": "Create column records_downloaded at index 5 based on column xml_download using expression grel:value.parseXml().select(\"allTrials fullTrial\").length()"
  },
  {
    "op": "core/column-addition",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "baseColumnName": "result_count",
    "expression": "grel:cells[\"result_count\"].value.toNumber() == cells[\"records_downloaded\"].value.toNumber()",
    "onError": "set-to-blank",
    "newColumnName": "result_count_equals_trials_downloaded",
    "columnInsertIndex": 7,
    "description": "Create column result_count_equals_trials_downloaded at index 7 based on column result_count using expression grel:cells[\"result_count\"].value.toNumber() == cells[\"records_downloaded\"].value.toNumber()"
  }
]

```

## Export search history

At this stage it is convenient to export a search history from OpenRefine, e.g. search terms and their record counts. 

Use the Custom Tabular Exporter to export to a tsv-file. Use the following option code for the _Custom tabular exporter_ to export the search strategy (columns _query_ and _result\_count_) as a tsv file. Paste this code into the _Option code_ tab and click on _Apply_:


```json

{
  "format": "tsv",
  "separator": "\t",
  "lineSeparator": "\n",
  "encoding": "UTF-8",
  "quoteAll": false,
  "outputColumnHeaders": true,
  "outputBlankRows": false,
  "columns": [
    {
      "name": "query",
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
      "name": "result_count",
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


## Extract record data

First, we create a record of every guidline entry found.

```json

[
  {
    "op": "core/column-addition",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "baseColumnName": "xml_download",
    "expression": "grel:value.parseXml().select(\"allTrials fullTrial\").join(\"|||\")",
    "onError": "set-to-blank",
    "newColumnName": "xml_record",
    "columnInsertIndex": 5,
    "description": "Create column xml_record at index 5 based on column xml_download using expression grel:value.parseXml().select(\"allTrials fullTrial\").join(\"|||\")"
  },
  {
    "op": "core/column-move",
    "columnName": "xml_record",
    "index": 0,
    "description": "Move column xml_record to position 0"
  },
  {
    "op": "core/multivalued-cell-split",
    "columnName": "xml_record",
    "keyColumnName": "xml_record",
    "mode": "separator",
    "separator": "|||",
    "regex": false,
    "description": "Split multi-valued cells in column xml_record"
  }
]

```

We extract some minimal data about the trials to be exported into a RIS file.

For the _year_ we use the _dateAssigned_ field of the trial.

```json

[
  {
    "op": "core/column-addition",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "baseColumnName": "xml_record",
    "expression": "grel:value.parseXml().select(\"fullTrial trial isrctn\")[0].xmlText()",
    "onError": "set-to-blank",
    "newColumnName": "isrctn_id",
    "columnInsertIndex": 1,
    "description": "Create column isrctn_id at index 1 based on column xml_record using expression grel:value.parseXml().select(\"fullTrial trial isrctn\")[0].xmlText()"
  },
  {
    "op": "core/column-move",
    "columnName": "isrctn_id",
    "index": 0,
    "description": "Move column isrctn_id to position 0"
  },
  {
    "op": "core/column-addition",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "baseColumnName": "isrctn_id",
    "expression": "grel:\"https://www.isrctn.com/ISRCTN\" + value",
    "onError": "set-to-blank",
    "newColumnName": "trial_url",
    "columnInsertIndex": 1,
    "description": "Create column trial_url at index 1 based on column isrctn_id using expression grel:\"https://www.isrctn.com/ISRCTN\" + value"
  },
  {
    "op": "core/column-addition",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "baseColumnName": "isrctn_id",
    "expression": "grel:\"10.1186/ISRCTN\" + value",
    "onError": "set-to-blank",
    "newColumnName": "DOI",
    "columnInsertIndex": 1,
    "description": "Create column DOI at index 1 based on column isrctn_id using expression grel:\"10.1186/ISRCTN\" + value"
  },
  {
    "op": "core/column-addition",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "baseColumnName": "xml_record",
    "expression": "grel:value.parseXml().select(\"fullTrial trial trialDescription title\")[0].xmlText()",
    "onError": "set-to-blank",
    "newColumnName": "title",
    "columnInsertIndex": 2,
    "description": "Create column title at index 2 based on column xml_record using expression grel:value.parseXml().select(\"fullTrial trial trialDescription title\")[0].xmlText()"
  },
  {
    "op": "core/column-addition",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "baseColumnName": "xml_record",
    "expression": "grel:value.parseXml().select(\"fullTrial trial isrctn\")[0].xmlAttr(\"dateAssigned\").substring(0, 4)",
    "onError": "set-to-blank",
    "newColumnName": "year",
    "columnInsertIndex": 2,
    "description": "Create column year at index 2 based on column xml_record using expression grel:value.parseXml().select(\"fullTrial trial isrctn\")[0].xmlAttr(\"dateAssigned\").substring(0, 4)"
  },
  {
    "op": "core/column-addition",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "baseColumnName": "xml_record",
    "expression": "grel:value.parseXml().select(\"fullTrial trial trialDescription plainEnglishSummary\")[0].wholeText().replace(/[\\r\\n]+/, \"\\r\\n      \").replace(/\\s+$/, \"\")",
    "onError": "set-to-blank",
    "newColumnName": "abstract",
    "columnInsertIndex": 2,
    "description": "Create column abstract at index 6 based on column xml_record using expression grel:value.parseXml().select(\"fullTrial trial trialDescription plainEnglishSummary\")[0].wholeText().replace(/[\\r\\n]+/, \"\\r\\n      \").replace(/\\s+$/, \"\")"
  }
]

```

## Deduplication

In the following steps we will deduplicate records and therefore delete some data. If a complete record of the data returned from searching the guideline register is wanted then now would be a good time to save an archive file of the OpenRefine project at the current state of the workflow. This file can be imported into OpenRefine for further reference.


We are still in records mode with each search term being a record. Now we distill the data down to individual guidelines. This way we go from records mode to row mode.

Then we deduplicate the guidelines along [this guide](https://guides.library.illinois.edu/openrefine/duplicates). A briefer documentation of this approach is found in the OpenRefine Recipes [here](https://github.com/OpenRefine/OpenRefine/wiki/Recipes#removing-duplicate-rows-when-exact-values-are-found-in-a-column). We use the `isrctn_id` column to deduplicate.

```json

[
  {
    "op": "core/row-reorder",
    "mode": "row-based",
    "sorting": {
      "criteria": [
        {
          "valueType": "string",
          "column": "isrctn_id",
          "blankPosition": 2,
          "errorPosition": 1,
          "reverse": false,
          "caseSensitive": false
        }
      ]
    },
    "description": "Reorder rows"
  },
  {
    "op": "core/blank-down",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "columnName": "isrctn_id",
    "description": "Blank down cells in column isrctn_id"
  },
  {
    "op": "core/row-removal",
    "engineConfig": {
      "facets": [
        {
          "type": "list",
          "name": "isrctn_id",
          "expression": "isBlank(value)",
          "columnName": "isrctn_id",
          "invert": false,
          "omitBlank": false,
          "omitError": false,
          "selection": [
            {
              "v": {
                "v": true,
                "l": "true"
              }
            }
          ],
          "selectBlank": false,
          "selectError": false
        }
      ],
      "mode": "row-based"
    },
    "description": "Remove rows"
  }
]

```


## Number of records found

The total number of records found after deduplication within ISRCTN by ISRCTN trial ID is the number of rows displayed by OpenRefine at this stage above the table. 


## Export of trial data

We are finished altering the data in the OpenRefine project. So we export an archival copy of the project as described above.

Now we can export the deduplicated guideline records.


### Export data to a RIS file using the Templating Export

For RIS file format see [Wikipedia](https://en.wikipedia.org/wiki/RIS_(file_format))

In the Templating Export form:

* Prefix: empty.
* Suffix: empty.
* Row Separator: Make sure to have an empty line in the so that the records are separated.
* Row Remplate:

```

TY  - WEB
TI  - {{cells["title"].value}}
PY  - {{cells["year"].value}}
UR  - {{cells["trial_url"].value}}
DO  - {{cells["DOI"].value}}
AB  - {{cells["abstract"].value}}
N1  - Mininmal data extracted from ISRCTN registry.
AN  - {{cells["isrctn_id"].value}}
DB  - ISRCTN registry
DP  - Springer Nature/BioMed Central
ER  - 

```

![Templating Export otions for RIS file format](media/ISRCTN/templating_export_RIS.png)

