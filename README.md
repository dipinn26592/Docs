# Lab deployment guide for Microsoft Graph Call Records API using Logic Apps

This blog is authored and technically implemented with the help of [**Phi-lay Nguyen**](https://github.com/phnguy). Thanks a lot :)

## Summary 

The following is a step-by-step guide for using Azure Logic Apps to leverage the Microsoft Graph Call Records API to solve customer challenges using CDR information.  

Reference Article for [Call Records API](https://docs.microsoft.com/en-us/graph/api/resources/callrecords-api-overview?view=graph-rest-beta )

### Disclaimer 

The guide is provided "as is", without warranty of any kind, express or implied, including but not limited to the warranties of merchantability, fitness for a particular purpose and noninfringement. in no event shall the authors or copyright holders be liable for any claim, damages, or other liability, whether in an action of contract, tort or otherwise, arising from, out of or in connection with the software or the use or other dealings in the software. 

### Lab Environment 

To follow this guide, you will need the following: 

1. Access to a tenant with paid Azure subscription associated to your O365 tenant where you can create Azure Application
2. Azure Logic Apps and Azure log analytics workspace. 
3. You will also need a valid Office 365 subscription which will be used to make and receive calls. 
 

### High Level Overview 

In this solution we will create 2 logic Apps. The first one will be used to create\renew a webhook subscription and the second one will be used to retrieve the call ID, post which the call ID's are save to an Azure SQL Data Base. 

#### Reference Documents 
https://docs.microsoft.com/en-us/graph/api/resources/callrecords-api-overview?view=graph-rest-1.0  
https://docs.microsoft.com/en-us/graph/api/callrecords-callrecord-get?view=graph-rest-1.0&tabs=http  


<img width="600" alt="Picture1" src="https://user-images.githubusercontent.com/41346103/169072494-1d579116-5132-4768-8bd8-b41d90dcdd7e.png">



## Deployment Steps 

 
### Step 1: Register AD Application 

- In a tenant where you have **Global Admin permissions**, sign-in to [Azure Portal](https://portal.azure.com)
- Navigate to Azure **Active Directory** > **App registrations**
- Click + **New registration**
- Provide a Name for your app (example: ***CallrecordsId***)
- Supported account types **“Single tenant”**
  `If you have subscription associated with different tenant select All Microsoftaccount Users`
- Click **Register**
- After app is registered, document the following
  - **Application (client) ID**: {guid}
  - **Directory (tenant) ID**: {guid}
- In the left rail, navigate to **Certificates & secrets**
- Click + New **client secret**
- After new secret is generated, document the following 
  - **Client secret**: {string} 
- In the left rail, navigate to **API permissions** 
- Click + **Add a permission**
  - Click **Microsoft Graph** 
  - Click **Application permissions** 
  - Expand **CallRecords** (1) and check the box for **CallRecords.Read.All**
  - Expand **User** (1) and check the box for **User.Read.All** 
  - Expand **OnlineMeeting**(1) and check all the boxes  
  - Expand **Calls** (1) and check all the boxes 
- Click **Add permissions**
- Remove any other permissions automatically added via App registration process 
- Finally, click **Grant admin consent** for {tenantName} 
- Output should look like the following

![Picture2](https://user-images.githubusercontent.com/41346103/169073234-20624b32-a12a-4ef9-94c5-0c59294d7574.png)
 
 ### Step 2.a: Register AD Application 
 
 - In a tenant where you have a paid Azure subscription, sign-in to [Azure Portal](https://portal.azure.com )
- Navigate to **Logic Apps** 
- Click **+ Add** 
  - Select the appropriate **Azure subscription** (example: Visual Studio Enterprise) 
  - Provide a name for your **new Resource group** (example: CallDataCollector_rg) 
  - Provide a name to **Logic App** as *“Webhooksubscribe”*
  - Select **Region**
  - Plan type **Consumption**
  - Other options are optional 
- Click **Review + create**
  - Once created, skip the Go to resource option for now 
 
 ### Step 2.b: Create Logic app to retrieve a call ID
 - Follow the same steps in 2.a and create another logic app with the name ***“RetrieveCallid”*** 
 
 Once done you should have 2 logic apps looking like below.  
 
 ![Picture3](https://user-images.githubusercontent.com/41346103/169073284-3b4ad374-9c51-46a7-b3ee-c304d0a54fd3.png)
 
  ### Step 3: Configure Logic Apps ( RetrieveCallId )
 
- Open logic app ***“RetrieveCallId”*** 
- Click **Logic app designer** in left rail 
- From the default template select When a **HTTP request** is received 
 
 <img src= "https://user-images.githubusercontent.com/41346103/169073299-604b79e7-d82b-45cc-bfcf-52e58ecea0c6.png" width="500" height="250">
 
- Click **Save** 
- Once saved, service will populate the **HTTP POST URL** 
- Under Request Body JSON Schema copy/paste the text from file ***HTTP_JSON_Schema.txt*** file  
 
Reference screenshot after this step : 
 
  <img src= "https://user-images.githubusercontent.com/41346103/172828928-b72bdac5-776a-49e9-8cad-101b680f2ff2.png" width="500" height="280">
  
 
- Add **New Step**
- Search for **Data Operations** Built-in and **Compose** 
- Rename the Compose windows to *“ValidationToken”* 
- Under Inputs Select **Expression** and type in  
 
`trigger()?['Outputs']?['Queries']?['ValidationToken'] `

- Click **OK** 
 
 ![Picture6](https://user-images.githubusercontent.com/41346103/169073365-c3c0fc35-ebe2-4a27-9959-ec4f1e912851.png)

 
- Add **New Step** > **Data Options** > **Compose**
- Rename the Compose window to **Client ID** 
- Input the **Client ID** from app registration (Step 1)  
 
- Add **New Step** > **Data Options** > **Compose**
- Rename the Compost window to **Tenant ID** 
- Input the **Tenant ID** from app registration (Step 1)  

- Add **New Step** > **Data Options** > **Compose**
- Rename the Compose window to **Secret**
- Input the **Secret** from app registration (Step 1)  

- Add **New Step** > **Control** > **Condition**
- Under Choose a value select Outputs from **Validation Token**
- Set the condition as is **not equal to**
- Point it to Expression **null**
 
 ![Picture7](https://user-images.githubusercontent.com/41346103/169073361-f8b12027-5707-459e-a72a-34af065e511d.png)


- If **True** > **Add an action** > **Responses**
   - Put the value of **Status Code** as **200** 
   - Keep the **Headers** as Empty 
   - Under **Body** select Outputs from **Validation Token**
 
 <img src= "https://user-images.githubusercontent.com/41346103/172829422-670c8452-1c00-46e3-8858-1c26658287bd.png" width="500" height="280">
   
 
 
 - If **False** > 
 
 >"Save the Call ID to the SQL database ( can be skipped )"
 
 >Steps to set up Azure SQL server and database [here SQLConnection.md](https://github.com/dipinn26592/Call-Records-API-Lab/blob/dipinn26592-patch-1/SQLConnection.md) 
 
 - **Add a Action** > **Built-in Control** > **ForEach**  
 - Under **ForEach** loop  
 - Enter Expression as `triggerBody()?['value'] `
 - Add **New Step** > **SQL** > **Insert row (V2)**
 - Select your **Server Name, Database name, Table name**
 - Select **Add new parameter**
 - Check Box **id** and Select **id** from when a HTTP request is received
 -
<img src= "https://user-images.githubusercontent.com/41346103/172836508-d8b4f831-73db-4798-b466-188fe677ba63.png" width="500" height="280">


>"Make HTTP Call against CallID and save the detailed output to Database ( can be skipped )"

 - **Add a Action** > **Built-in Control** > **ForEach**  
 - Under **ForEach** loop 
 - Enter Expression as `triggerBody()?['value'] `
 - Add **New Step** > **HTTP** 
  - Method : **GET** 
  - URI : *https://graph.microsoft.com/v1.0/communications/callRecords/[id]?$expand=sessions($expand=segments)*
  - Hover over Id you should see **items('DetailedCallRecords')?['resourceData']?['id']**
 

 ![image](https://user-images.githubusercontent.com/41346103/172834798-0e83f3f7-bf77-4ee6-9463-92be9032a290.png)
 
  - Select **Authentication Type** > Checkbox 
  - Authentication type: **Active Directory OAuth**
  - Authority: *https://login.microsoftonline.com*
  - Tenant: Output from previous compose of **TenantID** 
  - Audience: *https://graph.microsoft.com*
  - Client ID: Output from previous compose of **ClientID** 
  - CredentialType: **Secret** 
  - Secret: Output From previous compose of **Secret** 
 
- Add **New Step** > **Data Operations** > **Parse JSON** 
  - Input Content as Output **“Body”** from HTTP 
  - Schema: Copy/paste text from ***ParseJson.txt*** file 

- **Add a Action** > **Built-in Control** > **ForEach**  
 - Under **ForEach** loop  
 - Enter Expression as `triggerBody()?['value'] `
 - Add **New Step** > **SQL** > **Insert row (V2)**
 - Select your **Server Name, Database name, Table name**
 - Select **Add new parameter**
 - Select the Parameters as per SQL Table that you have specified.

- Outside the Foreach loop Add **New Step** > **Response** 
  - Status Code: **202** 
  - Body: Output from **Validation Token** 
 
 ### Step 4: Configure Logic Apps (Webhooksubscribe) 

- Go to Logic Apps and Open login app with name *Webhooksubscribe*
 - Navigate to **Logic App designer** 
 - Search **Schedule** > **Recurrence** 
 - Set Interval as **8** and Frequency as **hour** 

- Add **New Step** > **Data Options** > **Compose** 
- Rename the Compose window to **NotificationURL** 
- Input the **Notification URL** we got from previous logic app ***RetrieveCallID***

- Add **New Step** > **Data Options** > **Compose** 
- Rename the Compose window to **Client ID** 
- Input the **Client ID** from app registration (Step 1)  

 - Add **New Step** > **Data Options** > **Compose** 
- Rename the Compos window to **Secret** 
- Input the **Secret** from app registration (Step 1)  

 - Add **New Step** > **Data Options** > **Compose** 
- Rename the Compose window to **Tenant ID** 
- Input the **Tenant ID** from app registration (Step 1)  

- Add **New Step** > **Data Options** > **Compose** 
- Rename the Compose window to **ExpiryTime** 
- Input the value as **Expression**  
  
 `AddDays(utcNow(),1)`
 
 - Add **New Step** > **Data Options** > **Compose** 
 - Rename the Compose window to **SubscriptionBody** 
- Input the Value as below 

```
{ 
“changeType”: “created”, 
“clientState”: “secretClientValue”, 
“expirationDateTime”: “Output from previous compose expirationDatetime”, 
“notificationURL”: “Output from previous compose NotificationURl”, 
“resource”: “communications/callRecords” 
} 
 ```
 *Note: Copy paste might not work, you might have to type the function in correct format*
 
![Picture9](https://user-images.githubusercontent.com/41346103/169073353-dde9b038-6b58-46a7-98e6-5d2a54928980.png)

 
- Add New Step > **HTTP**  
  - Input Method as **POST** 
  - Input URI as ***https://graph.microsoft.com/v1.0/subscriptions/*** 
  - Leave Header and Queries Empty 
  - Input Body Outputs from **SubscriptionBody**
  - Add new parameter and checkbox **Authentication** 
  - Input Authentication as **Active Directory OAuth** 
  - Input Authority as ***https://login.microsoftonline.com*** 
  - Input **Tenant ID** from output from Compose  
  - Input Audience as ***https://graph.microsoft.com***  
  - Input value of ***Client ID*** as output from Compose 
  - Set Credential Type as **Secret** 
  - Input value of **Secret**t as output from Compose  
 
 - Run the logic app and validate the output.  
- If successful, you will see an output with Subscription ID as below. 
 
 ![Picture10](https://user-images.githubusercontent.com/41346103/169073351-5825550d-a74a-4554-b010-13b92e935a58.png)

 
- Open Logic App ***“Webhooksubscribe”*** again. 
- Navigate to **HTTP**  
- Replace the method from **POST** to **PATCH** 
- Update the URI and append the **subscription ID** which we received (screenshot for reference below)
 
 
 ![Picture11](https://user-images.githubusercontent.com/41346103/169073347-5ff6d8e5-a8ae-48b4-84a0-94c28b73aaa3.png)


 - Run the logic app again, validate if the renew of subscription was successful.  
 
 ![Picture12](https://user-images.githubusercontent.com/41346103/169073344-49b40fcc-b6c7-4eec-82ae-7a9a50fea16d.png)

 
 **Reason for Editing HTTP** : Subscription ID will remain same, and we need to patch the Subscription ID at regular intervals to renew the subscription to keep getting the data.  
 
 ### Step 5: Run Logic App 
- Run Logic App ***Webhooksubscribe***
- You should see the flow running successfully 

### Step 6: Make test calls and see call records. 
- Make test calls through Teams client. 
- Navigate to logic app RetrieveCallID 
- Check if last run was successful 
 
 Output Below 
 
<img width="292" alt="Picture13" src="https://user-images.githubusercontent.com/41346103/169073340-d978d2e4-f86a-4993-a85c-d0d05c2b2b2c.png">

SQL Server Output
<img width="1235" alt="image" src="https://user-images.githubusercontent.com/41346103/172856562-206118de-f912-4ff1-9316-e8bb9683f835.png">


## How to reduce cost of Storing the data in Azure
1. Set up your SQL Server OnPremise
2. Create and Expose a internet facing Webhook to listen to Subscription Output

