# Microsoft Graph OneDrive Photo Browser sample

The Microsoft Graph OneDrive Photo Browser sample is a Windows Universal app that uses the [Microsoft Graph .NET Client Library](https://github.com/microsoftgraph/msgraph-sdk-dotnet) for C#/.NET. 
The sample app displays only items that are images from a user's OneDrive. Note that this sample does not work with OneDrive for Business.

The sample uses the v2.0 authentication endpoint, which enables users to sign in with either their personal or work or school Microsoft accounts.


## Set up

### Prerequisites

To run the sample, you will need: 

* Visual Studio 2015, with Universal Windows App Development Tools **Note:** If you don't have Universal Windows App Development Tools installed, open **Control Panel** | **Uninstall a program**. Then right-click **Microsoft Visual Studio** and click **Change**. Select **Modify** and then choose **Universal Windows App Development Tools**. Click **Update**. For more info about setting up your machine for Universal Windows Platform development, see [Build UWP apps with Visual Studio](https://msdn.microsoft.com/en-us/library/windows/apps/dn609832.aspx).
* Windows 10 ([development mode enabled](https://msdn.microsoft.com/library/windows/apps/xaml/dn706236.aspx))
* Either a [Microsoft](www.outlook.com) or [Office 365 for business account](https://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account).
* Knowledge of Windows Universal app development

### Download the sample

1. Download the sample from [GitHub](https://github.com/OneDrive/graph-sample-photobrowser-uwp) by choosing **Clone in Desktop** or **Download Zip**. 
2. In Visual Studio, open the **OneDrivePhotoBrowser.sln** file and build it.

##Register and configure the app

1. Sign into the [App Registration Portal](https://apps.dev.microsoft.com/) using either your personal or work or school account.  
2. Select **Add an app**.  
3. Enter a name for the app, and select **Create application**. The registration page displays, listing the properties of your app.  
4. Under **Platforms**, select **Add platform**.  
5. Select **Mobile application**.  
6. Copy both the Client Id (App Id) value to the clipboard. You'll need to use it in the sample app. The app id is a unique identifier for your app.   
7. Select **Save**.  

After you've loaded the solution in Visual Studio, configure the sample to use the Client Id that you registered by adding it as a key in the **Application.Resources** node of the App.xaml file.

```xml
    <x:String x:Key="ida:ClientID">your Client Id</x:String>
```

## Run the sample

1. With the sample open in Visual Studio, at the top, select **Debug** for Solution Configurations and **x86** or **x64** for Solution Platforms, and **OneDrivePhotoBrowser** for Startup project. 
2. Check that you are running the sample on the **Local Machine**.
3. Press **F5** or click **Start** to run the sample.

The OneDrive Photo Browser sample app will open the signed-in user's personal OneDrive, with only folders and images displayed. If the file is not an image, it will not show up in the OneDrive Photo Browser app. Select a folder to see all images in that folder. Select an image to see a larger display of the image, with scroll view.


## API features

### MSAL sign-in

Users can log in with either a [Microsoft](www.outlook.com) or [Office 365 for business account](https://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account).

After the user signs in, the `AuthenticationHelper` class returns an MSAL `GraphServicesClient`.

```csharp
        public static GraphServiceClient GetAuthenticatedClient()
        {
            if (graphClient == null)
            {
                // Create Microsoft Graph client.
                try
                {
                    graphClient = new GraphServiceClient(
                        "https://graph.microsoft.com/v1.0",
                        new DelegateAuthenticationProvider(
                            async (requestMessage) =>
                            {
                                var token = await GetTokenForUserAsync();
                                requestMessage.Headers.Authorization = new AuthenticationHeaderValue("bearer", token);
                                // This header has been added to identify our sample in the Microsoft Graph service.  If extracting this code for your project please remove.
                                requestMessage.Headers.Add("SampleID", "uwp-csharp-photobrowser-sample");

                            }));
                    return graphClient;
                }

                catch (Exception ex)
                {
                    Debug.WriteLine("Could not create a graph client: " + ex.Message);
                }
            }

            return graphClient;
        }
```

### Get thumbnails for an image in OneDrive

In this example, thumbnails are returned for an item, if it is an image. `GetAsync()` is used to get the item's properties.

```csharp
           IEnumerable<DriveItem> items;

            var expandString = "thumbnails, children($expand=thumbnails)";

            // If id isn't set, get the OneDrive root's photos and folders. Otherwise, get those for the specified item ID.
            // Also retrieve the thumbnails for each item if using a consumer client.
            var itemRequest = string.IsNullOrEmpty(id)
                ? this.graphClient.Me.Drive.Root.Request().Expand(expandString)
                : this.graphClient.Me.Drive.Items[id].Request().Expand(expandString);

            var item = await itemRequest.GetAsync();
            items = item.Children == null
                ? new List<DriveItem>()
                : item.Children.CurrentPage.Where(child => child.Folder != null || child.Image != null);
```

## More resources

* [Microsoft Graph .NET Client Library](https://github.com/microsoftgraph/msgraph-sdk-dotnet)
* [Windows Universal apps](https://msdn.microsoft.com/en-us/library/windows/apps/dn726767.aspx) - More information about Windows Universal apps

## License

[License](LICENSE.txt)

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
