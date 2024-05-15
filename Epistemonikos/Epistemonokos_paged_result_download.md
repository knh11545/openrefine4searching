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
