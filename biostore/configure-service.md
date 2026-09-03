---
uid: configure-service-biostore
title: Configure Alcazar Biostore
---

# Biostore Configuration

## 1.1 Overview

This document provides the step-by-step instructions required to configure the **Alcazar Biostore** service and its supporting components 
after installation. It is intended for administrators preparing a newly deployed Biostore environment for first use, 
including configuring biometric classes and procs, process profiles, background tasks, accelerator maps, workflow integration, and optional single sign-on.

While the deployment guides describe how to install Biostore on Linux or Windows, this document focuses on what happens **after** 
installation, the configuration sequence that prepares the core service, background processing, and accelerator infrastructure for operational use.

The procedures described here are **Biostore-specific**, providing the required steps for initialising the environment and 
enabling biometric processing, identification capabilities, and workflow activities.

This guide describes the standard configuration sequence suitable for development, testing, and small-scale production environments.

---

## 1.2 Prerequisites

Before configuring **Biostore**, several components must be installed and prepared to ensure the system can operate correctly. 
This includes deploying Biostore on your chosen operating system, preparing the database environment, and completing the initial setup steps.

For detailed instructions, refer to the following guides:

* **Linux installations:** @deploy-linux-biostore
* **Windows installations:** @deploy-windows-biostore
* **First-time configuration:** @configure-first-time

**Note:** When using PostgreSQL, ensure that **PostgreSQL** is selected as the database provider during setup.

---

## 1.3 Upgrade Database Schema

### Upgrade to R3.1.2

To upgrade the Biostore database schema from a previous version to **R3.1.2**, first verify that the last required script for your current version has already been applied. Once confirmed, run all new migration scripts listed below for your database provider.

|                 | MS-SQL                      | PostgreSQL                  |
| --------------- | --------------------------- | --------------------------- |
| Previous script | `Upgrade-Biostore-V3.1.002` | `Upgrade-Biostore-V3.0.012` |
| New scripts     | `Upgrade-Biostore-V3.1.003` | `Upgrade-Biostore-V3.1.001` |

---

## 1.4 Scope

By following this guide, you will be able to:

* Configure and enable the required **biometric classes, procs, and strategies** used by Biostore.
* Define **process defaults** and **enrolment profiles** to control biometric processing behaviour.
* Perform the **conversion of biometric data from Biostore 2 to Biostore 3**, including bulk extraction and continuous extraction.
* Configure ongoing **synchronisation and validation** of biometric samples and features.
* Build and maintain the **BIO-key VST accelerator databases** used for high-performance identification searches.
* Configure essential **background tasks** that support extraction, validation, re-evaluation, and accelerator maintenance.
* Integrate and configure the **workflow service** for background processing and workflow activities.
* Optionally configure **single sign-on (SAML)** and **OAuth authorisation** for secure access and identity management.

---

## 1.5 Components

A complete Biostore installation, with workflow support and single sign-on, consists of these components:

![config-service-overview1](images/config-service-overview1.png)

-   The Biostore service accepts biometric requests from client applications, such as enrolments, verification requests, and identification searches.

-   The (optional) Biostore agent serves as accelerator for identification searches.

-   The Biostore website offers functionality for configuration, administration, and reporting.

-   The (optional) workflow service executes workflow activities and background tasks, supporting Biostore functionality. Background tasks could also be executed by the Biostore service, however the workflow service offers more granular administration and management functionality.

-   The (optional) workflow website administers workflow activities and background tasks.

-   The (optional) Single IDP provides OAuth authorisation for the Biostore service, and SAML authentication for all participating websites. Any OAuth authorisation server and SAML identity provider can be used in its place.

-   The (optional) Single website administers, configures, and reports on single sign-on activities using SAML and OAuth.

-   The (optional) Single SP (not shown above) can serve as test client for single sign-on activities.

The Biostore transact module allows all biometric transactions, such as enrolments, verification requests, and identification searches, to be initiated directly from the Biostore website. The transact functionality can be used to perform biometric transactions manually, or to demonstrate the capability of a fully featured biometric solution.

Where the Biostore transact capability is used, The BioClient service provides local, workstation-bound access to sensors (fingerprint readers or cameras) and engines (software modules) to extract biometric features, and to perform local verifications. BioClient also provides access to biometric stores, such as the Biostore service.


