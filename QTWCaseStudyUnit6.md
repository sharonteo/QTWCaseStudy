# Real-Time Location System Case Study
Jean Jeacha, Manjula Kottegoda, Sharon Teo, Jessica Wheeler  
10/7/2017  



#Introduction
In this case study we will be using a statistical method known as the k-nearest neighbor to estimate the location of a device from the strength of the signal detected between the device and serveral access points. Our dataset contains training data where the signal is measured to several access points from known positions throughout the building. When we get new observations from an unknown location, we find the observation from our training data that is closest to this new observation. This method allows us to predict the position of the new observation.

The variables in the dataset are t for timestamp in milliseconds, id for MAC address of scanning device, pos for the physical coordinate of the scanning device, degree for orientation of the user carrying the scanning device in degrees and finally, MAC for the MAC address of a corresponding peer.

#Raw Data Processing

Read in the data as a string character vector called 'txt'. Then search for lines that start with a hashtag

```r
options(digits = 2)
setwd("~/Documents/QTW")
txt = readLines("offline.final.trace.txt")

sum(substr(txt, 1, 1) == "#")
```

```
## [1] 5312
```


```r
length(txt)
```

```
## [1] 151392
```

There are 5,312 comment lines that start with a hashtag. Next we look at the number of lines in the file. There are 151,392 lines in the file. According to the documentation we expect there to be 146,080 lines. The defference between 151,392 and 146,080 is 5,312, which is what we expected


Next, we are going to manipulate the data. Notice how the main data elements are seperated by a semicolon.  Let's see how the semicolon splits the fourth line, which is the first line that is not a comment

```r
strsplit(txt[4], ";")[[1]]
```

```
##  [1] "t=1139643118358"                   
##  [2] "id=00:02:2D:21:0F:33"              
##  [3] "pos=0.0,0.0,0.0"                   
##  [4] "degree=0.0"                        
##  [5] "00:14:bf:b1:97:8a=-38,2437000000,3"
##  [6] "00:14:bf:b1:97:90=-56,2427000000,3"
##  [7] "00:0f:a3:39:e1:c0=-53,2462000000,3"
##  [8] "00:14:bf:b1:97:8d=-65,2442000000,3"
##  [9] "00:14:bf:b1:97:81=-65,2422000000,3"
## [10] "00:14:bf:3b:c7:c6=-66,2432000000,3"
## [11] "00:0f:a3:39:dd:cd=-75,2412000000,3"
## [12] "00:0f:a3:39:e0:4b=-78,2462000000,3"
## [13] "00:0f:a3:39:e2:10=-87,2437000000,3"
## [14] "02:64:fb:68:52:e6=-88,2447000000,1"
## [15] "02:00:42:55:31:00=-84,2457000000,1"
```
We can also split the character vector by a ';', '=' or ',' character and define the vector as 'tokens'. The first 10 elements of tokens give information about the hand-held device:


```r
tokens = strsplit(txt[4], "[;=,]")[[1]]
tokens
```

```
##  [1] "t"                 "1139643118358"     "id"               
##  [4] "00:02:2D:21:0F:33" "pos"               "0.0"              
##  [7] "0.0"               "0.0"               "degree"           
## [10] "0.0"               "00:14:bf:b1:97:8a" "-38"              
## [13] "2437000000"        "3"                 "00:14:bf:b1:97:90"
## [16] "-56"               "2427000000"        "3"                
## [19] "00:0f:a3:39:e1:c0" "-53"               "2462000000"       
## [22] "3"                 "00:14:bf:b1:97:8d" "-65"              
## [25] "2442000000"        "3"                 "00:14:bf:b1:97:81"
## [28] "-65"               "2422000000"        "3"                
## [31] "00:14:bf:3b:c7:c6" "-66"               "2432000000"       
## [34] "3"                 "00:0f:a3:39:dd:cd" "-75"              
## [37] "2412000000"        "3"                 "00:0f:a3:39:e0:4b"
## [40] "-78"               "2462000000"        "3"                
## [43] "00:0f:a3:39:e2:10" "-87"               "2437000000"       
## [46] "3"                 "02:64:fb:68:52:e6" "-88"              
## [49] "2447000000"        "1"                 "02:00:42:55:31:00"
## [52] "-84"               "2457000000"        "1"
```

```r
tokens[1:10]
```

```
##  [1] "t"                 "1139643118358"     "id"               
##  [4] "00:02:2D:21:0F:33" "pos"               "0.0"              
##  [7] "0.0"               "0.0"               "degree"           
## [10] "0.0"
```

