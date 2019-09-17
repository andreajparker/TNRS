# Files in this directory and subdirectories build the TNRS 3 database

Release date: 27 March 2014
Database version: 3.6.3
Database revision date: 27 March 2014
Application url: http://tnrs.iplantcollaborative.org/

# WARNING: Deprecated!

This version of TNRS db scripts is not compatible with PHP7+ and is now deprecated.

## I. Quick guide

1. Test build

A complete build of a TNRS database is performed by running the master script load_tnrs.php.

For a trial build of an example database (using a small sample of names from the Tropicos
API, USDA Plants, and the Global Compositae Checklist in Darwin Core format) enter your 
server and database information in the global parameters file (global_params.inc) and run 
load_tnrs.php. 

2. Production build

For a production build using one or more of your own taxonomic sources, you will need to
create an import directory for each source (named "import_[sourceName]"), modeled after
the contents of one of the four example import directories provided. Your raw data must
be in the same format as the example data in the data/ directory; you may also need to 
adjust source-specific parameters in the params.inc file for that source. 

We recommend using the example provided in import_dwcExample as the template for all your
sources. The readme file in import_dwcExample provides details on preparing your
source files, setting source-specific parameters (params.inc in each import directory) and
global parameters (global_params.inc). You will need to transform each of your taxonomic 
sources into the modified DwC (Darwin Core) format, and place it in its own import folder,
along with a copy of all the other files in the import_dwcExample directory. How you
wrangle your data into DwC format is up to you (scripting, Excel, whatever), but the 
final product must satisfy the requirements of modified DwC, as described in the readme
file.

3. Connecting the database to your instance of the TNRS

To connect the TNRS to your database, copy the database to MySQL on your TNRS server. 
You can call your TNRS database whatever you want. Next, make sure 'tnrs' is a MySQL user,
and has permission to read the TNRS database. Only SELECT permission is needed. Finally, 
you need to tell the TNRS which database to use. You do this by editing the api config
file (taxamatch-webservice-read-only/api/config.php) as follows:

'db_name'=> 'your_tnrs_database_name'

replacing your_tnrs_database_name with whatever name you gave to the tnrs database.

That's it!

## II. Details

Each major step in this process is performed by files in a single directory. Within each 
directory, one script (with extension .php) calls all the others (extensions .inc). 
The steps are as follows:

1. create_tnrs_core.php (in create_tnrs_core/)
- Creates empty tnrs database
2. import.php (in import_[sourceName]/) 
- Imports raw data for an individual taxonomic source into MySQL and performs initial 
loading to the staging table (nameStaging)
- Steps specific to an individual source are in this directory
3. prepare_staging.php (in prepare_staging/)
- Finishes structuring and populating staging table (nameStaging)
- These operations are universal, not source-specific
4. load_core_db.php (in load_core_db/)
- Normalizes the contents of the staging table to the core database
5. make_genus_family_lookups (in genus_family_lookups/)
- Builds lookup tables of current and historic genus-in-family classifications, based on
GRIN taxonomy website
6. taxamatch_tables.php (in taxamatch_tables/)
- Denormalizes names in core database into lookup tables used by TaxaMatch fuzzy
matching application
7. build_classifications.php (in build_classifications/)
- Builds table 'higherClassification', which classifies all names from all sources 
according to any source for which isHigherClassification=1 (set in params.inc for that
source)

For a build from multiple sources, step 1 is run ONCE. Steps 2-4 are run for EACH 
source. Finally, steps 5-6 are run ONCE.

This entire process is automated by the master script load_tnrs.php, which calls
all the others. Before running this script, you MUST set critical parameters in 
global_params.inc. Also, set source-specific parameters in params.inc in the import 
directory for each source. See instructions in load_tnrs.php, global_params.inc. For 
details on setting source-specific parameters, see the readme file in import_dwcExample/.

An individual source can be refreshed without rebuilding the entire database by loading
source only and setting $replace_db=false (in global_params.inc). This will run 
steps 2-7 above, replacing only names linked uniquely to the source in question. 
For a faster replace, set $replace=false in params.inc for the source being refreshed. 
Only entirely new names from that source will be added. Existing names (and metadata
such as source urls and date of access) will not be changed.

## III. Dependencies

- Custom PHP functions are in subdirectory functions/
- Custom MySQL function strSplit() must be present in your installation of MySQL. This 
function will be automatically installed if not already present. Alternatively, you can
use contents of file strSplit_function.sql to install this function manually
- perl utility dbf_dump is needed to extract downloaded GRIN taxonomy files to plain
text  (step 5). See readme in genus_family_lookups for details.

## IV. Change log

#### Version 3.6.3.

1. Added scripts in directory tropicos_fixes/. These are fixes specific to tropicos. 
Are not part of the pipeline, but should be run separately after completing build of
the database. See readme in tropicos_fixes.

#### Version 3.6: 

1. Changed Tropicos import routine to include three additional fields: NomenclatureStatusID, 
NomenclatureStatusName, Symbol. These fields are returned by Tropicos API, and provide 
additional information regarding nomenclatural status. Values in the three fields are 
equivalent representations of the same value. For names where the Tropicos ComputedAcceptance 
algorithm does not provide a taxonomic opinion (as indicated by NULL value of `acceptance`, 
the value of NomenclatureStatusName is transfered to `acceptance` WHERE NomenclatureStatusName 
IN ('Illegitimate','Invalid'). NomenclatureStatusName='nom. rej.' is translated as 
"Rejected name' and transfered to `acceptance`. 

The goal of transferring these values is to alert the user that the name in question is 
problematic, even if Tropicos does not provide a link to the accepted name.   

Questions?
Brad Boyle
bboyle@email.arizona.edu
