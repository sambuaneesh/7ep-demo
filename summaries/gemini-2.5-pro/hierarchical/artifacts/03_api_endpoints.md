| Component Name | HTTP Method | Endpoint Path | Brief Description |
| :--- | :--- | :--- | :--- |
| System | GET | `/` | Displays the application's welcome page. |
| System | GET | `/oups` | Intentionally triggers a server error to test error handling. |
| Owner Management | GET | `/owners/find` | Displays the form to search for pet owners. |
| Owner Management | GET | `/owners` | Processes owner search requests and displays results. |
| Owner Management | GET | `/owners/new` | Displays the form for registering a new pet owner. |
| Owner Management | POST | `/owners/new` | Processes the submission for a new pet owner. |
| Owner Management | GET | `/owners/{ownerId}` | Displays the detailed information for a specific pet owner. |
| Owner Management | GET | `/owners/{ownerId}/edit` | Displays the form for updating an existing pet owner's details. |
| Owner Management | POST | `/owners/{ownerId}/edit` | Processes the submission for updating a pet owner's details. |
| Owner Management | GET | `/owners/{ownerId}/pets/new` | Displays the form for adding a new pet to an owner. |
| Owner Management | POST | `/owners/{ownerId}/pets/new` | Processes the submission for adding a new pet. |
| Owner Management | GET | `/owners/{ownerId}/pets/{petId}/edit` | Displays the form for updating an existing pet's details. |
| Owner Management | POST | `/owners/{ownerId}/pets/{petId}/edit` | Processes the submission for updating a pet's details. |
| Owner Management | GET | `/owners/{ownerId}/pets/{petId}/visits/new` | Displays the form for adding a new medical visit for a pet. |
| Owner Management | POST | `/owners/{ownerId}/pets/{petId}/visits/new` | Processes the submission for a new medical visit. |
| Vet Management | GET | `/vets` | Displays a paginated list of all veterinarians for the web UI. |
| Vet Management | GET | `/vets` (API) | Returns a list of all veterinarians and their specialties (JSON/XML). |