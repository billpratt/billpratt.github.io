---
layout: post
title: 'Swagger and ASP.NET Web API - Part I: Adding Swagger to Web API project'
date: 2015-06-07 13:43:59.000000000 -04:00
permalink: /swagger-and-asp-net-web-api-part-1/
---
This is part one of a series on using Swagger with ASP.NET Web API.

All source code for this series can be found <a href="https://github.com/billpratt/SwaggerDemoApi" target="_blank">here</a>.

When you create a new ASP.NET Web API project, a nuget package called <a href="https://www.nuget.org/packages/Microsoft.AspNet.WebApi.HelpPage/5.2.3" target="_blank">Microsoft ASP.NET Web Api Help Page</a> is installed to generate help page content for the web APIs on your site. In my previous post <a href="http://wmpratt.com/runscope-and-continuous-integration/" target=_"blank">Runscope and continuous integration</a>, I used this to provide descriptions for the APIs. The help page package is a good start but it is lacking things like discoverability and live interactions.  This is where Swagger comes to the rescue.

> "Swagger is a simple yet powerful representation of your RESTful API. With the largest ecosystem of API tooling on the planet, thousands of developers are supporting Swagger in almost every modern programming language and deployment environment. With a Swagger-enabled API, you get interactive documentation, client SDK generation and discoverability."
>
-<a href="http://swagger.io/" target="_blank">swagger.io</a>

### ASP.NET Web API Help Page documentation
![](/assets/images/2015/06/help_pages_without_swagger.jpg)

### Swagger documentation
![](/assets/images/2015/06/swagger_help_pages.jpg)

Adding Swagger to your Web API **does not** replace ASP.NET Web API help pages. You can have both running side by side, if desired.

# Adding Swagger to Web Api Project
To add Swagger to an ASP.NET Web Api, we will install an open source project called <a href="https://www.nuget.org/packages/Swashbuckle/" target="_blank">Swashbuckle</a> via nuget.

![](/assets/images/2015/06/install_swashbuckle.PNG)

After the package is installed, navigate to App_Start in the Solution Explorer. You'll notice a new file called SwaggerConfig.cs. This file is where Swagger is enabled and any configuration options should be set here.

![](/assets/images/2015/06/swagger_config-1.png)

# Configuring Swagger
At minimum you'll need this line to enable Swagger and Swagger UI.
```
GlobalConfiguration.Configuration
  .EnableSwagger(c => c.SingleApiVersion("v1", "A title for your API"))
  .EnableSwaggerUi();
```

Start a new debugging session (F5) and navigate to http://localhost:[PORT_NUM]/swagger. You should see Swagger UI help pages for your APIs. 

![](/assets/images/2015/06/swagger_ui.png)

Expanding an api and clicking the "Try it out!" button will make a call to that specific API and return results. Pretty cool!

![](/assets/images/2015/06/swagger_get_superhero-1.png)
![](/assets/images/2015/06/swagger_get_response.png)

# Enable Swagger to use XML comments
The minimum configuration is nice to get started but let's add some more customization. We can tell Swashbuckle to use XML comments to add more details to the Swagger metadata. These are the same XML comments that ASP.NET Help Pages uses.

First, enable XML documentation file creation during build. In *Solution Explorer* right-click on the Web API project and click *Properties*. Click the *Build* tab and navigate to *Output*. Make sure *XML documentation file* is checked. You can leave the default file path. In my case its *bin\SwaggerDemoApi.XML*

![](/assets/images/2015/06/build_xml_docs.png)

Next, we need to tell Swashbuckle to include our XML comments in the Swagger metadata. Add the following line to SwaggerConfig.cs. Make sure to change the file path to the path of your XML documentation file.

```
c.IncludeXmlComments(string.Format(@"{0}\bin\SwaggerDemoApi.XML", System.AppDomain.CurrentDomain.BaseDirectory));
```

Full configuration, so far

```
GlobalConfiguration.Configuration
  .EnableSwagger(c =>
    {
      c.SingleApiVersion("v1", "SwaggerDemoApi");
      c.IncludeXmlComments(string.Format(@"{0}\bin\SwaggerDemoApi.XML",           
                           System.AppDomain.CurrentDomain.BaseDirectory));
    })
  .EnableSwaggerUi();
```

Finally, if you haven't already, add XML comments to your Models and API methods.

![](/assets/images/2015/06/superhero_xml_comments.png)

![](/assets/images/2015/06/xml_comments.png)

Run the project and navigate back to /swagger. You should see more details added to your API documentation. I've highlighted a few below with their corresponding XML comment.
![](/assets/images/2015/06/swagger_xml_comments.png)

Under *Response Class*, click *Model*. You should see any XML comments added to your models.

![](/assets/images/2015/06/swagger_xml_comments_model.png)

# Describing Enums As Strings
My Superhero class contains an Enum property called *Universe* which represents which comic universe they belong to.

![](/assets/images/2015/06/universe_enum.png)

By default, Swagger displays these Enum values as their integer value. IMO, this is not very descriptive. Let's change it to display the string representation.

Add the following line to *SwaggerConfig.cs*
```
c.DescribeAllEnumsAsStrings();
```
The full swagger configuration at this point
```
GlobalConfiguration.Configuration
  .EnableSwagger(c =>
  {
    c.SingleApiVersion("v1", "SwaggerDemoApi");
    c.IncludeXmlComments(string.Format(@"{0}\bin\SwaggerDemoApi.XML", 
                         System.AppDomain.CurrentDomain.BaseDirectory));
    c.DescribeAllEnumsAsStrings();
  })
  .EnableSwaggerUi();
```
If I look at Swagger now, the *Universe* Enum values are displayed as strings. Much better!

![](/assets/images/2015/06/swagger_xml_comments_enum.png)

These are just a few of the many configuration options you can specify in Swashbuckle to create your Swagger metadata. I encourage you to review the other options on <a href="https://github.com/domaindrivendev/Swashbuckle" target="_blank">Swashbuckle's GitHub</a>.

# Swagger JSON file
What we've seen so far is a UI representation our API Swagger metadata. To see the actual "Swagger", navigate to the URL that is in the header of the Swagger UI documentation page.
![](/assets/images/2015/06/swagger_url.png)

This is how your API is discoverable. The Swagger metadata can be used to tell other APIs how to interact with yours. You can also create a client library to interact with your API that can be distributed to customers/users/integration partners.

Here is a sample of my Swagger metadata

![](/assets/images/2015/06/swagger_output.png)

The Microsoft Azure team is currently in the process of including Swagger in their new <a href="http://azure.microsoft.com/en-us/services/app-service/" target="_blank">Azure App Service</a>, currently in Preview. I encourage you to watch the //build/ 2015 talk about <a href="http://channel9.msdn.com/Events/Build/2015/2-628" target="_blank">Azure App Service Architecture</a> with <a href="https://twitter.com/shanselman" target="_blank">Scott Hanselman</a> and <a href="https://twitter.com/coolcsh" target="_blank">Scott Hunter</a>.

Source code for this series: <a href="https://github.com/billpratt/SwaggerDemoApi" target="_blank">github.com/billpratt/SwaggerDemoApi</a>

*Related Posts*

* [Part II: Enabling OAuth 2.0]({{ site.baseurl }}{% link _posts/2016-01-24-part-ii-swagger-and-asp-net-web-api-enabling-oauth2.md %})
