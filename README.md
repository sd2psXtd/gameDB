# sd2psx GameDB

This repository contains python scripts for creating sd2psx minimal game databases.
The game databases contain the following information for PS1/PS2:

- **Game ID:** Each game disc has its own ID which usually consists of a 4 Character Prefix (SCES e.g.) and a numeric numer (12345)
- **Title:** The title of the game
- **Parent ID:** In case of multi disc games: The Id of Disc 1


## Format

The game database consists of a 3 parts. The databases is described using the following example:

Game ID     | Game Name
------------|------------------------------------------
ABCD-01     | Game One
BCDE-02     | Game Two (Disc 1)
BCDE-12     | Game Two (Disc 2)

### Part 1: Prefix Mapping

Each used 4 character prefix is associated with its start offset as a 4 byte big endian unsigned value within the database file.

*Example:*

Prefix      | Offset
------------|-------------------
ABCD        | 17
BCDE        | 29

*Assembles to:*
Address | Data                                      
--------|-------------------
0x00    | 0x41 0x42 0x43 0x44 0x00 0x00 0x00 0x10
0x08    | 0x42 0x43 0x44 0x45 0x00 0x00 0x00 0x1C

### Part 2: Game Mapping

Each Game is represented by a tuple of 3 by its:
- numeric ID (4 Bytes, big endian)
- name Offset (4 Bytes, big endian)
- parent numeric ID (4 Bytes, big endian)

Game ID     | Numeric ID            | Name Offset         | Parent ID
------------|-----------------------|---------------------|---------------------
ABCD-01     | 0x00 0x00 0x00 0x01   | 0x00 0x00 0x00 0x34 | 0x00 0x00 0x00 0x01
BCDE-02     | 0x00 0x00 0x00 0x02   | 0x00 0x00 0x00 0x3D | 0x00 0x00 0x00 0x02
BCDE-12     | 0x00 0x00 0x00 0x0C   | 0x00 0x00 0x00 0x3D | 0x00 0x00 0x00 0x02

### Part 3: Name list

Contains each game name that is used in the database. All game names are 0x00 terminated. All meta information (like region, Disc Number etc.) is stripped from the game title.

The names are are simple list of strings.

{
    'G', 'a', 'm', 'e', ' ', 'O', 'n', 'e', 0x0, 'G', 'a', 'm', 'e', ' ', 'T', 'w', 'o', 0x00
}

### Assembled example

Offset | Data                                                            | Content
-------|-----------------------------------------------------------------|------------------------------------
0x00   | 'A'  'B'  'C'  'D'  0x00 0x00 0x00 0x10                         | Prefix ABCD
0x08   | 'B'  'C'  'D'  'E'  0x00 0x00 0x00 0x1C                         | Prefix BCDE
0x10   | 0x00 0x00 0x00 0x01 0x00 0x00 0x00 0x35 0x00 0x00 0x00 0x01     | Game ID ABCD-01
0x1C   | 0x00 0x00 0x00 0x02 0x00 0x00 0x00 0x3E 0x00 0x00 0x00 0x02     | Game ID BCDE-01
0x28   | 0x00 0x00 0x00 0x0C 0x00 0x00 0x00 0x3E 0x00 0x00 0x00 0x02     | Game ID BCDE-12
0x34   | 'G'  'a'  'm'  'e'  ' '  'O'  'n'  'e'  0x00                    | Title "Game One"
0x3D   | 'G'  'a'  'm'  'e'  ' '  'T'  'w'  'o'  0x00                    | Title "Game Two"

## Scripts

The scripts/ folder contains scripts for generating the DB from different sources:

- scripts/parse_GameDB.py : Parses GameDB from niemasd
- scripts/parse_db.py : Parses redump DB from API
- scripts/parse_hdldb.py : Parses israpps hdl batch installer DB