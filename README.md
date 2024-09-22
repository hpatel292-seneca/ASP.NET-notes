# ASP.NET-notes
Here are notes based on the key topics from **Lecture 2.1 - Persisting Data** for your revision:

---

### **Persisting Data (Week 2 Topics)**

1. **Textbook Usage**: The course emphasizes using view model classes over directly interacting with persistent storage (database) through controllers, unlike the textbook examples.

2. **Project Template**: You will use a pre-configured database in upcoming weeks to build interactive web apps. Later, you'll learn to create your own data store.

3. **System Design and Architecture**: The course follows a layered architecture:
   - **Views** interact with **controllers**.
   - **Controllers** use **view model classes** for data exchange.
   - **Manager class** handles business logic and data service operations.
   - **Data store** (local database) is accessed indirectly using Entity Framework (EF).

---

### **Persistent Data Storage**

1. **App Data**: Relational databases are used, stored in the `App_Data` folder. The connection is managed via the `Web.config` file using a connection string.

2. **Local vs. Host-Based Databases**: 
   - During development, a local instance of SQL Server (within Visual Studio) is used.
   - When deploying, a host-based SQL Server can be configured.

---

### **Entity Framework (EF)**

1. **Object-Relational Mapper (ORM)**:
   - **Entity Framework** is used to map C# objects to database tables, simplifying database interactions.
   - The **Data Context** serves as the gateway to the data store and supports common operations (querying, adding, updating, deleting).

2. **Design Model Classes**:
   - These classes represent the entities in the database (e.g., tables), where each property maps to a column in the table.

---

### **Manager Class**

1. **Role**: 
   - The **Manager class** centralizes app and business logic. Controllers never directly interact with the database; they call Manager methods.
   - Methods in the Manager class can handle tasks like fetching data, adding records, etc.

2. **Layered Architecture**: 
   - Manager classes promote a safer and more structured coding style by isolating database operations from controllers and views.
   - Data passed between Manager methods and controllers is handled by **view model classes**.

---

### **View Model Classes**

1. **Purpose**:
   - View models are tailored for specific use cases and simplify interactions between the UI and the underlying data model.
   - They are customized to fit exactly what is needed for display, form population, or form submission.

2. **Advantages**:
   - Increased flexibility in how data is presented and processed.
   - Enhances security by preventing direct interaction between user data and the data model.

3. **Typical Use Cases**:
   - **Add**: To create new entities.
   - **Base**: For retrieving and displaying data.
   - **Edit**: To update existing records.

---

### **AutoMapper**

1. **Role**: 
   - AutoMapper helps map between **design model classes** (data from the database) and **view model classes**.
   - It works based on conventions, mapping properties with matching names and types automatically.

2. **Usage**:
   - In this course, you are required to use the **instance-based API** of AutoMapper, as using the static API will result in grading penalties.

---



### **Domain Models:**
- **Definition:** Domain models represent the core entities or objects in your application's business logic. These models directly map to the concepts or things in the problem domain (the real world or the business you are working with).
- **Purpose:** Domain models are used to interact with the database and manage business rules. They often mirror the structure of your database tables in applications using ORM (Object-Relational Mapping) tools like Entity Framework. 
- **Example:**
  - If you are building an e-commerce application, domain models could include `Product`, `Customer`, `Order`, etc.
  - For instance, a `Customer` domain model might look like:
    ```csharp
    public class Customer
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public string Email { get; set; }
        public DateTime DateOfBirth { get; set; }
        public ICollection<Order> Orders { get; set; }
    }
    ```

### **View Models:**
- **Definition:** View models are data transfer objects (DTOs) designed specifically for the presentation layer (the user interface). They contain only the data that is needed for the UI, making them lighter and more focused compared to domain models.
- **Purpose:** View models simplify the data that is sent to the front end (or received from the front end). They decouple the domain/business models from the user interface, ensuring that sensitive data (e.g., passwords) or unnecessary data (e.g., internal system information) is not exposed.
- **Example:**
  - In the same e-commerce application, a view model could represent only the necessary data for displaying customer information on the UI. For instance, a `CustomerBaseViewModel` might only include basic information:
    ```csharp
    public class CustomerBaseViewModel
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public string Email { get; set; }
    }
    ```

### **Differences between Domain Models and View Models:**
1. **Purpose:**
   - **Domain Models:** Represent the structure of your business or database entities. They often include relationships between objects (e.g., a customer having multiple orders).
   - **View Models:** Simplified objects used for displaying data in the user interface. They may omit fields that are unnecessary for the UI or sensitive to business logic.
   
