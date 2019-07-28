# Setup Instructions

**Important**: Make sure to run these commands at the base of this directory (same folder where Scripts and Data folders are located).

1. Import docker images into registry

`docker load -i neo4j-custom-docker.tar`

`docker load -i spark-custom-docker.tar`

2. Run the Docker Compose file. This will instantiate both the Spark and Neo4j containers, and create a network allowing these containers to communicate.

```
docker compose-up
```

This command will not only start the containers, but will instantiate jupyter notebook. It also binds the notebooks directory volume to our `Scripts` folder, allowing us to view and run these notebooks from outside the container. It also allows us to modify the notebooks without needing to update the image.

3. The console should output a series of info since we bound the spark container output to it using the `stdin_open: true` parameter. Included should be a URL to access the Jupyter notebook through the browser, which would look similar to:

`http://127.0.0.1:8888/?token=7d625919777fbe19e11d155b9d9e8fdfcf3c6ee2c274d76b`

4. Once the containers are running, browse to `http://localhost:7474` to view Neo4j console (this may take a minute or 2 to load). The database will currently be empty. The credentials are:
Username: **Neo4j** and password: **test**.

5. Open up the Jupyter URL from point (3) and run the first script titled `Spark Master Script`. This will start a spark instance, retrieve input data located in the `Data/input` directory in this folder, process the data, and output the results to the `Data/results` folder in the same directory.

6. The second script, `Load to Neo4j`, will then retrieve the resulting CSV files from the process above and insert them into Neo4j. Once this script has finished running, head back over to Neo4j console to view the resulting graph.

7. Remove all containers and network

```
docker-compose down
```




----

# Image Information

### Image name: spark-custom

The spark custom image begins as a dockerfile, which runs a base image of debian and on top of it installs java and python, as well as hadoop and spark libraries. It also sets the necessary configurations and environmental variables to run spark. We then installed a number of necessary python libraries, including:
- PySpark
- Py2Neo
- Azure Storage SDK
- Jupyter Notebook

We also included two iPython notebooks with the two necessary master scripts to run our system. These scripts perform the following actions:
1. MasterScript.ipynb: Loads text files from azure blob storage into spark and begins processing them. This involves deserializing, creating the necessary relationships, and storing as CSV files. This also includes the process of extracting all words from the abstracts, generating TFIDF scores for each word, and selecting the top 5 keywords for each paper.
2. Load_To_Neo4j_Master.ipynb: Retrieves the new CSV files from blob storage, and sends them along with the respective queries to Neo4j. Neo4j will then create nodes and relationships based on these CSV files.

### Image name: neo4j-custom

This image is based off of the default neo4j docker image. A plugin to run graph algorithms has also been installed on the custom image. The configuration settings are passed in the docker run command, and although they are left as default settings here, in our live version they were increased significantly. 

We also bind the the neo4j data and logs volumes onto the host machine. We bind the current working directory's data folder to the Neo4j import folder, as by default Neo4j will only allow access to CSV files within that folder, for security reasons. Therefore the resulting CSV data folders need to be placed there before importing into the system.

