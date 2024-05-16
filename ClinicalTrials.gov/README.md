ClinicalTrials.gov
==============================================================================

## Goals

The (as of 2024) new website of [ClinicalTrials.gov](https://www.clinicaltrials.gov/) has no export format for records in a format that can directly be imported into common reference management software. We want to run a search, create a search history and export the records found in RIS format.


## Searching in ClinicalTrials.gov

Advice on the search syntax of ClinicalTrials.gov is available in Table 2 of: Hunter K E, Webster A C, Page M J, Willson M, McDonald S, Berber S et al. Searching clinical trials registers: guide for systematic reviewers. _BMJ_ 2022; 377 :e068791 doi:[10.1136/bmj-2021-068791](https://doi.org/10.1136/bmj-2021-068791).

We will use the [expert search syntax](https://classic.clinicaltrials.gov/api/gui/ref/syntax) of ClinicalTrials.gov to search via the new [ClinicalTrials.gov API](https://www.clinicaltrials.gov/data-api/api).

Note (see [here](https://www.clinicaltrials.gov/data-api/api#refs)):

> Please note that the COVERAGE and EXPANSION operators are not fully implemented on the modernized ClinicalTrials.gov.

Here we use only a selection of the fields available for searching. This can be extended at any time.


### Population

Searching in the `Conditons/disease` field of the [search form](https://www.clinicaltrials.gov/) is equivalent to searching in the [ConditionSearch Area](https://www.clinicaltrials.gov/data-api/about-api/search-areas#ConditionSearch) with `AREA[ConditionSearch](<search terms>)` using the expert search syntax or using the `query.cond` parameter of the API as we do here.


### Intervention

Searching in `Intervention/treatment` of the [search form](https://www.clinicaltrials.gov/) is equivalent to searching in the [InterventionSearch Area](https://www.clinicaltrials.gov/data-api/about-api/search-areas#InterventionSearch) with `AREA[InterventionSearch](<search terms>)` using the expert search syntax or using the `query.intr` parameter of the API as we do here.

### Outcome

Searching in `Outcome measure` of the [search form](https://www.clinicaltrials.gov/) is equivalent to searching in the [OutcomeSearch Area](https://www.clinicaltrials.gov/data-api/about-api/search-areas#OutcomeSearch) with `AREA[OutcomeSearch](<search terms>)` using the expert search syntax or using the `query.outc` parameter of the API as we do here.

### Expert query syntax

It is also possible to filter with a query string in [Essie expression syntax](https://classic.clinicaltrials.gov/api/gui/ref/syntax). This will allow for advanced searching. The query string will be sent in the `filter.advanced` parameter of the API.

### Combination of concepts

The individual search components will be combined with the Boolean operator AND by specifying the respective query parameters for each search string in the API call.


## Prepared search strategy

We prepare the search strategy in an Excel file with only two rows. In the first row the API fields as given above are used as column headers. In the second row the search strings for the respective fields are entered.

**Remove the columns for fields that you do not want to search!** Empty search strings in a field may result in zero hits even though there are matching trial records when carrying out the same search in the search form in the browser.

[Here](data/ctgov_search_strategy_template.xlsx) is a template file.



## Process in OpenRefine

### Create a new project from an Excel file

From the [template file](data/ctgov_search_strategy_template.xlsx) we created an [Excel file with an example prepared search strategy](data/ctgov_prepared_search_strategy.xlsx). Note that we only have two search blocks und thus only two columns maust remain!


### Build the URL and run the search


```grel
"https://clinicaltrials.gov/api/v2/studies?" +
"pageSize=1000&" +
"countTotal=true&" +
forEach(
  row.columnNames,
  cn,
  cn + "=" + cells[cn].value.toString().strip().escape("url")
).join("&")
```

```json



```

### Search results

The query result contains the matching records up to the maximum of 1,000 records allowed by the API. Here we don't cover downloading more records in batches.

First we extract the result count from the JSON returned by the API with the following GREL code:

```grel
value.parseJson().totalCount
```

### Search history

At this time it may be convenient to export a search history.

Use the _Custom tabular exporter_, select as content all query parameters as imported from the Excel file `query.*`, the `query_url` and the `result_count` columns. Use the "Tab-separated values (TSV)" format in the download tab.

[Here](data/ctgov_search-history.tsv) is the search history table from our example search.


### Study records

Then we extract the `studies` array from the JSON and join all study records into a single string. We separate the study records with the string "|||DOCUMENT\_SEPARATOR|||":

```grel
value.parseJson().studies.join("|||DOCUMENT_SEPARATOR|||")
```

Then we split the multi-valued cell in the column `study_json` at the separator string "|||DOCUMENT\_SEPARATOR|||" that we introduced before. As a result we still have one record in OpenRefine but every study in JSON format in its own row. Change to "rows" view in OpenRefine to see that!

![Studies split into rows](media/rows_view.png)

Now we have to extract metadata from the `study_json` column that we can then reassemble into a RIS record.




