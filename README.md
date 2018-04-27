# Windows Kubernetes Samples

This is a group of sample apps that I use to build demos. Check the history to get an idea of when each was used last.


## RazorPagesMovie

> This is based on the samples from https://github.com/aspnet/Docs, specifically https://github.com/aspnet/Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie . I added the README.md and scripts needed to build locally with Docker for Windows, then deploy on Kubernetes.

### Build & test locally

Build it - `docker build --pull -t razorpagesmovie .`

Test it - `docker run --name razorpagesmovie --rm -it -p 8000:80 razorpagesmovie`


### Current issues

It looks like the entity framework is passing syntax that works with SQL, but not SQLite. SQLite uses nvarchar() without the max keyword

```
fail: Microsoft.EntityFrameworkCore.Database.Command[200102]
      Failed executing DbCommand (0ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
      CREATE TABLE "Movie" (
          "ID" int NOT NULL CONSTRAINT "PK_Movie" PRIMARY KEY,
          "Genre" nvarchar(max) NULL,
          "Price" decimal(18, 2) NOT NULL,
          "ReleaseDate" datetime2 NOT NULL,
          "Title" nvarchar(max) NULL
      );
Microsoft.Data.Sqlite.SqliteException (0x80004005): SQLite Error 1: 'near "max": syntax error'.
   at Microsoft.Data.Sqlite.SqliteException.ThrowExceptionForRC(Int32 rc, sqlite3 db)
   at Microsoft.Data.Sqlite.SqliteCommand.ExecuteReader(CommandBehavior behavior)
   at Microsoft.Data.Sqlite.SqliteCommand.ExecuteNonQuery()
   at Microsoft.EntityFrameworkCore.Storage.Internal.RelationalCommand.Execute(IRelationalConnection connection, DbCommandMethod executeMethod, IReadOnlyDictionary`2 parameterValues)
fail: RazorPagesMovie.Program[0]
      An error occurred seeding the DB.
Microsoft.Data.Sqlite.SqliteException (0x80004005): SQLite Error 1: 'near "max": syntax error'.
   at Microsoft.Data.Sqlite.SqliteException.ThrowExceptionForRC(Int32 rc, sqlite3 db)
   at Microsoft.Data.Sqlite.SqliteCommand.ExecuteReader(CommandBehavior behavior)
   at Microsoft.Data.Sqlite.SqliteCommand.ExecuteNonQuery()
   at Microsoft.EntityFrameworkCore.Storage.Internal.RelationalCommand.Execute(IRelationalConnection connection, DbCommandMethod executeMethod, IReadOnlyDictionary`2 parameterValues)
   at Microsoft.EntityFrameworkCore.Storage.Internal.RelationalCommand.ExecuteNonQuery(IRelationalConnection connection, IReadOnlyDictionary`2 parameterValues)
   at Microsoft.EntityFrameworkCore.Migrations.Internal.MigrationCommandExecutor.ExecuteNonQuery(IEnumerable`1 migrationCommands, IRelationalConnection connection)
   at Microsoft.EntityFrameworkCore.Migrations.Internal.Migrator.Migrate(String targetMigration)
   at RazorPagesMovie.Program.Main(String[] args) in C:\app\aspnetapp\Program.cs:line 26
```


## Random Tips

- Greg Bayer has a good blog post on [Moving FIles from one Git Repository to Another, Preserving History](https://gbayer.com/development/moving-files-from-one-git-repository-to-another-preserving-history/) which I used to seed this repo with just the sample code, instead of the full .Net doc set :)