# ETL Project Report

### Team members:
- Nelson Paulo Balino
- Ezequiel Gallo
- Angela Wang

## ERD Diagram
![alt text](Images/erd.png "ERD Diagram")

## GCP Server
We decided to use the Google Cloud Platform's SQL instance as it would allow the team to load and access the data remotely. To set up the server, we first created a new project in GCP, setting up a virtual machine having the bare minimum specifications to run our databases and whitelisted each of our IPs.

In GCP we have the option of choosing a PostgresSQL 13 type database, which is the technology of choice for Relational Databases. We called our GCP instance and database "moovooks123" and here is were we will create and publish our tables.

The process was completed by sharing the GCP server credentials with the team for everyone to connect through their pgadmin platform or SQLAlchemy in Python. 

## Goodreads and Amazon

#### Extract

From Kaggle, we were able to obtain an Amazon dataset that contained the bestseller list over the years. We were also able to obtain a large dataset from Goodreads that contained a large list of books that have been published.

The objective was to find which book titles were also best sellers. To achieve this we used Amazon best sellers data, as a base reference, and the Good Reads dataset.

#### Transform
Prior to classifying, both datasets are cleaned until a desirable state for comparison is achieved. The comparison is performed via title names. Due to names not being an ideal intersection parameter due to poor data quality on `title_names` we decided to incorporate a Jaro - Winkler string matching method from the `Jellyfish` Python library.

![alt text](Images/jellyfish_2.png "Jellyfish comparison test")

After a series of tests we decided that anything above a `0.74` jaro_winkler score was acceptable. Therefore, each title name from the Good Reads dataset is string-tested with those in Amazon and if the titles have a 0.75 or above score those titles are classified in `amazon_bestsellers` as `True`.

![alt text](Images/jellyfish_3.png "Further jellyfish comparison test")

#### Load
Lastly, the data is uploaded into our GCP Server postgres instance. This added a new layer of complexity requiring more cleaning and restructuring as well as synchronising uploads based on foreign key constraints by first populating `movie_numbers` or `goodread_books` and lastly `books_to_movies`

![alt text](Images/books_gcp.png "Final table of books in GCP server")

## Wikipedia Scraping

#### Extract
Through research we found that Wikipedia had a concise list of books that had been adapted to feature films. From our lessons on BeautifulSoup and Splinter we believed that we could easily follow the links in the page and obtain relationships between the `isbn` number of the books and the `imdb_id` of the film.

![alt text](Images/wikipedia.png "Screenshot of Wikipedia page")

After scraping all the data from the tables we combined all the tables using `concat` function to produce a complete dataset.

#### Transform
After inspecting the dataframe we noticed that both movies and books have multiple values in the same row, 
due to some books having multiple movie adaptations and books being in a series. By using `explode` function we were able to split the movies and books. This cleaned dataframe was then saved as a .csv file.

## Further Wikipedia Scraping

#### Extract
Using the movie names listed in movies column of the csv file produced from the previous Wikipedia scrape, we were able to scrape all possible `imdb_id` in `imdb_link` by going to each of the movie's wikipedia page.

After scraping for links we noticed that some movie pages contained external wikipedia links that we would have to access in order to obtain the `imdb_id` for those movies. We developed a further scraping function which targeted these external links specifically.

A similar process was used to retrieve the `isbn` for all corresponding books within the Wikipedia list.

#### Transform
Following the retrieval of the relevant data, we split the `imdb_id` from the `imdb_links` obtained in the scrape.

For the books, we split each book from the published year using regex split so that it will only split the year and not the subtitles of each books with the delimiter "(".

Next we used `jellyfish.jaro_winkler_similarity` module to score match the book title from kaggle goodreads against this dataframe. The dataframe was then cleaned and only the rows without any nan values were stored to avoid any errors when connecting the tables.

![alt text](Images/movies_to_books.PNG "Cleaned books to movies table")

#### Load
The csv file was imported into pgAdmin and acts as a composite key to connect the data from the movies to books.

![alt text](Images/books_movies_gcp.png "Final table that connects movies to books in GCP server")

## Movies Dataset

#### Extract
A large dataset from Kaggle was used to form the base of our movie dataset as it contained a large collection of information. However to answer our initial question about whether popular books led to profitable movies, we felt that we required additinal information regarding `production_budget` and `worldwide_gross` for the movies. 

We discovered that [The Numbers](https://www.the-numbers.com/movie/budgets/all) had a large collection of the information we required. 

![alt text](Images/the-numbers-data.png "Data obtained from The Numbers page")

Initially we had hoped to use Splinter to toggled through the various pages to scrape all the information, however we soon realised that there was no distinct 'next' button. Fortunately we realised that the URLs of the subsequent pages all followed a regular pattern and so simple loop was used to scrape the succeeding pages and appended the landing page data to create a complete dataframe.

![alt text](Images/the-numbers-url.png "URL pattern")

#### Transform
The data from [The Numbers](https://www.the-numbers.com/movie/budgets/all) page all came in as strings. This made it difficult to manipulate such columns such as `ProductionBudget`, `DomesticGross` and `WorldwideGross`. These columns were strip of symbols and converted to integers. The column `ReleaseDate` was also cleaned to extract the year as this would help match entries explained below. An additional column `profit_percent` was added to calculate the total revenue of each movie which would help provide insight into our initial question.

In order to join The Numbers information, we used the `jellyfish.jaro_winkler` module which took in two strings and provided a score of it's similarity with 1.0 being an exact match. A score of more than 0.9 was needed to join the entries with the year from both sets the same. Certain matches were less than 1.0 due to formatting from the various sources.

![alt text](Images/jellyfish.png "Jellyfish match scores")

#### Load
A connection was made to the GCP SQL server and the data was simply appended to the existing table with `sqlalchemy`. This provided a more streamline workflow through Juptyer Lab. 

![alt text](Images/movies_gcp.png "Final table of movies data in GCP server")