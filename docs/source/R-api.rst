.. _R-api:

========
R API
========

The MLflow R API allows you to use MLflow :doc:`Tracking <tracking/>`, :doc:`Projects <projects/>` and :doc:`Models <models/>`.

You can use the R API to `install MLflow`_, start the `user interface <Run MLflow user interface_>`_, `create <Create Experiment_>`_ and `list experiments <List Experiments_>`_, `save models <Save Model for MLflow_>`_, `run projects <Run in MLflow_>`_ and `serve models <Serve an RFunc MLflow Model_>`_ among many other functions available in the R API.

.. contents:: Table of Contents
    :local:
    :depth: 1

Crate a function to share with another process
==============================================

``crate()`` creates functions in a self-contained environment
(technically, a child of the base environment). This has two advantages:

-  They can easily be executed in another process.

-  Their effects are reproducible. You can run them locally with the
   same results as on a different process.

Creating self-contained functions requires some care, see section below.

.. code:: r

   crate(.fn, ...)

Arguments
---------

+-------------------------------+--------------------------------------+
| Argument                      | Description                          |
+===============================+======================================+
| ``.fn``                       | A fresh formula or function. “Fresh” |
|                               | here means that they should be       |
|                               | declared in the call to ``crate()``. |
|                               | See examples if you need to crate a  |
|                               | function that is already defined.    |
|                               | Formulas are converted to purrr-like |
|                               | lambda functions using               |
|                               | [rlang::as_function()].              |
+-------------------------------+--------------------------------------+
| ``...``                       | Arguments to declare in the          |
|                               | environment of ``.fn``. If a name is |
|                               | supplied, the object is assigned to  |
|                               | that name. Otherwise the argument is |
|                               | automatically named after itself.    |
+-------------------------------+--------------------------------------+

Examples
--------

.. code:: r

    # You can create functions using the ordinary notation:
    crate(function(x) stats::var(x))
    
    # Or the formula notation:
    crate(~stats::var(.x))
    
    # Declare data by supplying named arguments. You can test you have
    # declared all necessary data by calling your crated function:
    na_rm <- TRUE
    fn <- crate(~stats::var(.x, na.rm = na_rm))
    try(fn(1:10))
    
    # Arguments are automatically named after themselves so that the
    # following are equivalent:
    crate(~stats::var(.x, na.rm = na_rm), na_rm = na_rm)
    crate(~stats::var(.x, na.rm = na_rm), na_rm)
    
    # However if you supply a complex expression, do supply a name!
    crate(~stats::var(.x, na.rm = na_rm), !na_rm)
    crate(~stats::var(.x, na.rm = na_rm), na_rm = na_rm)
    
    # For small data it is handy to unquote instead. Unquoting inlines
    # objects inside the function. This is less verbose if your
    # function depends on many small objects:
    fn <- crate(~stats::var(.x, na.rm = !!na_rm))
    fn(1:10)
    
    # One downside is that the individual sizes of unquoted objects
    # won't be shown in the crate printout:
    fn
    
    
    # The function or formula you pass to crate() should defined inside
    # the crate() call, i.e. you can't pass an already defined
    # function:
    fn <- function(x) toupper(x)
    try(crate(fn))
    
    # If you really need to crate an existing function, you can
    # explicitly set its environment to the crate environment with the
    # set_env() function from rlang:
    crate(rlang::set_env(fn))

Is an object a crate?
=====================

Is an object a crate?

.. code:: r

   is_crate(x)

.. _arguments-1:

Arguments
---------

+----------+--------------------+
| Argument | Description        |
+==========+====================+
| ``x``    | An object to test. |
+----------+--------------------+

Active Run
==========

Retrieves the active run.

.. code:: r

   mlflow_active_run()

MLflow Command
==============

Runs a generic MLflow command through the command-line interface.

.. code:: r

   mlflow_cli(..., background = FALSE, echo = TRUE,
     stderr_callback = NULL)

.. _arguments-2:

Arguments
---------

+-------------------------------+--------------------------------------+
| Argument                      | Description                          |
+===============================+======================================+
| ``...``                       | The parameters to pass to the        |
|                               | command line.                        |
+-------------------------------+--------------------------------------+
| ``background``                | Should this command be triggered as  |
|                               | a background task? Defaults to       |
|                               | ``FALSE`` .                          |
+-------------------------------+--------------------------------------+
| ``echo``                      | Print the standard output and error  |
|                               | to the screen? Defaults to ``TRUE``  |
|                               | , does not apply to background       |
|                               | tasks.                               |
+-------------------------------+--------------------------------------+
| ``stderr_callback``           | NULL, or a function to call for      |
|                               | every chunk of the standard error.   |
+-------------------------------+--------------------------------------+

Value
-----

A ``processx`` task.

.. _examples-1:

Examples
--------

.. code:: r

    list("\n", "library(mlflow)\n", "mlflow_install()\n", "\n", "mlflow_cli(\"server\", \"--help\")\n") 
    

Create Experiment - Tracking Client
===================================

Creates an MLflow experiment.

.. code:: r

   mlflow_client_create_experiment(client, name, artifact_location = NULL)

.. _arguments-3:

Arguments
---------

+-------------------------------+--------------------------------------+
| Argument                      | Description                          |
+===============================+======================================+
| ``client``                    | An ``mlflow_client`` object.         |
+-------------------------------+--------------------------------------+
| ``name``                      | The name of the experiment to        |
|                               | create.                              |
+-------------------------------+--------------------------------------+
| ``artifact_location``         | Location where all artifacts for     |
|                               | this experiment are stored. If not   |
|                               | provided, the remote server will     |
|                               | select an appropriate default.       |
+-------------------------------+--------------------------------------+

Details
-------

The Tracking Client family of functions require an MLflow client to be
specified explicitly. These functions allow for greater control of where
the operations take place in terms of services and runs, but are more
verbose compared to the Fluent API.

Create Run
==========

