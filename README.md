# Crowe Microapp Container Backend


## Authentication
Access any endpoint at anytime via `x-api-key` in the request `header`
Base endpoint is `https://8pfnywkzv9.execute-api.us-east-2.amazonaws.com/dev`

## Credentials
`x-api-key` : `o1erX2EnkqvZnNJIx1XY6ZOIg1GTzhOCasdfaasdfdasef`

## Key Concepts
- The backend interaction with the database (MongoDB) uses a tableless approach, where each collection is created on the fly. The first time something is being insert into a `{type}` will initiate that collection/table in the database. The database does not perform any data schema validation.

## API Endpoint Overview

| Category | Description |  Method  |  Endpoint |
|--|--|--|--|
| S3 | Generating an S3 READ link | POST |  /s3/read | 
| S3 | Generating an S3 UPDATE (CREATE) presigned link. The response will provide an S3 endpoint to upload the file to | POST | /s3/update | 
| Database | Query records from a database `{type}` with options to filter returns. This endpoint uses [this](https://www.npmjs.com/package/api-query-params) library for filtering  | GET |  /db/{type} | 
| Database | Query a *single* record from a database `{type}` by `_id`| GET |  /db/{type}/{_id} | 
| Database | Create/Insert record(s) to a database `{type}` | POST |  /db/{type} | 
| Database | Query records from a database `{type}` | UPDATE |  /db/{type} | 
| Database | Delete record(s) from a database by a single `{_id}` | DELETE |  /db/{type}/{_id} | 
| Database | Generating a hierchical tree of a certain database `{type}` based on the attribute `parent_id` and `_id` This is particularly useful for creating the `microapps` folder tree or nested `group` tree | GET |  /db_tree/{type} | 
| Feed | Query records from a database `feeds` with options to filter returns. This endpoint uses [this](https://www.npmjs.com/package/api-query-params) library for filtering  | GET |  /db/{type} | 
| Feed Sources | Create the Feed Sources so the Feed Cron Job can fetch from them to create `Feed` objects | POST, GET |  /db/feed_sources | 
| Feed Sources | Update the Feed Sources so the Feed Cron Job can fetch from them to create `Feed` objects | PUT, DELETE |  /db/feed_sources/{_id} | 
| Microapps | Interact with the `microapps` collection | GET / POST |  /db/microapps | 
| Microapps | Interact with the `microapps` collection and return hierarchical structure | GET / POST |  /db_tree/microapps | 
| Microapps | Interact with one `microapps` collection object | GET / PUT / DELETE |  /db/microapps/{_id} | 

## Create Feed Sources
Method: *POST* to create; *GET* to read 
Endpoint:  `/db/feed_sources`

***SUMMARY:*** This endpoint is to CREATE the list of `feed_sources` so the backend can automatically create `feeds` object which then can be read later at `/db/feeds`

***MORE EXPLAINATION:*** In order to allow the dashboard administrator to automatically pull content from another resource into the mobile app's feed section, we need to first tell the backend *where* to get the data from. What the backend will do is then to pull every 15 minutes from the list of resources (XML or JSON) and using the `objectmapper` to do data attribute mapping from ANY XML and JSON schema to map the `feeds` section of the mobile app. The CRON job in the backend will then check and see if there are new objects in a certain **Feed Source** and create that in the Feed section.

**GET** will return the exact object plus meta data of the object

#### POST Request Body

| Attribute | Type | Description |
|--|--|--|
| name | String | Descriptive name for the resource endpoint |
| default_object | Object | Telling the backend what the default feed author is in case it is not provided by the XML or JSON |
| default_object.user.picture | String | The default user picture of a feed from this feed source |
| format | String | `JSON` or `XML` so the backend knows which parser to use |
| url | String | the http/https endpoint for where the backend should fetch |
| objectmapper | String | a string to tell the system how to do the data mapping, for documentatino please see this [link](https://www.npmjs.com/package/object-mapper). |
| auto_published | Boolean | `true` or `false` Telling the backend cron job whether to set all the newly fetched item is published or not published. If not published, it is up to the dashboard administrator to manually review them before ALL app user can see them. This is particularly useful if the project pulls from various sources but have duplicate content. |
| moment_time_format | String |the `moment.format('')` of the source so the backend can interpret it and format it to the right format for the feed objects |
| auth0_role | String |the default role who can see the the feed created by this `feed source`  |
```
// POST request body

{
  "default_object": {
    "user": {
      "picture": "https://yt3.ggpht.com/ytc/AKedOLQDltcSblAekgdmVn3h_xs72sAV5KTmg5mVVOnycg=s176-c-k-c0x00ffffff-no-rj"
    },    
  },
  "format": "XML",
  "url": "https://www.youtube.com/feeds/videos.xml?channel_id=UCHI2-KkOT19I1gfik6DWsDQ",
  "objectmapper": "feed.entry.[].media:group.media:title=[].content&feed.entry.[].media:group.media:thumbnail.@_url=[].media[0]&feed.entry.[].link.@_href=[].web_url&feed.entry.[].author.name=[].user.full_name&feed.entry.[].author.uri=[].user.profile&feed.entry.[].published=[].published_date",
  "auto_published": true,
  "moment_time_format": "YYYY-MM-DDTHH:mm:ssZ",
  "auth0_role": "rol_Fk5CB6VpSluwKcMp" //may modify later based on Auth0 schema
}

```

##  Microapps
Method: *POST* to create; *GET* to read 
Endpoint:  `/db/microapps` or `/db_tree/microapps`

#### POST Request Body

| Attribute | Type | Description |
|--|--|--|
| class | String | FOLDER or WEB or MEDIA or PDF |
| title | String | Title string for the microapp |
| description | String | Description for the microapp which will be displayed in the app |
| icon | String | https url string for icon image png |
| order | Integer | for sorting which microapp is listed first |
| scope | String | Auth0 scope to tell the mobile app about access, if null, then it is allowed for anyone to access |
| is_hidden | Boolean | Telling the mobile app if it should be shown. This attribute will override scope |
| payload | Object | This is the object that carries the specific attributes for each microapp class attribute. Each different class will have different schema. |
| payload.feed_collection | String | for class MEDIA only: the api url to the collection i.e. "/db/media?data.tags=5fd9ad6b391ef3000858ff77" |
| payload.url | String | for class WEB and PDF only: the url to link to |
| payload.is_user_query | Boolean | for class WEB only: letting the mobile app know whether to append user_id query string to the web url provided so the web resource can know which user is accessing the link |

```
// POST request body

{
  "class": "FOLDER",
  "title": "Title One",
  "icon": "https://assets.tcog.hk/aemc2zfhz5.png",
  "description": "Description text",
  "order": 1,
  "scope": "READ:FEED - Business Trends Podcast",
  "is_hidden": false,
  "payload": {
  
  }
}

```

Method: *PUT* to update *DELETE* to delete *GET* to read one
Endpoint: `/db/microapps/{_id}`

***SUMMARY:*** This endpoint is to manage the `microapps` collection. Microapp is the heart and soul of this mobile app application project. It allows dashboard administrator to manage content of the mobile app dynamically. It has a number of *classes* which tells the mobile app which *renderer* to use in the mobile app and that each different *class* will have a different *data model*. Keep reading to find out various classes available so far in this app.  
