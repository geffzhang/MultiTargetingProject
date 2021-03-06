# MultiTargetingProject

This is an example repository on how to target .NET 4.6, .NET 4.7 and UAP 10.0 using a single project.

This project has been written as an example for the [blog post about multi-targeting platforms with a single project](http://geertvanhorrik.com/2018/03/14/multi-targeting-net-4-6-net-4-7-uap-10-0-with-a-single-project/).

Along the way, this example has been expanded with the following new [Cake](https://www.cakebuild.net/) build scripts:

- Build multi-targeting class libraries (components)
- Build WPF apps
- Build UWP apps
- Build Chocolatey tools
- Build GitHub pages
- Build web apps (Octopus Deploy and Azure Web Apps)
- Build docker images

The scripts are set up in a way that allows a combination of these items as well (e.g. a WPF with an API library) can all be set up. For more information, see the [RepositoryTemplate repository](https://github.com/GeertvanHorrik/RepositoryTemplate).

# How to build this example

1. Open powershell with the repository as working directory
2. `.\build.ps1 -target package`

# How does it work?

The scripts are extremely generic. One only has to customize 1 file:

* `/build.cake`

Below is an example script to package a (multi-targeting) component, wpf and uwp app in a single solution:

    //=======================================================
    // DEFINE PARAMETERS
    //=======================================================
    
    // Define the required parameters
    var Parameters = new Dictionary<string, object>();
    Parameters["SolutionName"] = "Ghk.MultiTargeting";
    Parameters["Company"] = "Geert van Horrik";
    Parameters["RepositoryUrl"] = string.Format("https://github.com/GeertvanHorrik/{0}",  GetBuildServerVariable("SolutionName"));
    Parameters["StartYear"] = "2018";
    Parameters["UseVisualStudioPrerelease"] = "true";
    
    // Note: the rest of the variables should be coming from the build server,
    // see `/deployment/cake/*-variables.cake` for customization options
    // 
    // If required, more variables can be overridden by specifying them via the 
    // Parameters dictionary, but the build server variables will always override
    // them if defined by the build server. For example, to override the code
    // sign wild card, add this to build.cake
    //
    // Parameters["CodeSignWildcard"] = "Orc.EntityFramework";
    
    //=======================================================
    // DEFINE COMPONENTS TO BUILD / PACKAGE
    //=======================================================
    
    Components.Add("Ghk.MultiTargeting");
    Tools.Add("Ghk.MultiTargeting.Tool");
    WpfApps.Add("Ghk.MultiTargeting.Example.Wpf");
    UwpApps.Add("Ghk.MultiTargeting.Example.Uwp");
    TestProjects.Add("Ghk.Multitargeting.Tests");

    // Not included:
    // DockerImages.Add("Ghk.MultiTargeting.DockerImage");
    // WebApps.Add("Ghk.MultiTargeting.WebApp");
    // GitHubPages.Add("Ghk.MultiTargeting.GitHubPages");

    //=======================================================
    // REQUIRED INITIALIZATION, DO NOT CHANGE
    //=======================================================
    
    // Now all variables are defined, include the tasks, that
    // script will take care of the rest of the magic
    
    #l "./deployment/cake/tasks.cake"

And if you are not using a build server (seriously, you should be), you could choose to customize `/deployment/cake/*-variables.cake` as well.

The Cake magic happens in the generic scripts in `/deployment/cake/*.cake`

## How to use the scripts

Below are a few example targets **using exactly the same scripts**:

| Script | Description |
|--------|-------------|
| `./build.ps1 -Target BuildWpfApps` | Build all WPF apps in the solution |
| `./build.ps1 -Target Package` | Build & package all components & (WPF & UWP) apps in the solution |
| `./build.ps1 -Target PackageComponents` | Build & package all components in the solution |
| `./build.ps1 -Target PackageUwpApps` | Build & package all UWP apps in the solution |

As you can see, there is a pattern:

* Build[SpecificTarget]
* Package[SpecificTarget]

*Note that [SpecificTarget] is optional*

# Project differences compared to the original template

Compared to the [original multi-targeting example](https://oren.codes/2017/01/04/multi-targeting-the-world-a-single-project-to-rule-them-all/), the following changes were made.

## Project defines

To make sure I could still use the defines we invested in (to maximize code-sharing, we use a lot of `#if` in code shared between WPF and UWP). This project creates the following defines:

* .NET 4.6 => NET; NET46
* .NET 4.7 => NET; NET47
* UAP 10.0 => UAP; NETFX_CORE
* .NET Standard 2.0 => NS; NS20; NETSTANDARD; NETSTANDARD2_0

## Include solution assembly info

The project structure adds this line to the project to include the SolutionAssemblyInfo and GlobalSuppressions files which are in the root of the solution.

	<Compile Include="..\*.cs" />

## Allow shared code-behind (e.g. MyView.xaml.cs) with a *per platform* view (e.g. MyView.xaml)

To allow sharing of code-behind of a view (for example, for custom controls), we need to to a bit of hacking. We create the view inside each platform specific view directory:

* Platforms\net\Views\MyView.xaml (+ .cs)
* Platforms\uap10.0\Views\MyView.xaml (+ .cs)

After creating the views, one will need to customize the code-behind to add a partial method:

	public partial class MyView
	{
	     public MyView()
	     {
	         InitializeComponent();
	 
	         Construct();
	     }
	 
	     partial void Construct();
	}

Next, create a partial code-behind class inside the regular views folder that contains the shared code:

* Views\MyView.xaml.cs

This view should contain the following code and allows sharing code-behind but have per-platform xaml files:

	public partial class MyView
	{
	     partial void Construct()
	     {
	         // TODO: Add shared constructor info here
	     }
	}

As an example, a dependency property is added in the shared code-behind that shows how to deal with the different platforms in shared code using defines.

# Credits

- [Multi-targeting the world: a single project to rule them all](https://oren.codes/2017/01/04/multi-targeting-the-world-a-single-project-to-rule-them-all/) (Oren Novotny - January 4, 2017)