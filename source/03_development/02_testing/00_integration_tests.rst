.. role:: raw-html-m2r(raw)
   :format: html
.. # define a hard line break for HTML
.. |br| raw:: html

   <br />


EHRbase Integration Tests with Robot Framework :raw-html-m2r:`<!-- omit in toc -->`
=======================================================================================


* `Prerequisites <#prerequisites>`_
* `Test Environment & SUT <#test-environment--sut>`_
* `Test Execution (under Linux, Mac & Windows) <#test-execution-under-linux-mac--windows>`_

  * `With Robot Command <#with-robot-command>`_
  * `With Shell Script <#with-shell-script>`_
  * `Example Content Of Shell Script (run_local_tests.sh) <#example-content-of-shell-script-runlocaltestssh>`_

* `Local SUT / Manually Controlled SUT <#local-sut--manually-controlled-sut>`_

  * `Usage examples: <#usage-examples>`_

* `Remote SUT / OR how to execute the tests against other systems <#remote-sut--or-how-to-execute-the-tests-against-other-systems>`_

  * `Preconditions <#preconditions>`_
  * `Customize your configuration <#customize-your-configuration>`_
  * `Execute test against EHRSCAPE <#execute-test-against-ehrscape>`_

    * `Run all tests at one (\ **not** recommended) <#run-all-tests-at-one-not-recommended>`_
    * `Run single test suites, one by one (recommended) <#run-single-test-suites-one-by-one-recommended>`_

* `Execution Control - Test Suites & Tags <#execution-control---test-suites--tags>`_
* `CI/CD Pipeline (on CircleCI) <#cicd-pipeline-on-circleci>`_
* `Errors And Warnings <#errors-and-warnings>`_
* `Auto-Generated Test Report Summary And Detailed Log <#auto-generated-test-report-summary-and-detailed-log>`_

----

Prerequisites
-------------

1) Docker, Java 11 & Maven, Python 3.7+ & Pip are installed
2) Robot Framework & dependencies are installed (\ ``pip install -r requirements.txt``\ )
3) Build artefacts created (\ ``mvn package`` --> application/target/application-x.xx.x.jar) |br|
4) ⚠️ **No DB / no server running!** |br|
5) ⚠️ ports ``8080`` and ``5432`` not used by any other application! (check it w/ ``netstat -tulpn``\ )

Test Environment & SUT
----------------------

The test environment of this project consists of three main parts |br|
|br|
1) EHRbase OpenEHR server (application-*.jar) |br|
2) PostgreSQL database |br|
3) OS with Docker, Java runtime, Python runtime, Robot Framework (generic test automation framework) |br|

Let's refer to the first two parts as the SUT (system under test). The tests are implemented in a way that by default Robot Framework (RF) will take control of the SUT. That means to execute the tests locally all you have to do is to ensure your host machine meets required `prerequisites <#prerequisites>`_. RF will take care of properly starting up, restarting and shutting down SUT as it is required for test execution. There is an option to hand over control of SUT to you, though - described in section `Manually Controlled SUT <#manually-controlled-sut>`_.

Test Execution (under Linux, Mac & Windows)
-------------------------------------------

In general tests are executed by 1) cd into tests/ folder and 2) call the **\ ``robot``\ ** command with the folder wich contains the test suites as argument. Alternatively you can use prepared shell script: **run_local_tests.sh**.

With Robot Command
~~~~~~~~~~~~~~~~~~

The following examples will run all test-cases that are inside ``robot/`` folder

.. code-block:: bash

   # 1) from project's root
   cd tests/

   # 2) call robot command
   robot robot/     # Linux
   robot ./robot/   # Mac OS
   robot .\robot  # Windoofs

Everything between ``robot`` command and the last argument are commandline option to fine control test execution and the processing of test results. Examples:

.. code-block:: bash

   # QUICK COPY/PASTE EXAMPLES TO RUN ONLY A SPECIFIC TEST-SUITE

   robot -i composition     -d results --noncritical not-ready -L TRACE robot/COMPOSITION_TESTS/
   robot -i contribution    -d results --noncritical not-ready -L TRACE robot/CONTRIBUTION_TESTS/
   robot -i directory       -d results --noncritical not-ready -L TRACE robot/DIRECTORY_TESTS/
   robot -i ehr_service     -d results --noncritical not-ready -L TRACE robot/EHR_SERVICE_TESTS/
   robot -i ehr_status      -d results --noncritical not-ready -L TRACE robot/EHR_STATUS_TESTS/
   robot -i knowledge       -d results --noncritical not-ready -L TRACE robot/KNOWLEDGE_TESTS/
   robot -i aqlANDempty_db  -d results --noncritical not-ready -L TRACE robot/QUERY_SERVICE_TESTS/
   robot -i aqlANDloaded_db -d results --noncritical not-ready -L TRACE robot/QUERY_SERVICE_TESTS/

