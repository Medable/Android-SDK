# Cortex for Android
---

### Change log

#### v1.1.7

- **Context Class Registration**: Added support for context class registration.
- **Data Uploads** - Added support for upload headers auto refresh when they are expired.
- Disabled location manager, removed location permission.
- Bug fixes and improvements.
- `APIClient` 
   - Added support for **custom login routes**.
   - Added support for **custom HTTP headers**.
   - Support for primitive Json request bodies.
   - Upload Image made public.
- `APIParameterFactory`
   - Added basic support for pipelines.
   - Marked several factory methods as deprecated.
- `ImageOverlayPair` - Marked as deprecated.
- `Utils`
   - Added parser for JsonObject into a T class.
   - Added json parser for OkHttp Response.
- `FastBodyProperty` - Added default constructor and `addAttachment` returns the generated file name now.
- `BaseInstance` - Support for pathing into Document properties' sub-properties.
- **Deprecations**
   - `Conversation`
   - `Team`
   - `PatientFile`
   - Feed related: `Post`, `PostComment`, etc.
   - Several `APIParameter` factory methods in `APIParameterFactory`.
   - `APIClient`: Methods related to the avobe mentioned objects.

#### v1.1.6

- `APIClient` - Handle Faults in HTTP successful callbacks.
- `BaseDefinition` - Added support for sub-classes / type objects properties.
- Bug fixes and improvements.

#### v1.1.5

- APIParameters - Improved the way parameters are constructed.
- APIParameters - Added support for query operators:
   - Comparison Operators
   - Logical Operators
   - Evaluation Operators
   - Array Operators
- APIClient - Added a way to download a file by path.
- Bug fixes and improvements.

#### v1.1.4

- Added support for additional date format.
- Bug fixes

#### v1.1.3

- Logging api calls with FULL flags now works properly.
- Bug fixes and improvements.
    
#### v1.1.2

- Added a failsafe for NumberPropertyInstance.

#### v1.1.1

- Api responses will return the correct fault in case there is an error in the request.

#### v1.1.0

- Fixed package names: Changed to lowercase.
- Reduced method count.
- Other minor improvements and bug fixes.

---
