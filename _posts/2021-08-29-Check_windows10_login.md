---
layout: post
title: Windows Login History Event Log
---

I need to find out the login history of Windows 10.

[Reference] https://www.lepide.com/how-to/audit-who-logged-into-a-computer-and-when.html

### Enable event log for login

Below are the steps to enable auditing of user Logon/Logoff events

Step 1 – Open “Group Policy Management” console by running the “gpmc.msc” command.
Step 2 – If you want to configure auditing for the entire domain, right-click on the domain and click “Create a GPO in this domain, and Link it here…”.
Step 3 – Create a new GPO dialog box appears on the screen. Enter a new GPO name.
Step 4 – Go to the new GPO, right-click on it, and select “Edit” from the context menu.
Step 5 – “Group Policy Management Editor” window appears on the screen.
Step 6 – In the navigation pane, go to “Computer Configuration” ➔ “Policies” ➔ “Windows Settings” ➔ “Security Settings” ➔ “Local Policies” ➔ “Audit Policy”.

Figure : Configuring audit logon events policy
Step 7 – In the right pane, double-click “Audit logon events” policy to open its properties window.
Step 8 – Select the “Success” and “Failure” checkboxes, and click “OK”.
Step 9 – Similarly, you have to enable “Success” and “Failure” for “Audit Account Logon Events”.
Step 10 – Close “Group Policy Management Editor”.
Step 11 – Now, you have to configure this new Group Policy Object (containing this audit policy) on all Active Directory objects including all users and groups. Perform the following steps.
In In “Group Policy Management Console”, select the new GPO (containing above change).
In “Security Filtering” section in the right panel, click “Add” to access “Select User, Computer or Group” dialog box.
Type “Everyone”. Click “Check Names” to validate this entry. Click “OK” to add it and apply on all objects.

Figure : Applied the Group Policy Object to everyone
Step 12 – Close “Group Policy Management Console”.
Step 13 – Now, run following command to update GPO.
Step 14 – gpupdate /force

### Find login log in event viewer

After you have configured log on auditing, whenever users logon into network systems, the event logs will be generated and stored. To find out the details, you have to use Windows Event Viewer. Follow the below steps to view logon audit events:

Step 1 – Go to Start ➔ Type “Event Viewer” and click enter to open the “Event Viewer” window.
Step 2 – In the left navigation pane of “Event Viewer”, open “Security” logs in “Windows Logs”.
Step 3 – You will have to look for the following event IDs for the purposes mentioned herein below.
Event ID	Description
4624	A successful account logon event
4625	An account failed to log on
4648	A logon was attempted using explicit credentials
4634	An account was logged off
4647	User initiated logoff
For user logon, you have to search for 4624 and 4648 event IDs. For failed logon, you have to search for 4625. For logoff events, you have to search for 4634 and 4647.

In this article, we are searching for events 4624 and 4648. The following screenshot shows Windows Event ID 4648 for the user logon attempted using explicit credentials.


Figure : Logon event in Event Viewer

