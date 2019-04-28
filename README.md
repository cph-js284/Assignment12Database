# Assignment12Database
This is the 12th. assignment for PBA database soft2019spring

# What it is 
This is a repo containing a single markdown, that holds the answers for the [assignment 12](https://github.com/datsoftlyngby/soft2019spring-databases/blob/master/assignments/assignment12.md) <br>

# Setup
1. Download and unpack the zip-file
```
wget https://github.com/datsoftlyngby/soft2019spring-databases/raw/master/data/some2016UKgeotweets.csv.zip
unzip some2016UKgeotweets.csv.zip
```
2. Start up and Neo4j container, execute this command from the folder containing the .csv-file
```
sudo docker run -d --name neo4j --rm --publish=7474:7474 --publish=7687:7687 -v $(pwd):/var/lib/neo4j/import --env NEO4J_AUTH=neo4j/test1234 neo4j
```
3. Launch the Neo4j IDE [http://localhost:7474/browser](http://localhost:7474/browser) <br>
*you might need to give it a few seconds to spin up before connecting*<br>

4. Enter credentials<br>

```
username : neo4j
password: test1234
```

# Excercise 1

*Write a cypher command that loads the tweets, creates a number of objects labeled "Tweet" with the columns "User Name", "Nickname","Place (as appears on Bio)", "Latitude", "Longitude" and "Tweet content" renamed as something nice (single lowercase name), and which adds a list of mentions to each Tweet.* <br>
<br>
```
USING PERIODIC COMMIT
LOAD CSV WITH HEADERS FROM "file:///some2016UKgeotweets.csv" AS row  FIELDTERMINATOR ";"
WITH row
WHERE NOT row.Longitude IS NULL
CREATE (a:tweet{
	username:row.`User Name`,
    	nickname:row.Nickname,
    	place:row.`Place (as appears on Bio)`,
    	latt:toFloat(row.Latitude),
    	long:toFloat(row.Longitude),
    	text:row.`Tweet content`,
    	mentions:(extract( m in 
                filter(m in split(row.`Tweet content`," ") where m starts with "@" and size(m) > 1) 
                | right(m,size(m)-1)))
                })
```
*The PERIODIC COMMIT is commiting 1000(default) rows from the csv into the db*<br>
<b>NB the above cipher will take several seconds depending on your hardware</b><br>
<br>
<b>Result</b><br>
```
Added 136099 labels, created 136099 nodes, set 952681 properties, completed after 17933 ms.
```

# Excercise 2

*Use the mentions list of each tweet to create a new set of nodes labeled "Tweeters", whith a "Mentions" relation.*
```
MATCH (a:tweet) 
UNWIND a.mentions AS handle
WITH DISTINCT handle
CREATE (b:tweeters {handle: handle})
```
<br>
<b>Result</b><br>

```
Added 22913 labels, created 22913 nodes, set 22913 properties, completed after 1480 ms.
```
<br>
*Create a relation "Tweeted" between Tweeters and Tweet.* 
<br>

```
MATCH (b:tweeters), (a:tweet)
WHERE b.handle in a.mentions 
CREATE (tweeters)-[:MENTIONS]->(tweet)
```
<b>NB the above cipher will take several minutes(112 on my laptop so yeah :s) depending on your hardware</b><br>
<br>
<b>Result</b>
```
Created 81880 nodes, created 40940 relationships, completed after 6708501 ms.
```

# Excercise 3
*Find the top 10 list of tweeters whose tweets are the furtherst apart.*<br>
```
MATCH (a:tweet), (b:tweet) 
WHERE a.username = b.username AND a <> b
WITH DISTANCE(POINT({longitude:a.long, latitude:a.latt}), POINT({longitude:b.long, latitude:b.latt})) as dist, a, b
RETURN DISTINCT a.nickname, max(dist) as distance
ORDER BY distance DESC
LIMIT 10
```
<b>Result</b>
```
a.nickname		distance

"v1vekv"		null
"Keith_Rapley"		null
"v1_scope"		null
"googuns_lulz"		1287687.6134860725
"tmj_GBR_mgmt"		1101277.6468134406
"outonashout"		1030581.425638201
"MarsBots"		975157.1836022508
"x333xxx"		933365.4495465886
"TangoRaindrop"		893949.2533669937
"FoodTouristBlog"	893949.2533669937

Started streaming 10 records after 199500 ms and completed after 199500 ms.
```