![config-service-overview2](images/config-service-overview2.png)

---

# 2. Configuration

## 2.1 Biostore configuration

### 2.1.1 Biometric classes

By default, Biostore configures selected sample and feature classes.

Select *Configuration > Biostore > Biometric classes*. Add the following classes to the default feature classes:

-   Bio-key VST 6.6 (indexed, for enrol) -- class ID 723

-   Bio-key VST 6.6 (indexed, for lookup) -- class ID 724

![config-service-biometric-classes1](images/config-service-biometric-classes1.png)

### 2.1.2 Biometric procs

By default, Biostore configures selected sample and feature procs. Procs are processing atoms, which perform feature extraction, sample or feature conversions, and feature comparisons.

Select *Configuration > Biostore > Biometric procs*. Add the following classes to the default biometric procs, with the properties shown below:

-   BiokeyVST66Extractor

-   BiokeyVST66IndexedExtractor

-   BiokeyVST66Verifyer

![config-service-biometric-procs1](images/config-service-biometric-procs1.png)
  
### 2.1.3 Biometric strategies

After amending biometric classes or procs, select *Configuration > Biostore > Biometric strategies* to review and commit the changes.

Click on Commit to commit biometric strategies, until the system confirms that biometric strategies are up to date.

![config-service-biometric-strategies1](images/config-service-biometric-strategies1.png)

## 2.2 Process configuration

### 2.2.1 Process defaults

Select *Configuration > Processes > Process default* to set processing settings as default for all biometric operations, where process profiles to not override these settings.

Select audit settings as appropriate for your requirements. In many cases, you can accept the default settings.

![config-service-process-default1](images/config-service-process-default1.png)

Select security settings as appropriate for your requirements. In many cases, you can accept the default settings. Note that the *authentication method* options are obsolete, and will be replaced in due course.

![config-service-process-default2](images/config-service-process-default2.png)

Select general settings as appropriate for your requirements. In many cases, you can accept the default settings.

![config-service-process-default3](images/config-service-process-default3.png)

When done, select *Update* to save any changed settings.

### 2.2.2 Enrolment settings

Select *Configuration > Processes > Enrolment* to view, add, or change enrolment profiles.

![config-service-enrolment-setting1](images/config-service-enrolment-setting1.png)

Click on a profile to set processing settings for the enrolment profile, where differ from the process default settings.

![config-service-enrolment-setting2](images/config-service-enrolment-setting2.png)

Modify settings as required, and when done, select *Update* to save any changed settings.

---

# 3. Conversion to Biostore 3

The PROD conversion from Biostore 2 production to the Biostore 3 identification system is handled differently from the process employed for the POC.

The production migration uses these steps:

-   Step 1: Persons and properties are copied from Biostore 2 to 3, using SQL scripts. Primary keys are retained so that subsequent steps can reference persons between the databases.

-   Step 2: Person segments are set per person, using SQL scripts. These scripts are client-specific and are to be developed by the client. However, the following - `step 3` - allows for Biostore 3 to obtain the initial segment to be set from the most recent enrolment of that person.

-   Step 3: Fingerprint conversion by extracting features from images found in Biostore 2, without migrating the images to Biostore 3. This is achieved by a background task, which can execute in Biostore or in the workflow service.

-   Step 4: Fingerprint enrolment in the accelerator databases, using accellerator parameters set in step 2. This is achieved by two background tasks, which can execute in Biostore or in the workflow service.

## 3.1 Step 1: Persons and properties

This step is comprised of two parts

**Part 1** 

Requires the Biostore 2 database, and includes:

-   Creating a database named `Biostore3` in the same database instance as the `Biostore 2` database.

-   Running the `Biostore 3` migration (database schema) scripts

-   Running conversion scripts A, B, and C

-   Backing up the database, and detaching the database.

**Part 2**

The `Biostore2` database is no longer required, and includes:

-   Restoring the database in the target `Biostore3` instance

-   Starting up `Biostore3` and performing the standard application *first-time initialisation*. This includes creating standard and personal users and groups.

-   Creating the X-API-KEY to allow REST access

-   Run conversion scripts D and E

## 3.2 Step 2: Person segments

Person segments determine into which accelerator(s) a person is enrolled. Biostore provides an API to add segments to persons, and remove segments from persons. This API can be called by a client process to create segments.

