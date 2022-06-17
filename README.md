# ubu-block
Uchaguzi bila Ubaguzi Blockchain

## Getting Started

This is a development guide to getting started, binaries coming soon

```
git clone <repo> ubu-block
cd ubu-block
```

### Setup initial files

```
mkdir data

# Create the db files
cp src/sql/empty.db data/blockchain.db
cp src/sql/empty.db data/private.db

```

### Initialize a Blockchain

```
cargo run init --creator "Njuguna Mureithi"
```

You should get:

```
INFO  ubu_block] Blockchain was successfully initialized!
```

### Add some blocks

Currently `ubu-block` has very limited dummy data, but this should change as soon as IEBC publishes the voter register

For testing purposes, I used limited data from previous by elections. See `src/sql/main_db.sql`

```
cargo run insert --station 022113056303301 --candidate 1 --votes 21
cargo run insert --station 022113056303301 --candidate 2 --votes 66
```

You should get:

```
INFO ubu_block] Block was added successfully!
```

### Validate our blockchain

```
cargo run validate
```

You should get:

```
INFO ubu_block] Blockchain is valid!
```

### Querying

```
cargo run query -q "Select
  c.name as candidate,
   SUM(votes) as votes,
  ward_name as ward,
  constituency_name as constituency,
  county_name as county,
  parties.title as party
from
  results
  INNER JOIN stations ON stations.id = results.station_id
  INNER JOIN candidates c ON c.id = results.candidate_id
  INNER JOIN wards on stations.ward_code = wards.ward_code
  INNER JOIN parties ON parties.id = c.party_id
  INNER JOIN constituencies ON wards.constituency_code = constituencies.constituency_code
  INNER JOIN counties ON constituencies.county_code = counties.county_code
WHERE
  position_type = 'Mp'  and constituency = 'Juja' GROUP BY candidate ;"
```

**_NOTE:_** Currently each query has to return the following columns to work: candidate, votes, ward, constituency, county, party. This is just a temporary issue and should fixed in the next release

You should get:

```
+--------+--------------+----------+-----------+-------+-------+
| county | constituency |   ward   | candidate | party | votes |
+--------+--------------+----------+-----------+-------+-------+
| Kiambu |     Juja     | Kalimoni |   Mwas    |  ODM  |  21   |
+--------+--------------+----------+-----------+-------+-------+
| Kiambu |     Juja     | Kalimoni |   Omosh   |  PNU  |  66   |
+--------+--------------+----------+-----------+-------+-------+
```

### Trying to steal the election from Omosh

Since immutability is one of our main goal, lets try to edit votes for `Mwas` and see if we can get away with it

Open `blockchain.db` with your favourite sqlite editor and run a query that updates the results:

```sql
UPDATE "results" SET "votes"= 71 WHERE _rowid_ =1
```

Running `query` again we get:

```
+--------+--------------+----------+-----------+-------+-------+
| county | constituency |   ward   | candidate | party | votes |
+--------+--------------+----------+-----------+-------+-------+
| Kiambu |     Juja     | Kalimoni |   Mwas    |  ODM  |  71   |
+--------+--------------+----------+-----------+-------+-------+
| Kiambu |     Juja     | Kalimoni |   Omosh   |  PNU  |  66   |
+--------+--------------+----------+-----------+-------+-------+
```

Wow congrats to Omosh!

![Omosh](https://kenyaleo.co.ke/wp-content/uploads/2021/06/1-221.jpg)

Hold on, Hold on

![Rao Petion](https://images.hivisasa.com/1200/7JhHhhZWMbFB_IMG_1503130067538.jpg)

There is a petition, lets try validating our blockchain

```
cargo run validate

thread 'main' panicked at 'Could not verify block, found 0e70cebe0ab3bd8c3606a08d26483d092534eea4ccdb7816fc2692aee5ed3109, block: Block {... CandidateResult { station_id: 22113056303301, candidate_id: 1, votes: 71 }]......', src/db.rs:189:17

```

How about that? No opening servers and everything is public and sql friendly

## Roadmap

v 0.3

- [ ] Http API
- [ ] Mobile and Web apps

v 0.2

- [ ] P2p - ability to add nodes
- [ ] Fill regional data
- [ ] Views to simplify quering
- [ ] Setup triggers to `Before Insert` to prevent adding unmatching data

v 0.1

- [x] Clap
- [x] Database, sqlite
- [x] Blockchain
- [ ] CI/CD