Then we can extract the values for the 2nd, 4th, 6-8th and 10th lines of the variables time, id, position and x,y,z orientation.


```r
tokens[c(2, 4, 6:8, 10)]
```

```
## [1] "1139643118358"     "00:02:2D:21:0F:33" "0.0"              
## [4] "0.0"               "0.0"               "0.0"
```
The remainng values in the split vector are the recorded signals within the observation

```r
tokens[ - ( 1:10 ) ]
```

```
##  [1] "00:14:bf:b1:97:8a" "-38"               "2437000000"       
##  [4] "3"                 "00:14:bf:b1:97:90" "-56"              
##  [7] "2427000000"        "3"                 "00:0f:a3:39:e1:c0"
## [10] "-53"               "2462000000"        "3"                
## [13] "00:14:bf:b1:97:8d" "-65"               "2442000000"       
## [16] "3"                 "00:14:bf:b1:97:81" "-65"              
## [19] "2422000000"        "3"                 "00:14:bf:3b:c7:c6"
## [22] "-66"               "2432000000"        "3"                
## [25] "00:0f:a3:39:dd:cd" "-75"               "2412000000"       
## [28] "3"                 "00:0f:a3:39:e0:4b" "-78"              
## [31] "2462000000"        "3"                 "00:0f:a3:39:e2:10"
## [34] "-87"               "2437000000"        "3"                
## [37] "02:64:fb:68:52:e6" "-88"               "2447000000"       
## [40] "1"                 "02:00:42:55:31:00" "-84"              
## [43] "2457000000"        "1"
```
The results above show a 4-column data frame that shows the MAC address, signal, channel, and device type. We can now build a 4-column matrix called "tmp" and then bind these columns with the values from the first 10 entries using the function cbind

```r
tmp = matrix(tokens[ - (1:10) ], ncol = 4, byrow = TRUE)
mat = cbind(matrix(tokens[c(2, 4, 6:8, 10)], nrow = nrow(tmp),
                   ncol = 6, byrow = TRUE), tmp)
```

```r
tmp
```

```
##       [,1]                [,2]  [,3]         [,4]
##  [1,] "00:14:bf:b1:97:8a" "-38" "2437000000" "3" 
##  [2,] "00:14:bf:b1:97:90" "-56" "2427000000" "3" 
##  [3,] "00:0f:a3:39:e1:c0" "-53" "2462000000" "3" 
##  [4,] "00:14:bf:b1:97:8d" "-65" "2442000000" "3" 
##  [5,] "00:14:bf:b1:97:81" "-65" "2422000000" "3" 
##  [6,] "00:14:bf:3b:c7:c6" "-66" "2432000000" "3" 
##  [7,] "00:0f:a3:39:dd:cd" "-75" "2412000000" "3" 
##  [8,] "00:0f:a3:39:e0:4b" "-78" "2462000000" "3" 
##  [9,] "00:0f:a3:39:e2:10" "-87" "2437000000" "3" 
## [10,] "02:64:fb:68:52:e6" "-88" "2447000000" "1" 
## [11,] "02:00:42:55:31:00" "-84" "2457000000" "1"
```
The information about the device, ie time, id, position and x,y,z orientation is now combined with the recorded signals, ie MAC address, signal, channel and device type

```r
mat
```

