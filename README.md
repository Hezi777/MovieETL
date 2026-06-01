# IMDb Movies ETL Pipeline

An end-to-end ETL pipeline that ingests raw IMDb data, cleans and transforms it with Pandas, and loads it into a PostgreSQL database via SQLAlchemy. Splits movies and genres into two normalized tables for easy querying.

---

## Repository Structure

```
.
├── ETL file.ipynb        <- Jupyter notebook with the full ETL workflow
├── requirements.txt      <- Python dependencies
└── assets/
    └── erd_screenshot.png
```

---

## Prerequisites & Setup

**Prerequisites:** Python 3.8+, PostgreSQL, pip

```bash
git clone https://github.com/Hezi777/MovieETL.git
cd MovieETL
python -m venv venv
source venv/bin/activate      # macOS/Linux
# venv\Scripts\activate       # Windows
pip install -r requirements.txt
```

In the notebook's first code cell, update the connection URL:

```python
engine = create_engine(
    "postgresql://<username>:<password>@<host>:<port>/<database>"
)
```

---

## ETL Workflow

### 1. Extract

Load IMDb TSV files from [IMDb Datasets](https://www.imdb.com/interfaces/):
- `title.basics.tsv.gz` - movie metadata
- `title.ratings.tsv.gz` - user ratings

### 2. Transform

- Clean nulls (`\N` to `None`)
- Convert types: years to integers, ratings to floats, votes to big integers
- Filter to `"movie"` entries only
- Normalize genres: explode the list into separate rows

### 3. Load

- Create tables via SQLAlchemy ORM
- Bulk-insert `movies` and `movie_genres` DataFrames using `df.to_sql(..., method='multi')`

---

## Running

```bash
jupyter notebook
```

Open `ETL file.ipynb` and run all cells top to bottom. Verify in your PostgreSQL client that `movies` and `movie_genres` were populated.

---

## Database Schema

![ERD](assets/erd_screenshot.png)

**movies**

| Column | Type | Description |
|---|---|---|
| `movie_id` | TEXT | IMDb title identifier (tconst) |
| `title` | TEXT | Primary title |
| `is_adult` | BOOLEAN | Adult-only flag |
| `year` | INTEGER | Release year |
| `runtime_minutes` | INTEGER | Duration in minutes |
| `average_rating` | FLOAT | IMDb user rating (0-10) |
| `num_votes` | INT | Number of votes |

**movie_genres**

| Column | Type | Description |
|---|---|---|
| `id` | SERIAL | Auto-incrementing primary key |
| `movie_id` | TEXT | Foreign key to `movies.movie_id` |
| `genre` | TEXT | Single genre per row |

---

## Dependencies

Listed in `requirements.txt`:

- pandas
- numpy
- SQLAlchemy
- psycopg2-binary
- jupyter

---

## License

MIT - see the [LICENSE](LICENSE) file for details.
