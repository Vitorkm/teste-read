# Pipe-mesh

This is the repository corresponding to the container that runs the "mesh" portion of the pipeline.

## Installation

Use the package manager [pip](https://pip.pypa.io/en/stable/) to install [all modules](https://github.com/Khanto-Tecnologia/Pipe-mesh/blob/main/requirements.txt)

```bash
pip install requirements.txt
```

## How does it work?
The mesh is composed of **4** different scripts, we'll go through all of them now:

The **first thing** you should know is that, in the actual version, running all the scripts **it's a manual role**. You're supposed to start pureMesh, after finished, start the meshAcolyte... until **all scripts** have been run.

### **1. pureMesh**

In the present moment, **the pureMesh doesn't take anything as input**. In the future, we expect that the Pipe-mesh upgrades to a global level of research. Uses the **geopy** to create a geojson, inside the geojson, more specific in a key named **'boundingbox'**, you can find the coordinates of the region chosen (in our case Brazil) and starts checking if the scrapped items are in the parameters.  For **output** you should expect a **JSON temporary** file with contains **all the coordinates of the finals squares** created by the script, you can find the JSON [here](https://s3.console.aws.amazon.com/s3/buckets/pipe-intermediary?region=us-west-2&tab=objects). It usually takes **4-5 hours** to finish this script.

### 2. meshAcolyte

It receives as **input** the **JSON file made** by pureMesh, putting all the coordinates of the squares made earlier in a messages queue in SQS to be scrapped by the laracnaMeshNational after. For **output**, you receive **a queue in SQS with messages** named as **'mesh-laracna-inputs.fifo'** and inside each message, **contains the coordinates of an individual square in the JSON**. It usually takes a **few minutes** to complete this script.

### 3. laracnaMeshNational

This scrapper takes as **input** each message from the **'mesh-laracna-inputs.fifo'** and extracts all properties **ID's and related coordinates** inside the squares formed by the geodata given in the input. This script **runs on multiple machines** at the same time which one picks **one message at a time**, executes the scrapper in the square, put the data achieved as a message in a new queue, and starts **again** with another square coordinate in a new message **until there are no more messages in queue**. For **output** you should expect a new queue in SQS named **'mesh-laracna-output.fifo'** which contains **every ID's and related coordinates** of all the squares in the previous input queue. The time to complete this script **depends** on how many machines are you working with.

### 4. listingsCompiler

