# Android-SDK

## About the Medable SDK

Welcome to the Android Client SDK for [Medable](https://www.medable.com). This library provides a wrapper for the [Medable API](https://dev.medable.com/) as well as several helper tools to directly integrate those features to your Android application.

The Medable Android SDK is targeted to be used in apps that have a _minSdkVersion_ at 14 and the following instructions are targeted for Android Studio users, consider updating if you have an older version.

## Integration steps

### Gradle repo and dependencies
Add the following repository.

```groovy
defaultConfig {
    ...

    repositories {
        maven { url "https://github.com/Medable/Android-SDK/raw/master/" }
    }
}
```

Add the following dependency.

```groovy
dependencies {
    compile 'com.medable:AndroidSDK:0.1.+'    // or use the last version number instead of 0.1.+
}
```

Synch Gradle, and you're set!

## Code setup

### Initialization of the _APIClient_ class

This _APIClient_ class is a singleton and has convenience methods used to communicate with Medable servers. 
The first thing to do is to configure the singleton instance. For that, what's needed is a base URL, an Org and an App created on a Medable server.

```java
APIClient.Builder
        .applicationContext(getApplicationContext())
        .baseURL(kMedableEndpoint)
        .org(kOrg)
        .clientKey(kAppKey)
        .build(new FaultCallback()
        {
            @Override
            public void call(Fault fault)
            {
                if (fault != null)
                {
                    // Handle fault
                }
            }
        });
```

There are some optional configuratio parameters on the _APIClient.Builder_ builder class. 
For example to change the log level you need to use this builder parameter:
```java
.logLevel(APIClient.LogLevel.HEADERS_AND_ARGS)
```

### Content Downloader

There is a class that downloads information needed for the proper function of the SDK; like the Org's bundle information or all the definitions for the Org's objects. The default behavior is to automatically download that information when the _APIClient_ singleton is initializing. But if you need to make these downloads happen elsewhere in your app, you can turn that off and call the proper method when needed. Just make sure to get the info before any Medable related operation.

```java
// add this when calling APIClient.Builder...
.checkForContentDownload(false)

// when you're ready for the download, call this
ContentDownloader.checkForDownloads(new FaultCallback()
{
    @Override
    public void call(Fault fault)
    {
        // handle fault if necessary
    }
});
```

### GCM Notifications manager

On the SDK, there's a _GCMManager_ class that takes care of the registration of your app on GCM servers for you. All the notifications sent from Medable servers to your app will be received and stored by this manager.

As you may now, to be able to receive Google Cloud Messaging notifications, you need to first configure your project on the [Google Developer Console](https://console.developers.google.com/), and get a Sender ID for your project.

Then configure the _GCMManager_ singleton class, like this: 
```java
GCMManager.Builder
        .applicationContext(aContext)
        .senderID(aSenderId)
        .build();
```

### Handling particular events by listening to notifications.

There's a _NotificationCenter_ class that post API notifications you could use to hook to certain events. The notifications are defined in _com.medable.AndroidSDK.Constants_.

kMDNotificationAPIServerErrorDidOccur: This notification comes with the corresponding Fault object, ready to be handled. Fault codes are defined in _com.medable.AndroidSDK.Constants_.

kMDNotificationUserDidLogout: This notification is used to forward the app to the login screen. This notification is thrown when the user logs out, or when API session is over because of an error or session timeout.

To listen to a particular notification, you need to add an observer for that notification. Like this:
```java
// listens for sucessfull upload operations

NotificationCenter.defaultCenter().registerObserver(this, Constants.kMDNotificationUploadOperationEndedSuccessfully, new BroadcastReceiver()
{
    @Override
    public void onReceive(Context context, Intent intent)
    {
        AssetUploader.UploadOperation fileUpload = (AssetUploader.UploadOperation) intent.getSerializableExtra(Constants.kObject);
    }
});
```

Example use cases - going beyond simple API calls
------
The following use cases reference many Medable objects. Please refer to the <a href="https://dev.medable.com/#objects" target="_blank">API documentation</a> for more information on Medable objects and their properties.

### How do I get the images (attachments) for a <a href="https://dev.medable.com/#conversation-attachments-property" target="_blank">conversation</a>?
```java
Conversation conversation = ...;
conversation.imagesWithCallback(new PicsUpdateCallbackBlock()
{
    @Override
    public void call(String imageId, Bitmap bitmap, DataSource source, Fault fault)
    {
        if (null == fault && null != bitmap)
        {
            
        }
    }
});
```

### How do I get the thumbnail of an <a href="https://dev.medable.com/#account-image-property" target="_blank">account</a>?
```java
Account account...;
account.thumbnailWithCallback(new BitmapCallbackBlock()
{
    @Override
    public void call(Bitmap bitmap, DataSource source, Fault fault)
    {
        if (null == fault && null != bitmap)
        {
            
        }
    }
});
```

### How do I get the thumbnail of a <a href="https://dev.medable.com/?android_sdk#connections" target="_blank">connection</a>?
```java
Connection connection...;
connection.thumbnailWithCallback(new BitmapCallbackBlock()
{
    @Override
    public void call(Bitmap bitmap, DataSource source, Fault fault)
    {
        if (null == fault && null != bitmap)
        {
            
        }
    }
});
```

### How do I use the synchronize methods?

All objects have three synchronize methods that receive parameters such as includes, expands, paths filters, etc --
one for the object itself, one for posts, and one for connections. The reason for three methods instead of one is that each uses the base route to each resource, providing for more access, and more options for the parameter inputs.

A conversation expanding its patientFile property:
```java
Conversation conversation;
conversation.synchronizeObjectWithParameters(
        APIParameterFactory.parametersWithExpandPaths(Constants.kPatientFile),
        new ObjectFaultCallback<Conversation>()
        {
            @Override
            public void call(Conversation object, Fault fault)
            {
                
            }
        }
);
```

An account expanding account.connections.context:
```java
Account account;
account.synchronizeConnections(
        APIParameterFactory.parametersWithExpandPaths(Constants.kContext),
        new ObjectFaultCallback<ObjectInstance>()
        {
            @Override
            public void call(ObjectInstance updatedAccount, Fault fault)
            {
                
            }
        }
);
```

An account with its posts:
```java
Account account;
account.synchronizePosts(
        null,
        new ObjectFaultCallback<ObjectInstance>()
        {
            @Override
            public void call(ObjectInstance updatedAccount, Fault fault)
            {
                
            }
        });
```

### How do I use the asset manager to encrypt and cache media?
```java
Bitmap image;
int quality = 100;
String filename = "aGoodFileNameHere";

AssetManager.sharedInstance().saveImage(
        image,
        Bitmap.CompressFormat.JPEG,
        quality,
        filename,
        new BooleanCallback()
        {
            @Override
            public void call(boolean success)
            {
                
            }
        }
);
```
That image is saved in an encrypted cache on disk, which works on a per user basis and is encrypted with a different key for different users. Those keys change when the account's password is changed, as explained in the API documentation.

### How do I use the SDK to manage <a href="https://dev.medable.com/#custom-objects" target="_blank">custom objects</a>?
```java
// Let's suppose you have a custom object named 'c_aCustomObject', with a string property called 'c_aCustomProp1' and a number property called 'c_aCustomProp2'
// in addition to the base properties, inherited from MDObjectInstance, which is the base class for all objects.

// Get its definition
ObjectDefinition aCustomObjectDefinition = SchemaManager.sharedInstance().getDefinition("c_aCustomObject");

// You could get all of its custom property definitions like this:
List<PropertyDefinition> customProperties = aCustomObjectDefinition.getCustomProperties();

// Or you could get the properties definitions by name, like this:
PropertyDefinition aCustomProp1Definition = aCustomObjectDefinition.propertyWithName("c_aCustomProp1");
PropertyDefinition aCustomProp2Definition = aCustomObjectDefinition.propertyWithName("c_aCustomProp2");

// From there you could check what type of primitive each property is
PropertyType type1 = aCustomProp1Definition.getType();      // That type would be PropertyType.String
PropertyType type2 = aCustomProp2Definition.getType();      // That type would be PropertyType.Number

// Also, you can check if that object has a feed definition, and access all the post types it supports, each post's segments, comment definitions, etc
// Starting from here:
FeedDefinition aCustomObjectFeedDefinition = aCustomObjectDefinition.getFeedDefinition();

// We could get all the posts' definitions for the posts its feed accepts
List<PostDefinition> postDefinitions = aCustomObjectFeedDefinition.getPostDefinitions();

// That would be an array of these objects:
PostDefinition postDefinition = postDefinitions.get(0);

// Let's check the different segments this type of post has
List<PostSegmentDefinition> postSegments = postDefinition.getBody();

PostSegmentDefinition aPostSegmentDefinition = postSegments.get(0);

// And comment definition for this particular type of post.
// Notice that the comments definitions are also segment definitions, since they share the same structure.
PostCommentDefinition commentDefinition = postDefinition.getCommentDefinition();

// Ok, enough of structure for now. That's the definition world. Then we have the instances world. That is, object instances with all those
// definitions. Let's say we want to modify an instance of this custom object of type 'c_aCustomObject'.
ObjectInstance aCustomObjectInstance;

Map<String, Object> body = new HashMap<>();
body.put(aCustomProp1Definition.getName(), "a string value for prop 1");
body.put(aCustomProp2Definition.getName(), 7);

FileAttachments fileAttachments;

APIClient.sharedInstance().updateObject(
        aCustomObjectInstance.getObject().pluralNameForAPICalls(),
        aCustomObjectInstance.getId(),
        body,
        fileAttachments,
        new ObjectFaultCallback<ObjectInstance>()
        {
            @Override
            public void call(ObjectInstance editedObject, Fault fault)
            {
                if (null == fault)
                {
                    // You did great, and successfully modded the instance.
                }
            }
        }
);

// As you see, you can get any information you need form an object, either custom or not, by accessing its definitions' reference properties.
```

### How do I post to a feed?

##### References
- [File](https://dev.medable.com/?objective_c#files)
- [Post](https://dev.medable.com/?objective_c#posts)

##### Sample code
```java
/*
   - In this case, let's suppose we have an object whose feed definition consists of posts of type "c_post".
   - Each "c_post" has two body segments called "c_text" --of type String, and "c_image" --of type File[], that is, an array of File objects.
   - Each File object is composed by four facets, called "c_original", "c_overlay", "c_thumbnail" and "c_content".
   - There are two facet processors, that take "c_original" and "c_overlay" as inputs and produce "c_content" --the original image with the overlay included, and "c_thumbnail" --a reduced version of "c_content".
*/

// Let's create a post with a message saying "Hi, this is a post with text and images", and two images, one with overlay and one without overlay.

// The context object
ObjectInstance contextObject;

// Message
String text = "Hi, this is a post with text and images";

// Images
// faceImage is the Bitmap with the picture of the patient's face
// chestImage is the Bitmap with the picture of the patient's chest
// faceOverlay is the Bitmap with the transparent overlay to cover the patient's face
Bitmap faceImage;
Bitmap faceOverlay;
Bitmap chestImage;

// Prepare the body segments
//NSMutableArray *postSegments = [NSMutableArray new];

// Get the object's feed definition
FeedDefinition feedDefinition = contextObject.getObject().getFeedDefinition();

// Get the post's definition
PostDefinition postDefinition = feedDefinition.postDefinitionWithType("c_post");

List<PostSegmentStub> postSegments = new LinkedList<>();
FileAttachments attachments = new FileAttachments();

for (PostSegmentDefinition segmentDefinition : postDefinition.getBody())
{
    // Configure the Text segment
    if (segmentDefinition.getName().equals("c_text") && text.length() > 0)
    {
        PostSegmentStub textSegment = new PostSegmentStub("c_text");

        // Get the post segment property definition
        textSegment.addProperty("c_text", text);

        // Add the post body's text segment
        postSegments.add(textSegment);
    }

    // Configure the Image segment
    else if (segmentDefinition.getName().equals("c_image"))
    {
        // Get the post segment property's definition
        PostSegmentPropertyDefinition propertyDefinition = segmentDefinition.propertyWithName("c_image");

        PostSegmentStub imagesSegment = new PostSegmentStub(segmentDefinition.getName());

        // It's an array of Files... In this case, we have 2 attachments
        Attachment faceAttachment = new Attachment(propertyDefinition);
        faceAttachment.addFacetAttachment("c_original", "image/jpeg", faceImage);
        faceAttachment.addFacetAttachment("c_overlay", "image/png", faceOverlay);

        Attachment chestAttachment = new Attachment(propertyDefinition);
        chestAttachment.addFacetAttachment("c_original", "image/jpeg", chestImage);
        
        // Add the attachments to the container (FileAttachments)
        attachments.addAttachment(faceAttachment);
        attachments.addAttachment(chestAttachment);
    }
}

APIClient.sharedInstance().postToObject(
        contextObject.getObject().pluralNameForAPICalls(),
        contextObject.getId(),
        "c_post",
        postSegments,
        attachments,
        null,
        null,
        new ObjectFaultCallback<Post>()
        {
            @Override
            public void call(Post post, Fault fault)
            {
                if (null != fault)
                {
                    // Fault handling
                }
            }
        }
);
```

### Why can't I mod an object instance? why is that there are no setters?

All the modifications need to go through the api, and then you need to synchronize your object instance. This is to make it easier to avoid inconsistences between the client and the api.
That’s why all the modifications are done through APIClient object modification methods.


### About parameters validation in APIClient

APIClient's header is annotated with nullability annotations; this enables compile time parameter nullability checksfor the methods listed in APIClient's header file.

Besides that, required parameters are checked at runtime. If something is wrong, you'll get a _LocalFault_ object in any method's callback block, providing validation error details. Also, these _LocalFault_ obejcts are broadcasted inside notifications using the _NotificationCenter_ singleton. The notification name is **kMDNotificationLocalFaultNotification**. These are specially useful for when the validation error is that there's no callback block.

In general, LocalFault objects are used to inform the user about error that are not related to the API server (Fault objects are used for this), or when some API unrelated network issue was detected.

You can handle such notifications as described above in the "Handling particular events by listening to notifications" section.
