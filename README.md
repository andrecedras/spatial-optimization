# spatial-optimization
This notebook illustrates the use of the Google Maps API to determine the optimum route given a list of addresses

# Task: Find the best route for a vehicle to travel between the following locations:

1. 115 St Andrew’s Drive, Durban North, KwaZulu-Natal, South Africa
2. 67 Boshoff Street, Pietermaritzburg, KwaZulu-Natal, South Africa
3. 4 Paul Avenue, Fairview, Empangeni, KwaZulu-Natal, South Africa
4. 166 Kerk Street, Vryheid, KwaZulu-Natal, South Africa
5. 9 Margaret Street, Ixopo, KwaZulu-Natal, South Africa
6. 16 Poort Road, Ladysmith, KwaZulu-Natal, South Africa

## Steps to follow

### Obtain an API key
* If you do not already have one, please obtain an API key by following the instructions at:

[Google Maps home page](https://cloud.google.com/maps-platform/)


 ### Enable the various APIs on your console

    apikey = '################################' # Please enter your own

### Geocoding

Geocoding is the process of converting addresses (like "1600 Amphitheatre Parkway, Mountain View, CA") into 
geographic coordinates (like latitude 37.423021 and longitude -122.083739), which you can use to place markers on a map, 
or position the map.

Please see: [Google Maps Geocoding intro](https://developers.google.com/maps/documentation/geocoding/intro) for more details

    import pandas as pd # pandas - a powerful data analysis and manipulation library for Python
    import requests # Requests is an HTTP library, written in Python, for human beings.

At present, we have been given a short list of addresses, so we will keep things nice and simple.  If more addresses were given in a file or other format, we will have to import it accordingly.

     address_list = ["115 St Andrew’s Drive, Durban North, KwaZulu-Natal, South Africa",
                    "67 Boshoff Street, Pietermaritzburg, KwaZulu-Natal, South Africa",
                    "4 Paul Avenue, Fairview, Empangeni, KwaZulu-Natal, South Africa",
                    "166 Kerk Street, Vryheid, KwaZulu-Natal, South Africa",
                    "9 Margaret Street, Ixopo, KwaZulu-Natal, South Africa",
                    "16 Poort Road, Ladysmith, KwaZulu-Natal, South Africa"]
               
#### Geocoding API Request Format
A Geocoding API request takes the following form:

[Google Maps Geocoding Formatting](https://maps.googleapis.com/maps/api/geocode/outputFormat?parameters)
where outputFormat may be either of the following values:

* json (recommended) indicates output in JavaScript Object Notation (JSON); or
* xml indicates output in XML

    results = []
    locations_latitude = []
    locations_longitude = []
    formatted_address = []
    for location_string in address_list:
        r = requests.get('https://maps.googleapis.com/maps/api/geocode/json?address="%s"&key=%s'%
                         (location_string, apikey))
        result = r.json()['results']
        location = result[0]['geometry']['location']
        locations_longitude.append(location['lng'])
        locations_latitude.append(location['lat'])
        formatted_address.append(result[0]['formatted_address'])
        results.append(result)

* The code in this cell is based roughly on what is found in the geocoder documentation found at 
[Google Maps Geocoder API](https://geocoder.readthedocs.io/api.html)

    df = pd.DataFrame({'address':address_list,'latitude':locations_latitude,'longitude':locations_longitude,
                       'formatted_address':formatted_address})

## Directions

[Google Maps Directions API docs](https://developers.google.com/maps/documentation/directions/start)

The Directions API is a service that calculates directions between locations. You can search for directions for several modes of transportation, including transit, driving, walking, or cycling.

    origin=(-28.5588,29.77523)
    destination = (-29.595413,30.3799223)
    waypoints = [(-29.778758,31.043515),(-28.757862,31.902001),(-27.769209,30.79068899999999),(-30.154131,30.058675)]

    import gmaps
    from datetime import datetime
    now = datetime.now()

    #configure api
    gmaps.configure(api_key=apikey)

    #Create the map
    fig = gmaps.figure()
    #create the layer
    layer = gmaps.directions.Directions(origin, destination,waypoints = waypoints,optimize_waypoints=True,
                                        mode='car',api_key=apikey,departure_time = now)
    #Add the layer
    fig.add_layer(layer)
    fig

![Optimal route](spatial-optimization/spatial-optimization.png "Optimal route as per Google maps for our list of addresses")

    origin_dir='-28.5588,29.77523'
    destination_dir = '-29.595413,30.3799223'
    waypoints_dir = ['-29.778758,31.043515|-28.757862,31.902001|-27.769209,30.79068899999999|-30.154131,30.058675']

    from datetime import datetime
    now = datetime.now()
    import googlemaps

    #### Setting u the API key to connect to Google maps API

    #Perform request to use the Google Maps API web service
    gmaps = googlemaps.Client(key=apikey)

    for i in waypoints_dir:
         directions = gmaps.directions(origin = origin_dir,waypoints = i,destination = destination_dir,
                                      mode='driving',optimize_waypoints=True,departure_time = now)

    start_address = []
    end_address = []
    distance = []
    journey_time = []

    for i in range(0, (len(df)-1)):
        distance.append(directions[0]['legs'][i]['distance']['text'])
        journey_time.append(directions[0]['legs'][i]['duration']['text'])
        start_address.append(directions[0]['legs'][i]['start_address'])
        end_address.append(directions[0]['legs'][i]['end_address'])

     df_distance = pd.DataFrame({'start_address':start_address,'end_address':end_address,
                                'distance':distance,'journey_time':journey_time},
                                columns = ['start_address','end_address','distance','journey_time'])
                           
     ### Total distance travelled

     total_distance = 0
     for i in range(0, len(df)-1):
         total_distance += float(df_distance['distance'][i][:-3])
    
     print('Total distance travelled = {} km'.format(total_distance))

     ### Find journey time

     import numpy as np

     total_journey_time_hrs = 0
     total_journey_time_mins = 0
     for i in range(0, len(df)-1):
         total_journey_time_hrs += np.int(df_distance['journey_time'][i][0])
    
    total_journey_time_hrs = total_journey_time_hrs + total_journey_time_mins//60

    total_journey_time_mins = total_journey_time_mins%60

    print('The total journey time is: {} hours and {} minutes'.format(total_journey_time_hrs,
                                                                 total_journey_time_mins))

## Articles consulted for hints

[Using gmplot package](https://www.geeksforgeeks.org/python-plotting-google-map-using-gmplot-package/)
[How to use the Google Maps Distance-Matrix API](https://medium.com/how-to-use-google-distance-matrix-api-in-python/how-to-use-google-distance-matrix-api-in-python-ef9cd895303c)
[How to plot route using Python and Google Maps API](https://blog.alookanalytics.com/2017/02/05/how-to-plot-your-own-bikejogging-route-using-python-and-google-maps-api/)
[Google Maps in Python](https://blog.goodaudience.com/google-maps-in-python-part-2-393f96196eaf)
