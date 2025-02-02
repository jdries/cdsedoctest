# OpenSearch

## Using OpenSearch interface to query Data Catalogue

Due to the fact that offset is not a recommended form of searching repository pages, we had to implement limit to a maximum of 200k.
The requests over the limit will be rejected with the code 400.
We encourage you to limit your inquiries by geographic or temporal area.

All queries may be executed as simple HTTP-Get calls, by typing the query in web browser address line, by using any HTTP client, e.g. curl or wget, or from inside of users' program. The database is accessible free and anonymously (open for anonymous access for everyone, no authorization is used) It may be accessed both from the internal network (virtual machines in Creodias) and from outside, e.g. your home computer. Note, that the actual EO data themselves are restricted to authorized users, only the Data Catalogue is open.

### General Rules

The queries produce results in JSON format. Base url:

[http://catalogue.dataspace.copernicus.eu/resto/api/collections/search.json?](http://catalogue.dataspace.copernicus.eu/resto/api/collections/search.json?)

Most queries are case-sensitive.

### Collections

The data are organized in so-called collections, corresponding to various satellites. A query may search for data in all collections, or in one particular collection only. If only one satellite is in the field of interest, the second approach is faster and more efficient, than filtering the general query. For example, to find 10 most recent Sentinel-2 products with cloud cover below 10%, the query should look like:
```
$ wget -O - "http://catalogue.dataspace.copernicus.eu/resto/api/collections/Sentinel2/search.json?cloudCover=[0,10]&startDate=2022-06-11&completionDate=2022-06-22&maxRecords=10"
```
while if the collection field is missing in the URL, the products from all the satellites are returned:
```
$ wget -O - "https://catalogue.dataspace.copernicus.eu/resto/api/collections/search.json?cloudCover=[0,10]&startDate=2022-06-11&completionDate=2022-06-22&maxRecords=10"
```
As for today the following collections are defined and may be used:

**Sentinel1 or SENTINEL-1**

**Sentinel2 or SENTINEL-2**

**Sentinel3 or SENTINEL-3**

**Sentinel5P or SENTINEL-5P**

Note, that collection names vary a bit from satellite names, as they are used in EO Data repository. For example, the collection is named **Sentinel2,** while in the repository its data are located within **/eodata/Sentinel-2/....** branch of the repository tree.

### Output sorting and limiting

By default, maximum 20 products are returned only. You may change the limit (beware of long execution time for queries about thousands of products) using the phrase

**maxRecords=nnn**

If the query is very general and the number of matching products is large, the next pages of products may be retrieved

**page=nnn**

You may also change the order of how the products are presented, using the phrase like

**sortParam=startDate**

will sort the output by observation date. The following orderings are implemented:

**startDate** - the date when the observation was made (start)

**completionDate** - the date when the observation was made (end)

**published ** - the date when the product got published in our repository

each of them may be accompanied by

**sortOrder=ascending or sortOrder=descending**

For example the query

[http://catalogue.dataspace.copernicus.eu/resto/api/collections/Sentinel2/search.json?startDate=2021-07-01&completionDate=2021-07-31&sortParam=startDate&maxRecords=20](http://catalogue.dataspace.copernicus.eu/resto/api/collections/Sentinel2/search.json?startDate=2016-07-01&completionDate=2016-07-31&sortParam=cloudCover&maxRecords=20)

will return 20 products from July 2021, while the next query would return the next 20:

[http://catalogue.dataspace.copernicus.eu/resto/api/collections/Sentinel2/search.json?startDate=2021-07-01&completionDate=2021-07-31&sortParam=startDate&maxRecords=20&page=2](http://catalogue.dataspace.copernicus.eu/resto/api/collections/Sentinel2/search.json?startDate=2021-07-01&completionDate=2021-07-31&sortParam=startDate&maxRecords=20&page=2)

### Formal queries

The formal query is invoked as a sequence of sub phrases, separated by &. The result is a conjunction of all sub phrases. It is impossible to use an alternative in the question. The query must be specified as a formal query.

The example of formal query - about cloudless (cloud cover lower or equal to 10%) products for a specific location:

[https://catalogue.dataspace.copernicus.eu/resto/api/collections/Sentinel2/search.json?cloudCover=[0,10]&startDate=2021-06-21&completionDate=2021-09-22&lon=21.01&lat=52.22](https://catalogue.dataspace.copernicus.eu/resto/api/collections/Sentinel2/search.json?cloudCover=%5B0,10%5D&startDate=2021-06-21&completionDate=2021-09-22&lon=21.01&lat=52.22)

The queries are in form **param=value or param=[minvalue,maxvalue**]. Most of the parameters are common for all collections, but some are specific for some them (e.g. **cloudCover** applies to optical satellites, but polarisation applies to radar ones), or just single one.

### Geography and time-frame

The common set of parameters are:

**startDate, completionDate** - the date limits of the observation. The time may also be specified, e.g. 2021-10-01T021:30:00Z

**publishedAfter, publishedBefore** - the date limits when the product was published in our repository

**lon, lat** - geographical position, expressed in military style (EPSG:4326, as decimal fraction of degrees, positive for eastern latitude and northern longitude)
**radius** - region of interest, defined as a circle with centre in point determined by the longitude and latitude with radius expressed in meters (it won't work with point manually selected in EOFinder/Data Explorer)

**geometry** - region of interest, defined as WKT string (POINT, POLYGON, etc.)

**box** - region of interest, defined as the rectangle with given (west,south,east,north) values

### Volatile features

Some terrain-like feature masks are not permanent but describing a single scene only. The most commonly used such feature is cloudiness, or cloudCover, which is defined for most of the products coming from optical sensors. For example:

**cloudCover=[0,10]**

selects only those scenes, which are covered by clouds by no more than 10%.

Caution: to be meaningful, the cloudiness must be provided with each product, while in many products is missing. If the cloudiness is unknown for the scene, it is marked by a value of 0 or -1. cloudCover=0 is therefore ambiguous: it may either mean totally cloudless sky or the cloudy scene for which cloud cover had not been estimated during original data processing.

### Satellite features

**instrument** - meaningful only for satellites equipped with multiple instruments. The possible values are satellite specific.

**productType** - the actual types possible are specific for every satellite.

**sensorMode** - also satellite and sensor specific. E.g. (for Sentinel-1): sensorMode=EW

**orbitDirection** - ASCENDING or DESCENDING. For most heliosynchronous satellites descending orbits means the day scenes, while ascending means night ones. For many optical satellites (e.g. Sentinel-2) only day scenes are published.

**resolution** - expected spatial resolution of the product defined in meters.

**status:**

- **ONLINE**
- **OFFLINE**

Some additional parameters are strictly satellite-specific, e.g. polarisation, which is defined only for Sentinel-1

For every satellite (collection) its set of query-able parameters may be obtained by a query like:

[https://catalogue.dataspace.copernicus.eu/resto/api/collections/Sentinel1/describe.xml](https://catalogue.dataspace.copernicus.eu/resto/api/collections/Sentinel1/describe.xml)

The resulting XML file provides full list of the parameters for the collection, with their very brief descriptions.

<!-- Following lines are removed at request from Natalia Sobieska mail to BartBomans dd27.01.2023 16:43
## Order API Documentation


For comprehensive Order API description you can visit these pages:
was already in comment -- [**https://finder.creodias.eu/api/docs/**](https://finder.creodias.eu/api/docs/) --

[**EO Data Ordering API2 Manual**](https://creodias.eu/-/comletiondatestartdate-and-in-finder-api-v2?) -->