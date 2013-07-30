FOAF Site Integration In Stanbol
===============================

This project includes the results of the FOAF (Friend-of-a-Friend) ReferenceSite created for Stanbol. 
It also provides the necessary steps to configure the foaf-site in Stanbol entityhub and use it in the enhancement phase as an EntityHubLinking Engine to enhance the content.
Below sections will take you through a step by step guide on the FOAF site integration process

Selection of a FOAF Datasource
-------------------------------
FOAF data is mainly provided by Linked-Data projects. There are several datasources mentioned in the FOAF project wiki [1], most of them are social networking sites offering their data in FOAF format. However most of the projects are
out of date therefore it was not recommended to use them as the datasources for my project. The 2 best options were; <br/>

1. The billion-tripple challenge (btc) 2012 project [2] : <br/>
   A web-crawled dataset including data from dbpedia, freebase, datahub, timbl, rest datasources. Quantity wise this has a sufficient amount (1436545545 quads) of data, foaf data and it's fairly upto date. 
2. WebDataCommons project [3] : <br/>
   A linked-data project which has a dataset (1079175202 quads) created in August 2012. But the sources of the data is not specified in the project.

After a discussion with Stanbol community and other related FOAF communities I selected the btc2012 dataset as it has a sufficiently up-to-date FOAF dataset. Following section will describe how I developed a ReferenceSite in Stanbol project with the selected dataset.


Creating a ReferenceSite with FOAF dataset
------------------------------------------

For this purpose I used the generic-rdf indexing tool in Stanbol project. Some of the tasks such as FOAF filtering required additional configuration files to be copied to the tool from other sources. Below guide will explain how to develop a FOAF datasite as a custom vocabulary integration in Stanbol.

###Building the indexing tool 
The generic-rdf indexing tool can be found in the Stanbol trunk at [4]. Build it from source using <code>mvn clean install</code>. This will create the <code>org.apache.stanbol.entityhub.indexing.genericrdf-0.12.0-SNAPSHOT.jar</code> file in the target. Then intialize the tool with the below command : <br/>
 
<code>java -jar org.apache.stanbol.entityhub.indexing.genericrdf-0.12.0-SNAPSHOT.jar init</code> <br/>
Above initialization command will create the indexing tool directories for various purposes in the indexing process. The main directories are as below:
<pre>
/indexing
	/config {the main configuration directory}
	/destination {the target directory of Solr indexing files and extracted entity data}
	/dist  {the results of the indexing process including a reference-site data-file and solr-index}
	/resources {the rdf datasources to be used for the indexing process}
</pre>

For demo purpose I have uploaded the pre-built jar file and the indexing directories with init command executed. 
The uploaded files here under directory: generic-rdf/indexing are pre-configured with the required configurations to execute FOAF filtering and indexing.
Below steps will describe each configuration done to achieve FOAF filtering on the used btc2012 dataset. <br/>



###Configuring the tool to filter foaf entities 
indexing/config is the main configuration directory of the tool and the main configuration file is <code>indexing.properties</code>. <br/>
To give a unique name to the EntityHub site, set the 'name' value in indexing.properties to a suitable unique Site name (eg: foaf-site ) <br/> 
The FOAF filtering configurations require to edit the EntityDataIterable field to support FOAF entity iterations as below. <br/>
<code>entityDataIterable=org.apache.stanbol.entityhub.indexing.source.jenatdb.RdfIndexingSource,config:indexingsource,bnode:true</code><br/>(Please note the additional bnode:true parameter above is activated to process blank nodes in the dataset)<br/>

Above entityDataIterable configuration requires 2 additional configuration files : <code>indexingsource.properties</code> and <code>propertiyfilter.config</code>. These files are not included in generic-rdf index tool by default. 
You can use the 2 files used in freebase indexing tool at [5] for filtering purpose. 
Copy the 2 files into indexing/config and add the below entry to propertyfilter.config: <br/>
<code>foaf:*</code><br/>
Above entry instructs the tool to filter entities from the datasource which defines some foaf property in foaf namespace. <br/>

To index only <code>foaf:Person</code> and <code>foaf:Organization</code> type entities, activate 'values' in <code>entityTypes.properties</code> file as below: <br/>
<code>values=foaf:Person;foaf:Organization</code> <br/>
Check above entity filtering in entityTypes.properties is enabled in indexing.properties as a entityProcessor by searching for below entry. <br/>
<code>entityProcessor=org.apache.stanbol.entityhub.indexing.core.processor.FieldValueFilter,config:entityTypes;</code><br/>

