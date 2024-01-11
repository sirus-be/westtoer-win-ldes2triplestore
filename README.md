# Run

`docker-compose up`

Go to `http://localhost:3030/#/dataset/ds/query`

Example query:

```
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX geosparql: <http://www.opengis.net/ont/geosparql#>
PREFIX adres: <https://data.vlaanderen.be/ns/adres#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX generiek: <https://data.vlaanderen.be/ns/generiek#>
PREFIX logies: <https://data.vlaanderen.be/ns/logies#>
PREFIX log: <http://www.w3.org/2000/10/swap/log#>
PREFIX locn: <http://www.w3.org/ns/locn#>
PREFIX prov: <http://www.w3.org/ns/prov#>
PREFIX schema: <https://schema.org/>
PREFIX westtoerns: <https://westtoer.be/ns#>
PREFIX dcmitype: <http://purl.org/dc/dcmitype/>
PREFIX wgs84: <http://www.w3.org/2003/01/geo/wgs84_pos#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX adms: <http://www.w3.org/ns/adms#>

SELECT DISTINCT ?winId ?wijzigingsdatum ?beschrijving ?naam ?typeLabel ?status ?faciliteiten ?huisnummer ?straatnaam ?gemeente ?provincie ?postcode ?niscode ?website ?email ?telefoon ?lat ?long
WHERE {
    ?productVersie a schema:TouristAttraction .

    OPTIONAL {
    	?productVersie adms:identifier [ skos:notation ?winId ]
    }

    OPTIONAL {
        ?productVersie prov:generatedAtTime ?wijzigingsdatum .
    }
	
  	OPTIONAL {
    	?productVersie schema:amenityFeature [
        	schema:name ?faciliteit
    	]
  	}
  
  	OPTIONAL {
      SELECT DISTINCT ?productVersie (group_concat(?faciliteit;separator=', ') as ?faciliteiten)
         WHERE {
           ?productVersie schema:amenityFeature [
        		schema:name ?faciliteit
    		]
         } 
          GROUP BY ?productVersie
    }
  
  	OPTIONAL {
    	?productVersie schema:contactPoint [
        	foaf:page ?website ;
         	schema:email ?email ;
          	schema:telephone ?telefoon
    	]
  	}
  
    OPTIONAL {
    	?productVersie schema:additionalType/dcterms:isVersionOf ?type .
    	?type skos:prefLabel ?typeLabel .
    	FILTER (lang(?typeLabel) = 'nl')
    }

    OPTIONAL {
      ?productVersie westtoerns:Product.status/dcterms:isVersionOf/skos:prefLabel ?status .
      FILTER (lang(?status) = 'nl')
    }
	
  	OPTIONAL {
    	?productVersie locn:geometry [
        	wgs84:lat ?lat ;
        	wgs84:long ?long 
    	] .
  	}
  
    OPTIONAL {
        ?productVersie schema:name ?naam .
        FILTER (lang(?naam) = 'nl')
    }
  
  	OPTIONAL {
        ?productVersie schema:description ?beschrijving .
        FILTER (lang(?beschrijving) = 'nl')
    }

    OPTIONAL {
        ?productVersie locn:address [
        locn:thoroughfare ?straatnaam ;
        adres:Adresvoorstelling.huisnummer ?huisnummer ;
        locn:postCode ?postcode ;
        adres:gemeentenaam ?gemeente ;
        westtoerns:gemeenteniscode ?niscode ;
        ] .
    }
    OPTIONAL {
        ?productVersie locn:address [
        adres:Adresvoorstelling.busnummer ?busnummer
        ] .
    }
    OPTIONAL {
        ?productVersie locn:address [
        	locn:adminUnitL2 ?provincie
        ] .
    	FILTER (lang(?provincie) = "nl")
    }

    OPTIONAL {
        ?productVersie logies:behoortTotToeristischeRegio/skos:prefLabel ?toeristischeregio .
        FILTER (lang(?toeristischeregio) = 'nl')
    }

  # Filter op WinId
  # FILTER (str(?winId) = "1000309")
  
  # Enkel producttypes onder bepaald hoofdtype
  # ?parentType skos:prefLabel "Permanent Aanbod"@nl ;
  #           skos:narrower+ ?type .
}
```