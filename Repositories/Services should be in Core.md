# Services should be in .Core
All services should be in core, as they (hopefully) should contain business logic, like getting the content node count, or delivering user-data.
This does come with some problems though, as what if we want a service that retrieves data from the database? 
As a Junior dev, i thought you could just write raw SQL in the service, but that is a no-go, as the core should not know anything about the database, we could technically be using a file-based system or whatever have you.

Repositories to the rescue! While you might still say, but Nikolaj, repositories should be in the infrastructre project, and Core should know anything about the infrastructure project?
Well we can but the repository interface in the Core project, but still have the implementation of said interface in the Infrastructure project.
