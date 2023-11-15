## CLUE Document Utilities

This is a jupyter notebook intended to be a composable set of utilities for working with data derived from a set of CLUE documents.
Everything in here is dependent on there being a collection of .json or .txt files in the `documents` directory.
The current best place to get these is by running the `download-documents.ts` script in the `clue` repository, and moving the resulting files into the `documents` directory.

For this to work you'll need to create three directories in the root of this repository:

```bash
mkdir documents
mkdir outputs
mkdir pickles
```

The scripts here will use those directories by name.
- The `documents` directory is where you should put the documents you want to work with.
- The `outputs` directory is where the scripts will put their output.
- The `pickles` directory is where the scripts will put and access pickled data that gets read into memory for some scripts.

### Document Server

Serve documents from the `documents` directory.  This is useful if you want to use the CLUE document viewer to view documents, but don't want to run the entire CLUE stack.

[Serve Documents](serve_docs.ipynb)

### Text Chunk Getter

Extract human readable text chunks from all the text tiles in all the documents in the `documents` directory.

[Get Text Chunks](get_text_chunks.ipynb)

### Document Stastistics

Get some basic statistics about the documents in the `documents` directory.

[Document Statistics](document_statistics.ipynb)

### Running this within a Google Workbench

In the future there may be a usecase in which it is easiest to access documents stored in a GC Cloud Storage bucket and run these and other scripts in the Google Workbench environment.  The following is the code needed to get documents in hand in that context.

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