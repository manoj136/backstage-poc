## Summary

An admin user from the supplier fills the new contact information and selects the role **"Supplier Admin"** or **"Supplier User"**.

 When clicks the _Create_ button a multi-step asyncrononous process will begin:

- The UserProfile information will be stored in the database
- The user will be created on the IDP.
- A mail to the user notifying that he can now login on the application

## Diagram

```plantuml format="svg_inline"
@startuml
title Creation of a user managed by the supplier 
actor "Suplier\nAdmin User" as Adm #yellow
participant "Frontend" as F <<App>>
participant "UserProfile" as UPA <<Api>>
database "Omega Db" as DB
participant "StockTools" as ST <<Api>>
queue "Rabbit" as R
participant "UserAdd" as UAT <<Trigger>>
participant "Idp" as IDP <<Api>>
participant "UserCreation" as UCT <<Trigger>>
participant "SendWelcome" as SWT <<Trigger>>
participant "EmailService" as EST <<Trigger>>
actor "Suplier\nManaged User" as U 

Adm -> F: **(1)** Create new contact
F -> UPA: **(2)** POST UserProfile
activate UPA
UPA -> ST: **(3)** POST CreateOrSelectGlobalUser
activate ST
ST --> UPA: LegacyId
deactivate ST
UPA -> DB: Store user profile \ninformation
UPA -> R: **(4)** User Activation notification
note right: UserAdd queue
UPA --> F: Result
deactivate UPA
== async request processing ==
R --> UAT: **(5)** User activation notification
activate UAT
UAT -> IDP: **(6)** POST User
activate IDP
IDP --> UAT: UserId
deactivate IDP
UAT -> R: **(7)** User created notification
note left: UserCreation queue
deactivate UAT
== async request processing ==
R --> UCT: **(8)** User created notification
activate UCT
UCT -> UPA: **(9)** PATCH UserProfile
activate UPA
UPA -> DB: Store changes
UPA --> UCT: Result
deactivate UPA
UCT -> R: **(10)** User activation notification
note left: SendWelcome queue
deactivate UCT
== async request processing ==
R --> SWT: **(11)** User activation notification
activate SWT
SWT -> SWT: Fill template
SWT -> R: **(12)** Mail request notification
note left: MailService queue
deactivate SWT
== async request processing ==
R --> EST: **(13)** Mail request notification
activate EST
EST -> U: Welcome mail
deactivate EST
@enduml
```
> Legend

> 1. The **Admin** fills the new contact information and clicks the create button
> 2. The **Frontend** send the user profile information to the **Front-office.UserProfileApi**
> 3. The **Front-office.UserProfileApi** request the user's legacyID from the **Stock-Tools api**, stores the information on the database with the user as active.
> 4. The **Front-office.UserProfileApi** stores a UserActivation notification on the **UserAdd.Queue**
> 5. The **Idp.UserAddTrigger** retrieves the notification and processes it
> 6. The **Idp.UserAddTrigger** sends a request to the **IdpApi** to create the user in the IDP
> 7. The **Idp.UserAddTrigger** sends a UserCreation notification to the **UserCreation.Queue**
> 8. The **Idp.UserCreationTrigger** retrieves and processes the notification
> 9. The **Idp.UserCreationTrigger** sends a request to the **UserProfileApi** to update the idpUserId
> 10. The **Idp.UserCreationTrigger** sends a user activation notification to the **SendWelcome.Queue**
> 11. The **MailingService.SendWelcomeTrigger** retrieves the notification from the
    **SendWelcome.Queue**, retrieves the corresponding mail template,fills it out and sends a
    notification to the **MailService.Queue**
> 12. The **MailService.EmailServiceTrigger** sends the mail to the user

