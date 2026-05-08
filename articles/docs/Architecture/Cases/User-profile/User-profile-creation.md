## Short summary

The administrator can create users via the front-end application without the need for preexisting user registration.

Users can be created individually or in bulk (by uploading an Excel with the user information) via the front-end application

When users are created via a bulk upload, they will be created in the Omega database with a status that indicates that
they were created via a bulk upload.
In this case instead of waiting for all the users to be created, the request is stored in the UploadBulkUsers.Queue to be processed asyncronously

In both cases (individual and bulk creation) the required user configuration will be checked and completed by the administrator, and the user's status will be changed to "pending confirmation".

To enable the access to the application, the administrator will use the "confirm user" action, and the [User Confirmation Flow](./User-profile-confirmation.md) will start

## Diagram

### Individual User Creation
```plantuml format="svg_inline"
@startuml
title User profile creation
hide footbox
actor Admin as U #yellow
participant "Frontend" as F <<App>>
participant "UserProfile" as UPA <<Api>>
database "Omega Db" as DB

U -> F: Create user
F -> UPA: POST UserProfile
activate UPA
UPA -> DB: Stores user profile information
UPA --> F: Result
deactivate UPA
@enduml
```
### Bulk user creation

> Note: 
The following diagram ilustrates the flow of a single upload of file.

```plantuml format="svg_inline"
@startuml
title Bulk User profile creation
hide footbox
actor Admin as U #yellow
participant "Frontend" as F <<App>>
participant "UserProfile" as UPA <<Api>>
database "Blob Storage" as Blob
queue Rabbit as R
database "Omega Db" as DB
participant "UploadBulkUsers" as UBUT <<Trigger>>

U -> F: Upload excel
F -> UPA: POST UserProfile
activate UPA
UPA -> Blob: Stores the Excel File
UPA -> R: Bulk Upload Request
note right: UploadBulkUsers queue
UPA --> F: Result
deactivate UPA
== async processing of the request ==
R -->> UBUT:  Bulk upload request
activate UBUT
Blob -> UBUT: Retrieves excel
loop "for each user on the file"
    UBUT -> UPA: POST UserProfile
    activate UPA
    UPA -> DB: Stores user profile information
    UPA --> UBUT: Result
    deactivate UPA
end
UBUT -> Blob: Stores excel with results
deactivate UBUT
@enduml
```