With Shell Script
~~~~~~~~~~~~~~~~~

Use `shell script <./run_local_tests.sh>`_ to run **all** available tests at once or use it as a reference to see which `command line options <http://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#using-command-line-options>`_ are available to the ``robot`` command. Examples below demonstrate it's usage: 

.. code-block:: bash

   # Linux
   . run_local_tests.sh

   # Mac OS
   ./run_local_tests.hs

   # Windows
   robot -d results --noncritical not-ready -L TRACE robot/

   (No script there yet. TODO: create a proper .bat file)

Example Content Of Shell Script (run_local_tests.sh)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

   robot --include contribution \
         --exclude TODO -e future -e obsolete -e libtest \
         --loglevel TRACE \
         --noncritical not-ready \
         --flattenkeywords for \
         --flattenkeywords foritem \
         --flattenkeywords name:_resources.* \
         --outputdir results \
         --name CONTRIBUTION \
         robot/CONTRIBUTION_TESTS/

Local SUT / Manually Controlled SUT
-----------------------------------

In case you don't want Robot to start up and shut down server and database for you - i.e. during local development iterations - there is a command line option (\ ``-v nodocker``\ ) to turn this off. **This option should be used with some precaution, though!**

..

   ⚠️ 

   Test Suite Setups and Teardowns will NOT be orchestrated by Robot any more. This can lead to issues when trying to run ALL tests at once (i.e. with ``robot robot/``\ ) - tests may impact each other and fail. Thus you will have to pass at least a test suite folder as argument or limit test selection by using tags to avoid this (see section below). Moreover


   * you have to start the server with cache DISABLED (\ ``--cache.enabled=false``\ )
   * you have to ensure your server configuration applies to Robot's DEV configuration (see tests/robot/_resources/suite_settings.robot)
   * you have to ensure your DB configuration applies to the one described in main README
   * you have to restart server and rollback/reset database properly
   * when in doubt about your results, compare them with results in CI pipeline
   * YOU HAVE BEEN WARNED!

   ⚠️


**Usage Examples:**

.. code-block:: bash

   robot --variable nodocker:true robot/TEST_SUITE_FOLDER

   # short variant
   robot -v nodocker robot/TEST_SUITE_FOLDER
   robot -v nodocker -i get_ehr robot/EHR_SERVICE_TESTS

Robot will print proper warning in console if it can't connect to server or database:

.. code-block:: bash

   [ WARN ] //////////////////////////////////////////////////////////
   [ WARN ] //                                                     ///
   [ WARN ] // YOU HAVE CHOSEN TO START YOUR OWN TEST ENVIRONMENT! ///
   [ WARN ] // BUT IT IS NOT AVAILABLE OR IS NOT SET UP PROPERLY!  ///
   [ WARN ] //                                                     ///
   [ WARN ] //////////////////////////////////////////////////////////
   [ WARN ] 
   [ WARN ] [ check "Manually Controlled SUT" in Integration Test Documentation ]
   [ WARN ] [ https://docs.ehrbase.org/en/latest/03_development/02_testing/index.html#local-sut--manually-controlled-sut ]
   [ WARN ] 
   [ ERROR ] ABORTING EXECUTION DUE TO TEST ENVIRONMENT ISSUES:
   [ ERROR ] Could not connect to server!
   [ ERROR ] Could not connect to database!

Remote SUT / OR how to execute the tests against other systems
--------------------------------------------------------------

All integration tests in this repository can be executed against other (possiblty remotely accessible) OpenEHR conform systems (other than EHRbase). Here we will demonstrate how to run the test against your own remote system. We'll use EHRSCAPE as an example configuration. If you don't have access to EHRSCAPE you'll have to adjust related parts to your needs.

**Preconditions**

1) the following environment variables have to be available:

