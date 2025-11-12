# Library Manager (RedTag Bootcamp Project)

This is a demo Salesforce application, "Library Manager," created as a technical assignment. The project demonstrates proficiency in modern Salesforce development, including Lightning Web Components (LWC) for the user interface and Apex for backend business logic.

A key feature of this project is a **custom authentication system**. Instead of using standard Salesforce login, users (e.g., library staff) authenticate via a custom LWC. This component verifies their credentials against a custom `User_Credentials__c` object, which stores securely hashed and salted passwords.

## Table of Contents

1.  Data Model (Schema)
2.  Component Breakdown
3.  Installation Instructions   // If you choose not to complete the full installation, please refer to the video demo in point 5
4.  How to Use (Testing Steps)
5.  Video Project Demo



---

## Data Model (Schema)
<img width="3816" height="1832" alt="objectSchema" src="https://github.com/user-attachments/assets/848238be-47aa-495d-9905-06de41c742fa" />


The application is built on three custom objects.

* **Author (`Author__c`)**
    * Stores information about the authors.
    * `Author Name` (Name) - *Text (80)*
    * `Date of Birth` (Date_of_Birth__c) - *Date*
    * `Pseudonyms` (Pseudonyms__c) - *Text (255)*

* **Book (`Book__c`)**
    * Stores information about the books.
    * `Book Name` (Name) - *Text (80)*
    * `Author` (Author__c) - *Lookup(Author)*
    * `Book Genre` (Book_Genre__c) - *Picklist*

* **User Credentials (`User_Credentials__c`)**
    * Stores the private credentials for the custom login system.
    * `Username` (Username__c) - *Text (255) (Unique, Case-Sensitive)*
    * `Display Name` (Display_Name__c) - *Text (255)*
    * `Password Hash` (Password_Hash__c) - *Text (255)*
    * `Password Salt` (Password_Salt__c) - *Text (255)*

### Design Rationale: Lookup vs. Master-Detail

The relationship between `Author__c` and `Book__c` is intentionally a **Lookup** relationship, not Master-Detail.

While a Master-Detail relationship would automatically cascade-delete child `Book__c` records if an `Author__c` is deleted, I chose a Lookup. This was done specifically to demonstrate the ability to manage this deletion logic manually in Apex, rather than relying on standard Salesforce features, as a way to showcase backend development skills.


---

## Component Breakdown

The project clearly separates client-side and server-side responsibilities.

### Backend (Apex)

1.  **`UserCredentialManager.cls`**
    * **Role:** The main service class for all authentication logic.
    * **Methods:**
        * `login(username, password)`: Called from LWC. It finds the `User_Credentials__c` record by username and validates the provided password by computing a new hash with the stored salt and comparing it using `Crypto.verifyMac()`.
        * `register(username, password, displayName)`: Creates a new `User_Credentials__c` record. It securely prepares the password by generating a new salt (`Crypto.generateAesKey`) and a new hash (`Crypto.generateMac`) before saving.
2.  **`LibraryController.cls`**
    * **Role:** Data controller for the library components.
    * **Methods:**
        * `getBooks()`: An `@AuraEnabled(cacheable=true)` method that returns a list of `Book__c` records, including the related Author's name (`Author__r.Name`). It correctly uses `WITH SECURITY_ENFORCED` to respect Field-Level Security (FLS).
3.  **`LoginResultDTO.cls`**
    * **Role:** A DTO (Data Transfer Object) class. It provides a structured wrapper (`isSuccess`, `message`) for sending clean, predictable responses from Apex back to LWC.

### Frontend (LWC)

1.  **`loginShell`**
    * **Role:** The main parent component and state manager. It controls whether to show the login/register forms or the main app content.
    * **Logic:** Uses `if:true={isAuthenticated}` in its HTML. If `isAuthenticated` is `false`, it displays a `lightning-tabset` with the "Login" and "Register" forms. When a successful login event is received, it sets `isAuthenticated=true` and renders the `c-library-manager` component instead.
2.  **`libraryManager`**
    * **Role:** A simple container component that is rendered after a successful login.
    * **Logic:** It wraps and displays the `c-book-list` component.
3.  **`bookList`**
    * **Role:** Displays the list of books.
    * **Logic:** Uses the `@wire(getBooks)` adapter to call the `LibraryController.getBooks()` Apex method. The returned data is then displayed in a `lightning-datatable`.

---

## Installation Instructions

To deploy and run this project, follow these steps.

### 1. Deploy Source Code

1.  Clone or download the repository.
2.  Authenticate with your target Sandbox or Developer Org using SFDX CLI.
3.  Deploy the source code:
### 2. Assign Permissions

1. Make sure that after deployment user has access to new objects and fields.

### 3. Configure the UI

The `loginShell` component must be placed on the app's home page.

1.  In **Setup**, navigate to **Lightning App Builder**.
2.  Find and **Edit** the `Library_Home_Page`.
3.  Drag the **`loginShell`** custom LWC from the component panel onto the main page canvas.
4.  **Save** the page.
5.  Click **Activate**, select **App Default**, and assign it as the App Default page for the **`Library_Manager`** app.

---

## How to Use (Testing Steps)

1.  From the **App Launcher**, open the **`Library_Manager`** app.
2.  You will see the custom login/register component.
3.  **Register:**
    * Click the **Register** tab.
    * Enter a `Display Name`, `Username`, and `Password` (e.g., `testuser` / `testpass123`).
    * Click "Register". This will create the `User_Credentials__c` record.
4.  **Login:**
    * Click the **Login** tab.
    * Enter the `Username` and `Password` you just created.
    * Click "Login".
5.  **Result:**
    * The login component will disappear and be replaced by the `libraryManager` component, which displays the `bookList` datatable.

*Note: If the book list is empty, you may need to manually create a few `Author__c` and `Book__c` records via 'Add Book' and 'Add Author' buttons

---

## Project Demo

I also prepared demo video of how my app works

Video recorded with Loom application

https://www.loom.com/share/0e4a09d8de7a43cb8f510c4270b02679

---