Create a new run within an experiment. A run is usually a single
execution of a machine learning or data ETL pipeline.

.. code:: r

   mlflow_client_create_run(client, experiment_id, user_id = NULL,
     run_name = NULL, source_type = NULL, source_name = NULL,
     entry_point_name = NULL, start_time = NULL, source_version = NULL,
     tags = NULL)

.. _arguments-4:

Arguments
---------

+-------------------------------+--------------------------------------+
| Argument                      | Description                          |
+===============================+======================================+
| ``client``                    | An ``mlflow_client`` object.         |
+-------------------------------+--------------------------------------+
| ``experiment_id``             | Unique identifier for the associated |
|                               | experiment.                          |
+-------------------------------+--------------------------------------+
| ``user_id``                   | User ID or LDAP for the user         |
|                               | executing the run.                   |
+-------------------------------+--------------------------------------+
| ``run_name``                  | Human readable name for run.         |
+-------------------------------+--------------------------------------+
| ``source_type``               | Originating source for this run. One |
|                               | of Notebook, Job, Project, Local, or |
|                               | Unknown.                             |
+-------------------------------+--------------------------------------+
| ``source_name``               | String descriptor for source. For    |
|                               | example, name or description of the  |
|                               | notebook, or job name.               |
+-------------------------------+--------------------------------------+
| ``entry_point_name``          | Name of the entry point for the run. |
+-------------------------------+--------------------------------------+
| ``start_time``                | Unix timestamp of when the run       |
|                               | started in milliseconds.             |
+-------------------------------+--------------------------------------+
| ``source_version``            | Git version of the source code used  |
|                               | to create run.                       |
+-------------------------------+--------------------------------------+
| ``tags``                      | Additional metadata for run in       |
|                               | key-value pairs.                     |
+-------------------------------+--------------------------------------+

.. _details-1:

Details
-------

MLflow uses runs to track Param, Metric, and RunTag, associated with a
single execution.

The Tracking Client family of functions require an MLflow client to be
specified explicitly. These functions allow for greater control of where
the operations take place in terms of services and runs, but are more
verbose compared to the Fluent API.

Delete Experiment
=================

Marks an experiment and associated runs, params, metrics, etc. for
deletion. If the experiment uses FileStore, artifacts associated with
experiment are also deleted.

.. code:: r

   mlflow_client_delete_experiment(client, experiment_id)

.. _arguments-5:

Arguments
---------

+-----------------------------------+-----------------------------------+
| Argument                          | Description                       |
+===================================+===================================+
| ``client``                        | An ``mlflow_client`` object.      |
+-----------------------------------+-----------------------------------+
| ``experiment_id``                 | ID of the associated experiment.  |
|                                   | This field is required.           |
+-----------------------------------+-----------------------------------+

.. _details-2:

Details
-------

The Tracking Client family of functions require an MLflow client to be
specified explicitly. These functions allow for greater control of where
the operations take place in terms of services and runs, but are more
verbose compared to the Fluent API.

Delete a Run
============

Delete a Run

.. code:: r

   mlflow_client_delete_run(client, run_id)

.. _arguments-6:

Arguments
---------

+------------+------------------------------+
| Argument   | Description                  |
+============+==============================+
| ``client`` | An ``mlflow_client`` object. |
+------------+------------------------------+
| ``run_id`` | Run ID.                      |
+------------+------------------------------+

.. _details-3:

Details
-------

The Tracking Client family of functions require an MLflow client to be
specified explicitly. These functions allow for greater control of where
the operations take place in terms of services and runs, but are more
verbose compared to the Fluent API.

Download Artifacts
==================

Download an artifact file or directory from a run to a local directory
if applicable, and return a local path for it.

.. code:: r

   mlflow_client_download_artifacts(client, run_id, path)

.. _arguments-7:

Arguments
---------

+------------+-----------------------------------------------+
| Argument   | Description                                   |
+============+===============================================+
| ``client`` | An ``mlflow_client`` object.                  |
+------------+-----------------------------------------------+
| ``run_id`` | Run ID.                                       |
+------------+-----------------------------------------------+
| ``path``   | Relative source path to the desired artifact. |
+------------+-----------------------------------------------+

.. _details-4:

Details
-------

The Tracking Client family of functions require an MLflow client to be
specified explicitly. These functions allow for greater control of where
the operations take place in terms of services and runs, but are more
verbose compared to the Fluent API.

Get Experiment by Name
======================

Gets metadata for an experiment by name.

.. code:: r

   mlflow_client_get_experiment_by_name(client, name)

.. _arguments-8:

Arguments
---------

+------------+------------------------------+
| Argument   | Description                  |
+============+==============================+
| ``client`` | An ``mlflow_client`` object. |
+------------+------------------------------+
| ``name``   | The experiment name.         |
+------------+------------------------------+

.. _details-5:

Details
-------

The Tracking Client family of functions require an MLflow client to be
specified explicitly. These functions allow for greater control of where
the operations take place in terms of services and runs, but are more
verbose compared to the Fluent API.

Get Experiment
==============

Gets metadata for an experiment and a list of runs for the experiment.

.. code:: r

   mlflow_client_get_experiment(client, experiment_id)

.. _arguments-9:

Arguments
---------

+-------------------+---------------------------------+
| Argument          | Description                     |
+===================+=================================+
| ``client``        | An ``mlflow_client`` object.    |
+-------------------+---------------------------------+
| ``experiment_id`` | Identifer to get an experiment. |
+-------------------+---------------------------------+

.. _details-6:

Details
-------

The Tracking Client family of functions require an MLflow client to be
specified explicitly. These functions allow for greater control of where
the operations take place in terms of services and runs, but are more
verbose compared to the Fluent API.

Get Run
=======

Gets metadata, params, tags, and metrics for a run. Only last logged value
for each metric is returned.

.. code:: r

   mlflow_client_get_run(client, run_id)

.. _arguments-10:

Arguments
---------