```
##       [,1]            [,2]                [,3]  [,4]  [,5]  [,6] 
##  [1,] "1139643118358" "00:02:2D:21:0F:33" "0.0" "0.0" "0.0" "0.0"
##  [2,] "1139643118358" "00:02:2D:21:0F:33" "0.0" "0.0" "0.0" "0.0"
##  [3,] "1139643118358" "00:02:2D:21:0F:33" "0.0" "0.0" "0.0" "0.0"
##  [4,] "1139643118358" "00:02:2D:21:0F:33" "0.0" "0.0" "0.0" "0.0"
##  [5,] "1139643118358" "00:02:2D:21:0F:33" "0.0" "0.0" "0.0" "0.0"
##  [6,] "1139643118358" "00:02:2D:21:0F:33" "0.0" "0.0" "0.0" "0.0"
##  [7,] "1139643118358" "00:02:2D:21:0F:33" "0.0" "0.0" "0.0" "0.0"
##  [8,] "1139643118358" "00:02:2D:21:0F:33" "0.0" "0.0" "0.0" "0.0"
##  [9,] "1139643118358" "00:02:2D:21:0F:33" "0.0" "0.0" "0.0" "0.0"
## [10,] "1139643118358" "00:02:2D:21:0F:33" "0.0" "0.0" "0.0" "0.0"
## [11,] "1139643118358" "00:02:2D:21:0F:33" "0.0" "0.0" "0.0" "0.0"
##       [,7]                [,8]  [,9]         [,10]
##  [1,] "00:14:bf:b1:97:8a" "-38" "2437000000" "3"  
##  [2,] "00:14:bf:b1:97:90" "-56" "2427000000" "3"  
##  [3,] "00:0f:a3:39:e1:c0" "-53" "2462000000" "3"  
##  [4,] "00:14:bf:b1:97:8d" "-65" "2442000000" "3"  
##  [5,] "00:14:bf:b1:97:81" "-65" "2422000000" "3"  
##  [6,] "00:14:bf:3b:c7:c6" "-66" "2432000000" "3"  
##  [7,] "00:0f:a3:39:dd:cd" "-75" "2412000000" "3"  
##  [8,] "00:0f:a3:39:e0:4b" "-78" "2462000000" "3"  
##  [9,] "00:0f:a3:39:e2:10" "-87" "2437000000" "3"  
## [10,] "02:64:fb:68:52:e6" "-88" "2447000000" "1"  
## [11,] "02:00:42:55:31:00" "-84" "2457000000" "1"
```
There should be 11 rows, one for each MAC address and 10 columns where the first 6 columns represent the information for the handheld device and the last 4 columns representing the recorded signals 

```r
dim(mat)
```

```
## [1] 11 10
```
Now we can create a function called "processLine" so we can repeat this operation for each row

```r
processLine = function(x) {
  tokens = strsplit(x, "[;=,]")[[1]]
  tmp = matrix(tokens[ - (1:10) ], ncol = 4, byrow = TRUE)
  cbind(matrix(tokens[c(2, 4, 6:8, 10)], nrow = nrow(tmp),
               ncol = 6, byrow = TRUE), tmp)
}
```
Let's apply our function to several lines using lapply. We start at the 4th line because the first three lines are comments and end on the 20th line. The result is a list of 17 matrices

```r
tmp = lapply(txt[4:20], processLine)
```
We use sapply to determine how many signals were detected at each point

```r
sapply(tmp, nrow)
```

```
##  [1] 11 10 10 11  9 10  9  9 10 11 11  9  9  9  8 10 14
```
To stack the matrices together we use the function do.call()


```r
offline = as.data.frame(do.call("rbind", tmp))
dim(offline)
```

```
## [1] 170  10
```

```r
lines = txt[ substr(txt, 1, 1) != "#" ]
tmp = lapply(lines, processLine) # This line generates the error
```

```
## Warning in matrix(tokens[c(2, 4, 6:8, 10)], nrow = nrow(tmp), ncol = 6, :
## data length exceeds size of matrix

## Warning in matrix(tokens[c(2, 4, 6:8, 10)], nrow = nrow(tmp), ncol = 6, :
## data length exceeds size of matrix

## Warning in matrix(tokens[c(2, 4, 6:8, 10)], nrow = nrow(tmp), ncol = 6, :
## data length exceeds size of matrix

## Warning in matrix(tokens[c(2, 4, 6:8, 10)], nrow = nrow(tmp), ncol = 6, :
## data length exceeds size of matrix

## Warning in matrix(tokens[c(2, 4, 6:8, 10)], nrow = nrow(tmp), ncol = 6, :
## data length exceeds size of matrix

## Warning in matrix(tokens[c(2, 4, 6:8, 10)], nrow = nrow(tmp), ncol = 6, :
## data length exceeds size of matrix
```
We get a warning message. We can trace the error by setting the option to handle errors

```r
options(error = recover, warn = 2)
```
The error is due to a missing signal value. This observation can be deleted to fix the error. We change our function to return NULL if the tokens vector only has 10 elements. Here is our revised function 


```r
processLine = function(x) {
  tokens = strsplit(x, "[;=,]")[[1]]
  
  if (length(tokens) == 10) 
    return(NULL)
 
  tmp = matrix(tokens[ - (1:10) ], , 4, byrow = TRUE)
  cbind(matrix(tokens[c(2, 4, 6:8, 10)], nrow(tmp), 6, 
               byrow = TRUE), tmp)
}

tmp = lapply(lines, processLine)
offline = as.data.frame(do.call("rbind", tmp), 
                        stringsAsFactors = FALSE)

dim(offline)
```

```
## [1] 1181628      10
```
This time we do not get an error message and we have over one million rows and 10 columns

# Cleaning the Data




