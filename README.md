# Update-Multi-Select-Picklist-Using-URL-Parameter
This Deluge script allows you to update a multi-select Picklist using URL Parameters from Zoho CRM into Zoho Projects.

## Core Idea
Zoho Projects Rest API allows you to interact with Zoho Projects from another Application as long as you have the appropiate connection setup on the Application that you want to use it with.[Example with Zoho CRM REST API.](https://github.com/TheWorkflowAcademy/Zoho-CRM-Custom-Connections/blob/master/README.md).

Note: There are various different functions which Zoho Projects API can provide as seen there. [Different Functionality of Zoho Projects API](https://www.zoho.com/projects/help/rest-api/projects-api.html)

### Configuration
* Configure Zoho CRM connection to Zoho Projects.
* Zoho Projects OAuth Connections Scopes Needed (Adjust as required by you):
  * ZohoCRM.modules.ALL
  * ZohoProjects.portals.ALL
  * ZohoProjects.projects.ALL
  * ZohoProjects.status.ALL
  * ZohoProjects.tags.ALL

### Tutorial

#### Identify the Custom Field ID that will be updated on Zoho Projects 
To update a custom field in Zoho Project will require the Field ID, an API call will be used to identify the Custom Field ID in Zoho Projects. 

For the purpose of getting the Custom Field ID, 
* Create a standalone function on CRM.
* This deluge code will return you the list of custom fields that is available in your Zoho Projects by using a API call from CRM.

```javascript
//getCustomFields
getCustomFields = invokeurl
[
	url :"https://projectsapi.zoho.com/restapi/portal/[Portal ID/Name]/projects/customfields/"
	type :GET
	connection:"zprojects"
];
return getCustomFields;
```
* Portal ID/Name can be gathered from the Connection created eariler.
* From the output will give a list of all custom fields available in Zoho Project that you have added.
* The custom field IDs can be gathered from the list. There are 3 types of IDs:
  * UDF_CHAR1       (String value for single-line fields)
  * UDF_MULTI1      (Multi-value pick list)
  * UDF_MULTIUSER1  (Multi-user pick list)
  * UDF_TEXT1	(Text area field)
    * FYI The number behind each ID correspond to the number of instences of the same type of field types that are available. Example if you have 2 multi-value pick list, one will be UDF_MULTI1 and another will be UDF_MULTI2.

Note: For an indepth look into the functionality of this API call and also to see an example of the response visit [here](https://www.zoho.com/projects/help/rest-api/projects-api.html#alink3) 

#### Update Custom Fields using their IDs
After gathering the Custom Field IDs we can begin to construct the code to update the Custom Fields.

To update the custome fields using the IDs,
* Retrieve the data from CRM.

```javascript
record = zoho.crm.getRecordById([Module_Name],[record_ID]);
//Retrieve Data and assign to variables
charValue = record.get("CharValue")
textValue = record.get("TextValue")
project_ID = record.get("ProjectID")
multiValue = record.get("MultiValue") //Change the value in get() according to the Field_API_Name that you are using
//Add all of the multi-values into a list
multiValueList = multiValue.toList();
```

* Define the parameter type that you will be using, this will be parse to the API when called.

```javascript
//Defining the parameter type
charID = "UDF_CHAR1";
textID = "UDF_TEXT1";
```

* For Multi-value pick list formating is required as there will be multiple values involved.

```javascript
//Constructing & Formating the parameter that will be used to parse the data for the multi-value pick list
for each  multi in multiValueList
{
	URLparam = URLparam + "UDF_MULTI1=" + encodeUrl(multi) + "&";
}
multiValueParam = URLparam.removeLastOccurence("&");
info multiValueParam;
```

* Compile the non-multi value data into one Map that will be parse as a parameter.

```javascript
updateMap = Map();
updateMap.put(charID,charValue);
updateMap.put(textID,textValue);
```


#### Update Multi-Select Picklist using URL Parameter (API)
Once done compiling the data needed to be updated, its time to update the related Zoho Project via an API.

```javascript
updateProject = invokeurl
[
	url :"https://projectsapi.zoho.com/restapi/portal/[Portal ID/Name]/projects/" + project_ID + "/?" + multiValueParam
	type :POST
	parameters:updateMap
	connection:"zprojects"
];
```