+------------+------------------------------+
| Argument   | Description                  |
+============+==============================+
| ``client`` | An ``mlflow_client`` object. |
+------------+------------------------------+
| ``run_id`` | Run ID.                      |
+------------+------------------------------+

.. _details-7:

Details
-------

The Tracking Client family of functions require an MLflow client to be
specified explicitly. These functions allow for greater control of where
the operations take place in terms of services and runs, but are more
verbose compared to the Fluent API.

List Artifacts
==============

Gets a list of artifacts.

.. code:: r

   mlflow_client_list_artifacts(client, run_id, path = NULL)

.. _arguments-11:

Arguments
---------

+-------------------------------+--------------------------------------+
| Argument                      | Description                          |
+===============================+======================================+
| ``client``                    | An ``mlflow_client`` object.         |
+-------------------------------+--------------------------------------+
| ``run_id``                    | Run ID.                              |
+-------------------------------+--------------------------------------+
| ``path``                      | The run's relative artifact path to  |
|                               | list from. If not specified, it is   |
|                               | set to the root artifact path        |
+-------------------------------+--------------------------------------+

.. _details-8:

Details
-------

The Tracking Client family of functions require an MLflow client to be
specified explicitly. These functions allow for greater control of where
the operations take place in terms of services and runs, but are more
verbose compared to the Fluent API.

List Experiments
================

Gets a list of all experiments.

.. code:: r

   mlflow_client_list_experiments(client, view_type = c("ACTIVE_ONLY",
     "DELETED_ONLY", "ALL"))

.. _arguments-12:

Arguments
---------

+-------------------------------+--------------------------------------+
| Argument                      | Description                          |
+===============================+======================================+
| ``client``                    | An ``mlflow_client`` object.         |
+-------------------------------+--------------------------------------+
| ``view_type``                 | Qualifier for type of experiments to |
|                               | be returned. Defaults to             |
|                               | ``ACTIVE_ONLY``.                     |
+-------------------------------+--------------------------------------+

.. _details-9:

Details
-------

The Tracking Client family of functions require an MLflow client to be
specified explicitly. These functions allow for greater control of where
the operations take place in terms of services and runs, but are more
verbose compared to the Fluent API.

Log Artifact
============

Logs a specific file or directory as an artifact for a run.

.. code:: r

   mlflow_client_log_artifact(client, run_id, path, artifact_path = NULL)

.. _arguments-13:

Arguments
---------

+-------------------+-------------------------------------------------+
| Argument          | Description                                     |
+===================+=================================================+
| ``client``        | An ``mlflow_client`` object.                    |
+-------------------+-------------------------------------------------+
| ``run_id``        | Run ID.                                         |
+-------------------+-------------------------------------------------+
| ``path``          | The file or directory to log as an artifact.    |
+-------------------+-------------------------------------------------+
| ``artifact_path`` | Destination path within the run's artifact URI. |
+-------------------+-------------------------------------------------+

.. _details-10:

Details
-------

The Tracking Client family of functions require an MLflow client to be
specified explicitly. These functions allow for greater control of where
the operations take place in terms of services and runs, but are more
verbose compared to the Fluent API.

When logging to Amazon S3, ensure that the user has a proper policy
attach to it.

Additionally, at least the ``AWS_ACCESS_KEY_ID`` and
``AWS_SECRET_ACCESS_KEY`` environment variables must be set to the
corresponding key and secrets provided by Amazon IAM.

Log Metric
==========

Logs a metric for a run. Metrics key-value pair that records a
single float measure. During a single execution of a run, a particular
metric can be logged several times. Backend will keep track of
historical values along with timestamps.

.. code:: r

   mlflow_client_log_metric(client, run_id, key, value, timestamp = NULL)

.. _arguments-14:

Arguments
---------

+-----------------------------------+-----------------------------------+
| Argument                          | Description                       |
+===================================+===================================+
| ``client``                        | An ``mlflow_client`` object.      |
+-----------------------------------+-----------------------------------+
| ``run_id``                        | Run ID.                           |
+-----------------------------------+-----------------------------------+
| ``key``                           | Name of the metric.               |
+-----------------------------------+-----------------------------------+
| ``value``                         | Float value for the metric being  |
|                                   | logged.                           |
+-----------------------------------+-----------------------------------+
| ``timestamp``                     | Unix timestamp in milliseconds at |
|                                   | the time metric was logged.       |
+-----------------------------------+-----------------------------------+

.. _details-11:

Details
-------

The Tracking Client family of functions require an MLflow client to be
specified explicitly. These functions allow for greater control of where
the operations take place in terms of services and runs, but are more
verbose compared to the Fluent API.

Log Parameter
=============

Logs a parameter for a run. Examples are params and
hyperparams used for ML training, or constant dates and values used in
an ETL pipeline. A param is a STRING key-value pair. For a run, a
single parameter is allowed to be logged only once.

.. code:: r

   mlflow_client_log_param(client, run_id, key, value)

.. _arguments-15:

Arguments
---------

+------------+--------------------------------+
| Argument   | Description                    |
+============+================================+
| ``client`` | An ``mlflow_client`` object.   |
+------------+--------------------------------+
| ``run_id`` | Run ID.                        |
+------------+--------------------------------+
| ``key``    | Name of the parameter.         |
+------------+--------------------------------+
| ``value``  | String value of the parameter. |
+------------+--------------------------------+

.. _details-12:

Details
-------

The Tracking Client family of functions require an MLflow client to be
specified explicitly. These functions allow for greater control of where
the operations take place in terms of services and runs, but are more
verbose compared to the Fluent API.

Restore Experiment
==================

Restores an experiment marked for deletion. This also restores associated
metadata, runs, metrics, and params. If experiment uses FileStore,
underlying artifacts associated with experiment are also restored.

.. code:: r

   mlflow_client_restore_experiment(client, experiment_id)

.. _arguments-16:

Arguments
---------

