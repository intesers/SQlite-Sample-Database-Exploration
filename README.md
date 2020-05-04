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
    
## Intermediate Challenge

**5. Do longer or shorter length albums tend to generate more revenue?**

    WITH song_sales AS (SELECT invoice_items.TrackId, SUM(invoice_items.UnitPrice) 'revenue'
    FROM invoice_items 
    GROUP BY invoice_items.TrackId)
    
    SELECT albums.Title, SUM(tracks.Milliseconds)/1000/60 'Album Length', SUM(song_sales.revenue) 'Total Revenue'
    FROM albums
    JOIN tracks ON albums.AlbumId = tracks.AlbumId
    JOIN song_sales ON tracks.TrackId = song_sales.TrackId
    GROUP BY albums.Title
    ORDER BY SUM(song_sales.revenue) DESC;

**6. Is the number of times a track appear in any playlist a good indicator of sales?**

    WITH sales AS (SELECT invoice_items.TrackId, SUM(invoice_items.UnitPrice) 'Revenue'
    FROM invoice_items
    GROUP BY invoice_items.TrackId)

    SELECT playlist_track.TrackId, COUNT(playlist_track.TrackId) 'Number of Appearance in Playlists', SUM(sales.Revenue) 'Total Sales'
    FROM playlist_track
    JOIN sales ON playlist_track.TrackId = sales.TrackId
    GROUP BY playlist_track.TrackId
    ORDER BY SUM(sales.Revenue) DESC;
    
 ## Advanced Challenge

**7. How much revenue is generated each year, and what is its [percent change from the previous year?**

    WITH prevYear AS (SELECT CAST(strftime('%Y', invoices.InvoiceDate) AS INT) 'PreviousYear', SUM(invoices.Total) 'PrevYearRevenue'
    FROM invoices
    GROUP BY CAST(strftime('%Y', invoices.InvoiceDate) AS INT)),
    currentYear AS (SELECT CAST(strftime('%Y', invoices.InvoiceDate) AS INT) 'CurrentYear', SUM(invoices.Total) 'CurrentYearRevenue'
    FROM invoices
    GROUP BY CAST(strftime('%Y', invoices.InvoiceDate) AS INT))
    
    SELECT currentYear.CurrentYear, currentYear.CurrentYearRevenue, ROUND((CurrentYearRevenue - PrevYearRevenue)/PrevYearRevenue * 100, 2) 'Percent Change'
    FROM currentYear
    JOIN prevYear ON currentYear.CurrentYear = prevYear.PreviousYear + 1;
   
    
Alternative and easier version that I late figured out,

    SELECT 
      CAST(strftime('%Y', InvoiceDate) AS INT) Year, 
      SUM(Total) "Total Revenue",
      LAG(SUM(Total)) OVER() 'previous value', /*The lag fucnction returns the previous value of a row in a column*/
      ROUND((SUM(Total) - LAG(SUM(Total)) OVER())/LAG(SUM(Total)) OVER() * 100, 2) 'percentage change'
    FROM invoices
    GROUP BY Year;

