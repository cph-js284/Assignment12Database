# Assignment12Database
This is the 12th. assignment for PBA database soft2019spring

```
LOAD CSV WITH HEADERS FROM "file:///some2016UKgeotweets.csv" AS row  FIELDTERMINATOR ";"
WITH row
MERGE (a:tweet{
	username:row.`User Name`,
    nickname:row.Nickname,
    place:row.`Place (as appears on Bio)`,
    latt:toFloat(row.Latitude),
    long:toFloat(row.Longitude),
    text:row.`Tweet content`
})
return a
LIMIT 10
```
