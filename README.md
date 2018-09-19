What they say: http://localhost:8088/api/vemonet/kraken-grlc-api/spec

Reality: http://localhost:8001/api/vemonet/kraken-grlc-api/spec



```shell
cd grlc
docker-compose -f docker-compose.yml up
```

**Swagger UI**: http://localhost:8001/api/vemonet/kraken-grlc-api/api-docs



Gene name: SLC7A3







```sql
#+ summary: Bio2RDF What drugs/compounds target gene products of [gene]?
#+ endpoint: http://bio2rdf.org/sparql
#+ tags:
#+   - Bio2RDF
#+ enumerate:   
#+   - gene
#+ method: GET
#+ pagination: 50

PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX db: <http://bio2rdf.org/drugbank_vocabulary:>
PREFIX hgnc: <http://bio2rdf.org/hgnc_vocabulary:>
PREFIX omim: <http://bio2rdf.org/omim_vocabulary:>
SELECT distinct ?omimGene ?geneName ?drug ?drugLabel
{
 ?drug a db:Drug .
 ?drug db:target ?target .
 ?drug rdfs:label ?drugLabel .
 ?target a db:Target .
 ?target db:gene-name ?geneName .
 FILTER regex(str(?geneName), "?_gene") .
 ?target db:x-hgnc ?_hgncGeneSymbol .
 ?hgncGeneSymbol hgnc:x-omim ?omimGene .
 ?omimGene a omim:Gene .
}
```

