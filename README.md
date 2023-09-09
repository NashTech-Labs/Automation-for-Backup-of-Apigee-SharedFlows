# Sharedflows in Apigee

In Apigee, "shared flows" are a powerful feature that allows you to encapsulate and reuse logic, policies, and configurations across multiple API proxies. Shared flows streamline the development and management of API proxies by enabling you to centralize common functionality and apply it consistently to various APIs. Shared flows promote reusability of code and policies. Instead of duplicating the same logic in multiple API proxies, you can create a shared flow once and then attach it to multiple API proxies as needed.

## Taking Backup of Sharedflows from Apigee to Github Repository using Jenkins

This repository contains Jenkinsfile which has script for taking Backup of sharedflow from Apigee to Github Repository using Jenkins.

For using this template we have to update following values in  jenkinsfile:

`<GCP_CREDENTIAL_ID>`

`<GITHUB_CREDS_ID>`

`<GITHUB_USER_NAME>`

`<GITHUB_USER_EMAIL>`

`<APIGEE_ORG_NAME>`
            
1. Create a Jenkins pipeline using this repository. Configure your jenkins pipeline to use this github repository accordingly.

2. This jenkins pipeline contains a scheduled cronjob (weekly basis) which will run to automate the backup process at regular intervals. This ensures that backups are performed consistently without manual intervention.

3. Jenkins pipeline for backup is utilizing the Apigee Command Line Interface (Apigeecli) to export the identified Apigee resources. Apigeecli allows to interact with the Apigee API and export configurations and definitions.

4. Jenkins Pipeline for backup has a stage for cleaning the backup data of a month ago from Github Repository. Regular cleanup ensures that only relevant and recent backup files are retained.

5. When this pipeline will run and then it will create a following type of directory structure here in this repository and will store backup data in this:

`<APIGEE_ORG_NAME> > <DATE_OF_BACKUP> > sharedflows > <sharedflows_zip_bundles>`

![jenkins_pipeline](https://i.postimg.cc/KjxyxYf0/download-4.png)


6. To verify this move to:

   GCP > Apigee > sharedflows

