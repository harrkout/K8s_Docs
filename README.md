# Kubernetes Guide 
 

Personal documentation while learning to deploy containers 

>Ideally this will be converted over time to the ReadTheDocs documentation container. 


### The guide can be seen from here in the directory:

`docs/source`

---------------

Clone the readthedocs.org repository:

`git clone --recurse-submodules https://github.com/readthedocs/readthedocs.org/` <br/>

---------------

Install the requirements from common submodule:

`pip install -r common/dockerfiles/requirements.txt`

---------------

Build the Docker image:

`inv docker.build`

---------------

Pull down Docker images for the builders:

`inv docker.pull --only-required`

---------------

Start all the containers:

`inv docker.up  --init  # --init is only needed the first time`

---------------

Visit http://devthedocs.org
