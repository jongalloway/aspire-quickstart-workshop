# .NET Aspire Quickstart Workshop

## What you'll be building

In this workshop, you'll learn the basics of .NET Aspire through three quickstart labs:

1. **Lab 1**: Create a project with Redis cache using .NET Aspire
1. **Lab 2**: Add a database to your project using .NET Aspire
1. **Lab 3**: Deploy a project to Azure using .NET Aspire

This workshop is designed to be self-paced, but you can also use it in an instructor-led workshop session. The focus is on really learning the fundamentals before moving on to more complex applications. This workshop is designed to be completed in 2-3 hours, and to prepare you for the [.NET eShop - App Building Workshop](https://github.com/dotnet-presentations/eshop-app-workshop).

## Setup

### Using Windows and Visual Studio

If you're on Windows and using Visual Studio, you must use [Visual Studio 2022 Preview](https://visualstudio.com/preview) (version 17.10.0 Preview 5.0 or later). The preview version of Visual Studio 2022 is safe to install side-by-side with the release version. We recommend using Visual Studio 2022 Preview if you're on Windows as it includes support for working with .NET Aspire projects.

> Note: When installing Visual Studio you only need to install the `ASP.NET and web development` workload.

If you're in an instructor-led workshop session and have issues downloading the installers we may have USB sticks with offline installers for you to use.

### Using macOS, Linux, or Windows but not using Visual Studio

If you're using macOs or Linux, or on Windows but don't want to use Visual Studio, you must [download](https://www.microsoft.com/net/download) and install the .NET SDK (version 8.0.100 or newer). You can use the editor or IDE of your choice but note that some operations might be more difficult due to lack of support for .NET Aspire at this time.

### Updating and installing the .NET SDK workload for Aspire

After installing Visual Studio Preview or the required .NET SDK, you will need to update and install the .NET SDK workload for Aspire. This workshop is using the latest preview of .NET Aspire (preview.6).

1. To ensure that you install the latest version of the .NET Aspire workload, run the following [dotnet workload update](/dotnet/core/tools/dotnet-workload-update) command before you install .NET Aspire:

    ```dotnetcli
    dotnet workload update
    ```

1. To install the .NET Aspire workload from the .NET CLI, use the [dotnet workload install](/dotnet/core/tools/dotnet-workload-install) command:

    ```dotnetcli
    dotnet workload install aspire
    ```

1. To check your version of .NET Aspire, run this command:

    ```dotnetcli
    dotnet workload list
    ```

    You should see the `aspire` workload listed.
