# Setup

`docker-compose up`

Om een clean heropstart te doen, doe:

```
docker-compose down -v
docker-compose up
```

## Overzicht van producten

Ga naar `http://localhost:3030/#/dataset/ds/query`

Voorbeeld-query:

```
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
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
PREFIX datatourism: <https://www.datatourisme.fr/ontology/core#>

SELECT DISTINCT ?winId ?wijzigingsdatum ?beschrijving ?naam ?typeId ?typeLabels ?rootTypeId ?statusId ?status ?faciliteiten ?afbeeldingURL ?huisnummer ?straatnaam ?gemeente ?provincie ?postcode ?niscode ?website ?email ?telefoonnummers ?lat ?long ?toeristischeregioWesttoerId ?toeristischeregioTVLId ?toeristischeregioLabel ?product ?wkt ?beoordelingsBeschrijving ?hoogsteBeoordeling ?laagsteBeoordeling ?beoordelingsId ?validFrom ?validThrough ?opens ?closes ?dayOfWeekString ?linkUrlString ?linkTypeId ?hoogteRuimte ?oppervlakteRuimte ?indelingCapaciteit ?indelingTypeId
WHERE {
    GRAPH <urn:x-arq:DefaultGraph> {
      ?product a schema:TouristAttraction .

      ?product adms:identifier [ skos:notation ?winId ] .

      ?product prov:generatedAtTime ?wijzigingsdatum .

      OPTIONAL {
      	  ?product logies:heeftMedia/schema:contentUrl ?afbeeldingURL .
      }

      OPTIONAL {
          ?product schema:amenityFeature [
              schema:name ?faciliteit
          ]
      }

      OPTIONAL {
        SELECT DISTINCT ?product (group_concat(?faciliteit;separator=', ') as ?faciliteiten)
           WHERE {
             ?product schema:amenityFeature [
                  schema:name ?faciliteit
              ]
         FILTER(lang(?faciliteit) = 'nl')
           } 
            GROUP BY ?product
      }

      OPTIONAL {
          ?product schema:contactPoint [
              foaf:page ?website ;
              schema:email ?email 
          ]
      }
    
    	OPTIONAL {
          SELECT DISTINCT ?product (group_concat(?telefoon;separator=', ') as ?telefoonnummers)
             WHERE {
               ?product schema:contactPoint/schema:telephone ?telefoon
             } 
              GROUP BY ?product
        }

      OPTIONAL {
          ?product schema:additionalType ?type .
          ?type skos:prefLabel ?typeLabel .
          FILTER (lang(?typeLabel) = 'nl')
      	  BIND(replace(str(?type), 'https://westtoer.be/id/concept/producttype/', '') as ?typeId)

     	  # Retrieve id of root type, e.g. 2e577149-7520-450a-9de6-824cd5d8f652 for "Tijdelijk aanbod"	
          ?rootType skos:narrower+ ?type .
          BIND(replace(str(?rootType), 'https://westtoer.be/id/concept/producttype/', '') as ?rootTypeId)
          FILTER NOT EXISTS {
      	    ?parentOfRootParentType skos:narrower ?rootType .
          }
      }
    
      OPTIONAL {
          SELECT DISTINCT ?product (group_concat(?typeLabel;separator=', ') as ?typeLabels)
             WHERE {
               ?product schema:additionalType ?type .
              ?type skos:prefLabel ?typeLabel .
              FILTER (lang(?typeLabel) = 'nl')
             } 
              GROUP BY ?product
        }

      ?product westtoerns:Product.status/skos:prefLabel ?status .
      FILTER (lang(?status) = 'nl')
      ?product westtoerns:Product.status ?statusUri .
      BIND(replace(str(?statusUri), 'https://westtoer.be/id/concepts/', '') as ?statusId)

      OPTIONAL {
          ?product locn:geometry [
              wgs84:lat ?lat ;
              wgs84:long ?long 
          ] .
          BIND (STRDT(CONCAT('POINT(', str(?long), ' ', str(?lat), ')'), <http://www.opengis.net/ont/geosparql#wktLiteral>) as ?wkt)
      }

      OPTIONAL {
          ?product schema:name ?naam .
          FILTER (lang(?naam) = 'nl')
      }

      OPTIONAL {
          ?product schema:description ?beschrijving .
          FILTER (lang(?beschrijving) = 'nl')
      }

      OPTIONAL {
          ?product locn:address [
          locn:thoroughfare ?straatnaam ;
          adres:Adresvoorstelling.huisnummer ?huisnummer ;
          locn:postCode ?postcode ;
          adres:gemeentenaam ?gemeente ;
          westtoerns:gemeenteniscode ?niscode ;
          ] .
      }
      OPTIONAL {
          ?product locn:address [
          adres:Adresvoorstelling.busnummer ?busnummer
          ] .
      }
      OPTIONAL {
          ?product locn:address [
              locn:adminUnitL2 ?provincie
          ] .
          FILTER (lang(?provincie) = "nl")
      }

      OPTIONAL {
          ?product logies:behoortTotToeristischeRegio ?toeristischeregio .
          ?toeristischeregio owl:sameAs ?toeristischeregioTVL .
          BIND (str(?toeristischeregioTVL) as ?toeristischeregioTVLId) .
          ?toeristischeregio skos:prefLabel ?toeristischeregioLabel .
          FILTER (lang(?toeristischeregioLabel) = 'nl')

          BIND(replace(str(?toeristischeregio), 'https://westtoer.be/id/toeristischeregio/', '') as ?toeristischeregioWesttoerId)
      }

      OPTIONAL {
          ?product schema:starRating ?beoordeling .
          ?beoordeling schema:description ?beoordelingsBeschrijving .
          FILTER(lang(?beoordelingsBeschrijving) = 'nl')
          OPTIONAL {
            ?beoordeling schema:worstRating ?laagsteBeoordeling .
          }
          OPTIONAL {
            ?beoordeling schema:bestRating ?hoogsteBeoordeling .
          }
          OPTIONAL {
            ?beoordeling schema:ratingValue ?beoordelingsId .
          }
      }

     OPTIONAL {
        ?product westtoerns:heeftRuimte ?ruimte .
        OPTIONAL {
      		?ruimte schema:floorSize/schema:value ?oppervlakteRuimte .
        }
        OPTIONAL {
        	?ruimte schema:height/schema:value ?hoogteRuimte .
        }
        OPTIONAL {
        	?ruimte datatourism:hasLayout ?indeling .
            ?indeling westtoerns:indelingBeschikbaar true .
        	?indeling dcterms:type ?indelingTypeUri .
            BIND(replace(str(?indelingTypeUri), 'https://westtoer.be/id/concepts/', '') as ?indelingTypeId)
			?indeling logies:capaciteit/schema:value ?indelingCapaciteit .            
        }
     }

     OPTIONAL {
	  ?product schema:contactPoint/schema:hoursAvailable ?openinghoursSpecification .
	  OPTIONAL {
	    ?openinghoursSpecification schema:opens ?opens .
	  } 
	  OPTIONAL {
	    ?openinghoursSpecification schema:closes ?closes .
	  }
	  OPTIONAL {
	    ?openinghoursSpecification schema:dayOfWeek ?dayOfWeek .
		BIND(str(?dayOfWeek) as ?dayOfWeekString)
	  }
	  OPTIONAL {
	    ?openinghoursSpecification schema:validFrom ?validFrom .
	  }
	  OPTIONAL {
	    ?openinghoursSpecification schema:validThrough ?validThrough .
	  }
      }

    OPTIONAL {
       ?product rdfs:seeAlso ?link .
       ?link schema:additionalType ?linkTypeUri .
       BIND(replace(str(?linkTypeUri), 'https://westtoer.be/id/concepts/', '') as ?linkTypeId)
       ?link schema:url ?linkUrl .
       BIND(str(?linkUrl) as ?linkUrlString)
    }
        
    # Filter op WinId
    # FILTER (str(?winId) = "1000309")

    # Filter op productstatus
    # FILTER (str(?status) = "Goedgekeurd")
    # FILTER (str(?statusId) = "f6c056f4-5db9-4030-a1f5-7727cd475715")

    # Filter op regio
    # FILTER (str(?toeristischeregioWesttoerId) = "158cd294-810e-4211-9a2d-5dcb799d0554")

    # Filter op links om te boeken
    # FILTER (?linkTypeId = "343d08a7-f435-427a-8ef6-c2bd6187c856")

     # Enkel producttypes onder "Logies"
    #?parentType skos:prefLabel "Logies"@nl ;
    #           skos:narrower+ ?type .

    # Enkel producttypes onder "Eet- en drinkgelegenheden"
    #?parentType skos:prefLabel "Eet- en drinkgelegenheden"@nl ;
    #           skos:narrower+ ?type .

    # Enkel producttypes onder "MICE"
    #?parentType skos:prefLabel "MICE"@nl ;
    #           skos:narrower+ ?type .

    # Enkel producttypes onder "Permanent Aanbod"
    #?parentType skos:prefLabel "Permanent Aanbod"@nl ;
    #           skos:narrower+ ?type .

    # Enkel producttypes onder "Tijdelijk aanbod"
    #?parentType skos:prefLabel "Tijdelijk aanbod"@nl ;
    #           skos:narrower+ ?type .

    # Wijzingen sedert timestamp
    # FILTER (?wijzigingsdatum >= "2024-02-05T18:07:52Z"^^<http://www.w3.org/2001/XMLSchema#dateTime>)

    # Filteren op NIScode
    # FILTER(?niscode = 35013)
  }
}
LIMIT 500
```

Onderaan de query kan gefilterd worden op WIN ID en producttypes dat onder "Permanent Aanbod" vallen.
Verwijder het spoorwegteken om uit commentaar te zetten.

