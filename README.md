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

### PTS and UNGDC data
```{r}
#Upload UNGDC data

ungd_files <- readtext("Converted sessions/*", 
                                 docvarsfrom = "filenames", 
                                 dvsep="_", 
                                 docvarnames = c("Country", "Session", "Year"))


ungd_files$doc_id <- str_replace(ungd_files$doc_id , ".txt", "") %>%
   str_replace(. , "_\\d{2}", "")

```


##Creating corpus object(s)
```{r}
ungd_corpus <- corpus(ungd_files, text_field = "text") 

ungdc.2018 <- corpus_subset(ungd_corpus, Year==2018)

ungdc.2006t <- corpus_subset(ungd_corpus, Year>2006)
```


#Pre-processing

Tokenizing corpus.

```{r}
#Tokenization and basic pre-processing
tok <- tokens(ungd_corpus, what = "word",
              remove_punct = TRUE,
              remove_symbols = TRUE,
              remove_numbers = TRUE,
              remove_twitter = TRUE,
              remove_url = TRUE,
              remove_hyphens = FALSE,
              verbose = TRUE, 
              include_docvars = TRUE)
```

```{r}
tok.r <- tokens_tolower(tok)

tok.s <- tokens_select(tok.r, stopwords("english"), selection = "remove", padding = FALSE)

tok.n <- tokens(tok.s, ngrams = c(1:2), include_docvars = TRUE) 
 
```

```{r}
?tokens_remove
```


#Dictionary Analysis and KWIC (25-word context)

## Stage 1 - Human Rights terms

Creating compound tokens from the key terms identified:
 
```{r}

listHHRR <- list( c("Human", "Rights")) 
```
 

```{r}
token.compound <- tokens_compound(tok.r, listHHRR, valuetype = "fixed", concatenator = "_")
```

Creating the dictionary object of Human Rights terms:

```{r}
HHRR_dict <- dictionary(list(Human.Rights=c("Human Rights", "Rights")))
```

## Look up Human Rights term counts

Looking up and counting terms from Human Rights in the DFM

```{r}
?dfm_lookup
```

DFM_LOOKUP --> Apply a dictionary (HHRR) to a dfm by looking up all dfm features for matches in a a set of dictionary values, and replace those features with a count of the dictionary's keys.

```{r}
dfm_1 <- dfm(token.compound)

HHRR_dfm <- dfm_lookup(dfm_1, HHRR_dict, valuetype = "fixed")
```

```{r}
head(HHRR_dfm)
```

Data Frame

```{r}
#creating a data frame --> CHECK IF THIS IS NECESARY IN THIS POINT OF THE PROJECT
Human.Rights <- convert(HHRR_dfm, to = "data.frame")

#converting documentID into two columns - country and year
HHRR_counts <- Human.Rights %>%
  separate(document, c("country", "year"), "_")

#year as numeric
HHRR_counts$year <- as.numeric(HHRR_counts$year)

HHRR_counts <- plyr::rename(HHRR_counts, c("Human.Rights"="HHRR"))
```

```{r}
head(HHRR_counts)
```



## Presentation of results

### Map for 2017 of HHRR

Keeping only country-years with at least one mention of HHRR


```{r}
mentions_HHRR <- subset(HHRR_counts, "human rights"!=0)
```

```{r}
head(mentions_HHRR)
```



Error in joinCountryData2Map --> check the packages

```{r}

map <- joinCountryData2Map(subset(mentions_HHRR, year==2017), joinCode="ISO3", nameJoinColumn="country")

new_world <- subset(map, continent != "Antarctica")

pdf("worldmap_2017_health.pdf", width = 7, height = 3)

par(mai=c(0,0,0.2,0),xaxs="i",yaxs="i")

mapParams <- mapCountryData(new_world, nameColumnToPlot="HHRR", 
                            mapTitle="2017 UN General Debate: HHRR", 
                            catMethod = "categorical", 
                            colourPalette = "heat", 
                            oceanCol = "lightblue", 
                            missingCountryCol = "white", 
                            addLegend="FALSE")

do.call( addMapLegendBoxes, c(mapParams,title="Number of mentions", x = "left", cex=0.5))

dev.off()
```

## Time series of total counts plot

### Total counts

```{r}
?country2Region
```


```{r}
# calculating the total number of mentions by year
sum <- summarise(group_by(country2Region(count), year), count = sum(Human.Rights), mean = mean(Human.Rights))

ggplot(sum, aes(x=year)) +
  theme_bw() +
  geom_line(aes(y= HHRR_count), colour = "blue", alpha = 0.9) +
  ylab("Total number of references per UNGD session") + xlab("Year") + 
#  scale_y_continuous(limits=c(0, 140), breaks = c(1, 50, 100, 134)) +
  scale_x_continuous(limits=c(1970, 2017), breaks = c(1970, 1988, 2000, 2009,2014, 2017)) +
   annotate("text", x = 1993, y = 250, label = "Human Rights", colour = "blue")

```
