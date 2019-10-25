---
title:  "Display OpenCover results in Azure DevOps"
date:   2019-10-25 22:00:00 +0200
tag: 
    - OpenCover
    - Azure DevOps
    - PowerShell
    - IIS
    - Code Coverage
    - Test Automation
---

So with my first post I managed to run OpenCover in the build and measure the code coverage on my .net API by end-2-end tests. Of course this is fun, but it would be much more of value if we could also show the results somewhere. So in this post I will explain the update to the build to make this happen. In the end the results will be displayed in an Azure DevOps dashboard or SonarQube.

Kopje
To get the coverage as an artifact of the build we can use the Publish Code Coverage Result Task (1). Because this task needs Cobertura or JaCoCo as input we need to make sure we get this output from OpenCover. Out-of-the-box OpenCover is not able to create such a file. However with ReportGenerator(2) OpenCover output can be transformed into one of those formats. (and a lot of other formats) ReportGenerator also has to capability to merge coverage files into one file. So with this line of code we can make sure we get the right format:
ReportGenerator.exe "-reports:OpenCover.xml" "-targetdir:coveragereport" -reporttypes:HTML;Cobertura;SonarQube

You could either set the targetdir to a file share or copy the report folder back to the build agent in the end.

Publish artifact
Now we've got our Cobertura  file (and SonarQube file) we can use the publish code coverage task. Set the Cobertura as 'Summary file' and we are good to go. After running the build you should be able to see the code coverage in the build results. As a final step we can add the "Code Coverage Widgets"(3) to a dashboard in Azure Devops. When configuring this widget select our build and choose one of the coverage measurements. Small blocks will show as number/percentage and larger blocks will show graphs. 

SonarQube
Beside Azure DevOps we can also supply the coverage to SonarQube. Therefore we need to set the 'sonar.cs.opencover.reportsPaths' property to the path of the SonarQube.xml created by ReportGenerator. Now the Sonarscanner should be able to upload the file to SonarQube.


https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/test/publish-code-coverage-results?view=azure-devops
https://github.com/danielpalme/ReportGenerator
https://marketplace.visualstudio.com/items?itemName=shanebdavis.code-coverage-dashboard-widgets