To match entity-mentions in the content and link them to Entities in the FOAF dataset, certain foaf properties should be identified as the fields to map entities and copy them as label fields in the entityhub. For this purpose I have used foaf fields like foaf:name, firstName, givenName as label fields. These entries should be configured in the <code>mappings.txt</code> as below;<br/>

<pre>
foaf:name > rdfs:label
foaf:nick > rdfs:label
foaf:givenName > rdfs:label
foaf:familyName > rdfs:label
foaf:firstName > rdfs:label	
</pre>

In the enhancement phase, to traverse between entities, the Stanbol engine uses the redirect field. In FOAF there are 2 main fields to link similar/related entities. They are <code>rdfs:seeAlso</code> and <code>owl:sameAs</code>. To use both of them as redirect fields in Stanbol engines, they have to be converged as Stanbol only allows 1 redirect field. Therefore I will merge both these fields into Stanbol internally used <code>fise:redirects</code> and used as the single redirect field in the linking engine configuration explained later.<br/>

Following are the extra configurations to be added to mappings.txt in the indexing tool:<br/>
<pre>
rdfs:seeAlso | d=entityhub:ref
owl:sameAs | d=entityhub:ref

rdfs:seeAlso > fise:redirects
owl:sameAs > fise:redirects
</pre><br/>

###Running the Indexing Tool and Deploying the FOAF dataset to Stanbol
Now all the necessary configurations to index and filter a FOAF dataset is done. So you can run the indexing tool using below command:<br/>
<code>java -Xmx1024m -jar  org.apache.stanbol.entityhub.indexing.genericrdf-0.12.0-SNAPSHOT.jar index</code> <br/>

Above will execute the entity extraction and indexing process and create 2 files in the indexing/dist directory.
Copy the generated <code>org.apache.stanbol.data.site.foaf-site-1.0.0.jar</code> to ${stanbol-server}/fileinstall directory.
Copy the generated <code>foaf-site.solrindex.zip</code> to ${stanbol-server}/datafiles directory. <br/>

Launch Stanbol server using full-launcher and access the foaf-site at : localhost:8080/entityhub/site/foaf-site
The next step is to create an Enhancement Engine in Stanbol utilizing above created FOAF Site.

###Configuring an Enhancement Engine and a Enhancement Chain using the FOAF Site
After successfully deploying the foaf-site, I configured an enhancement chain to perform content enhancements using above ceated foaf-site. Most of these configurations can be done via the osgi console configuration manager of Apache stanbol accessible at : http://localhost:8080/system/console/configMgr <br/>

Following are the enhancement engine configurations required to create a FOAF site linking engine. <br/>

* Configure a new entityhub-linking-engine [6] with below configuration changes: <br/>
<pre>
Name : foaf-site-linking
Referenced site : foaf-site
Redirect field : fise:redirects
Case sensitivity : disabled
</pre>

* Configure a weighted enhancement chain [7] using above created foaf-site-linking engine by doing below configuration changes. In the enhancement-chain I have added several available engines to perform language detection and natural language processing prior to foaf-linking: <br/>
<pre>
Name : foaf-site-chain
Engines : langdetect, opennlp-sentence, opennlp-token, opennlp-pos, foaf-site-linking
</pre>

Now you can invoke the new foaf-site-chain by going to : http://localhost:8080/enhancer/chain/foaf-site-chain
and giving a test content like : "Tim Bernes Lee is the inventor of World Wide Web"

If the configurations are done correctly <code>Timb Berness Lee</code> and <code>World Wide Web</code> should be identified as entities from the foaf-site dataset. Please refer the screen-shot image attached here with the demo results. This foaf-site-linking engine will be used as the base of the foaf-disambiguation engine to be created in the 2nd phase of the GSOC project. <br/>


[1] http://www.w3.org/wiki/FoafSites <br/>
[2] http://km.aifb.kit.edu/projects/btc-2012/ <br/>
[3] http://webdatacommons.org/ <br/>
[4] https://svn.apache.org/repos/asf/stanbol/trunk/entityhub/indexing/genericrdf<br/>
[5] https://svn.apache.org/repos/asf/stanbol/trunk/entityhub/indexing/freebase <br/>
[6] https://stanbol.apache.org/docs/trunk/components/enhancer/engines/entityhublinking <br/>
[7] http://stanbol.apache.org/docs/trunk/components/enhancer/chains/weightedchain.html
