---
title:  "Display OpenCover results in Azure DevOps"
date:   2019-10-30 21:00:00 +0200
tag: 
    - OpenCover
    - Azure DevOps
    - PowerShell
    - IIS
    - Code Coverage
    - Test Automation
---

So with my first post I managed to run OpenCover in the build and measure the code coverage on my .Net API by end-2-end tests. Of course this is fun, but it would be much more of value if we could also show the results somewhere. So in this post I will explain how to update the build to make this happen. In the end the results will be displayed in an Azure DevOps dashboard or SonarQube.

![picture](/assets/20191025/joshua-earle-Dwheufds6kQ-unsplash.jpg)
_Photo by Joshua Earle on [Unsplash](https://unsplash.com/photos/Dwheufds6kQ){:target="_blank"}_

### Coverage
To get the coverage attached to the build we can use the [Publish Code Coverage Result Task](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/test/publish-code-coverage-results?view=azure-devops){:target="_blank"}. Because this task needs Cobertura or JaCoCo as input we need to make sure we get this output from OpenCover. Out-of-the-box OpenCover is not able to create such a file. 
Also the OpenCover result file contains paths to probably folders which don't exist anymore. So in order to fix that we need to replace the paths to the existing application source on the build agent:

{% highlight ruby %}
{% raw %}
$content = (Get-Content $(Build.SourcesDirectory)\opencover-reports\result.xml) -replace '(\w\:)\\(\w+)\\(\d+)\\(\w+)', '$(Build.SourcesDirectory)'; 
[System.IO.File]::WriteAllLines( '$(Build.SourcesDirectory)\opencover-reports\result.xml', $content)
{% endraw %}
{% endhighlight %}

After this is done, with [ReportGenerator](https://github.com/danielpalme/ReportGenerator){:target="_blank"} OpenCover output can be transformed into one of the required formats. (and a lot of other formats) So with this command we can make sure we get the right format:
{% highlight ruby %}
ReportGenerator.exe "-reports:$(Build.SourcesDirectory)\opencover-reports\result.xml" "-targetdir:coveragereport" -reporttypes:HTML;Cobertura;SonarQube
{% endhighlight %}
You could either set the targetdir to a file share or copy the report folder back to the build agent in the end.
ReportGenerator also has to capability to merge multiple coverage files into one file. So if you want to merge multiple test output files into one test result you can use this:
{% highlight ruby %}
ReportGenerator.exe "-reports:OpenCover.xml;OpenCover2.xml" "-targetdir:coveragereport" -reporttypes:HTML;Cobertura;SonarQube
{% endhighlight %}

![Publish Code Coverage Result Task](/assets/20191025/task.png)

### Publish coverage
Now we've got our Cobertura file (and SonarQube file) we can use the publish code coverage task. Set the Cobertura file as 'Summary file' and we are good to go. After running the build you should be able to see the code coverage in the build results. As a final step we can add the [Code Coverage Widgets](
https://marketplace.visualstudio.com/items?itemName=shanebdavis.code-coverage-dashboard-widgets){:target="_blank"} to a dashboard in Azure Devops. When configuring this widget select our build and choose one of the coverage measurements. Small blocks will show as number/percentage and larger blocks will show graphs.

![Code Coverage Widgets](/assets/20191025/graphs.png)

### SonarQube
Beside Azure DevOps we can also supply the coverage to SonarQube. Therefore we need to set the 'sonar.cs.opencover.reportsPaths' property to the path of the SonarQube.xml created by ReportGenerator. Now the Sonarscanner should be able to upload the file to SonarQube.

### Result
Now code coverage will automatically be updated and published to the desired tool/dashboard. New tests or missing tests will immediately affect the results. By doing this, we avoid the need to publish this by hand and we are always able to get the latest state of coverage for different kinds of tests. 

An example of how the pipeline could look like can be found [here](https://github.com/NielsNijveldt/OpenCover-Scripts/blob/master/pipeline-example.yml){:target="_blank"}.

If you have any questions, feel free to contact me.