2. **Usage:**
   - **Domain Models:** Used internally in your application's business logic or to interact with a database.
   - **View Models:** Used to transfer data between the controller and the view/UI. They are mapped from domain models but serve a presentation-specific purpose.

3. **Structure:**
   - **Domain Models:** Can include all properties related to the business entity, including relationships (e.g., orders for a customer).
   - **View Models:** Typically contain only the information required by the view (e.g., a customer’s name and email but not their orders).


Yes, you're exactly right! The purpose of **AutoMapper** is to simplify the process of converting (or **mapping**) between **domain models** and **view models** (or other data transfer objects, DTOs). AutoMapper automates this process, avoiding the need to manually copy properties between these objects. Let me explain how this works step-by-step:

### **Purpose of AutoMapper:**
AutoMapper is a library that helps you:
1. **Map domain models to view models** (and vice versa).
2. **Automatically handle data transformation** between objects that have similar or related structures.
3. **Reduce boilerplate code** that you would otherwise write manually to transfer data from one model to another.

### **Typical Workflow with AutoMapper:**
Here’s how AutoMapper fits into the flow:

1. **Domain Model and Data Fetching**:
   - First, you fetch data from the database using a **domain model**. The domain model often has all the data, including relationships, detailed properties, and sensitive information that are part of the business logic.
   - Example:
     ```csharp
     public class Customer
     {
         public int Id { get; set; }
         public string Name { get; set; }
         public string Email { get; set; }
         public string PasswordHash { get; set; }  // Sensitive data
         public ICollection<Order> Orders { get; set; }  // Relationships
     }
     ```

2. **Mapping Domain Model to View Model**:
   - After fetching the **domain model**, you map it to a **view model** using AutoMapper.
   - The view model contains **only the data needed for the UI**, so you strip out unnecessary or sensitive fields (e.g., `PasswordHash`).
   - AutoMapper automates the mapping of fields with the same or similar names between the domain model and the view model.
   - Example of View Model:
     ```csharp
     public class CustomerBaseViewModel
     {
         public int Id { get; set; }
         public string Name { get; set; }
         public string Email { get; set; }  // Only what UI needs
     }
     ```

   - AutoMapper setup:
     ```csharp
     var config = new MapperConfiguration(cfg =>
     {
         cfg.CreateMap<Customer, CustomerBaseViewModel>();
     });
     var mapper = config.CreateMapper();
     ```

   - Using AutoMapper to map the domain model to the view model:
     ```csharp
     var customerViewModels = mapper.Map<IEnumerable<Customer>, IEnumerable<CustomerBaseViewModel>>(customers);
     ```

3. **Displaying the View Model in the UI**:
   - The view model is sent to the UI, and it **only contains the data that the view needs** (e.g., customer name, email). This ensures that sensitive or irrelevant data is not exposed to the front end.

4. **Receiving Input (Reverse Mapping)**:
   - When the user submits a form (e.g., adding or editing a customer), the input data is often sent as a **view model** (such as `CustomerAddViewModel`).
   - You then use AutoMapper to map the **view model back to the domain model**.
   - Example:
     ```csharp
     var newCustomer = mapper.Map<CustomerAddViewModel, Customer>(customerAddViewModel);
     ```

   - You can then use the `newCustomer` object to save it in the database.

### **Key Benefits of AutoMapper:**
1. **Reduces Boilerplate Code**: Without AutoMapper, you'd have to manually map each property from the domain model to the view model and vice versa. AutoMapper handles this for you, making the code cleaner and easier to maintain.
   - Example of manual mapping without AutoMapper:
     ```csharp
     var customerViewModel = new CustomerBaseViewModel
     {
         Id = customer.Id,
         Name = customer.Name,
         Email = customer.Email
     };
     ```

2. **Consistent and Maintainable**: AutoMapper ensures that mappings are handled consistently across your application. If your model changes (e.g., you add or remove properties), you only need to update the mappings in one place.

3. **Separation of Concerns**: The domain model remains focused on the business logic and data storage, while the view model is optimized for displaying data in the UI. AutoMapper bridges these two models seamlessly.

### **Summary:**
- **Domain Models** represent all the data fetched from the database, including sensitive and detailed information.
- **View Models** contain only the necessary information required for the UI, making the data lighter and more secure.
- **AutoMapper** automatically maps domain models to view models and vice versa, saving you from writing repetitive code and ensuring that only the required data is sent to the front end.

