SQlite-Sample-Database-Exploration  
Exploring a SQLite Sample Database

The database can be found [here](http://www.sqlitetutorial.net/sqlite-sample-database)

## Basic Challenge

**1.  Which tracks appeared in the most playlists? how many playlist did they appear in?**

    SELECT playlist_track.TrackId, tracks.Name PlaylistId, COUNT(playlist_track.TrackId) "Number of appearance in playlist"
    FROM playlist_track
    JOIN tracks ON  playlist_track.TrackId = tracks.TrackId
    GROUP BY playlist_track.TrackId
    ORDER BY COUNT(playlist_track.TrackId) DESC;




**2. Which track generated the most revenue? which album? which genre?**

    SELECT invoice_items.TrackId, tracks.Name, albums.Title Album, genres.Name Genre, SUM(invoice_items.UnitPrice * invoice_items.Quantity) Total
    FROM invoice_items
    JOIN tracks ON invoice_items.TrackId = tracks.TrackId
    JOIN albums ON tracks.AlbumId = albums.AlbumId
    JOIN genres ON tracks.GenreId = genres.GenreId
    GROUP BY invoice_items.TrackId
    ORDER BY SUM(invoice_items.UnitPrice * invoice_items.Quantity) DESC;


**3. Which countries have the highest sales revenue? What percent of total revenue does each country make up?**

    SELECT invoices.BillingCountry, SUM(invoice_items.UnitPrice * invoice_items.Quantity) Total, SUM(invoice_items.UnitPrice * invoice_items.Quantity)/ SUM(SUM(invoice_items.UnitPrice * invoice_items.Quantity)) OVER() * 100 Percentage
    FROM invoices
    JOIN invoice_items ON invoice_items.InvoiceId = invoices.InvoiceId
    GROUP BY invoices.BillingCountry
    ORDER BY SUM(invoice_items.UnitPrice * invoice_items.Quantity) DESC;

**4. How many customers did each employee support, what is the average revenue for each sale, and what is their total sale?**

    SELECT employees.EmployeeId, customers.SupportRepId, COUNT(DISTINCT customers.CustomerId) 'Customers Supported', SUM(invoices.Total) 'Total Revenue', SUM(invoices.Total)/COUNT(DISTINCT customers.CustomerId) 'Average Revenue'
    FROM customers
    JOIN employees ON employees.EmployeeId = customers.SupportRepId
    JOIN invoices ON invoices.CustomerId = customers.CustomerId
    GROUP BY customers.SupportRepId;

