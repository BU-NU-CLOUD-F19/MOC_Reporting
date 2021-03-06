
Massachusetts Open Cloud (MOC) Reporting
=============

## Goal

The ultimate goal of the MOC Reporting Project is to enable the creation of 
actionable business objects detailing system utilization data across the axes of
Institution, Project, and User. Furthermore, the system must provide some 
intermediate raw format (CSV) that can be handed off to remote users for use in
their own business analytics package. 


## Stretch Goals

Although the current goal of the project revolves around generating reports and 
CSV dumps for just OpenStack, we have several stretch goals for implementation:
 1. Extend functionality of the our system to collect data from and generate CSV 
    files and reports for Zabbix, Openshift, and Ceph
 2. Develop, deploy and automate the process of consistency and quality checks 
    of the data collected
 3. Develop pricing model for billing reports for paying customers


## Users and Personas

Three Primary Groups are relevant for discussing the goals of the project. The
following definitions will be used throughout this document and repository:
 - MOC Administrators: technical personnel at the MOC who are responsible for
   MOC operations and generating MOC billing and usage reports
 - Partner Institutions: businesses and schools that are working with and using 
   MOC resources
 - Partner Administrators: Persons-of-Contact at MOC partner corporations
   and institutions who are responsible for the partner's investment in and
   participation with the MOC
 - Partner Users: researchers and engineers who use the MOC in their day-to-day
   tasks

The project will begin with MOC Administrators who need to produce usage reports
for the partner institutions. The project is anticipated to expand to other 
users, however, those users' personas are not yet well defined.


## Scope

At the highest level, the system must be able to tally the total usage for every
Virtual Machine at the MOC and produce a usage report that could be sent to any 
of the above personas for viewing. Further, the system must produce intermediary
data store in the form of CSV dumps that serve as "raw" data that can be 
provided to personas that wish to generate their own reports. Lastly, the system
must aggregate that data across three major segments:
 1. Projects
 2. Institutions
 3. Timeframes where VMs have been in use(start time and end time)

"Projects" refers to collections of MOC Service instances. Each Project defines
an area of control and will have one User that is responsible for that Project.
Further, Projects are a recursive data type: sub-projects can be defined on a
given project, and reports generated for that project must include appropriately
labeled usage data for all sub-projects. The project tree will be rooted in a
node that represents the whole of the MOC. Lastly, a notion of relative buy-in /
investment will need to be defined for all projects with multiple funding
Institutions.

The system created will automatically gather usage data from OpenStack and build
an intermediary store from which reports and dump files can be generated. All 
collected data will be persistent. The project will not address garbage 
collection of this data. 

The system will be able to produce reports accurate to one hour. The system may
be extend to provide finer reporting capabilities. Reports generated must be
consistent with the raw data collected from OpenStack. The system must run
automated consistency verification routines against all data source streams.

The system must support the following front-ends for data export:
 - CSV File, a Dump of all usage data over a given time period
 - PDF File, a Human-Readable Report

A complete billing system with graphical front-end is considered beyond the
scope of this project, however defining a model for pricing will be attempted if
time allows.

If time permits and the initial Scope of the project has been satisfied and 
completed, we can extend this project to collect data from and produce reports 
for up to three other services in the MOC: Openshift, Ceph, or Zabbix; again 
across the three major segments described above.


## Features

