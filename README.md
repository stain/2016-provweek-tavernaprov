# Tracking workflow execution with TavernaProv

* Authors: [Stian Soiland-Reyes](http://orcid.org/0000-0001-9842-9718),
[Pinar Alper](http://orcid.org/0000-0002-2224-0780),
[Carole Goble](http://orcid.org/0000-0003-1219-2137); [eScience Lab](http://www.esciencelab.org.uk/), University of Manchester
* Document id: https://github.com/stain/2016-provweek-tavernaprov/
* In reply to: [PROV: Three Years Later](http://provenanceweek.org/2016/p3yl/)
* Published: 2016-05-09
* Modified: 2016-05-11
* License:  <a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International License</a>
* Keywords:
  + scientific workflow
  + provenance
  + research object
  + reproducibility


<a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png" /></a>

[Apache Taverna](https://taverna.incubator.apache.org) [1] is a scientific workflow
system for combining web services and local tools. Taverna
[records provenance](https://taverna.incubator.apache.org/documentation/provenance/) [2] of
workflow runs, intermediate values and user interactions, both as an aid for
debugging while designing the workflow, but also as a record for later
reproducibility and comparison.

Taverna also records provenance of the evolution
of the workflow definition
(including a chain of `wasDerivedFrom` relations),
attributions and annotations; for brevity we here focus on
how Taverna's workflow run provenance extends PROV and
is embedded with Research Objects.

## Data bundle

A workflow run can be exported from Taverna as a
[workflow data bundle](https://github.com/apache/incubator-taverna-engine/tree/master/taverna-prov#structure-of-exported-provenance); a [Research Object bundle](https://w3id.org/bundle/) [3]
in the form of a ZIP archive that contains the
[workflow definition](https://taverna.incubator.apache.org/documentation/scufl2/)
(itself a Research Object), annotations, inputs, outputs and intermediate
values, and a [PROV-O](https://www.w3.org/TR/prov-o/) trace of the workflow
execution, showing every process execution within the workflow
run, linking to the produced and consumed values using relative paths.

A workflow run can thus be downloaded as a single file from a
[Taverna Server](https://taverna.incubator.apache.org/download/server/),
shared  ([myExperiment](http://myexperiment.org/),
[SEEK](http://seek4science.org/)), published  ([ROHub](http://www.rohub.org/),
[Zenodo](https://zenodo.org/)), imported in another
[Taverna Workbench](https://taverna.incubator.apache.org/download/workbench/), shown in the
[Databundle Viewer](https://github.com/apache/incubator-taverna-databundle-viewer)
or modified with the [DataBundle API](https://github.com/apache/incubator-taverna-language/tree/0.15.1-incubating/taverna-databundle).


## Abstraction levels

[PROV](https://www.w3.org/TR/prov-dm/)
is a generic model for describing provenance. While this means there are
generally many multiple ways to express the same history in PROV,
a scientific workflow run with processors and data values naturally match the
[`Activity`]((https://www.w3.org/TR/prov-dm/#term-entity)) and [`Entity`](https://www.w3.org/TR/prov-dm/#term-entity) relations
[`wasGeneratedBy`](https://www.w3.org/TR/prov-dm/#dfn-wasgeneratedby) and
[`used`](https://www.w3.org/TR/prov-dm/#dfn-used).

However, using PROV-O to describe the details of a Taverna execution
meant a significant [increase in verbosity](https://github.com/apache/incubator-taverna-engine/blob/master/taverna-prov/example/helloanyone.bundle/workflowrun.prov.ttl).  To simplify query and interoperability with PROV tools, we declare
relations both with [starting point terms](https://www.w3.org/TR/prov-o/#description-starting-point-terms)
and as [qualified terms](https://www.w3.org/TR/prov-o/#description-qualified-terms),
e.g. to represent an `Activity` that
used different values as different input parameters, we provide
both a direct [used](http://www.w3.org/TR/prov-o/#used), but also a
[qualifiedUsage](http://www.w3.org/TR/prov-o/#qualifiedUsage) to a
[Usage](http://www.w3.org/TR/prov-o/#Usage) that specify [hadRole](http://www.w3.org/TR/prov-o/#hadRole) and [entity](http://www.w3.org/TR/prov-o/entity).


PROV deliberately does not mandate how to make the
design decisions on
what activities, entities
and agents participate in a particular scenario,
but for interoperability purposes this
flexibility means that PROV is a kind of
"XML for provenance" - a common
language with a defined [semantics](https://www.w3.org/TR/prov-sem/),
but which can be applied in many different ways.

One interoperability design question for representing computational
workflow runs is how much of the
workflow engine's internal logic and language should be explicit in the PROV
representation. As we primarily wanted to convey what happened in the workflow
at the same granularity as its definition, we tried to hide provenance that
would be intrinsic to the Taverna Engine, e.g. [implicit iteration](http://taverna.knowledgeblog.org/2010/12/13/iteration-in-taverna-workflows/) is
not shown as a separate `Activity`.

However keeping provenance only at the dataflow level
(input/outputs of workflow processes) meant that TavernaProv
could not easily represent "deeper" provenance such as the
intermediate values of [while-loops](http://dev.mygrid.org.uk/wiki/display/tav250/Loops)
or intermittent failures that were automatically recovered by Taverna's [retry mechanism](http://dev.mygrid.org.uk/wiki/display/tav250/Retries), as we wanted to
avoid [unrolled workflow provenance](https://www.sci.utah.edu/publications/davidson07/davidson-provenance-lebull-07.pdf).


Keeping the link between the workflow definition and execution is essential to
understanding Taverna provenance, yet PROV doesn't describe the structure of a
[Plan](http://www.w3.org/TR/prov-o/#Plan). Taverna's workflow definition is in Linked Data using the
[SCUFL2](http://www.essepuntato.it/lode/owlapi/http://taverna.incubator.apache.org/ns/2010/scufl2.ttl) vocabulary, which includes many
implementation details for the Taverna Engine,
and so forming a meaningful query like
_"What is the value made by calls to webservice X"_ means understanding the
whole conceptual model of Taverna workflow definitions.

Therefore Taverna's PROV export also includes an
annotation with the  
[wfdesc](https://w3id.org/ro/2016-01-28/wfdesc/) [4] abstraction of the
workflow definition, embedding user annotations and higher-level
information like [web service location](https://w3id.org/ro/2016-01-28/wf4ever/#rootURI).
_wfdesc_ deliberately leaves out execution details like iteration and parallelism controls
as it primarily functions as a target for user-driven annotations about the
workflow steps.

Correspondingly the provenance bundle from TavernaPROV
includes a higher level [wfprov](https://w3id.org/ro/2016-01-28/wfprov/)
abstraction of the workflow execution, with direct shortcuts like
[describedByProcess](https://w3id.org/ro/2016-01-28/wfprov/#describedByProcess)
and
[describedByParameter](https://w3id.org/ro/2016-01-28/wfprov/#describedByParameter)
to bypass the indirection of PROV qualified terms; simplifying
[queries](https://github.com/apache/incubator-taverna-engine/tree/master/taverna-prov#querying-provenance) like
_"Which web service consumed value Y?"_.

The duality between wfdesc and wfprov is similar to the
"future provenance" model of [P-Plan](http://purl.org/net/p-plan) and [OPMW](ttp://www.opmw.org/model/OPMW/#WorkflowTemplateProcess)
and its [workflow templates](http://www.isi.edu/~gil/papers/garijo-etal-works14.pdf) [5],
and similarly the the split between  "prospective provenance"
and "retrospective provenance" of the
[ProvONE Data Model for Scientific Workflow Provenance](http://vcvcomputing.com/provone/provone.html). [6]

<!--
## Provenance capture

Taverna 2 captures workflow run provenance using a layer in the processor [dispatch stack](http://www.taverna.org.uk/pages/wp-content/uploads/2010/04/T2Architecture.pdf#page6) and recorded in
[Apache Derby](http://db.apache.org/derby) SQL tables; this gave us
flexibility to later add PROV-O export as a separate Taverna plugin [TavernaProv](https://github.com/apache/incubator-taverna-engine/tree/master/taverna-prov),
which got included in Taverna 2.5 to replace the earlier [OPM](http://dev.mygrid.org.uk/wiki/display/tav250/Provenance+export+to+OPM+and+Janus)/[JANUS](http://www.mygrid.org.uk/files/presentations/SP-IPAW10.pdf) prototype.

However using Derby database was not ideal; recording provenance in SQL
slowed down workflow execution significantly, and it meant it was hard
to transfer partial provenance from a remote Taverna Server without
repeatedly re-creating the PROV-O trace.

So in Apache Taverna 3 we changed provenance capture to a dynamically
updated [hierarchical object tree](https://taverna.incubator.apache.org/javadoc/taverna-engine/org/apache/taverna/platform/report/WorkflowReport.html) representing invocations of
a workflow, which invoked several processors, which iterated over
calls to service invocations (_activities_). Intermediate values are stored
directly into the data bundle during the run.
-->


## Identifiers and interoperability

A great advantage of using Linked Data was that we could use the same
identifiers in all three formats. One challenge was that Taverna workflows are often run within a
desktop user interface or on the command line, and with privacy concerns
we didn't have the luxury of a server to mint URIs; we already [learnt our lesson with LSIDs](http://dev.mygrid.org.uk/blog/2016/02/what-exactly-happened-to-lsid/) [7].

Taverna therefore generate UUID-based structured `http://` URIs within [our namespaces](http://ns.taverna.org.uk/),
e.g.:

* `http://ns.taverna.org.uk/2011/run/d5ee659e-e11e-43a5-bc0a-58d93674e5e2/process/1e027057-2aeb-47f7-97dc-03e19e9772be/`
* `http://ns.taverna.org.uk/2010/workflowBundle/2f0e94ef-b5c4-455d-aeab-1e9611f46b8b/workflow/HelloWorld/processor/hello/`

Resolving these URIs gives the [scufl2-info](https://github.com/stain/scufl2-info),
web-service, which provide a minimal [JSON-LD](http://json-ld.org/) wfprov/wfdesc representation
identifying the URI as a provenance or workflow item, but (by design) not having access to the data bundle it can't say anything more.


We found that our UUID-based URIs don't play too well with
[PROV Toolbox](http://lucmoreau.github.io/ProvToolbox/) and alternative
PROV formats like
[PROV-N](https://www.w3.org/TR/prov-n/) and
[PROV-XML](https://www.w3.org/TR/prov-xml/), as
every URI ending in ``/`` is registered as
a separate namespace in order to form valid QNames.
A suggested improvement for TavernaProv
is to generate [provly identifiers](https://github.com/lucmoreau/ProvToolbox/wiki/Mapping-PROV-Qualified-Names-to-xsd:QName#4-provly-identifiers), while remaining compliant with the [10 simple rules for identifiers](https://github.com/ResearchObject/identifier-rules). [8]

Similarly, [OWL reasoning](https://www.w3.org/TR/owl2-profiles/#Reasoning_in_OWL_2_RL_and_RDF_Graphs_using_Rules) is not generally applied by PROV-O consumers, so even though
_wfprov_ formally extends
PROV in its ontology definitions, we needed to add explicitly the
implied PROV-O statements in TavernaProv's Turtle output.


## Common Workflow Language

[Common Workflow Language](http://commonwl.org/) has created a
workflow language [specification](http://www.commonwl.org/draft-3/),
a reference implementation
[cwltool](https://github.com/common-workflow-language/cwltool),
and a large [community](http://www.commonwl.org/#Participating_Organizations)
of workflow system developers who are adding
CWL support across bioinformatics,
including Apache Taverna and Galaxy. Unlike wfdesc, OPMW and P-Plan, CWL workflows are
primarily intended to be executed,
with a strong emphasis on the dataflow between
command line tools packaged as [Docker](https://docker.com/) images.

CWL is specified using [Schema Salad](http://www.commonwl.org/draft-3/SchemaSalad.html),
which provides [JSON-LD](http://json-ld.org/) constructs in
[YAML](http://yaml.org/). The CWL dataflow
model is inspired by wfdesc and Apache Taverna and thus have similar
execution semantics and provenance requirements.
CWL is [planning its provenance format](https://github.com/common-workflow-language/common-workflow-language/issues/84)
based on PROV-O, wfprov and JSON-LD. As one of the CWL adopters, Apache Taverna
will naturally also aim to support the CWL provenance model.



## Future work

To face the verbosity issue, we are considering to split out wfprov statements
to a different file; as a ZIP archive the Taverna Data Bundle can contain
many provenance formats. Similarly splitting out the details using
PROV-O Qualified Terms to a separate file is worth considering, this could also
improve PROV visualization of workflow provenance.
Having such separate [PROV bundles](https://www.w3.org/TR/prov-dm/#component4)
would also make it easier for Taverna to support the ProvONE model
as an additional format.

[PROV Links](https://www.w3.org/TR/prov-links/) could be added to
Research Object Bundles to relate its data files to the
then multiple workflow provenance traces that describe their generation
and usage.

## References

1.  Katherine Wolstencroft, Robert Haines, Donal Fellows, Alan Williams, David Withers, Stuart Owen, Stian Soiland-Reyes, Ian Dunlop, Aleksandra Nenadic, Paul Fisher, Jiten Bhagat, Khalid Belhajjame, Finn Bacall, Alex Hardisty, Abraham Nieva de la Hidalga, Maria P. Balcazar Vargas, Shoaib Sufi, Carole Goble (2013): **The Taverna workflow suite: designing and executing workflows of Web Services on the desktop, web or in the cloud.** In: _Nucleic Acids Research_, **41**(W1): W557–W561. [doi:10.1093/nar/gkt328](http://dx.doi.org/10.1093/nar/gkt328)
2. Paolo Missier, Satya Sahoo, Jun Zhao, Carole Goble, Amit Sheth. (2010): **Janus: from Workflows to Semantic Provenance and Linked Open Data** in _Provenance and Annotation of Data and Processes, Third International Provenance and Annotation Workshop, (IPAW'10)_, 15–16 Jun 2010. Springer, Berlin: 129–141. [doi:10.1007/978-3-642-17819-1_16](http://dx.doi.org/10.1007/978-3-642-17819-1_16) [[pdf]](http://tw.rpi.edu/media/2013/12/31/96a5/IPAW2010_FP_Missier.pdf)
3. Stian Soiland-Reyes, Matthew Gamble, Robert Haines (2014): **Research Object Bundle 1.0**. _researchobject.org Specification_. [https://w3id.org/bundle/](https://w3id.org/bundle/) 2014-11-05. [doi:10.5281/zenodo.12586](http://dx.doi.org/10.5281/zenodo.12586)  
4. Khalid Belhajjame, Jun Zhao, Daniel Garijo, Matthew Gamble, Kristina Hettne, Raul Palma, Eleni Mina, Oscar Corcho, José Manuel Gómez-Pérez, Sean Bechhofer, Graham Klyne, Carole Goble (2015): **Using a suite of ontologies for preserving workflow-centric research objects**, _Web Semantics: Science, Services and Agents on the World Wide Web_. [doi:10.1016/j.websem.2015.01.003](http://dx.doi.org/doi:10.1016/j.websem.2015.01.003)
5. Daniel Garijo, Yolanda Gil, Oscar Corcho (2014): **Towards workflow ecosystems through semantic and standard representations**. _Proceeding
WORKS '14 Proceedings of the 9th Workshop on Workflows in Support of Large-Scale Science_. [doi:10.1109/WORKS.2014.13](http://dx.doi.org/10.1109/WORKS.2014.13) [[pdf]](http://conferences.computer.org/works/2014/papers/7067a094.pdf)
6. Víctor Cuevas-Vicenttín, Parisa Kianmajd, Bertram Ludäscher, Paolo Missier, Fernando Chirigati, Yaxing Wei, David Koop, Saumen Dey (2014): **The PBase Scientific Workflow Provenance Repository**. _International Journal of Digital Curation_ **9**(2). [doi:10.2218/ijdc.v9i2.332](http://dx.doi.org/10.2218/ijdc.v9i2.332)
7. Stian Soiland-Reyes, Alan R Williams (2016): **What exactly happened to LSID?** _myGrid developer blog_, 2016-02-26. [http://dev.mygrid.org.uk/blog/2016/02/what-exactly-happened-to-lsid/](http://dev.mygrid.org.uk/blog/2016/02/what-exactly-happened-to-lsid/) [doi:10.5281/zenodo.46804](http://dx.doi.org/10.5281/zenodo.46804)
8. Julie A McMurry, Niklas Blomberg, Tony Burdett, Nathalie Conte, Michel Dumontier, Donal Fellows, Alejandra Gonzalez-Beltran, Philipp Gormanns, Janna Hastings, Melissa A Haendel, Henning Hermjakob, Jean-Karim Hériché, Jon C Ison, Rafael C Jimenez, Simon Jupp, Nick Juty, Camille Laibe, Nicolas Le Novère, James Malone, Maria Jesus Martin, Johanna R McEntyre, Chris Morris, Juha Muilu, Wolfgang Müller, Christopher J Mungall, Philippe Rocca-Serra, Susanna-Assunta Sansone, Murat Sariyar, Jacky L Snoep, Natalie J Stanford, Neil Swainston, Nicole L Washington, Alan R Williams, Katherine Wolstencroft, Carole Goble, Helen Parkinson (2015): **10 Simple rules for design, provision, and reuse of identifiers for web-based life science data**. _Zenodo_. Submitted to PLoS Computational Biology.
[doi:10.5281/zenodo.31765](http://dx.doi.org/10.5281/zenodo.31765)
