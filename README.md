# epanetReader

epanetReader is an R package for reading water network simulation data in
Epanet's .inp and .rpt formats into R.  Some basic summary information and
plots are also provided.   

Epanet is a highly popular tool for water network simulation. It can be difficult
to access network information for subsequent analysis and visualization. This
is a real strength of R however, and there many tools already existing in R to
support analysis and visualization.  

Reading an .inp or .rpt file (from Epanet or Epanet-msx) creates an s3 object which can be used with the
usual generic methods.  

The package is also useful in conjunction with the ggplot2 and animation
packages. 

## Installation 

* the latest released version: install.packages("epanetReader")
* the development version: devtools::install_github("bradleyjeck/epanetReader")   

## Getting Started 

### Network files 

Read network information from an .inp file with a similar syntax as the popular read.table or read.csv functions.  

```R
>n1 <- read.inp("Net1.inp") 
```

Retrieve summary information about the network.
```R 
>summary(n1)
$Title
[1] "EPANET Example Network 1"                                  
[2] "A simple example of modeling chlorine decay. Both bulk and"
[3] "wall reactions are included."                              

$entryCounts
            Number
Junctions        9
Tanks            1
Reservoirs       1
Pipes           12
Pumps            1
Coordinates     11

attr(,"class")
[1] "summary.epanet.inp"
```

