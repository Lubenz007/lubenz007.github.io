---
title: Getting Started with MSGraph API
date: 2023-03-25 13:49:00 -0000
categories: [MSGraph]
tags: [azure,msgraph]     # TAG names should always be lowercase
---
## Authentication and Authorization

### MSGraph API is a powerful tool for authentication and authorization that allows developers to access Microsoft services and data. It provides a secure way to authenticate users, authorize applications, and manage user data. With MSGraph API, developers can easily integrate their applications with Microsoft services like Outlook, OneDrive, Office 365, Azure Active Directory, and more. In this article, we'll explore how to get started with MSGraph API concepts and use cases of the API.
## Authentication
### We will use the Azure CLI to create a service principal and get the app id and secret. We will also use the Azure CLI to view all the API permissions available in Microsoft Graph.

> Get the Azure CLI from [here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
{: .prompt-info }

```powershell
# login to Azure
az login

#View All API Permissions Microsoft Graph to see what is available
$Permissions = az ad sp list --filter "appId eq '00000003-0000-0000-c000-000000000000'" | ConvertFrom-Json
#to see all permissions
$Permissions.appRoles | Select-Object -Property value,allowedMemberTypes,description
# Get select permission from list above and id to use in next command
$Permissions.appRoles | Select-Object -Property value,allowedMemberTypes,id,description | Where-Object {$_.value -eq "User.Read.All"}

# Create service principal
az ad app create --display-name "Alit-GraphAPI"
# Graph API App ID: from output "appId": "d52bda31-bd71-4a17-b563-7921a17d79e7"

# Reset app secret to get new secret
az ad app credential reset --id "d52bda31-bd71-4a17-b563-7921a17d79e7"
# Graph API App Secret: from output
#{
# "appId": "d52bda31-bd71-4a17-b563-7921a17d79e7",
#  "password": "3b3b3b3b-3b3b-3b3b-3b3b-3b3b3b3b3b3b",
#  "tenant": "5eeb8561-5493-4b39-906f-038356850aaa"
#}

# set secret to 3 years
#az ad app credential reset --id 13d15b6f-94ad-4df4-a88d-72f355e5f92d --years 3

# Add Microsoft Graph application permission user.read.all # Guide https://learn.microsoft.com/en-us/cli/azure/ad/app/permission?view=azure-cli-latest
az ad app permission add --id d52bda31-bd71-4a17-b563-7921a17d79e7 --api 00000003-0000-0000-c000-000000000000 --api-permissions df021288-bdef-4463-88db-98f22de89214=Role

# Grant admin consent #this needs to be done by a Cloud Application Administrator
az ad app permission admin-consent --id d52bda31-bd71-4a17-b563-7921a17d79e7

# Get app id 
az ad app list --query "[?displayName=='Alit-GraphAPI'].{id:appId,secret:passwordCredentials[0].value}"
#{
#    "id": "d52bda31-bd71-4a17-b563-7921a17d79e7",
#    "secret": null
#  }

#get app api permissions
az ad app permission list --id d52bda31-bd71-4a17-b563-7921a17d79e7 --output json
#{
 #   "resourceAccess": [
 #     {
 #       "id": "df021288-bdef-4463-88db-98f22de89214",
 #       "type": "Role"
 #     }
 #   ],
 #   "resourceAppId": "00000003-0000-0000-c000-000000000000"
 # }

```
Reference: 
* [nielskok.tech](https://www.nielskok.tech/azure-ad/view-all-api-permissions-microsoft-graph/)
* [how-to-create-service-principal-portal](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal)
* [how-to-authenticate-service-principal-cli](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-authenticate-service-principal-cli)
* [how to-add-app-roles-in-azure-ad-apps](https://www.c-sharpcorner.com/article/how-to-add-app-roles-in-azure-ad-apps/)
