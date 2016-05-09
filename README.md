# TavernaProv

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
system for combining web services and local tools. Taverna records provenance of
workflow runs, intermediate values and user interactions, both as an aid for
debugging while designing the workflow, but also as a record for later
reproducibility and comparison.

## Flexibility

Taverna 2 captures workflow run provenance using a layer in the processor [dispatch stack](http://www.taverna.org.uk/pages/wp-content/uploads/2010/04/T2Architecture.pdf#page6) and recorded in
[Apache Derby](http://db.apache.org/derby) SQL tables; this gave us
flexibility to later add PROV-O export as a separate Taverna plugin [TavernaProv](https://github.com/apache/incubator-taverna-engine/tree/master/taverna-prov),
which got included in Taverna 2.5 to replace the earlier [OPM](http://dev.mygrid.org.uk/wiki/display/tav250/Provenance+export+to+OPM+and+Janus)/ [JANUS](http://www.mygrid.org.uk/files/presentations/SP-IPAW10.pdf) prototype.

## TavernaProv

This gave flexibility in how Taverna export provenance, meaning we could
replace earlier prototypes exporting in with the [TavernaProv](https://github.com/apache/incubator-taverna-engine/tree/master/taverna-prov) plugin which export [PROV-O](https://www.w3.org/TR/prov-o/) within a
[Research Object bundle](https://w3id.org/bundle/), the
_workflow data bundle_.

Within the ZIP archive we add PROV in Turtle format, using
relative identifiers to embedded data files, in addition to
workflow definition (itself a Research Object) and its annotations.

A remote run on a
[Taverna Server](https://taverna.incubator.apache.org/download/server/) can
therefore be downloaded as a single file.

This archive can then be shared as a complete workflow run on
[myExperiment](http://myexperiment.org/) and imported to other user's
Taverna Workbench.

Using the Derby database was not ideal neither for the most common
provenance queries ("All values in a given processor") or for the PROV export,
plus the fact it was hard to get partial provenance from the Taverna Server,
in Apache Taverna 3 we changed provenance capture to a dynamically
updated [hierarchical object tree](https://taverna.incubator.apache.org/javadoc/taverna-engine/org/apache/taverna/platform/report/WorkflowReport.html) representing invocations of
workflows, processors (iterations) and at the deepest level
activities (service invocations), with the
intermediate values stored directly into the
Research object bundle during the run.

## Abstraction levels

_wfdesc, wfprov, tavernaprov_


## Identifiers

A great advantage of using Linked Data was that we could use the same identifiers in all three formats. One challenge was that Taverna workflows are often run within a
desktop user interface or on a local server, and with privacy concerns
we didn't have the luxury of a server to mint URIs.
(We [learnt our lesson with LSIDs](http://dev.mygrid.org.uk/blog/2016/02/what-exactly-happened-to-lsid/)).

Taverna therefore generate UUID-based structured URIs within [our namespaces](http://ns.taverna.org.uk/),
e.g. http://ns.taverna.org.uk/2011/run/d5ee659e-e11e-43a5-bc0a-58d93674e5e2/process/1e027057-2aeb-47f7-97dc-03e19e9772be/ - the [scufl2-info](https://github.com/stain/scufl2-info)
web-service provide a minimal [JSON-LD](http://json-ld.org/) representation of
the provenance.



A complete workflow run can then be exported as a
_workflow data bundle_, a specialization of the
.

We use

The landscape of Web Services and datasets in fields like bioinformatics is quite
dynamic - new reference data are available every day.
