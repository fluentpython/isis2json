===============================================
isis2json: CDS/ISIS to JSON database converter
===============================================

The isis2json.py is a Python/Jython script to export ISIS (MST+XRF)
or ISO-2709 databases to JSON files, optionally compatible with CouchDB
and MongoDB.

Running under Jython, both MST+XRF and ISO-2709 files can be read,
thanks to the Bruma Java library found in the lib/ directory.
Running under Python, only ISO-2709 files can be read.

A full description of how this script is used can be found in 
`From ISIS to CouchDB: Databases and Data Models for Bibliographic Records`__.

__ http://journal.code4lib.org/articles/4893

Usage
======

::

  $ ./isis2json.py -h
  usage: isis2json.py [-h] [-o OUTPUT.json] [-c] [-m] [-t ISIS_JSON_TYPE]
                      [-q QTY] [-s SKIP] [-i TAG_NUMBER] [-u] [-p PREFIX]
                      [-n] [-k TAG:VALUE]
                      INPUT.(mst|iso)

  Convert an ISIS .mst or .iso file to a JSON array

  positional arguments:
    INPUT.(mst|iso)     .mst or .iso file to read

  optional arguments:
    -h, --help          show this help message and exit
    -o OUTPUT.json, --out OUTPUT.json
                        the file where the JSON output should be written
                          (default: write to stdout)
    -c, --couch         output array within a "docs" item in a JSON document
                          for bulk insert to CouchDB via POST to db/_bulk_docs
    -m, --mongo         output individual records as separate JSON objects,
                          one per line for bulk insert to MongoDB via
                          mongoimport utility
    -t ISIS_JSON_TYPE, --type ISIS_JSON_TYPE
                        ISIS-JSON type, sets field structure:
                          1=string, 2=alist, 3=dict
    -q QTY, --qty QTY   maximum quantity of records to read (default=ALL)
    -s SKIP, --skip SKIP  records to skip from start of .mst (default=0)
    -i TAG_NUMBER, --id TAG_NUMBER
                        generate an "_id" from the given unique TAG field
                          number for each record
    -u, --uuid          generate an "_id" with a random UUID for each record
    -p PREFIX, --prefix PREFIX
                        concatenate prefix to every numeric field tag
                          (ex. 99 becomes "v99")
    -n, --mfn           generate an "_id" from the MFN of each record
                          (available only for .mst input)
    -k TAG:VALUE, --constant TAG:VALUE
                        Include a constant tag:value in every record
                          (ex. -k type:AS)


ISIS-JSON Record Types
=======================

There are many ways to represent CDS/ISIS records in JSON [#]_. This
utility currently exports ISIS-JSON types 1, 2 and 3.

Given an ISIS record with this strcuture::

   2 «538886»
  10 «Kanda, Paulo Afonso^1USP^2FMUSP^3CRDC^pBrasil^cSão Paulo^rorg»
  10 «Smidth, Magali Taino^1USP^2FMUSP^3CRDC^pBrasil^cSão Paulo^rorg»

Below are the three supported representations of that record in JSON:

ISIS-JSON type 1
-----------------

::

  {"10":
      ["Kanda, Paulo Afonso^1USP^2FMUSP^3CRDC^pBrasil^cSão Paulo^rorg",
       "Smidth, Magali Taino^1USP^2FMUSP^3CRDC^pBrasil^cSão Paulo^rorg"],
   "2":
      ["538886"]
  }

ISIS-JSON type 2
-----------------

::

    {"10":
        [
            [
                ("_", "Kanda, Paulo Afonso"),
                ("1", "USP"),
                ("2", "FMUSP"),
                ("3", "CRDC"),
                ("p", "Brasil"),
                ("c", "São Paulo"),
                ("r", "org")
            ],
            [
                ("_", "Smidth, Magali Taino"),
                ("1", "USP"),
                ("2", "FMUSP"),
                ("3", "CRDC"),
                ("p", "Brasil"),
                ("c", "São Paulo"),
                ("r", "org")
            ]
        ],
     "2":
        [
            [
                ("_", "538886")
            ]
        ]
    }

ISIS-JSON type 3
-----------------

::

    {"10":
        [
            {
                "_": "Kanda, Paulo Afonso",
                "1": "USP",
                "2": "FMUSP",
                "3": "CRDC",
                "c": "São Paulo",
                "p": "Brasil",
                "r": "org"
            },
            {
                "_": "Smidth, Magali Taino",
                "1": "USP",
                "2": "FMUSP",
                "3": "CRDC",
                "c": "São Paulo",
                "p": "Brasil",
                "r": "org"
            }
        ],
     "2":
        [
            {
                "_": "538886"
            }
        ]
    }


.. [#] See section 4.1 of http://journal.code4lib.org/articles/4893


Dependencies
=============

Under Python, isis2json.py depends on:

- Python2.6 or 2.7
- argparse.py (bundled; also part of the CPython 2.7 distribution)

Under Jython, isis2json.py depends on:

- Jython 2.5;
- argparse.py (bundled)
- Bruma.jar on the CLASSPATH (bundled);
- jyson-1.0.1.jar on the CLASSPATH (bundled);

Example CLASSPATH::

  export CLASSPATH=/home/luciano/lib/Bruma.jar:/home/luciano/lib/jyson-1.0.1.jar


Troubleshooting
================

SyntaxError on `yield fields` running isis2json.py under Jython
-------------------------------------------------------------------

If you see this::

  Traceback (innermost last):
    (no code object) at line 0
    File "./isis2json.py", line 84
          yield fields
              ^
  SyntaxError: invalid syntax

You are probably running Jython 2.2, an old version that is packaged
with several Linux distributions such as Debian and Ubuntu. To verify,
type::

  $ jython --version
  Jython 2.2.1 on java1.6.0_20

To fix, download and install Jython 2.5 or later from the Jython project
on SourceForge_.

.. _SourceForge: http://sourceforge.net/projects/jython/files/jython/

IMPORT ERROR: Jython 2.5 and Bruma.jar are required to read .mst files
-----------------------------------------------------------------------

Check if Jython 2.5 or later is installed::

  $ jython --version
  Jython 2.5.2

If it is not, se issue above. If it is, add the path to Bruma.jar to
the CLASSPATH environment variable, or pass it via the `jython -J-cp`
command line option when running isis2json.py, like this::

  $ jython -J-cp lib/jyson-1.0.1.jar:lib/Bruma.jar isis2json.py fixtures/LILACS1.mst