1. OpenStack Usage data collector
  - Data that will be collected include:
   - [User](https://docs.openstack.org/mitaka/install-guide-obs/common/glossary.html#term-user)
   - [Flavors](https://docs.openstack.org/mitaka/install-guide-obs/common/glossary.html#term-flavor)
   - [Router](https://docs.openstack.org/mitaka/install-guide-obs/common/glossary.html#term-router)
   - [Neutron Information](https://docs.openstack.org/mitaka/install-guide-obs/common/get_started_networking.html)
     - [Networks](https://docs.openstack.org/mitaka/install-guide-obs/common/glossary.html#term-network)
     - [Subnets](https://docs.openstack.org/mitaka/install-guide-obs/common/glossary.html#term-subnet)
     - [Floating IPs](https://docs.openstack.org/mitaka/install-guide-obs/common/glossary.html#term-floating-ip-address)
   - [Instances](https://docs.openstack.org/mitaka/install-guide-obs/common/glossary.html#term-instance)
   - [Volumes from Cinder](https://docs.openstack.org/mitaka/install-guide-obs/common/glossary.html#term-volume)
   - [Panko Data](https://docs.openstack.org/panko/latest/webapi/index.html)
  - Data collected will be stored in a PostgreSQL database
  - Data collection scripts will be run every 15 minutes
  - Python 3 or Perl Scripts

2. Database
  - Contains raw data from OpenStack
  - Contains tables for Institutions, Users, and Projects.
  - Database is auditable (READ actions only performed)
  - PostgreSQL RDBMS
  - The ER diagram below shows the data model structure.
  ![ER Diagram](/images/ER_diagram.png)

3. Data pipeline
  - Extracts raw OpenStack usage data from PostgreSQL database.
  - Processes raw data into CSV files
    - Each CSV file containing all entries from a user-specified time period.
    - Each CSV file is mapped to a single table within the database.
  - Pipeline will produce data consistent with MOC logs
  - Pipeline will be automated to run every day.
  - CSV Files will be stored on MOC servers and will be persistent
  - Can only perform READ actions on the database containing raw OpenStack data.

4. "Front-end" server for accessing processed data
  - Provides Interface point for user utilities for generating reports

5. CSV Database dump utility
  - Will write all entries in the usage database over a specified time period
    to downloadable files
  - Allows checking of consistency with MOC Logs

6. Basic Monthly Aggregate Usage Report Generator
  - Will extend current work that produces elementary reports

7. Runtime and access specifications
  - Our system must run on any reasonably sized physical machine or VM:
     - 1 GB memory
     - 2 CPUs
  - Our system must have access to a database for storage
  - Some limited amount of persistant storage for generating CSV dumps and 
    report files


## Solution Concept
#### Global Overview

The system can conceptually be understood has consisting of three major layers:
 1. MOC Service Provider Systems
 2. The Data Collection Engine
 3. "Front-End" Consumers

![Solution Architecture Diagram](/images/architecture_diagram.png)

#### Design Implications and Discussion

Below is a description of system components that are being used to implement the features:

 - PostgreSQL DBMS: Database
 - Perl : Data Collection 
 - Python : Building data dump utilities
    - Psycopg2 library: PostgreSQL connection management
 - R : Business Analytics and Report Generation
 - Openstack: Open source VM federation management
 - Openshift: Open source containerization software
 
Layer 1 consists of the "real services" on the MOC that are responsible for
providing the MOC's Virtualization Services. OpenStack is the keystone element
here. Layer 2 will be implemented during the course of this project. It will be
responsible for using the interfaces provided by the services in Layer 1 to 
collect, aggregate and store data and provide an API to the Layer 3 services. 
Layer 2 will also provide functionality to dump data store into an intermediary 
raw format in the form of CSV dumps. Proof-of-Concept demonstration applications
at Layer 3 will be developed to showcase the ability of the Layer 2 aggregation 
system. The finished system will be deployed on MOC VMs. 


## Minimum Acceptance Criteria
 1. The system must be able to both generate a human readable report,
    summarizing OpenStack usage and dump across Institutions, Projects, and 
    Timeframe.
 2. The system creates intermediate CSV files that represent the state of the
    database tables from a current period of time and be stored on MOC servers.
 3. ~~Openstack data collection, storing into databases, and saving as CSV~~
    ~~files will be automated.~~
    Note 12/4: Team decided with advisor during the project that generation of 
    any files by our system would be completed on-demand
 

## Reach Acceptance Criteria 
 1. The system must be able to both generate a human readable report
    summarizing Openshift usage and dump across Institutions, Projects, and Users.
 2. The system must be able to both generate a human readable report
    summarizing Ceph usage and dump across Institutions, Projects, and Users.
 3. The system must be able to both generate a human readable report
    summarizing Zabbix usage and dump across Institutions, Projects, and Users.
 4. The system will have automated data quality and consistency checks.
 5. Openshift, Ceph and Zabbix usage data collection, storing into databases, 
    and saving as CSV files will be automated.


## Release Planning
#### Sprint #1: (Sept 26)
 - Analysis of requirements and setting up the PostgreSQL database on the test 
   environment along with other dependencies.
 - Second draft of Project Proposal after further investigation. Constructed 
   entity relationship diagrams, cardinality relationship diagrams, and solution
   architecture diagrams.
 - Create Script to extract data from database and convert it into CSV files so 
   that it could be further processed and customized reports could be generated from it.
 - Get test data from mentor and generate csv files for each tables.

#### Sprint #2: (Oct 10)
 - Implement data filteration for csv dump based on timeframe(start_date and 
  end_date where VMs where active).
 - Create setup script that would install all the dependencies, schedule the csv
   generating script and setup the report generation tool in Linux machine.
 - Draft R reporting script to generate usage reports from csv files.
 - Database remodelling.
 - Add functionality to CSV script to pull data from another VM hosting the 
   database.

#### Sprint #3: (Oct 24)
 - Finalize R report generation script for OpenStack usage data.
 - Implement data filteration based on a project ID and timeframe before dumping
   to CSV files.
 - Implement data filteration based on an institution ID and timeframe before 
   dumping to CSV files.
 - Create Summary tables in database to sumamrize the VM usage based on daily, 
   weekly, monthly basis.
 - Update database config ini files to json format.

#### Sprint #4: (Nov 7)
 - Remodel summary table attributes.
 - Write a data schema migration script to transfer data from old data model to 
   fit into new one.
 - Instrument Zabbix data collection
 - Transition all SQL queries to follow new schema model.
 - Make a REST API design document about the csv_dump_api service the client 
   uses to curl the csv dump packages.  

#### Sprint #5: (Nov 21)
 - Implement the REST API to curl to a server endpoint and dump the filtered 
   Openstack usage data to CSV files.
 - Project Retrospective to evaluate the project success and team process.
 - Preparation for Paper Presentation on Jupiter Rising.


## Open Questions
 - ~~What is the minimum necessary level of usage data granularity?~~
    - ~~What minimum level of granularity is needed internally to provide~~
      ~~this, if different?~~
 - ~~How often will the pipeline need to run?~~
    - N/A; will be run on-demand
 - ~~Guidelines/Expectations for outputted report?~~
    - Of minimal consequnce at this time
 - ~~What are the hardware specs of the machines to be running this system?~~
    - Very Minmal; see above
 - ~~Containerization: What software is necessary in containers to run scripts?~~
    - Working with advisor, we decided contanerization was of little value at 
      present
 - ~~Definition of Production level~~
    - To be addressed in future work


 ## Presentations + Videos
 - [Sprint 1](https://docs.google.com/presentation/d/12604SFjpRfqNQNBQ3ePZvH79kq0-ihbKXSHs0ffKcXQ/)
 - [Sprint 2](https://docs.google.com/presentation/d/1dm3UGdc3P_4b5UifQM6C4nB_20OWXBRQcUdXFZ0owNg/)
 - [Sprint 3](https://docs.google.com/presentation/d/1NIUT4NMnndiZX0jPS-VODNg4AAYO_2hbqc_hw5cD_F4/)
 - [Sprint 4](https://docs.google.com/presentation/d/15ipVvw_tJ8az4gGcC-GOwnAKGxc-IqUGe2auCgMxTfQ/)
 - [Sprint 5](https://docs.google.com/presentation/d/1UItGtiM944jlSvJnVHLxWiUEWsnEuvF1dAEYKvO9zDY/)
 - [Jupiter Rising](https://docs.google.com/presentation/d/1ostKRN2FZnqzaJ-8uUCbqlU3tE2DqcTxGqawXl3JtXk/)
 - [Final](https://youtu.be/FklN4jVnSqo)

