# @tsonic/efcore

TypeScript declarations and CLR binding metadata for Entity Framework Core 10.

`@tsonic/efcore` is a generated binding package for
`Microsoft.EntityFrameworkCore.*` assemblies. It provides the TypeScript import
surface and compiler metadata; your Tsonic workspace still references the real
EF Core NuGet assemblies.

## Install

```bash
npm install @tsonic/efcore @tsonic/dotnet @tsonic/core
```

## Use with Tsonic

Add the NuGet package and bind it to this type package:

```bash
tsonic add nuget Microsoft.EntityFrameworkCore <version> @tsonic/efcore
tsonic restore
```

Provider packages use the same model:

```bash
tsonic add nuget Microsoft.EntityFrameworkCore.Sqlite <version> @tsonic/efcore-sqlite
tsonic add nuget Microsoft.EntityFrameworkCore.SqlServer <version> @tsonic/efcore-sqlserver
tsonic add nuget Npgsql.EntityFrameworkCore.PostgreSQL <version> @tsonic/efcore-npgsql
```

- SQLite: `@tsonic/efcore-sqlite`
- SQL Server: `@tsonic/efcore-sqlserver`
- PostgreSQL (Npgsql): `@tsonic/efcore-npgsql`

## Imports

Import EF Core namespaces through explicit ESM subpaths:

```ts
import { DbContext, DbContextOptions, DbSet } from "@tsonic/efcore/Microsoft.EntityFrameworkCore.js";
```

Provider-specific APIs come from the provider package:

```ts
import { SqliteDbContextOptionsBuilderExtensions } from "@tsonic/efcore-sqlite/Microsoft.EntityFrameworkCore.js";
```

## LINQ and EF extension methods

Generated packages expose extension-method helper types. Use them with
`asinterface` to express C#-style `using` semantics at the TypeScript call site:

```ts
import { asinterface } from "@tsonic/core/lang.js";
import type { ExtensionMethods as Linq } from "@tsonic/dotnet/System.Linq.js";
import type { ExtensionMethods as Ef } from "@tsonic/efcore/Microsoft.EntityFrameworkCore.js";
import type { DbSet } from "@tsonic/efcore/Microsoft.EntityFrameworkCore.js";

type DbSetQuery<T> = Ef<Linq<DbSet<T>>>;

const q = asinterface<DbSetQuery<PostEntity>>(db.posts);

const pageviews = await q.Where((p) => p.Published === true).CountAsync();
const rows = await q.Where((p) => p.Published === true).ToArrayAsync();
```

## Minimal context example

```ts
import { DbContext, DbContextOptions, DbContextOptionsBuilder, DbSet } from "@tsonic/efcore/Microsoft.EntityFrameworkCore.js";
import { SqliteDbContextOptionsBuilderExtensions } from "@tsonic/efcore-sqlite/Microsoft.EntityFrameworkCore.js";

export class BlogDbContext extends DbContext {
  get posts(): DbSet<PostEntity> {
    return this.Set<PostEntity>();
  }

  constructor(options: DbContextOptions) {
    super(options);
  }
}

export const createDbOptions = (dbPath: string): DbContextOptions => {
  const optionsBuilder = new DbContextOptionsBuilder();
  SqliteDbContextOptionsBuilderExtensions.UseSqlite(optionsBuilder, `Data Source=${dbPath}`);
  return optionsBuilder.Options;
};
```

## Package shape

The package contains generated namespace facades, ESM stubs, internal
declarations, extension buckets, and `bindings.json` compiler metadata:

```text
@tsonic/efcore/
  Microsoft.EntityFrameworkCore.d.ts
  Microsoft.EntityFrameworkCore.js
  Microsoft.EntityFrameworkCore/
    bindings.json
    internal/index.d.ts
  __internal/extensions/index.d.ts
```

`bindings.json` preserves CLR identity, overloads, receiver semantics,
extension methods, nullable/reference metadata, and generic constraints.

## Broad CLR values

Generated CLR object slots are represented with TypeScript `unknown`, not a
package-specific catch-all value. This is important for EF Core because many
extension points accept broad CLR objects while query and model APIs still need
strict metadata for overload selection. Value-type constraints are represented
with `NonNullable<unknown>`.

## Versioning

This repo is versioned by .NET major:

- .NET 10 → npm: `@tsonic/efcore@10.x`

## Development

Regenerate from sibling checkouts:

```bash
npm install
./__build/scripts/generate.sh
```

The generation script requires .NET 10, `../tsbindgen`, `../dotnet`, and
`../microsoft-extensions`.

## License

MIT