Opmerkingen:
* Deze resultaten kunnen verwerkt worden als JSON (sparql11-results-json). Zie meer info [hier](https://www.w3.org/TR/sparql11-results-json/).
* Er komt een punt dat er te veel producten zijn om in 1 HTTP response te passen. Dan dient er gepagineerd te worden met LIMIT en OFFSET. Zie meer info [hier](https://www.w3.org/TR/sparql11-query/#sparqlOffsetLimit).

## Monitoring

Het kan even duren vooraleer alle productversie's uit de LDES gesynchronizeerd zijn met de lokale triplestore.
Om te checken of alles goed binnenstroomt / geen nieuwe producten meer binnenkomen, kan volgende COUNT query gebruikt worden:
```
PREFIX schema: <https://schema.org/>

SELECT (count(distinct ?product) as ?c)
WHERE {
  GRAPH <urn:x-arq:DefaultGraph> {
    ?product a schema:TouristAttraction .
  }
}
```

Ook het "info" tabblad op `http://localhost:3030/#/dataset/ds/info` geeft inzicht of er (nog) data binnenstroomt of niet.

Tip: als je twijfelt, herstart de Docker containers:
CTRL-C en `docker-compose start`
OF `docker-compose restart`

## Alle info van 1 specifiek product

Om zicht te krijgen wat er allemaal beschikbaar is over 1 product gebruiken we een SPARQL CONSTRUCT query.

Ga naar `http://localhost:3030/#/dataset/ds/query`

### Turtle - mensleesbaar

Zorg ervoor dat Content Type (GRAPH) op "Turtle" staat om een mooi resultaat te krijgen.

Voorbeeld-query:
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

CONSTRUCT {
  ?product ?p ?o .
  ?o ?p2 ?o2 .
  ?o2 ?p3 ?o3 .
} WHERE {
  GRAPH <urn:x-arq:DefaultGraph> {
    ?product ?p ?o ;
   		adms:identifier [ skos:notation ?winId ]

    OPTIONAL {
    ?o ?p2 ?o2 .
      OPTIONAL {
      ?o2 ?p3 ?o3 .
      }
    }
    VALUES ?winId { "1029472"^^<https://westtoer.be/id/concept/identificatiesysteem/win> }
    # Of met product URI: 
    # VALUES ?product { <https://westtoer.be/id/product/f04a019e-ba7e-455c-89df-36516bdb1a40> }
  }
}
```

### JSON-LD - machineleesbaar

Voor verwerking in code kan het handig zijn om het product in JSON-LD terug te krijgen.

Stap 1: pas Content Type (GRAPH) aan naar "JSON-LD"
In code betekent dit dat een HTTP request met "Content-Type": "application/ld+json" wordt verstuurd naar het SPARQL endpoint.

In code betekent dit dat je een [SPARQL request](https://www.w3.org/TR/sparql11-protocol/) moet sturen naar "http://localhost:3030/ds/sparql".
Hier zie je een [voorbeeld](https://query.linkeddatafragments.org/#datasources=http%3A%2F%2Flocalhost%3A3030%2Fds%2Fsparql&query=SELECT%20%3Fs%20%3Fp%20%3Fo%0AWHERE%20%7B%0A%20%20GRAPH%20%3Curn%3Ax-arq%3ADefaultGraph%3E%20%7B%0A%20%20%20%09%3Fs%20%3Fp%20%3Fo%20.%20%0A%20%20%7D%0A%7D%0ALIMIT%2010) in de Comunica client.
 
Stap 2: frame het JSON-LD object in de JSON-LD playground
Klik op de tab "Framed"

Links plak je de output van Stap 1.
Rechts maak je een object met als @id het product.
Zie [hier](https://json-ld.org/playground/#startTab=tab-framed&json-ld=%7B%22%40id%22%3A%22https%3A%2F%2Fwesttoer.be%2Fid%2Fproduct%2F76ec83f9-6dfb-4039-9eb7-701da08efbfd%2F2023-11-23T12%3A46%3A11.6388872Z%22%2C%22%40type%22%3A%22https%3A%2F%2Fschema.org%2FTouristAttraction%22%2C%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2FisVersionOf%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fwesttoer.be%2Fid%2Fproduct%2F76ec83f9-6dfb-4039-9eb7-701da08efbfd%22%7D%2C%22http%3A%2F%2Fwww.w3.org%2Fns%2Fadms%23identifier%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Fadms%23Identifier%22%2C%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2Fcreator%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fdata.vlaanderen.be%2Fid%2Forganisatie%2FOVO018769%22%7D%2C%22http%3A%2F%2Fwww.w3.org%2F2004%2F02%2Fskos%2Fcore%23notation%22%3A%7B%22%40type%22%3A%22https%3A%2F%2Fwesttoer.be%2Fid%2Fconcept%2Fidentificatiesysteem%2Fwin%22%2C%22%40value%22%3A%221000309%22%7D%2C%22http%3A%2F%2Fwww.w3.org%2Fns%2Fadms%23schemaAgency%22%3A%22Westtoer%22%7D%2C%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23address%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23Address%22%2C%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23adminUnitL2%22%3A%5B%7B%22%40language%22%3A%22en%22%2C%22%40value%22%3A%22West%20Flanders%22%7D%2C%7B%22%40language%22%3A%22nl%22%2C%22%40value%22%3A%22West-Vlaanderen%22%7D%5D%2C%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23postCode%22%3A%228670%22%2C%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23thoroughfare%22%3A%7B%22%40language%22%3A%22nl%22%2C%22%40value%22%3A%22Zeedijk%22%7D%2C%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fadres%23Adresvoorstelling.huisnummer%22%3A%22439%22%2C%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fadres%23gemeentenaam%22%3A%7B%22%40language%22%3A%22nl%22%2C%22%40value%22%3A%22Koksijde%22%7D%2C%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fadres%23land%22%3A%7B%22%40language%22%3A%22nl%22%2C%22%40value%22%3A%22Belgi%C3%83%C2%83%C3%82%C2%AB%22%7D%2C%22https%3A%2F%2Fwesttoer.be%2Fns%23gemeenteniscode%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23integer%22%2C%22%40value%22%3A%2238014%22%7D%2C%22https%3A%2F%2Fwesttoer.be%2Fns%23isToegekendDoorDeelgemeente%22%3A%7B%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2FisVersionOf%22%3A%7B%7D%2C%22http%3A%2F%2Fwww.w3.org%2F2004%2F02%2Fskos%2Fcore%23prefLabel%22%3A%7B%22%40language%22%3A%22nl%22%2C%22%40value%22%3A%22Oostduinkerke%22%7D%7D%2C%22https%3A%2F%2Fwesttoer.be%2Fns%23isToegekendDoorGemeente%22%3A%7B%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2FisVersionOf%22%3A%7B%7D%2C%22http%3A%2F%2Fwww.w3.org%2F2004%2F02%2Fskos%2Fcore%23prefLabel%22%3A%7B%22%40language%22%3A%22nl%22%2C%22%40value%22%3A%22Koksijde%22%7D%7D%7D%2C%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23geometry%22%3A%7B%22%40type%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgeneriek%23Geometrie%22%2C%22http%3A%2F%2Fwww.w3.org%2F2003%2F01%2Fgeo%2Fwgs84_pos%23lat%22%3A%7B%22%40type%22%3A%22https%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23double%22%2C%22%40value%22%3A%225.11325169E1%22%7D%2C%22http%3A%2F%2Fwww.w3.org%2F2003%2F01%2Fgeo%2Fwgs84_pos%23long%22%3A%7B%22%40type%22%3A%22https%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23double%22%2C%22%40value%22%3A%222.6709437E0%22%7D%7D%2C%22http%3A%2F%2Fwww.w3.org%2Fns%2Fprov%23generatedAtTime%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23dateTime%22%2C%22%40value%22%3A%222023-11-23T12%3A46%3A11.6388872Z%22%7D%2C%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgeneriek%23lokaleIdentificator%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23string%22%2C%22%40value%22%3A%2276ec83f9-6dfb-4039-9eb7-701da08efbfd%22%7D%2C%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgeneriek%23naamruimte%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23string%22%2C%22%40value%22%3A%22https%3A%2F%2Fwesttoer.be%2Fid%2Fproduct%22%7D%2C%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgeneriek%23versieIdentificator%22%3A%222023-11-23T12%3A46%3A11.6388872Z%22%2C%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23behoortTotToeristischeRegio%22%3A%7B%22%40type%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23ToeristischeRegio%22%2C%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2FisVersionOf%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fwesttoer.be%2Fid%2Ftoeristischeregio%2F9f5fd8a9-d354-492b-bda8-25ebd8422fe9%22%2C%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgeneriek%23lokaleIdentificator%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23string%22%2C%22%40value%22%3A%229f5fd8a9-d354-492b-bda8-25ebd8422fe9%22%7D%7D%2C%22http%3A%2F%2Fwww.w3.org%2F2004%2F02%2Fskos%2Fcore%23prefLabel%22%3A%7B%22%40language%22%3A%22nl%22%2C%22%40value%22%3A%22Kust%22%7D%7D%2C%22https%3A%2F%2Fschema.org%2FadditionalType%22%3A%7B%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2FisVersionOf%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fwesttoer.be%2Fid%2Fconcept%2Fproducttype%2F0236363e-18b7-4d43-95f2-44bcd27ba64d%22%2C%22http%3A%2F%2Fwww.w3.org%2F2004%2F02%2Fskos%2Fcore%23prefLabel%22%3A%5B%7B%22%40language%22%3A%22nl%22%2C%22%40value%22%3A%22Eetgelegenheid%22%7D%2C%7B%22%40language%22%3A%22fr%22%2C%22%40value%22%3A%22Eetgelegenheid%22%7D%2C%7B%22%40language%22%3A%22en%22%2C%22%40value%22%3A%22Eetgelegenheid%22%7D%2C%7B%22%40language%22%3A%22de%22%2C%22%40value%22%3A%22Eetgelegenheid%22%7D%5D%2C%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgeneriek%23lokaleIdentificator%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23string%22%2C%22%40value%22%3A%220236363e-18b7-4d43-95f2-44bcd27ba64d%22%7D%7D%7D%2C%22https%3A%2F%2Fschema.org%2Famount%22%3A%7B%22%40type%22%3A%22https%3A%2F%2Fschema.org%2FMonetaryAmount%22%2C%22https%3A%2F%2Fschema.org%2Fcurrency%22%3A%22EUR%22%7D%2C%22https%3A%2F%2Fschema.org%2FcontactPoint%22%3A%7B%22%40type%22%3A%22https%3A%2F%2Fschema.org%2FContactPoint%22%2C%22http%3A%2F%2Fxmlns.com%2Ffoaf%2F0.1%2Fpage%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23anyURI%22%2C%22%40value%22%3A%22https%3A%2F%2Ftapavino.be%2F%22%7D%2C%22https%3A%2F%2Fschema.org%2Femail%22%3A%22info%40tapavino.be%22%2C%22https%3A%2F%2Fschema.org%2Ftelephone%22%3A%22%2B32%20472%2067%2023%2060%22%7D%2C%22https%3A%2F%2Fschema.org%2Fdescription%22%3A%5B%7B%22%40language%22%3A%22fr%22%2C%22%40value%22%3A%22Vous%20pouvez%20nous%20rejoindre%20pour%20un%20ap%C3%83%C2%83%C3%82%C2%A9ritif%20savoureux%2C%20des%20tapas%2C%20une%20entr%C3%83%C2%83%C3%82%C2%A9e%2C%20un%20plat%20principal%2C%20un%20dessert.%5Cr%5CnTapa%20Vino%2C%20un%20bon%20verre%20de%20vin%20accompagn%C3%83%C2%83%C3%82%C2%A9%20d'un%20grand%20choix%20de%20tapas%20!%22%7D%2C%7B%22%40language%22%3A%22de%22%2C%22%40value%22%3A%22Sie%20k%C3%83%C2%83%C3%82%C2%B6nnen%20bei%20uns%20einen%20leckeren%20Aperitif%2C%20Tapas%2C%20eine%20Vorspeise%2C%20einen%20Hauptgang%20oder%20ein%20Dessert%20genie%C3%83%C2%83%C3%82%C2%9Fen.%5Cr%5CnTapa%20Vino%2C%20ein%20gutes%20Glas%20Wein%20mit%20einer%20gro%C3%83%C2%83%C3%82%C2%9Fen%20Auswahl%20an%20Tapas!%22%7D%2C%7B%22%40language%22%3A%22en%22%2C%22%40value%22%3A%22You%20can%20join%20us%20for%20a%20tasty%20aperitif%2C%20tapas%2C%20starter%2C%20main%20course%2C%20dessert.%5Cr%5CnTapa%20Vino%2C%20a%20nice%20glass%20of%20wine%20with%20a%20great%20choice%20of%20tapas!%22%7D%2C%7B%22%40language%22%3A%22nl%22%2C%22%40value%22%3A%22Je%20kan%20bij%20ons%20terecht%20voor%20een%20lekkere%20aperitief%2C%20tapa's%2C%20voorgerecht%2C%20hoofdgerecht%2C%20dessert.%5Cr%5CnTapa%20Vino%2C%20een%20lekker%20glas%20wijn%20met%20een%20grote%20keuze%20aan%20tapa's!%22%7D%5D%2C%22https%3A%2F%2Fschema.org%2Fname%22%3A%5B%7B%22%40language%22%3A%22de%22%2C%22%40value%22%3A%22Tapa%20Vino%22%7D%2C%7B%22%40language%22%3A%22en%22%2C%22%40value%22%3A%22Tapa%20Vino%22%7D%2C%7B%22%40language%22%3A%22fr%22%2C%22%40value%22%3A%22Tapa%20Vino%22%7D%2C%7B%22%40language%22%3A%22nl%22%2C%22%40value%22%3A%22Tapa%20Vino%22%7D%5D%2C%22https%3A%2F%2Fwesttoer.be%2Fns%23Product.status%22%3A%7B%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2FisVersionOf%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fwesttoer.be%2Fid%2Fconcept%2Fproductstatus%2Ff6c056f4-5db9-4030-a1f5-7727cd475715%22%2C%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgeneriek%23lokaleIdentificator%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23string%22%2C%22%40value%22%3A%22f6c056f4-5db9-4030-a1f5-7727cd475715%22%7D%7D%7D%2C%22https%3A%2F%2Fwesttoer.be%2Fns%23heeftKenmerk%22%3A%7B%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2FisVersionOf%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fwesttoer.be%2Fid%2Fconcept%2Fkenmerk%2F402834da-241b-4522-8027-80d1173ef3ef%22%2C%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgeneriek%23lokaleIdentificator%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23string%22%2C%22%40value%22%3A%22402834da-241b-4522-8027-80d1173ef3ef%22%7D%7D%7D%2C%22https%3A%2F%2Fwesttoer.be%2Fns%23isRelevantVoorWesttoer%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%2C%22%40value%22%3A%22true%22%7D%2C%22https%3A%2F%2Fwww.datatourisme.fr%2Fontology%2Fcore%23hasContact%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Forg%23Organization%22%2C%22https%3A%2F%2Fschema.org%2FcontactPoint%22%3A%7B%22%40type%22%3A%22https%3A%2F%2Fschema.org%2FContactPoint%22%7D%7D%2C%22https%3A%2F%2Fwww.datatourisme.fr%2Fontology%2Fcore%23isLocatedAt%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2FLocation%22%7D%7D&frame=%7B%22%40id%22%3A%22https%3A%2F%2Fwesttoer.be%2Fid%2Fproduct%2F76ec83f9-6dfb-4039-9eb7-701da08efbfd%2F2023-11-23T12%3A46%3A11.6388872Z%22%7D&context=%7B%22%40context%22%3A%7B%22westtoer%22%3A%22https%3A%2F%2Fwesttoer.be%2Fid%2F%22%2C%22westtoerns%22%3A%22https%3A%2F%2Fwesttoer.be%2Fns%23%22%2C%22westtoerproduct%22%3A%22westtoer%3Aproduct%2F%22%2C%22westtoersamengesteldproduct%22%3A%22westtoer%3Asamengesteldproduct%2F%22%2C%22westtoerproductlist%22%3A%22westtoer%3Aproductlist%2F%22%2C%22westtoerconcept%22%3A%22westtoer%3Aconcept%2F%22%2C%22westtoerregio%22%3A%22westtoer%3Aregio%2F%22%2C%22dc%22%3A%22http%3A%2F%2Fpurl.org%2Fdc%2Felements%2F1.1%2F%22%2C%22dcmitype%22%3A%22http%3A%2F%2Fpurl.org%2Fdc%2Fdcmitype%2F%22%2C%22dcterms%22%3A%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2F%22%2C%22skos%22%3A%22http%3A%2F%2Fwww.w3.org%2F2004%2F02%2Fskos%2Fcore%23%22%2C%22logies%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23%22%2C%22GeregistreerdeOrganisatie%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Fregorg%23RegisteredOrganization%22%2C%22Adres%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fadres%23Adres%22%2C%22Logies%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23Logies%22%2C%22ToeristischeRegio%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23ToeristischeRegio%22%2C%22Adresvoorstelling%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23Address%22%2C%22Collectie%22%3A%22dcmitype%3ACollection%22%2C%22taal%22%3A%22%40language%22%2C%22bestaatUit%22%3A%7B%22%40container%22%3A%22%40set%22%2C%22%40id%22%3A%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2FhasPart%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22administratieveEenheidNiveau1%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23adminUnitL1%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22administratieveEenheidNiveau2%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23adminUnitL2%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22adres%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23address%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22busnummer%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fadres%23Adresvoorstelling.busnummer%22%7D%2C%22toegekendDoor%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2Fcreator%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22toegekendDoorString%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Fadms%23schemaAgency%22%7D%2C%22contactPunt%22%3A%7B%22%40id%22%3A%22schema%3AcontactPoint%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22contactnaam%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fxmlns.com%2Ffoaf%2F0.1%2Fname%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22email%22%3A%7B%22%40id%22%3A%22schema%3Aemail%22%7D%2C%22fax%22%3A%7B%22%40id%22%3A%22schema%3AfaxNumber%22%7D%2C%22postbus%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23poBox%22%7D%2C%22postcode%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23postCode%22%7D%2C%22gemeentenaam%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fadres%23gemeentenaam%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22straatnaam%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23thoroughfare%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22geometrie%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23geometry%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22verwijstNaar%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fadres%23verwijstNaar%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22website%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fxmlns.com%2Ffoaf%2F0.1%2Fpage%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23anyURI%22%7D%2C%22wgs84Latitude%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2F2003%2F01%2Fgeo%2Fwgs84_pos%23lat%22%2C%22%40type%22%3A%22https%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23double%22%7D%2C%22wgs84Longitude%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2F2003%2F01%2Fgeo%2Fwgs84_pos%23long%22%2C%22%40type%22%3A%22https%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23double%22%7D%2C%22wkt%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.opengis.net%2Font%2Fgeosparql%23asWKT%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.opengis.net%2Font%2Fgeosparql%23wktLiteral%22%7D%2C%22behoortTotToeristischeRegio%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23behoortTotToeristischeRegio%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22beschrijving%22%3A%7B%22%40id%22%3A%22schema%3Adescription%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22huisnummer%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fadres%23Adresvoorstelling.huisnummer%22%7D%2C%22telefoon%22%3A%7B%22%40id%22%3A%22schema%3Atelephone%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22schema%22%3A%22https%3A%2F%2Fschema.org%2F%22%2C%22TouristAttraction%22%3A%22schema%3ATouristAttraction%22%2C%22additionalType%22%3A%7B%22%40id%22%3A%22schema%3AadditionalType%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22isBasedOnUrl%22%3A%7B%22%40id%22%3A%22schema%3AisBasedOnUrl%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22naam%22%3A%7B%22%40id%22%3A%22schema%3Aname%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22Geometrie%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgeneriek%23Geometrie%22%2C%22Contactinfo%22%3A%22schema%3AContactPoint%22%2C%22OpeningHoursSpecification%22%3A%22schema%3AOpeningHoursSpecification%22%2C%22gemeenteniscode%22%3A%7B%22%40id%22%3A%22westtoerns%3Agemeenteniscode%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23integer%22%7D%2C%22gemeente%22%3A%7B%22%40id%22%3A%22westtoerns%3AisToegekendDoorGemeente%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22deelgemeente%22%3A%7B%22%40id%22%3A%22westtoerns%3AisToegekendDoorDeelgemeente%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22provincie%22%3A%7B%22%40id%22%3A%22westtoerns%3AisToegekendDoorProvincie%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22label%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2F2004%2F02%2Fskos%2Fcore%23prefLabel%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22latitude%22%3A%22schema%3Alatitude%22%2C%22longitude%22%3A%22schema%3Alongitude%22%2C%22isVergundDoor%22%3A%7B%22%40id%22%3A%22westtoerns%3AisVergundDoor%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22Vergunning%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fbesluit%2F%23Vergunning%22%2C%22Vergunning.statusTVL%22%3A%7B%22%40id%22%3A%22westtoerns%3AVergunning.statusTVL%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22isRelevantVoorWesttoer%22%3A%7B%22%40id%22%3A%22westtoerns%3AisRelevantVoorWesttoer%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22gepubliceerd%22%3A%7B%22%40id%22%3A%22westtoerns%3Agepubliceerd%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22tijdelijkGesloten%22%3A%7B%22%40id%22%3A%22westtoerns%3AtijdelijkGesloten%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22extraInfoLogiesType%22%3A%7B%22%40id%22%3A%22westtoerns%3AextraInfoLogiesType%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22inventarisatiejaar%22%3A%7B%22%40id%22%3A%22westtoerns%3Ainventarisatiejaar%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23integer%22%7D%2C%22actief1juli%22%3A%7B%22%40id%22%3A%22westtoerns%3Aactief1juli%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22publicatieOpRegiowebsitesEnVisitWVL%22%3A%7B%22%40id%22%3A%22westtoerns%3ApublicatieOpRegiowebsitesEnVisitWVL%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22identificator%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Fadms%23identifier%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22Identificator%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Fadms%23Identifier%22%2C%22Identificator.identificator%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2F2004%2F02%2Fskos%2Fcore%23notation%22%7D%2C%22lokaleId%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgeneriek%23lokaleIdentificator%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23string%22%7D%2C%22versieId%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgeneriek%23versieIdentificator%22%7D%2C%22naamruimte%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgeneriek%23naamruimte%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23string%22%7D%2C%22behoortTotMacroproduct%22%3A%7B%22%40container%22%3A%22%40set%22%2C%22%40id%22%3A%22westtoerns%3AbehoortTotMacroproduct%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22behoortTotSamengesteldProduct%22%3A%7B%22%40container%22%3A%22%40set%22%2C%22%40id%22%3A%22westtoerns%3AbehoortTotSamengesteldProduct%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22behoortTotProductlijst%22%3A%7B%22%40container%22%3A%22%40set%22%2C%22%40id%22%3A%22westtoerns%3AbehoortTotProductlijst%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22generatedAtTime%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Fprov%23generatedAtTime%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23dateTime%22%7D%2C%22isVersieVan%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2FisVersionOf%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22Product.status%22%3A%7B%22%40id%22%3A%22westtoerns%3AProduct.status%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22prefLabel%22%3A%7B%22%40id%22%3A%22skos%3AprefLabel%22%7D%2C%22heeftKwaliteitslabel%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23heeftKwaliteitsLabel%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22Kwaliteitslabel%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23Kwaliteitslabel%22%2C%22heeftAuteur%22%3A%7B%22%40id%22%3A%22schema%3Aauthor%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22toegekendOp%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2Fissued%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23dateTime%22%7D%2C%22Faciliteit%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23Faciliteit%22%2C%22heeftFaciliteit%22%3A%7B%22%40id%22%3A%22schema%3AamenityFeature%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22isSpecialisatieVan%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23isSpecialisatieVan%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22beschikbaarheid%22%3A%7B%22%40container%22%3A%22%40set%22%2C%22%40id%22%3A%22schema%3AhoursAvailable%22%7D%2C%22beschikbaarTot%22%3A%7B%22%40id%22%3A%22schema%3AvalidThrough%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23dateTime%22%7D%2C%22beschikbaarVan%22%3A%7B%22%40id%22%3A%22schema%3AvalidFrom%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23dateTime%22%7D%2C%22opent%22%3A%7B%22%40id%22%3A%22schema%3Aopens%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23time%22%7D%2C%22sluit%22%3A%7B%22%40id%22%3A%22schema%3Acloses%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23time%22%7D%2C%22dagInWeek%22%3A%7B%22%40id%22%3A%22schema%3AdayOfWeek%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22interneOpmerking%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23comment%22%7D%2C%22locatieaanduiding%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23locatieaanduiding%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22Locatieaanduiding.type%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fadres%23Locatieaanduiding.type%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22Locatieaanduiding%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Fprov%23Location%22%2C%22Locatieaanduiding.aanduiding%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23label%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22land%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fadres%23land%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22heeftRegistratie%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23heeftRegistratie%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22registratie%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Fregorg%23registration%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22Registratie%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23Registratie%22%2C%22registratieStatus%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23registratieStatus%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22Stopzetting%22%3A%22westtoerns%3AStopzetting%22%2C%22Stopzetting.reden%22%3A%7B%22%40id%22%3A%22westtoerns%3AStopzetting.reden%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23string%22%7D%2C%22Overname%22%3A%22westtoerns%3AOvername%22%2C%22Oprichting%22%3A%22westtoerns%3AOprichting%22%2C%22isStopgezet%22%3A%7B%22%40id%22%3A%22westtoerns%3AisStopgezet%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22isOvergenomen%22%3A%7B%22%40id%22%3A%22westtoerns%3AisOvergenomen%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22type%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2Ftype%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22verantwoordelijkeOrganisatie%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Fprov%23wasAssociatedWith%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22aantalKamers%22%3A%7B%22%40id%22%3A%22schema%3AnumberOfRooms%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23integer%22%7D%2C%22aantalSlaapplaatsen%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23aantalSlaapplaatsen%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23integer%22%7D%2C%22aantalVerhuureenheden%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23aantalVerhuureenheden%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23integer%22%7D%2C%22heeftOprichting%22%3A%7B%22%40id%22%3A%22westtoerns%3AheeftOprichting%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22Activiteit.jaar%22%3A%7B%22%40id%22%3A%22westtoerns%3AActiviteit.jaar%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23integer%22%7D%2C%22Activiteit.maand%22%3A%7B%22%40id%22%3A%22westtoerns%3AActiviteit.maand%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23integer%22%7D%2C%22heeftOvername%22%3A%7B%22%40id%22%3A%22westtoerns%3AheeftOvername%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22heeftStopzetting%22%3A%7B%22%40id%22%3A%22westtoerns%3AheeftStopzetting%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22wettelijkeNaam%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Fregorg%23legalName%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22heeftUitbater%22%3A%7B%22%40id%22%3A%22westtoerns%3AheeftUitbater%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22heeftMedia%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23heeftMedia%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22seeAlso%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23seeAlso%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22contentUrl%22%3A%7B%22%40id%22%3A%22schema%3AcontentUrl%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22publicatieDatum%22%3A%7B%22%40id%22%3A%22schema%3AdatePublished%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23dateTime%22%7D%2C%22copyright%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fschema.org%2FcopyrightNotice%22%7D%2C%22isSpotlight%22%3A%7B%22%40id%22%3A%22westtoerns%3AisSpotlight%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22sortOrder%22%3A%7B%22%40id%22%3A%22westtoerns%3AsortOrder%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23integer%22%7D%2C%22CreatiefWerk%22%3A%22schema%3ACreativeWork%22%2C%22url%22%3A%7B%22%40id%22%3A%22schema%3Aurl%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23anyURI%22%7D%2C%22heeftContactpersoon%22%3A%7B%22%40id%22%3A%22westtoerns%3AheeftContactpersoon%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22Persoon%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Fperson%23Person%22%2C%22voornaam%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fxmlns.com%2Ffoaf%2F0.1%2FgivenName%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23string%22%7D%2C%22achternaam%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fxmlns.com%2Ffoaf%2F0.1%2FfamilyName%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23string%22%7D%2C%22contactType%22%3A%7B%22%40id%22%3A%22schema%3AcontactType%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22prijs%22%3A%7B%22%40id%22%3A%22schema%3Aamount%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22Geldbedrag%22%3A%7B%22%40id%22%3A%22schema%3AMonetaryAmount%22%7D%2C%22waarde%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fschema.org%2Fvalue%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23double%22%7D%2C%22valuta%22%3A%22schema%3Acurrency%22%2C%22isGratis%22%3A%7B%22%40id%22%3A%22schema%3AisAccessibleForFree%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22Beoordeling%22%3A%22schema%3ARating%22%2C%22heeftOfficieleBeoordeling%22%3A%7B%22%40id%22%3A%22schema%3AstarRating%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22hoogsteBeoordeling%22%3A%7B%22%40id%22%3A%22schema%3AbestRating%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23string%22%7D%2C%22laagsteBeoordeling%22%3A%7B%22%40id%22%3A%22schema%3AworstRating%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23string%22%7D%2C%22beoordeling%22%3A%7B%22%40id%22%3A%22schema%3AratingValue%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23string%22%7D%2C%22gewijzigdDoor%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Fprov%23wasAttributedTo%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22uitsluitenVanPublicatie%22%3A%7B%22%40id%22%3A%22westtoerns%3AuitsluitenVanPublicatie%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22uitsluitenVanJaarlijkseBevraging%22%3A%7B%22%40id%22%3A%22westtoerns%3AuitsluitenVanJaarlijkseBevraging%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22broader%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2F2004%2F02%2Fskos%2Fcore%23broader%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22exactMatch%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2F2004%2F02%2Fskos%2Fcore%23exactMatch%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22heeftAlsLocatie%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fwww.datatourisme.fr%2Fontology%2Fcore%23isLocatedAt%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22Plaats%22%3A%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2FLocation%22%2C%22heeftAlsContact%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fwww.datatourisme.fr%2Fontology%2Fcore%23hasContact%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22Organisatie%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Forg%23Organization%22%2C%22doorsturenNaarUiT%22%3A%7B%22%40id%22%3A%22westtoerns%3AdoorsturenNaarUiT%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22heeftKenmerk%22%3A%7B%22%40id%22%3A%22westtoerns%3AheeftKenmerk%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22heeftRuimte%22%3A%7B%22%40id%22%3A%22westtoerns%3AheeftRuimte%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22KwantitatieveWaarde%22%3A%22schema%3AQuantitativeValue%22%2C%22KwantitatieveWaarde.eenheid%22%3A%7B%22%40id%22%3A%22schema%3AunitText%22%7D%2C%22KwantitatieveWaarde.standaardEenheid%22%3A%7B%22%40id%22%3A%22schema%3AunitCode%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22Zaal%22%3A%22https%3A%2F%2Fwww.datatourisme.fr%2Fontology%2Fcore%23MultiPurposeRoomOrCommunityRoom%22%2C%22heeftAlsVloerOppervlakte%22%3A%7B%22%40id%22%3A%22schema%3AfloorSize%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22hoogte%22%3A%7B%22%40id%22%3A%22schema%3Aheight%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22aantalSubzalen%22%3A%7B%22%40id%22%3A%22westtoerns%3AaantalSubzalen%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23integer%22%7D%2C%22heeftIndeling%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fwww.datatourisme.fr%2Fontology%2Fcore%23hasLayout%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22Indeling%22%3A%22https%3A%2F%2Fwww.datatourisme.fr%2Fontology%2Fcore%23RoomLayout%22%2C%22indelingBeschikbaar%22%3A%7B%22%40id%22%3A%22westtoerns%3AindelingBeschikbaar%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22heeftInvoerder%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2Fcontributor%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22capaciteit%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23capaciteit%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22aantalVerblijfplaatsen%22%3A%7B%22%40id%22%3A%22westtoerns%3AaantalVerblijfplaatsen%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23integer%22%7D%7D%7D) het resultaat.


In code betekent dit dat je een JSON-LD library moet gebruiken om framing toe te passen.

Stap 3: compacteer het object in de JSON-LD playground
Klik op de tab "Compacted"
Links plak je de output van Stap 2.
Rechts plak je de nieuwe JSON-LD context zoals beschikbaar in deze repo.

Zie [hier](https://json-ld.org/playground/#startTab=tab-compacted&json-ld=%7B%22%40id%22%3A%22https%3A%2F%2Fwesttoer.be%2Fid%2Fproduct%2F76ec83f9-6dfb-4039-9eb7-701da08efbfd%2F2023-11-23T12%3A46%3A11.6388872Z%22%2C%22%40type%22%3A%22https%3A%2F%2Fschema.org%2FTouristAttraction%22%2C%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2FisVersionOf%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fwesttoer.be%2Fid%2Fproduct%2F76ec83f9-6dfb-4039-9eb7-701da08efbfd%22%7D%2C%22http%3A%2F%2Fwww.w3.org%2Fns%2Fadms%23identifier%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Fadms%23Identifier%22%2C%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2Fcreator%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fdata.vlaanderen.be%2Fid%2Forganisatie%2FOVO018769%22%7D%2C%22http%3A%2F%2Fwww.w3.org%2F2004%2F02%2Fskos%2Fcore%23notation%22%3A%7B%22%40type%22%3A%22https%3A%2F%2Fwesttoer.be%2Fid%2Fconcept%2Fidentificatiesysteem%2Fwin%22%2C%22%40value%22%3A%221000309%22%7D%2C%22http%3A%2F%2Fwww.w3.org%2Fns%2Fadms%23schemaAgency%22%3A%22Westtoer%22%7D%2C%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23address%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23Address%22%2C%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23adminUnitL2%22%3A%5B%7B%22%40language%22%3A%22en%22%2C%22%40value%22%3A%22West%20Flanders%22%7D%2C%7B%22%40language%22%3A%22nl%22%2C%22%40value%22%3A%22West-Vlaanderen%22%7D%5D%2C%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23postCode%22%3A%228670%22%2C%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23thoroughfare%22%3A%7B%22%40language%22%3A%22nl%22%2C%22%40value%22%3A%22Zeedijk%22%7D%2C%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fadres%23Adresvoorstelling.huisnummer%22%3A%22439%22%2C%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fadres%23gemeentenaam%22%3A%7B%22%40language%22%3A%22nl%22%2C%22%40value%22%3A%22Koksijde%22%7D%2C%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fadres%23land%22%3A%7B%22%40language%22%3A%22nl%22%2C%22%40value%22%3A%22Belgi%C3%83%C2%83%C3%82%C2%AB%22%7D%2C%22https%3A%2F%2Fwesttoer.be%2Fns%23gemeenteniscode%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23integer%22%2C%22%40value%22%3A%2238014%22%7D%2C%22https%3A%2F%2Fwesttoer.be%2Fns%23isToegekendDoorDeelgemeente%22%3A%7B%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2FisVersionOf%22%3A%7B%7D%2C%22http%3A%2F%2Fwww.w3.org%2F2004%2F02%2Fskos%2Fcore%23prefLabel%22%3A%7B%22%40language%22%3A%22nl%22%2C%22%40value%22%3A%22Oostduinkerke%22%7D%7D%2C%22https%3A%2F%2Fwesttoer.be%2Fns%23isToegekendDoorGemeente%22%3A%7B%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2FisVersionOf%22%3A%7B%7D%2C%22http%3A%2F%2Fwww.w3.org%2F2004%2F02%2Fskos%2Fcore%23prefLabel%22%3A%7B%22%40language%22%3A%22nl%22%2C%22%40value%22%3A%22Koksijde%22%7D%7D%7D%2C%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23geometry%22%3A%7B%22%40type%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgeneriek%23Geometrie%22%2C%22http%3A%2F%2Fwww.w3.org%2F2003%2F01%2Fgeo%2Fwgs84_pos%23lat%22%3A%7B%22%40type%22%3A%22https%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23double%22%2C%22%40value%22%3A%225.11325169E1%22%7D%2C%22http%3A%2F%2Fwww.w3.org%2F2003%2F01%2Fgeo%2Fwgs84_pos%23long%22%3A%7B%22%40type%22%3A%22https%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23double%22%2C%22%40value%22%3A%222.6709437E0%22%7D%7D%2C%22http%3A%2F%2Fwww.w3.org%2Fns%2Fprov%23generatedAtTime%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23dateTime%22%2C%22%40value%22%3A%222023-11-23T12%3A46%3A11.6388872Z%22%7D%2C%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgeneriek%23lokaleIdentificator%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23string%22%2C%22%40value%22%3A%2276ec83f9-6dfb-4039-9eb7-701da08efbfd%22%7D%2C%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgeneriek%23naamruimte%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23string%22%2C%22%40value%22%3A%22https%3A%2F%2Fwesttoer.be%2Fid%2Fproduct%22%7D%2C%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgeneriek%23versieIdentificator%22%3A%222023-11-23T12%3A46%3A11.6388872Z%22%2C%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23behoortTotToeristischeRegio%22%3A%7B%22%40type%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23ToeristischeRegio%22%2C%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2FisVersionOf%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fwesttoer.be%2Fid%2Ftoeristischeregio%2F9f5fd8a9-d354-492b-bda8-25ebd8422fe9%22%2C%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgeneriek%23lokaleIdentificator%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23string%22%2C%22%40value%22%3A%229f5fd8a9-d354-492b-bda8-25ebd8422fe9%22%7D%7D%2C%22http%3A%2F%2Fwww.w3.org%2F2004%2F02%2Fskos%2Fcore%23prefLabel%22%3A%7B%22%40language%22%3A%22nl%22%2C%22%40value%22%3A%22Kust%22%7D%7D%2C%22https%3A%2F%2Fschema.org%2FadditionalType%22%3A%7B%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2FisVersionOf%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fwesttoer.be%2Fid%2Fconcept%2Fproducttype%2F0236363e-18b7-4d43-95f2-44bcd27ba64d%22%2C%22http%3A%2F%2Fwww.w3.org%2F2004%2F02%2Fskos%2Fcore%23prefLabel%22%3A%5B%7B%22%40language%22%3A%22nl%22%2C%22%40value%22%3A%22Eetgelegenheid%22%7D%2C%7B%22%40language%22%3A%22fr%22%2C%22%40value%22%3A%22Eetgelegenheid%22%7D%2C%7B%22%40language%22%3A%22en%22%2C%22%40value%22%3A%22Eetgelegenheid%22%7D%2C%7B%22%40language%22%3A%22de%22%2C%22%40value%22%3A%22Eetgelegenheid%22%7D%5D%2C%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgeneriek%23lokaleIdentificator%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23string%22%2C%22%40value%22%3A%220236363e-18b7-4d43-95f2-44bcd27ba64d%22%7D%7D%7D%2C%22https%3A%2F%2Fschema.org%2Famount%22%3A%7B%22%40type%22%3A%22https%3A%2F%2Fschema.org%2FMonetaryAmount%22%2C%22https%3A%2F%2Fschema.org%2Fcurrency%22%3A%22EUR%22%7D%2C%22https%3A%2F%2Fschema.org%2FcontactPoint%22%3A%7B%22%40type%22%3A%22https%3A%2F%2Fschema.org%2FContactPoint%22%2C%22http%3A%2F%2Fxmlns.com%2Ffoaf%2F0.1%2Fpage%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23anyURI%22%2C%22%40value%22%3A%22https%3A%2F%2Ftapavino.be%2F%22%7D%2C%22https%3A%2F%2Fschema.org%2Femail%22%3A%22info%40tapavino.be%22%2C%22https%3A%2F%2Fschema.org%2Ftelephone%22%3A%22%2B32%20472%2067%2023%2060%22%7D%2C%22https%3A%2F%2Fschema.org%2Fdescription%22%3A%5B%7B%22%40language%22%3A%22fr%22%2C%22%40value%22%3A%22Vous%20pouvez%20nous%20rejoindre%20pour%20un%20ap%C3%83%C2%83%C3%82%C2%A9ritif%20savoureux%2C%20des%20tapas%2C%20une%20entr%C3%83%C2%83%C3%82%C2%A9e%2C%20un%20plat%20principal%2C%20un%20dessert.%5Cr%5CnTapa%20Vino%2C%20un%20bon%20verre%20de%20vin%20accompagn%C3%83%C2%83%C3%82%C2%A9%20d'un%20grand%20choix%20de%20tapas%20!%22%7D%2C%7B%22%40language%22%3A%22de%22%2C%22%40value%22%3A%22Sie%20k%C3%83%C2%83%C3%82%C2%B6nnen%20bei%20uns%20einen%20leckeren%20Aperitif%2C%20Tapas%2C%20eine%20Vorspeise%2C%20einen%20Hauptgang%20oder%20ein%20Dessert%20genie%C3%83%C2%83%C3%82%C2%9Fen.%5Cr%5CnTapa%20Vino%2C%20ein%20gutes%20Glas%20Wein%20mit%20einer%20gro%C3%83%C2%83%C3%82%C2%9Fen%20Auswahl%20an%20Tapas!%22%7D%2C%7B%22%40language%22%3A%22en%22%2C%22%40value%22%3A%22You%20can%20join%20us%20for%20a%20tasty%20aperitif%2C%20tapas%2C%20starter%2C%20main%20course%2C%20dessert.%5Cr%5CnTapa%20Vino%2C%20a%20nice%20glass%20of%20wine%20with%20a%20great%20choice%20of%20tapas!%22%7D%2C%7B%22%40language%22%3A%22nl%22%2C%22%40value%22%3A%22Je%20kan%20bij%20ons%20terecht%20voor%20een%20lekkere%20aperitief%2C%20tapa's%2C%20voorgerecht%2C%20hoofdgerecht%2C%20dessert.%5Cr%5CnTapa%20Vino%2C%20een%20lekker%20glas%20wijn%20met%20een%20grote%20keuze%20aan%20tapa's!%22%7D%5D%2C%22https%3A%2F%2Fschema.org%2Fname%22%3A%5B%7B%22%40language%22%3A%22de%22%2C%22%40value%22%3A%22Tapa%20Vino%22%7D%2C%7B%22%40language%22%3A%22en%22%2C%22%40value%22%3A%22Tapa%20Vino%22%7D%2C%7B%22%40language%22%3A%22fr%22%2C%22%40value%22%3A%22Tapa%20Vino%22%7D%2C%7B%22%40language%22%3A%22nl%22%2C%22%40value%22%3A%22Tapa%20Vino%22%7D%5D%2C%22https%3A%2F%2Fwesttoer.be%2Fns%23Product.status%22%3A%7B%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2FisVersionOf%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fwesttoer.be%2Fid%2Fconcept%2Fproductstatus%2Ff6c056f4-5db9-4030-a1f5-7727cd475715%22%2C%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgeneriek%23lokaleIdentificator%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23string%22%2C%22%40value%22%3A%22f6c056f4-5db9-4030-a1f5-7727cd475715%22%7D%7D%7D%2C%22https%3A%2F%2Fwesttoer.be%2Fns%23heeftKenmerk%22%3A%7B%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2FisVersionOf%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fwesttoer.be%2Fid%2Fconcept%2Fkenmerk%2F402834da-241b-4522-8027-80d1173ef3ef%22%2C%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgeneriek%23lokaleIdentificator%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23string%22%2C%22%40value%22%3A%22402834da-241b-4522-8027-80d1173ef3ef%22%7D%7D%7D%2C%22https%3A%2F%2Fwesttoer.be%2Fns%23isRelevantVoorWesttoer%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%2C%22%40value%22%3A%22true%22%7D%2C%22https%3A%2F%2Fwww.datatourisme.fr%2Fontology%2Fcore%23hasContact%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Forg%23Organization%22%2C%22https%3A%2F%2Fschema.org%2FcontactPoint%22%3A%7B%22%40type%22%3A%22https%3A%2F%2Fschema.org%2FContactPoint%22%7D%7D%2C%22https%3A%2F%2Fwww.datatourisme.fr%2Fontology%2Fcore%23isLocatedAt%22%3A%7B%22%40type%22%3A%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2FLocation%22%7D%7D&frame=%7B%22%40id%22%3A%22https%3A%2F%2Fwesttoer.be%2Fid%2Fproduct%2F76ec83f9-6dfb-4039-9eb7-701da08efbfd%2F2023-11-23T12%3A46%3A11.6388872Z%22%7D&context=%7B%22%40context%22%3A%7B%22westtoer%22%3A%22https%3A%2F%2Fwesttoer.be%2Fid%2F%22%2C%22westtoerns%22%3A%22https%3A%2F%2Fwesttoer.be%2Fns%23%22%2C%22westtoerproduct%22%3A%22westtoer%3Aproduct%2F%22%2C%22westtoersamengesteldproduct%22%3A%22westtoer%3Asamengesteldproduct%2F%22%2C%22westtoerproductlist%22%3A%22westtoer%3Aproductlist%2F%22%2C%22westtoerconcept%22%3A%22westtoer%3Aconcept%2F%22%2C%22westtoerregio%22%3A%22westtoer%3Aregio%2F%22%2C%22dc%22%3A%22http%3A%2F%2Fpurl.org%2Fdc%2Felements%2F1.1%2F%22%2C%22dcmitype%22%3A%22http%3A%2F%2Fpurl.org%2Fdc%2Fdcmitype%2F%22%2C%22dcterms%22%3A%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2F%22%2C%22skos%22%3A%22http%3A%2F%2Fwww.w3.org%2F2004%2F02%2Fskos%2Fcore%23%22%2C%22logies%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23%22%2C%22GeregistreerdeOrganisatie%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Fregorg%23RegisteredOrganization%22%2C%22Adres%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fadres%23Adres%22%2C%22Logies%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23Logies%22%2C%22ToeristischeRegio%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23ToeristischeRegio%22%2C%22Adresvoorstelling%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23Address%22%2C%22Collectie%22%3A%22dcmitype%3ACollection%22%2C%22taal%22%3A%22%40language%22%2C%22bestaatUit%22%3A%7B%22%40container%22%3A%22%40set%22%2C%22%40id%22%3A%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2FhasPart%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22administratieveEenheidNiveau1%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23adminUnitL1%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22administratieveEenheidNiveau2%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23adminUnitL2%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22adres%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23address%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22busnummer%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fadres%23Adresvoorstelling.busnummer%22%7D%2C%22toegekendDoor%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2Fcreator%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22toegekendDoorString%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Fadms%23schemaAgency%22%7D%2C%22contactPunt%22%3A%7B%22%40id%22%3A%22schema%3AcontactPoint%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22contactnaam%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fxmlns.com%2Ffoaf%2F0.1%2Fname%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22email%22%3A%7B%22%40id%22%3A%22schema%3Aemail%22%7D%2C%22fax%22%3A%7B%22%40id%22%3A%22schema%3AfaxNumber%22%7D%2C%22postbus%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23poBox%22%7D%2C%22postcode%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23postCode%22%7D%2C%22gemeentenaam%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fadres%23gemeentenaam%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22straatnaam%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23thoroughfare%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22geometrie%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23geometry%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22verwijstNaar%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fadres%23verwijstNaar%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22website%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fxmlns.com%2Ffoaf%2F0.1%2Fpage%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23anyURI%22%7D%2C%22wgs84Latitude%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2F2003%2F01%2Fgeo%2Fwgs84_pos%23lat%22%2C%22%40type%22%3A%22https%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23double%22%7D%2C%22wgs84Longitude%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2F2003%2F01%2Fgeo%2Fwgs84_pos%23long%22%2C%22%40type%22%3A%22https%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23double%22%7D%2C%22wkt%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.opengis.net%2Font%2Fgeosparql%23asWKT%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.opengis.net%2Font%2Fgeosparql%23wktLiteral%22%7D%2C%22behoortTotToeristischeRegio%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23behoortTotToeristischeRegio%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22beschrijving%22%3A%7B%22%40id%22%3A%22schema%3Adescription%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22huisnummer%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fadres%23Adresvoorstelling.huisnummer%22%7D%2C%22telefoon%22%3A%7B%22%40id%22%3A%22schema%3Atelephone%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22schema%22%3A%22https%3A%2F%2Fschema.org%2F%22%2C%22TouristAttraction%22%3A%22schema%3ATouristAttraction%22%2C%22additionalType%22%3A%7B%22%40id%22%3A%22schema%3AadditionalType%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22isBasedOnUrl%22%3A%7B%22%40id%22%3A%22schema%3AisBasedOnUrl%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22naam%22%3A%7B%22%40id%22%3A%22schema%3Aname%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22Geometrie%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgeneriek%23Geometrie%22%2C%22Contactinfo%22%3A%22schema%3AContactPoint%22%2C%22OpeningHoursSpecification%22%3A%22schema%3AOpeningHoursSpecification%22%2C%22gemeenteniscode%22%3A%7B%22%40id%22%3A%22westtoerns%3Agemeenteniscode%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23integer%22%7D%2C%22gemeente%22%3A%7B%22%40id%22%3A%22westtoerns%3AisToegekendDoorGemeente%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22deelgemeente%22%3A%7B%22%40id%22%3A%22westtoerns%3AisToegekendDoorDeelgemeente%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22provincie%22%3A%7B%22%40id%22%3A%22westtoerns%3AisToegekendDoorProvincie%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22label%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2F2004%2F02%2Fskos%2Fcore%23prefLabel%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22latitude%22%3A%22schema%3Alatitude%22%2C%22longitude%22%3A%22schema%3Alongitude%22%2C%22isVergundDoor%22%3A%7B%22%40id%22%3A%22westtoerns%3AisVergundDoor%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22Vergunning%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fbesluit%2F%23Vergunning%22%2C%22Vergunning.statusTVL%22%3A%7B%22%40id%22%3A%22westtoerns%3AVergunning.statusTVL%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22isRelevantVoorWesttoer%22%3A%7B%22%40id%22%3A%22westtoerns%3AisRelevantVoorWesttoer%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22gepubliceerd%22%3A%7B%22%40id%22%3A%22westtoerns%3Agepubliceerd%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22tijdelijkGesloten%22%3A%7B%22%40id%22%3A%22westtoerns%3AtijdelijkGesloten%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22extraInfoLogiesType%22%3A%7B%22%40id%22%3A%22westtoerns%3AextraInfoLogiesType%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22inventarisatiejaar%22%3A%7B%22%40id%22%3A%22westtoerns%3Ainventarisatiejaar%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23integer%22%7D%2C%22actief1juli%22%3A%7B%22%40id%22%3A%22westtoerns%3Aactief1juli%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22publicatieOpRegiowebsitesEnVisitWVL%22%3A%7B%22%40id%22%3A%22westtoerns%3ApublicatieOpRegiowebsitesEnVisitWVL%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22identificator%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Fadms%23identifier%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22Identificator%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Fadms%23Identifier%22%2C%22Identificator.identificator%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2F2004%2F02%2Fskos%2Fcore%23notation%22%7D%2C%22lokaleId%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgeneriek%23lokaleIdentificator%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23string%22%7D%2C%22versieId%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgeneriek%23versieIdentificator%22%7D%2C%22naamruimte%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgeneriek%23naamruimte%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23string%22%7D%2C%22behoortTotMacroproduct%22%3A%7B%22%40container%22%3A%22%40set%22%2C%22%40id%22%3A%22westtoerns%3AbehoortTotMacroproduct%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22behoortTotSamengesteldProduct%22%3A%7B%22%40container%22%3A%22%40set%22%2C%22%40id%22%3A%22westtoerns%3AbehoortTotSamengesteldProduct%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22behoortTotProductlijst%22%3A%7B%22%40container%22%3A%22%40set%22%2C%22%40id%22%3A%22westtoerns%3AbehoortTotProductlijst%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22generatedAtTime%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Fprov%23generatedAtTime%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23dateTime%22%7D%2C%22isVersieVan%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2FisVersionOf%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22Product.status%22%3A%7B%22%40id%22%3A%22westtoerns%3AProduct.status%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22prefLabel%22%3A%7B%22%40id%22%3A%22skos%3AprefLabel%22%7D%2C%22heeftKwaliteitslabel%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23heeftKwaliteitsLabel%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22Kwaliteitslabel%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23Kwaliteitslabel%22%2C%22heeftAuteur%22%3A%7B%22%40id%22%3A%22schema%3Aauthor%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22toegekendOp%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2Fissued%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23dateTime%22%7D%2C%22Faciliteit%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23Faciliteit%22%2C%22heeftFaciliteit%22%3A%7B%22%40id%22%3A%22schema%3AamenityFeature%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22isSpecialisatieVan%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23isSpecialisatieVan%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22beschikbaarheid%22%3A%7B%22%40container%22%3A%22%40set%22%2C%22%40id%22%3A%22schema%3AhoursAvailable%22%7D%2C%22beschikbaarTot%22%3A%7B%22%40id%22%3A%22schema%3AvalidThrough%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23dateTime%22%7D%2C%22beschikbaarVan%22%3A%7B%22%40id%22%3A%22schema%3AvalidFrom%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23dateTime%22%7D%2C%22opent%22%3A%7B%22%40id%22%3A%22schema%3Aopens%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23time%22%7D%2C%22sluit%22%3A%7B%22%40id%22%3A%22schema%3Acloses%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23time%22%7D%2C%22dagInWeek%22%3A%7B%22%40id%22%3A%22schema%3AdayOfWeek%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22interneOpmerking%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23comment%22%7D%2C%22locatieaanduiding%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23locatieaanduiding%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22Locatieaanduiding.type%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fadres%23Locatieaanduiding.type%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22Locatieaanduiding%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Fprov%23Location%22%2C%22Locatieaanduiding.aanduiding%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23label%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22land%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fadres%23land%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22heeftRegistratie%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23heeftRegistratie%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22registratie%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Fregorg%23registration%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22Registratie%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23Registratie%22%2C%22registratieStatus%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23registratieStatus%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22Stopzetting%22%3A%22westtoerns%3AStopzetting%22%2C%22Stopzetting.reden%22%3A%7B%22%40id%22%3A%22westtoerns%3AStopzetting.reden%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23string%22%7D%2C%22Overname%22%3A%22westtoerns%3AOvername%22%2C%22Oprichting%22%3A%22westtoerns%3AOprichting%22%2C%22isStopgezet%22%3A%7B%22%40id%22%3A%22westtoerns%3AisStopgezet%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22isOvergenomen%22%3A%7B%22%40id%22%3A%22westtoerns%3AisOvergenomen%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22type%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2Ftype%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22verantwoordelijkeOrganisatie%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Fprov%23wasAssociatedWith%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22aantalKamers%22%3A%7B%22%40id%22%3A%22schema%3AnumberOfRooms%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23integer%22%7D%2C%22aantalSlaapplaatsen%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23aantalSlaapplaatsen%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23integer%22%7D%2C%22aantalVerhuureenheden%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23aantalVerhuureenheden%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23integer%22%7D%2C%22heeftOprichting%22%3A%7B%22%40id%22%3A%22westtoerns%3AheeftOprichting%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22Activiteit.jaar%22%3A%7B%22%40id%22%3A%22westtoerns%3AActiviteit.jaar%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23integer%22%7D%2C%22Activiteit.maand%22%3A%7B%22%40id%22%3A%22westtoerns%3AActiviteit.maand%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23integer%22%7D%2C%22heeftOvername%22%3A%7B%22%40id%22%3A%22westtoerns%3AheeftOvername%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22heeftStopzetting%22%3A%7B%22%40id%22%3A%22westtoerns%3AheeftStopzetting%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22wettelijkeNaam%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Fregorg%23legalName%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22heeftUitbater%22%3A%7B%22%40id%22%3A%22westtoerns%3AheeftUitbater%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22heeftMedia%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23heeftMedia%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22seeAlso%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23seeAlso%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22contentUrl%22%3A%7B%22%40id%22%3A%22schema%3AcontentUrl%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22publicatieDatum%22%3A%7B%22%40id%22%3A%22schema%3AdatePublished%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23dateTime%22%7D%2C%22copyright%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fschema.org%2FcopyrightNotice%22%7D%2C%22isSpotlight%22%3A%7B%22%40id%22%3A%22westtoerns%3AisSpotlight%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22sortOrder%22%3A%7B%22%40id%22%3A%22westtoerns%3AsortOrder%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23integer%22%7D%2C%22CreatiefWerk%22%3A%22schema%3ACreativeWork%22%2C%22url%22%3A%7B%22%40id%22%3A%22schema%3Aurl%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23anyURI%22%7D%2C%22heeftContactpersoon%22%3A%7B%22%40id%22%3A%22westtoerns%3AheeftContactpersoon%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22Persoon%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Fperson%23Person%22%2C%22voornaam%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fxmlns.com%2Ffoaf%2F0.1%2FgivenName%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23string%22%7D%2C%22achternaam%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fxmlns.com%2Ffoaf%2F0.1%2FfamilyName%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23string%22%7D%2C%22contactType%22%3A%7B%22%40id%22%3A%22schema%3AcontactType%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22prijs%22%3A%7B%22%40id%22%3A%22schema%3Aamount%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22Geldbedrag%22%3A%7B%22%40id%22%3A%22schema%3AMonetaryAmount%22%7D%2C%22waarde%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fschema.org%2Fvalue%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23double%22%7D%2C%22valuta%22%3A%22schema%3Acurrency%22%2C%22isGratis%22%3A%7B%22%40id%22%3A%22schema%3AisAccessibleForFree%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22Beoordeling%22%3A%22schema%3ARating%22%2C%22heeftOfficieleBeoordeling%22%3A%7B%22%40id%22%3A%22schema%3AstarRating%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22hoogsteBeoordeling%22%3A%7B%22%40id%22%3A%22schema%3AbestRating%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23string%22%7D%2C%22laagsteBeoordeling%22%3A%7B%22%40id%22%3A%22schema%3AworstRating%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23string%22%7D%2C%22beoordeling%22%3A%7B%22%40id%22%3A%22schema%3AratingValue%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23string%22%7D%2C%22gewijzigdDoor%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Fprov%23wasAttributedTo%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22uitsluitenVanPublicatie%22%3A%7B%22%40id%22%3A%22westtoerns%3AuitsluitenVanPublicatie%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22uitsluitenVanJaarlijkseBevraging%22%3A%7B%22%40id%22%3A%22westtoerns%3AuitsluitenVanJaarlijkseBevraging%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22broader%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2F2004%2F02%2Fskos%2Fcore%23broader%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22exactMatch%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fwww.w3.org%2F2004%2F02%2Fskos%2Fcore%23exactMatch%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22heeftAlsLocatie%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fwww.datatourisme.fr%2Fontology%2Fcore%23isLocatedAt%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22Plaats%22%3A%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2FLocation%22%2C%22heeftAlsContact%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fwww.datatourisme.fr%2Fontology%2Fcore%23hasContact%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22Organisatie%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Forg%23Organization%22%2C%22doorsturenNaarUiT%22%3A%7B%22%40id%22%3A%22westtoerns%3AdoorsturenNaarUiT%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22heeftKenmerk%22%3A%7B%22%40id%22%3A%22westtoerns%3AheeftKenmerk%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22heeftRuimte%22%3A%7B%22%40id%22%3A%22westtoerns%3AheeftRuimte%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22KwantitatieveWaarde%22%3A%22schema%3AQuantitativeValue%22%2C%22KwantitatieveWaarde.eenheid%22%3A%7B%22%40id%22%3A%22schema%3AunitText%22%7D%2C%22KwantitatieveWaarde.standaardEenheid%22%3A%7B%22%40id%22%3A%22schema%3AunitCode%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22Zaal%22%3A%22https%3A%2F%2Fwww.datatourisme.fr%2Fontology%2Fcore%23MultiPurposeRoomOrCommunityRoom%22%2C%22heeftAlsVloerOppervlakte%22%3A%7B%22%40id%22%3A%22schema%3AfloorSize%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22hoogte%22%3A%7B%22%40id%22%3A%22schema%3Aheight%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22aantalSubzalen%22%3A%7B%22%40id%22%3A%22westtoerns%3AaantalSubzalen%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23integer%22%7D%2C%22heeftIndeling%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fwww.datatourisme.fr%2Fontology%2Fcore%23hasLayout%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22Indeling%22%3A%22https%3A%2F%2Fwww.datatourisme.fr%2Fontology%2Fcore%23RoomLayout%22%2C%22indelingBeschikbaar%22%3A%7B%22%40id%22%3A%22westtoerns%3AindelingBeschikbaar%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23boolean%22%7D%2C%22heeftInvoerder%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fpurl.org%2Fdc%2Fterms%2Fcontributor%22%2C%22%40type%22%3A%22%40id%22%7D%2C%22capaciteit%22%3A%7B%22%40id%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23capaciteit%22%2C%22%40type%22%3A%22%40id%22%2C%22%40container%22%3A%22%40set%22%7D%2C%22aantalVerblijfplaatsen%22%3A%7B%22%40id%22%3A%22westtoerns%3AaantalVerblijfplaatsen%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23integer%22%7D%7D%7D) het resultaat.

In code betekent dit dat je een JSON-LD library moet gebruiken om compaction toe te passen.

## Overzicht van producten in JSON-LD

Om een overzicht van producten in JSON-LD te krijgen, doen we hetzelfde als vorige stap (Content-Type JSON-LD), mits volgende aanpassing:
in plaats van een VALUES block te gebruiken, voegen we een SELECT query (zoals eerste stap) toe in de WHERE-clausule.

Dit ziet er ruwweg zo uit:
```
CONSTRUCT {
  ?product ?p ?o .
  ?o ?p2 ?o2 .
  ?o2 ?p3 ?o3 .
} WHERE {
  GRAPH <urn:x-arq:DefaultGraph> {
    ### SELECT in plaats van VALUES
    {
      SELECT DISTINCT ?product
      WHERE { ... }
    }
    ###

    ?product ?p ?o .
    OPTIONAL {
    ?o ?p2 ?o2 .
      OPTIONAL {
      ?o2 ?p3 ?o3 .
      }
    }
  }
}
```

Voorbeeld-query:

```
PREFIX owl: <http://www.w3.org/2002/07/owl#>
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

CONSTRUCT {
  ?product ?p ?o .
  ?o ?p2 ?o2 .
  ?o2 ?p3 ?o3 .
}
WHERE {
  GRAPH <urn:x-arq:DefaultGraph> {
  {
    SELECT DISTINCT ?productVersie
    WHERE {
      ?product a schema:TouristAttraction .

      OPTIONAL {
        ?product adms:identifier [ skos:notation ?winId ]
      }

      OPTIONAL {
        ?product prov:generatedAtTime ?wijzigingsdatum .
      }

      OPTIONAL {
        ?product schema:amenityFeature [
            schema:name ?faciliteit
        ]
      }

      OPTIONAL {
        SELECT DISTINCT ?product (group_concat(?faciliteit;separator=', ') as ?faciliteiten)
        WHERE {
          ?product schema:amenityFeature [
              schema:name ?faciliteit
          ]
        } 
        GROUP BY ?product
      }

      OPTIONAL {
        ?product schema:contactPoint [
            foaf:page ?website ;
            schema:email ?email ;
            schema:telephone ?telefoon
        ]
      }

      OPTIONAL {
        ?product schema:additionalType ?type .
        ?type skos:prefLabel ?typeLabel .
        FILTER (lang(?typeLabel) = 'nl')
      }

      OPTIONAL {
        ?product westtoerns:Product.status/skos:prefLabel ?status .
        FILTER (lang(?status) = 'nl')
      }

      OPTIONAL {
        ?product locn:geometry [
            wgs84:lat ?lat ;
            wgs84:long ?long 
        ] .
      }

      OPTIONAL {
        ?product schema:name ?naam .
        FILTER (lang(?naam) = 'nl')
      }

      OPTIONAL {
        ?product schema:description ?beschrijving .
        FILTER (lang(?beschrijving) = 'nl')
      }

      OPTIONAL {
        ?product locn:address [
            locn:thoroughfare ?straatnaam ;
            adres:Adresvoorstelling.huisnummer ?huisnummer ;
            locn:postCode ?postcode ;
            adres:gemeentenaam ?gemeente ;
            westtoerns:gemeenteniscode ?niscode ;
            ] .
      }
      OPTIONAL {
        ?product locn:address [
            adres:Adresvoorstelling.busnummer ?busnummer
        ] .
      }
      OPTIONAL {
        ?product locn:address [
            locn:adminUnitL2 ?provincie
        ] .
        FILTER (lang(?provincie) = "nl")
      }

      OPTIONAL {
        ?product logies:behoortTotToeristischeRegio ?toeristischeregio .
        ?toeristischeregio owl:sameAs ?toeristischeregioTVL .
        BIND (str(?toeristischeregioTVL) as ?toeristischeregioTVLId) .
        ?toeristischeregio skos:prefLabel ?toeristischeregioLabel .
        FILTER (lang(?toeristischeregioLabel) = 'nl')
      }

      # Filter op WinId
      # FILTER (str(?winId) = "1000309")

      # Enkel producttypes onder bepaald hoofdtype
      # ?parentType skos:prefLabel "Permanent Aanbod"@nl ;
      #           skos:narrower+ ?type .
    }
    LIMIT 10000
    OFFSET 0
  }
  
  ?product ?p ?o .
  OPTIONAL {
    ?o ?p2 ?o2 .
    OPTIONAL {
      ?o2 ?p3 ?o3 .
    }
  }
  }
}
```

### Overzicht van productlijsten

```
PREFIX dcm: <http://purl.org/dc/dcmitype/>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
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
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT DISTINCT ?productlist ?wijzigingsdatum ?name ?description ?product 
WHERE {
  GRAPH <urn:x-arq:DefaultGraph> {
    ?productlist a dcmitype:Collection ;
    	prov:generatedAtTime ?wijzigingsdatum .
     # geef niet samengestelde producten terug (zijn zowel een collectie als tourist attraction)
   	FILTER NOT EXISTS { ?productlist a schema:TouristAttraction . }

    OPTIONAL {
      ?productlist schema:name ?name .
      FILTER (lang(?name) = "nl")
    }

    OPTIONAL {
      ?productlist schema:description ?description .
      FILTER (lang(?description) = "nl")
    }
    OPTIONAL {
      ?productlist dcterms:hasPart ?product .
    }
  }
}
```

Gebruik de query [hierboven](#alle-info-van-1-specifiek-product) om alle info van de specifieke producten binnen de lijst op te halen.

## Visualisatie

YASGUI kan gebruikt worden om makkelijk de producten op een kaart te plotten.
Klik [hier](http://yasgui.triply.cc/#query=PREFIX%20owl%3A%20%3Chttp%3A%2F%2Fwww.w3.org%2F2002%2F07%2Fowl%23%3E%0APREFIX%20foaf%3A%20%3Chttp%3A%2F%2Fxmlns.com%2Ffoaf%2F0.1%2F%3E%0APREFIX%20geosparql%3A%20%3Chttp%3A%2F%2Fwww.opengis.net%2Font%2Fgeosparql%23%3E%0APREFIX%20adres%3A%20%3Chttps%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fadres%23%3E%0APREFIX%20skos%3A%20%3Chttp%3A%2F%2Fwww.w3.org%2F2004%2F02%2Fskos%2Fcore%23%3E%0APREFIX%20dcterms%3A%20%3Chttp%3A%2F%2Fpurl.org%2Fdc%2Fterms%2F%3E%0APREFIX%20generiek%3A%20%3Chttps%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgeneriek%23%3E%0APREFIX%20logies%3A%20%3Chttps%3A%2F%2Fdata.vlaanderen.be%2Fns%2Flogies%23%3E%0APREFIX%20log%3A%20%3Chttp%3A%2F%2Fwww.w3.org%2F2000%2F10%2Fswap%2Flog%23%3E%0APREFIX%20locn%3A%20%3Chttp%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23%3E%0APREFIX%20prov%3A%20%3Chttp%3A%2F%2Fwww.w3.org%2Fns%2Fprov%23%3E%0APREFIX%20schema%3A%20%3Chttps%3A%2F%2Fschema.org%2F%3E%0APREFIX%20westtoerns%3A%20%3Chttps%3A%2F%2Fwesttoer.be%2Fns%23%3E%0APREFIX%20dcmitype%3A%20%3Chttp%3A%2F%2Fpurl.org%2Fdc%2Fdcmitype%2F%3E%0APREFIX%20wgs84%3A%20%3Chttp%3A%2F%2Fwww.w3.org%2F2003%2F01%2Fgeo%2Fwgs84_pos%23%3E%0APREFIX%20xsd%3A%20%3Chttp%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23%3E%0APREFIX%20adms%3A%20%3Chttp%3A%2F%2Fwww.w3.org%2Fns%2Fadms%23%3E%0A%0ASELECT%20DISTINCT%20%3FwinId%20%3Fwijzigingsdatum%20%3Fbeschrijving%20%3Fnaam%20%3FtypeLabels%20%3Fstatus%20%3Ffaciliteiten%20%3Fhuisnummer%20%3Fstraatnaam%20%3Fgemeente%20%3Fprovincie%20%3Fpostcode%20%3Fniscode%20%3Fwebsite%20%3Femail%20%3Ftelefoonnummers%20%3Flat%20%3Flong%20%3FtoeristischeregioTVLId%20%3FtoeristischeregioLabel%20%3Fproduct%20%3Fwkt%0AWHERE%20%7B%0A%20%20%20%20GRAPH%20%3Curn%3Ax-arq%3ADefaultGraph%3E%20%7B%0A%20%20%20%20%20%20%3Fproduct%20a%20schema%3ATouristAttraction%20.%0A%0A%20%20%20%20%20%20OPTIONAL%20%7B%0A%20%20%20%20%20%20%20%20%20%20%3Fproduct%20adms%3Aidentifier%20%5B%20skos%3Anotation%20%3FwinId%20%5D%0A%20%20%20%20%20%20%7D%0A%0A%20%20%20%20%20%20OPTIONAL%20%7B%0A%20%20%20%20%20%20%20%20%20%20%3Fproduct%20prov%3AgeneratedAtTime%20%3Fwijzigingsdatum%20.%0A%20%20%20%20%20%20%7D%0A%0A%20%20%20%20%20%20OPTIONAL%20%7B%0A%20%20%20%20%20%20%20%20%20%20%3Fproduct%20schema%3AamenityFeature%20%5B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20schema%3Aname%20%3Ffaciliteit%0A%20%20%20%20%20%20%20%20%20%20%5D%0A%20%20%20%20%20%20%7D%0A%0A%20%20%20%20%20%20OPTIONAL%20%7B%0A%20%20%20%20%20%20%20%20SELECT%20DISTINCT%20%3Fproduct%20(group_concat(%3Ffaciliteit%3Bseparator%3D'%2C%20')%20as%20%3Ffaciliteiten)%0A%20%20%20%20%20%20%20%20%20%20%20WHERE%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%3Fproduct%20schema%3AamenityFeature%20%5B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20schema%3Aname%20%3Ffaciliteit%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%5D%0A%20%20%20%20%20%20%20%20%20FILTER(lang(%3Ffaciliteit)%20%3D%20'nl')%0A%20%20%20%20%20%20%20%20%20%20%20%7D%20%0A%20%20%20%20%20%20%20%20%20%20%20%20GROUP%20BY%20%3Fproduct%0A%20%20%20%20%20%20%7D%0A%0A%20%20%20%20%20%20OPTIONAL%20%7B%0A%20%20%20%20%20%20%20%20%20%20%3Fproduct%20schema%3AcontactPoint%20%5B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20foaf%3Apage%20%3Fwebsite%20%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20schema%3Aemail%20%3Femail%20%0A%20%20%20%20%20%20%20%20%20%20%5D%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%0A%20%20%20%20%09OPTIONAL%20%7B%0A%20%20%20%20%20%20%20%20%20%20SELECT%20DISTINCT%20%3Fproduct%20(group_concat(%3Ftelefoon%3Bseparator%3D'%2C%20')%20as%20%3Ftelefoonnummers)%0A%20%20%20%20%20%20%20%20%20%20%20%20%20WHERE%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%3Fproduct%20schema%3AcontactPoint%2Fschema%3Atelephone%20%3Ftelefoon%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20GROUP%20BY%20%3Fproduct%0A%20%20%20%20%20%20%20%20%7D%0A%0A%20%20%20%20%20%20OPTIONAL%20%7B%0A%20%20%20%20%20%20%20%20%20%20%3Fproduct%20schema%3AadditionalType%20%3Ftype%20.%0A%20%20%20%20%20%20%20%20%20%20%3Ftype%20skos%3AprefLabel%20%3FtypeLabel%20.%0A%20%20%20%20%20%20%20%20%20%20FILTER%20(lang(%3FtypeLabel)%20%3D%20'nl')%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%0A%20%20%20%20%20%20OPTIONAL%20%7B%0A%20%20%20%20%20%20%20%20%20%20SELECT%20DISTINCT%20%3Fproduct%20(group_concat(%3FtypeLabel%3Bseparator%3D'%2C%20')%20as%20%3FtypeLabels)%0A%20%20%20%20%20%20%20%20%20%20%20%20%20WHERE%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%3Fproduct%20schema%3AadditionalType%20%3Ftype%20.%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%3Ftype%20skos%3AprefLabel%20%3FtypeLabel%20.%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20FILTER%20(lang(%3FtypeLabel)%20%3D%20'nl')%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20GROUP%20BY%20%3Fproduct%0A%20%20%20%20%20%20%20%20%7D%0A%0A%20%20%20%20%20%20OPTIONAL%20%7B%0A%20%20%20%20%20%20%20%20%3Fproduct%20westtoerns%3AProduct.status%2Fskos%3AprefLabel%20%3Fstatus%20.%0A%20%20%20%20%20%20%20%20FILTER%20(lang(%3Fstatus)%20%3D%20'nl')%0A%20%20%20%20%20%20%7D%0A%0A%20%20%20%20%20%20OPTIONAL%20%7B%0A%20%20%20%20%20%20%20%20%20%20%3Fproduct%20locn%3Ageometry%20%5B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20wgs84%3Alat%20%3Flat%20%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20wgs84%3Along%20%3Flong%20%0A%20%20%20%20%20%20%20%20%20%20%5D%20.%0A%20%20%20%20%20%20%20%20%20%20BIND%20(STRDT(CONCAT('POINT('%2C%20str(%3Flong)%2C%20'%20'%2C%20str(%3Flat)%2C%20')')%2C%20%3Chttp%3A%2F%2Fwww.opengis.net%2Font%2Fgeosparql%23wktLiteral%3E)%20as%20%3Fwkt)%0A%20%20%20%20%20%20%7D%0A%0A%20%20%20%20%20%20OPTIONAL%20%7B%0A%20%20%20%20%20%20%20%20%20%20%3Fproduct%20schema%3Aname%20%3Fnaam%20.%0A%20%20%20%20%20%20%20%20%20%20FILTER%20(lang(%3Fnaam)%20%3D%20'nl')%0A%20%20%20%20%20%20%7D%0A%0A%20%20%20%20%20%20OPTIONAL%20%7B%0A%20%20%20%20%20%20%20%20%20%20%3Fproduct%20schema%3Adescription%20%3Fbeschrijving%20.%0A%20%20%20%20%20%20%20%20%20%20FILTER%20(lang(%3Fbeschrijving)%20%3D%20'nl')%0A%20%20%20%20%20%20%7D%0A%0A%20%20%20%20%20%20OPTIONAL%20%7B%0A%20%20%20%20%20%20%20%20%20%20%3Fproduct%20locn%3Aaddress%20%5B%0A%20%20%20%20%20%20%20%20%20%20locn%3Athoroughfare%20%3Fstraatnaam%20%3B%0A%20%20%20%20%20%20%20%20%20%20adres%3AAdresvoorstelling.huisnummer%20%3Fhuisnummer%20%3B%0A%20%20%20%20%20%20%20%20%20%20locn%3ApostCode%20%3Fpostcode%20%3B%0A%20%20%20%20%20%20%20%20%20%20adres%3Agemeentenaam%20%3Fgemeente%20%3B%0A%20%20%20%20%20%20%20%20%20%20westtoerns%3Agemeenteniscode%20%3Fniscode%20%3B%0A%20%20%20%20%20%20%20%20%20%20%5D%20.%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20OPTIONAL%20%7B%0A%20%20%20%20%20%20%20%20%20%20%3Fproduct%20locn%3Aaddress%20%5B%0A%20%20%20%20%20%20%20%20%20%20adres%3AAdresvoorstelling.busnummer%20%3Fbusnummer%0A%20%20%20%20%20%20%20%20%20%20%5D%20.%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20OPTIONAL%20%7B%0A%20%20%20%20%20%20%20%20%20%20%3Fproduct%20locn%3Aaddress%20%5B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20locn%3AadminUnitL2%20%3Fprovincie%0A%20%20%20%20%20%20%20%20%20%20%5D%20.%0A%20%20%20%20%20%20%20%20%20%20FILTER%20(lang(%3Fprovincie)%20%3D%20%22nl%22)%0A%20%20%20%20%20%20%7D%0A%0A%20%20%20%20%20%20OPTIONAL%20%7B%0A%20%20%20%20%20%20%20%20%20%20%3Fproduct%20logies%3AbehoortTotToeristischeRegio%20%3Ftoeristischeregio%20.%0A%20%20%20%20%20%20%20%20%20%20%3Ftoeristischeregio%20owl%3AsameAs%20%3FtoeristischeregioTVL%20.%0A%20%20%20%20%20%20%20%20%20%20BIND%20(str(%3FtoeristischeregioTVL)%20as%20%3FtoeristischeregioTVLId)%20.%0A%20%20%20%20%20%20%20%20%20%20%3Ftoeristischeregio%20skos%3AprefLabel%20%3FtoeristischeregioLabel%20.%0A%20%20%20%20%20%20%20%20%20%20FILTER%20(lang(%3FtoeristischeregioLabel)%20%3D%20'nl')%0A%20%20%20%20%20%20%7D%0A%0A%20%20%20%20%23%20Filter%20op%20WinId%0A%20%20%20%20%23%20FILTER%20(str(%3FwinId)%20%3D%20%221000309%22)%0A%0A%20%20%20%20%20%23%20Enkel%20producttypes%20onder%20%22Logies%22%0A%20%20%20%20%23%3FparentType%20skos%3AprefLabel%20%22Logies%22%40nl%20%3B%0A%20%20%20%20%23%20%20%20%20%20%20%20%20%20%20%20skos%3Anarrower%2B%20%3Ftype%20.%0A%0A%20%20%20%20%23%20Enkel%20producttypes%20onder%20%22Eet-%20en%20drinkgelegenheden%22%0A%20%20%20%20%23%3FparentType%20skos%3AprefLabel%20%22Eet-%20en%20drinkgelegenheden%22%40nl%20%3B%0A%20%20%20%20%23%20%20%20%20%20%20%20%20%20%20%20skos%3Anarrower%2B%20%3Ftype%20.%0A%0A%20%20%20%20%23%20Enkel%20producttypes%20onder%20%22MICE%22%0A%20%20%20%20%23%3FparentType%20skos%3AprefLabel%20%22MICE%22%40nl%20%3B%0A%20%20%20%20%23%20%20%20%20%20%20%20%20%20%20%20skos%3Anarrower%2B%20%3Ftype%20.%0A%0A%20%20%20%20%23%20Enkel%20producttypes%20onder%20%22Permanent%20Aanbod%22%0A%20%20%20%20%23%3FparentType%20skos%3AprefLabel%20%22Permanent%20Aanbod%22%40nl%20%3B%0A%20%20%20%20%23%20%20%20%20%20%20%20%20%20%20%20skos%3Anarrower%2B%20%3Ftype%20.%0A%0A%20%20%20%20%23%20Enkel%20producttypes%20onder%20%22Tijdelijk%20aanbod%22%0A%20%20%20%20%23%3FparentType%20skos%3AprefLabel%20%22Tijdelijk%20aanbod%22%40nl%20%3B%0A%20%20%20%20%23%20%20%20%20%20%20%20%20%20%20%20skos%3Anarrower%2B%20%3Ftype%20.%0A%0A%20%20%20%20%23%20Wijzingen%20sedert%20timestamp%0A%20%20%20%20%23%20FILTER%20(%3Fwijzigingsdatum%20%3E%3D%20%222024-02-05T18%3A07%3A52Z%22%5E%5E%3Chttp%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23dateTime%3E)%0A%20%20%7D%0A%7D%0ALIMIT%20500&endpoint=http%3A%2F%2Flocalhost%3A3030%2Fds%2Fsparql&requestMethod=POST&tabTitle=Query&headers=%7B%7D&contentTypeConstruct=application%2Fn-triples%2C*%2F*%3Bq%3D0.9&contentTypeSelect=application%2Fsparql-results%2Bjson%2C*%2F*%3Bq%3D0.9&outputFormat=geo&outputSettings=%7B%22visualization%22%3A%22heatmap%22%7D) om het overzicht op een kaart te zien.

## Producttypes oplijsten

```
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
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

SELECT DISTINCT ?type ?label
WHERE {
    GRAPH <urn:x-arq:DefaultGraph> {
      ?type skos:prefLabel ?label .
      FILTER (contains(str(?type), "https://westtoer.be/id/concept/producttype/"))
      FILTER(lang(?label) = 'nl')
  }
}
```
