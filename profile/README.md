# JESR - Just Easy Selection & Retreival
PDFs are widely used for invoice storage and invoice sharing, yet extracting information embeded within PDFs, especially tables, can be challenging and time-consuming. Many organisations struggle to retrieve textual data for further processing, often relying on manual methods or expensive third-party tools that don't always give enough flexibility or accuracy to complete the desired task.
<br />

### Problems with PDF
- **Lack of Structure**: PDFs prioritise visual layout over data hierarchy.
- **Rigid Tooling**: Most third-party tools are "black boxes" that don't allow for custom extraction logic or integration into existing workflows.
- **Manual Overhead**: Organisations still spend thousands of man-hours manually re-entering data into spreadsheets.

### Motivation:
- Learn How to Build Microservices – By designing a modular system that processes PDFs, extracts tables, and serves structured data, I will explore the principles of service decomposition, scalability, and communication.
- Understand the Technology Choices – Dynamic Rendering, RestAPIs, Containerisation, SpringBoot.
- Provide a free alternative
<br />

## System Architecture:
The project consists of multiple services working together to each with an individual purpose.
<br />

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
<br />

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
<br />

### PDF_Service_Data
PDF_Service_Data is a specialised Java-based processing engine designed to transform raw PDF documents into a JSON dataset; It handles the compute heavy tasks.

- Core Functionality: Performs processing on base64 PDF files to generate a JSON schema, mapping coordinates to text items based on rendered images.
- Visual Rendering: Generates visuals for document pages that will be used to render the documents in the front end.
- Metadata Identification: Extracts relevant metadata information from document structures like the total number of pages, exact size (height/width).
- Page Duplicate Removal: Assigns a unique hash value to each page; any duplicates are removed automatically.
- Stateless Architecture: Built on a stateless architecture for horizontal scalability.

<br />

## Project Demonstration
### Login Page
| Initial Page  | Missing Required Fields | Incorrect Credentials |
| :---: | :---: | :---: |
| <img width="239" height="290" alt="image" src="https://github.com/user-attachments/assets/b47abadf-9ed6-4392-9b37-0de5e54de7bd" /> | <img width="239" height="290" alt="image" src="https://github.com/user-attachments/assets/ba976d28-58c2-4773-a471-47810009da06" />  | <img width="239" height="290" alt="image" src="https://github.com/user-attachments/assets/3ded6071-1b01-41d4-9e09-df2bf753774c" /> |
| Image showing the extreamly basic login UI, containing both a login and register page.  | Shows an error message that informs that both fields must be filled before they attempt to login.  | Informs the user that the server has returned 403 forbidden.  |
<br />

### PDF Dashboard
The dashboard uses a very simple layout with a basic navigation bar. The documents section consists of a pageinated view of docments uploaded by the user; you can select how many rows each page should contain.

<img width="681" height="418" alt="image" src="https://github.com/user-attachments/assets/87082830-2d1a-4d16-9a2c-422f9dcd496c" />

#### Actions
Each row has a couple of actions that the user can preform: view and delete.
- View - Renders the document, and allow the user to view it. This is the main view that will allow users to select areas of a document for targeted extraction.
- Delete - Sends a request to the API server to delete the document and any infomation connected to its ID; including meta data and images. 

#### Upload Popup
The upload button spawns a popup that can be used to upload pdf documents, also incorporating a drag and drop feature.

<img width="547" height="328" alt="image" src="https://github.com/user-attachments/assets/ae437ff0-1302-494e-807e-2cc50a4c900f" />

<br />

### Pdf Viewer
The PDF Viewer is tailored to provide precise interactions with any document. This interface presents documents in the form of a continuous list of page images.
- Fine-grain Extraction: Users can manipulate particular pages and make selections of certain areas. Such selections are then utilised in data extraction.
- Zoom Invariant Selection: The viewer offers flexible zoom in and out capabilities. Relative coordinate mapping is utilised by the selection engine to pin selections to the document irrespective of the current zoom (NOTE: works with browser zoom, tested on chrome & firefox).

#### UX Design
The selection process was designed to provide as much visual feedback as possible without compromising simplicity. A two-point coordinate system is used to define selections. This process is defined started by clicking the first coordinate. After, a red dot will appear, letting you know where the inital coordinate was placed. Once a second coordinate has been choosen, the red dot will disappear and will leave behind a rectangular border between the two points. The internal design utilises a relative coordinate system, all coordinates are relative to the page, which is not affected by zoom.

| Type | Visual Representation | Description |
| :-: | :-: | :--|
| Initial Coordinate | <img width="30" height="20" alt="image" src="https://github.com/user-attachments/assets/6c641d0f-22c0-47ef-afa0-c16e8563a3aa" /> | A red dot visually representing the first selection coordinate
| Selection | <img width="89" height="78" alt="image" src="https://github.com/user-attachments/assets/caec5700-3879-4a59-a00b-f8598b56dcd2" /> | A rectangle created between the top-right and bottom left coordinates of a selection. A visual 'x' can be seen that allows the user to delete the selection, removing it from the database.
| Document Control | <img width="141" height="90" alt="image" src="https://github.com/user-attachments/assets/e784cce7-e7c7-481b-9dd9-b0f567610886" /> | Document controls apply globally. <br/><br/> Save - Save any selections made by the user.<br/> Extract - Extract any selections<br/> Zoom functionality - The -/+ signs allow the document to be zoomed in and out. |

`(NOTE: THE EXTRACTION IS CURRENTLY LIMITED TO LINE BY LINE EXTRACTION. FUTURE UPDATES WILL FEATURE A WIDER RANGE OF EXTRACTION TECHNIQUES)`
<br />

#### Results Table
When 'extract' is pressed. A table will appear once the extraction has been completed. This table is scrollable.
<img width="958" height="443" alt="image" src="https://github.com/user-attachments/assets/fb6ea214-446e-4f07-af19-061de98a6b1d" />
`(NOTE: FUTURE VERSIONS WILL INCLUDE AN EXPORT TO CSV BUTTON, DRAG-DROP ROW/COLUMN EDITOR/ORGANISER, AS WELL AS A SCRATCH LIKE EVENT-DRIVEN BLOCK SYSTEM)`


### Toast Notifications
Toast notifications are used to inform the user of a successful or failed request, under the hood, they also provide additional functionality such as automatic page or html fragment refreshes. Visual notifications have the following functions:
- Auto-dismiss - The notification will disappear after an interval
- Manual Dismissal - clicking the 'x' will dismiss the notification
- Pause on hover - hovering over a toast notification freezes it in place
- Close icon - A more descriptive term for the 'x'

#### Examples of toast notifications. 
<img width="407" height="170" alt="Screenshot 2026-06-22 221336" src="https://github.com/user-attachments/assets/dc5168d4-9310-4269-82a1-3b1ea580a3c7" />
<img width="407" height="170" alt="Screenshot 2026-06-22 221323" src="https://github.com/user-attachments/assets/f1ac6c05-9bbc-4d76-b105-bca42b85635c" />

<br />

## Project Setup
Each repository will contain README.md instructions on how to configure, run and test each respository.
<br />
## Contributing
Pull requests are welcome. For major changes, please open an issue first
to discuss what you would like to change.

Please make sure to update tests as appropriate.
