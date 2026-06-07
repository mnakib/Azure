
### Create a Junior Admins group containing the user account as its member.

Use the Azure PowerShell to add a group named __*"Junior Admins 1981"*__ and user named __*"Dylan Williams"*__, then make the user member of the group
```powershell

# Connect to Microsoft Entra ID
Connect-AzureAD

# Create a password profile
$passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
```

> **NOTE:** This command creates a blank, structured password profile object in memory and saves it to the variable $passwordProfile. This object specifically holds password-related settings (like the actual password string and whether the user must reset it) required when creating or updating a user account in Azure Active Directory.
> By itself, this command does not change anything in your directory. It is used as a preparatory step. You instantiate the blank object, fill out its properties, and then pass it into a user creation command.

```powershell
# Assign the password value
$passwordProfile.Password = "Pa55w.rd1234"

# Define the domain name
DOMAIN_NAME=mouradnakibhotmail.onmicrosoft.com

# Create the user "Isabel Garcia"
New-AzureADUser -DisplayName 'Isabel Garcia' -PasswordProfile $passwordProfile -UserPrincipalName "Isabel@$DOMAIN_NAME" -AccountEnabled $true -MailNickName 'Isabel'


# Create the "Junior Admins 1981" group
New-AzureADGroup -DisplayName "Junior Admins 1981" -MailEnabled $false -SecurityEnabled $true -MailNickName "JuniorAdmins1981"


# Add the user "Isabel Garcia" to the group "Junior Admins 1981" 
Add-AzADGroupMember -MemberUserPrincipalName $user.userPrincipalName -TargetGroupDisplayName "Junior Admins 1981"

# List the users contained in the "Junior Admins 1981" group
Get-AzADGroupMember -GroupDisplayName "Junior Admins 1981"
```

### Create a Service Desk group containing the user account as its member.

Use Azure CLI to add a group named __*"Service Desk 1981"*__ and user named __*"Dylan Williams"*__, then make the user member of the group

```bash
# Define the domain name
DOMAIN_NAME=mouradnakibhotmail.onmicrosoft.com

# Create the user "Dylan Williams"
az ad user create --display-name "Dylan Williams" --password "Pa55w.rd1234" --user-principal-name Dylan@$DOMAIN_NAME

# Create the group "Service Desk 1981"
az ad group create --display-name "Service Desk 1981" --mail-nickname "ServiceDesk1981"

# Get the user's object ID 
USER_ID=az ad user show --id Dylan@$DOMAIN_NAME --query id --output tsv

# Add the user "Dylan Williams" to the group "Service Desk 1981" 
az ad group member add --group "Service Desk 1981" --member-id $USER_ID

# List the users contained in the "Service Desk 1981" group
az ad group member list --group "Service Desk 1981" --output table
```


### Assign the Virtual Machine Contributor role to the Service Desk group

Create a resource group

```bash
RG_NAME=AZ500Lab01
RG_LOCATION=eastus

az group create --name $RG_NAME --location $RG_LOCATION
```

Assign the Service Desk Virtual Machine Contributor permissions.

```bash
# Get the Group Object ID
GROUP_OBJECT_ID=$(az ad group show --group "Service Desk 1981" --query "id" --output tsv)

# Get the Subscription ID
SUBSCRIPTION_ID=$(az account show --query "id" --output tsv)

# Scope the permission to the resource group
az role assignment create \
  --assignee $GROUP_OBJECT_ID \
  --role "Virtual Machine Contributor" \
  --scope "/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RG_NAME"
```
