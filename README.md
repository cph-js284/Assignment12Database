# Assignment12Database
This is the 12th. assignment for PBA database soft2019spring

# What it is 
This is a repo containing asingle markdown, that holds the answers for the [assignment 12](https://github.com/datsoftlyngby/soft2019spring-databases/blob/master/assignments/assignment12.md) <br>

# Setup
1) Download the zip file and unpack the zip-file
```
wget https://github.com/datsoftlyngby/soft2019spring-databases/raw/master/data/some2016UKgeotweets.csv.zip
unzip some2016UKgeotweets.csv.zip
```
2) Start up and Neo4j container
```
sudo docker run -d --name neo4j --rm --publish=7474:7474 --publish=7687:7687 -v $(pwd):/var/lib/neo4j/import --env NEO4J_AUTH=neo4j/test1234 neo4j
```
3) Launch the Neo4j IDE [http://localhost:7474/browser](http://localhost:7474/browser) <br>
*you might need to give it a few seconds to spin up before connecting*<br>
<br>
4) Enter credentials
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
MERGE (a:tweet{
	username:row.`User Name`,
    	nickname:row.Nickname,
    	place:row.`Place (as appears on Bio)`,
    	latt:toFloat(row.Latitude),
    	long:coalesce(toFloat(row.Longitude),0.0),
    	text:row.`Tweet content`,
    	mentions:(extract( m in 
                filter(m in split(row.`Tweet content`," ") where m starts with "@" and size(m) > 1) 
                | right(m,size(m)-1)))
                })
```
*Im using coalesce here for the Longitude merge - this will trip me up in excercise 3. The PERIODIC COMMIT is  merging 1000(default) rows from the csv into the db*

# Excercise 2

*Use the mentions list of each tweet to create a new set of nodes labeled "Tweeters", whith a "Mentions" relation.*
```
MATCH (a:tweet) 
UNWIND a.mentions AS handle
WITH DISTINCT handle
CREATE (b:tweeters {handle: handle})
```

*Create a relation "Tweeted" between Tweeters and Tweet.*
```
MATCH (b:tweeters), (a:tweet)
WHERE b.handle in a.mentions 
CREATE (tweeters)-[:MENTIONS]->(tweet)
```

# Excercise 3
*Find the top 10 list of tweeters whose tweets are the furtherst apart.*<br>
*Having forced 0.0 in for some of the Longitudes in Ex1 is coming back to bit me (maybe - some tweets could be sent from further away), now where I have to use the point-function*
```
MATCH (a:tweet),(b:tweet)
WHERE a.username = b.username and a <> b
WITH DISTANCE(POINT({long:a.long, latt:a.latt}), POINT({long:b.long, latt:b.latt})) as dist, a, b
ORDER BY dist DESC
RETURN a.username, dist as dist_in_mtrs
LIMIT 10
```