+-----------------------------------+-----------------------------------+
| Argument                          | Description                       |
+===================================+===================================+
| ``client``                        | An ``mlflow_client`` object.      |
+-----------------------------------+-----------------------------------+
| ``experiment_id``                 | ID of the associated experiment.  |
|                                   | This field is required.           |
+-----------------------------------+-----------------------------------+

.. _details-13:

Details
-------

Throws ``RESOURCE_DOES_NOT_EXIST`` if the experiment was never created or was
permanently deleted.

The Tracking Client family of functions require an MLflow client to be
specified explicitly. These functions allow for greater control of where
the operations take place in terms of services and runs, but are more
verbose compared to the Fluent API.

Restore a Run
=============

Restore a Run

.. code:: r

   mlflow_client_restore_run(client, run_id)

.. _arguments-17:

Arguments
---------

+------------+------------------------------+
| Argument   | Description                  |
+============+==============================+
| ``client`` | An ``mlflow_client`` object. |
+------------+------------------------------+
| ``run_id`` | Run ID.                      |
+------------+------------------------------+

.. _details-14:

Details
-------

The Tracking Client family of functions require an MLflow client to be
specified explicitly. These functions allow for greater control of where
the operations take place in terms of services and runs, but are more
verbose compared to the Fluent API.

Set Tag
=======

Sets a tag on a run. Tags are run metadata that can be updated during a run and
after a run completes.

.. code:: r

   mlflow_client_set_tag(client, run_id, key, value)

.. _arguments-18:

Arguments
---------

+-------------------------------+--------------------------------------+
| Argument                      | Description                          |
+===============================+======================================+
| ``client``                    | An ``mlflow_client`` object.         |
+-------------------------------+--------------------------------------+
| ``run_id``                    | Run ID.                              |
+-------------------------------+--------------------------------------+
| ``key``                       | Name of the tag. Maximum size is 255 |
|                               | bytes. This field is required.       |
+-------------------------------+--------------------------------------+
| ``value``                     | String value of the tag being        |
|                               | logged. Maximum size is 500 bytes.   |
|                               | This field is required.              |
+-------------------------------+--------------------------------------+

.. _details-15:

Details
-------

The Tracking Client family of functions require an MLflow client to be
specified explicitly. These functions allow for greater control of where
the operations take place in terms of services and runs, but are more
verbose compared to the Fluent API.

Terminate a Run
===============

Terminates a run.

.. code:: r

   mlflow_client_set_terminated(client, run_id, status = c("FINISHED",
     "SCHEDULED", "FAILED", "KILLED"), end_time = NULL)

.. _arguments-19:

Arguments
---------

+--------------+-------------------------------------------------------+
| Argument     | Description                                           |
+==============+=======================================================+
| ``client``   | An ``mlflow_client`` object.                          |
+--------------+-------------------------------------------------------+
| ``run_id``   | Unique identifier for the run.                        |
+--------------+-------------------------------------------------------+
| ``status``   | Updated status of the run. Defaults to ``FINISHED``.  |
+--------------+-------------------------------------------------------+
| ``end_time`` | Unix timestamp of when the run ended in milliseconds. |
+--------------+-------------------------------------------------------+
| ``run_id``   | Run ID.                                               |
+--------------+-------------------------------------------------------+

.. _details-16:

Details
-------

The Tracking Client family of functions require an MLflow client to be
specified explicitly. These functions allow for greater control of where
the operations take place in terms of services and runs, but are more
verbose compared to the Fluent API.

Initialize an MLflow Client
===========================

Initializes an MLflow client.

.. code:: r

   mlflow_client(tracking_uri = NULL)

.. _arguments-20:

Arguments
---------

+-------------------------------+--------------------------------------+
| Argument                      | Description                          |
+===============================+======================================+
| ``tracking_uri``              | The tracking URI. If not provided,   |
|                               | defaults to the service set by       |
|                               | ``mlflow_set_tracking_uri()``.       |
+-------------------------------+--------------------------------------+

Create Experiment
=================

Creates an MLflow experiment.

.. code:: r

   mlflow_create_experiment(name, artifact_location = NULL)

.. _arguments-21:

Arguments
---------

+-------------------------------+--------------------------------------+
| Argument                      | Description                          |
+===============================+======================================+
| ``name``                      | The name of the experiment to        |
|                               | create.                              |
+-------------------------------+--------------------------------------+
| ``artifact_location``         | Location where all artifacts for     |
|                               | this experiment are stored. If not   |
|                               | provided, the remote server will     |
|                               | select an appropriate default.       |
+-------------------------------+--------------------------------------+

.. _details-17:

Details
-------

The fluent API family of functions operate with an implied MLflow client
determined by the service set by ``mlflow_set_tracking_uri()``. For
operations involving a run it adopts the current active run, or, if one
does not exist, starts one through the implied service.

End a Run
=========

Ends an active MLflow run (if there is one).

.. code:: r

   mlflow_end_run(status = c("FINISHED", "SCHEDULED", "FAILED", "KILLED"))

.. _arguments-22:

Arguments
---------

+------------+------------------------------------------------------+
| Argument   | Description                                          |
+============+======================================================+
| ``status`` | Updated status of the run. Defaults to ``FINISHED``. |
+------------+------------------------------------------------------+

.. _details-18:

Details
-------

The fluent API family of functions operate with an implied MLflow client
determined by the service set by ``mlflow_set_tracking_uri()``. For
operations involving a run it adopts the current active run, or, if one
does not exist, starts one through the implied service.

Get Remote Tracking URI
=======================

Gets the remote tracking URI.

.. code:: r

   mlflow_get_tracking_uri()

Install MLflow
==============

Installs MLflow for individual use.

.. code:: r

   mlflow_install()

.. _details-19:

Details
-------

MLflow requires Python and Conda to be installed. See
https://www.python.org/getit/ and
https://conda.io/docs/installation.html.

.. _examples-2:

Examples
--------

.. code:: r

    list("\n", "library(mlflow)\n", "mlflow_install()\n") 
    

