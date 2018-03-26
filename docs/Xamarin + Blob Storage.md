# Add Cloud Storage to Your App with Azure Blob Storage

Cloud Storage has become a must-have feature for mobile apps. It gives us the ability to store our large files securely in the cloud, which helps to reduce the size of our APKs and IPAs, and to safely and securely distribute files across our mobile app users.

[Azure Blob Storage](https://aka.ms/xamarinblog/azureblobstorage) makes it easy for Xamarin developers to implement Cloud Storage for our cross-platform apps, allowing us to share 100% code.

What is Azure Blob Storage? It is like a folder in the cloud. In this folder, we can put anything: PDFs, photos, Word documents, etc., and each of these files is assigned a unique URL to access it, along with the added benefits of cloud storage, like GeoReplication, and User Authentication!

## Getting Started

Before writing the code for our Xamarin app, let's first use the Azure Portal to [create an Azure Storage account for Blob Storage](https://aka.ms/XamarinBlog/CreateAzureStorageAccount), then upload an image.

### Free Azure Credits

If you are new to Azure, [click here](https://aka.ms/XamarinBlog/FreeAzureCredit/Storage) to get a free $200 credit!

### Create New Azure Storage Resource

1. In the [Azure Portal](https://portal.azure.com), create a new Storage Account.

![Create Azure Blob Storage Container](/guide/resources/CreateAzureStorageContainer.png)

### Create New Blob Container

Now, let's navigate to the newly create Azure Storage resource and upload some photos!

2. In the Azure Notification menu (the bell icon at the top of the Azure Portal), select "Go to resource"

![Navigate to Azure Blob Storage Resource](/guide/resources/NavigateToResource.png)

3. Click on Browse Blobs

![Click On Browse Blobs](/guide/resources/BrowseBlobs.png)

4. Click on "+ Container"
   - **Note**: A container is like a folder in the cloud. In it we will store "Blobs" which is just another name for a file.

5. Name the new container "photos"

6. Set the "Public access level" to "Blob"

    - **Note**: For this example, we are creating a public blob storage container. For future projects that you do not want to be publicly accessible, set the "Public access level" to "Private"

7. Click Ok

![Click On Add New Container](/guide/resources/AddNewContainer.png)

8. Click on the newly created container, "photos"

![Select Photos Container](/guide/resources/SelectPhotosContainer.png)

9. Select Upload

10. Select an image to upload from your computer

    - **Note**: For this example, we'll be using a [Microsoft Logo](https://aka.ms/microsoftlogo)

11. Select Upload

![Select Upload Images](/guide/resources/UploadImages.png)

12. On the Azure Storage menu, select "Access Keys"

13. Copy the "Connection string" for Key1. We'll need this for our Xamarin app!

![Copy Connection String](/guide/resources/ConnectionString.png)

Our Azure Storage resource is set up. Let's start working on our app!

## Create a Simple Xamarin.Forms App

Let's make a simple Xamarin.Forms app that uses our image from Azure Blob Storage. It will be a single-page app that communicates directly with our Azure Blob Storage Container using the [Azure Storage SDK for .NET](https://aka.ms/xamarinblog/azurestoragesdk).

Add the following code to a Xamarin.Forms project, making sure to replace `"Your Connection String"` with the Connection String created in Step 13.

![Simple Xamarin Forms App Using Azure Blob Storage](/guide/resources/XamarinBlobStorageSampleAppDiagram.png)

### App.cs

`App.cs` is the starting point for every Xamarin.Forms app! In `App.cs`, we will set the `MainPage` to be our `ImagePage` which will be wrapped in a `NavigationPage` to provide a title bar.

```csharp
public class App : Application
{
    public App()
    {
        MainPage = new NavigationPage(new ImagePage());
    }
}
```

### ImagePage.cs

`ImagePage` will display our image. In its `OnAppearing` method, which runs every time the page appears in our app, we include the logic to retrieve the image from Blob Storage.

```csharp
public class ImagePage : ContentPage
{
    readonly Label _title = new Label{ HorizontalTextAlignment = TextAlignment.Center };
    readonly Image _image = new Image();
    readonly ActivityIndicator _activityIndicator = new ActivityIndicator();

    public ImagePage()
    {
        Content = new StackLayout
        {
            Spacing = 15,
            HorizontalOptions = LayoutOptions.Center,
            VerticalOptions = LayoutOptions.Center,
            Children = {
                _title,
                _image,
                _activityIndicator
            }
        };

        Title = "Image Page";
    }

    protected override async void OnAppearing()
    {
        base.OnAppearing();

        _activityIndicator.IsRunning = true;

        var blobList = await BlobStorageService.GetBlobs<CloudBlockBlob>("photos");

        var firstBlob = blobList?.FirstOrDefault();
        var photo =  new PhotoModel { Title = firstBlob?.Name, Uri = firstBlob?.Uri };

        _title.Text = photo?.Title;
        _image.Source = ImageSource.FromUri(photo?.Uri);

        _activityIndicator.IsRunning = false;
        _activityIndicator.IsVisible = false;
    }
}
```

### BlobStorageService.cs

`BlobStorageService` is a generic static class that can be used in any project. It contains two public methods: `GetBlobs` and `SaveBlockBlob` which can be called from anywhere in our app to retrieve and upload blobs from/to Blob Storage accordingly.

```csharp
readonly static CloudStorageAccount _cloudStorageAccount = CloudStorageAccount.Parse("Your Connection String");
readonly static CloudBlobClient _blobClient = new CloudBlobClient _cloudStorageAccount.CreateCloudBlobClient();

public static async Task<List<T>> GetBlobs<T>(string containerName, string prefix = "", int? maxresultsPerQuery = null, BlobListingDetails blobListingDetails = BlobListingDetails.None) where T : ICloudBlob
{
    var blobContainer = _blobClient.GetContainerReference(containerName);

    var blobList = new List<T>();
    BlobContinuationToken continuationToken = null;

    try
    {
        do
        {
            var response = await BlobContainer.ListBlobsSegmentedAsync(prefix, true, blobListingDetails, maxresultsPerQuery, continuationToken, null, null);

            continuationToken = response?.ContinuationToken;

            foreach (var blob in response?.Results?.OfType<T>())
            {
                blobList.Add(blob);
            }

        } while (continuationToken != null);
    }
    catch (Exception e)
    {
        DebugServices.Log(e);
    }

    return blobList;
}

public static async Task<CloudBlockBlob> SaveBlockBlob(string containerName, byte[] blob, string blobTitle)
{
    var blobContainer = _blobClient.GetContainerReference(containerName);

    var blockBlob = blobContainer.GetBlockBlobReference(blobTitle);
    await blockBlob.UploadFromByteArrayAsync(blob, 0, blob.Length);

    return blockBlob;
}
```

### PhotoModel.cs

`PhotoModel` is the model class for our photo metadata. It is used to store the photo's Uri and Title.

```csharp
public class PhotoModel
{
    public System.Uri Uri { get; set; }
    public string Title { get; set; }
}
```

## Going Serverless with Azure Functions + Azure Storage

For a more advanced example that incorporates [Azure Functions](https://aka.ms/XamarinBlog/AzureFunctions), check out this sample app:
https://github.com/brminnick/AzureBlobStorageSampleApp

[Azure Functions](https://aka.ms/XamarinBlog/AzureFunctions) are a great compliment to [Azure Blob Storage](https://aka.ms/xamarinblog/azureblobstorage) because they allow us to offload any metadata processing from the Xamarin app to the cloud, and they can help ensure our metadata doesn't get corrupted!

This Xamarin app uses a [SQLite Database](https://github.com/praeclarum/sqlite-net) to store the metadata of the Photos (e.g. Url, Title) locally on our Xamarin device. The local database syncs, via an [Azure Function](https://aka.ms/XamarinBlog/AzureFunctions), with an [Azure SQL Database](https://aka.ms/XamarinBlog/AzureSQL) that contains the metadata of the Photos stored in [Azure Blob Storage](https://aka.ms/xamarinblog/azureblobstorage).

This Xamarin app also allows the user to take photos and save them to [Azure Blob Storage](https://aka.ms/xamarinblog/azureblobstorage). To do this, the Xamarin app uploads the image to an [Azure Function](https://aka.ms/XamarinBlog/AzureFunctions), and the [Azure Function](https://aka.ms/XamarinBlog/AzureFunctions) saves the image in [Azure Blob Storage](https://aka.ms/xamarinblog/azureblobstorage), then adds the image metadata to the [Azure SQL Database](https://aka.ms/XamarinBlog/AzureSQL).

![Azure Blob Storage Sample App Diagram](/guide/resources/AzureBlobStorageSampleAppDiagram.png)

## Learn More

Visit the Microsoft Docs to learn more about Azure Blob Storage and Azure Functions!

- [Azure Blob Storage](https://aka.ms/xamarinblog/azureblobstorage)
- [How to use Blob Storage from Xamarin](https://aka.ms/XamarinBlog/AzureBlobStorageWithXamarin)
- [Azure Functions](https://aka.ms/XamarinBlog/AzureFunctions)
- [Azure Blob Storage + Azure Functions Sample](https://github.com/brminnick/AzureBlobStorageSampleApp)
