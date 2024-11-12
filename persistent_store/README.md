In an ASP.NET MVC application, a persistent store refers to a storage mechanism where data is saved so that it persists across sessions and application restarts. This can include databases (like SQL Server, MySQL, or SQLite), file storage, or cloud-based solutions like Azure Blob Storage. Persistent stores allow data to be retained over time rather than being lost when the application shuts down or a session ends. In ASP.NET, this is typically implemented using Entity Framework or other ORM tools to interact with a relational database.

## Configuration:

To configure a persistent store in an ASP.NET Core MVC app, follow these steps to set up a database using Entity Framework Core with SQL Server:

1. **Install Entity Framework Core Packages**:
   Install the required packages for Entity Framework Core and SQL Server by running the following commands in the NuGet Package Manager Console:

   ```bash
   dotnet add package Microsoft.EntityFrameworkCore.SqlServer
   dotnet add package Microsoft.EntityFrameworkCore.Tools
   ```

2. **Define the Database Context**:
   Create a class that inherits from `DbContext` to represent your database context, which will be responsible for managing your entities and database operations. This class is usually placed in a folder called `Data`.

   ```csharp
   using Microsoft.EntityFrameworkCore;

   public class ApplicationDbContext : DbContext
   {
       public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options)
       {
       }

       // Define DbSets for each entity model you want to store in the database
       public DbSet<Product> Products { get; set; }
       public DbSet<Order> Orders { get; set; }
       // Add other DbSets as needed
   }
   ```

3. **Configure the Database Connection String**:
   In `appsettings.json`, add your database connection string. Replace `YourDatabaseName` and other details with actual values for your SQL Server instance.

   ```json
   {
     "ConnectionStrings": {
       "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=YourDatabaseName;Trusted_Connection=True;MultipleActiveResultSets=true"
     }
   }
   ```

4. **Register the Database Context in Dependency Injection**:
   In the `Startup.cs` (or `Program.cs` for newer .NET versions) file, register your `ApplicationDbContext` with the dependency injection system in the `ConfigureServices` method.

   ```csharp
   using Microsoft.EntityFrameworkCore;

   public void ConfigureServices(IServiceCollection services)
   {
       services.AddControllersWithViews();

       // Register the database context with a connection string
       services.AddDbContext<ApplicationDbContext>(options =>
           options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));
   }
   ```

5. **Create Entity Models**:
   Define entity models representing the tables in your database. For example, a `Product` class would look like this:

   ```csharp
   public class Product
   {
       public int Id { get; set; }
       public string Name { get; set; }
       public decimal Price { get; set; }
   }
   ```

6. **Apply Migrations and Update Database**:
   Run migrations to create the database schema. Use the following commands in the terminal:

   ```bash
   dotnet ef migrations add InitialCreate
   dotnet ef database update
   ```

   - `InitialCreate` is the name of the migration, and you can change it as needed.
   - These commands will create the database and tables based on your defined `DbContext` and entity models.

7. **Use the Database Context in Controllers**:
   Inject `ApplicationDbContext` into your controllers or services to interact with the database.

   ```csharp
   public class ProductsController : Controller
   {
       private readonly ApplicationDbContext _context;

       public ProductsController(ApplicationDbContext context)
       {
           _context = context;
       }

       public IActionResult Index()
       {
           var products = _context.Products.ToList();
           return View(products);
       }
   }
   ```

This configuration provides a basic setup for persistent data storage in an ASP.NET Core MVC application using a SQL Server database.

## Designing Model Classes for a Persistent Store

#### Conventions for Model Class Design
- **Folder Organization**: Add all design model classes to the `Data` folder for organization and clarity.
- **Identifier Naming**: Use `Id` as the name for the unique identifier property, with an `int` data type. Avoid alternative names like `EntityId` to maintain standardization.
- **Data Annotations**:
  - Use `[Required]` and `[StringLength(n)]` for validation where needed.
  - **Scalar Properties** (e.g., `int`, `double`): Do not use the `[Required]` attribute for scalar properties as they are required by default.
- **Associations Between Classes**:
  - Configure **navigation properties** in both classes when defining an association.
  - For **"to-one" navigation properties** (relationships requiring one related entity), include the `[Required]` attribute.
  - For **"to-many" navigation properties**, use `ICollection<T>` type and initialize it with `HashSet<T>`.
- **DateTime Properties**:
  - Initialize `DateTime` properties and any default data in the no-argument constructor to ensure values are set at instantiation.

#### Class Diagrams for Model Verification
- **Create a Class Diagram** after completing design model classes. This diagram helps visually confirm the accuracy and relationships defined in the design model classes.
- **Add DbSet<TEntity> Properties** in the `IdentityModels.cs` file to represent each model class in the database context.

#### Database Creation and Initialization
- **Database Initialization Trigger**:
  - The **database is created** when it is accessed for the first time (read or write operation).
- **Data Context Role**:
  - The **data context class** acts as the gateway to the database.
  - When accessed, the data context calls a **database initializer**, which checks for database existence:
    - **If the database does not exist**: The initializer creates it.
    - **If the database exists**: The initializer detects it and makes no changes.

## Adding new Objects to the Presistent Store:
To add new objects (records) to the persistent store (database) in an ASP.NET Core MVC app, you can use the following approach:

