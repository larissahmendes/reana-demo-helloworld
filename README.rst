===============================
 REANA example - "hello world"
===============================

.. image:: https://img.shields.io/travis/reanahub/reana-demo-helloworld.svg
   :target: https://travis-ci.org/reanahub/reana-demo-helloworld

.. image:: https://badges.gitter.im/Join%20Chat.svg
   :target: https://gitter.im/reanahub/reana?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge

.. image:: https://img.shields.io/github/license/reanahub/reana-demo-helloworld.svg
   :target: https://github.com/reanahub/reana-demo-helloworld/blob/master/LICENSE

About
=====

This repository provides a simple "hello world" application example for `REANA
<http://www.reanahub.io/>`_ reusable research data analysis plaftorm. The example
generates greetings for persons specified in an input file.

Analysis structure
==================

Making a research data analysis reproducible basically means to provide
"runnable recipes" addressing (1) where is the input data, (2) what software was
used to analyse the data, (3) which computing environments were used to run the
software and (4) which computational workflow steps were taken to run the
analysis. This will permit to instantiate the analysis on the computational
cloud and run the analysis to obtain (5) output results.

1. Input dataset
----------------

The input file is a text file containing person names:

.. code-block:: console

    $ cat data/names.txt
    Jane Doe
    Joe Bloggs

2. Analysis code
----------------

A simple "hello world" Python code for illustration:

- `helloworld.py <code/helloworld.py>`_

The program takes an input file with the names of the persons to greet and an
optional sleeptime period and produces an output file with the greetings:

.. code-block:: console

    $ python code/helloworld.py \
        --inputfile data/names.txt
        --outputfile results/greetings.txt
        --sleeptime 0
    $ cat results/greetings.txt
    Hello Jane Doe!
    Hello Joe Bloggs!

3. Compute environment
----------------------

In order to be able to rerun the analysis even several years in the future, we
need to "encapsulate the current compute environment", for example to freeze the
Jupyter notebook version and the notebook kernel that our analysis was using. We
shall achieve this by preparing a `Docker <https://www.docker.com/>`_ container
image for our analysis steps.

For example:

.. code-block:: console

    $ cat environments/python/Dockerfile
    FROM python:2.7-slim

Since we don't need any additional Python packages for this simple example to
work, we can directly rely on the ``python`` image from the Docker Hub. The
trailing ``:2.7-slim`` makes sure we are using the Python 2.7 slim image
version.

4. Analysis workflow
====================

This analysis is very simple because it consists basically of running only the
Python executable which will produce a text file.

The workflow can be represented as follows:

.. code-block:: console

                START
                |
                |
                V
    +-------------------------------+
    |                               | <-- inputfile=data/names.txt
    |    $ python helloworld.py ... | <-- sleeptime=0
    +-------------------------------+
                |
                | otuputfile=results/greetings.txt
                V
                STOP


Note that you can also use `CWL <http://www.commonwl.org/v1.0/>`_ or `Yadage
<https://github.com/diana-hep/yadage>`_ workflow specifications:

- `Yadage workflow definition <workflow/yadage/workflow.yaml>`_
- `CWL workflow definition <workflow/cwl/helloworld.cwl>`_

5. Output results
-----------------

The example produces a file greeting all names included in the
`names.txt <data/names.txt>`_ file.

.. code-block:: text

     Hello Jane Doe!
     Hello Joe Bloggs!

Running the example on REANA cloud
==================================

We are now ready to run this example and on the `REANA <http://www.reana.io/>`_
cloud.

First we need to create a `reana.yaml <reana.yaml>`_ file describing the
structure of our analysis with its inputs, the code, the runtime environment,
the computational workflow steps and the expected outputs:


.. code-block:: yaml

    version: 0.3.0
    inputs:
      files:
        - code/helloworld.py
        - data/names.txt
      parameters:
        helloworld: code/helloworld.py
        inputfile: data/names.txt
        outputfile: results/greetings.txt
        sleeptime: 0
    workflow:
      type: serial
      specification:
        steps:
          - environment: 'python:2.7-slim'
            commands:
              - python "${helloworld}"
                  --inputfile "${inputfile}"
                  --outputfile "${outputfile}"
                  --sleeptime ${sleeptime}
    outputs:
      files:
       - results/greetings.txt
       
We can now install the REANA command-line client, run the analysis and download the resulting file:

1. Prerequisites
-----------------