.. code-block:: bash

     BASIC_AUTH (basic auth string for EHRSCAPE, i.e.: export BASIC_AUTH="Basic abc...")
     EHRSCAPE_USER
     EHRSCAPE_PASSWORD

2) Python 3.7+ installed
3) Test dependencies installed

.. code-block:: bash

   cd tests
   pip install -r requirements.txt

**Customize your configuration**

Open **tests/robot/_resources/suite_settings.robot** and adjust the following part to your needs if you don't have access to EHRSCAPE. If you do any changes here, remember to adjust your environment variables in step 1)

.. code-block:: bash

   &{EHRSCAPE}             URL=https://rest.ehrscape.com/rest/openehr/v1
   ...                     HEARTBEAT=https://rest.ehrscape.com/
   ...                     CREDENTIALS=@{scapecreds}
   ...                     AUTH={"Authorization": "%{BASIC_AUTH}"}
   ...                     NODENAME=piri.ehrscape.com
   ...                     CONTROL=NONE
   @{scapecreds}           %{EHRSCAPE_USER}    %{EHRSCAPE_PASSWORD}

**Execute test against EHRSCAPE**

The only difference in contrast to normal execution is that you now want to specify that EHRSCAPE configuration from suite_settings.robot should be used. This is done by setting ``SUT`` variable to ``EHRSCAPE`` which you can achieve by passing ``-v SUT:EHRSCAPE`` when calling ``robot``. Check examples below.

Run all tests at one (\ **not** recommended)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This is not recommend because it may take from 30 to 60 minutes and makes it harder to analyse the results.

.. code-block:: bash

   robot -v SUT:EHRSCAPE -e future -e circleci -e TODO -e obsolete -e libtest -d results -L TRACE --noncritical not-ready robot/

Run single test suites, one by one (recommended)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Execute the test suite that you are interested in by copy&pasting one of the lines below, then analyse the results of that test suite.

..

   Best practice is also to reset your system under test to a clear state before executing the next test suite.

   .. code-block:: bash

      robot -v SUT:EHRSCAPE -d results/composition -e future -e circleci -e TODO -e obsolete -e libtest -L TRACE --noncritical not-ready --name COMPO robot/COMPOSITION_TESTS
      robot -v SUT:EHRSCAPE -d results/contribution -e future -e circleci -e TODO -e obsolete -e libtest -L TRACE --noncritical not-ready --name CONTRI robot/CONTRIBUTION_TESTS
      robot -v SUT:EHRSCAPE -d results/directory -e future -e circleci -e TODO -e obsolete -e libtest -L TRACE --noncritical not-ready --name FOLDER robot/DIRECTORY_TESTS
      robot -v SUT:EHRSCAPE -d results/ehr_service -e future -e circleci -e TODO -e obsolete -e libtest -L TRACE --noncritical not-ready --name EHRSERVICE robot/EHR_SERVICE_TESTS
      robot -v SUT:EHRSCAPE -d results/ehr_status -e future -e circleci -e TODO -e obsolete -e libtest -L TRACE --noncritical not-ready --name EHRSTATUS robot/EHR_STATUS_TESTS
      robot -v SUT:EHRSCAPE -d results/knowledge -e future -e circleci -e TODO -e obsolete -e libtest -L TRACE --noncritical not-ready --name KNOWLEDGE robot/KNOWLEDGE_TESTS
      robot -v SUT:EHRSCAPE -d results/aql_1 -e future -e circleci -e TODO -e obsolete -e libtest -L TRACE --noncritical not-ready --name "QUERY empty_db"  -i empty_db  robot/QUERY_SERVICE_TESTS
      robot -v SUT:EHRSCAPE -d results/aql_2 -e future -e circleci -e TODO -e obsolete -e libtest -L TRACE --noncritical not-ready --name "QUERY SMOKE"  -i SMOKE  robot/QUERY_SERVICE_TESTS
      robot -v SUT:EHRSCAPE -d results/aql_3 -e future -e circleci -e TODO -e obsolete -e libtest -L TRACE --noncritical not-ready --name "QUERY loaded_db" -i loaded_db robot/QUERY_SERVICE_TESTS


Execution Control - Test Suites & Tags
--------------------------------------

Execution of **all** integration tests takes **about 30 minutes** (on a fast dev machine). To avoid waiting for all results you can specify exactly which test-suite or even which subset of it you want to execute. There are seven test-suites to choose from by passing proper TAG to ``robot`` command via the ``--include`` (or short ``-i``\ ) option: 

