---
title: Alfresco-AD integration
published: 2025-09-18
description: 'Integrating Alfresco with Active Directory user authentication and management.'
image: '/alfresco-ad/images.png'
tags: []
category: 'IT'
draft: false 
lang: 'en'
---

## Integrating Alfresco with Active Directory
 
 In this article, we will explore how to integrate Alfresco with AD for user authentication and management. This integration allows users to log in to Alfresco using their existing AD credentials, streamlining access and improving security.



## My setup

 - Alfresco acs 25.2 (community edition), installed with Docker from [github](https://github.com/Alfresco/acs-deployment)
 - Access to a working Active Directory server.
 - make sure both are on the same network and can communicate with each other.

we'll be working with the alfresco container, so either mount its filesystem or edit directly on the docker desktop

## Step 1: Enable ldap logging

to track errors and find out what is going wrong faster, enable ldap logging in `log4j2.properties` at 
`/usr/local/tomcat/webapps/alfresco/WEB-INF/classes/`
```
log4j.logger.org.alfresco.repo.importer.ImporterJob=debug
log4j.logger.org.alfresco.repo.importer.ExportSourceImporter=debug
log4j.logger.org.alfresco.repo.security.authentication.ldap=debug
```

## Step 2: Configure alfresco-global.properties
create a user for alfresco in your AD, this user will be used to bind and search the AD for users and groups, make sure this user has read access to the relevant OUs in your AD


Next, edit the `alfresco-global.properties` file located in `/usr/local/tomcat/shared/classes/` and add the following configuration settings:

```
###############################
## Alfresco Repository LDAP/AD Configuration ##
###############################

# Authentication chain: Alfresco + LDAP
authentication.chain=alfinst:alfrescoNtlm,ldap1:ldap

###############################
## LDAP Authentication ##
###############################
ldap.authentication.active=true
ldap.authentication.java.naming.provider.url=ldap://<AD_VM_PORT>:389
ldap.authentication.userNameFormat=%s@<DOMAIN>.<TLD>
ldap.authentication.allowGuestLogin=false

###############################
## LDAP Synchronization ##
###############################
ldap.synchronization.active=true
ldap.synchronization.java.naming.provider.url=ldap://<AD_VM_PORT>:389
ldap.synchronization.java.naming.security.principal=<ALFRESCO_USERNAME>@<DOMAIN>.<TLD>
ldap.synchronization.java.naming.security.credentials=<ALFRESCO_PASSWORD>

# User sync
ldap.synchronization.userSearchBase=CN=Users,DC=<DOMAIN>,DC=<TLD> # here i am not working with OUs, just the default Users container
ldap.synchronization.personQuery=(objectclass=user)

# Group sync
ldap.synchronization.groupSearchBase=CN=Users,DC=<DOMAIN>,DC=<TLD> # here i am not working with OUs, just the default Users container
ldap.synchronization.groupQuery=(objectclass=group)

# Cron for periodic sync (every minute)
synchronization.import.cron=0 */1 * * * ? # change this to your needs 2 hours is good for most cases

# Sync options
synchronization.syncOnStartup=true
synchronization.allowDeletions=true
ldap.synchronization.userIdAttributeName=sAMAccountName
ldap.synchronization.userEmailAttributeName=mail
ldap.synchronization.groupIdAttributeName=cn
ldap.synchronization.groupDisplayNameAttributeName=cn
ldap.synchronization.groupType=group
ldap.synchronization.personType=person
ldap.synchronization.groupMemberAttributeName=member
#ldap.synchronization.modifyTimestampAttributeName=modifyTimestamp
ldap.synchronization.timestampFormat=yyyyMMddHHmmss'.0Z' # you may need to adjust this based on your AD setup
```

make sure to replace these placeholders with your actual values:
- `<AD_VM_PORT>`: The IP address or hostname of your AD server.
- `<DOMAIN>`: Your AD domain name.
- `<TLD>`: Your AD top-level domain (e.g., com, local).
- `<ALFRESCO_USERNAME>`: The username of the AD user created for Alfresco
- `<ALFRESCO_PASSWORD>`: The password of the AD user created for Alfresco

and <strong>read</strong> the comments in the file for more information about other settings.

## Step 3: Restart Alfresco
After making these changes, restart the Alfresco container and watch the logs for any errors or issues related to LDAP/AD integration.
