# MVC Super-Duper Basics

Using (but not `using`) these tutorials: 
- https://docs.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app/?view=aspnetcore-5.0
- https://docs.microsoft.com/en-us/aspnet/core/data/ef-mvc/?view=aspnetcore-5.0

Anything in (()) should be named by you. DO NOT NAME YOUR PROJECT THE SAME AS ANY OF YOUR MODELS. 

Days we talked about this in class (for the Academy Twitch stream): 
- 11/17/2020 ("PangketManster" day - whole project setup)
- 11/18/2020 (John talks specifically about DB setups and linking)

For linking two databases: https://github.com/uugengiven/Session11MvcTwotables
The start of our sticky notes project (no model): https://github.com/jeanlange/Session11StickyStart/tree/main/Models
John's stickies with lots of good jquery stuff: https://github.com/uugengiven/Session11Stickies


## Create a project and open it in VS Code
```
$ dotnet new mvc -o ((Name))
code -r ((Name))
```

## Setting up the database
- Create ((Name)).Data namespace (folder) in your project 
- Create a dbcontext file in that folder for your project
-- Add whatever models from your project into your `using` at the top of this file (ie, `using project.Models`)
- In startup.cs, use dependency injection to pass dbcontext file into MVC app (add `using dbcontext.Data`)
-- Under `ConfigureServices` function, add db stuff.
- Update appsettings.json file
- Before you migrate, `dotnet build` (or ctrl+f5) to check for errors.

- Migrate! `dotnet ef migrations add Initial((Name))Create` <-- This creates the migration folder.

- Build the database! `dotnet ef database update`

## The general pattern for setting up a database
After you do these steps, *build it* to make sure there are no errors.

1. Create a class
- Make a Data folder, with a ((Name))DbContext.cs file inside
2. Give that class a connection string 
- Also, edit appsettings.json file under AllowedHosts section
3. Make the class global so that it can be used throughout the program
- In the Startup.cs file, under the ConfigureServices section 
- `services.AddDbContext<((Name))DbContext> (options => options.UseSqlite(Configuration.GetConnectionString("((Name))Db"))` <-- You will probably just copy/paste this or look at a tutorial for it, but that's the general pattern.
- Add a `using ((Name))Db.Data` to your Startup.cs and `using Microsoft.EntityFrameworkCore;` or whatever you need to make Sqlite (or your db) happen

## Scaffolding stuff for Visual Studio Code 
You need your models to exist before this happens.

Navigate to the project folder and run these to get all the packages you need for VS Code:
```
$ dotnet tool install --global dotnet-ef
dotnet tool install --global dotnet-aspnet-codegenerator
dotnet add package Microsoft.EntityFrameworkCore.SQLite
dotnet add package Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.Extensions.Logging.Debug
```
Run the scaffolding code - your model needs to exist and you should use that name, as well as the existing context name.

```
$ dotnet aspnet-codegenerator controller -name ((Names))Controller -m ((MODELName)) -dc ((Name))Context --relativeFolderPath Controllers --useDefaultLayout --referenceScriptLibraries
```
- Controller name: Plural, Capitalized, ie MoviesController
- Model name: Singlular, Capitalized, By Itself, ie Movie
- Context name: Capitalized, ie MvcMovieContext, it's named after the PROJECT and not the tables and will probably be the same for any scaffolding in these smaller projects.

***BUILD IT HERE!*** 
`$ dotnet build`

## To set up multiple tables in a db:
- Create your first file under Models ((Name.cs)) 
```
public class Show
{
    public int Id {get;set;}
    public string Network {get;set;}
    public string Name {get;set;}
    public string ArtisticDirector {get;set;}

    public virtual List<Character> Characters {get;set;}
}
```

- Create your second file under Models ((OtherName.cs))
```
public class Character
{
    public int Id {get;set;}
    public string Name {get;set;}
    public string OriginStory {get;set;}

    public virtual Show Show {get;set;} 
}
```

The "public virtual" lines connect and pull from the other DBs. "Navigation properties" is what they're called. 

- Connect it all back through the Data/((Name))DbContext.cs file.
```
public DbSet<Show> Shows {get;set;}
public DbSet<Character> Characters {get;set;}
```
If they're linked, migrating one will actually create both...but if you want to be able to select from both, you need them both in your context file.

- Scaffold both! See above.

## Connect the two inside your "Create" function
- Update the Create view for the one to take in info for the other (ie, select the Show for a Character in your create view). 

- Update your Controller Create function to look at the reference ID (ie, get the Show for a Character). Make sure you also pass it as an argument into the Controller (int showId).  
```
Show selectedShow = _context.Shows.Find(showId);
character.Show = selectedShow;
```
## To add items from one table in another table's view:
- If you want to display from both in the Index, update that too (@item.Show.Name or whatever).

- Update your Controller Index function with a JOIN. `return View(await _context.Characters.Include(c => c.Show).ToListAsync());` <-- "Include" is how you do a JOIN in C#.

- In your Controller, `var allShows = _context.Shows.ToList()` <-- this gets all the shows from the current instance of the database. Then, `ViewBag.Shows = allShows;` <-- this gives you access to those shows so that you can access them in that view. Then, you can iterate over them in your View `@foreach (var show in ViewBag.Shows)` <-- this lets you apply HTML to each `show`

- You can nest a foreach loop inside a foreach loop to list every character in a show, for example. 