In short, AutoMapper helps you **extract only the needed data** from your domain model and pass it to the view in a clean and efficient way.

## Customer Index View
This `Customer/Index.cshtml` code is the Razor view for displaying a list of customers in a table format. Here’s a breakdown of the key components:

### **Model Declaration**

```csharp
@model IEnumerable<MyFirstWebApp.Models.CustomerBaseViewModel>
```
- This line declares the type of the model the view will work with. In this case, the view expects a collection of `CustomerBaseViewModel` objects (which is likely passed from the controller).

### **HTML and Razor Syntax**

```csharp
<h2>Index</h2>

<p>
    @Html.ActionLink("Create New", "Create")
</p>
```
- Displays an "Index" header.
- The `@Html.ActionLink("Create New", "Create")` generates a hyperlink that points to the `Create` action method in the controller. It allows users to navigate to a page for creating a new customer.

### **Table Setup**

```csharp
<table class="table">
    <tr>
        <th>
            @Html.DisplayNameFor(model => model.FirstName)
        </th>
        <!-- More columns here -->
    </tr>
```
- A table is used to display customer information.
- `@Html.DisplayNameFor(...)` renders the display name for each property (like `FirstName`, `LastName`, etc.), based on what is defined in the `CustomerBaseViewModel` class.

### **Rendering Customer Data**

```csharp
@foreach (var item in Model)
{
    <tr>
        <td>
            @Html.DisplayFor(modelItem => item.FirstName)
        </td>
        <td>
            @Html.DisplayFor(modelItem => item.LastName)
        </td>
        <!-- More columns here -->
        <td>
            @Html.ActionLink("Edit", "Edit", new { id=item.CustomerId }) |
            @Html.ActionLink("Details", "Details", new { id=item.CustomerId }) |
            @Html.ActionLink("Delete", "Delete", new { id=item.CustomerId })
        </td>
    </tr>
}
```
- `@foreach (var item in Model)` iterates over the collection of customers (`Model`) passed to the view.
- Inside the loop, each customer (`item`) is displayed row by row.
- `@Html.DisplayFor(...)` renders the value of each property for the current customer (`item`), like `FirstName`, `LastName`, etc.
  
### **Action Links**
```csharp
@Html.ActionLink("Edit", "Edit", new { id=item.CustomerId }) |
@Html.ActionLink("Details", "Details", new { id=item.CustomerId }) |
@Html.ActionLink("Delete", "Delete", new { id=item.CustomerId })
```
- These are links that let the user interact with each customer:
    - **Edit**: Takes the user to the `Edit` action method for the specific customer, identified by `CustomerId`.
    - **Details**: Takes the user to the `Details` action method to view details for that customer.
    - **Delete**: Takes the user to the `Delete` action method to delete the customer.

Each of these action links passes the `CustomerId` as a route parameter to the corresponding action method in the controller.

### **Summary**
- This Razor view:
  - Displays a table with customer information.
  - Uses `Html.DisplayNameFor` to display the column names.
  - Uses `Html.DisplayFor` to show the values for each customer in the list.
  - Provides "Edit", "Details", and "Delete" actions for each customer.

Let me know if you'd like any further clarification!

In an ASP.NET project, using **Scaffolding** is a way to automatically generate code for common CRUD (Create, Read, Update, Delete) operations, views, and controllers based on the data models you have defined in your project. It helps developers save time by creating the necessary files and structures instead of writing everything manually.

Specifically, when you **scaffold a view** in an ASP.NET project, the Scaffolding tool will:

1. **Analyze the model**: It looks at your data model (e.g., a class that represents a table in a database).
2. **Generate HTML and Razor code**: It automatically creates a `.cshtml` file, which includes the necessary Razor syntax and HTML to display or manage the model’s data in the view.
3. **Generate CRUD views**: If you're using Entity Framework, scaffolding can generate views like:
   - **Create.cshtml**: For creating new records.
   - **Edit.cshtml**: For updating records.
   - **Delete.cshtml**: For deleting records.
   - **Details.cshtml**: For viewing individual record details.
   - **Index.cshtml**: For listing records.

### Steps to scaffold a view in ASP.NET:
1. Right-click on the **Controllers** folder or the **Views** folder.
2. Select **Add > New Scaffolded Item**.
3. Choose the type of scaffold, such as **MVC View Page**, **Controller with Views using Entity Framework**, etc.
4. Select the model and data context to generate the view based on that model.

Scaffolding speeds up the development process and ensures consistent code structure across your project.
