# Set-up environment variables

<span style="color:red">/!\ IMPORTANT </span> : your **appName** value MUST BE UNIQUE

## Core variables
```sh
# az account list-locations : francecentral | northeurope | westeurope | eastus2
location=northeurope
echo "location is : " $location 

appName="azconapps" 
echo "appName is : " $appName 

capps_env="pocenstock"
echo " CONTAINERAPPS ENVIRONMENT is : " $capps_env

github_usr="<Your Git Hub Account>"
echo "GitHub user Name : " $github_usr 

git_url="https://github.com/$github_usr/azurecontainerapps-helloworld" #.git
echo "Project git repo URL : " $git_url

git_branch="main"
echo "Git branch : " $git_branch

git_pat="<SET YOUR OWN Personal access token>"
echo "Git Personal access token : " $git_pat

```


## Extra variables
Note: The here under variables are built based on the varibales defined above, you should not need to modify them, just run this snippet

```sh
rg_name="rg-${appName}-${location}" 
echo " RG name:" $rg_name 

# Storage account name must be between 3 and 24 characters in length and use numbers and lower-case letters only
storage_name="stcloudshell""${appName,,}"
echo "Storage name:" $storage_name

fs_share_name="sharefs"
echo "FS share name:" $fs_share_name

analytics_workspace_name="log-${appName}"
echo "Analytics Workspace Name :" $analytics_workspace_name

acr_registry_name="acr${appName,,}"
echo "ACR registry Name :" $acr_registry_name

```