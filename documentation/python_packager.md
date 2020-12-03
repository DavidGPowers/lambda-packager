# Python Lambda Packager

## BUGS:
- **Local Paths Fail**: To flip ssh credentials to https within requirments.txt files (if ssh fails) this packager uses the PyPi package [`requirements-parser`](https://pypi.org/project/requirements-parser/). Unfortanitely there is a bug with the current parser when working with local paths such as `./my_packages/my_sweet_package`, this can be hacked around but this bug will remain till the ticket is fixed: [See the ticket and hack around here](https://github.com/davidfischer/requirements-parser/issues/54). You can also set `SSH_FLIP=False` (the default) and this issue will not occur.

## TL;DR
Lets say all our code is within a `src` directory and within that `src` directory we have a `requirements.txt`. We want the output to be `deployment.zip` at the root of our project. Then all we need to do is:
```bash
	docker run -it --rm \
        -v $(if ${PWD},${PWD},${CURDIR}):/src \ # First mount our code to the container
		3mcloud/lambda-packager:python-3.6
```
And **boom**, `deployment.zip` should be in your repository root.

**Note:** `$(if ${PWD},${PWD},${CURDIR})` is a ternary operator which we use to make it Windows and Mac agnostic with Makefiles.
______
## Container Variables

You can change the default behavior of this packager by using the environment variables of the container

| Variable Name | Default | Required | Description |
|-------------- | ------- | ---------| ----------- |
| `MANIFEST_FILE` | ` ` | no | If you only want to package a single lambda, ignore this environment variable, else see [Manifest File](./python_manifest_file.md). This is the path, relative to `CI_WORKSPACE` or absolute, to the manifest file. Empty (default) if you don't want to use it. |
| `LAMBDA_CODE_DIR` | `./src` | no | Directory, relative to `CI_WORKSPACE` (or absolute), that is to be packaged and zipped. |
| `CONTAINER_BUILD_DIRECTORY` | `/build` | no | The directory within the container where the code is moved to, the pip install is done, and the code is zipped from. Shouldn't need to be changed. |
| `ARTIFACT_NAME` | `deployment.zip` | no | Path and name of the zip file (or artifact) that will be outputted. Relative to `CI_WORKSPACE` (or absolute). |
| `GLOB_IGNORE` | `*.pyc,__pycache__` | no | Comma delimited glob expressions for files and folder to ignore while zipping. |
| `CI_WORKSPACE` | `$(pwd) - i.e. root level` | no | Workspace directory within the container. |
| `REQUIREMENTS_FILE` | `requirements.txt` | no | Path relative to `LAMBDA_CODE_DIR` (or absolute) where the requirements file is. If not requirements file is found, it will attempt to use the setup.py |
| `SETUP_FILE` | `setup.py` | no | Path relative to `LAMBDA_CODE_DIR` (or absolute) where the setup.py file is. |
| `MAX_LAMBDA_SIZE_BYTES` | `52428800` | no | Used for lambda size checking message. Should be an integer value which represents the maximum size of a lambda in bytes. |
| `FAIL_ON_TOO_BIG` | `False` | no | If set to `True`, the container will exit with a status code of `1` if the lambda is too big. |
| `SSH_FLIP` | `False` | no | Only relevant if you are pip installing directly from a repo using `ssh` or `https`. If set to `True` and you are using a `REQUIREMENTS_FILE`, the container will check if it has access to `ssh`. If the container does not have access, then it will swap all requirements that use `ssh` to use `https`. |


________________
## Examples


Our code is in a directory named `code` and we have a `reqs.txt` file within that directory. We want `deployment.zip` to be at the root level of our project.
```bash
	docker run -it --rm \
        -v $(if ${PWD},${PWD},${CURDIR}):/src \ # First mount our project to the container
        -e CI_WORKSPACE=/src \ # We will be working with /src
		-e LAMBDA_CODE_DIR=/code \ # The code to be packaged is within /src/code
		-e REQUIREMENTS_FILE=reqs.txt \ # Requirements are in /src/code/reqs.txt
		3mcloud/lambda-packager:python-3.6
```
Alternatively, lets say we only want to mount the `code` directory and not the entire root of our project:
```bash
	docker run -it --rm \
        -v $(if ${PWD},${PWD},${CURDIR})/code:/src \ # First mount our code to the container
		-e LAMBDA_CODE_DIR=/src \ # The code to be packaged is within /src (Note CI_WORKSPACE is /)
		-e REQUIREMENTS_FILE=reqs.txt \ # Requirements are in /src/reqs.txt
        -e ARTIFACT_NAME=/src/deployment.zip \ # We need to save the zip within /src/ since we didn't mount the root level.
		3mcloud/lambda-packager:python-3.6
```
Lasty, we can try to leverage the default values. Lets say all our code is within a `src` directory and within that `src` directory we have a `requirements.txt`. We want the output to be `deployment.zip` at the root of our project. Then all we need to do is:
```bash
	docker run -it --rm \
        -v $(if ${PWD},${PWD},${CURDIR}):/src \ # First mount our code to the container
		3mcloud/lambda-packager:python-3.6
```