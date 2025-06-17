Nowadays containerized code such as docker is very popular, though we still would like to segregate our modules and environment for our project.

Venv is a popular way to set up a virtual environment in python to avoid conflicting versions and modules for different python codes.

It is always confusing to set up a venv in python this is to memo those methods.


### Setting up VENV

`python -m venv ".venv"`
`python3.11 -m venv ".venv"`

python3 and on supports venv inately
can specify the python version by adding the version upfront

`﻿virtualenv .venv --python=python3.8`

using the virtualenv module installed via pip
can specify the python version by adding --python=python3.11 at the back
