# ServiceNow Inbound (to xMatters) Integration via IntegrationHub
IntegrationHub is a feature in ServiceNow that makes integrating to third-party (in this example, xMatters) apps easier than ever before, and the best part about it? No coding! Well, at least minimal... With this inbound integration, you can set up a trigger from any (that's right, any!) table and table entry to create an event in xMatters.

Check out the sweet video [here](media/mysweetvideo.mov). 
<kbd>
  <img src="https://github.com/xmatters/xMatters-Labs/raw/master/media/disclaimer.png">
</kbd>

# Pre-Requisites
* ServiceNow instance, with IntegrationHub purchased and installed - if you don't have IntegrationHub, see details on how to get it [here]()
* Existing communications plan - Use this packaged integration from the integrations page, or build your own
* xMatters account - If you don't have one, [get one](https://www.xmatters.com)!

# Files
* [ExampleCommPlan.zip](ExampleCommPlan.zip) - This is an example comm plan to help get started. 
* [EmailMessageTemplate.html](EmailMessageTemplate.html) - This is an example HTML template for emails and push messages. 
* [Payload](PayloadBody) - An example payload body to use in ServiceNow

# How it works
Add some info here detailing the overall architecture and how the integration works. The more information you can add, the more helpful this sections becomes. For example: An action happens in Application XYZ which triggers the thingamajig to fire a REST API call to the xMatters inbound integration on the imported communication plan. The integration script then parses out the payload and builds an event and passes that to xMatters. 

# Installation
## In xMatters:
### Setting up the Communication Plan
1. Download the [zip file](ExampleCommPlan.zip) onto your computer
2. Log in to xMatters and navigate to the **DEVELOPER** tab
3. Click **Import Plan** and select the zip file you just downloaded, then click **Import Plan**
4. On the righthand side of the Communication Plan you just imported, click **Edit** > **Integration Builder**
5. Next to **Inbound Integrations**, open the dropdown menu by clicking the blue **1 Configured**, then click **ServiceNow IntegrationHub Inbound**
6. At the bottom of the page, copy the request URL to something accessible
### Setting up an account to access the Communication Plan
1. Navigate to the **USERS** tab
2. Click **Add** on the righthand side
3. Give the user a first and last name (e.g. ServiceNow IntegrationHub), user ID (username used for logging in) and password, and give it the *REST Web Service User* role, then click **Add**
4. Navigate back to the **DEVELOPER** tab and next to the Communication Plan you just imported, click **Edit** > **Access Permissions**, then type the user ID of the account you just created, select the account, and click **Save Changes**. Alternatively, you can select **Accessible by All**, however this allows all users in your xMatters instance to view and edit the Communication Plan
## In ServiceNow:
### Setting up a Credential and Connection Alias:
Now that you have an xMatters Communication Plan, we need to set up a Connection alias in ServiceNow to connect quickly and easily to the Comm Plan
1. Log in to your ServiceNow instance
2. In the *Filter navigator*, search for "alias", and select **Connection & Credential aliases**
3. In the top left next to where you see **Connection & Credential aliases**, click **New**
4. In the dropdown menu next to **Type**, select *Credential*, then give the Credential a Name, and hit **Submit**
5. You will now see a list of Connections and Credentials; click (or search if you have many set up) the Credential you just created
6. Next to **Credentials** (right below the Credential *Name* and *Type*), click **New**, and select the *Basic Auth Credentials* option on the following page
7. Give the Credential a *Name*, and enter the *User* and *Password* for the xMatters account with access to the Communication Plan, then click **Submit** to finish 
8. Now that you have a Credential, click the back button to return to the list of Connections & Credentials, then hit **New**
9. This time, in the dropdown menu next to **Type**, select *Connection and Credential*, and in the dropdown menu next to **Connection Type**, select *HTTP*, then give the Connection a Name, and click **Submit**
10. You will again see a list of Connections and Credentials; click (or search for) the Connection you just created
11. Next to **Connections** (right below the Connection *Name* and *Type*), click **New**
12. Give the Connection a Name, then click the **Credential**, and select the Credential that you just created
13. Paste the URL you copied from the xMatters Communication Plan into the **Connection URL** box, then Submit
Now you have a working Connection Alias that we can use to easiliy communicate to xMatters! Now for setting up an Action
### Setting up an Action
1. In the *Filter navigator*, search for "flow", and under *Flow Designer*, click **Designer**
2. You should see a list of flows. On the righthand side, click **New** > **New Action**
3. Give the Action a Name and Description, then **Submit**
4. In the Inputs, click **Create Input**, and give the input variable a name, then select the type and the table from which the input will be grabbed (for example, an input of type *Reference* from the table *Incident [incident]*)
5. Now for the magic. Between the Inputs and Outputs, click the **+** button to add a step. With IntegrationHub installed, we have the **REST** step available. Select this and you will have a bunch of options show up
6. Next to **Connection**, leave the default of *Use Connection Alias*, then select your Connection Alias in the dropdown menu next to **Connection and Credential Aliases**
7. From the dropdown menu next to **HTTP Method**, select *POST* (the Base URL is grayed out because it is already defined in the Connection Alias). We will not need to fill anything in for **Resource Path** in this example because the full URL from the Communication Plan is in the Connection Alias. However, if you would like to set up multiple integrations using IntegrationHub, it may be a good idea to define the URL in the Connection Alias as being ```https://example.xmatters.com/api/integration/1/functions/``` (with example being replaced with your instance), and then you could create multiple inbound integrations and have different actions use different inbound integration URLs by using the same Connection alias, and only change the Resource Path to ```[API key]/triggers``` .
8. For headers, use *Content-Type* under **Name**, and *application/json* under **Value**
9. Under body, you will have to enter each part of the input you would like to send to xMatters (yeah, this part kind of sucks...). Here is a good example of an input body:
```
{"properties": {
"active": "action➛incident➛Active",
"assigned_to": "action➛incident➛Assigned to",
"assignment_group": "action➛incident➛Assignment group",
"category": "action➛incident➛Category", 
"contact_type": "action➛incident➛Contact type",
"created": "action➛incident➛Created",
"created_by": "action➛incident➛Created by",
"number": "action➛incident➛Number",
"priority": "action➛incident➛Priority",
"severity": "action➛incident➛Severity",
"short_description": "action➛incident➛Short description",
"sys_id": "action➛incident➛Sys ID",
"urgency": "action➛incident➛Urgency"
}}
```
10. Once you are happy with your body, hit **Save** in the top right, and then **Publish** (if you don't publish, you will not be able to see your Action when you are creating your Flow!)
### Setting up a Flow
1. Now that you have an Action created, it is time to define a Flow. Next to the tab of your newly created Action, click the **+** button, and select **New Flow**
2. Give your Flow a Name and a Description, then click **Submit**
3. Under *Triggers*, click **Click to add a Trigger**, and select the kind of Trigger you want (for example, use *Created or Updated* to fire the Flow whenever a table entry is created or updated, combined with whatever conditions you define).
4. Add any conditions you would like to add (for example, Assignment group => changes would fire the action or flow logic whenever an incident is created or updated where the assignment group changes), set **Run Triggers** to *Always*, then click **Done**
5. Now that you have a Trigger, you can add the Action using the Trigger as the input. Click **Click to add Action or Flow Logic**, select the Action you just created, then use the *Data Pill Picker* to select the Trigger input, and click **Done** to save it
There you have it! Once you have made all the changes you want to the body and the trigger, make sure to test your flow with the **Test** button, using any incident to test the integration, then hit **Save** and **Activate** to activate your integration!

## xMatters set up
1. More help on importing a [communication plan](http://help.xmatters.com/OnDemand/xmodwelcome/communicationplanbuilder/exportcommplan.htm)
2. If you would like to create your own communication plan, it is recommended to use [this](


## ServiceNow set up
Some screenshots to help:
```
<kbd>
  <img src="media/cat-tax.png" width="200" height="400">
</kbd>
```

<kbd>
  <img src="media/cat-tax.png" width="200" height="400">
</kbd>


# Testing
To test the Flow, you can use the Test button and add an input incident, then in xMatters navigate to the communication plan and hit **Edit** > **Integration Builder**, then on the righthand side of the Inbound integration to ServiceNow IntegrationHub, click the gear icon and select **Activity Stream**. In the Activity Stream, you should be able to see a successful POST and an email should be sent to the recipients.

# Troubleshooting
The most difficult part about setting up an integration with IntegrationHub using the "Content-Type: application/json" header is definitely setting up the body. Using the click and drag GUI has its advantages, but it makes the body very difficult to reproduce. Some pointers:
1. Make sure the format is 
```
{"recipients": "[Recipient group]",
"properties": {
"[Property 1 name]": "[Property 1 value]",
"[Property 2 name]": "[Property 2 value]",
.....
"[Property n name]": "[Property n value]"
}}
```
Each [property name], you can type in yourself, but each [property value], you have to set it up with two double quotes (e.g. ```"assignment_group": ""```), place the cursor between the two double quotes, then click and drag the property value (dropdown menu under incident > click and drag "assignment group" to the cursor)
