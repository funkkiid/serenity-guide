# Allowing Multiple Genre Selection

It happens. Requirements change. Now we want to allow selecting multiple genres for a movie.

For this, we need a M-N mapping table that will let us link any movie to multiple genres.


### Creating MovieGenres Table

As usual, we start with a migration:

**Modules/Common/Migrations/DefaultDB/ DefaultDB_20160528_115400_MovieGenres.cs:**

```
using FluentMigrator;

namespace MovieTutorial.Migrations.DefaultDB
{
    [Migration(20160528115400)]
    public class DefaultDB_20160528_115400_MovieGenres : Migration
    {
        public override void Up()
        {
            Create.Table("MovieGenres").InSchema("mov")
                .WithColumn("MovieGenreId").AsInt32()
                    .Identity().PrimaryKey().NotNullable()
                .WithColumn("MovieId").AsInt32().NotNullable()
                    .ForeignKey("FK_MovieGenres_MovieId", 
                        "mov", "Movie", "MovieId")
                .WithColumn("GenreId").AsInt32().NotNullable()
                    .ForeignKey("FK_MovieGenres_GenreId", 
                        "mov", "Genre", "GenreId");

            Execute.Sql(
              @"INSERT INTO mov.MovieGenres (MovieId, GenreId) 
                    SELECT m.MovieId, m.GenreId 
                    FROM mov.Movie m 
                    WHERE m.GenreId IS NOT NULL");

            Delete.ForeignKey("FK_Movie_GenreId")
                .OnTable("Movie").InSchema("mov");
            Delete.Column("GenreId")
                .FromTable("Movie").InSchema("mov");
        }

        public override void Down()
        {
        }
    }
}
```

I tried to save existing Genre declarations on Movie table, by copying them to our new MovieGenres table. The line above with *Execute.Sql* does this.

Then we should remove GenreId column, by first deleting the foreign key declaration *FK_Movie_GenreId* that we defined on it previously.


### Deleting Mapping for GenreId Column

As soon as you build and open the Movies page, you'll get this error:

![GenreId Error](img/mdb_genreid_error.png)

This is because we still have mapping for GenreId column in our row. Error above is received from AJAX call to *List* service handler for *Movie* table. 

> Repeating of error message originates from SQL server. *MovieId* column name passes several times within the generated dynamic SQL.

Remove GenreId and GenreName properties and their related field objects from *MovieRow.cs*:

```cs
// remove this
public Int32? GenreId
{
    get { return Fields.GenreId[this]; }
    set { Fields.GenreId[this] = value; }
}

// remove this
public String GenreName
{
    get { return Fields.GenreName[this]; }
    set { Fields.GenreName[this] = value; }
}

public class RowFields : RowFieldsBase
{
    // and remove these
    public Int32Field GenreId;
    public StringField GenreName;
}
```

Remove GenreName property from *MovieForm.cs*:

```cs
// remove this
[Width(100), QuickFilter]
public String GenreName { get; set; }
```

After building, we at least have a working *Movies* page again.


### Adding GenreList Field

As one movie might have multiple genres now, instead of a Int32 property, we need a list of Int32 values, e.g. `List<Int32>`. Add the GenreList property to *MovieRow.cs*:

```cs

//...
[DisplayName("Kind"), NotNull, DefaultValue(MovieKind.Film)]
public MovieKind? Kind
{
    get { return (MovieKind?)Fields.Kind[this]; }
    set { Fields.Kind[this] = (Int32?)value; }
}

[LookupEditor(typeof(GenreRow), Multiple = true), ClientSide]
[LinkingSetRelation(typeof(GenreRow), "MovieId", "GenreId")]
public List<Int32> GenreList
{
    get { return Fields.GenreList[this]; }
    set { Fields.GenreList[this] = value; }
}

public class RowFields : RowFieldsBase
{
    //...
    public Int32Field Kind;
    public CustomClassField<List<Int32>> GenreList;
```

For field definition, we have to use `CustomClassField<List<Int32>>` as we don't have a *list of something* field type. *CustomClassField* is a generic field type that accepts any class type as its generic parameter.

Our property has [LookupEditor] attribute just like *GenreId* property had, but with one difference. This one accepts multiple genre selection. We set it with *Multiple = true* argument.

This property also has *ClientSide* flag, which is something similar to *Unmapped* fields in Serenity. It specifies that this property has no matching database column in database. 

We don't have a *GenreList* column in *Movie* table, so we should set it as an unmapped field. Otherwise, Serenity will try to *SELECT* it, and we'll get SQL errors.



