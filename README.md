# Property Graph

Data downloaded from :

https://www.pa.martin.fl.us/tools-downloads/data-downloads/real-master/download


Before importing into Neo4j, we need to clean up the double quotes.

Replace all the double quotes with nothing:
Using vi run this command :%s/"//g


Schema
------


	CREATE CONSTRAINT ON (n:Location) ASSERT n.id IS UNIQUE;
	CREATE CONSTRAINT ON (n:Owner) ASSERT n.name IS UNIQUE;
	CREATE INDEX ON :Address(addr, zip);


Data Exploration
----------------


	LOAD CSV WITH HEADERS from 'https://raw.githubusercontent.com/maxdemarzi/property_graph/master/smaller.csv' as row
	FIELDTERMINATOR '\t'
	RETURN toInteger(TRIM(row.LRSN)) AS id, TRIM(row.PIN) AS pin, TRIM(row.LocAddr) AS addr, TRIM(row.LocCity) AS city, TRIM(row.LocState) AS state, TRIM(row.LocZip) AS zip, 
	TRIM(row.Owner1) AS owner, 	TRIM(row.Owner2) AS owner2,	TRIM(row.MailAddr) AS addr2, TRIM(row.MailCity) AS city2, TRIM(row.MailStat) AS state2, TRIM(row.MailZip) AS zip2
	, TRIM(row.LegalAc) AS acres, TRIM(row.PCDesc) AS parcel_type, TRIM(row.Legal1) AS legal1, TRIM(row.Legal2) AS legal2, TRIM(row.Legal3) AS legal3, toInteger(TRIM(row.LandVal1)) AS land_value, toInteger(TRIM(row.DwlgVal1)) AS dwelling_value, toInteger(TRIM(row.TotVal1)) AS total_value
	LIMIT 10


Import
------


	// Nodes
	USING PERIODIC COMMIT
	LOAD CSV WITH HEADERS from 'https://raw.githubusercontent.com/maxdemarzi/property_graph/master/smaller.csv' as row
	FIELDTERMINATOR '\t'
	CREATE (l:Location)
	SET l.id = toInteger(TRIM(row.LRSN)), l.pin = TRIM(row.PIN), l.addr = TRIM(row.LocAddr), l.city = TRIM(row.LocCity),
	l.state = TRIM(row.LocState), l.zip = TRIM(row.LocZip), l.acres = TRIM(row.LegalAc), l.parcel_type = TRIM(row.PCDesc), 
	l.legal1 = TRIM(row.Legal1),l.legal2 = TRIM(row.Legal2), l.legal3 = TRIM(row.Legal3), l.land_value = toInteger(TRIM(row.LandVal1)), 
	l.dwelling_value = toInteger(TRIM(row.DwlgVal1)), l.total_value = toInteger(TRIM(row.TotVal1));


	USING PERIODIC COMMIT
	LOAD CSV WITH HEADERS from 'https://raw.githubusercontent.com/maxdemarzi/property_graph/master/smaller.csv' as row
	FIELDTERMINATOR '\t'
	MERGE (p:Owner {name: TRIM(row.Owner1)});


	USING PERIODIC COMMIT
	LOAD CSV WITH HEADERS from 'https://raw.githubusercontent.com/maxdemarzi/property_graph/master/smaller.csv' as row
	FIELDTERMINATOR '\t'
	WITH row WHERE TRIM(row.Owner2) <> ""
	MERGE (p:Owner {name: TRIM(row.Owner2)});


	USING PERIODIC COMMIT
	LOAD CSV WITH HEADERS from 'https://raw.githubusercontent.com/maxdemarzi/property_graph/master/smaller.csv' as row
	FIELDTERMINATOR '\t'
	WITH row WHERE TRIM(row.MailAddr) <> "NO ADDRESS AVAILABLE"
	MERGE (a:Address {addr: TRIM(row.MailAddr), zip:TRIM(row.MailZip)})
	ON CREATE SET a.city = TRIM(row.MailCity), a.state = TRIM(row.MailStat);


