Use case: we have presigned urls from S3. Every request to generate a presigned url creates a url with new query string params, so the browser does not cache it.

Solution : Cache the presigned urls in Indexed DB.

Details: Create a database in Indexed DB of Chrome.
          Add key as the original image url and the value as the presigned url
          Store a creation time along with this data in a map in the Indexed DB
          Write a 'Cache' class which checks for the requested image's presigned url in the Indexed DB.
          Check if the creation time is greater than time for expiry in presigned url, if yes then delete the url from cache and generate a new presigned url and store it in the DB.
          If the creation time is whithin expiry time of presigned url then return the presigned url.
          This will be the same url requested esrlier and should hit the browser/cloud front cache.
          
Code to be added*
