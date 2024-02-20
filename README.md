# Setup

`docker-compose up`

## Overzicht van producten

Ga naar `http://localhost:3030/#/dataset/ds/query`

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

SELECT DISTINCT ?winId ?wijzigingsdatum ?beschrijving ?naam ?typeLabel ?status ?faciliteiten ?huisnummer ?straatnaam ?gemeente ?provincie ?postcode ?niscode ?website ?email ?telefoon ?lat ?long ?toeristischeregioTVLId ?toeristischeregioLabel
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
	   FILTER(lang(?faciliteit) = 'nl')
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
        ?productVersie logies:behoortTotToeristischeRegio ?toeristischeregio .
		?toeristischeregio dcterms:isVersionOf/owl:sameAs ?toeristischeregioTVL .
	    BIND (str(?toeristischeregioTVL) as ?toeristischeregioTVLId) .
    	?toeristischeregio skos:prefLabel ?toeristischeregioLabel .
        FILTER (lang(?toeristischeregioLabel) = 'nl')
    }

  # Filter op WinId
  # FILTER (str(?winId) = "1000309")
  
   # Enkel producttypes onder "Logies"
  #?parentType skos:prefLabel "Logies"@nl ;
  #           skos:narrower+ ?type .
  
  # Enkel producttypes onder "Eet- en drinkgelegenheden"
  #?parentType skos:prefLabel "Eet- en drinkgelegenheden"@nl ;
  #           skos:narrower+ ?type .
  
  # Enkel producttypes onder "Eet- en drinkgelegenheden"
  #?parentType skos:prefLabel "MICE"@nl ;
  #           skos:narrower+ ?type .
  
  # Enkel producttypes onder "Eet- en drinkgelegenheden"
  #?parentType skos:prefLabel "Permanent Aanbod"@nl ;
  #           skos:narrower+ ?type .
  
  # Enkel producttypes onder "Eet- en drinkgelegenheden"
  #?parentType skos:prefLabel "Tijdelijk aanbod"@nl ;
  #           skos:narrower+ ?type .
  
  # Wijzingen sedert timestamp
  # FILTER (?wijzigingsdatum >= "2024-02-05T18:07:52Z"^^<http://www.w3.org/2001/XMLSchema#dateTime>)
}
```

Onderaan de query kan gefilterd worden op WIN ID en producttypes dat onder "Permanent Aanbod" vallen.
Verwijder het spoorwegteken om uit commentaar te zetten.

Opmerkingen:
* Deze resultaten kunnen verwerkt worden als JSON (sparql11-results-json). Zie meer info [hier](https://www.w3.org/TR/sparql11-results-json/).
* Er komt een punt dat er te veel producten zijn om in 1 HTTP response te passen. Dan dient er gepagineerd te worden met LIMIT en OFFSET. Zie meer info [hier](https://www.w3.org/TR/sparql11-query/#sparqlOffsetLimit).

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
  ?productVersie ?p ?o .
  ?o ?p2 ?o2 .
  ?o2 ?p3 ?o3 .
} WHERE {
  ?productVersie ?p ?o ;
    		dcterms:isVersionOf ?product .
  OPTIONAL {
  ?o ?p2 ?o2 .
    OPTIONAL {
    ?o2 ?p3 ?o3 .
    }
  }
  VALUES ?product { <https://westtoer.be/id/product/76ec83f9-6dfb-4039-9eb7-701da08efbfd> }
  # Of met versie:
  # VALUES ?productVersie { <https://westtoer.be/id/product/76ec83f9-6dfb-4039-9eb7-701da08efbfd/2023-11-23T12:46:11.6388872Z> }
}
```

### JSON-LD - machineleesbaar

Voor verwerking in code kan het handig zijn om het product in JSON-LD terug te krijgen.

Stap 1: pas Content Type (GRAPH) aan naar "JSON-LD"
In code betekent dit dat een HTTP request met "Content-Type": "application/ld+json" wordt verstuurd naar het SPARQL endpoint.

In code betekent dit dat je een [SPARQL request](https://www.w3.org/TR/sparql11-protocol/) moet sturen naar "http://localhost:3030/ds/sparql".
Hier zie je een [voorbeeld](https://query.linkeddatafragments.org/#datasources=http%3A%2F%2Flocalhost%3A3030%2Fds%2Fsparql&query=SELECT%20%3Fs%20%3Fp%20%3Fo%0AWHERE%20%7B%0A%20%20%20%3Fs%20%3Fp%20%3Fo%20.%20%0A%7D%0ALIMIT%2010) in de Comunica client.
 
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
  ?productVersie ?p ?o .
  ?o ?p2 ?o2 .
  ?o2 ?p3 ?o3 .
} WHERE {
  ### SELECT in plaats van VALUES
  {
    SELECT DISTINCT ?productVersie
    WHERE { ... }
  }
  ###

  ?productVersie ?p ?o .
  OPTIONAL {
  ?o ?p2 ?o2 .
    OPTIONAL {
    ?o2 ?p3 ?o3 .
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
  ?productVersie ?p ?o .
  ?o ?p2 ?o2 .
  ?o2 ?p3 ?o3 .
}
WHERE {
  {
    SELECT DISTINCT ?productVersie
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
        ?productVersie logies:behoortTotToeristischeRegio ?toeristischeregio .
        ?toeristischeregio dcterms:isVersionOf/owl:sameAs ?toeristischeregioTVL .
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
  
  ?productVersie ?p ?o .
  OPTIONAL {
    ?o ?p2 ?o2 .
    OPTIONAL {
      ?o2 ?p3 ?o3 .
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


SELECT DISTINCT ?productlist ?productlistVersie ?latestGeneratedAtTime ?name ?description ?product WHERE {
  ?productlistVersie prov:generatedAtTime ?latestGeneratedAtTime ;
                     dcterms:isVersionOf ?productlist .

  OPTIONAL {
    ?productlistVersie schema:name ?name .
    FILTER (lang(?name) = "nl")
  }

  OPTIONAL {
    ?productlistVersie schema:description ?description .
    FILTER (lang(?description) = "nl")
  }
  OPTIONAL {
    ?productlistVersie dcterms:hasPart ?product .
  }
  {
    # Get latest version timestamp (generatedAtTime) for every productlist
    SELECT ?productlist (MAX(?generatedAtTime) as ?latestGeneratedAtTime)
    WHERE {
      ?productlistVersie a dcmitype:Collection ;
             dcterms:isVersionOf ?productlist ;
             prov:generatedAtTime ?generatedAtTime .
      # geef niet samengestelde producten terug (zijn zowel een collectie als tourist attraction)
      FILTER NOT EXISTS { ?productlistVersie a schema:TouristAttraction . }
    }
    GROUP BY ?productlist
    
    # Paginering is nodig wanneer er teveel lijsten zijn om in 1 HTTP response te steken
    # LIMIT 50
    # OFFSET 0
  }
}
```

Gebruik de query [hierboven](#alle-info-van-1-specifiek-product) om alle info van de specifieke producten binnen de lijst op te halen.