Alternatively, the biometric conversion allows for a default segment to be obtained from the most recent enrolment audit record in Biostore 2, from the location property. This mechanism will only work, if the branch code (which is used as location) is used as segment identifier.

## 3.3 Biometric conversion

The remaining two steps are described in the following sections:

-   [Section 4 - Bulk-extract BIO-key V6.6 features](#4-bulk-extract-bio-key-v66-features) -- to create features suitable for identification

-   [Section 6 - Build VST database](#6-build-vst-databases) -- to create accelerator databases performing identification

---

# 4. Bulk-extract BIO-key V6.6 features

This section describes Step 3 of the biometric conversion process, where features are re-extracted from samples to ensure compatibility with the BIO-key VST database, enabling accelerated identification.

## 4.1 Bulk extraction options

Server-based extraction works as a background task in the Biostore web application, or in the workflow service. Two different server-based extraction processes are implemented:

-   Extract from Biostore 2 images

-   Extract from Biostore 3 images

This document only describes the process to extract from Biostore 2. The Biostore 3 variant is still available but was used only for the POC.

Client-based extraction was used for POC operation and was required due to unexplained crashes of server-based extraction. This issue has been resolved, and for PROD, only server-based extraction is used.

Therefore, the client-based extraction strategy is not further described in this document.

## 4.2 Bulk extraction process

The background task to convert Biostore 2 images into Biostore 3 relies on:

-   Persons having been migrated from Biostore 2 to Biostore 3 by migration script. This script applies the same pk to Biostore 3, which allows conversion to link persons between the databases.

The background task performs this process:

1.  Obtain a batch of persons which do not yet have a `Conversion` property, which indicates that conversion has not yet taken place. This property is ONLY used for this bulk conversion process; and can be deleted once completed.

2.  For each person, obtain WSQ samples from Biostore 2, extract to VST 6.6 enrol format, and store the feature in Biostore 3. Any features previously enrolled will not be attempted again. This allows for restartability. To accelerate the process, parallel processing is used.

3.  Optionally, obtain the initial segment from Biostore 2, and add the person to this segment.

4.  For each person, record a progress message in the `Conversion` property, and any error messages are recorded in the `Error` property. Failure messages prevent the process to be stuck on the same person; and must be cleared and resolved manually. After clearing the failure message, conversion of the person can be attempted again.

## 4.3 Configuration

### 4.3.1 Biostore 2 connection

To retrieve WSQ images from Biostore2, startup.config must contain a database configuration for Biostore2.

```yaml
<Databases>
 <Profile paradigm="profile" profile="Biostore2">
  <ConnectionString value="Data Source=...;Initial Catalog=..." />
  <IsProtected value="false" />
 </Profile>
</Databases>
```

### 4.3.2 Background task (conversion)

This section describes the process of configuring and executing the conversion task in the Biostore website. 

*This task is replaced with Biostore2Synchronisation.*

Open the Biostore web application, and sign in with `Workflow admins` privileges. Add the *Biostore2ConversionTask* to the list of configured background tasks.

![config-service-background-task1](images/config-service-background-task1.png)

Configure the background task as desired, using the example configuration (from startup.config) below. 

Note that custom properties can be configured either by editing `startup.config` or using the generic property interface `(Alcazar > Properties > Startup settings)`.

It may be required to adjust some standard background task properties:

-   Interval (in sec) - can be reduced to 1 sec for faster processing

-   IntervalFailedPermanent (in sec) - can be reduced to 1 sec to allow faster resumption after a permanent error (e.g. no synchronisable features)

**Task execution parameters**

The `BatchCount` value determines the number of persons retrieved from Biostore 2 in one batch.

The `Threads` value determines the number of parallel threads to perform conversion.

The `Subtypes` value specifies which subtypes (fingers) are to be converted.

The `IsActive` value should remain false, until the task is set to execute automatically.

The `IsAddSegment` value determines the initial segment that should be obtained from the Biostore 2 enrolment audit record.

```xml
<Tasks>
 <Profile paradigm="profile" profile="Biostore2ConversionTask">
  <IsActive value="false" />
  <BatchCount value="100" />
  <Threads value="8" />
  <ConversionProperty value="Conversion" />
  <ErrorProperty value="ConversionError" />
  <SourceSampleClass value="WSQ" />
  <TargetClass value="BioKeyVST66Enrol" />
  <Subtypes value="1|2|3|6|7|8" paradigm="array" />
 </Profile>
</Tasks>
```

Once configured, the background task will be available in the list of configured tasks.

![config-service-background-task2](images/config-service-background-task2.png)

### 4.3.3 Priorities

Priorities can be set on the person to select which identities should be converted first. Conversion is a long-running task, and it might be desirable to have a subset of clients available for identification first.

Set the `Priority` field in the `Person` table to a numeric value. Higher values indicate higher priority (which might be counter intuitive to common use of priority values).

### 4.3.4 Segments

Segments should be set on the person to select to which segment a person belongs, and where they are to be enrolled.

Changing a segment after a person has been enrolled in the accelerator does not move the person to a different accelerator segment immediately, only after the enrolment of the person is re-evaluated. This process is described below.

## 4.4 Execution

Click on *Register* to commence executing the synchronisation task. In order to automatically start the synchronisation task whenever the Biostore web site is active, click on *Activate* (this may be required to continue synchronising when new identities are enrolled, or when the application pool is recycled).

Select *Administration > Background tasks* to and select the *Biostore2ConversionTask*.

![config-service-execution1](images/config-service-execution1.png)

Repeatedly refresh the page to see the progress of conversion and extraction. More detailed error information, if any, can be found when selecting *Biostore2ConversionTask.*

To monitor the progress, you can also observe the creation of *Conversion* markers in the database. These markers contain information about extracted subtypes.

```sql
SELECT *

FROM [Biostore3].[bs].[PersonProperty]

WHERE Name = 'Conversion'
```

When the conversation and extraction is done, and no more messages and markers are produced, you can unregister the background task.

---

# 5. Continuous extract BIO-key V6.6 features

## 5.1 Continuous extraction process

The background task to convert Biostore 2 images into Biostore 3 relies on the high-water mark of the latest Biostore 2 sample, which was covered by bulk conversion, being set as `LastSampleID` in the task configuration.

The background task performs this process:

1.  Obtain a batch of samples with a pk higher than the `LastSampleID`, which indicates that conversion has not yet taken place.

2.  For samples of each person, obtain WSQ samples from Biostore 2, extract to VST 6.6 enrol format, and store the feature in Biostore 3. If a feature of the same person and subtype has been previously enrolled, it will be converted again. This allows for the latest features to be converted. To accelerate the process, parallel processing is used.

3.  Optionally, obtain the initial segment from Biostore 2, and add the person to this segment.

4.  For each person, record a progress message in the `Conversion` property, and any error messages are recorded in the `Error` property. Progress and failure messages are not used to track progress, but are informational only; and must be cleared and resolved manually. After clearing the failure message, conversion of the feature can be attempted again - a process for this has yet to be designed.

5.  Update the the `LastSampleID` to the highest pk of the processed samples, before the process starts again.

## 5.2 Configuration

### 5.2.1 Background task - synchronisation

This section describes the process of configuring and executing the conversion task in the Biostore website.

Open the Biostore web application, and sign in with `Workflow admins` privileges. Add the `Biostore2SynchronisationTask` to the list of configured background tasks.

Configure the background task as desired, using the example configuration (from startup.config) below. 

Note that custom properties can be configured either by editing startup.config or using the generic property interface (*Alcazar > Properties > Startup settings*).

**Task execution parameters**

The `BatchCount` value determines the number of features retrieved from Biostore 2 in one batch.

The `Threads` value determines the number of parallel threads to perform conversion.

The `Subtypes` value specifies which subtypes (fingers) are to be converted.

The `IsActive` value should remain false, until the task is set to execute automatically.

The `IsAddSegment` value determines the initial segment that should be obtained from the Biostore 2 enrolment audit record.

The `ConvertSampleOption` value determines which samples are to be converted, if more than one sample of a subtype of the same person is found. Setting this value to `NewerOnly` can prevent older samples being converted after a new one, but it will not prevent an older sample being converted before the newer one is encountered.

```xml
<Tasks>
 <Profile paradigm="profile" profile=" Biostore2SynchronisationTask">
  <IsActive value="false" />
  <BatchCount value="100" />
  <Threads value="8" />
  <IsAddSegment value="true" />
  <ConversionProperty value="Conversion" />
  <ErrorProperty value="ConversionError" />
  <SourceSampleClass value="WSQ" />
  <TargetClass value="BioKeyVST66Enrol" />
  <Subtypes value="1|2|3|6|7|8" paradigm="array" />
  <LastSampleID value="1" />
  <UpperSampleID value="10000" />
  <ConvertSampleOption value="NewerOnly" />
 </Profile>
</Tasks>
```

Once configured, the background task will be available in the list of configured tasks.

### 5.2.2 Background task - validation

This section describes the process of configuring and executing the validation task in the Biostore website. The validation task follows these steps, per person in Biostore 2:

1.  Accept persons and their samples from Biostore2, where the person in Biostore 2 may or may not be in Biostore3 yet

    a.  If the person is not yet in Biostore 3, it is created

    b.  If the person is a user, its authn-user will be created

    c.  Specified person properties are created

2.  Migrates the best and latest samples across to BS3, if they have not yet been copied.

3.  Extracts features from best and latest samples, which have been migrated

4.  Older features already in BS3 are invalidated.

The steps followed for samples and features are:

1.  Always migrate the newest sample, if it is not yet in Biostore 3.

2.  Always extract a feature from the newest sample, if it is not yet in Biostore 3.

3.  Never extract from older samples.

4.  If there are other features extracted already, of worse quality, invalidate them.

5.  If there are other features extracted already, of better quality, retain the best of them, and migrate its sample too.

6.  Invalidate all other features already existing in Biostore 3.

That way, there will always be exactly ONE valid feature in Biostore 3, and the corresponding sample is migrated. There might be multiple samples (the newest sample, and the one the best feature came from, and possibly any other samples which have already been there.

Open the Biostore web application, and sign in with `Workflow admins` privileges. Add the ***Biostore2ValidationTask*** to the list of configured background tasks with the type name `Sample and feature validation from BS2`.

Configure the background task as desired, using the example configuration (from background.config) below. Note that custom properties can be configured either by editing `background.config` or using the generic property interface `(Alcazar > Properties > Background settings)`.

**Task execution parameters**

The `BatchCount` value determines the number of persons retrieved from Biostore 2 in one batch. The recommended range is 5 to 20 persons.

The `Threads` value determines the number of parallel threads to perform conversion. The recommended range is 1 to 4 threads.

The `Subtypes` value specifies which subtypes (fingers) are to be migrated. Typically, you will migrate all fingers, in which case this property can be omitted.

The `IsAddSegment` value determines the initial segment that should be obtained from the Biostore 2 enrolment audit record.

The `PersonProperties` specifies which person properties should be migrated.

**Note** that the `ConvertSampleOption` does not exist for this task, due to the large number of permutations between feature and sample quality and dates, and what has already been migrated, the migration strategy is implied in the process described above.

The `IsActive` value should remain false, until the task is set to execute automatically.

```xml
<Tasks>
 <Profile paradigm="profile" profile="***Biostore2ValidationTask***">
  <Settings>
   <IsActive value="false" />
   ...
  </Settings>
  <Variables>
   <BatchCount value="10" />
   <Threads value="4" />
   <Subtypes value="1|2|3|6|7|8" paradigm="array" />
   <SourceSampleClass value="WSQ" />
   <TargetFeatureClass value="723" />
   <LowerPersonID value="1" />
   <UpperPersonID value="999999" />
   <IsAddSegment value="true" />
   <PersonProperties value="One|Two|Three" paradigm="array" />
  </Variables>
 </Profile>
</Tasks>
```

Once configured, the background task will be available in the list of configured tasks.

## 5.3 Execution

Click on *Register* to commence executing the synchronisation task. In order to automatically start the synchronisation task whenever the Biostore web site is active, click on *Activate* (this may be required to continue synchronising when new identities are enrolled, or when the application pool is recycled).

Select *Administration > Background tasks* to and select the *Biostore2ValidationTask*.

Repeatedly refresh the page to see the progress of conversion and extraction. More detailed error information, if any, can be found when selecting *Biostore2ValidationTask*.

To monitor the progress, you can also observe the as the `<LastPersonID>` *high-water mark* increments; and retrieve event log entries. Note that no person properties are created for auditing.

```sql
SELECT *

FROM [Biostore3].[base].[EventLog]
```

When the validation is done, and no more messages are produced, the `LastPersonID` will be set to the highest person pk in the Biostore 2 database. The background task may continue to execute indefinitely, while Biostore 2 is operational. The background task can also be reset to start from the beginning.

## 5.4 Testing

There is a large number of permutations between feature and sample quality and dates, and what has already been migrated, therefore it is impossible to test all variations. Therefore, comprehensive testing in pre-prod environments is required, and a phased, slow implementation is PROD is recommended.

Please validate in particular:

-   That there is a sample for every feature

-   That there is always exactly one valid feature per subtype

-   That any feature which has been invalidated, has a corresponding better feature. This variation can be identified by these messages:

    - For {subtype} newest feature #{newestFeature3} invalidated due to #{bestFeature3}

    - For {subtype} stored feature #{worseFeature3} invalidated due to #{bestFeature3}

    - For {subtype} other feature #{existingFeature3} invalidated due to #{betterFeature3}

-   That for every better feature than the newest feature, the corresponding sample is migrated.

    - For {subtype} saved sample #{bestSample3}

---

# 6. Build VST databases

Building the accelerator databases is server-based. It works as a set of two background task in the Biostore web application or the Workflow service. Theses are managed from the web application.

Building and re-building of accelerator databases consists of two steps:

-   **Evaluating** accelerator enrolments is based on available features of a person, segments the person belongs to, and accelerator maps determining in which accelerator (search agent and sub-address) to enrol. 
    **Re-evaluate** refers to the process of continuously monitor new features and changed segments, to update enrolment records (also called feature addresses)

-   **Building** accelerators utilises enrolment records to enrol features into accelerator databases, or to remove them. 
    **Re-build** refers to the process of continuously monitoring new enrolment records and executing them by enrolling or removing features.

![vst-db1](images/config-service-vst-db1.png)

Re-evaluation and re-building does not differentiate between features created by bulk-conversion, ongoing conversion, or enrolment directly into Biostore 3. The same process handles all re-evaluation and re-building, irrespective of the trigger.

## 6.1 Re-evaluation

Re-evaluation refers to the process of determining in which accelerator a person or feature should or should not be. Re-evaluation is triggered by adding new features, or by adding or removing person segments. Re-evaluation may also be required when an accell-map entry changes, however this is not triggered automatically.

The re-evaluation background task first checks for new features, incrementing the feature high-water mark, and only when no more new features are found, will it check for new person segments. This sequence was chosen to simplify the process where one person may have both new features and new person segments.

When processing new features, these new features will be applied to **active** person segments only. This is because pending person segments will be processed thereafter, after all new features have been dealt with.

When processing new person segments, all features will already have been processed, and person segments can be applied to all features at the same time, irrespective of how long they have been in the system.

## 6.2 Configuration

### 6.2.1 Accelerator settings

Configure accelerator settings as follows:

| *FeatureClass*          | BioKeyVST66Enrol                                                                 |
|-------------------------|----------------------------------------------------------------------------------|
| Biostore Accelerator    | Alcazar.Biostore.Accell.RestAccellerator, Alcazar.Biostore.Process               |
| Agent Accelerator type  | Alcazar.Biostore.Accell.StoreAccellerator, Alcazar.Biostore.Process              |

### 6.2.2 Identification profiles

Configure the default identification profile as follows:

-   Pre-steps: CheckValidUser

-   Post-steps: (none)

-   Storage selector for samples: Incoming - Never

-   Storage selector for features: (empty)

Identification-specific settings: 

-   Use accelerator : true                                         

![config-service-id-profiles1](images/config-service-id-profiles1.png)

### 6.2.3 Accelerator profiles

The accelerator profiles must be configured in `biometric.config` as follows.

**REST accelerator**

The REST accelerator operates in the Biostore 3 service and redirects enrolment and search requests to configured search agents.

The `Timeout` defines the timeout allowed for agent requests in seconds. 100 seconds appears to be a sufficiently long timeout.

```xml
<Biometric>
 <Accellerators>
  <Profile paradigm="profile" profile="RestAccellerator">
   <KnownAdapterSettings>
    <Timeout value="100" />
   </KnownAdapterSettings>
  </Profile>
 </Accellerators>
</Biometric>
```

**Store accelerator**

The Store or Agent accelerator operates in the Biostore 3 search agent and performs enrolment and search requests within the search agents.

The `PathName` must be a template to a folder on the server. The parameter {0} will be replaced with the agent sub-address, and the resulting subfolder will contain the VST database for the given sub-address.

The `MaxSize` defines the maximum size of the VST database in 100MB increments.

```xml
<Biometric>
 <Accellerators>
  <Profile paradigm="profile" profile="StoreAccellerator">
   <KnownAdapterSettings>
    <AgentStore
          value="Amaqele.Biometric.Store.Biokey.AgentStoreBiokeyVST66,
          Biometric.Engine.Biokey" />
    <PathName value="c:progvstdatabase-{0}.vst" />
    <MaxSize value="10" />
    <BatchCount value="1" />
    <MaxHitCount value="3" />
    <FeatureClass value="BioKeyVST66Enrol" />
    <SearchFlags value ="0" />
    <ThreadCount value ="30" />
    <FilterA value ="5" />
    <FilterS value ="5" />
    <FullSize value ="100" />
    <Threshold value ="50" />
    <UpperThreshold value="60"/>
    <MaxHandleCount value="99"/>
    <HandleTimeout value="5"/>
   </KnownAdapterSettings>
  </Profile>
 </Accellerators>
</Biometric>
```

The `UpperThreshold` sets the zone of uncertainty between hits and no-hits, and is expressed as native score.

The `MaxHandleCount` defines the maximum number of handles which the system may obtain simultaneously for one given accelerator. Zero means unlimited.

The `HandleTimeout` defines the number of seconds a process may wait for a handle before it times out and fails. The handle timeout only applies if the maximum number of handles is temporarily exceeded.

The `HandleStyle` defines which methods to apply for handle pooling. Always 0, future versions will add additional methods.

### 6.2.4 Accelerator agents and maps

Biostore keeps track of its search agents and sub-addresses available on each search agent. Populate the AccellAgent table with a list of all search agents and their URL.

  | pkAgentID | AgentAddress              | AgentProperties |
|------------|---------------------------|-----------------|
| 1          | https://localhost:44330   | NULL            |
| 2          | https://localhost:44384   | NULL            |

Populate the AccellMap table with a list of sub-addresses of each search agent, and the data they contain. Each sub-address will result in a separate VST database.

| pkMapID  | Segment  | Subtype  | AgentSubAddress  | Flags  | fkAgentID  |
|----------|----------|----------|------------------|--------|------------|
| 1        | reg99    | 0        | 0000             | 1      | 1          |
| 2        | reg01    | 1        | 0001             | 1      | 1          |
| 3        | reg01    | 2        | 0002             | 1      | 1          |
| 5        | reg01    | 3        | 0003             | 1      | 1          |
| 6        | reg01    | 4        | 0004             | 1      | 1          |
| 7        | reg01    | 5        | 0005             | 1      | 1          |
| 8        | reg01    | 6        | 0006             | 1      | 1          |
| 9        | reg01    | 7        | 0007             | 1      | 1          |
| 10       | reg01    | 8        | 0008             | 1      | 1          |
| 11       | reg01    | 9        | 0009             | 1      | 1          |
| 12       | reg01    | 10       | 0010             | 1      | 1          |

The `Segment` field identifies a segment of the database which this sub-address contains. Each person resides in one or more segments; and can be found by an identity search against these segments.

The `Subtype` identifies the finger which this sub-address contains. Zero means all fingers of this segment are contained in this single sub-address.

The `AgentSubAddress` identifies a sub-address, which translates to a separate VST database.

The `Flags` determine the use of the sub-address. One means an active sub-address, into which new features will be enrolled. Only one active sub-address for a segment and subtype may exist at a time.

### 6.2.5 Background task -- re-evaluate

This section describes the process of configuring and executing the re-evaluation task in the Biostore website.

Open the Biostore web application, and sign in with `Workflow admins` privileges. Add the `AccelleratorReevaluateTask` to the list of configured background tasks.

Configure the background task as desired, using the example configuration (from startup.config) below. Note that custom properties can be configured either by editing startup.config or using the generic property interface (*Alcazar > Properties > Startup settings*).

The `FeatureBatchCount` value determines the number of features retrieved in one batch.

The `SegmentBatchCount` value determines the number of persons for which person segments are retrieved in one batch.

The `Threads` value determines the number of parallel threads to perform re-evaluation. This property is used for both feature and segment evaluation.

The `UseFeatureOption` value determines which features are to be enrolled, if more than one features of a subtype of the same person is found. This value is used when applying person-segments, however it is not used when newly enrolled features are applied.

The `BatchMode` value determines how threading is handled.

-   0 means no threading (legacy)

-   1 means retrieving a batch of person segments [without]{.underline} consideration for other person segments of the same person, which might then be processed later

-   2 means retrieving a batch of person segments [with]{.underline} consideration for other person segments of the same person, which are then all processed together

The `IsActive` value should remain false, until the task is set to execute automatically.

The `Subtype` identifies the finger which are to be evaluated for person segments. This property is NOT used when evaluating new features.

```xml
<Tasks>
 <Profile paradigm="profile" profile=" AccelleratorReevaluateTask">
  <IsActive value="false" />
  <FeatureBatchCount value="100" />
  <SegmentBatchCount value="30" />
  <FeatureClass value="BioKeyVST66Enrol" />
  <LastFeatureID value="0000" />
  <UseFeatureOption value="NewerOnly" />
  <BatchMode value="1" />
  <Threads value="1" />
  <Subtypes value="RightThumb|LeftMiddleFinger" paradigm="array" />
 </Profile>
</Tasks>
```

Once configured, the background task will be available in the list of configured tasks.

### 6.2.6 Background task -- re-build

This section describes the process of configuring and executing the re-build task in the Biostore website.

Open the Biostore web application, and sign in with `Workflow admins` privileges. Add the `AccelleratorRebuildTask` to the list of configured background tasks.

Configure the background task as desired, using the example configuration (from startup.config) below. Note that custom properties can be configured either by editing startup.config or using the generic property interface (*Alcazar > Properties > Startup settings*).

The `BatchCount` value determines the number of enrolment records retrieved in one batch.

The `BatchMode` value determines how threading is handled.

-   0 means no threading (legacy)

-   1 means multi-threading

The `Threads` value determines the number of parallel threads to perform re-build.

The `AgentID` value determines which agent is rebuilt. If not set, or set to zero, all agents are being rebuilt by this instance.

The `IsActive` value should remain false, until the task is set to execute automatically.

```xml
<Tasks>
 <Profile paradigm="profile" profile=" AccelleratorRebuildTask">
  <IsActive value="false" />
  <BatchCount value="100" />
  <BatchMode value="1" />
  <Threads value="8" />
  <AgentID value="1" />
 </Profile>
</Tasks>
```

Once configured, the background task will be available in the list of configured tasks.

## 6.3 Execution

Click on *Register* to commence executing the re-evaluation or re-build task. In order to automatically start the tasks whenever the Biostore web site is active, click on *Activate*.

Select *Administration > Background tasks* and select the corresponding task.

Repeatedly refresh the page to see the progress of re-evaluation and re-building. More detailed error information, if any, can be found when selecting the task*.*

To monitor the progress, you can also observe status changes of *segments (person-segments)* and creation and status changes of *enrolment records (feature-addresses)* in the database. These markers contain information about enrolled features.

The background task should not be unregistered; but continue to execute indefinitely.

---

# 7. Workflow configuration

For Workflow configuration refer to @biostore-wf-config

---

# 8. Single sign-on configuration

## 8.1 Identity provider

For Biostore single sign-on and OAuth authorisation, an identity provider is required which supports SAML 2.0 and OAuth 2.0. 
Alcazar is providing an evaluation license for Alcazar Single for this purpose. See the Alcazar Single installation and configuration guide 
to configure the Alcazar Single identity provider and authorisation server.

## 8.2 Security definitions

Before attempting to configure single sign-on or OAuth authorisation, import the following security definitions into Biostore:

-   authn-SP 2025.xml

## 8.3 Single sign-on

Single sign-on for the Biostore website simplifies user management by having one central set of identities and credentials.
Refer to @configure-client-sso for configuring single sign-on for the Biostore service.

## 8.4 OAuth authorisation

OAuth authorisation provides a secure, standards-based authorisation and authentication mechanism for the Biostore service.
Refer to @configure-client-oauth for configuring OAuth authorisation for the Biostore service.

The recommended scope value for the Biostore service is `Biostore`.