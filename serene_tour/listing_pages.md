# Listing Pages

Serene has listing pages and editing interface for Northwind database. Let's have a look at the Products page under Northwind module.

![Products Page Initial](img/products_page_initial.jpg)

Here we see list of products sorted by product name (initial sort order).

You can change order by clicking column headers. To sort descending, click the same column header again.

To sort by multiple columns, you can use Shift+Click.

Here is what it looks like after sorting by Category then Supplier columns:

![Products Category Supplier Sort](img/products_category_supplier.jpg)

When you changed sort order, grid loaded data from a service with an AJAX request. 

> When you open the page first time, initial records were also loaded by an AJAX call.

By default grid loads records by 100 page size. Only records in current page are loaded from server. In the sample image, i changed page size to 20 (bottom left of grid) to show paging in effect.

All sorting, paging and filtering is done on server side with dynamic SQL queries generated by Serenity service handlers.

On top left of the grid, you can type something to do a simple search.

Type *coffee* for example to see products containing it in their names.




