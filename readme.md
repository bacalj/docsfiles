# CLUE Document Utilities

This is a jupyter notebook intended to be a composable set of utilities for working with data derived from a set of CLUE documents.
Everything in here is dependent on there being a collection of .json or .txt files in the `documents` directory.
The current best place to get these is by running the `download-documents.ts` script in the `clue` repository, and moving the resulting files into the `documents` directory.

## Setup

Create three directories in the root of this repository.  The scripts here will use those directories by name. They are listed in the `.gitignore`.

```bash
mkdir documents
mkdir outputs
mkdir pickles
```

- The `documents` directory: where you put the json documents you want to work with.
- The `outputs` directory: where any output will go.
- The `pickles` directory is where the scripts will put and access pickled data that gets read into memory for some scripts.

## Document Server

It makes sense to serve documents locally while working on these scripts.  That way you can refer to rendered and json representations of individual documents.  Start by clicking this link and running the cell there: [Serve Documents](serve_docs.ipynb)

## Text Chunk Getter

Extract human readable text chunks from all the text tiles in all the documents in the `documents` directory: [Get Text Chunks](get_text_chunks.ipynb)

## Document Stastistics

Get some basic statistics about the documents in the `documents` directory: [Document Statistics](document_statistics.ipynb)

## Deployment ?

At the moment, this is designed to be run locally, but here is some code that will be useful if we want to run this in a Workbench AI (Google Cloud) notebook:
The following is the boilerplate code needed to get documents into the project when the documents are stored in a Google Cloud bucket.

```python
import os
from google.cloud import aiplatform
from google.cloud import storage

dataset_id = "your-dataset-id"
bucket_name = "your-bucket-name"
service_key_path = "your-service-key.json"
os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = service_key_path

vertex_client = aiplatform.gapic.DatasetServiceClient(client_options={"api_endpoint": "us-central1-aiplatform.googleapis.com"})
storage_client = storage.Client()
bucket = storage_client.get_bucket(bucket_name)
dataset = vertex_client.get_dataset(name=dataset_id)

print(dataset)
```