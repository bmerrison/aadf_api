# Average Annual Daily Flow API
My API is hosted on an Amazon Elastic Beanstalk instance and can be accessed [here](http://aadf-api-live.eu-west-2.elasticbeanstalk.com). The API was created using the [Django REST framework](http://www.django-rest-framework.org/) and the database backend is Postgres, as specified.
## Object Types
The API uses the following object types, which correspond to database tables:
* A **Traffic Count** is a count that occured at a particular *Count Point* in a given year. It has associated counts for each vehicle type.
* A **Count Point** is a location at which a count occured, including the postion (easting/northing), the *Road* it occured on, the *Junction* at each end of the link, the council *Ward*, etc. Note that each *Count Point* has a reference number (matching the supplied CSV file), but in some cases there can several *Count Points* with the same reference in the database. This is because the properties of a count point can be different for different years.
* A **Ward** is a small geographical area within a larger *Loocal Authority*.
* A **Local Authority** is typically a council area. Note that I haven't used the "LocalAuthority" field from the supplied CSV, but instead use the more detailed local authorities from the supplied wards shapefile.
* A **Region** is a much larger geographical region. At the moment, the only region is the south west.
* A **Junction** is an end point of a road link. It just contains a description.
* A **Road** is a road that a count took place on. It has a name and a *Road Category*.
* A **Road Category** consists of a two-letter code describing a road type, and a text description of that type.
* An **Estimation Method** is a method by which a traffic count can be calculated. The name is a slight misnomer, as even non-estimated counts have an associated method ("Manual Count").
## Using the API
The API is has a standard RESTful interface. The available methods are documented [here](http://aadf-api-live.eu-west-2.elasticbeanstalk.com/docs/). The interface follows the [CoreAPI](http://www.coreapi.org/) standard, meaning the CoreAPI command line interface or Python/Javascript libraries can (optionally) be used to make calls. The [documentation](http://aadf-api-live.eu-west-2.elasticbeanstalk.com/docs/) can show code examples using CLI/Python/Javascript for each available method.
### Examples
List of all wards, as an HTML page:
[http://aadf-api-live.eu-west-2.elasticbeanstalk.com/wards/](http://aadf-api-live.eu-west-2.elasticbeanstalk.com/wards/)
The same, but as raw JSON:
[http://aadf-api-live.eu-west-2.elasticbeanstalk.com/wards.json](http://aadf-api-live.eu-west-2.elasticbeanstalk.com/wards.json)
Details of ward with ID 1, as an HTML page:
[http://aadf-api-live.eu-west-2.elasticbeanstalk.com/wards/1/](http://aadf-api-live.eu-west-2.elasticbeanstalk.com/wards/1/)
The same, but as raw JSON:
[http://aadf-api-live.eu-west-2.elasticbeanstalk.com/ward/1.json](http://aadf-api-live.eu-west-2.elasticbeanstalk.com/wards/1.json)
For **Count Points** and **Traffic Counts**, optional parameters can be passed (see docs for a full list) to filter the results. These API endpoints also return partially de-normalized data for convenience, to save having to make multiple API calls to get all details.
Find all traffic counts that occurred in the "Halberton" ward in 2010:
[http://aadf-api-live.eu-west-2.elasticbeanstalk.com/traffic_counts/?ward_name=Halberton&year=2010](http://aadf-api-live.eu-west-2.elasticbeanstalk.com/traffic_counts/?ward_name=Halberton&year=2010)
Find all count points within a rectangle given by easting/northing co-ordinates (useful for showing them as pins on a map!), in raw JSON format:
[http://aadf-api-live.eu-west-2.elasticbeanstalk.com/count_points.json?min_easting=239468&min_northing=051064&max_easting=260404&max_northing=066148](http://aadf-api-live.eu-west-2.elasticbeanstalk.com/count_points.json?min_easting=239468&min_northing=051064&max_easting=260404&max_northing=066148)
## Notes
* The script `importer/importer.py` uses the API to populate the database with a list of counts from a CSV file and ward geometry files. Note that this script assumes the database is empty. A more sophisticated script would be able to add additional data to an existing database.
* The optional ability to query by ward is implemented. The import script uses the pyshp and Shapely libraries, as well as an algorithm from Hannah Fry (www.hannahfry.co.uk) for converting E/N to Lat/Lon, to figure out the Ward for each Count Point and store this in the database. A more elegant solution would probably be to use a Postgres extension that allows GIS data to be stored in the database, but it's not clear how easily that would work with Django and I didn't have time to try!
* The database is fully normalized, but for convenience the API de-normalizes to an extent. For example, the `CountPoint` table has a foreign key to the `Road` table, where the name and category of each road is stored. To save having to do two API calls (i.e. to GET a `CountPoint` and then the corresponding `Road`) the response when you GET a `CountPoint` includes the name and category of the road.
* It's possible (but I'm not sure) that the data could be cleaned up in a few places (e.g. road categories changing from year to year). To safe I haven't modified the data from the spreadsheet in any way, with the sole exception of rounding link lengths to one decimal place.
* There are some automated tests (based on pytest) defined in the "tests" directory, although since this isn't a production program these are more to prove the point and are very incomplete! Nevertheless, continuous integration is performed using [Travis](www.travis-ci.org) every time changes are pushed to Github. 