---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "Optimising a route round the Christmas lights using the Travelling Salesman Problem"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2020-12-06T20:46:49Z
lastmod: 2020-12-06T20:46:49Z
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: "Smart"
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
My local village has set up a lovely Christmas trail this year, with many houses going overboard on the flashing lights and decorations being listed on a charity-produced map. It's a delightful idea, but as with all villages that have grown organically from [their Doomsday roots](https://opendomesday.org/place/SK6130/keyworth/), planning the optimal route round all the twisting roads and cut-throughs is a non-trivial problem.

So obviously it needed solving.

At its core, this is the [travelling salesman problem](https://en.wikipedia.org/wiki/Travelling_salesman_problem). Finding the quickest route (or shortest, assuming that walking pace is pretty much constant despite the hills) between all the points, while visiting each display once and only once. Luckily Google's [OR-Tools](https://developers.google.com/optimization) can [solve this](https://developers.google.com/optimization/routing) in a fairly efficient way in Python (in a number of ways, in fact), and the [Google Maps API](https://developers.google.com/maps/documentation) provides a handy hook into pulling distances between addresses which can feed into this.

First step is to create a data model to store a matrix of distances, including a list of Google Maps readable addresses:

```python
def create_data_model():
    """Stores the data for the problem."""
    data = {}
    data['API_key'] = 'YOUR_KEY_GOES_HERE' # Needs GCloud API key
    data['addresses'] = ['list+of+addresses', # Google maps readable
                        ...
                        ]
    data['distance_matrix'] = [] # This will get filled
    data['num_vehicles'] = 1 # Only one of you
    data['depot'] = 0 # Start point
    return data
  ```

Next up [use Google Maps to build a matrix of distances](https://developers.google.com/optimization/routing/vrp#distance_matrix_api) between the addresses in ```data['addresses']``` and store it in ```data['distance_matrix']```:

```python
def create_distance_matrix(data):
# Generate the correct sized distance matrix ready to be filled
  addresses = data["addresses"]
  API_key = data["API_key"]
  max_elements = 100                        
  # Distance Matrix API only accepts 100 elements per request
  # so need to get rows in multiple requests
  num_addresses = len(addresses) 
  # Maximum number of rows that can be computed per request
  max_rows = max_elements // num_addresses  
  q, r = divmod(num_addresses, max_rows)    # num_addresses = q * max_rows + r
  dest_addresses = addresses
  distance_matrix = []
  
  for i in range(q): # Send q requests, returning max_rows rows per request.
    origin_addresses = addresses[i * max_rows: (i + 1) * max_rows]
    response = send_request(origin_addresses, dest_addresses, API_key)
    distance_matrix += build_distance_matrix(response)
    
  if r > 0:  # Get the remaining remaining r rows, if necessary.
    origin_addresses = addresses[q * max_rows: q * max_rows + r]
    response = send_request(origin_addresses, dest_addresses, API_key)
    distance_matrix += build_distance_matrix(response)
  return distance_matrix


def send_request(origin_addresses, dest_addresses, API_key):
# Send requests to Google Maps API

  def build_address_str(addresses):
  # Build a pipe-separated string of addresses
    address_str = ''
    for i in range(len(addresses) - 1):
      address_str += addresses[i] + '|'
    address_str += addresses[-1]
    return address_str
    
  request = 'https://maps.googleapis.com/maps/api/distancematrix/json?units=imperial'
  origin_address_str = build_address_str(origin_addresses)
  dest_address_str = build_address_str(dest_addresses)
  request = request + '&origins=' + origin_address_str + '&destinations=' + \
                       dest_address_str + '&key=' + API_key
  jsonResult = urllib.request.urlopen(request).read()
  response = json.loads(jsonResult)
  return response
  
  
def build_distance_matrix(response):
# Build a n*n distance matrix from the responses
  distance_matrix = []
  for row in response['rows']:
    row_list = [row['elements'][j]['distance']['value'] for j in range(len(row['elements']))]
    distance_matrix.append(row_list)
  return distance_matrix


data = create_data_model()
addresses = data['addresses']
API_key = data['API_key']
data['distance_matrix'] = create_distance_matrix(data)
```

Plug that into a routing model, set a distance callback, cost of travel, and search parameters for the solution, and boom:

```python
# Build a routing model
manager = pywrapcp.RoutingIndexManager(len(data['distance_matrix']),
                                       data['num_vehicles'], data['depot'])
routing = pywrapcp.RoutingModel(manager)


# Set a distance callback
def distance_callback(from_index, to_index):
    """Returns the distance between the two nodes."""
    # Convert from routing variable Index to distance matrix NodeIndex.
    from_node = manager.IndexToNode(from_index)
    to_node = manager.IndexToNode(to_index)
    return data['distance_matrix'][from_node][to_node]
transit_callback_index = routing.RegisterTransitCallback(distance_callback)


# Set the cost (simple - travel distance)
routing.SetArcCostEvaluatorOfAllVehicles(transit_callback_index)


# Set search parameters
search_parameters = pywrapcp.DefaultRoutingSearchParameters()
search_parameters.first_solution_strategy = (
    routing_enums_pb2.FirstSolutionStrategy.PATH_CHEAPEST_ARC)


# Solve and print the solution
solution = routing.SolveWithParameters(search_parameters)
if solution:
    print_solution(manager, routing, solution) 
    # see full code for print_solution()
```

The full code including all the functions mentioned above is [on my Github](https://github.com/nickopotamus/keyworth_xmas_lights/), and I'll see you on the optimal Christmas trail!

