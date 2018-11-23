# Starting with .NET Core in Linux TDD style

[.NET Core](https://www.microsoft.com/net) is an open source, multi-platform, and multi-purpose runtime for .NET applications developed by [Microsoft](https://www.microsoft.com).

With .NET Core you can build console and web applications using a variety of languages, like [C#](https://en.wikipedia.org/wiki/C_Sharp_(programming_language)) and [F#](https://en.wikipedia.org/wiki/F_Sharp_(programming_language)) and run them in Windows, Mac, Linux or Docker.

## What will we be building?

The obvious answer when you want to build a demo application is **a todo list**.

## What do we need to start?

Before starting there are certain tools you need to install.

## Setting up the workspace

Install the [.NET Core SDK](https://www.microsoft.com/net/download), we will be using version 2.1.

Download and install [Visual Studio Code](https://code.visualstudio.com/Download).

Install the [C# extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp). If you want to set up a better workspace for C# then you can follow [this article](https://code.visualstudio.com/Docs/languages/csharp).

Now we are prepared to start.

## Let's start!

Assuming you followed the steps listed in the `Setting up the workspace` section we can start building our application.

## Setting up the projects

Create a new ASP .NET MVC project by running the following commands:

```
mkdir Todolist
cd Todolist
dotnet new mvc --name=Todolist
```

This will create a new project called `Todolist` inside the current directory.

Now we have to create a [xUnit](https://xunit.github.io/) project. We do that by running:

```
dotnet new xunit --name=Todolist.Tests
```

This will create a new `xUnit` project inside the `Todolist.Tests` directory, it will generate a first test called `UnitTest1.cs`, you can delete it.

Now we have to add a reference to our ASP project into our xUnit project. We do that by running the following commands.

```
cd Todolist.Tests
dotnet add reference ../Todolist/Todolist.csproj
```

After that we have to add some dependencies for the task of testing:

```
dotnet add package Microsoft.AspNetCore.App --version 2.1.6
dotnet add package Microsoft.AspNetCore.Mvc.Testing --version 2.1.3
```

And specify that we will be using the web SDK in our `Todolist.Tests.csproj`

```
Before
<Project Sdk="Microsoft.NET.Sdk">

After
<Project Sdk="Microsoft.NET.Sdk.Web">
```

Now we have two projects inside the `Todolist` directory, an ASP MVC project, and a xUnit project, we have to establish a relation between the two projects, we do that by using a `solution`, first of all, let's create a solution:

```
cd ../
dotnet new sln
```

Now let's add our projects to the solution file by running:

```
dotnet sln add ./Todolist/Todolist.csproj
dotnet sln add ./Todolist.Tests/Todolist.Tests.csproj
```

Now we can start the development server and see your project working. This template includes some views built with [Bootstrap 3](https://getbootstrap.com/docs/3.3/), so the project is not empty by default. Let's run the following commands:

```
dotnet run
```

Our application should start working in `http://localhost:5000` and `https://localhost:5001`, we will be using the HTTPS route because configuring the SSL certificate for Linux will depend on the distribution you are using.

If you visit `http://localhost:5000` you can see the application working, start browsing.

## Writing our first test

As we are following the [TDD](https://en.wikipedia.org/wiki/Test-driven_development) approach the first thing we need to do is define our tests, then write our implementation, for simplicity I will write tests for all the functionalities, and then I will write the actual implementations, in a real scenario you should write your tests one at a time.

Let's create a new file called `Controllers/TodoControllerTest` inside our tests project, it will look like this:

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using Todolist.Controllers;
using Todolist.Models;
using Xunit;

namespace Todolist.Tests.Controllers
{
    public class TodoControllerTest : IDisposable
    {
        private TodoController controller;

        private TodoContext todoContext;

        public TodoControllerTest()
        {
            var optionsBuilder = new DbContextOptionsBuilder<TodoContext>();

            optionsBuilder.UseInMemoryDatabase("test");

            todoContext = new TodoContext(optionsBuilder.Options);

            todoContext.Todos.Add(new Todo { Id = 1, Title = "Go to the supermarket" });
            todoContext.SaveChanges();

            controller = new TodoController(todoContext);
        }

        public void Dispose()
        {
            var todo = todoContext.Todos.SingleOrDefault(currentTodo => currentTodo.Id == 1);

            if (todo == null) {
                return;
            }

            todoContext.Todos.Remove(todo);
            todoContext.SaveChanges();
        }

        [Fact]
        public void Index_ReturnsViewResult_WithAListOfTodos_WhenSucceeded()
        {
            var result = controller.Index();
            var viewResult = Assert.IsType<ViewResult>(result);

            Assert.IsAssignableFrom<List<Todo>>(viewResult.Model);
        }

        [Fact]
        public void Create_ReturnsViewResult_WhenSucceeded()
        {
            var result = controller.Create();
            var viewResult = Assert.IsType<ViewResult>(result);
        }

        [Fact]
        public void PostCreate_RedirectsToIndex_WhenSucceeded()
        {
            var result = controller.Create(new Todo { Id = 2, Title = "Go to the supermarket" });
            var redirectToActionResult = Assert.IsType<RedirectToActionResult>(result);

            Assert.Equal("Index", redirectToActionResult.ActionName);
        }

        [Fact]
        public void Edit_ReturnsViewResult_WithATodo_WhenSucceeded()
        {
            var result = controller.Edit(1);
            var viewResult = Assert.IsType<ViewResult>(result);

            Assert.IsAssignableFrom<Todo>(viewResult.Model);
        }

        [Fact]
        public void PostEdit_RedirectsToIndex_WhenSucceeded()
        {
            var result = controller.Edit(new Todo { Id = 1, Title = "Buy Gas" });
            var redirectToActionResult = Assert.IsType<RedirectToActionResult>(result);

            Assert.Equal("Index", redirectToActionResult.ActionName);
        }

        [Fact]
        public void Show_ReturnsViewResult_WithATodo_WhenSucceeded()
        {
            var result = controller.Show(1);
            var viewResult = Assert.IsType<ViewResult>(result);

            Assert.IsAssignableFrom<Todo>(viewResult.Model);
        }

        [Fact]
        public void Delete_RedirectsToIndex_WhenSucceeded()
        {
            var result = controller.Delete(1);
            var redirectToActionResult = Assert.IsType<RedirectToActionResult>(result);

            Assert.Equal("Index", redirectToActionResult.ActionName);
        }
    }
}
```

Here we are defining a test class for our `TodoController`, which in this case will be our single controller in the whole application.

First of all, we defined two private properties, `controller` and `todoContext`, these objects will help us during our tests.

Then we put our object in a valid state by initializing our properties in the constructor, we create new context pointing to an in-memory database, insert a new register, and then inject the context to our `TodoController`. In the `Dispose` method we delete the dummy register in our in-memory database after running each test, that way we make sure we start every test with fresh data; the `if` clause is for avoiding errors after testing the `Delete` method, which deletes our only register.

The rest of the methods are simple tests for each one of the controller methods.

If we run this test class we will get nothing but errors, which is good, we know how our code should behave, the only thing we need to do now is writing it.

## Setting up the database

In our `Todolist` project, inside our `Models` directory we will create two classes `TodoContext` and our model,  `Todo`.

```c#
using Microsoft.EntityFrameworkCore;

namespace Todolist.Models
{
    public class TodoContext : DbContext
    {
        public TodoContext(DbContextOptions<TodoContext> options)
            : base(options)
        {
        }

        public DbSet<Todo> Todos { get; set; }
    }
}
```

```c#
namespace Todolist.Models
{
    public class Todo
    {
        public int Id { get; set; }
        public string Title { get; set; }
    }
}
```

We just defined how we will be interacting with our database, now we have to inject our context in the service container to be resolved automatically. We will be using an SQLite database, but first, we have to install a package in our `Todolist` project, because SQLite is not supported by Entity Framework out of the box. We do so by running:

```
dotnet add Todolist package Microsoft.EntityFrameworkCore.Sqlite --version 2.1.4
```

Now we have to go to the `Startup.cs` file in the root of our `Todolist` project and modify the `ConfigureServices` methods by adding the following lines:

```
services.AddDbContext<TodoContext>(options =>
                options.UseSqlite("Data Source=todos.db"));
```

**Note**: Don't forget to add the `using Todolist.Models;` and `using Microsoft.EntityFrameworkCore;` statements at the top of the file!

Perfect, now our application knows we want to use an SQLite database called `todos.db` which will be in the root of our `Todolist` project.

## Running the first migration

After configuring our application to work with SQLite we have to create a migration with our table(s) schema, luckily you don't have to write any code for this, just use the following commands:

```
cd Todolist
dotnet ef migrations add InitialCreate
dotnet ef database update
```

## Writing the business logic

Now inside the `Controllers` directory, we will define the `TodoController` class.

```c#
using System.Linq;
using Microsoft.AspNetCore.Mvc;
using Todolist.Models;

namespace Todolist.Controllers
{
    public class TodoController : Controller
    {
        private TodoContext context;

        public TodoController(TodoContext context)
        {
            this.context = context;
        }

        public IActionResult Index()
        {
            return View(context.Todos.ToList());
        }

        public IActionResult Create()
        {
            return View();
        }

        [HttpPost]
        public IActionResult Create(Todo todo)
        {
            context.Todos.Add(todo);
            context.SaveChanges();

            return RedirectToAction("Index");
        }

        public IActionResult Edit(int id)
        {
            Todo todo = context.Todos.Find(id);

            return View(todo);
        }

        [HttpPost]
        public IActionResult Edit(Todo editedTodo)
        {
            var todo = context.Todos.SingleOrDefault(currentTodo => currentTodo.Id == editedTodo.Id);
            todo.Title = editedTodo.Title;

            context.SaveChanges();

            return RedirectToAction("Index");

        }

        public IActionResult Show(int id)
        {
            Todo todo = context.Todos.Find(id);

            return View(todo);
        }

        public IActionResult Delete(int id)
        {
            context.Todos.Remove(context.Todos.Single(todo => todo.Id == id));
            context.SaveChanges();

            return RedirectToAction("Index");
        }
    }
}
```

## Writing the views

Now we have all the behavior on our application, the last thing to do is define our views, create a new directory called `Todo` inside `Views`, and put the following files:

```Index.cshtml```

```html
@model List<Todolist.Models.Todo>

@{
    ViewData["Title"] = "Index";
}

<h2>Todos</h2>

<table class="table">
    <thead>
        <tr>
            <th>
                Title
            </th>
            <th></th>
        </tr>
    </thead>
    <tbody>
        @foreach (var todo in Model)
        {
            <tr>
                <td>
                    @todo.Title
                </td>
                <td>
                    <a asp-action="Edit" asp-route-id="@todo.Id" class="btn btn-default">Edit</a>
                    <a asp-action="Show" asp-route-id="@todo.Id" class="btn btn-info">Show</a>
                    <a asp-action="Delete" asp-route-id="@todo.Id" class="btn btn-danger">Delete</a>
                </td>
            </tr>
        }
    </tbody>
</table>

<p>
    <a class="btn btn-success" asp-action="Create">Add Todo</a>
</p>
```

```Edit.cshtml```

```html
@model Todolist.Models.Todo

@{
    ViewData["Title"] = "Edit";
}

<h4>Edit @Model.Id</h4>
<hr />
<div class="row">
    <div class="col-md-4">
        <form asp-action="Edit">
            <div asp-validation-summary="ModelOnly" class="text-danger"></div>
            <input type="hidden" asp-for="Id" />
            <div class="form-group">
                <label asp-for="Title" class="control-label"></label>
                <input asp-for="Title" class="form-control" />
                <span asp-validation-for="Title" class="text-danger"></span>
            </div>
            <div class="form-group">
                <input type="submit" value="Save" class="btn btn-default" />
            </div>
        </form>
    </div>
</div>

<div>
    <a asp-action="Index">Go back</a>
</div>
```

```Create.cshtml```

```html
@model Todolist.Models.Todo

@{
    ViewData["Title"] = "Create";
}

<h2>Create</h2>

<h4>Todo</h4>
<hr />
<div class="row">
    <div class="col-md-4">
        <form asp-action="Create">
            <div asp-validation-summary="ModelOnly" class="text-danger"></div>
            <div class="form-group">
                <label asp-for="Title" class="control-label"></label>
                <input asp-for="Title" class="form-control" />
                <span asp-validation-for="Title" class="text-danger"></span>
            </div>
            <div class="form-group">
                <input type="submit" value="Create" class="btn btn-default" />
            </div>
        </form>
    </div>
</div>

<div>
    <a asp-action="Index">Go back</a>
</div>
```

```Show.cshtml```

```html
@model Todolist.Models.Todo

@{
    ViewData["Title"] = "Show";
}

<div>
    <h4>Todo</h4>
    <hr />
    <dl class="dl-horizontal">
        <dt>
            @Html.DisplayNameFor(model => model.Title)
        </dt>
        <dd>
            @Html.DisplayFor(model => model.Title)
        </dd>
    </dl>
</div>
<div>
    <a asp-action="Edit" asp-route-id="@Model.Id">Edit</a> |
    <a asp-action="Index">Go back</a>
</div>
```

Now our application is finished! If you run the command `dotnet run` inside the `Todolist` project, or `dotnet run --project Todolist` in the main directory (the one that contains both the application, and the testing project) everything should be working properly. Pretty good, ah?

**Note**: Remember, you have to navigate manually to `http://localhost:5000/Todo` or `https://localhost:5001/Todo` because we didn't add a link in the navbar.

You can find the project on [Github](https://github.com/maxalmonte14/simple-todolist), feel free to fork it and improve, use it to improve your skills on TDD and .NET.

### Post-credits

Here we built a very simple application using a TDD approach, we are just testing the [happy path](https://en.wikipedia.org/wiki/Happy_path) of our application, we are not catching errors or testing undesirable behavior, what would happen if we pass a number higher than the maximum integer allowed by the language to the `Edit` method in our `TodoController`? Those are things to take into account when you are following the TDD approach.

We did not use tools to automate tasks like [Scaffolding](https://github.com/aspnet/scaffolding), the main goal here was to illustrate how to do it all from scratch. I could cover those other topics in future posts.

Also this application lacks security in many ways, we are not doing any kind of validation, neither in our models nor in the frontend, for example, **a user could create an empty todo and the system wouldn't complain!**, remember that **this is not production code** and this application exists with the sole purpose of making this tutorial fully operational.

Also, we didn't use asynchronous methods in our controller (spoiler alert: in a real application you want to do it) because testing asynchronous methods is a little more complicated.

I hope you have found this article useful. Thanks for staying this long!