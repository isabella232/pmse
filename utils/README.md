# Benchmarking tools

## What are these files?

**path_configuration.txt** - file contains full paths for root directories of YCSB and MongoDB

**run_suite.py** - will generate bash file testplan.sh with commands to run YCSB in various configurations

**run_workload.sh** - bash file which is called from testplan.sh generated by run_suite.py

**test_suite.txt** - set of user-defined suites, file is created by user

**testplan.sh** - script executing commands to achieve goals defined in script above

**results/configuration.json** - recently read configuration from test_suite.txt stored as JSON file

## Before you start
-	Build PMSE with MongoDB
-	Install Yahoo! Cloud Solution Benchmarking
-	Set absolute paths to MongoDB and YCSB in path_configuration.txt
-	Write benchmark scenarios to test_suite.txt

## Start benchmarks
When you write some benchmark suites and have paths written to path_configuration.txt you should generate testplan.sh using run_suite.py. Testplan.sh will run YCSB many times according to test suites and generate files in results directory. Each directory is named and contains results related to specific suite. 
If everything is configured correctly to start benchmark just type: 
```
python run_suite.py
./testplan.sh
```

## Parsing a results
After this step you get CSV files in proper directories and will printed to the screen.
Python parsing script does not take any argument: 
```
python parser.py
```

## Benchmark suite layout

In the **test_suite.txt** you can describe many benchmark scenarios. Every suite is described with some specific keywords explained below. Also you can comment some lines using ‘**#**’ symbol:

Keyword | Meaning
--------|--------
SUITE [text] | beginning of benchmark suite with given name, 
ENDSUITE | end of suite
THREADS [list of numbers] | different amount of threads for each iteration
LOAD | if keyword is present, script will run YCSB client to fill database, otherwise YCSB will start benchmarking
JOURNALING <enabled/disabled> | when enabled set YCSB operation to have WriteConcern set to "Journaled" 
RECORDS [number] | number of records in DB when executing 'run' phase or number of records to be inserted during 'load'
OPERATIONS [number] | number of operation what will be performed
READ_PROPORTION <floating point number> | proportion of reads
INSERT_PROPORTION <floating point number> | proportion of inserts
UPDATE_PROPORTION <floating point number> | proportion of updates
YCSB_NUMA [number] | NUMA node for YCSB instance
DROP_BEFORE | drop collection before suite start (helpful in context of running insert cases with different number of threads)
CREATE_AFTER_DROP | create collection after collection drop 


## Examples
### Only inserts
```
SUITE only_inserts
THREADS 48
JOURNALING disabled
RECORDS 1000000
OPERATIONS 1000000
INSERT_PROPORTION 1.0
READ_PROPORTION 0.0
UPDATE_PROPORTION 0.0
YCSB_NUMA 1
DROP_BEFORE
CREATE_AFTER_DROP
ENDSUITE
```
### Load database at the beginning then perform 100% read
```
SUITE load_phase
LOAD
THREADS 48
JOURNALING disabled
RECORDS 1000000
OPERATIONS 1000000
INSERT_PROPORTION 1.0
READ_PROPORTION 0.0
UPDATE_PROPORTION 0.0
YCSB_NUMA 1
DROP_BEFORE
CREATE_AFTER_DROP
ENDSUITE
# Suite below will start three times with different thread configuration: 16 32 48
SUITE run_phase
THREADS 16 32 48
JOURNALING disabled
RECORDS 1000000
OPERATIONS 1000000
YCSB_NUMA 1
INSERT_PROPORTION 0.0
READ_PROPORTION 1.0
UPDATE_PROPORTION 0.0
ENDSUITE
```

## Authors
* [Krzysztof Filipek](https://github.com/KFilipek)
