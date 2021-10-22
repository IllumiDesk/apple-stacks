[![Github actions badge](https://github.com/illumidesk/apple-stacks/actions/workflows/docker.yaml/badge.svg)](https://github.com/illumidesk/apple-stacks/actions/workflows/docker.yaml "Docker images build status")

# IllumiDesk Apple Stacks

Dockerfiles and related assets for IllumiDesk's workspace images for the `Apple`.

## Pre Requisits

- [Docker](https://docs.docker.com/get-docker/)

## Quickstart

1. Install dependencies

```bash
make venv
```

2. Build images

```bash
make build-all
```

3. Run:

Running the image standalone is helpful for testing:

```bash
docker run -p 8888:8888 illumidesk/apple-notebook:latest
```

Then, navigate to `http://localhost:8888` to access your Jupyter Notebook server.

> Refer to [docker's documentation](https://docs.docker.com/engine/reference/run/) for additional `docker run ...` options.

4. Test:

```bash
make test
```

## Image catalogue

| Image | DockerHub Link |
| --- | --- |
| illumidesk/apple-notebook | [![Docker Image](https://img.shields.io/docker/automated/illumidesk/apple-notebook)](https://hub.docker.com/repository/docker/illumidesk/apple-notebook?label=apple-notebook) |
| illumidesk/apple-grader | [![Docker Image](https://img.shields.io/docker/automated/illumidesk/apple-grader)](https://hub.docker.com/repository/docker/illumidesk/apple-grader?label=apple-grader) |

## Build Mechanism

1. Build and tag the base image or all images at once. Use the `TAG` argument to add your custom tag. The `TAG` argument defaults to `latest` if not specified.

Build all images:

```bash
make build-all
```

Build one image with custom tag:

```bash
make build/apple-notebook TAG=mytag
```

2. (Optional) Use the base image from step 1 above as a base image for an image compatible with the IllumiDesk stack.

```
FROM jupyter/scipy-notebook:python-3.9.5

RUN ... do stuff

# make sure you run fix-permissions after doing stuff
USER root
RUN fix-permissions "${HOME} \
 && fix-permissions "${CONDA_DIR}

USER "${NB_USER}"
```

3. (Optional) Push images to DockerHub

If you would like to use an organization other than IllumiDesk's, then step requires creating an Organization account in DockerHub or other docker image compatible registry. The `docker push ...` command will push the image to the DockerHub registry by default. Please refer to the official Docker documentation if you would like to push another registry.

For example:

```bash
docker push illumidesk/apple-notebook:python-3.9.5
```

## Development and Testing

### Build and Test

1. Create your virtual environment and install dev-requirements:

```bash
make venv
```

2. Check Dockerfiles with linter:

```base
make lint-all
```

3. Build the images and run tests:

```base
make build-all
make test
```

### Update Dependencies

There are two ways to add additional dependencies:

1. Update the `spec-list.txt` file with conda/pip dependencies

Similar to Python's `requirements.txt`, the `spec-list.txt` locks a dependency tree. The difference is that the `spec-list.txt` [combines both conda and pip dependencies](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#building-identical-conda-environments).

After you have updated the dependencies in your docker image update the `spec-list.txt` file with the following the steps below:

```bash
docker run -it --rm -v $(pwd):/tmp illumidesk/apple-notebook:<your-tag> /bin/bash

... steps to install conda/pip packages ...

conda list --explicit > /tmp/apple-notebook/spec-list.txt
```

> **Note**: replace `<your-tag>` with the tag you used to build the image. The default value is located in the `Makefile` if using the `make` command or the `FROM` directive in the Dockerfile if using the native `docker build ...` command.

2. Update the Dockerfile(s) with additional dependencies

3. Build the docker image with custom arguments (ARGS):

Use the `--build-args` flag with the `docker build` command to override the arguments included in the Dockerfile. For example, if you would like to build/test an image with a different version of Apache Spark (let's assume version `1.2.3`) then build the image like so:

```bash
docker build --build-args spark_version=1.2.3 illumidesk/apple-notebook:mytag apple-notebook/.
```

Or

```bash
make DARGS="--build-args spark_version=1.2.3" build/apple-notebook
```

## Releasing

### Overview

There are two environments configured with this repo:

- **Development**: associated to the `main` branch
- **Production**: associated to the `production` branch

Development and production versions are built with the DockerHub build actions located in the `illumidesk/apple-grader` and the `illumidesk/apple-notebook` for the grader and end-user notebooks, respectively.

### Create a Pull Request

The steps below are applicable to both the `main` and `production` branches.

1.  Fork this repository with GitHub's `Fork` option.
2.  Clone the repository to your local machine using:

```
$ git clone `https://github.com/<github-account>/apple-stacks.git`
```

> **NOTE**: Replace the `<apple-account>` placeholder above with your GitHub username.

3. Add the remote upstream branch:

```bash
git remote add upstream https://github.com/illumidesk/apple-stacks.git
```

4. Verify that your git remote settings are set up correctly:

```bash
git remote -v
```
The command above should return an output similar to:

```bash
origin  https://github.com/foobar/apple-stacks (fetch)
origin  https://github.com/foobar/apple-stacks (push)
upstream        https://github.com/illumidesk/apple-stacks (fetch)
upstream        https://github.com/illumidesk/apple-stacks (push)
```

5.  Install the dependencies required for the project by running:

```
$ make dev
$ source venv/bin/activate
```

or

```bash
$ virtualenv -p python3 venv
$ python3 -m pip install -r dev-requirements.txt
$ source venv/bin/activate
```

6.  Create a new branch for your feature or fix using:

```
$ git checkout -b branch-name-here
```

7.  Make the appropriate changes for the issue you are trying to address. Build and test the image(s) as instructed in the [Build and Test](#build-and-test) section above. Please note that these are very basic validations. It is recommended to run tests with actual Jupyter Notebook (*.ipynb) files.

8.  Add and commit the changed files:

```bash
git add .
git commit -m "my commit message here"
```

9. Push the changes to your fork:

```
$ git push origin <branch-name-here>
```

11. Submit a pull request to the upstream repository.
12. Set the description of the pull request. We recommend using declaritive terms, such as `Updates` instead of `Updated` or `Updating`.
14. Wait for the pull request to be reviewed by a maintainer.
15. Make changes to the pull request if the reviewing maintainer recommends them.
16. Celebrate your success after your pull request is merged! :tada:

### Squash and Merge for Release

1. Once the Pull Request is approved, squash and merge the Pull Request into the `main` or `production` branch.
2. Merges to `main` will trigger a build with DockerHub with the `dev` tag. Merges to production will create a build with the `prod` tag.
3. Once the images are built the development and production clusters are automatically updated. However there are some caveats to consider:

- Running end-user Notebooks are not automatically culled when a new release is available. The end-user should logout and log back into their environment using the `Logout` button in the Notebook interface.
- The new image is pulled once the the end-user restarts their environment.
- The new image may be a small update or it may required downloading all image layers from scratch. Therefore the first user to launch the new Notebook may epxerience a longer wait time until their Notebook is available.
- Shared Grader Notebooks are not automatically updated by the system. To update a Shared Grader Notebook, please send a support request to IllumiDesk Support in the `#apple` Slack channel. If you do not have access to this channel then you may request support by visiting https://support.illumidesk.com to chat with a customer support agent.

## References

These images are based on the `jupyter/docker-stacks` images. [Refer to their documentation](https://jupyter-docker-stacks.readthedocs.io/en/latest/) for the full set of configuration options.

## Attributions

- [jupyter/docker-stacks images](https://github.com/jupyter/docker-stacks)

## License

MIT