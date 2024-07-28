# MMUH Deployment Dashboard

## Overview

The MMUH Deployment Dashboard is a comprehensive Power BI project designed to track the deployment progress of various devices such as EUD (End User Devices), MFD (Multi-Functional Devices), AV (Audio Visual equipment), and 724 machines at the Midland Metropolitan University Hospital. The dashboard provides real-time insights, helping management monitor the deployment process, track team performance, and ensure timely completion.

## Introduction
The MMUH Deployment Dashboard was developed to provide a clear and concise overview of the device deployment status within the new Midland Metropolitan University Hospital. It tracks the progress of deployment, team performance, and device stock levels, helping management make informed decisions and ensure timely project completion.

## Main Dashboard Overview
The main dashboard offers a holistic view of the entire project:

- Project Completion Percentage: Displays the total project completion status.
- Schedule Status: Indicates whether the project is on track, behind schedule, or ahead of schedule with a colour tag.
- Burndown Chart: Visual representation of the remaining work over time.
- Floor Completion Bar Chart: Shows the percentage of completion and remaining work by floor.
- Device Type Completion: Four pie charts representing the completion percentage of EUD, MFD, AV, and 724 devices.
![MMUH Dashboard](https://github.com/user-attachments/assets/4fb86605-1037-4521-9777-e711dc1d3d80)


## Device-Specific Dashboards
### EUD Dashboard

- Device Count and Types: Displays the total and deployed counts of different EUDs (monitors, desktops, printers, scanners, etc.).
- EUD Completion Percentage: Overall completion percentage of EUDs.
- Schedule Status: Indicates if EUD deployment is on schedule, behind, or ahead.
- EUD Burndown Chart: Tracks the remaining work specific to EUDs.
- Floor-Based Completion: Bar chart and table showing deployment status by floor, with the ability to drill down to department-level details.
![EUD](https://github.com/user-attachments/assets/96e37f9c-a6a3-448c-9bf8-e1eca57a0a85)

## AV, 724, and MFD Dashboards
Similar structure as the EUD Dashboard, with specific details for each device type:

- Total Completion Status
- Burndown charts similar to EUD, but specific to device type.
- Floor-based percentage of completion and remaining, specific to device type.
- Table showing the count and percentage of completion by floor, with the ability to drill down to departments.
![AV](https://github.com/user-attachments/assets/0acf49d1-6340-4633-b2d2-69bbf16b6391)

## Stock Dashboards

Stock Dashboard: Displays the stock count of different devices.
## Team Performance Dashboards

Team Rollout Info Page: Analyzes the average number of devices deployed by each team, categorized by device type.

## Data Sources and Preparation

- Data is sourced from an Excel sheet, including details like asset numbers, device types, locations, and scheduled deployment dates.
- Engineers update the deployed date once the device is installed.
- The data refresh is scheduled hourly to ensure that the dashboard displays the most up-to-date information.

## Technologies and Tools Used

### Power BI: 
- Use To extract the data.
- Used to design the dashboard.
- Used to Model the data.
![Data Modeling](https://github.com/user-attachments/assets/54106dfa-45ef-4453-a011-1bb3b6bb6c8d)
### DAX: 
Complex DAX formula has been created for calculations, burndown chart, and status tracking. The complex DAX formula are discuse below.
#### DAX Formula for Burndown
The below formula is used to get date from all table.
```
Dashboard_Burndown = CALENDAR(MIN(MIN(MIN
    ('EUD Date Range'[Date]),MIN(
        'MFD Date Range'[Date])),MIN(MIN(
        '724 Date Range'[Date]),MIN('AV Date Range'[Date]
            )
        )
    ),
    MAX(MAX(MAX(
        'EUD Date Range'[Date]),MAX(
            'MFD Date Range'[Date])),MAX(MAX(
                '724 Date Range'[Date]),MAX(
                    'AV Date Range'[Date]
                    )
                )
            )
        )
```
#### DAX code for Index
The below formula is used to create a custom coloum for unique id for all date.
```
Index Dashboard = RANKX(
    'Dashboard_Burndown','Dashboard_Burndown'[Date],,ASC
    )
```
#### DAX formula for planned deployment
The below formula is used to get count of planned deployment for the burndown chart.
```
Planned Deploy Dashboard = COUNTROWS(
    FILTER(
        'EUD','EUD'[Scheduled Date] >= EARLIER(([Date]))
    )) + COUNTROWS(
        FILTER(
            '724', '724'[Scheduled Date] >= EARLIER(('Dashboard_Burndown'[Date]))
        )) + COUNTROWS(
            FILTER(
                'AV', 'AV'[Scheduled Date] >= EARLIER(('Dashboard_Burndown'[Date]))
            )) + COUNTROWS(
                FILTER(
                    'MFD', 'MFD'[Scheduled Date] >= EARLIER(('Dashboard_Burndown'[Date]))
            )
        )
```
#### DAX formula for Deployed count
The below formula is used to get count of deployment count for the burndown chart.
```
Deployed Devices = IF(
    'Dashboard_Burndown'[Date] <= TODAY(), 
        COUNTA('EUD'[Scheduled Date]) 
            + COUNTA('724'[Scheduled Date]) 
                + COUNTA('MFD'[Scheduled Date])
                    + COUNTA('AV'[Scheduled Date])
                        - SUMX(FILTER('Dashboard_Burndown','Dashboard_Burndown'[Index Dashboard]< EARLIER('Dashboard_Burndown'[Index Dashboard])),'Dashboard_Burndown'[Total deployed] ))
```
#### DAX formula to Count Total deployment
The formula below is used to calculate the Total deployment count for the burndown chart.
```
Total deployed = 
    COUNTROWS(
        FILTER(
            'EUD',
            'EUD'[Deployment Date] = EARLIER('Dashboard_Burndown'[Date])
        )
    ) + COUNTROWS(
            FILTER(
            '724',
            '724'[Deployment Date] = EARLIER('Dashboard_Burndown'[Date])
            )
        ) + COUNTROWS(
                FILTER(
                'MFD',
                'MFD'[Deployment Date] = EARLIER('Dashboard_Burndown'[Date])
                )
            ) + COUNTROWS(
                    FILTER(
                    'AV',
                    'AV'[Deployment Date] = EARLIER('Dashboard_Burndown'[Date])
                    )
                )

```
#### Formula to count the percetage
The below formulas are used to count the percentage of project completed and remaining project completion. This measures can be used with multiple charts.
```
Total Completion % Dashboard = DIVIDE([Total Deployed] - [Total Target], [Total Target])
```
```
Deployed Dashboard % = MIN ( 1 + Dashboard[Total Completion % Dashboard], 1)
```
```
Remaining Dashboard % = MAX( - Dashboard[Total Completion % Dashboard],0)
```
#### Schedule Status
The "Status EUD" is a custom column used to determine whether the schedule date of the respective device is in the future or past. The shown formula is of EUD but the same formula is used in other category.
```
Status EUD = IF(EUD[Agreed/Additional] = "Agreed",
                IF(
                    'EUD'[Scheduled Date] = TODAY() , 
                    "Past", 
                        IF('EUD'[Scheduled Date] > TODAY(), 
                        "Future", 
                        "Past"
                        )
                )
            )
```
The DAX formula below is used to count the past schedule dates from the respective date, which can then be used for the Schedule Colour and Schedule Label DAX formulA.
```
Count Past EUD = CALCULATE (
            COUNTROWS ( EUD ) ,
            EUD[Status EUD] = "Past"
        )
```
The formula below is used to show the colour and tag based on the current status of the respective project.
```
Schedule Colour EUD = SWITCH(TRUE(),
    EUD[Count Past EUD] > EUD[Total Deployed],
    "#FF5555",
    EUD[Count Past EUD] < EUD[Total Deployed],
    "#007FFF",
    "#00ff00"
)
```
```
Schedule Label EUD = IF(
    EUD[Count Past EUD] < EUD[Total Deployed],
        "Ahead of Schedule",
            IF(
                EUD[Count Past EUD] > EUD[Total Deployed],
                "Behind Schedule",
                    "On Schedule"
            )
)
```
#### Team Performance
The formula is used to calculate the average number of deployments per day. This count can then be used to display the average deployment of different types of devices, as well as the average number of devices deployed by each team, which can be used for future planning.
```
Avergae performnace = DIVIDE(
    COUNTA(Performance[Deployment Date]),
        DISTINCTCOUNTNOBLANK(Performance[Deployment Date])
    )
```

### Power Query: 
Utilized for data manipulation and transformation.
