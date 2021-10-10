# Planner-PowerBI-with-PowerAutomate

# Synching Planner Plans to Power BI using Power Automate

## Introduction

This repository contains files and information required to sync the tasks from Microsoft Planner to an Excel file stored in SharePoint, which can then be read by Power BI; so in simple terms it provides an automatic sync between Planner and Power BI.

The following files are included

1. An exported Flow package that copies the Planner data into an Excel file.
2. An Excel file that holds the Planner data.
3. A Power BI template (.pbit) file that visualizes the Planner data.

## Deployment Overview

The following outlines the deployment and configuration process to deploying the solution. A full YouTube video detailing the deployment process can be found here.

1. Associate each Plan with a Team in Microsoft Teams
2. Upload the Excel file into a SharePoint document library
3. Import the Flow package into Power Automate
4. Update the flow to point to the SharePoint document library
  1. Test, test, test, until the flow runs successfully!!
5. Open the .pbit file and enter the web URL of the Excel file

## Associate each Plan with a Team in Microsoft Teams.

Power Automate queries every Team in turn and returns the Planner plans associated with the Team; therefore if no Team exists for a Plan then a Team must be created and associated with the existing Office365 group that was created as part of the Plan creation. If a team already exists, and the Plan is being created, associate the Plan with the relevant team. The PDF version of this document contains relevant images.

![](RackMultipart20211010-4-1d0sggf_html_4f67f8762bc72a87.png)

_Figure 1 - Create a Team and associate it with the existing Group created when the Planner Plan was created_

![](RackMultipart20211010-4-1d0sggf_html_ce34be96f9a76d27.png)

_Figure 2 - Create a new Plan against and existing Group_

### Store the Excel file in a SharePoint folder.

Upload the Excel file into a SharePoint folder. It doesn&#39;t matter what the folder or file name is, but make sure you remember where you have saved the file as this will need to be entered multiple times in the Power Automate flow. Please leave the table names the same, as Power BI uses hard coded table names as part of the query.

### Import the Flow into Power Automate

Open Power Automate and import the Flow package. The flow is a cloud flow that is set to run each night at 11:00pm.

You will need to ensure you configure and select connections for the following resource types.

| Microsoft Teams Connection |
| --- |
| Excel Online (Business) Connection |
| Planner Connection |
| Office 365 Groups Connection |

### Update the flow to point to the SharePoint document library

The flow has 5 major sections to it. They comprise of sections to update the following tables in the Excel file, and each section is broadly similar.

| **Table Name** | **Description** |
| --- | --- |
| Plan\_tbl | Holds the Plan Id and Plan Name |
| Bucket\_tbl | Holds the Plan Id, Bucket Id and Bucket Name |
| Task\_tbl | Holds the Task Id, Plan Id, Bucket Id and many other task fields |
| Assignments\_tbl | Holds the Task Id, Plan Id and UserAssignmentId |
| User\_tbl | Holds the User Id and User Name |

Each table is updated as part of the Power Automate flow. The following steps are applied in each section.

1. Every row in the current table is deleted
2. A list of Plans is retrieved for each group associated with each team in Microsoft Teams
3. The relevant data for each plan is written into the relevant table, with the exception of the User\_tbl which lists the group members for each group, and therefore does not query Planner at all.

Once the Flow has been imported, edit the Flow so that each section points to the location where the Excel file is located in SharePoint. Note that this needs to be done at least twice for each section, once to select the table to remove rows from, and once to select the table to add rows to. Specifically the following items need to be set.

| **Entity** | **Description** |
| --- | --- |
| Location | This is the location in SharePoint, eg _SharePoint Site – \&lt;site name\&gt;_ |
| Document Library | A drop down list. Choose the library where the file is located |
| File | A drop down list. Choose the filename, eg _/Planner tasks from flow.xlsx_ |
| Table | The relevant table for the section. eg _task\_tbl, plan\_tbl etc_ |

### Configure the .pbit file

Open the .pbit file and enter the web URL of the Excel file. This can be found by opening the Excel file from the SharePoint site in the desktop app (ie in Excel) and clicking on File and then Info. Click on the button that says Copy Path and then paste in into the ExcelFileWebURL parameter when the Power BI template loads. Remove the ?web=1 from the end of the URL, leaving you with a URL in the following format

https://\&lt;tenantname\&gt;/\&lt;sitename\&gt;/\&lt;folername\&gt;/\&lt;filename\&gt;.xlsx

Publish the Power BI file and then set up a scheduled refresh on the dataset. My recommendation is to set this to run at least one hour after the schedule for the Power Automate Flow, especially if you have a large number of tasks.
