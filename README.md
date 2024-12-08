# Matrix 4

### Local Installation steps

**Clone the project:**

```
git clone https://bitbucket-eng-rtp1.cisco.com/bitbucket/scm/cm/matrix4.git
cd matrix4
```

**Build matrix image:**

```
docker build --no-cache -f deployment/docker/Dockerfile -t matrix .
```

**Map the local matrix4 project to docker container:**

In `deployment/docker/docker-compose.yml` file, under matrixnis service add/update the below volumes to map the local
code to container

```
     volumes:
        - ../../app/:/matrixnis/app
```

**Setup pre-commit (create local env in host machine for linting)**

```
pip install -r deployment/docker/requirements/requirements_dev.txt
git config --global init.templateDir ~/.git-template
pre-commit init-templatedir ~/.git-template
```

**Run stack using docker-compose**

```
docker-compose -f deployment/docker/docker-compose.yml up -d
```
### START MATRIX

* `python manage.py init_matrix --dev` will run dev server
* In production omit --dev flag.

### ENABLE PLUGINS

In admin page:
`plugin` -> select `tenant plugins` -> create `tennat plugins` -> select required tenant and plugin.

** FOR SHARED APPS:
Need to expicity enable plugins for `global` tenant first.

### ENABLE SSO

* Enable SSO for`global` tenant.
* Enable SSO for the targeted tennat.
* In`AuthMethod` Add SSO method for the tenant.
* Configure SSO details for the tenant.

### ENABLE ASYNC MIGRATIONS

* Have workers subscribed to channel `migrations`
* Use `./manage.py amigrate --celery` to migrate using celery.
* For async `./manage.py amigrate`, this will migrate using python async.

NOTE: For enabling async migrations of tenant from UI set env variable -> MIGRATE_SCHEMA_ASYNC=1

### SQL BASED MIGRATIONS

* create .sql file
* Put it in sql_migrations folder in the respective app.
NOTE: The sql migrations will be applied only after the actual migrations.

## MOCK DATA

* Run python manage.py mock_data --count 100 --schema test metrics-data

### REMOTE DEBUGGING WITH VSCODE/PYCHARM

This provides ability to debug Django code as if it is running in the IDE .

![](https://bitbucket-eng-rtp1.cisco.com/bitbucket/rest/api/1.0/projects/CM/repos/matrix4/attachments/3368)

If you wanted to debug remote code or code running in a docker container, on the remote machine or container, follow the below steps

```
python -m debugpy --listen 0.0.0.0:5678 app/manage.py runserver 0.0.0.0:8080
```

The associated configuration file `launch.json` would then look as follows.

```
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python: Attach",
            "type": "python",
            "request": "attach",
            "connect": {
              "host": "localhost",
              "port": 5678
            },
            "pathMappings": [
              {
                "localRoot": "${workspaceFolder}",
                "remoteRoot": "."
              }
            ]
          }
    ]
}
```

Note: Be aware that when you specify a host value other than 127.0.0.1 or localhost you are opening a port to allow access from any machine, which carries security risks. You should make sure that you're taking appropriate security precautions, such as using SSH tunnels, when doing remote debugging.

for more details for VSCODE remote debugging refer [link](https://code.visualstudio.com/docs/python/debugging#_debugging-by-attaching-over-a-network-connection)

For details on Pycharm remote debugging refer [link](https://www.jetbrains.com/help/pycharm/using-docker-as-a-remote-interpreter.html#run)


### Pytest coverage Commands :
In matrix shell , Do cd matrixnis/testing
source pytest.sh directory_to_cover html:report_pytestcov
The pytest will run recursively in matrixnis/app/{directory_to_cover} wherever the file name and folder name starts with test.
The results will be available at the directory  matrixnis/app/testing/{report_pytestcov}.
example:
source pytest.sh data_aggregation html:report_pytestcov
The above command will cover all files and folders starting with test inside directory matrixnis/app/data_aggregation.The results will be saved at directory app_pytestcov in matrixnis/app/testing/ directory.

To run pytest for all apps use the command : source pytest.sh . html:report_pytestcov
