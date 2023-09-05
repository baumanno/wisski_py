# WissKI API
This project provides a minimal python library to easily interact with WissKI systems through the WissKI API.

## Installation:
For now clone this repo and install locally using pip.
```
git clone git@github.com:kaiamann/wisski_py.git
cd wisski_py
pip install .
```
After installing you should be able to import the module simply by doing:
```py
from wisski.api import Api, Pathbuilder, Entity
```

## API Initialization:
For initializing the API you have to supply it with a URL and some credentials.
If your WissKI systems API is configured to be accessible without authentication may not need any credentials, although this is not recommended.
In case you want to specify some headers you can do so by supplying them with the `headers` parameter:
```py
api_url = "https://example.wisski.url/wisski/api/v0"
auth = ("some_username", "super_secure_password")
headers = {"Cache-Control": "no-cache"}
api = Api(api_url, auth, headers)
```

## Pathbuilders
To be able to import/edit WissKI entities the API also needs to load a pathbuilder for context.
The context tells the API which fields are present in which Bundle, and thus which values have to be mapped to which path/field.

Per default the API uses **ALL** pathbuilders that are available in the system, which may lead to problems, e.g. when WissKI linkblock pathbuilders are present(?). This is not tested yet, but it is not unlikely that problems might occur.
For this purpose you can configure which pathbuilders are used by the API.
```py
# Check which pathbuilders are present in the system.
print(api.get_pathbuilder_ids()) # output e.g.: ['pathbuilder1', 'pathbuilder2', 'linkblock_pathbuilder']
# In case you only want to use pathbuilder1:
api.pathbuilders = ['pathbuilder1']
```
### Multiple Pathbuilders:
The API can also handle multiple pathbuilders, by combining several pathbuilders.
Note that the combining only happens on the client (Python) side.
This functionality allows assigning values to paths from multiple pathbuilders:
E.g. 
- `pathbuilder1` has a path that documents the name of a person.
- `pathbuilder2` has a path that documents the occupation of this person.
Assuming that the pathbuilders are configured in a way that these paths belong to the same `person` bundle we can now build a combined pathbuilder like this:
```py
# Make sure to use direct assignment!
# Altering the pathbuilder list with list functions (e.g. pop, append, etc.) won't work properly for now.
self.api.pathbuilders = ['pathbuilder1', 'pathbuilder2']
```

## Entities

### Loading Entities:
Entities can be easily loaded by:
```py
entity = api.get_entity("https://some.random.uri")
print(entity.values)
print(entity.uri)
print(entity.bundle_id)
```

### Editing Entities:
Entities can be easily edited by:
```py
entity.values["some_field_id"] = ["This value comes from Python!"]
# save to remote
api.save(entity)
```
Values are always encapsulated in arrays.


In case there are sub entities:
```py
# Get the first sub-entity
sub_entity = entity.values["sub_bundle_id"][0]
# Change the field values
sub_entity.values["sub_bundle_field_id"] = ["This value also comes from Python!"]
# save to remote
api.save(entity)
```

### Importing/Creating Entities:
To import entities from a `.csv` file you just need to supply an entity with a dict that contains the corresponding `field_id` -> `value` mapping.

Let's look at the following example pathbuilder structure to illustrate:
- **Collection Object**: `object_bundle_id`
  - Inventory number: `inventory_number_field_id`
  - Title: `title_field_id`
  - **Production**: `production_bundle_id`
    - Date: `date_field_id`

Format: 
- PATH_NAME: `BUNDLE/FIELD_ID`
- **bold** font denotes that the path belongs to a bundle.

Code for importing into this structure:
```py
# First set up the production sub-entity
production_values = {
    'date_field_id': ["11.11.1111"]
}
production = Entity(values=production_values, bundle_id="production_bundle_id")

# Set up the collection object entity
object_values = {
    'inventory_number_field_id': ["I1234"],
    'title_field_id': ["some Title", "another Title"],
    'production_bundle_id': [production]
}
collection_object = Entity(values=object_values, bundle_id="object_bundle_id")
# Save the collection object to the remote.
api.save(collection_object)
```

You can also import it from a flat data structure like this:
```py
values = {
    'date_field_id': ["11.11.1111"]
    'inventory_number_field_id': ["I1234"],
    'title_field_id': ["some Title", "another Title"],
}
collection_object = api.build_entity('object_bundle_id','object_bundle_id',  values)
api.save(collection_object)
```

Its also possible to directly import csv files:
```py
responses = api.import_csv("production_bundle_id", "test_ent.csv")
```