Load MLflow Model Flavor
========================

Loads an MLflow model flavor, to be used by package authors to extend
the supported MLflow models.

.. code:: r

   mlflow_load_flavor(model_path)

.. _arguments-23:

Arguments
---------

+----------------+------------------------------------------------------------+
| Argument       | Description                                                |
+================+============================================================+
| ``model_path`` | The path to the MLflow model wrapped in the correct class. |
+----------------+------------------------------------------------------------+

Load MLflow Model
=================

Loads an MLflow model. MLflow models can have multiple model flavors. Not all flavors / models
can be loaded in R. This method by default searches for a flavor supported by R/MLflow.

.. code:: r

   mlflow_load_model(model_path, flavor = NULL, run_id = NULL)

.. _arguments-24:

Arguments
---------

+-------------------------------+--------------------------------------+
| Argument                      | Description                          |
+===============================+======================================+
| ``model_path``                | Path to the MLflow model. The path   |
|                               | is relative to the run with the      |
|                               | given run-id or local filesystem     |
|                               | path without run-id.                 |
+-------------------------------+--------------------------------------+
| ``flavor``                    | Optional flavor specification. Can   |
|                               | be used to load a particular flavor  |
|                               | in case there are multiple flavors   |
|                               | available.                           |
+-------------------------------+--------------------------------------+
| ``run_id``                    | Optional MLflow run-id. If supplied  |
|                               | model will be fetched from MLflow    |
|                               | tracking server.                     |
+-------------------------------+--------------------------------------+

.. _log-artifact-1:

Log Artifact
============

Logs a specific file or directory as an artifact for this run.

.. code:: r

   mlflow_log_artifact(path, artifact_path = NULL)

.. _arguments-25:

Arguments
---------

+-------------------+-------------------------------------------------+
| Argument          | Description                                     |
+===================+=================================================+
| ``path``          | The file or directory to log as an artifact.    |
+-------------------+-------------------------------------------------+
| ``artifact_path`` | Destination path within the run's artifact URI. |
+-------------------+-------------------------------------------------+

.. _details-20:

Details
-------

The fluent API family of functions operate with an implied MLflow client
determined by the service set by ``mlflow_set_tracking_uri()``. For
operations involving a run it adopts the current active run, or, if one
does not exist, starts one through the implied service.

When logging to Amazon S3, ensure that the user has a proper policy
attach to it.

Additionally, at least the ``AWS_ACCESS_KEY_ID`` and
``AWS_SECRET_ACCESS_KEY`` environment variables must be set to the
corresponding key and secrets provided by Amazon IAM.

.. _log-metric-1:

Log Metric
==========

Logs a metric for this run. Metrics key-value pair that records a
single float measure. During a single execution of a run, a particular
metric can be logged several times. Backend will keep track of
historical values along with timestamps.

.. code:: r

   mlflow_log_metric(key, value, timestamp = NULL)

.. _arguments-26:

Arguments
---------

+-----------------------------------+-----------------------------------+
| Argument                          | Description                       |
+===================================+===================================+
| ``key``                           | Name of the metric.               |
+-----------------------------------+-----------------------------------+
| ``value``                         | Float value for the metric being  |
|                                   | logged.                           |
+-----------------------------------+-----------------------------------+
| ``timestamp``                     | Unix timestamp in milliseconds at |
|                                   | the time metric was logged.       |
+-----------------------------------+-----------------------------------+

.. _details-21:

Details
-------

The fluent API family of functions operate with an implied MLflow client
determined by the service set by ``mlflow_set_tracking_uri()``. For
operations involving a run it adopts the current active run, or, if one
does not exist, starts one through the implied service.

Log Model
=========

Logs a model for this run. Similar to ``mlflow_save_model()`` but
stores model as an artifact within the active run.

.. code:: r

   mlflow_log_model(fn, artifact_path)

.. _arguments-27:

Arguments
---------

+-------------------------------+--------------------------------------+
| Argument                      | Description                          |
+===============================+======================================+
| ``fn``                        | The serving function that will       |
|                               | perform a prediction.                |
+-------------------------------+--------------------------------------+
| ``artifact_path``             | Destination path where this MLflow   |
|                               | compatible model will be saved.      |
+-------------------------------+--------------------------------------+

.. _log-parameter-1:

Log Parameter
=============

Logs a parameter for this run. Examples are params and
hyperparams used for ML training, or constant dates and values used in
an ETL pipeline. A params is a STRING key-value pair. For a run, a
single parameter is allowed to be logged only once.

.. code:: r

   mlflow_log_param(key, value)

.. _arguments-28:

Arguments
---------

+-----------+--------------------------------+
| Argument  | Description                    |
+===========+================================+
| ``key``   | Name of the parameter.         |
+-----------+--------------------------------+
| ``value`` | String value of the parameter. |
+-----------+--------------------------------+

.. _details-22:

Details
-------

The fluent API family of functions operate with an implied MLflow client
determined by the service set by ``mlflow_set_tracking_uri()``. For
operations involving a run it adopts the current active run, or, if one
does not exist, starts one through the implied service.

Read Command-Line Parameter
===========================

Reads a command-line parameter.

.. code:: r

   mlflow_param(name, default = NULL, type = NULL, description = NULL)

.. _arguments-29:

Arguments
---------

+-------------------------------+--------------------------------------+
| Argument                      | Description                          |
+===============================+======================================+
| ``name``                      | The name of the parameter.           |
+-------------------------------+--------------------------------------+
| ``default``                   | The default value of the             |
|                               | parameter.                           |
+-------------------------------+--------------------------------------+
| ``type``                      | Type of the parameter. Required if   |
|                               | ``default`` is not set. If           |
|                               | specified, must be one of “numeric”, |
|                               | “integer”, or “string”.              |
+-------------------------------+--------------------------------------+
| ``description``               | Optional description for the         |
|                               | parameter.                           |
+-------------------------------+--------------------------------------+

