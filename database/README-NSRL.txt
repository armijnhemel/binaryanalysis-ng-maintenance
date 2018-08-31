The NSRL forensics database can be found at:

https://www.nist.gov/software-quality-group/national-software-reference-library-nsrl

The main contents of NSRL are published as multiple ISO images, each containing a few CSV files. NSRL version 2.60 (March 2018) has the following sets:

* modern
* android
* iOS
* legacy

Each of these ISO files contain several CSV files:

NSRLFile.txt (stored ZIP compressed as NSRLFile.txt.zip) -- maps checksums and names of individual files to products
NSRLMfg.txt -- information about manufacturers
NSRLOS.txt -- information about operating systems
NSRLProd.txt -- information about individual products

The first line of each file lists the field names.

Importing the data
==================

According to the documentation these CSV files ought to be UTF-8, but they are not and they need to be translated first.

Then they need to be made searchable by putting them into a database.

The script nsrlimporter.py takes care of both steps.

Requirements
------------

* PostgreSQL version that supports UPSERT (9.5 or higher)
* Python 3
* psycopg2 2.7.x

Loading the data
----------------

To use the files do the following:

1. add the right tables to the database:

$ psql -U username < nsrl-init.sql

2. mount the ISO files and copy the CSV files to a directory and make sure
that NSRLFile.txt.zip has been unzipped.

The easiest is to mount each file over loopback, for example:

# mkdir /tmp/mnt
# mount /home/armijn/nsrl/RDS_android.iso /tmp/mnt/

and then as a normal user:

$ cp /tmp/mnt/* /home/armijn/nsrl/android/
$ unzip NSRLFile.txt.zip

3. Import the data. It is advised to start with the 'modern' dataset, as this is the biggest.

run nsrlimporter.py:

$ python3 nsrlimporter.py -c /path/to/configuration/file -d /path/to/nsrl/directory

for example:

$ python3 nsrlimporter.py -c bang.config -d /home/armijn/nsrl/modern/

4. After the first directory has been processed the indexes should be created to ensure data correctness:

$ psql -U username < nsrl-index.sql

This limitation will be removed in the future.

5. import the other directories in a similar fashion

The database tables are now ready to be used, for example with the plugin for BANG.

Statistics:
-----------

Some statistics for a recent version of NSRL (2.60, March 2018):

bang=> \dt+
                         List of relations
 Schema |       Name        | Type  | Owner |  Size   | Description 
--------+-------------------+-------+-------+---------+-------------
 public | nsrl_entry        | table | bang  | 13 GB   | 
 public | nsrl_hash         | table | bang  | 7477 MB | 
 public | nsrl_manufacturer | table | bang  | 4208 kB | 
 public | nsrl_os           | table | bang  | 104 kB  | 
 public | nsrl_product      | table | bang  | 12 MB   | 
(5 rows)

bang=> \di+
                                      List of relations
 Schema |          Name          | Type  | Owner |       Table       |  Size   | Description 
--------+------------------------+-------+-------+-------------------+---------+-------------
 public | nsrl_entry_sha1        | index | bang  | nsrl_entry        | 16 GB   | 
 public | nsrl_hash_pkey         | index | bang  | nsrl_hash         | 5434 MB | 
 public | nsrl_manufacturer_pkey | index | bang  | nsrl_manufacturer | 1976 kB | 
 public | nsrl_os_pkey           | index | bang  | nsrl_os           | 48 kB   | 
 public | nsrl_product_pkey      | index | bang  | nsrl_product      | 6720 kB | 
(5 rows)

bang=> select from nsrl_entry ;
--
(179741841 rows)

bang=> select from nsrl_hash ;
--
(62275938 rows)

bang=> select from nsrl_manufacturer ;
--
(80636 rows)

bang=> select from nsrl_os ;
--
(1003 rows)

bang=> select from nsrl_product ;
--
(164295 rows)


