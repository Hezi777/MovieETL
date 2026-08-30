<a id="readme-top"></a>

<h1 align="center">
  <img width="150" alt="IMDb" src="https://cdn.simpleicons.org/imdb/F5C518" />
  <br />
  <b>IMDb Movies ETL Pipeline</b>
</h1>

<p align="center">
  An end-to-end ETL pipeline that ingests raw IMDb TSV dumps, cleans and
  transforms them with Pandas, and loads them into PostgreSQL as two normalized
  tables.
</p>

<p align="center">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.8%2B-3776AB?style=for-the-badge&logo=python&logoColor=white" />
  <img alt="pandas" src="https://img.shields.io/badge/pandas-150458?style=for-the-badge&logo=pandas&logoColor=white" />
  <img alt="SQLAlchemy" src="https://img.shields.io/badge/SQLAlchemy-D71F00?style=for-the-badge&logo=sqlalchemy&logoColor=white" />
  <img alt="PostgreSQL" src="https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white" />
  <img alt="Jupyter" src="https://img.shields.io/badge/Jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white" />
  <img alt="License" src="https://img.shields.io/github/license/Hezi777/MovieETL?style=for-the-badge" />
</p>

<p align="center">
  <a href="#about">About</a> &nbsp;|&nbsp;
  <a href="#etl-workflow">ETL Workflow</a> &nbsp;|&nbsp;
  <a href="#database-schema">Database Schema</a> &nbsp;|&nbsp;
  <a href="#tech-stack">Tech Stack</a> &nbsp;|&nbsp;
  <a href="#prerequisites--setup">Getting Started</a>
</p>

---

## About

IMDb publishes its catalogue as gzipped TSV dumps, which are wide, denormalized,
and use `\N` rather than empty cells for missing values. This pipeline turns
that into a queryable relational schema.

**The one modelling decision worth knowing:** genres arrive as a comma-separated
list inside a single column. Rather than storing that string, the transform
explodes it into a `movie_genres` row per genre, so "every drama released after
2010" is a join instead of a `LIKE '%Drama%'` scan.

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

## Tech Stack

Pinned in `requirements.txt`.

| Layer | Technology |
|---|---|
| Language | Python 3.8+ |
| Transform | pandas, numpy |
| Database | PostgreSQL via SQLAlchemy + psycopg2-binary |
| Environment | Jupyter Notebook |

---

## License

MIT - see the [LICENSE](LICENSE) file for details.

> IMDb is a trademark of IMDb.com, Inc. This is an independent learning project
> and is not affiliated with or endorsed by IMDb. Their datasets are used under
> the terms published at [imdb.com/interfaces](https://www.imdb.com/interfaces/).

<p align="right">(<a href="#readme-top">back to top</a>)</p>