1. **Define the Entity Model**:
   Ensure you have an entity model representing the object you want to store in the database. For example, if you want to add a new product, ensure you have a `Product` model.

   ```csharp
   public class Product
   {
       public int Id { get; set; }
       public string Name { get; set; }
       public decimal Price { get; set; }
   }
   ```

2. **Add a Form to Collect Data**:
   In the controller, create an action method to render a form that collects data from the user. This method will show the form view.

   ```csharp
   public class ProductsController : Controller
   {
       public IActionResult Create()
       {
           return View();
       }
   }
   ```

   Then, create a view file `Create.cshtml` under the `Views/Products` folder with a form that matches the properties of the `Product` model:

   ```html
   @model Product

   <form asp-action="Create" method="post">
       <div>
           <label for="Name">Product Name:</label>
           <input asp-for="Name" />
       </div>
       <div>
           <label for="Price">Price:</label>
           <input asp-for="Price" />
       </div>
       <button type="submit">Add Product</button>
   </form>
   ```

3. **Handle the Form Submission and Add the Object to the Database**:
   Add a `[HttpPost]` version of the `Create` action method to handle form submissions. Inject the `ApplicationDbContext` into the controller to save the new product to the database.

   ```csharp
   public class ProductsController : Controller
   {
       private readonly ApplicationDbContext _context;

       public ProductsController(ApplicationDbContext context)
       {
           _context = context;
       }

       [HttpGet]
       public IActionResult Create()
       {
           return View();
       }

       [HttpPost]
       public IActionResult Create(Product product)
       {
           if (ModelState.IsValid)
           {
               // Add the new product to the DbSet and save changes
               _context.Products.Add(product);
               _context.SaveChanges();

               // Redirect to the Index or another page after adding the product
               return RedirectToAction("Index");
           }

           return View(product);
       }
   }
   ```

4. **Validate and Save the Object**:
   - `_context.Products.Add(product);`: Adds the new product to the `Products` table in the database.
   - `_context.SaveChanges();`: Saves the changes to the database, committing the new product to the persistent store.

5. **Test Adding a New Object**:
   - Run the application.
   - Navigate to `/Products/Create` to open the form.
   - Fill out the form and submit it.
   - The new object will be added to the database, and you should be redirected to the specified action (e.g., `Index`), where you can view the updated list of products.



  ## Migrations

  **1. What Are Migrations?**

Migrations in Entity Framework Core provide a way to incrementally apply changes to your database schema as your application's model evolves. When you modify your entity models (e.g., adding new fields, creating new tables), migrations enable you to update the database schema to match your changes without manually writing SQL scripts.

**2. Configuring Migrations in ASP.NET Core MVC**

To set up migrations, follow these steps:

- **Ensure Entity Framework Core is Installed**:
   Make sure `Microsoft.EntityFrameworkCore.Tools` is installed in your project.

   ```bash
   dotnet add package Microsoft.EntityFrameworkCore.Tools
   ```

- **Initialize the First Migration**:
   Use the following command in the terminal to create an initial migration. This will generate migration files in a `Migrations` folder (or any folder name you choose).

   ```bash
   dotnet ef migrations add InitialCreate
   ```

   - `InitialCreate` is the name of this migration and can be any descriptive name.
   - This command creates a migration file containing the SQL commands to set up the database based on your current entity model.

- **Apply the Migration to the Database**:
   After creating a migration, apply it to the database by running:

   ```bash
   dotnet ef database update
   ```

   - This command updates the database schema to match the model defined in the migration.

**3. Do I Need to Migrate Data for Every CRUD Operation?**

No, you don’t need to run migrations for each CRUD operation. Migrations are specifically for changes in the database schema, not for regular data operations. Here’s when you do or don’t need migrations:

- **When Migrations Are Needed**:
  - Adding, modifying, or removing properties from an entity model (e.g., adding a new field).
  - Creating or deleting entire entity models (e.g., creating a new table or removing an existing one).
  - Changing relationships between entities (e.g., setting up a foreign key relationship).

- **When Migrations Are NOT Needed**:
  - Performing CRUD operations on existing tables (e.g., adding, reading, updating, or deleting rows).
  - These actions work with the existing schema, so no schema changes (or migrations) are required.

**4. How to Use Migrations When the Model Changes**

If you make changes to your model and need to update the database schema, follow these steps:

1. **Create a New Migration**:
   Each time you update the model (like adding or removing properties), create a new migration:

   ```bash
   dotnet ef migrations add NameOfChange
   ```

   - Replace `NameOfChange` with a description, like `AddDescriptionToProduct`.

2. **Apply the New Migration**:
   After creating the migration, apply it to the database using:

   ```bash
   dotnet ef database update
   ```

**5. Example Workflow**

Imagine you have a `Product` model with `Id` and `Name` properties. Later, you add a `Price` property to `Product`. To update the database schema:

- Add the `Price` property to the `Product` model in code.
- Run:

   ```bash
   dotnet ef migrations add AddPriceToProduct
   ```

- Apply it:

   ```bash
   dotnet ef database update
   ```

This approach keeps the database schema and your application model in sync without needing migrations for each CRUD operation.