A basic network plot is also available
```R
>plot(n1)
```
![Net 1 plot](https://github.com/bradleyjeck/epanetReader/blob/master/img/Net1inp.png)

The read.inp function returns an object with structure similar to the .inp file
itself. A section in the .inp file corresponds to a named entry in the list.
These entries are accessed using the $ syntax of R.
```R
> names(n1)
 [1] "Title"       "Junctions"   "Tanks"       "Reservoirs"  "Pipes"      
 [6] "Pumps"       "Valves"      "Patterns"    "Curves"      "Energy"     
[11] "Times"       "Options"     "Coordinates"
```

Sections of the .inp file are stored as a data.frame or character vector. For
example, the junction table is stored as a data.frame and retrieved as follows.
In this case patterns were not specified in the junction table and so are
marked NA.
```R
> n1$Junctions
  ID Elevation Demand Pattern
1 10       710      0      NA
2 11       710    150      NA
3 12       700    150      NA
4 13       695    100      NA
5 21       700    150      NA
6 22       695    200      NA
7 23       690    150      NA
8 31       700    100      NA
9 32       710    100      NA
```

A summary of the junction table shows that Net1.inp has nine junctions with
elevations ranging from 690 to 710 and demands ranging from 0 to 200. Note that
the node ID is stored as a character rather than an integer or factor.  
```R
> summary(n1$Junctions)
      ID              Elevation         Demand      Pattern       
 Length:9           Min.   :690.0   Min.   :  0.0   Mode:logical  
 Class :character   1st Qu.:695.0   1st Qu.:100.0   NA's:9        
 Mode  :character   Median :700.0   Median :150.0                 
                    Mean   :701.1   Mean   :122.2                 
                    3rd Qu.:710.0   3rd Qu.:150.0                 
                    Max.   :710.0   Max.   :200.0                 
```

### Epanet Simulation Results 

Results of the network simulation specified in Net.inp may be stored in
Net1.rpt by running Epanet from the command line. Note that the report section
of the .inp file should contain the following lines in order to generate output
readable by this package.   

>[REPORT]   
>Page 0  
>Links All   
>Nodes All    

On windows, calling the epanet executable epanet2d runs the simulation. 
```
>epanet2d Net1.inp Net1.rpt 

... EPANET Version 2.0

  o Retrieving network data
  o Computing hydraulics 
  o Computing water quality
  o Writing output report to Net1.rpt

... EPANET completed.
```  

The .rpt file generated by Epanet may be read into R using read.rpt().  The
simulation is summarized over junctions, tanks and pipes. 
```R
>n1r <- read.rpt("Net1.rpt") 
>summary(n1r)
Contains node results for  25 time steps 

Summary of Junction Results: 
     Demand         Pressure        Chlorine     
 Min.   :  0.0   Min.   :106.8   Min.   :0.1500  
 1st Qu.: 80.0   1st Qu.:116.1   1st Qu.:0.3500  
 Median :120.0   Median :119.8   Median :0.5100  
 Mean   :122.2   Mean   :119.6   Mean   :0.5434  
 3rd Qu.:160.0   3rd Qu.:123.0   3rd Qu.:0.7400  
 Max.   :320.0   Max.   :133.9   Max.   :1.0000  

Summary of Tank Results:
     Demand             Pressure        Chlorine    
 Min.   :-1100.000   Min.   :48.22   Min.   :0.590  
 1st Qu.: -660.000   1st Qu.:52.00   1st Qu.:0.660  
 Median :  258.000   Median :55.52   Median :0.750  
 Mean   :   -5.741   Mean   :54.86   Mean   :0.764  
 3rd Qu.:  505.380   3rd Qu.:57.54   3rd Qu.:0.850  
 Max.   : 1029.420   Max.   :60.04   Max.   :1.000  

Contains link results for  25 time steps 

Summary of Pipe Results:
      Flow             Velocity         Headloss    
 Min.   :-1029.42   Min.   :0.0000   Min.   :0.000  
 1st Qu.:   41.37   1st Qu.:0.3475   1st Qu.:0.110  
 Median :  113.08   Median :0.5700   Median :0.300  
 Mean   :  245.35   Mean   :0.8070   Mean   :0.644  
 3rd Qu.:  237.23   3rd Qu.:1.0075   3rd Qu.:0.755  
 Max.   : 1909.42   Max.   :2.7300   Max.   :3.210  
```  


The default plot of simulation results is a map for time period 00:00:00. Note that the
object created from the .inp file is a required argument to make the plot. 
```R
plot( n1r, n1)
```
![Net 1 plot](https://github.com/bradleyjeck/epanetReader/blob/master/img/Net1rpt.png)

In contrast to the treatment of .inp files described above, data from .rpt
files is stored using a slightly different structure than the .rpt file.  The
function returns an object (list) with a data.frame for node results and
data.frame for link results.  These two data frames contain results from all
the time periods.  This storage choice was made to facilitate time series plots. 

Entries in the epanet.rpt object (list) created by read.rpt() are found using the names() function. 
```R
> names(n1r)
[1] "nodeResults" "linkResults"
```

Results for a chosen time period can be retrieved using the subset function.
```R
> subset(n1r$nodeResults, Timestamp == "0:00:00")
     ID   Demand    Head Pressure Chlorine      note Timestamp timeInSeconds  nodeType
1    10     0.00 1004.35   127.54      0.5             0:00:00             0  Junction
2    11   150.00  985.23   119.26      0.5             0:00:00             0  Junction
3    12   150.00  970.07   117.02      0.5             0:00:00             0  Junction
4    13   100.00  968.87   118.67      0.5             0:00:00             0  Junction
5    21   150.00  971.55   117.66      0.5             0:00:00             0  Junction
6    22   200.00  969.08   118.76      0.5             0:00:00             0  Junction
7    23   150.00  968.65   120.74      0.5             0:00:00             0  Junction
8    31   100.00  967.39   115.86      0.5             0:00:00             0  Junction
9    32   100.00  965.69   110.79      0.5             0:00:00             0  Junction
10    9 -1866.18  800.00     0.00      1.0 Reservoir   0:00:00             0 Reservoir
11    2   766.18  970.00    52.00      1.0      Tank   0:00:00             0      Tank
```
A comparison with the corresponding entry of the .rpt file, shown below for
reference, shows that four columns have been added to the table. These pieces of extra 
info make visualizing the results easier. 

``` 
  Node Results at 0:00:00 hrs:
  --------------------------------------------------------
                     Demand      Head  Pressure  Chlorine
  Node                  gpm        ft       psi      mg/L
  --------------------------------------------------------
  10                   0.00   1004.35    127.54      0.50
  11                 150.00    985.23    119.26      0.50
  12                 150.00    970.07    117.02      0.50
  13                 100.00    968.87    118.67      0.50
  21                 150.00    971.55    117.66      0.50
  22                 200.00    969.08    118.76      0.50
  23                 150.00    968.65    120.74      0.50
  31                 100.00    967.39    115.86      0.50
  32                 100.00    965.69    110.79      0.50
  9                -1866.18    800.00      0.00      1.00  Reservoir
  2                  766.18    970.00     52.00      1.00  Tank
```

### Epanet-msx simulation results 

Results of a multi-species simulation by Epanet-msx can be read as well. 

The read.msxrpt() function creates an s3 object of class epanetmsx.rpt.
Similar to the approach above, there is a data frame for node results
and link results. 

```R
> x <- read.msxrpt("5deg.msxrpt")
> names(x)
[1] "Title"       "nodeResults" "linkResults"
> summary(x)
  Water Age + temp + Cl + TTHM + HAA6  
 node results
      AGE             TEMP              CL              TTHM       
 Min.   :    0   Min.   : 5.000   Min.   :0.0000   Min.   :0.0000  
 1st Qu.: 3233   1st Qu.: 5.302   1st Qu.:0.7282   1st Qu.:0.6792  
 Median : 7508   Median : 5.652   Median :0.8283   Median :0.6850  
 Mean   : 8446   Mean   : 6.298   Mean   :0.7104   Mean   :0.5808  
 3rd Qu.:12547   3rd Qu.: 6.023   3rd Qu.:0.9209   3rd Qu.:0.6915  
 Max.   :27294   Max.   :10.000   Max.   :1.0000   Max.   :0.7703  
      HAA6       
 Min.   :0.0000  
 1st Qu.:0.7337  
 Median :0.7387  
 Mean   :0.6256  
 3rd Qu.:0.7443  
 Max.   :0.8114  
 Link results
NULL
```



## Usage with other packages  

### ggplot2
The ggplot2 package makes it easy to create complex graphics by allowing users
to describe the plot in terms of the data. Continuing the Net1 example Here we
plot chlorine concentration over time at each node in the network.  

```R
library(ggplot2)
qplot( data= n1r$nodeResults,  
       x = timeInSeconds/3600, y = Chlorine, 
       facets = ~ID, xlab = "Hour")  
```

![Net 1 Cl plot](https://github.com/bradleyjeck/epanetReader/blob/master/img/Net1cl.png)

### Animation
The animation package is useful for creating a video from successive plots. 

```R
# example with animation package 
library(animation)

#unique time stamps
ts <- unique((n1r$nodeResults$Timestamp))
imax <- length(ts)

# generate animation of plots at each time step
saveHTML(
  for( i in 1:imax){
    plot(n1r, n1, Timestep = ts[i]) 
  }
)
```
## References 

Rossman, L. A. (2000) [Epanet 2 users manual](http://nepis.epa.gov/Adobe/PDF/P1007WWU.pdf).
US EPA, Cincinnati, Ohio.


