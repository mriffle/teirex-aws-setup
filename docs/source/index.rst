===================================================
TEI-REX Workflows - Guide for setting up AWS Batch
===================================================
AWS Batch is a service provided by Amazon's cloud computing service called Amazon Web Services (AWS). It is
effectively an extremely large computer cluster that will bring up computing resources as you need
them, instead of having the computers sitting idle when not in use. These set of documents will
serve as a guide for setting up AWS Batch as an executor for the Nextflow workflows developed for
the IARPA TEI-REX project.

.. important::

   You are not required to set up AWS Batch to use the TEI-REX workflows. By default they will work
   run tasks on the same computer on which you are executing the ``nextflow`` command. This guide
   describes how to set up AWS Batch for remote cloud-based execution of the steps instead.

What are executors?
===================
Executors are a concept in Nextflow workflows that represent the actual computer resources that
will be running the steps of the workflow. Example include your local computer, a traditional
computer cluster (e.g., Condor or Slurm), AWS Batch, and many others. The TEI-REX Nextflow workflows
have been designed such that different executors can be used interchangeably. After following this
guide, users will be able to specify *awsbatch* as their executor, and the steps of the workflow
will run remotely on AWS Batch.

Technical expertise required
============================
The intention of this guide is to provide the necessary information to users with existing expertise
with setting up AWS services with the information needed to properly set up and configure AWS Batch
for use with the TEI-REX Nextflow workflows. As such, some technical expertise, particularly with AWS
is assumed.

Getting started
===============
To get started, follow the links below (or in the left-hand navigation panel) for information on setting
up and configuring AWS Batch and where to find the parameters required by the workflow for running on
AWS Batch.

Getting Help, Providing Feedback, or Reporting Problems
=======================================================
If you need help, have ideas for new features, encounter a problem, or have any questions or comments, please contact Michael Riffle at mriffle@uw.edu.

Documentation Sections
=======================================================

.. toctree::
   :maxdepth: 2

   self
   set_up_aws
   configure_workflow