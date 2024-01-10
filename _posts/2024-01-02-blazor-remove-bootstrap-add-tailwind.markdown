---
title:  "Use Tailwind instead of bootstrap in your Blazor startup project"
date:   2024-01-10 11:00:00 +0200
tag: 
    - Blazor
    - TailwindCSS
    - Bootstrap
    - .NET
---

When creating a new Blazor project, by default, it always includes Bootstrap as a CSS framework. This is used for both the layout and component styling. And, of course, this is fine because it's easy to use and set up, and Bootstrap is still well maintained. But I can imagine that you will probably start from scratch in most cases. In my case, I wanted to start with [TailwindCSS](https://tailwindcss.com/), so I created a Blazor starter project with Tailwind.

![picture](/assets/20240110/BlazorTailwind.png){: width="250" }

I wanted to use Tailwind since it's one of the more modern CSS frameworks as of today. It's highly flexible and suits well with creating reusable UI components.
However, how does this work with Blazor?
Since Tailwind CSS output is generated based on the classes used in your HTML. There is this fantastic Nuget package called [Tailwind.Extensions.AspNetCore](https://github.com/Practical-ASP-NET/Tailwind.Extensions.AspNetCore), which helps make this work within your Blazor project.
Just follow the steps in the readme, and you are good to go.

I added this additional configuration to make everything work with a .NET 8 solution setup with a Blazor Server and Blazor WebAssembly project.
By doing so, Tailwind will also pull all the classes from components in the Blazor WebAssebmly project.
{% highlight Typescript %}
module.exports = {
    content: ['./**/*.{razor,html}', '../BlazorTailwind.Client/**/*.{razor,html}'],
    theme: { },
    plugins: [],
}
{% endhighlight %}

After that, I removed all other styling and styling sheets from the projects and, one by one, restyled the pages and components with the Tailwind classes. I tried to make it look as similar to the original default Blazor template as possible. The result can be found here: [https://blazortailwind.azurewebsites.net/](https://blazortailwind.azurewebsites.net/).
The source code can be found here: [https://github.com/NielsNijveldt/BlazorTailwind](https://github.com/NielsNijveldt/BlazorTailwind).

## Pipeline
I also added a GitHub Actions pipeline, which builds the Blazor Project, compiles and minifies the Tailwind output, and then publishes it to Azure.
For the Tailwind path, I added this script to the package.json.
{% highlight Console %}
npx tailwindcss -i ./Styles/tailwind.css -o ./wwwroot/css/tailwind.css --minify
{% endhighlight %}

## Hot reload
If you want to develop your application in combination with hot reload, make sure to start the app with "Start without debugging" (or CTRL + F5). Otherwise hot reload doesn't seem to work. You shouldn't need to restart your application after creating a component with a newly used utility class since the installed NuGet package ensures it gets added to the CSS output file. 

![picture](/assets/20240110/HotReload.gif)