REANA cluster uses Kubernetes container orchestration system. The best way to try it out locally on your laptop is to install (follow instructions at https://reana-cluster.readthedocs.io/en/latest/userguide.html#delete-reana-cluster-deployment):
   - docker 
   - helm
   - kubectl 
   - minikube
   - virtualbox 
   
2. Start minikube
-----------------

Once you have installed kubectl and minikube, you can start minikube by running:

.. code-block:: console

    $ # start minikube
    $ minikube config set memory 4096
    $ minikube start --vm-driver=virtualbox --feature-gates="TTLAfterFinished=true"
    
3. Install reana-cluster
-----------------

.. code-block:: console

    $ # create new virtual environment
    $ virtualenv ~/.virtualenvs/myreana
    $ source ~/.virtualenvs/myreana/bin/activate
    $ # install reana-cluster utility
    $ pip install reana-cluster
    
4. Start reana-cluster
-----------------

.. code-block:: console

    $ reana-cluster init
    REANA cluster is initialised.
    
This may take a couple of minutes. You can verify whether the REANA cluster is ready to serve the user requests by running the ``status`` command:

.. code-block:: console

    $ reana-cluster status
    COMPONENT             STATUS           
    message-broker        Running          
    server                ContainerCreating
    workflow-controller   ContainerCreating
    db                    ContainerCreating
    cache                 ContainerCreating
    REANA cluster is not ready.
    $
    $ reana-cluster status
    COMPONENT             STATUS           
    message-broker        Running          
    server                Running
    workflow-controller   Running
    db                    Running
    cache                 Running
    REANA cluster is ready.
    
5. Display commands to set up the environment for the REANA client
-----------------

You can print the list of commands to configure the environment for the reana-client:

.. code-block:: console

    $ reana-cluster env --include-admin-token
    export REANA_SERVER_URL=http://192.168.39.247:31106
    export REANA_ACCESS_TOKEN=pE2Q3It1C-fjisismHD7djAAjk6alkf0ADRZlg_nYY76k

You can execute the displayed command as:

.. code-block:: console

    $ eval $(reana-cluster env --include-admin-token)
    
You can now run REANA examples on the locally-deployed cluster using reana-client.

6. Install REANA client

.. code-block:: console

    $ pip install reana-client
   
You now should be able to comunicate with the REANA cloud. You can test the connection doing:

.. code-block:: console

    $ reana-client ping
    Server is running.

If you have an error message like: 

.. code-block:: console

    $ reana-client ping
    Could not connect to the selected REANA cluster server at http://192.168.39.247:31106.
    
Try adding an "s" to the adress, doing:    

.. code-block:: console

    $ export REANA_SERVER_URL=https://192.168.39.247:31106
    
Now, `` reana-client ping ` should work.
    
7. Run the analysis

You can now create a new computational workflow:

.. code-block:: console

    $ reana-client create
    workflow.1
    
You can check the status of our previously created workflow:

.. code-block:: console

    $ reana-client status -w workflow.1
    NAME       RUN_NUMBER   CREATED               STATUS    PROGRESS
    workflow   1            2018-08-10T07:27:15   created   -/-

Instead of passing -w argument with the workflow name every time, we can define a new environment variable REANA_WORKON which specifies the workflow we would like to work on:


.. code-block:: console

    $ export REANA_WORKON=workflow.1

Now, to upload the code:

.. code-block:: console

    $ reana-client upload ./code/helloworld.py
    File code/helloworld.py was successfully uploaded.
    $ reana-client upload ./data/names.txt
    File data/names.txt was successfully uploaded.
    
and check whether it was well seeded into our input workspace:

.. code-block:: console

    $ # list workspace files
    $ reana-client ls
    NAME                 SIZE   LAST-MODIFIED
    data/names.txt         18   2018-08-10T07:31:15
    code/helloworld.py   2905   2018-08-10T07:29:54    
    
Now that the input data and code was uploaded, we can start the workflow execution:

.. code-block:: console

    $ # start computational workflow
    $ reana-client start
    workflow.1 has been started.

Let us enquire about its running status; we may see that it is still in the “running” state:

.. code-block:: console

    $ reana-client status
    NAME       RUN_NUMBER   CREATED               STATUS    PROGRESS
    workflow   1            2018-08-10T07:27:15   running   -/-

    $ reana-client status
    NAME       RUN_NUMBER   CREATED               STATUS    PROGRESS
    workflow   1            2018-08-10T07:27:15   running   0/1

After a few minutes, the workflow should be finished:

.. code-block:: console

    $ reana-client status
    NAME       RUN_NUMBER   CREATED               STATUS     PROGRESS
    workflow   1            2018-08-10T07:27:15   finished   1/1
    
You can now check the list of output files:

.. code-block:: console

    $ reana-client ls
    NAME                    SIZE   LAST-MODIFIED
    code/helloworld.py      2905   2018-08-06T13:58:21
    data/names.txt            18   2018-08-06T13:59:59
    results/greetings.txt     32   2018-08-06T14:01:02

and download the resulting output file:

.. code-block:: console

    $ reana-client download results/greetings.txt
    File results/greetings.txt downloaded to /home/reana/reanahub/reana-demo-helloworld.


The output will be

.. code-block:: console

    $ cat results/greetings.txt
    Hello Jane Doe!
    Hello Joe Bloggs!
    
 
For more details access:
============

    - https://github.com/reanahub/reana-demo-helloworld
    - https://reana-cluster.readthedocs.io/en/latest/gettingstarted.html
    - https://reana-cluster.readthedocs.io/en/latest/userguide.html#delete-reana-cluster-deployment
    - https://reana-client.readthedocs.io/en/latest/gettingstarted.html#install-reana-client
