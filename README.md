# Planner-PowerBI-with-PowerAutomate
*** Jan 2024 - IMPORTANT - due to a change in the flow import UI you MUST add the following parameter onto the URL **?v3=false** during the import. ***
# Synching Planner Plans to Power BI using Power Automate

## Introduction

This repository contains files and information required to sync the tasks from Microsoft Planner to an Excel file stored in SharePoint, which can then be read by Power BI; so in simple terms it provides an automatic sync between all Planner plans and Power BI.  The orgininal flow has been amended several times, for an overivew of the whole process you need to watch both the original video @ https://youtu.be/-U1Nnj95VMo and then follow it up with https://youtu.be/82aYZBKiwoA

The repository contains **four** versions of the PowerAutomate flow.  

1. The original flow (as per the youtube videos reference above) and is available in the **Parallel with no Office Scripts** folder.  

2. The next modification was to use Office Scripts and this is available in the **Parallel with Office Scripts** folder - https://youtu.be/82aYZBKiwoA . Anecdotally the flow with Office Scripts is more stable and runs faster.

3. The folder called **Sequential** is modification of the Office Scripts flow, however it consists of just a single branch in the flow.  Some people have found that this is the fastest flow, but on my system this is inconclusive.

4. The folder called **Single Team** allows you to just sync Planner tasks for a single Office Team.  It is based on the **Parallel with Office Scripts** flow.

The following files are included in each folder 

1. An exported Flow package that copies the Planner data into an Excel file.
2. The office script files that are used to empty the tables in Excel (if necessary)
3. A "map" of the flow as documentation.

Additionally, the following two files are common across all of the flows
1. An Excel file that holds the Planner data.
2. A Power BI template (.pbit) file that visualizes the Planner data.

## Deployment Overview

The following outlines the deployment and configuration process to deploying the solution, and assumes you are using the flow with Office Scripts.

1. Associate each Plan with a Team in Microsoft Teams
2. Upload the Excel file into a SharePoint document library
3. Import the Flow package into Power Automate  *** Jan 2024 - IMPORTANT - due to a change in the flow import UI you MUST add the following parameter onto the URL /?v3=false during the import. ***
4. Add the scripts to the automate tab in Excel online, and test that they work (assuming you are using one of the flows that needs an Office Script)
5. Update the flow to point to the SharePoint document library.
  1. Test, test, test, until the flow runs successfully!!
6. Open the .pbit file and enter the web URL of the Excel file

## Associate each Plan with a Team in Microsoft Teams.

PowerAutomate queries every Team in turn and returns the Planner plans associated with the Team; therefore if no Team exists for a Plan then a Team must be created and associated with the existing Office365 group that was created as part of the Plan creation. If a team already exists, and the Plan is being created, associate the Plan with the relevant team.

![2021-10-10_20-27-16](https://user-images.githubusercontent.com/37085234/136714533-f194727f-a55a-4d4a-969a-7233b7afabe3.png)

_Figure 1 - Create a Team and associate it with the existing Group created when the Planner Plan was created_

![2021-10-10_20-20-25](https://user-images.githubusercontent.com/37085234/136714433-caa51832-131f-4711-888c-16ed78ade5ab.png)
_Figure 2 - Create a new Plan against and existing Group_

### Store the Excel file in a SharePoint folder.

Upload the Excel file into a SharePoint folder. It doesn&#39;t matter what the folder or file name is, but make sure you remember where you have saved the file as this will need to be entered multiple times in the Flow. Please leave the table names the same, as Power BI uses hard coded table names as part of the query.

### Update the Excel file with the scripts.
Create the new scripts against the Excel file by copying and pasting the code from each script file into the scripts code editor in Excel.

### Import the Flow into Power Automate

Open Power Automate and import the Flow package. The Flow is a cloud flow that is set to run each night at 11:00pm.

You will need to ensure you configure and select connections for the following resource types.

| **Connection Name** |
| --- |
| Microsoft Teams Connection |
| Excel Online (Business) Connection |
| Planner Connection |
| Office 365 Groups Connection |

If you cannot see a relevant connection, then open the Data > Connections tab, and click the button to add a new connection at the top of the page.

If get the following error when you import the flow then you have more configuration to perform.

![parameters](https://user-images.githubusercontent.com/37085234/153899782-3cc6a372-07ec-4bdb-bf30-dc0f3a8ab6ff.png)

Please see this video on YouTub to walk you through the solution - https://www.youtube.com/watch?v=oWyDJmDDFfU 

Alternatively please review the relevant flow map.pdf file included in the repository to manually create the flow.

### Update the flow to point to the SharePoint document library

The Flow has 6 major sections to it. They comprise of sections to update the following tables in the Excel file, and each section is broadly similar.

| **Table Name** | **Description** |
| --- | --- |
| Plan\_tbl | Holds the Plan Id and Plan Name |
| Bucket\_tbl | Holds the Plan Id, Bucket Id and Bucket Name |
| Task\_tbl | Holds the Task Id, Plan Id, Bucket Id and many other task fields |
| Assignments\_tbl | Holds the Task Id, Plan Id and UserAssignmentId |
| User\_tbl | Holds the User Id and User Name |
| Checklist\_tbl | Holds the Task Id and the Checklist item(s) |

Each table is updated as part of the Flow. The following steps are applied in each section

1. Every row in the current table is deleted (via the office scripts)
2. A list of Plans is retrieved for each group associated with each team in Microsoft Teams
3. The relevant data for each plan is written into the relevant table, with the exception of the User\_tbl which lists the group members for each group, and therefore does not query Planner at all.

Once the Flow has been imported, edit the Flow so that each section points to the location where the Excel file is located in SharePoint. Note that this needs to be done at least three times for each section, one to list all the rows in the table, once to remove the rows from the table, and once to add rows to the table. Specifically the following items need to be set.

| **Entity** | **Description** |
| --- | --- |
| Location | This is the location in SharePoint, eg _SharePoint Site – \&lt;site name\&gt;_ |
| Document Library | A drop down list. Choose the library where the file is located |
| File | A drop down list. Choose the filename, eg _/Planner tasks from flow.xlsx_ |
| Table | The relevant table for the section. eg _Task\_tbl, Plan\_tbl etc_ |


The following image gives and example of the UI in Flow for configurating the Excel details.
![2021-10-10_23-14-01](https://user-images.githubusercontent.com/37085234/136714595-c41e6c6b-154f-45a7-a754-eebe1f7e587a.png)

_Figure 3 - Update the details for the Excel file_

Note that the specific details for matching fields for each entity in Planner to the relevant columns in each table do not have to be configured, and indeed more columns exist in the tables than are actually configured.  

### Configure the .pbit file

Open the .pbit file and enter the web URL of the Excel file. This can be found by opening the Excel file from the SharePoint site in the desktop app (ie in Excel) and clicking on File and then Info. Click on the button that says Copy Path and then paste in into the ExcelFileWebURL parameter when the Power BI template loads. Remove the ?web=1 from the end of the URL, leaving you with a URL in the following format

https://tenantname/sitename/foldername/filename.xlsx

Publish the Power BI file and then set up a scheduled refresh on the dataset. My recommendation is to set this to run two hours after the schedule for the Power Automate Flow, especially if you have a large number of tasks.  The default time for the Flow is 11:00pm, so I recommend setting the Power BI Scheduled Refresh to 1:00am, or of course, you could modify the flow to refresh the Power BI file as the final step.
