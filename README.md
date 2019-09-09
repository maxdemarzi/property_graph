// Replace all the double quotes:
// Usinv vi run this command :%s/"//g

CREATE CONSTRAINT ON (n:Location) ASSERT n.id IS UNIQUE;
CREATE CONSTRAINT ON (n:Owner) ASSERT n.name IS UNIQUE;
CREATE INDEX ON :Address(addr, zip);



LOAD CSV WITH HEADERS from 'file:///real_master.csv' as row
FIELDTERMINATOR '|'
RETURN toInteger(TRIM(row.LRSN)) AS id, TRIM(row.PIN) AS pin, TRIM(row.`Public_NeiNum`) AS nei, TRIM(row.LocAddr) AS addr, TRIM(row.LocCity) AS city, TRIM(row.LocState) AS state, TRIM(row.LocZip) AS zip, 
TRIM(row.Owner1) AS owner, 	TRIM(row.Owner2) AS owner2,	TRIM(row.MailAddr) AS addr2, TRIM(row.MailCity) AS city2, TRIM(row.MailStat) AS state2, TRIM(row.MailZip) AS zip2
, TRIM(row.LegalAc) AS acres, TRIM(row.PCDesc) AS parcel_type, TRIM(row.Legal1) AS legal1, TRIM(row.Legal2) AS legal2, TRIM(row.Legal3) AS legal3, toInteger(TRIM(row.LandVal1)) AS land_value, toInteger(TRIM(row.DwlgVal1)) AS dwelling_value, toInteger(TRIM(row.TotVal1)) AS total_value
LIMIT 10


// Nodes
USING PERIODIC COMMIT
LOAD CSV WITH HEADERS from 'file:///real_master.csv' as row
FIELDTERMINATOR '|'
CREATE (l:Location)
SET l.id = toInteger(TRIM(row.LRSN)), l.pin = TRIM(row.PIN), l.nei = TRIM(row.`Public_NeiNum`), l.addr = TRIM(row.LocAddr), l.city = TRIM(row.LocCity),
l.state = TRIM(row.LocState), l.zip = TRIM(row.LocZip), l.acres = TRIM(row.LegalAc), l.parcel_type = TRIM(row.PCDesc), l.legal1 = TRIM(row.Legal1),l.legal2 = TRIM(row.Legal2), l.legal3 = TRIM(row.Legal3), l.land_value = toInteger(TRIM(row.LandVal1)), l.dwelling_value = toInteger(TRIM(row.DwlgVal1)), l.total_value = toInteger(TRIM(row.TotVal1));

USING PERIODIC COMMIT
LOAD CSV WITH HEADERS from 'file:///real_master.csv' as row
FIELDTERMINATOR '|'
MERGE (p:Owner {name: TRIM(row.Owner1)});

USING PERIODIC COMMIT
LOAD CSV WITH HEADERS from 'file:///real_master.csv' as row
FIELDTERMINATOR '|'
WITH row WHERE TRIM(row.Owner2) <> ""
MERGE (p:Owner {name: TRIM(row.Owner2)});

USING PERIODIC COMMIT
LOAD CSV WITH HEADERS from 'file:///real_master.csv' as row
FIELDTERMINATOR '|'
MERGE (a:Address {addr: TRIM(row.MailAddr), zip:TRIM(row.MailZip)})
ON CREATE SET a.city = TRIM(row.MailCity), a.state = TRIM(row.MailStat);

// Relationships
USING PERIODIC COMMIT
LOAD CSV WITH HEADERS from 'file:///real_master.csv' as row
FIELDTERMINATOR '|'
MATCH (l:Location {id: toInteger(TRIM(row.LRSN))}), (p:Owner {name: TRIM(row.Owner1)})
CREATE (p)-[:OWNS]->(l);

USING PERIODIC COMMIT
LOAD CSV WITH HEADERS from 'file:///real_master.csv' as row
FIELDTERMINATOR '|'
WITH row WHERE TRIM(row.Owner2) <> ""
MATCH (l:Location {id: toInteger(TRIM(row.LRSN))}), (p:Owner {name: TRIM(row.Owner2)})
CREATE (p)-[:OWNS]->(l);

USING PERIODIC COMMIT
LOAD CSV WITH HEADERS from 'file:///real_master.csv' as row
FIELDTERMINATOR '|'
MATCH (l:Location {id: toInteger(TRIM(row.LRSN))}), (a:Address {addr: TRIM(row.MailAddr), zip:TRIM(row.MailZip)})
MERGE (l)-[:MAILED_AT]->(a);



RETURN toInteger(TRIM(row.LRSN)) AS id, TRIM(row.PIN) AS pin, TRIM(row.`Public_NeiNum`) AS nei, TRIM(row.LocAddr) AS addr, TRIM(row.LocCity) AS city, TRIM(row.LocState) AS state, TRIM(row.LocZip) AS zip, 
TRIM(row.Owner1) AS owner, 	TRIM(row.Owner2) AS owner2,	TRIM(row.MailAddr) AS addr2, TRIM(row.MailCity) AS city2, TRIM(row.MailStat) AS state2, TRIM(row.MailZip) AS zip2
, TRIM(row.LegalAc) AS acres, TRIM(row.PCDesc) AS parcel_type, TRIM(row.Legal1) AS legal1, TRIM(row.Legal2) AS legal2, TRIM(row.Legal3) AS legal3, toInteger(TRIM(row.LandVal1)) AS land_value, toInteger(TRIM(row.DwlgVal1)) AS dwelling_value, toInteger(TRIM(row.TotVal1)) AS total_value
LIMIT 10