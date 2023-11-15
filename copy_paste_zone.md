# Copy/Paste Zone

Copying various snippets from cloud hosted notebooks for future use/reference

## Pickle Loader and access example

```python
import pickle

pickle_name = "all_docs_10_27.pkl"

with open(pickle_name, 'rb') as file:
    docs = pickle.load(file)
    print("loaded documents into docs list!")


print(docs[135])
```

## Key Finder

When you have a key and want to find the document it belongs to

```python
my_key = "tcFnh85YqJksuZ-A"

def find_key_location(docs_list, key_to_find, path=None):
    if path is None:
        path = []

    if isinstance(docs_list, list):
        for i, item in enumerate(docs_list):
            new_path = path + [i]
            result = find_key_location(item, key_to_find, new_path)
            if result:
                return result

    elif isinstance(docs_list, dict):
        for key, value in docs_list.items():
            new_path = path + [key]
            if key == key_to_find:
                return new_path
            result = find_key_location(value, key_to_find, new_path)
            if result:
                return result

    return None


location = find_key_location(docs, my_key)
if location:
    print(f"Key '{my_key}' found at location: {location}")
else:
    print(f"Key '{my_key}' not found in the list of dictionaries.")
```

## Get Text from a Document's Text Tiles

```python

import json


nice_doc_map = docs[1410]['tileMap']


def get_text_tiles_from_document_tile_map(tile_map):
    tile_list = []
    count = 0
    for key, tile in doc_tiles.items():
        is_text = tile.get("content", {}).get("type") == "Text"
        if (is_text):
            tile_info = {}
            count += 1
            tile_info['count'] = count
            tile_info['tile_type'] = tile.get("content", {}).get("type")
            tile_info['tile_id'] = tile.get("id", "")
            tile_info['tile_text_obj'] = tile.get("content",{}).get("text")
            tile_list.append(tile_info)
    return tile_list

def extract_text_values_list(data):
    text_values = []

    if isinstance(data, dict):
        for key, value in data.items():
            if key == "text":
                text_values.append(value)
            else:
                text_values.extend(extract_text_values(value))
    elif isinstance(data, list):
        for item in data:
            text_values.extend(extract_text_values(item))

    return text_values

def get_as_text(tile):
    if len(tile.get('tile_text_obj', {})) > 0:
        tile_text_dict = json.loads(tile.get('tile_text_obj'))
        as_text_list = extract_text_values_list(tile_text_dict)
        as_text = " ".join(as_text_list)
        return as_text
    else:
        return ""

my_tile_list = get_text_tiles_from_document_tile_map(nice_doc_map)

# pprint(my_tile_list)

# print("------\n")

for tile in my_tile_list:
    as_text = get_as_text(tile)
    print("\n", tile.get('tile_id', "\n"))
    print(as_text)
```

## Getting at documents in Google Cloud

```python

import os

# setup for access
bucket_name = "cloud-ai-platform-d76df5a1-f27c-4288-8b89-f41e345567b9"
service_key_path = "joe-try-cloud1-e9fe55ea1681.json"
os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = service_key_path

# configure
blob_start_string = "all_docs_10_27/document"
new_pickle_name = "all_docs_10_27.pkl"
run_pickling = False

import json
import pickle

from google.cloud import aiplatform
from google.cloud import storage


if run_pickling:

    # get storage client and bucket
    storage_client = storage.Client()
    bucket = storage_client.get_bucket(bucket_name)

    # get all doc blobs in a list
    doc_blobs = []
    blobs = bucket.list_blobs()
    for blob in blobs:
        if blob.name.endswith(".txt") and blob.name.startswith(blob_start_string):
            doc_blobs.append(blob)

    # ingest each one into a massive list of objects (qdocs)
    batch_size = 100
    total_docs = len(doc_blobs)
    count = 0
    qdocs = []

    for i in range(0, total_docs, batch_size):
        batch = doc_blobs[i:i + batch_size]
        batch_data = []

        for b in batch:
            strunged = b.download_as_text()
            obj = json.loads(strunged)
            batch_data.append(obj)
            count += 1

        qdocs.extend(batch_data)

        if count % batch_size == 0:
           print("Ingested", count, "documents")

    print("Ingested a total of", count, "documents")

    with open(new_pickle_name, 'wb') as file:
        pickle.dump(qdocs, file)

```