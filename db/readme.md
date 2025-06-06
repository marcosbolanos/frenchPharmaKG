# FrenchPharmaKG 0.1

The aim of this project is to build a comprehensive knowledge graph of clinical pharmacology information, in particular with regards to the health products, guidelines and regulations specific to France. This could enable the development of agentic tools and workflows that enhance patient care.

An image of this database is available on [Dockerhub](https://hub.docker.com/r/marcosbolanos/frenchpharmakg), but you can also build your own from this repo.

## GitHub LFS

**IMPORTANT : to clone this repo, make sure to have git-lfs installed on your system**

```#
sudo apt install git-lfs
git lfs install
```

You can then `git clone` the repo as usual. If you cloned the file without LFS installed, you can fix it by running `git lfs pull`.

## Database Structure

The CSV folder contains the actual database, which is written in a format designed for Apache AGE. 

- Each file corresponds to a diffrent node or edge type (called "label")
- Node files contain unique IDs as well as a list of properties for each node
- Edge files contain the IDs and types for starting and end nodes, as well as edge properties.

## Running the database

### Using Docker

Make sure you've set your `$OPENAI_API_KEY`, `$PGUSER` and `$PGPASSWORD` environment variables.

With docker installed on your system, you may run the database simply building this image from the dockerfile : 

```
# Replace 5431 by the port you want
PORT=5431

# Create a volume for persistent storage
docker volume create fpkg_volume

# Build the image from the Dockerfile
docker build -t frenchpharmakg .

# Run container with networking and persistence
docker run \
  --name frenchpharmakg -d \
  -p "$PORT":5432 \
  -e OPENAI_API_KEY \
  -e POSTGRES_USER=$PGUSER \
  -e POSTGRES_PASSWORD=$PGPASSWORD \
  -v fpkg_volume:/var/lib/postgresql/data:Z \
  frenchpharmakg

# Quickly load the knowledge graph from the CSVs
docker exec frenchpharmakg python3 csv_loader.py
```

### Manual installation

The `csv-loader.py` file contains code that loads the files into an Apache AGE graph. The extension must be installed inside of Postgresql 16, by following [these instructions](https://age.apache.org/getstarted/quickstart). You then create a database named `fpkg0_1` : 

`psql`

`CREATE DATABASE fpkg0_1`

The extension has to be loaded once inside the database :

`psql fpkg0_1`

`CREATE EXTENSION age`

Afterwards, the database is ready to be used. You may set your Postgres login credentials in the code, or as environment variables PGUSER and PGPASSWORD.

Optionally, create a viritual environment :

`python -m venv .venv`

`source .venv/bin/activate`

Install the requirements, load the cs_loader and enjoy !

`pip install -r requirements.txt`

`python csv_loader.py`
