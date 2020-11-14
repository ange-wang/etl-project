# ETL Project

## Team members:
- Nelson Paulo Balino
- Ezequiel Gallo
- Angela Wang

## Potential Analyis of Database
1. Understanding the relationship between top selling books and movie adaptations.
2. Understanding if there is a distinction between profit and budget of Netflix movies compared to traditional movies.
3. Emergence of trends in blockbuster movies.

## Extract:
Using the following Kaggle dataset as a base:
- Amazon Top 50 Bestselling Books (https://www.kaggle.com/sootersaalu/amazon-top-50-bestselling-books-2009-2019)
- GoodReads Books (https://www.kaggle.com/jealousleopard/goodreadsbooks)
- Movies Metadata (https://www.kaggle.com/rounakbanik/the-movies-dataset?select=movies_metadata.csv)
-  Netflix Shows (https://www.kaggle.com/shivamb/netflix-shows)
Using the following websites to scrape data from:
- Movie Budgets and Gross (https://www.the-numbers.com/)
- Wikepedia List of Bestselling books and Revenue (https://en.wikipedia.org/wiki/List_of_best-selling_books)
- Wikipedia List 

## Transform:
- Identify key fields and determine necessary fields to push to appropriate collections.
- Using Jellyfish library to match book titles to movie titles and acts as a composite key.

## Load:
- Create a sqlite file which would be hosted in a bucket in GCP to ensure all members have access to the latest file.
