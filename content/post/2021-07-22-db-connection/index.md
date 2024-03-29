---
title: MS Data Base Connection with R
author: Rodrigo
date: '2021-07-22'
slug: db-connection
categories: []
tags:
  - Microsoft Access
  - odbc
  - DBI
subtitle: 'Connecting and importing data from Microsoft Access Data Bases with R'
summary: 'Microsoft Access Data Bases may contain several tables to be extracted. Using the odbc and DBI packages we are able to import all desired tables to the R environment with just a few steps.'
authors: []
lastmod: '2021-07-22T10:30:57-03:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

Microsoft Access Data Bases may contain several tables of interest. Using the `odbc` and `DBI` packages we are able to import all desired tables to the R environment with just a few steps.

<br>

# Connecting to multiple mdb files from R

First, let´s list all the files we want to work with:


```r
  # Full file path to Access DB
    
      files_path <- list.files("./mdb", full.names = TRUE, recursive  = TRUE)
```

<br>

Opening a 'mdb' or 'accdb' file creates a temporary 'Microsoft Access Record-Locking Information (laccdb)' file in the same folder. So, before moving on, you must close all 'laccdb' files to allow a remote connection from R.


```r
  # Install and load packages 

    library("odbc", quietly = FALSE) # ODBC for MS Access files
                                  
    library("DBI",  quietly = FALSE) # read tables from connections

  # Pass MS Access file path to connection string in loop

    connections <- as.list(files_path)
      
       for (i in seq(files_path)) {
          
          connections[[i]] <- dbConnect(drv = odbc(),
                                        .connection_string = paste0("Driver={Microsoft Access Driver (*.mdb, *.accdb)};DBQ=",
                                                                    files_path[i], ";"))
       }
```

The `.connection_string` parameter indicates both 'mdb' and 'accdb' extensions. 

<br>

We should retrieve connections names. Later on, we might use it to identify the tables.   


```r
  # Retrieve connections names
    
    conn_ls <- list()
    
    for (i in seq(connections)) {
      
      conn_ls[i] <- connections[i][[1]]@info$dbname
      
    }
```

<br>

If we wish to select some of the tables from multiple data sets, we can list the names in the following way.


```r
    # List tables

      all_tables <- lapply(connections, dbListTables)
```

<br>

Let´s create a vector of names to subset from the list.


```r
    slct_tables <- c("Table 1", "Table 2", "Table 3")
```

<br>

Now, we can read the selected tables to a list, assigning names based on connections and original tables names. We access each connection, and check for the existence of any of the tables names using `dbExistsTable()`. If the table exists in that connection the function uses `assign()` to determine the name and valueof the table. It uses the `conn_ls`object created before, so that we can identify from which MSdb it came from. The last lines simply print a note so that we can identify the function progress. 


```r
    # Read tables to list
        
      for (i in seq(connections)) {
            
        for (j in seq(slct_tables)) {
          
          if(dbExistsTable(connections[[i]], slct_tables[j]) == FALSE) next
              
              assign(x = paste0(slct_tables[j], "_", conn_ls[[i]]),
                     value = dbReadTable(connections[[i]], slct_tables[j]))
              
              print(paste0(" extraction for ", "'", slct_tables[j], "'", " table",
                           " in connection nº ", i))
            }
          }
```

<br>

Before moving on your code, close the connections, which will remove record locking (laccdb) files. 


```r
    # Disconnect from databases

      lapply(connections, dbDisconnect)
```

