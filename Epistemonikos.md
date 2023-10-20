Epistemonikos
==============================================================================

Issues:

* Website handles query strings entered in unpredictable ways. Sometimes unwanted field codes are added to the search strings. Various forms of a search string may be shown in the search forms. It is not always obvious what is actually searched. The URL shown in the browser is not a reliable reflection of what is going on in the web page.

* For title-and-abstract searches the search form acts differently from what is usually done in other interfaces. This may deviate from the intention of a searcher:

Entering a search string `concept_A AND concept_B` in the _Title/Abstract_ field of the advanced search form is translated into a search string `(title:(concept_A AND concept_B) OR abstract:(concept_A AND concept_B))`. This will not find records where _concept_A_ occurs in the title filed and _concept_B_ in the abstract field only. And vice versa. This may come unexpected.

Therefore, the following structure of the search strategy may be preferable but is not created by Epistemonikos' search form:

```
(title:(concept_A) OR abstract:(concept_A))
AND 
(title:(concept_B) OR abstract:(concept_B))
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

