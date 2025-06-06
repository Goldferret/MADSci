# MADSci Data Manager

Handles capturing, storing, and querying data, in either JSON value or file form, created during the course of an experiment (either collected by instruments, or synthesized during anaylsis).

![MADSci Data Manager Diagram](./assets/data_manager.drawio.svg)

## Notable Features

- Collects and stores data generated in the course of an experiment as "datapoints"
- Current datapoint types supported:
  - Values, as JSON-serializable data
  - Files, stored as-is
- Datapoints include metadata such as ownership info and date-timestamps
- Datapoints are queryable and searchable based on both value and metadata

## Installation

The MADSci Data Manager is available via [the Python Package Index](https://pypi.org/project/madsci.data_manager/), and can be installed via:

```bash
pip install madsci.data_manager
```

This python package is also included as part of the [madsci Docker image](https://github.com/orgs/AD-SDL/packages/container/package/madsci). You can see an example docker image in [this example compose file](./data_manager.compose.yaml).

Note that you will also need a MongoDB database (included in the example compose file)

## Usage

### Manager

To create and run a new MADSci Data Manager, do the following in your MADSci lab directory:

- If you're not using docker compose, provision and configure a MongoDB instance.
- If you're using docker compose, define your data manager and mongodb services based on the [example compose file](./data_manager.compose.yaml).

```bash
# Create a Data Manager Definition
madsci manager add -t data_manager
# Start the database and Data Manager Server
docker compose up
# OR
python -m madsci.data_manager.data_server
```

You should see a REST server started on the configured host and port. Navigate in your browser to the URL you configured (default: `http://localhost:8004/`) to see if it's working.

You can see up-to-date documentation on the endpoints provided by your event manager, and try them out, via the swagger page served at `http://your-data-manager-url-here/docs`.

### Client

You can use MADSci's `DataClient` (`madsci.client.data_client.DataClient`) in your python code to save, get, or query datapoints.

Here are some examples of using the `DataClient` to interact with the Data Manager:

```python
from madsci.client.data_client import DataClient
from madsci.common.types.datapoint_types import ValueDataPoint, FileDataPoint
from datetime import datetime

# Initialize the DataClient
client = DataClient(url="http://localhost:8004")

# Create a ValueDataPoint
value_datapoint = ValueDataPoint(
    label="Temperature Reading",
    value={"temperature": 23.5, "unit": "Celsius"},
    data_timestamp=datetime.now()
)

# Submit the ValueDataPoint
submitted_value_datapoint = client.submit_datapoint(value_datapoint)
print(f"Submitted ValueDataPoint: {submitted_value_datapoint}")

# Retrieve the ValueDataPoint by ID
retrieved_value_datapoint = client.get_datapoint(submitted_value_datapoint.datapoint_id)
print(f"Retrieved ValueDataPoint: {retrieved_value_datapoint}")

# Create a FileDataPoint
file_datapoint = FileDataPoint(
    label="Experiment Log",
    path="/path/to/experiment_log.txt",
    data_timestamp=datetime.now()
)

# Submit the FileDataPoint
submitted_file_datapoint = client.submit_datapoint(file_datapoint)
print(f"Submitted FileDataPoint: {submitted_file_datapoint}")

# Retrieve the FileDataPoint by ID
retrieved_file_datapoint = client.get_datapoint(submitted_file_datapoint.datapoint_id)
print(f"Retrieved FileDataPoint: {retrieved_file_datapoint}")

# Save the file from the FileDataPoint to a local path
client.save_datapoint_value(submitted_file_datapoint.datapoint_id, "/local/path/to/save/experiment_log.txt")
print("File saved successfully.")
```
