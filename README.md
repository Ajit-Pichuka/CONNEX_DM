## Overview

This repository is tied to the specific database and it's environment variants which are represented in branches: **dev**, **qa**, **prod**. <br>
The primary tool behind is [Schemachange](https://github.com/Snowflake-Labs/schemachange), with some custom features developed by us.
. 
## Table of Contents
1. [Overview](#overview)
1. [Branching Strategy](#branching-strategy)
1. [Feature Branches](#feature-branches)
    * [Branch Creation](#branch-creation)
    * [Schema Cloning](#schema-cloning)
    * [Deployment to Clone](#deployment-to-clone)
    * [Promotion to Dev](#promotion-to-dev)
1. [Promotion to higher environments](#promotion-to-higher-environments)
1. [Cherry Picking (Forced deployements)](#cherry-picking)
1. [Jinja support](#jinja-support)

## Branching strategy
The branching strategy is somewhat a mix between [Github Flow](https://gitversion.net/docs/learn/branching-strategies/githubflow/) and [Gitflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow) <br>
The flow is the following: **feature** > **dev** > **qa** > **prod** with the exception of [Cherry Pick](#cherry-picking). <br>
The updates to branches are done via **pull requests**.

## Feature Branches / Schemas

https://github.com/danone/onesource.cicd-sf-template/assets/45002662/9d3698b5-f129-4dfc-8c72-5bedbacfb172


#### Branch Creation
The naming for a feature branch is important. The format is the following:
```
feature/{UID}/{free-text}
```
**UID:** is the unique identifier, which will be suffixed to your cloned schemas. ([Schema Cloning](#schema-cloning))

#### Schema Cloning

You can clone schemas by running **Clone/Sync schemas** workflow manually.

#### Deployment to Clone

https://github.com/danone/onesource.cicd-sf-template/assets/45002662/44d6138c-2a6e-47ee-9f9f-ba8203965937


Make sure to suffix **uid** to the schema names inside DDL scripts.
```
USE SCHEMA SOMESCHEMA{{uid}};
CREATE TABLE TBL1
(
    FIRST_NAME VARCHAR,
    LAST_NAME VARCHAR
);

CREATE TABLE SOMESCHEMA{{uid}}.TBL2
(
    FIRST_NAME VARCHAR,
    LAST_NAME VARCHAR
);

```
#### Promotion to Dev

https://github.com/danone/onesource.cicd-sf-template/assets/45002662/7e82cdbd-a5e3-43ab-a927-c1f2d4710260

* The promotion is done via **pull requests**.
* After the promotion, you can either drop cloned schemas using **Drop Clones** workflow, or you can keep working on them, just make sure to synchronize with the latest changes from DEV by using **CLONE/SYNC** workflow.
## Promotion to higher environments

Promotion to QA and PROD can be done either via pull request or [Cherry Pick](#cherry-picking).
## Cherry-picking

https://github.com/danone/onesource.cicd-sf-template/assets/45002662/1e73c6be-6615-4c90-b446-ebc217016f76

There are cases when you don't want/can't deploy all changes to the target ENV. For those situations we've the "Cherry-Pick" workflow. This workflow expects a configuration file, where you specify DDL scripts you wan't to promote. You can create as many config files as you like, just make sure to specify the right one, before you run the workflow. <br>
*With this approach you **force** the DDL script deployment and execution, even if the **maximum applied version** in the target env is higher.* <br>
**WARNING** The "Cherry-picking" approach, doesn't promote commits, therefore in the future, when you'll want to promote changes via regular merge, GIT will still think that the file, which was "cherry-picked" before, doesn't exists ( but it does exist ).

## Rollback

Rollback via automated pipeline is only possible if rollback scripts are provided by development team. <br>
Example:  <br>
DEV has following scripts avaialble
* change_scripts/v1.1__create_table_A.sql
* change_scripts/v1.2__create_table_B.sql
* change_scripts/v1.3__create_table_C.sql


In order to be able to rollback in DEV, following scripts need to be developed
(naming convention: rb_scripts/${version_to_rollback}/A__description.sql)

* rb_scripts/1.1/A__Rollback_dropTableA.sql
* rb_scripts/1.2/A__Rollback_dropTableB.sql
* rb_scripts/1.3/A__Rollback_dropTableC.sql

Lets imagine versions 1.1, 1.2 and 1.3 are deployed to both DEV and QA. 
* Now Maximum versions are: DEV 1.3 and QA also 1.3
* If we run Rollback pipeline selecting QA branch
  * process will execute the QA rollback process for last QA version (1.3)
  * Script executed: rb_scripts/1.3/A__Rollback_dropTableC.sql (from QA branch) into snowflake QA env
  * Now Maximum versions are: DEV 1.3 and QA 1.2

* If we run Rollback pipeline selecting QA branch
  * process will execute the QA rollback process for last QA version (1.2)
  * Script executed: rb_scripts/1.2/A__Rollback_dropTableB.sql (from QA branch) into snowflake QA env
  * Now Maximum versions are: DEV 1.3 and QA 1.1


* If we run Rollback pipeline selecting DEV branch
  * process will execute the DEV rollback process for last DEV version (1.3)
  * Script executed: rb_scripts/1.3/A__Rollback_dropTableC.sql (from DEV branch) into snowflake DEV env
  * NNow Maximum versions are: DEV 1.2 and QA 1.1



## Jinja support
Inside **Modules** directory, you can create Jinja macros, templates and import them inside your scripts for advanced cases. Example: https://github.com/Snowflake-Labs/schemachange/tree/master/demo/citibike_jinja


## Schema Change Naming conventions

| Entity | Convention | Purpose |
| ------- | ------ | ----- |
| Dev Branch | dev | represents the development database and the schemas associated to this UC |
| QA Branch | qa | represents the QA database and the schemas associated to this UC |
| PROD Branch | prod | represents the PROD database and the schemas associated to this UC |
| Feature Branch | **feature/{uid}/{action}** <br><br> feature/basile/importantWork <br> | Represents personal developer environment in the form of cloned schemas. |
| Versioned scripts | **V{version}__{object_type}.{schema}.{object_name}.{action}.sql** <br><br>V1.0.0__table.schema_A.table_1.create_table.sql<br>V1.0.1__table.schema_A.table_1.add_columns.sql<br>V1.0.2__vw.schema_B.view_1.create_view.sql | Regular versioned scripts, latest applied version is tracked in Snowflake and same or lower versions won't be applied again. |
| Repeatable scripts | **R__{object_type}.{action}.sql** <br><br> R__fn.get_sales.sql <br> R__vw.schema1_table2.sql <br> R__sp.get_sales.sql| Repeatable scripts could be used for maintaining code that always needs to be applied in its entirety. e.g. stores procedures, functions and view definitions etc. All repeatable scripts are applied each time the utility is run, if there is a change in the file. |
| Always scripts | **A__{action}.sql** <br><br>A__load_data.sql |  Always change scripts are executed with every run of schemachange. They always apply last | 

# Transport Authorization and Transport Validation Strategy

Check authorization and validation strategy on ppt publised on [confluence](https://confluence-danone.atlassian.net/wiki/spaces/ADM/pages/67964675/CICD+User+Documentation). (Section "Transport Authorization/Validation")

Functional Authorizations


https://github.com/danone/onesource.cicd-sf-template/assets/45002662/711b9747-15d0-4d24-9064-a42e7086c8c3



Functional Job Validations


https://github.com/danone/onesource.cicd-sf-template/assets/45002662/4745458b-3e8e-4ff7-a729-3851c05bc8bd

