# JESR - Just Easy Selection & Retreival
PDFs are widely used for invoice storage and invoice sharing, yet extracting information embeded within PDFs, especially tables, can be challenging and time-consuming. Many organisations struggle to retrieve textual data for further processing, often relying on manual methods or expensive third-party tools that don't always give enough flexibility or accuracy to complete the desired task.

### Problems with PDF
- **Lack of Structure**: PDFs prioritise visual layout over data hierarchy.
- **Rigid Tooling**: Most third-party tools are "black boxes" that don't allow for custom extraction logic or integration into existing workflows.
- **Manual Overhead**: Organisations still spend thousands of man-hours manually re-entering data into spreadsheets.

## Motivation:
- Learn How to Build Microservices – By designing a modular system that processes PDFs, extracts tables, and serves structured data, I will explore the principles of service decomposition, scalability, and communication.
- Understand the Technology Choices – Dynamic Rendering, RestAPIs, Containerisation, SpringBoot.
- Provide a free alternative

## System Architecture:
The project consists of multiple services working together to each with an individual purpose.

### PDF_Service_Web
PDF_Service_Web is a lightweight stateless web server that hosts the user interface for interacting with internal services. It is designed as a Single Page Application (SPA) and provides a responsive and modern user experience using Go templates and HTMX.

- Built in [Go](https://go.dev/) with the [Gin](https://github.com/gin-gonic/gin) high-performance HTTP web framework
- Renders a dynamic SPA using Go's `html/template` and [HTMX](https://htmx.org/)
- Real-time toast notifications pushed to connected users.
- Proxies requests to PDF_Service_API, and forces authorisation to restricted endpoints.
- Connects to a instance of [Keycloak](https://www.keycloak.org/)
- Stateless Architecture: Built on a stateless architecture for horizontal scalability.

#### Toast Notifications
The server provides a dedicated endpoint that enables toast-style notifications. These are managed across active sessions and users, which allows for: system-wide alerts, session specific messages, user wide alerts, updates and visual feedback without requiring full page reloads. This system is optional, if a session fails to connect to the endpoint, redirects act as a stand-in replacement.

#### Why Keycloak?
We use Keycloak as our primary oAuth2 server, enabling role-based access control and resource management. 
- Role-based Resource Management.
- Built-in JWT mangement.
- Easy to use web interface for administrative tasks.

### PDF_Service_Api
PDF_Service_Api is our stateless Orchestration Layer, handles the communication between the user-facing frontend and specialised internal services.

- Built in [Go](https://go.dev/) with the [Gin](https://github.com/gin-gonic/gin) high-performance HTTP web framework.
- Utilises use of the database PostgreSQL for storing documents, metadata, and user ownership information.
- Reliability & Integrity: It has a comprehensive set of 50 integration tests which ensure the reliability of the API contract and the entire orchestration workflow.
- Stateless Architecture: Built on a stateless architecture for horizontal scalability.
  
#### Why PostgreSQL?
We use PostgreSQL as our primary "[single point of truth](https://en.wikipedia.org/wiki/Single_source_of_truth)" because of its:
- ACID Compliance: Guarantees that document status changes and transfer are done accurately.
- JSONB Support: Allowing us to store flexible PDF metadata alongside structured relational data.
- Performance: Handling complex relational queries efficiently.

### PDF_Service_Data
PDF_Service_Data is a specialised Java-based processing engine designed to transform raw PDF documents into a JSON dataset; It handles the compute heavy tasks.

- Core Functionality: Performs processing on base64 PDF files to generate a JSON schema, mapping coordinates to text items based on rendered images.
- Visual Rendering: Generates visuals for document pages that will be used to render the documents in the front end.
- Metadata Identification: Extracts relevant metadata information from document structures like the total number of pages, exact size (height/width).
- Page Duplicate Removal: Assigns a unique hash value to each page; any duplicates are removed automatically.
- Stateless Architecture: Built on a stateless architecture for horizontal scalability.

## Project Demonstration
### Login Page
| Initial Page  | Missing Required Fields | Incorrect Credentials |
| :---: | :---: | :---: |
| <img width="239" height="290" alt="image" src="https://github.com/user-attachments/assets/b47abadf-9ed6-4392-9b37-0de5e54de7bd" /> | <img width="239" height="290" alt="image" src="https://github.com/user-attachments/assets/ba976d28-58c2-4773-a471-47810009da06" />  | <img width="239" height="290" alt="image" src="https://github.com/user-attachments/assets/3ded6071-1b01-41d4-9e09-df2bf753774c" /> |
| Image showing the extreamly basic login UI, containing both a login and register page.  | Shows an error message that informs that both fields must be filled before they attempt to login.  | Informs the user that the server has returned 403 forbidden.  |


## Project Setup
Each repository will contain README.md instructions on how to configure, run and test each respository.

## Contributing
Pull requests are welcome. For major changes, please open an issue first
to discuss what you would like to change.

Please make sure to update tests as appropriate.