.. list-table::
   :header-rows: 1

   * - TEST SUITE
     - SUPER TAG
     - SUB TAG(s)
     - EXAMPLE(s)
   * - COMPOSITION_TESTS
     - composition
     - json, json1, json2, :raw-html-m2r:`<br>` xml, xml1, xml2
     - ``robot --include composition`` :raw-html-m2r:`<br>` ``robot -i composition`` :raw-html-m2r:`<br>` ``robot -i compositionANDjson``
   * - CONTRIBUTION_TESTS
     - contribution
     - contribution_commit, :raw-html-m2r:`<br>` contributions_list, :raw-html-m2r:`<br>` contribution_has, :raw-html-m2r:`<br>` contribution_get
     - ``robot -i contribution``
   * - DIRECTORY_TESTS
     - directory
     - directory_create, :raw-html-m2r:`<br>` directory_update, :raw-html-m2r:`<br>` directory_get, :raw-html-m2r:`<br>` directory_delete, :raw-html-m2r:`<br>` directory\ *get*\ @time, :raw-html-m2r:`<br>` ...
     - ``robot -i directory`` :raw-html-m2r:`<br>` ``robot -i directory_createORdirectory_update``
   * - EHR_SERVICE_TESTS
     - ehr_service
     - create_ehr, update_ehr, :raw-html-m2r:`<br>` has_ehr, get_ehr, :raw-html-m2r:`<br>`  ehr_status
     - ``robot -i ehr_service``
   * - EHR_STATUS_TESTS
     - ehr_status
     - ehr_status_get, :raw-html-m2r:`<br>` ehr_status_set, :raw-html-m2r:`<br>`  ehr_status_set_queryable, :raw-html-m2r:`<br>` ehr_status_clear_queryable, :raw-html-m2r:`<br>` ehr_status_set_modifiable, :raw-html-m2r:`<br>` ehr_status_clear_modifiable
     - ``robot -i ehr_status``
   * - KNOWLEDGE_TESTS
     - knowledge
     - opt14
     - ``robot -i knowledge``
   * - QUERY_SERVICE_TESTS
     - aql
     - aql_adhoc-query, :raw-html-m2r:`<br>` aql_stored-query, :raw-html-m2r:`<br>` aql_register-query, :raw-html-m2r:`<br>` aql_list-query
     - ``robot -i adhoc-query``


The **SUPER TAG** is meant to reference *all* tests from related test-suite. The **SUB TAGs** can be used (in combination with a SUPER TAG) to further narrow down which tests to include into execution. As you can see from the examples in the table above it is possible to combine TAGs with ``AND`` and ``OR`` operators. Tags themself are case insensitive but **the operators have to be upper case**. In addition to ``--include`` or ``-i`` option there is also an ``--exclude`` / ``-e`` option. It is possible to combine ``-i`` and ``-e`` in one call. Example below would include all test from EHR_SERVICE_TESTS folder which have the ``ehr_service`` **and** ``get_ehr`` tags and would irgnore all test which have the ``future`` tag.

.. code-block:: bash

   robot -i ehr_serviceANDget_ehr -e future robot/EHR_SERVICE_TESTS/

`Using TAGs to include/exclude tests <http://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#by-tag-names>`_ from execution is very well documented in `Robot Framework's User Guide <http://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html>`_.

CI/CD Pipeline (on CircleCI)
----------------------------

Check out .circleci/config.yml in project root for an CircleCI example pipline whiche runs Robot test suites in parallel.

Errors And Warnings
-------------------

⚠️ You will see ``[WARN]`` and ``[ERROR]`` in console output and in log.html

``[ERROR]`` --> take a closer look, probably important

``[WARN]``  --> minor issues like wrong status code or keyword deprecation warning.

..

   NOTE: ``[WARN] Response body content is not JSON. Content-Type is: text/html``

   You will see this warning very often. IGNORE it! It's caused by a RF library.


Auto-Generated Test Report Summary And Detailed Log
---------------------------------------------------

After each test run Robot creates a report.html (summary) and a log.html
(details) in results folder. The files are overwritten after each run by default.
If you want to keep history of your test runs you can `time-stamp <http://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#timestamping-output-files>`_ the output files.
