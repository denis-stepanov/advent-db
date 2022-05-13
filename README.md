# AdVent Database
A database for [AdVent](https://github.com/denis-stepanov/advent), the TV ads arrestor. For a description of how AdVent works, refer to its repository. This repository contains a database of TV jingle hashes. These are used by AdVent in order to decide when the ads sound has to be cut.

## Database Population or Update for Regular Users
To pull the latest updates into your database:

1. Download the latest snapshot (clone or get a zip):
```
$ git clone https://github.com/denis-stepanov/advent-db.git
$ cd advent-db
```
2. (recommended) delete countries or TV channels you are not expected to use. This will decrease CPU load and reduce the number of false positives while running AdVent:
```
$ rm -r DB/VA # DB/....
```
3. Update or import your database:
```
$ find DB -name "*.djv" | xargs db-djv-pg.py import
```
If your want to overwrite existing definitions (slower), add `-o` to the end of the import command.

