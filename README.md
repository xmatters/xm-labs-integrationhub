# ServiceNow Inbound (to xMatters) Integration via IntegrationHub
IntegrationHub is a feature in ServiceNow that makes integrating to third-party (in this example, xMatters) apps easier than ever before, and the best part about it? No coding! Well, at least minimal... With this inbound integration, you can set up a trigger from any (that's right, any!) table and table entry to create an event in xMatters.

<kbd>
<a href="https://support.xmatters.com/hc/en-us/community/topics">
   <img src="https://github.com/xmatters/xMatters-Labs/raw/master/media/disclaimer.png">
</a>
</kbd>

# Pre-Requisites
* ServiceNow instance, with IntegrationHub purchased and installed - if you don't have IntegrationHub, see details on how to get it [here](https://docs.servicenow.com/bundle/kingston-servicenow-platform/page/administer/integrationhub/tasks/request-integrationhub.html)
* Existing workflow - Use this packaged integration from the integrations page, or build your own
* xMatters account - If you don't have one, [get one](https://www.xmatters.com)!

# Files
* [ExampleWorkflow.zip](ExampleWorkflow.zip) - This is an example workflow to help get started. 
* [EmailMessageTemplate.html](EmailMessageTemplate.html) - This is an example HTML template for emails and push messages. 
* [Payload](PayloadBody.png) - An example payload body to use in ServiceNow

# How it works
Add some info here detailing the overall architecture and how the integration works. The more information you can add, the more helpful this sections becomes. For example: An action happens in Application XYZ which triggers the thingamajig to fire a REST API call to the xMatters inbound integration on the imported Workflow. The integration script then parses out the payload and builds an event and passes that to xMatters. 

# Installation
## In xMatters:
### Setting up the Workflow
1. Download the [zip file](ExampleWorkflow.zip) onto your computer
2. Log in to xMatters and navigate to the **DEVELOPER** tab
3. Click **Import Plan** and select the zip file you just downloaded, then click **Import Plan**
4. On the righthand side of the Workflow you just imported, click **Edit** > **Integration Builder**
5. Next to **Inbound Integrations**, open the dropdown menu by clicking the blue **1 Configured**, then click **ServiceNow IntegrationHub Inbound**
6. At the bottom of the page, copy the request URL to something accessible
### Setting up an account to access the Workflow
1. Navigate to the **USERS** tab
2. Click **Add** on the righthand side
3. Give the user a first and last name (e.g. ServiceNow IntegrationHub), user ID (username used for logging in) and password, and give it the *REST Web Service User* role, then click **Add**
4. Navigate back to the **DEVELOPER** tab and next to the Workflow you just imported, click **Edit** > **Access Permissions**, then type the user ID of the account you just created, select the account, and click **Save Changes**. Alternatively, you can select **Accessible by All**, however this allows all users in your xMatters instance to view and edit the Workflow
## In ServiceNow:
### Option 1: Setting it up yourself
#### Setting up a Credential and Connection Alias:
Now that you have an xMatters Workflow, we need to set up a Connection alias in ServiceNow to connect quickly and easily to the Workflow
1. Log in to your ServiceNow instance

2. In the *Filter navigator*, search for "alias", and select **Connection & Credential aliases**
<kbd>
<img src="media/alias filter.png" width="300">
</kbd>

3. In the top left next to where you see **Connection & Credential aliases**, click **New**
<kbd>
<img src="media/new credential.png">
</kbd>

4. In the dropdown menu next to **Type**, select *Credential*, then give the Credential a Name, and hit **Submit**
<kbd>
<img src="media/create credential.png">
</kbd>

5. You will now see a list of Connections and Credentials; click (or search if you have many set up) the Credential you just created

6. Next to **Credentials** (right below the Credential *Name* and *Type*), click **New**, and select the *Basic Auth Credentials* option on the following page
<kbd>
<img src="media/add credential.png">
</kbd>

7. Give the Credential a *Name*, and enter the *User* and *Password* for the xMatters account with access to the Workflow, then click **Submit** to finish
<kbd>
<img src="media/define credential.png">
</kbd>

8. Now that you have a Credential, click the back button to return to the list of Connections & Credentials, then hit **New**
<kbd>
<img src="media/new connection.png">
</kbd>

9. This time, in the dropdown menu next to **Type**, select *Connection and Credential*, and in the dropdown menu next to **Connection Type**, select *HTTP*, then give the Connection a Name, and click **Submit**
<kbd>
<img src="media/create connection.png">
</kbd>

10. You will again see a list of Connections and Credentials; click (or search for) the Connection you just created
11. Next to **Connections** (right below the Connection *Name* and *Type*), click **New**
<kbd>
<img src="media/add connection.png">
</kbd>

12. Give the Connection a Name, then click the **Credential**, and select the Credential that you just created
<kbd>
<img src="media/create connection.png">
</kbd>

13. Paste the URL you copied from the xMatters Workflow into the **Connection URL** box, then Submit
<kbd>
<img src="media/define connection.png">
</kbd>

Now you have a working Connection Alias that we can use to easiliy communicate to xMatters! Now for setting up an Action
#### Setting up an Action
1. In the *Filter navigator*, search for "flow", and under *Flow Designer*, click **Designer**
<kbd>
<img src="media/flow filter.png" width="300">
</kbd>

2. You should see a list of flows. On the righthand side, click **New** > **New Action**
<kbd>
<img src="media/new action.png">
</kbd>

3. Give the Action a Name and Description, then **Submit**

4. In the Inputs, click **Create Input**, and give the input variable a name, then select the type and the table from which the input will be grabbed (for example, an input of type *Reference* from the table *Incident [incident]*)
<kbd>
<img src="media/create input.png">
</kbd>

5. Now for the magic. Between the Inputs and Outputs, click the **+** button to add a step. Click on **REST Step** (Note: you must have IntegrationHub installed for this to be available)
<kbd>
<img src="media/rest step.png">
</kbd>

6. Next to **Connection**, leave the default of *Use Connection Alias*, then select your Connection Alias in the dropdown menu next to **Connection and Credential Alias**
<kbd>
<img src="media/rest step connection.png">
</kbd>

7. From the dropdown menu next to **HTTP Method**, select *POST* (the Base URL is grayed out because it is already defined in the Connection Alias). We will not need to fill anything in for **Resource Path** in this example because the full URL from the Workflow is in the Connection Alias. However, if you would like to set up multiple integrations using IntegrationHub, it may be a good idea to define the URL in the Connection Alias as being ```https://example.xmatters.com/api/integration/1/functions/``` (with example being replaced with your instance), and then you could create multiple inbound integrations and have different actions use different inbound integration URLs by using the same Connection alias, and only change the Resource Path to ```[API key]/triggers``` .

8. For headers, use *Content-Type* under **Name**, and *application/json* under **Value**

9. Under body, you will have to enter each part of the input you would like to send to xMatters (this part isn't so much fun...). Here is a good example of an input body:
<kbd>
<img src="media/step body.png">
</kbd>

10. Once you are happy with your body, hit **Save** in the top right, and then **Publish** (if you don't publish, you will not be able to see your Action when you are creating your Flow!)
#### Setting up a Flow
1. Now that you have an Action created, it is time to define a Flow. Next to the tab of your newly created Action, click the **+** button, and select **New Flow**
<kbd>
<img src="media/new flow.png" width="500">
</kbd>

2. Give your Flow a Name and a Description, then click **Submit**
<kbd>
<img src="media/define flow.png">
</kbd>

3. Under *Triggers*, click **Click to add a Trigger**, and select the kind of Trigger you want (for example, use *Created or Updated* to fire the Flow whenever a table entry is created or updated, combined with whatever conditions you define).
<kbd>
<img src="media/add trigger.png">
</kbd>

4. Add any conditions you would like to add (for example, Assignment group => changes would fire the action or flow logic whenever an incident is created or updated where the assignment group changes), set **Run Triggers** to *Always*, then click **Done**
<kbd>
<img src="media/define trigger.png">
</kbd>

5. Now that you have a Trigger, you can add the Action using the Trigger as the input. Click **Click to add Action or Flow Logic**, select the Action you just created, then use the *Data Pill Picker* to select the Trigger input, and click **Done** to save it
<kbd>
<img src="media/add action.png">
</kbd>

<kbd>
<img src="media/add action input.png">
</kbd>

There you have it! Once you have made all the changes you want to the body and the trigger, make sure to test your flow with the **Test** button, using any incident to test the integration, then hit **Save** and **Activate** to activate your integration!

### Option 2: Installing an Update Set
1. Download the [update set](UpdateSet.xml)

2. Log in to your ServiceNow instance

3. In the *Filter navigator*, search for "update sets", and under "System update sets", click "Update Sets to Commit"
<kbd>
<img src="media/filter update.png" width="300">
</kbd>

4. Under *Related Links*, click **Import Update Sets from XML**
<kbd>
<img src="media/update sets.png">
</kbd>

5. Click the **Choose file** button, select the update set you downloaded and click **Open**. Next, hit the **Upload** button.
6. Now you should see a list of retrieved update sets. Click on the update set *IntegrationHub Demo*, then click **Preview Update Set**
<kbd>
<img src="media/choose update.png">
</kbd>
<kbd>
<img src="media/select update.png">
</kbd>
<kbd>
<img src="media/preview update.png">
</kbd>

7. After the preview loads, click **Commit Update Set** to add the update set to your ServiceNow instance

8. Now you should have an example Connection, Credential, Action, and Flow. First, using the *Filter navigator*, search for "alias" and click **Connection and Credential alias**
<kbd>
<img src="media/alias filter.png" width="300">
</kbd>

9. Click on **Example Credential**, then click on **xMatters Example** under the credentials list
<kbd>
<img src="media/example credential.png">
</kbd>
<kbd>
<img src="media/example user.png">
</kbd>

10. Change the *Username* and *Password* to the account that can access the Workflow in xMatters, and then click **Update**
11. Click the back button, and open the **Example Connection**, then click **New** next to the *Connections* list
<kbd>
<img src="media/example connection.png">
</kbd>

12. Give the Connection a *Name*, select the *Credential* you just changed, then fill in the *Connection URL* with the URL from your xMatters Workflow, and click **Update**
<kbd>
<img src="media/example connection builder.png">
</kbd>

13. Search for "flow" in the *Filter navigator*, and under *Flow Designer*, click **Designer**
<kbd>
<img src="media/flow filter.png" width="300">
</kbd>

14. Search for "Example" and click on **Example Flow**, then click **Edit Properties**, and rename the Flow
<kbd>
<img src="media/example flow.png">
</kbd>

15. Click on the trigger **[Incident] Created or Updated** and set the table to whatever table you would like to grab the input from. Now, set the trigger to what you would like it to be, then click **Done**
<kbd>
<img src="media/example trigger.png">
</kbd>
<kbd>
<img src="media/change trigger.png">
</kbd>

16. Click the Home button in the top left, then click on **Actions**, and search "Example", selecting **Example Action**, and rename the Action using **Edit Properties**
<kbd>
<img src="media/example action.png">
</kbd>

17. Change the input to the same table as the one selected in the Flow, then click on the **REST Step**
<kbd>
<img src="media/change action.png">
</kbd>

18. Verify that the correct *Connection and Credential Alias* is selected, then edit the body to send the desired values over to xMatters (see Troubleshooting section for more help on this)
There you have it! Make sure to hit **Publish** on the Action to commit the changes you make, then use the **Test** button in the Flow to troubleshoot body errors

## xMatters set up
### Change the endpoint
1. Navigate to the **DEVELOPER** tab, and locate the ServiceNow IntegrationHub Workflow
2. Next to the Workflow, click **Edit** > **Integration Builder**
3. Click **Edit Endpoints**, then click **ServiceNow**
4. Change the *Base URL* to the URL of your ServiceNow instance
5. Hit **Save Changes** then **Close**
### Change the default recipients
1. Open the *Inbound Integrations* dropdown, and select **ServiceNow IntegrationHub Inbound**
2. Click **Open Script Editor** near the center of the page
3. Navigate to line 21, and change the *Default Recipients* to the name of the xMatters group you would like to notify by default
4. Hit **Save**. Your IntegrationHub flow will now fire to xMatters and notify your default recipients with the response options defined in the form! Happy Integrating!


# Testing
To test the Flow, you can use the Test button and add an input incident, then in xMatters navigate to the workflow and hit **Edit** > **Integration Builder**, then on the righthand side of the Inbound integration to ServiceNow IntegrationHub, click the gear icon and select **Activity Stream**. In the Activity Stream, you should be able to see a successful POST and an email should be sent to the recipients.

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