Predict over MLflow Model Flavor
================================

Performs prediction over a model loaded using ``mlflow_load_model()`` ,
to be used by package authors to extend the supported MLflow models.

.. code:: r

   mlflow_predict_flavor(model, data)

.. _arguments-30:

Arguments
---------

+-----------+----------------------------------+
| Argument  | Description                      |
+===========+==================================+
| ``model`` | The loaded MLflow model flavor.  |
+-----------+----------------------------------+
| ``data``  | A data frame to perform scoring. |
+-----------+----------------------------------+

Generate Prediction with MLflow Model
=====================================

Generates a prediction with an MLflow model.

.. code:: r

   mlflow_predict_model(model, data)

.. _arguments-31:

Arguments
---------

+-----------+-------------------------+
| Argument  | Description             |
+===========+=========================+
| ``model`` | MLflow model.           |
+-----------+-------------------------+
| ``data``  | Dataframe to be scored. |
+-----------+-------------------------+

Restore Snapshot
================

Restores a snapshot of all dependencies required to run the files in the
current directory.

.. code:: r

   mlflow_restore_snapshot()

Predict using RFunc MLflow Model
================================

Performs prediction using an RFunc MLflow model from a file or data frame.

.. code:: r

   mlflow_rfunc_predict(model_path, run_uuid = NULL, input_path = NULL,
     output_path = NULL, data = NULL, restore = FALSE)

.. _arguments-32:

Arguments
---------

+-------------------------------+--------------------------------------+
| Argument                      | Description                          |
+===============================+======================================+
| ``model_path``                | The path to the MLflow model, as a   |
|                               | string.                              |
+-------------------------------+--------------------------------------+
| ``run_uuid``                  | Run ID of run to grab the model      |
|                               | from.                                |
+-------------------------------+--------------------------------------+
| ``input_path``                | Path to JSON or CSV file to be       |
|                               | used for prediction.                 |
+-------------------------------+--------------------------------------+
| ``output_path``               | JSON or CSV file where the           |
|                               | prediction will be written to.       |
+-------------------------------+--------------------------------------+
| ``data``                      | Data frame to be scored. This can be |
|                               | used for testing purposes and        |
|                               | can only be specified when           |
|                               | ``input_path`` is not specified.     |
+-------------------------------+--------------------------------------+
| ``restore``                   | Should ``mlflow_restore_snapshot()`` |
|                               | be called before serving?            |
+-------------------------------+--------------------------------------+

.. _examples-3:

Examples
--------

.. code:: r

    list("\n", "library(mlflow)\n", "\n", "# save simple model which roundtrips data as prediction\n", "mlflow_save_model(function(df) df, \"mlflow_roundtrip\")\n", "\n", "# save data as json\n", "jsonlite::write_json(iris, \"iris.json\")\n", "\n", "# predict existing model from json data\n", "mlflow_rfunc_predict(\"mlflow_roundtrip\", \"iris.json\")\n") 
    

Serve an RFunc MLflow Model
===========================

Serves an RFunc MLflow model as a local web API.

.. code:: r

   mlflow_rfunc_serve(model_path, run_uuid = NULL, host = "127.0.0.1",
     port = 8090, daemonized = FALSE, browse = !daemonized,
     restore = FALSE)

.. _arguments-33:

Arguments
---------

+-------------------------------+--------------------------------------+
| Argument                      | Description                          |
+===============================+======================================+
| ``model_path``                | The path to the MLflow model, as a   |
|                               | string.                              |
+-------------------------------+--------------------------------------+
| ``run_uuid``                  | ID of run to grab the model from.    |
+-------------------------------+--------------------------------------+
| ``host``                      | Address to use to serve model, as a  |
|                               | string.                              |
+-------------------------------+--------------------------------------+
| ``port``                      | Port to use to serve model, as       |
|                               | numeric.                             |
+-------------------------------+--------------------------------------+
| ``daemonized``                | Makes ``httpuv`` server daemonized so|
|                               | R interactive sessions are not       |
|                               | blocked to handle requests. To       |
|                               | terminate a daemonized server, call  |
|                               | ``httpuv::stopDaemonizedServer()``   |
|                               | with the handle returned from this   |
|                               | call.                                |
+-------------------------------+--------------------------------------+
| ``browse``                    | Launch browser with serving landing  |
|                               | page?                                |
+-------------------------------+--------------------------------------+
| ``restore``                   | Should ``mlflow_restore_snapshot()`` |
|                               | be called before serving?            |
+-------------------------------+--------------------------------------+

.. _examples-4:

Examples
--------

.. code:: r

    list("\n", "library(mlflow)\n", "\n", "# save simple model with constant prediction\n", "mlflow_save_model(function(df) 1, \"mlflow_constant\")\n", "\n", "# serve an existing model over a web interface\n", "mlflow_rfunc_serve(\"mlflow_constant\")\n", "\n", "# request prediction from server\n", "httr::POST(\"http://127.0.0.1:8090/predict/\")\n") 

Run in MLflow
=============

Wrapper for ``mlflow run``.

.. code:: r

   mlflow_run(entry_point = NULL, uri = ".", version = NULL,
     param_list = NULL, experiment_id = NULL, mode = NULL,
     cluster_spec = NULL, git_username = NULL, git_password = NULL,
     no_conda = FALSE, storage_dir = NULL)

.. _arguments-34:

Arguments
---------

