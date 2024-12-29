![DomainDrivenHexagon](https://github.com/user-attachments/assets/88d40c61-6733-4ed9-8d1c-8a1c5998b7ba)

In short, data flow looks like this (from left to right):

- Request/CLI command/event is sent to the controller using plain DTO;
- Controller parses this DTO, maps it to a Command/Query object format and passes it to an Application service;
- Application service handles this Command/Query; it executes business logic using domain services and entities/aggregates and uses the infrastructure layer through ports(interfaces);
- Infrastructure layer maps data to a format that it needs, retrieves/persists data from/to a database, uses adapters for other I/O communications (like sending an event to an external broker or calling external APIs), maps data back to domain format and returns it back to Application service;
- After the Application service finishes doing its job, it returns data/confirmation back to Controllers;
- Controllers return data back to the user (if application has presenters/views, those are returned instead).

Each layer is in charge of its own logic and has building blocks that usually should follow a [Single-responsibility principle](https://en.wikipedia.org/wiki/Single-responsibility_principle) when possible and when it makes sense (for example, using `Repositories` only for database access, using `Entities` for business logic, etc.).

**Keep in mind** that different projects can have more or less steps/layers/building blocks than described here. Add more if the application requires it, and skip some if the application is not that complex and doesn't need all that abstraction.

General recommendation for any project: analyze how big/complex the application will be, find a compromise and use as many layers/building blocks as needed for the project and skip ones that may over-complicate things.

More in details on each step below.
