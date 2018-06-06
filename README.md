# Insight DevOps Engineering Systems Puzzle Solution

## Table of Contents
1. [Puzzle description](README.md#understanding-the-puzzle)
2. [Solution approach](README.md#solution-approach)
3. [Repo directory structure](README.md#repo-directory-structure)

## Puzzle description
https://github.com/InsightDataScience/systems-puzzle

## Solution approach
### Set up and understanding
1. Initially I had to set up the correct docker-engine and docker-compose in order to be able to use version 3 in `docker-compose.yml`.
2. I took a moment to read through the code and understand what is actually going on:
    * The db container sets up a postgres database image. Based on `database.py` it appears the database is hosted on port `5432`.
    * The flaskapp container is building/starting up the flask app server.
    * The nginx container is the image for the web server in `flaskapp.conf` where the HOST port is at `80` and is connecting to a container port at `8080`.
    * The python scripts (`app.py`, `forms.py`, `models.py`) are setting up the skeleton and functioning of the web page. 

### Testing and debugging
#### nginx server fix 
1. Upon running the docker commands, an error is thrown `listen tcp 0.0.0.0:80: listen: address already in use`. It seems the issue is with the nginx container.
2. Taking a look at `flaskapp.conf`, the server is trying to listen at port `80` but failing there.
3. This is due to the fact that in our container we specify port 80 as our HOST port thereby already occupying it.
4. I determine that the HOST port for the container should be `8080` because localhost will later connect to `8080` to pull up the web page.
    Port `8080` should connect to port `80`; the port the web server will be listening on.
5. Changed the line `80:8080` in `docker-compose.yml` to `8080:80`.
#### Bad gateway fix
1. The nginx container runs fine, and upon trying to open the web page a `Bad Gateway Error` is thrown.
2. I was using the docker plugins in PyCharm to debug the code and I saw in the log files for the flaskapp container it was `Running on http://0.0.0.0:5000/`
3. However the proxy_pass in the sever is trying to connect flaskapp to port `5001`. That's a bug! :D Change that to `5000` in `flaskapp.conf`
4. The only other place in the code `5001` was specified is in exposing that port in the `Dockerfile`. So I changed that as well.
#### Web page fix 
1. Web page opens up just fine! Testing it reveals that the success page has a bug in it. It does not show the items that have been input.
2. Since the success page had the bug it was quite clear that the bug was in the success function in `app.py`
3. For a simple solution I had the page return a dictionary where the keys are the items and the value is the quantity
4. More features can be added to success page, however no specs were provided on how the developer wanted in the success page.

## Repo directory structure

    .
    ├── app.py 
    ├── database.py
    ├── docker-compose.yml
    ├── Dockerfile
    ├── env_file
    ├── forms.py
    ├── models.py
    ├── requirements.txt
    ├── README.md
    ├── conf.d
        └── flaskapp.conf
    ├── templates
        └── index.html