+-------------------------------+--------------------------------------+
| Argument                      | Description                          |
+===============================+======================================+
| ``entry_point``               | Entry point within project, defaults |
|                               | to ``main`` if not specified.        |
+-------------------------------+--------------------------------------+
| ``uri``                       | A directory containing modeling      |
|                               | scripts, defaults to the current     |
|                               | directory.                           |
+-------------------------------+--------------------------------------+
| ``version``                   | Version of the project to run, as a  |
|                               | Git commit reference for Git         |
|                               | projects.                            |
+-------------------------------+--------------------------------------+
| ``param_list``                | A list of parameters.                |
+-------------------------------+--------------------------------------+
| ``experiment_id``             | ID of the experiment under which to  |
|                               | launch the run.                      |
+-------------------------------+--------------------------------------+
| ``mode``                      | Execution mode to use for run.       |
+-------------------------------+--------------------------------------+
| ``cluster_spec``              | Path to JSON file describing the     |
|                               | cluster to use when launching a run  |
|                               | on Databricks.                       |
+-------------------------------+--------------------------------------+
| ``git_username``              | Username for HTTP(S) Git             |
|                               | authentication.                      |
+-------------------------------+--------------------------------------+
| ``git_password``              | Password for HTTP(S) Git             |
|                               | authentication.                      |
+-------------------------------+--------------------------------------+
| ``no_conda``                  | If specified, assume that MLflow is  |
|                               | running within a Conda environment   |
|                               | with the necessary dependencies for  |
|                               | the current project instead of       |
|                               | attempting to create a new Conda     |
|                               | environment. Only valid if running   |
|                               | locally.                             |
+-------------------------------+--------------------------------------+
| ``storage_dir``               | Valid only when ``mode`` is local.   |
|                               | MLflow downloads artifacts from      |
|                               | distributed URIs passed to           |
|                               | parameters of type ``path`` to       |
|                               | subdirectories of ``storage_dir``.   |
+-------------------------------+--------------------------------------+

.. _value-1:

Value
-----

The run associated with this run.

Save MLflow Keras Model Flavor
==============================

Saves model in MLflow Keras flavor.

.. code:: r

   list(list("mlflow_save_flavor"), list("keras.engine.training.Model"))(x,
     path = "model", r_dependencies = NULL, conda_env = NULL)

.. _arguments-35:

Arguments
---------

+-------------------------------+--------------------------------------+
| Argument                      | Description                          |
+===============================+======================================+
| ``x``                         | The serving function or model that   |
|                               | will perform a prediction.           |
+-------------------------------+--------------------------------------+
| ``path``                      | Destination path where this MLflow   |
|                               | compatible model will be saved.      |
+-------------------------------+--------------------------------------+
| ``r_dependencies``            | Optional vector of paths to          |
|                               | dependency files to include in the   |
|                               | model, as in ``r-dependencies.txt``  |
|                               | or ``conda.yaml`` .                  |
+-------------------------------+--------------------------------------+
| ``conda_env``                 | Path to Conda dependencies file.     |
+-------------------------------+--------------------------------------+

.. _value-2:

Value
-----

This function must return a list of flavors that conform to the MLmodel
specification.

Save MLflow Model Flavor
========================

Saves model in MLflow flavor, to be used by package authors to extend
the supported MLflow models.

.. code:: r

   mlflow_save_flavor(x, path = "model", r_dependencies = NULL,
     conda_env = NULL)

.. _arguments-36:

Arguments
---------

+-------------------------------+--------------------------------------+
| Argument                      | Description                          |
+===============================+======================================+
| ``x``                         | The serving function or model that   |
|                               | will perform a prediction.           |
+-------------------------------+--------------------------------------+
| ``path``                      | Destination path where this MLflow   |
|                               | compatible model will be saved.      |
+-------------------------------+--------------------------------------+
| ``r_dependencies``            | Optional vector of paths to          |
|                               | dependency files to include in the   |
|                               | model, as in ``r-dependencies.txt``  |
|                               | or ``conda.yaml`` .                  |
+-------------------------------+--------------------------------------+
| ``conda_env``                 | Path to Conda dependencies file.     |
+-------------------------------+--------------------------------------+

.. _value-3:

Value
-----

This function must return a list of flavors that conform to the MLmodel
specification.

Save Model for MLflow
=====================

Saves model in MLflow format that can later be used for prediction and
serving.

.. code:: r

   mlflow_save_model(x, path = "model", r_dependencies = NULL,
     conda_env = NULL)

.. _arguments-37:

Arguments
---------

+-------------------------------+--------------------------------------+
| Argument                      | Description                          |
+===============================+======================================+
| ``x``                         | The serving function or model that   |
|                               | will perform a prediction.           |
+-------------------------------+--------------------------------------+
| ``path``                      | Destination path where this MLflow   |
|                               | compatible model will be saved.      |
+-------------------------------+--------------------------------------+
| ``r_dependencies``            | Optional vector of paths to          |
|                               | dependency files to include in the   |
|                               | model, as in ``r-dependencies.txt``  |
|                               | or ``conda.yaml`` .                  |
+-------------------------------+--------------------------------------+
| ``conda_env``                 | Path to Conda dependencies file.     |
+-------------------------------+--------------------------------------+

Run MLflow Tracking Server
==========================

Wrapper for ``mlflow server``.

.. code:: r

   mlflow_server(file_store = "mlruns", default_artifact_root = NULL,
     host = "127.0.0.1", port = 5000, workers = 4,
     static_prefix = NULL)

.. _arguments-38:

Arguments
---------

+-------------------------------+--------------------------------------+
| Argument                      | Description                          |
+===============================+======================================+
| ``file_store``                | The root of the backing file store   |
|                               | for experiment and run data.         |
+-------------------------------+--------------------------------------+
| ``default_artifact_root``     | Local or S3 URI to store artifacts   |
|                               | in, for newly created experiments.   |
+-------------------------------+--------------------------------------+
| ``host``                      | The network address to listen on     |
|                               | (default: 127.0.0.1).                |
+-------------------------------+--------------------------------------+
| ``port``                      | The port to listen on (default:      |
|                               | 5000).                               |
+-------------------------------+--------------------------------------+
| ``workers``                   | Number of gunicorn worker processes  |
|                               | to handle requests (default: 4).     |
+-------------------------------+--------------------------------------+
| ``static_prefix``             | A prefix which will be prepended to  |
|                               | the path of all static paths.        |
+-------------------------------+--------------------------------------+

