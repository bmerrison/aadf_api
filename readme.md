# Average Annual Daily Flow API
## Notes
* The script `importer/importer.py` uses the API to populate the database with a list of counts from a CSV file and ward geometry files. Note that this script assumes the database is empty. A more sophisticated script would be able to add additional data to an existing database.
* The optional ability to query by ward is implemented. The import script uses the pyshp and Shapely libraries, as well as an algorithm from Hannah Fry (www.hannahfry.co.uk) for converting E/N to Lat/Lon, to figure out the Ward for each Count Point and store this in the database. A more elegant solution would probably be to use a Postgres extension that allows GIS data to be stored in the database, but it's not clear how easily that would work with Django and I didn't have time to try!
* The database is fully normalized, but for convenience the API de-normalizes to an extent. For example, the `CountPoint` table has a foreign key to the `Road` table, where the name and category of each road is stored. To save having to do two API calls (i.e. to GET a `CountPoint` and then the corresponding `Road`) the response when you GET a `CountPoint` includes the name and category of the road.
* It's possible (but I'm not sure) that the data could be cleaned up in a few places (e.g. road categories changing from year to year). To safe I haven't modified the data from the spreadsheet in any way, with the sole exception of rounding link lengths to one decimal place.
* There are some automated tests (based on pytest) defined in the "tests" directory, although since this isn't a production program these are more to prove the point and are very incomplete! Nevertheless, continuous integration is performed using Travis (www.travis-ci.org) every time changes are pushed to Github. 