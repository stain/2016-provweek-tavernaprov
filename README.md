# TavernaProv

_[Stian Soiland-Reyes](http://orcid.org/0000-0001-9842-9718),
[Carole Goble](http://orcid.org/0000-0003-1219-2137),
[Pinar Alper](http://orcid.org/0000-0002-2224-0780); [eScience Lab](http://www.esciencelab.org.uk/), University of Manchester_

<small><a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International License</a>.</small>

<!--
The following is a non-exhaustive list of topics for position statements reporting on experiences and impact:

    API and software that use PROV
    Datasets and resources that use PROV
    Impact of provenance
    Scalability
    Presentation and explanation of provenance to users
    Multi-level provenance (provenance of provenance)
    Tradeoff and choices of different serializations

The following is a non-exhaustive list of topics for position statements reporting on interoperability and requirements:

    Interoperability issues across serializations or within serializations
    Missing features, expressivity shortcomings
    Adoption hurdles
    Security and provenance, provenance and signatures
    Embedding provenance in various types of documents
    Graphical representation of provenance
    Inter-operability across standards
    Extensions of PROV for additional requirements in different domains and applications
    Abstraction of PROV records

Authors are strongly encouraged, where appropriate, to make an explicit link between requirements and application needs.

-->

[Apache Taverna](https://taverna.incubator.apache.org) is a scientific workflow
system for combining web services and local tools. Taverna
[records provenance](https://taverna.incubator.apache.org/documentation/provenance/) of
workflow runs, intermediate values and user interactions, both as an aid for
debugging while designing the workflow, but also as a record for later
reproducibility and comparison.


## Data bundle

A workflow run can be exported from Taverna as a
[workflow data bundle](https://github.com/apache/incubator-taverna-engine/tree/master/taverna-prov#structure-of-exported-provenance); a [Research Object bundle](https://w3id.org/bundle/)
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
is a generic model for describing provenance, and a scientific workflow
run with processors and data values naturally match the
[`Activity`]((https://www.w3.org/TR/prov-dm/#term-entity)) and [`Entity`](https://www.w3.org/TR/prov-dm/#term-entity) relations
[`wasGeneratedBy`](https://www.w3.org/TR/prov-dm/#dfn-wasgeneratedby) and
[`used`](https://www.w3.org/TR/prov-dm/#dfn-used).

However, using PROV-O to describe the details of a Taverna execution
meant a significant [increase in verbosity](https://github.com/apache/incubator-taverna-engine/blob/master/taverna-prov/example/helloanyone.bundle/workflowrun.prov.ttl).

To simplify query and interoperability with PROV tools, we declare
relations both with [starting point terms](https://www.w3.org/TR/prov-o/#description-starting-point-terms)
and as [qualified terms](https://www.w3.org/TR/prov-o/#description-qualified-terms),
e.g. to represent an `Activity` that
used different values as different input parameters, we provide
both a direct [used](http://www.w3.org/TR/prov-o/#used), but also a
[qualifiedUsage](http://www.w3.org/TR/prov-o/#qualifiedUsage) to a
[Usage](http://www.w3.org/TR/prov-o/#Usage) that specify [hadRole](http://www.w3.org/TR/prov-o/#hadRole) and [entity](http://www.w3.org/TR/prov-o/entity).

One interoperability design question for representing workflow runs is how much of the
workflow engine's internal logic and language should be explicit in the PROV
representation.  As we primarily wanted to convey what happened in the workflow
at the same granularity as its definition, we tried to hide provenance that
would be intrinsic to the Taverna Engine, e.g. [implicit iteration](http://taverna.knowledgeblog.org/2010/12/13/iteration-in-taverna-workflows/) is
not shown as a separate `Activity`. This mean TavernaProv might show
an Activity generating a list of entiries (a [PROV Dictionary](https://www.w3.org/TR/prov-dictionary/#dictionary-ontological-definition))
and the next Activity consume a value from that collection, seemingly generated out
of nowhere.

Keeping the link between the workflow definition and execution is essential to
understanding Taverna provenance, yet PROV don't describe the structure of a
[Plan](http://www.w3.org/TR/prov-o/#Plan). Taverna's workflow definition is in Linked Data using the
[SCUFL2](http://www.essepuntato.it/lode/owlapi/http://taverna.incubator.apache.org/ns/2010/scufl2.ttl) vocabulary, which include many
implementation details for the Taverna Engine,
and so forming a meaningful query like
_"What is the value made by calls to webservice X"_ means understanding the
whole conceptual model of Taverna workflow definitions.

Therefore Taverna's PROV export also include an annotation with the
[wfdesc](https://w3id.org/ro/2016-01-28/wfdesc/) abstraction of the
workflow definition, embedding user annotations and higher-level
information like [web service location](https://w3id.org/ro/2016-01-28/wf4ever/#rootURI). wfdesc deliberately leaves out execution details like iteration and parallelism controls.

Correspondingly the provenance trace
includes a [wfprov](https://w3id.org/ro/2016-01-28/wfprov/)
abstraction of the workflow execution, with direct shortcuts like
[describedByProcess](https://w3id.org/ro/2016-01-28/wfprov/#describedByProcess)
and
[describedByParameter](https://w3id.org/ro/2016-01-28/wfprov/#describedByParameter)
to bypass the indirection of PROV qualified terms; simplifying queries like
_"Which web service consumed value Y?"_.

The duality between wfdesc and wfprov is similar to the
"future provenance" model of [P-Plan](http://purl.org/net/p-plan) and [OPMW](ttp://www.opmw.org/model/OPMW/#WorkflowTemplateProcess)
and its [workflow templates](http://www.isi.edu/~gil/papers/garijo-etal-works14.pdf).


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



## Identifiers

A great advantage of using Linked Data was that we could use the same identifiers in all three formats. One challenge was that Taverna workflows are often run within a
desktop user interface or on the command line, and with privacy concerns
we didn't have the luxury of a server to mint URIs; we already [learnt our lesson with LSIDs](http://dev.mygrid.org.uk/blog/2016/02/what-exactly-happened-to-lsid/).

Taverna therefore generate UUID-based structured `http://` URIs within [our namespaces](http://ns.taverna.org.uk/),
e.g.
* http://ns.taverna.org.uk/2011/run/d5ee659e-e11e-43a5-bc0a-58d93674e5e2/process/1e027057-2aeb-47f7-97dc-03e19e9772be/
* http://ns.taverna.org.uk/2010/workflowBundle/2f0e94ef-b5c4-455d-aeab-1e9611f46b8b/workflow/HelloWorld/processor/hello/

The [scufl2-info](https://github.com/stain/scufl2-info)
web-service provide a minimal [JSON-LD](http://json-ld.org/) wfprov/wfdesc representation
identifying the URI as a provenance or workflow item, but (by design) not having access to the data bundle it can't say anything more.

## Interoperability issues

We found that our UUID identifiers don't play too well with
PROV Toolbox and the other
PROV formats, e.g in PROV-N, every URI ending in ``/`` is registered as
a separate namespace.

Similarly, OWL reasoning is not normally applied, so
even though wfprov extends PROV in its ontology,
we needed to include explicitly the
derived PROV-O statements.


## Future work

Facing the verbosity issue we are considering to split out wfprov statements
to a different file; as a ZIP archive the Taverna data bundle can contain
many provenance formats.

Similarly splitting out the details of
PROV-O qualified terms to a separate file is worth considering, this could also
improve PROV visualization of workflow provenance.

[Common Workflow Language](http://commonwl.org/) has created a
workflow language specification, a reference implementation and a large community of workflow systems developing
CWL support across bioinformatics, including Apache Taverna and Galaxy.

CWL is [planning its provenance format](https://github.com/common-workflow-language/common-workflow-language/issues/84)
based on PROV-O, wfprov and JSON-LD. As one of the CWL adopters, Apache Taverna
will naturally also support this provenance model.
