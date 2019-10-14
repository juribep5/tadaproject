# tadaproject
Human Rights in the UNGA

Using the code of the  The Lancet Countdown report: Nick Watts et al. (2018). "The 2018 report of the Lancet Countdown on health and climate change: shaping the health of nations for centuries to come." The Lancet, 392(10163): 2479-2514. With the UNGA Corpus.

#Data

Loading data from the UNGDC data


```{r message=FALSE}
#Loading packages and data
library(readtext)
library(quanteda)
library(dplyr)
library(tidyr)
library(stringr)
library(ggplot2)
library(rworldmap)
library(RColorBrewer)
library(haven)
library(readxl)
```

```{r}

ungd_files <- readtext("Converted sessions/*", 
                                 docvarsfrom = "filenames", 
                                 dvsep="_", 
                                 docvarnames = c("Country", "Session", "Year"))


ungd_files$doc_id <- str_replace(ungd_files$doc_id , ".txt", "") %>%
   str_replace(. , "_\\d{2}", "")
```

