FOAF Site
========

This project includes the results of the FOAF (Friend-of-a-Friend) ReferenceSite created for Stanbol. 
It also provides the necessary steps to configure the foaf-site in Stanbol entityhub and use it in the enhancement phase as an EntityHubLinking Engine.
Following section provides an introduction of the task.


Selection of a FOAF Datasource
----------------------------
FOAF data is mainly provided by Linked-Data projects. There are several datasources mentioned in the FOAF project wiki [1], most of them are social networking sites offering their data in FOAF format. However most of the projects are
out of date therefore it was not recommended to use them as the datasources for my project. The 2 best options were

1. The billion-tripple challenge 2012 project [2] : <br/>
   A web-crawled dataset including data from dbpedia, freebase, datahub, timbl, rest datasources. Quantity wise this has a sufficient amount (1436545545 quads) of data, foaf data and it's fairly upto date. 
2. WebDataCommons project [3] : <br/>
   A linked-data project which has a dataset (1079175202 quads) created in August 2012. But the sources of the data is not specified in the project










[1] http://www.w3.org/wiki/FoafSites <br/>
[2] http://km.aifb.kit.edu/projects/btc-2012/ <br/>
[3] http://webdatacommons.org/