Set Experiment
==============

Sets an experiment as the active experiment. If the experiment does not exist,
creates an experiment with provided name.

.. code:: r

   mlflow_set_experiment(experiment_name)

.. _arguments-39:

Arguments
---------

+---------------------+-------------------------------------+
| Argument            | Description                         |
+=====================+=====================================+
| ``experiment_name`` | Name of experiment to be activated. |
+---------------------+-------------------------------------+

.. _details-23:

Details
-------

The fluent API family of functions operate with an implied MLflow client
determined by the service set by ``mlflow_set_tracking_uri()``. For
operations involving a run it adopts the current active run, or, if one
does not exist, starts one through the implied service.

.. _set-tag-1:

Set Tag
=======

Sets a tag on a run. Tags are run metadata that can be updated during and
after a run completes.

.. code:: r

   mlflow_set_tag(key, value)

.. _arguments-40:

Arguments
---------

+-------------------------------+--------------------------------------+
| Argument                      | Description                          |
+===============================+======================================+
| ``key``                       | Name of the tag. Maximum size is 255 |
|                               | bytes. This field is required.       |
+-------------------------------+--------------------------------------+
| ``value``                     | String value of the tag being        |
|                               | logged. Maximum size is 500 bytes.   |
|                               | This field is required.              |
+-------------------------------+--------------------------------------+

.. _details-24:

Details
-------

The fluent API family of functions operate with an implied MLflow client
determined by the service set by ``mlflow_set_tracking_uri()``. For
operations involving a run it adopts the current active run, or, if one
does not exist, starts one through the implied service.

Set Remote Tracking URI
=======================

Specifies the URI to the remote MLflow server that will be used to track
experiments.

.. code:: r

   mlflow_set_tracking_uri(uri)

.. _arguments-41:

Arguments
---------

+----------+--------------------------------------+
| Argument | Description                          |
+==========+======================================+
| ``uri``  | The URI to the remote MLflow server. |
+----------+--------------------------------------+

Dependencies Snapshot
=====================

Creates a snapshot of all dependencies required to run the files in the
current directory.

.. code:: r

   mlflow_snapshot()

Source a Script with MLflow Params
==================================

This function should not be used interactively. It is designed to be
called via ``Rscript`` from the terminal or through the MLflow CLI.

.. code:: r

   mlflow_source(uri)

.. _arguments-42:

Arguments
---------

+----------+----------------------------------------------------------+
| Argument | Description                                              |
+==========+==========================================================+
| ``uri``  | Path to an R script, can be a quoted or unquoted string. |
+----------+----------------------------------------------------------+

Start Run
=========

Starts a new run within an experiment, should be used within a ``with``
block.

.. code:: r

   mlflow_start_run(run_uuid = NULL, experiment_id = NULL,
     source_name = NULL, source_version = NULL, entry_point_name = NULL,
     source_type = "LOCAL")

.. _arguments-43:

Arguments
---------

+-------------------------------+--------------------------------------+
| Argument                      | Description                          |
+===============================+======================================+
| ``run_uuid``                  | If specified, get the run with the   |
|                               | specified UUID and log metrics and   |
|                               | params under that run. The run's end |
|                               | time is unset and its status is set  |
|                               | to running, but the run's other      |
|                               | attributes remain unchanged.         |
+-------------------------------+--------------------------------------+
| ``experiment_id``             | Used only when ``run_uuid`` is       |
|                               | unspecified. ID of the experiment    |
|                               | under which to create the current    |
|                               | run. If unspecified, the run is      |
|                               | created under a new experiment with  |
|                               | a randomly generated name.           |
+-------------------------------+--------------------------------------+
| ``source_name``               | Name of the source file or URI of    |
|                               | the project to be associated with    |
|                               | the run. Defaults to the current     |
|                               | file if none provided.               |
+-------------------------------+--------------------------------------+
| ``source_version``            | Optional Git commit hash to          |
|                               | associate with the run.              |
+-------------------------------+--------------------------------------+
| ``entry_point_name``          | Optional name of the entry point for |
|                               | to the current run.                  |
+-------------------------------+--------------------------------------+
| ``source_type``               | Integer enum value describing the    |
|                               | type of the run (“local”, “project”, |
|                               | etc.).                               |
+-------------------------------+--------------------------------------+

.. _details-25:

Details
-------

The fluent API family of functions operate with an implied MLflow client
determined by the service set by ``mlflow_set_tracking_uri()``. For
operations involving a run it adopts the current active run, or, if one
does not exist, starts one through the implied service.

.. _examples-5:

Examples
--------

.. code:: r

    list("\n", "with(mlflow_start_run(), {\n", "  mlflow_log(\"test\", 10)\n", "})\n") 
    

Run MLflow User Interface
=========================

Launches the MLflow user interface.

.. code:: r

   mlflow_ui(x, ...)

.. _arguments-44:

Arguments
---------

+-------------------------------+--------------------------------------+
| Argument                      | Description                          |
+===============================+======================================+
| ``x``                         | An ``mlflow_client`` object.         |
+-------------------------------+--------------------------------------+
| ``...``                       | Optional arguments passed to         |
|                               | ``mlflow_server()`` when ``x`` is a  |
|                               | path to a file store.                |
+-------------------------------+--------------------------------------+

.. _examples-6:

Examples
--------

.. code:: r

    list("\n", "library(mlflow)\n", "mlflow_install()\n", "\n", "# launch mlflow ui locally\n", "mlflow_ui()\n", "\n", "# launch mlflow ui for existing mlflow server\n", "mlflow_set_tracking_uri(\"http://tracking-server:5000\")\n", "mlflow_ui()\n") 
    

Uninstall MLflow
================

Uninstalls MLflow by removing the Conda environment.

.. code:: r

   mlflow_uninstall()

.. _examples-7:

Examples
--------

.. code:: r

    list("\n", "library(mlflow)\n", "mlflow_install()\n", "mlflow_uninstall()\n") 
    
