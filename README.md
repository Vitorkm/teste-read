# Pipe-mesh

This is the repository corresponding to the container that runs the "mesh" portion of the pipeline.

## Installation

Use the package manager [pip](https://pip.pypa.io/en/stable/) to install [all modules](https://github.com/Khanto-Tecnologia/Pipe-mesh/blob/main/requirements.txt)

```bash
pip install requirements.txt
```

## How does it work?
The mesh is composed of **4** different scrappers, we'll go through all of them now:

**1. pureMesh**

First thing 
```python
def handle():

    print('Entered handle')
    geolocator = Nominatim(user_agent="saldsjasnfasknd")
    location = geolocator.geocode(
        "Brazil", geometry='geojson')

    geojson = location.raw['geojson']

    boundingbox = location.raw['boundingbox']

    location = Mesh(boundingbox=boundingbox, geojson=geojson)

    location.mesh(16, 16)
    
    listOfPolygons = location.mesh_elements

    listOfPolygons_copy = listOfPolygons.copy()


    controller = True
    lost_polygons = controller_loop = 0
    while controller:
        controller_loop+=1
        
        for pol in listOfPolygons:

            if pol.listingsNumber < 300:
                continue

            mapResponse = map_request(pol)

            if len(mapResponse) == 1:
                print('len 1:',mapResponse)
                pass

            if mapResponse == "ERROR":
                lost_polygons+=1
                print('ERROR IN MAP RESPONSE')
                listOfPolygons_copy.remove(pol)
                print('lost_polygons = ',lost_polygons)

            else:
                listOfPolygons_copy.remove(pol)
                listOfPolygons_copy.extend(mapResponse)

        
        print('RAM: ',psutil.virtual_memory().percent, ' CPU: ', psutil.cpu_percent(),
         ' list_len: ', len(listOfPolygons_copy), ' loop: ', controller_loop)

        controller = False in [
            mesh_element.listingsNumber < 300 for mesh_element in listOfPolygons_copy]


        listOfPolygons = listOfPolygons_copy.copy()

        

        print('Passou a iteracao')

    bulk_list = []
    exception_count = 0
    print('Started bulk_list.append() for')
    for mesh in listOfPolygons:

        try:
            current_mesh = {}
            current_mesh['created_at'] = str(utc_datetime())
            current_mesh['mesh'] = json.dumps(mapping(mesh.polygon)['coordinates'][0])
            current_mesh['numberOfListings'] = str(mesh.listingsNumber)

            bulk_list.append(current_mesh)
        except:
            exception_count += 1
            print('Exception on json segment, exception number: ', exception_count)
            traceback.print_exc()

    print('Uploading to s3')
    s3_json_updload(bulk_list, 'pipe-intermediary', 'meshElements.json')

print('Started mesh')
handle()
```
