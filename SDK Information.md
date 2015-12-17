#Medable Android SDK
-

Welcome to the Android Client SDK for [Medable](https://www.medable.com). This library provides a wrapper for the [Medable API](https://dev.medable.com/) as well as several helper tools to directly integrate those features to your Android application.

Take a look at our main [documentation](https://dev.medable.com) page.

For SDK integration steps, please refer to the [Readme.md](https://github.com/Medable/Android-SDK) displayed on the SDK's main page on Github.

##Main Classes

###APIClient
This is one of the most important classes. It is used to communicate with [Medable](https://www.medable.com) servers.

It is a singleton instance, and among its main responsibilities we find:

- All networking tasks. Communication with [Medable](https://www.medable.com) servers.
   - Session. Login / Logout.
   - Account registration / activation / 2fa.
   - Password resetting, lost password methods.
   - [Medable objects](https://dev.medable.com/?android_sdk#objects) CRUD methods for **base** and **custom** objects.
      - Among these, we have list methods that receive all the valid query parameters. Pagination is supported through query parameters.
   - Medable feed CRUD methods for [Post](https://dev.medable.com/?android_sdk#posts) and [Post Comments](https://dev.medable.com/?android_sdk#comments). 
   - Create / Accept / Reject / Modify / Delete [Connections](https://dev.medable.com/?android_sdk#connections).
   - Methods to manage Medable [Notifications](https://dev.medable.com/?android_sdk#notifications).

- Initialization of the SDK.
- Holds current user [Account](https://dev.medable.com/?android_sdk#accounts) instance.
- Holds current org [Org](https://dev.medable.com/?android_sdk#organizations) instance.
- Accessor methods for Invitation token.
- Accessor methods for the GCM regId.
- Upload / Download data methods.

Some useful tools held by the APIClient are:

- Two [Gson](https://github.com/google/gson) instances configured to be used with this Android SDK. One is for pretty JSON formatting.
- A [JsonParser](https://google-gson.googlecode.com/svn/trunk/gson/docs/javadocs/com/google/gson/JsonParser.html) instance to mostly parse JSON strings into Gson objects.

-
###Model Objects

The Medable API defines operations that are directly related mainly to [Context Objects](https://dev.medable.com/?android_sdk#objects). 

There are two groups of context objects:

- **Base** context objects: that are already defined and ready to be used out of the box. The main ones are:
   - [Account](https://dev.medable.com/?android_sdk#accounts).
   - [Patient Files](https://dev.medable.com/?android_sdk#patient-files).
   - [Care Conversations](https://dev.medable.com/?android_sdk#care-conversations).
   - [Teams](https://dev.medable.com/?android_sdk#teams).
   - [Organizations](https://dev.medable.com/?android_sdk#organizations).
   - [Connections](https://dev.medable.com/?android_sdk#connections).
   - [Notifications](https://dev.medable.com/?android_sdk#notifications).
   - [Posts](https://dev.medable.com/?android_sdk#posts).
   - [Comments](https://dev.medable.com/?android_sdk#comments).

- **Custom** context objects: custom user-created objects. These custom objects are created through our web-based [Medable API app]().

In the Android SDK, there are model objects for all the base context objects and also there is a base model object for the custom ones and methods to handle any (base and custom) object in a generic way.

That is achieved using a common base class called **ObjectInstance**.

The generic handling of objects is extended among other objects that are not context objects (like Connections for example), using, once again, another common class called **BaseInstance**.

Server side, the objects are defined using JSON schemas. Those schemas are downloaded from the server and are used to construct the "definition" objects. For this, we have on the other hand the definitions classes: **BaseDefinition** and **ObjectDefinition**. The class in charge of the schemas and definitions is the **SchemaManager**; which is a singleton.

This definition/instance model is also used at a property level. So, there is a **PropertyDefinition** and a **PropertyInstance**.

This generic model, is what allows us to handle any object in a very generic way. In other words we can use Account objects using its accessor methods or using the custom way.

-
###Objects Feed 
#### The 'custom' structure
Any context object can have a feed; and then users can post messages and comments to it. Feeds also have a definition, and its posts and the posts' comments too.

Hence, we have a **FeedDefinition** object for each **ObjectDefinition**.

The **FeedDefinition** in turn, is composed by 0 or more **PostDefinition** objects.

The **PostDefinition** in turn, defines the structure of the posts that can be posted on 'this particular' object's feed. Also the structure of the comments that can used as replies to 'this particular' kind of posts is defined here too.

Post and PostComments are defined in terms of segments. Each segment has a type, a name and the properties we want to be involved. So, there's a **PostSegmentDefinition** object that represents the definition of Post and PostComment segments.

And finally, inside the post segments, we find the actual primitive properties (**PostSegmentPropertyDefinition**).

-
###APIParameterFactory
This class is used to create query parameters to feed **APIClient** methods that take **APIParameters** arguments. Check our main [documentation](https://dev.medable.com) page for the valid query parameters on each particular case.

###Body helpers
Inside the **com.medable.AndroidSDK.API.helpers** package, there are some useful classes to construct both **Body** and **PostBody** objects, mostly involved in the CRUD methods for context objects and feed related objects.

The body constructed body structures must follow the same body structure of the object you are trying to create or update.

The Medable API defines some property primitive types like: String, Number, Boolean, Date, ObjectId, Reference, Document and File. And the array version of any of these (i.e. 'Document[]', 'File[]', 'String[]').

Each type of property has its own body helper class that represents it. Also we have container classes to compose bodies with more than one property.

The containers are:

- **Body**: Used in context objects CRUD methods. It is a container class. All the body properties instances are added to this container prior sending the information to our servers.
- **PostBody**: Used in feed related objects CRUD methods. It is a container class. All the post / post comment body properties instances are added to this container prior sending the information to our servers.

The body property instances, that can be added to the above mentioned containers, are:

- **SimpleBodyProperty**: Represents a "simple" property and its value. By simple I mean, it is not an Array, or a Document or a File property.
- **ArrayBodyProperty**: Represents an array property and its elements. The elements inside are instances of any other **BodyProperty**. Notice that there could be an array of any primitive type.
- **DocumentBodyProperty**: Represents a Document property and its sub properties.
- **FileBodyProperty**: Represents a File property and its data attachments.

The constructor for all the above take at least one argument; the **PropertyDefinition** of the involved property. Then you provide the value for that property and then add that **BodyProperty** right to either the **Body** or the **PostBody** container.

That creates the validated JSON body the Medable server expects to get for each service call.

Despite it is secure, you might find it a slow process. So, if you know what is the right JSON body; there is a **FastBodyProperty** that takes a Gson's **JsonObject** as the constructor argument, and that's exactly what is going to be sent as JSON body. The only drawback of this approach is that you cannot use the fast approach for **File** attachments --for that use **FileBodyProperty** instead.

-
###HIPAA compliance and the AssetManager
One of the key features of Medable is that it is [HIPAA](http://www.hhs.gov/ocr/privacy/hipaa/understanding/) compliant. To ensure this we store all the media encrypted on disk. If you need to do the same, use the **AssetManager** singleton instance.

-
###Pagination
Object pagination on list calls is achieved using query parameters (APIParameters built with APIParameterFactory).

Besides that, there are some helpers to help make this even easier.
The pagination helper classes are: 

- **ContextObjectPaginationHelper**: For context object pagination.
- **FeedPaginationHelper**: For feed (posts) pagination.

-
###Local Notifications
Similar to Apple's iOS, we've created a **NotificationCenter** inside the Android SDK, used to post local notifiactions.

-
###Utils
And last but not least, there's a bunch of useful static methods at your disposal inside the **Utils** class, to do common tasks as *create paths*, *parse Date objects from a string*, etc.

One of the most useful util methods is the *findPropertyDefinition* that takes a property definition path strign as parameter, and gets the requested **PropertyDefinition** object for you.

Since the path argument is a mere String, there is a path format described in this methods *javadoc* documentation.