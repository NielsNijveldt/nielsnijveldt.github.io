---
layout: post
title:  "Integrate OpenCover with Azure DevOps"
date:   2019-10-07 16:52:35 +0200
categories: OpenCover Azure DevOps
---
[OpenCover][github-opencover] is a code coverage tool which measures both branch and sequence points for a given .Net application. In my case I wanted to measure code coverage of a .Net web API project. The idea is to start OpenCover, run end-2-end tests (or other tests invoking my API) and generate a coverage file in the end.

In order to achieve this I use [OpenCover.Console.exe][github-opencover-console] and attach it to the IIS process of the backend application as described on the [wiki of OpenCover][github-opencover-wiki]. This works fine for manual triggering everything, but in my Azure DevOps pipeline everything should be automated. In order to do so I created two PowerShell scripts. One to start OpenCover and one to close OpenCover.

The first script stops the current running backend IIS application. After it is stopped the actual OpenCover.Console.exe can be started:
{% highlight ruby %}
OpenCover.Console.exe -target:C:\Windows\System32\inetsrv\w3wp.exe -targetargs:-debug -targetdir:$targetDir -output:$oututFileDir -filter:+[*]* -register:user 
{% endhighlight %}
Once this is completed the backend application is running again and can be used for testing.
In the Task manager you will find two processes.

After testing is done it’s important to not end the OpenCover process itself but the w3wp process. If the OpenCover process is ended first no coverage file will be outputted after it.
So in the second script the w3wp process is searched for and when found the process gets killed. Now OpenCover will generate the coverage file after a couple of seconds. In the end the backend IIS application can be started again in IIS like it was originally running.

Both scripts can be used inside a Azure DevOps pipeline with a test execution in between. Also the coverage output file can be imported in a analyze tool like SonarQube.
![Azure Devops](/assets/20191007/opencoverazuredevops.png)

TODO:
- Powershell code toevoegen
- Plaatje van task manager toevoegen

[github-opencover]: https://github.com/OpenCover/opencover
[github-opencover-wiki]: https://github.com/OpenCover/opencover/wiki/IIS-Support
[github-opencover-console]: https://github.com/OpenCover/opencover/tree/master/main/OpenCover.Console