Relationships:


	USING PERIODIC COMMIT
	LOAD CSV WITH HEADERS from 'https://raw.githubusercontent.com/maxdemarzi/property_graph/master/smaller.csv' as row
	FIELDTERMINATOR '\t'
	MATCH (l:Location {id: toInteger(TRIM(row.LRSN))}), (p:Owner {name: TRIM(row.Owner1)})
	CREATE (p)-[:OWNS]->(l);


	USING PERIODIC COMMIT
	LOAD CSV WITH HEADERS from 'https://raw.githubusercontent.com/maxdemarzi/property_graph/master/smaller.csv' as row
	FIELDTERMINATOR '\t'
	WITH row WHERE TRIM(row.Owner2) <> ""
	MATCH (l:Location {id: toInteger(TRIM(row.LRSN))}), (p:Owner {name: TRIM(row.Owner2)})
	CREATE (p)-[:OWNS]->(l);


	USING PERIODIC COMMIT
	LOAD CSV WITH HEADERS from 'https://raw.githubusercontent.com/maxdemarzi/property_graph/master/smaller.csv' as row
	FIELDTERMINATOR '\t'
	MATCH (l:Location {id: toInteger(TRIM(row.LRSN))}), (a:Address {addr: TRIM(row.MailAddr), zip:TRIM(row.MailZip)})
	MERGE (l)-[:MAILED_AT]->(a);


Queries:


	MATCH (address)<-[:MAILED_AT]-(l:Location)
	RETURN address, SUM(l.total_value) AS values
	ORDER BY values DESC
	LIMIT 25;


	MATCH (address)<-[:MAILED_AT]-(l:Location)
	WITH address, l
	ORDER BY l.total_value DESC
	RETURN address, SUM(l.total_value) AS values, COLLECT(DISTINCT l)[0..3]
	ORDER BY values DESC
	LIMIT 25;


	CALL algo.unionFind.stream(
	  'MATCH (n) RETURN id(n) as id',
	  'MATCH (n)-->(n2)
	   RETURN id(n) as source, id(n2) as target',
	  {graph:'cypher'}
	) YIELD nodeId, setId
	RETURN algo.asNode(nodeId)AS node, setId
	LIMIT 10;


	CALL algo.unionFind(
	  'MATCH (n) RETURN id(n) as id',
	  'MATCH (n)-->(n2)
	   RETURN id(n) as source, id(n2) as target',
	  {graph:'cypher'}
	) YIELD setCount;


	MATCH (o:Owner { name: "16330B55 TRUST" }),(o2:Owner { name: "941 TARPON AVENUE TRUST" }), p = shortestPath((o)-[*]-(o2))
	RETURN p


	MATCH (n:Owner)
	RETURN n.partition, COUNT(*) AS members, COLLECT(n.name) AS names
	ORDER BY members DESC
	LIMIT 10

Some addresses and their owners:

"1015 10th Street, Lake Park"
https://www.flahomesolutions.com/our-company/
Alex Pardo

"10700 Pecan Park Blvd Ste 150"
http://www.hoovers.com/company-information/cs/company-profile.forestar_group_inc.a1a71ec022246a5f.html?aka_re=1
https://www.forestar.com/home/default.aspx

"203 MAIN ST FRANKLIN louisiana"
https://www.jmbcompanies.com/

"4750 owings mills blvd"
https://www.chesapeakerealtypartners.com/about/#bios
http://search.sunbiz.org/Inquiry/CorporationSearch/SearchResultDetail?inquirytype=EntityName&directionType=Initial&searchNameOrder=BANYANBAYMACKS%20M160000097350&aggregateId=forl-m16000009735-c29ec2c5-0f12-4a1c-9c02-97e5bf712057&searchTerm=Banyan%20Bay%20Condominium%20Association%2C%20Inc.&listNameOrder=BANYANBAYCONDOMINIUMASSOCIATIO%20N050000007540

"3900 commonwealth blvd tallahassee fl"
https://floridadep.gov

"500 e ocean blvd stuart fl"
1950 Richmond Rd. Lyndhurst, Ohio
https://my.clevelandclinic.org/locations/directions/251-cleveland-clinic-lyndhurst-campus


"8985 SE Bridge Rd, Hobe Sound, FL 33455, USA"
http://search.sunbiz.org/Inquiry/corporationsearch/SearchResultDetail?inquirytype=EntityName&directionType=Initial&searchNameOrder=BRIDGES%20P040001424411&aggregateId=domp-p04000142441-2ec7e8b3-f6ac-45b4-b8c8-4f884e260a18&searchTerm=BRIDGER%20RANGE%20INC&listNameOrder=BRIDGERRANGE%20P120001045710
http://npbc.blog.palmbeachpost.com/2017/05/22/charles-modica-buys-another-property-in-north-county/