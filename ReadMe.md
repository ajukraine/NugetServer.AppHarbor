# Hosting Nuget server at AppHarbor

This repository is an example of fully and correctly working [NuGet](http://nuget.org/packages/NuGet.Server/ "NuGet") server, deployed with [AppHarbor](https://appharbor.com/ "AppHarbor") [PaaS](http://en.wikipedia.org/wiki/Platform_as_a_service "PaaS").

It is not possible to setup NuGet server at AppHarbor out of the box (at least, at the time of first commit to this repo). There are several obstacles, caused by the nature the of AppHarbor platform. Keep reading next section to perform succesfull setup.

## Installation

You may choose one of the following ways to install NuGet server. They lead you to the same final result.

#### Quick

If you want to setup NuGet server right away, please be free to fork (or clone) the repository and link it with you AppHarbor application.

#### Explained

The solution was reached by following these steps:

1. Create blank Visual Studio solution (I'm using VS 2010)
2. Create "ASP.NET Empty Web Application" project under the solution
3. Install "Nuget.Server" package, using Nuget command line or package manager
4. Enable "NuGet package restore" through solution's context menu under
5. Now we need to fix first problem. NuGet server's WCF service is not working without following modifications to Web.config. Please be sure to add them:
	```xml
	<system.serviceModel>
			<serviceHostingEnvironment multipleSiteBindingsEnabled="true" />
	    	<protocolMapping>
	        	<add scheme="http" binding="basicHttpBinding" />
	    	</protocolMapping>
	</system.serviceModel>
	```
More details are available in [StackOverflow's answer](http://stackoverflow.com/a/12160312/592377)

6. Next problem is the port number, that appears as part of urls in default page. This port is internal port, that is used behind AppHarbor load balancer. But there is workaround - please be sure to update *appSettings* section in Web.config with following line:
	```xml
	<add key="aspnet:UseHostHeaderForRequestUrl" value="true " />
	```
More details are available at AppHarbor knowledge base [page](http://support.appharbor.com/kb/getting-started/workaround-for-generating-absolute-urls-without-port-number) 

7. Also check if you set password for server:
	```xml
	<add key="apiKey" value="apiPassword" />
	```

8. One thing, that you need to know, is that AppHarbor clears all files during deploy. So that nuget packages disappear after deploy. But do not worry, it is enough for testing purpose. Also by default AppHarbor gives you access only to *App_Data* folder; that's why set packages path to be inside *App_Data*:
	```xml
	<add key="packagesPath" value="~/App_Data/Packages" />
	```
9. This step is optional. But I recommend you to follow it too. Do you remember file structure, that we achieved so far? Let's place appropriate *.gitgnore* files to leave binaries and logs outside version control:
	- NugetServer.AppHarbor
		- ...
		- **.gitignore** (ignore *App_Data*)
		- packages
			- ...
			- **.gitignore** (ignore Nuget packages)


Now you may build your solution and test it locally, and then push it to AppHarbor.

## Demonstration

You are able to access my testing purpose installation - [http://nuget-gallery.apphb.com/](http://nuget-gallery.apphb.com/)

## References

- [NuGet.Server package](http://nuget.org/packages/NuGet.Server/)
- Issue [#2716](https://nuget.codeplex.com/workitem/2716) at Nuget tracker :: Nuget.Server not working on AppHarbor
- Issue [#7690](http://support.appharbor.com/discussions/problems/7690-nugetserver-app-results-in-502-bad-gateway) at AppHarbor support :: Nuget.Server app results in 502 Bad Gateway 
- Bug [#11](https://github.com/trilobyte/Premotion-AspNet-AppHarbor-Integration/issues/11) at Premotion-AspNet-AppHarbor-Integration tracker :: **X-NuGet-ApiKey** header is lost
- Issue [#7148](http://support.appharbor.com/discussions/problems/7148-custom-headers-are-changedlost) at AppHarbor support :: Custom Headers are changed/lost
- [AppHarbor's load balanacer infromation](http://support.appharbor.com/kb/getting-started/information-about-our-load-balancer)
- [One more HoWTo](http://andrey.moveax.ru/post/nuget-creating-and-deploying-simple-server.aspx) of installing NuGet server ( *in russian*)
