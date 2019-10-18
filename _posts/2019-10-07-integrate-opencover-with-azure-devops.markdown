---
title:  "Integrate OpenCover with Azure DevOps"
date:   2019-10-18 13:00:00 +0200
tag: 
    - OpenCover
    - Azure DevOps
    - PowerShell
    - IIS
    - Code Coverage
    - Test Automation
---

[OpenCover][github-opencover] is a code coverage tool which measures both branch and sequence points for a given .Net application. In my case I wanted to measure code coverage of a .Net web API project. The idea is to start OpenCover, run end-2-end tests (or other tests invoking my API) and generate a coverage file. This gives you insight about how much of your API is covered by for example end-2-end tests.

![picture](/assets/20191007/kai-dahms-217U8oxGoQ4-unsplash.jpg)
_Photo by Kai Dahms on [Unsplash](unsplash)_

In order to achieve this I use [OpenCover.Console.exe][github-opencover-console] and attach it to the IIS process of the backend application as described on the [wiki of OpenCover][github-opencover-wiki]. This works fine for manually starting OpenCover, but in my Azure DevOps pipeline everything should be automated. In order to do so I created three PowerShell scripts. One to start OpenCover, one to Invoke the start script on another machine and one to close OpenCover and generate a report.

### Starting OpenCover
The start script stops the current running backend IIS application. After it is stopped the actual OpenCover.Console.exe can be started:
{% highlight ruby %}
OpenCover.Console.exe -target:C:\Windows\System32\inetsrv\w3wp.exe -targetargs:-debug -targetdir:C:\iisprojectdir -output:C:\reports\opencover-result.xml -filter:+[*]* -register:user 
{% endhighlight %}
Once this is completed the backend application is running again and can be used for testing.
In the Task manager you will now find two processes for the same application.
It might be good to perform some kind of warmup for the new running instance of your application.

Starting OpenCover should not be done with remote PowerShell.
Remote PowerShell will start a session, start OpenCover and when the task is finished it will close the session.
When the PowerShell session is ended OpenCover will be killed.
This can be avoided by using [Invoke-Command](invoke-command-docs) instead with the **-InDisconnectedSession** parameter:

{% highlight ruby %}
Invoke-Command -ComputerName $server -Credential $credential -InDisconnectedSession -ScriptBlock { <Insert OpenCover Start script> }
{% endhighlight %}

**Note**: Be aware that the application will be running as the user running the script.

### Stopping OpenCover
After testing is done it’s important to not end the OpenCover process itself but the new w3wp process. If the OpenCover process is ended first no coverage file will be generated after it.
So in the second script the right w3wp process is searched for and when found the process gets killed. Now OpenCover will generate the coverage file after a couple of seconds. In the end the backend IIS application can be started again in IIS like it was originally running.

### Pipeline
Both scripts can be used inside a Azure DevOps pipeline with a test execution in between. When OpenCover is stopped correctly the generated result xml can be used in for example SonarQube.
![Azure Devops](/assets/20191007/pipeline.png)

The scripts can be found on my [GitHub](github-opencover-scripts).
Feel free to ask questions!

[github-opencover]: https://github.com/OpenCover/opencover
[github-opencover-wiki]: https://github.com/OpenCover/opencover/wiki/IIS-Support
[github-opencover-console]: https://github.com/OpenCover/opencover/tree/master/main/OpenCover.Console
[github-opencover-scripts]: https://github.com/NielsNijveldt/OpenCover-Scripts
[invoke-command-docs]: https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/invoke-command?view=powershell-6#description
[unsplash]: https://unsplash.com/photos/217U8oxGoQ4