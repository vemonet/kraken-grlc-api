### Equivalent query on Bio2RDF

```sql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX db: <http://bio2rdf.org/drugbank_vocabulary:>
PREFIX hgnc: <http://bio2rdf.org/hgnc_vocabulary:>
PREFIX omim: <http://bio2rdf.org/omim_vocabulary:>
SELECT distinct ?hgncGeneSymbol ?omimGene ?geneName ?drug ?drugLabel
{
 ?drug a db:Drug .
 ?drug db:target ?target .
 ?drug rdfs:label ?drugLabel .
 ?target a db:Target .
 ?target db:gene-name ?geneName .
 FILTER regex(str(?geneName), "SLC7A3") .
 ?target db:x-hgnc ?hgncGeneSymbol .
 ?hgncGeneSymbol hgnc:x-omim ?omimGene .
 ?omimGene a omim:Gene .
}
```



### Construct query Drugbank

```sql
PREFIX x2rm: <http://ids.unimaas.nl/rdf2xml/model/>
PREFIX ido:<http://identifiers.org/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX bioentity: <http://bioentity.io/vocab/>
CONSTRUCT 
{ 
    ?drugUri rdfs:label ?drugName .
    ?drugUri dcterms:identifier ?dbid .
    ?drugUri obo:IAO_0000115 ?drugDescription.
    ?drugUri a bioentity:Drug .
    
    ?drugUri bioentity:affects ?geneProductUri .
    
    ?geneProductUri a bioentity:GeneProduct .
    ?geneProductUri rdfs:label ?targetLabel .
    ?geneProductUri dcterms:identifier ?targetId .
    
    ?hgncUri a bioentity:Gene .
    ?hgncUri bioentity:id ?hgncUri .
    ?hgncUri bioentity:has_gene_product ?geneProductUri
    
    }
WHERE {
select distinct ?dbid ?drugUri ?drugName ?targetId ?targetLabel ?drugDescription ?drugObj ?hgncIdentifier ?identifierObj ?hgncUri ?geneProductUri where { 
    ?drugObj a x2rm:drugbank\/drug .
    ?drugObj x2rm:hasChild ?dbidObj .
    ?drugObj x2rm:hasChild ?drugNameObj .
    ?drugObj x2rm:hasChild ?drugDescriptionObj .
    BIND ( iri(concat("http://identifiers.org/drugbank:", ?dbid)) AS ?drugUri )
    
    # Get DrugBank ID (only primary)
	?dbidObj a x2rm:drugbank\/drug\/drugbank-id .
    ?dbidObj x2rm:hasAttribute ?isPrimary .
    ?dbidObj x2rm:hasValue ?dbid .
    ?dbidObj x2rm:hasXPath ?xpath .
    ?isPrimary a x2rm:drugbank\/drug\/drugbank-id\/\@primary .
    
    # Get the drug name and description
    ?drugNameObj a x2rm:drugbank\/drug\/name .
    ?drugNameObj x2rm:hasValue ?drugName .
    ?drugDescriptionObj a x2rm:drugbank\/drug\/description .
    ?drugDescriptionObj x2rm:hasValue ?drugDescription .
    
    # Get the drugs target
    ?drugObj x2rm:hasChild ?targetsObj .
    ?targetsObj a x2rm:drugbank\/drug\/targets .
    ?targetsObj x2rm:hasChild ?targetObj .
    ?targetObj a x2rm:drugbank\/drug\/targets\/target .
    
    ?targetObj x2rm:hasChild ?targetIdObj .
    ?targetIdObj a x2rm:drugbank\/drug\/targets\/target\/id  .
    ?targetIdObj x2rm:hasValue ?targetId .
    BIND ( iri(concat("http://identifiers.org/drugbank:", ?targetId)) AS ?geneProductUri )
    
    ?targetObj x2rm:hasChild ?targetLabelObj .
    ?targetLabelObj a x2rm:drugbank\/drug\/targets\/target\/name  .
    ?targetLabelObj x2rm:hasValue ?targetLabel .
    
    # Get the HGNC identifier for the gene coding for the gene product which is in target/polypeptide/external-identifier
    ?targetObj x2rm:hasChild ?polypeptideObj .
    ?polypeptideObj a x2rm:drugbank\/drug\/targets\/target\/polypeptide .
    ?polypeptideObj x2rm:hasChild ?extIdentifiersObj .
    ?extIdentifiersObj a x2rm:drugbank\/drug\/targets\/target\/polypeptide\/external-identifiers .   
    ?extIdentifiersObj x2rm:hasChild ?extIdentifierObj .
    ?extIdentifierObj a x2rm:drugbank\/drug\/targets\/target\/polypeptide\/external-identifiers\/external-identifier .
    ?extIdentifierObj x2rm:hasChild ?identifierObj .
    ?identifierObj a x2rm:drugbank\/drug\/targets\/target\/polypeptide\/external-identifiers\/external-identifier\/identifier .
    ?identifierObj x2rm:hasValue ?hgncIdentifier .
    FILTER( REGEX( ?hgncIdentifier, "^HGNC:" ) ) .
    BIND ( iri(concat("http://identifiers.org/", lcase(?hgncIdentifier))) AS ?hgncUri )
} 
}
```



### SPARQL construct HGNC

```sql
PREFIX kraken:<http://kraken//data/ncats/hgnc/hgnc_complete_set.ts/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX bioentity: <http://bioentity.io/vocab/>
PREFIX dcterms: <http://purl.org/dc/terms/>
CONSTRUCT
{
   ?hgncUri a bioentity:Gene.
   ?hgncUri rdfs:label ?geneName .
   ?hgncUri dcterms:identifier ?hgncid.
   ?hgncUri bioentity:id ?hgncUri .
   ?hgncUri bioentity:systematic_synonym ?symbol .

}
WHERE {SELECT ?s ?hgncid ?geneName ?symbol ?hgncUri
   {
     ?s kraken:ApprovedName ?geneName .
      ?s kraken:HgncId ?hgncid.
      ?s kraken:ApprovedSymbol ?symbol.
     ?s ?p ?o .
      BIND ( iri(concat("http://identifiers.org/", lcase(?hgncid))) AS ?hgncUri )
   }
}
```



### Get NCATS answer

What drugs/compounds target gene products of [gene]?

```sql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX bioentity: <http://bioentity.io/vocab/>
SELECT distinct ?gene ?geneLabel ?geneProductLabel ?drug ?drugLabel
{
    ?gene a bioentity:Gene .
    ?gene bioentity:id ?geneId . 
    ?gene rdfs:label ?geneLabel .
    ?gene bioentity:has_gene_product ?geneProduct .
    ?geneProduct rdfs:label ?geneProductLabel .
    ?drug bioentity:affects ?geneProduct .
    ?drug a bioentity:Drug .
    ?drug rdfs:label ?drugLabel .
   
 #FILTER regex(str(?geneName), "SLC7A3") .
}
```