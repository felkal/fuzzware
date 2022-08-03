# Continuous integration (Gitlab)
To be able to test new features or to be sure that changes not only build on the local system, we set up a test environment featuring several stages.
The stages are executed both for the docker-based setup [.github/workflows/docker.yml](.github/workflows/docker.yml) and the local [.github/workflows/local.yml](.github/workflows/local.yml) setup.

1. Build
2. Fuzzing
3. Fuzzware commands
4. Test self-containment of the fuzzware-project folder

## 1. Build

During the build stage we make sure that ssh is setup correctly with the corresponding key for our runner instance. Furthermore
We checkout fuzzware's submodules with
```yml
    - run: git submodule sync --recursive
    - run: git submodule update --init --recursive
```
This is done both for the local setup and the docker build.
To work-around github-runner using different architectures we remove `-march=native` from the fuzzware-unicorn Makefile.

## 2. Fuzzing

While the docker-fuzzing stage is pretty straight forward, need to use some workarounds for the local setup:
```yml
    run: |
        VIRTUALENVWRAPPER_PYTHON="${VIRTUALENVWRAPPER_PYTHON:=$(which python3)}"
        export VIRTUALENVWRAPPER_PYTHON
        source $(which virtualenvwrapper.sh)
        set +e
        workon fuzzware
```
Before we can use 'workon fuzzware' we need to set the VIRTUALENVWRAPPER_PYTHON environment variable and source the virtualenvwrapper.sh script.
Additionally we need to tell bash to not stop at the first error using 'set +e', otherwise we can't enter the python venv.
At the end of each fuzzing job the fuzzware-project-folder is cached so we can transfer it to the next stage.
```yml
    needs: build_local
```
With the 'needs' keyword we influence the job sequence.

## 3. Fuzzware commands

In this stage we test several fuzzware subcommands to make sure everything works as expected.


## 4. Test self-containment of the fuzzware-project folder

After we got the fuzzware-project folder from the gitlab artifact, we move it to another place to detach it from predefined paths.

```yml
- name: Move project directory
        run: mv examples/pw-recovery/ARCH_PRO/fuzzware-project/ ./still_replayable_ARCH_PRO
```
Afterwards we execute the same fuzzware commands as in the previous stage.
