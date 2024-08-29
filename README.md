# SPARQL examples

This is a collection of SPARQL examples usable on different SIB related SPARQL endpoints or datasets. The examples are stored one query per file in project specific repositories in the [examples](https://github.com/sib-swiss/sparql-examples/tree/master/examples) folder. 

Each SPARQL query is itself in a turtle file. We use the following ontologies for the basic concepts.

* ShACL for the relation to the text of the Select/Ask queries, and declaring prefixes
* RDFS for comments and labels as shown in the user interfaces
* RDF for basic type relations
* schema.org for the target SPARQL endpoint and tagging relevant keywords

The following illustrates an example to retrieve all taxa from the UniProt SPARQL endpoint.

```sparql
prefix ex: <https://sparql.uniprot.org/.well-known/sparql-examples/>  # <!-- change per dataset
prefix sh: <http://www.w3.org/ns/shacl#> 
prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
prefix rdfs:<http://www.w3.org/2000/01/rdf-schema#> 
ex:1  # <!-- UniProt, Rhea and Swiss-Lipids are numbered but this can be anything.
    a sh:SPARQLSelectExecutable, sh:SPARQLExecutable ;
    sh:prefixes _:sparql_examples_prefixes ; # <!-- required for the import of the prefix declarations. Note the blank node
    rdfs:comment """A comment <em>May have HTML in them</em>. Example: Select all taxa from the <a href=\"https://www.uniprot.org/taxonomy/\">UniProt taxonomy</a>"""^^rdf:HTML ;
    sh:select """PREFIX up: <http://purl.uniprot.org/core/>

SELECT ?taxon
FROM <http://sparql.uniprot.org/taxonomy>
WHERE
{
    ?taxon a up:Taxon .
}""" ;
    schema:target <https://sparql.uniprot.org/sparql/> ;
    schema:keywords "taxa".
```

# Quality Assurance (QA).

We use the [SIB SPARQL Examples utils](https://github.com/sib-swiss/sparql-examples-utils/) for testing

# Conversion for upload in SPARQL endpoint

To load the examples into a SPARQL endpoint they should be concatenated into one example file. Use the script `convertIntoOneTurtle.sh` provide the project name with a `-p` parameter

This expects the Jena tools to be available in your $PATH. e.g. `export PATH="$JENA_HOME/bin:$PATH"`

```bash
# e.g. make file examples_uniprot.ttl
./convertToOneTurtle.sh -p uniprot
```

Another option is to build the converter and use that.

```bash
wget 'https://github.com/sib-swiss/sparql-examples-utils/releases/download/v1.0.0/sparql-examples-util-1.0.0-uber.jar'
java -jar sparql-examples-util-1.0.0-uber.jar -i examples/ -p all -f jsonld
# Or for a specific example folder, as turtle, to a file:
java -jar sparql-examples-util-1.0.0-uber.jar -i examples/ -p Bgee -f ttl > examples_Bgee.ttl
```

## Conversion to RQ files

For easier use by other tools we can also generate [rq](https://www.w3.org/TR/2013/REC-sparql11-query-20130321/#mediaType) files. Following the syntax of [grlc](https://grlc.io/) allowing to use these queries as APIs.
```bash
wget 'https://github.com/sib-swiss/sparql-examples-utils/releases/download/v1.0.0/sparql-examples-util-1.0.0-uber.jar'
java -jar sparql-examples-util-1.0.0-uber.jar -i examples/ -p all -r
```

## Generate markdown file

Generate markdown files with the query and a mermaid diagram of the queries, to be used to deploy a static website for the query examples.

```bash
java -jar sparql-examples-util-*-uber.jar -i examples/ -m
```

# Querying for queries

As the SPARQL examples are themselves RDF, they can be queried for as soon as they are loaded in a SPARQL endpoint.
```sparql
PREFIX sh: <http://www.w3.org/ns/shacl#>
SELECT *
WHERE {
  ?ex sh:select|sh:ask|sh:construct|sh:describe ?query .
}
```

# Testing your queries

The queries are parsed and validated but not executed with junit using the Tester

```bash
wget 'https://github.com/sib-swiss/sparql-examples-utils/releases/download/v1.0.0/sparql-examples-util-1.0.0-uber.jar'
java -cp sparql-examples-util-1.0.0-uber.jar swiss.sib.rdf.sparql.examples.Tester --input-directory=./examples

```
should return no test failures. RDF4j and Jena are both a lot stricter than virtuoso.


# Labeling queries

If you want to add a label to a query please use [schema.org keyword](https://schema.org/keywords)

# Testing the queries actually work

The queries can be executed automatically on all endpoints they apply to using

```
mvn test -PallTests
```

This does change the queries to add a LIMIT 1 if no limit was set in the query. Then if there is a result it is fetched.

## Finding queries that run on more than one endpoint

```bash

./contvertAllToOneTurtle.sh -a 

sparql --data examples_all.ttl "SELECT ?query (GROUP_CONCAT(?target ; separator=', ') AS ?targets) WHERE { ?query <https://schema.org/target> ?target } GROUP BY ?query HAVING (COUNT(DISTINCT ?target) > 1) "
```

