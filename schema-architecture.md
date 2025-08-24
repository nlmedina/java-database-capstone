This Spring Boot application uses both MVC and REST controllers. Thymeleaf templates are used for the Admin and Doctor dashboards, while REST APIs serve all other modules. The application interacts with two databases—MySQL (for patient, doctor, appointment, and admin data) and MongoDB (for prescriptions). All controllers route requests through a common service layer, which in turn delegates to the appropriate repositories. MySQL uses JPA entities while MongoDB uses document models.

1. Users access the application through Thymeleaf-based web dashboards or REST API clients like mobile apps.
2. The user's action is routed to the appropriate Thymeleaf controller for web pages or a REST controller for API calls.
3. The controller calls the service layer, which contains the core business logic and validation rules.
4. To access data, the service layer communicates with the repository layer, which includes repositories for both MySQL and MongoDB.
5. The repository interfaces directly with the corresponding database engine, either MySQL for structured data or MongoDB for document-based data.
6. Data retrieved from the database is mapped into Java model classes, such as JPA entities for MySQL or document objects for MongoDB.
7. Finally, these models are either passed to Thymeleaf to be rendered as HTML or serialized into JSON and sent back as an HTTP response to the client.
