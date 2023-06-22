=========================================
Configure and Run Workflow for AWS Batch
=========================================

You will need three values from the :doc:`set_up_aws` guide to configure the workflow:

1. The path to the S3 bucket that was set up. This will be referred to as ``BUCKET_PATH`` below.
2. The logs group that was set up. This will be referred to as ``LOGS_GROUP`` below.
3. The job queue name. This will be referred to as ``JOB_QUEUE`` below.

Follow the directions below to configure and run your workflow using AWS Batch.

Configure Workflow
===================
Each of the sets of documentation for the different TEI-REX workflows (see: https://mriffle.github.io/teirex-workflows/)
includes a description of the ``pipeline.config`` file for that workflow. Regardless of which workflow you are running, the
``pipeline.config`` file will need the same modifications in order to run the workflow on AWS. We recommend making these
modifications once and re-using the same ``pipeline.config`` for the respective workflow, making modifications as necessary
for the specific data being searched.

First, add the following block to the end of your ``pipeline.config`` file. Note, you will need to substitute in the
value for ``LOGS_GROUP`` (described above). If you configured your AWS Batch in a region other than *us-west-2*, you
should update the value for ``region`` below to match your region.

.. code-block:: groovy

    aws {

        batch {
            // NOTE: this setting is only required if the AWS CLI tool is installed in a custom AMI
            cliPath = '/usr/local/aws-cli/v2/current/bin/aws'
            logsGroup = 'LOGS_GROUP'
            maxConnections = 20
            connectionTimeout = 10000
            uploadStorageClass = 'INTELLIGENT_TIERING'
            storageEncryption = 'AES256'
            retryMode = 'standard'
        }

        region = 'us-west-2'
    }

Next, update the ``profiles`` section of ``pipeline.config`` to include an ``aws`` section. Before adding the ``aws`` section, this 
will appearly similarly to:

.. code-block:: groovy

    profiles {

        // "standard" is the profile used when the steps of the workflow are run
        // locally on your computer. These parameters should be changed to match
        // your system resources (that you are willing to devote to running
        // workflow jobs).
        standard {
            params.max_memory = '8.GB'
            params.max_cpus = 4
            params.max_time = '240.h'

            params.mzml_cache_directory = '/data/mass_spec/nextflow/nf-teirex-dda/mzml_cache'
            params.panorama_cache_directory = '/data/mass_spec/nextflow/panorama/raw_cache'
        }
    }

After adding the ``aws`` section, this will appear similarly to:

.. code-block:: groovy

    profiles {

        // "standard" is the profile used when the steps of the workflow are run
        // locally on your computer. These parameters should be changed to match
        // your system resources (that you are willing to devote to running
        // workflow jobs).
        standard {
            params.max_memory = '8.GB'
            params.max_cpus = 4
            params.max_time = '240.h'

            params.mzml_cache_directory = '/data/mass_spec/nextflow/nf-teirex-dda/mzml_cache'
            params.panorama_cache_directory = '/data/mass_spec/nextflow/panorama/raw_cache'
        }

        // "aws" profile -- parameters used for running jobs on AWS Batch
        aws {
            process.executor = 'awsbatch'
            process.queue = 'JOB_QUEUE'

            // params for running pipeline on aws batch
            // These can be overridden in local config file

            // max params allowed for your AWS Batch compute environment
            params.max_memory = '124.GB'
            params.max_cpus = 32
            params.max_time = '240.h'

            // where to cache mzml files after running msconvert
            params.mzml_cache_directory = 's3://BUCKET_PATH/mzml_cache'
            params.panorama_cache_directory = 's3://BUCKET_PATH/panorama_cache'
        }
    }

Replace ``JOB_QUEUE`` and ``BUCKET_PATH`` with the values described above. A description of each
parameter is below:

- ``process.executor`` - This instructs Nextflow which executor to use when the *aws* profile is used. Do not change.
- ``process.queue`` - This is the job queue set up for AWS Batch (see: :doc:`set_up_aws` guide).
- ``params.max_memory`` - This is the maximum amount of memory a task may use in the workflow. This should not exceed the maximum memory available in the hardware profiles for the AWS Batch compute environment.
- ``params.max_cpus`` - This is the maximum number of cores a task may use in the workflow. This should not exceed the maximum cores available in the hardware profiles for the AWS Batch compute environment.
- ``params.max_time`` - This is the maximum amount of time (in hours) a task may be run before being automatically killed and retried.
- ``params.mzml_cache_directory`` - This is the path to the cache of mzML files created by processing raw files.
- ``params.panorama_cache_directory`` - This is the path to the cache of raw files created by downloading raw files from PanoramaWeb. If not using PanoramaWeb a value is still required, but will never be used.

Run Workflow
============
Once all the AWS setup and configuration is complete, running the workflow on AWS is very simple: merely specify ``aws`` as the profile and specify the S3 work directory on the command line. For example:

- For the DDA workflow:

.. code-block:: bash

    nextflow run -resume -r main -profile aws mriffle/nf-teirex-dda -bucket-dir s3://BUCKET_PATH/work -c pipeline.config

- For the DIA workflow:

.. code-block:: bash

    nextflow run -resume -r main -profile aws mriffle/nf-teirex-dia -bucket-dir s3://BUCKET_PATH/work -c pipeline.config

.. note::

    Replace ``BUCKET_PATH`` with the S3 bucket you set up in the :doc:`set_up_aws` guide.

