---
title: "Data Analysis and Visualisation of Crime in Chicago"
author: 'Student ID: 201081646'
output:
  pdf_document:
    fig_caption: yes
    fig_crop: yes
    fig_height: 3.2
    fig_width: 5.2
    keep_tex: yes
    number_sections: yes
    toc_depth: 2
  html_document:
    fig_caption: yes
    theme: journal
    toc: yes
    toc_depth: 2
header-includes:
 \usepackage{float}
 \floatplacement{figure}{H}
---
```{r, include=FALSE}
knitr::opts_chunk$set(fig.pos = 'H')
```

# Introduction

This report is the first assessment of the **MATH5741M Statistical Theory and Methods** module. Its aims are to summarise a crime dataset from the city of Chicago and answer the following research questions:

- How has crime evolved over time in the city of Chicago? 

- What time of day does most crime occur?

- In which locations of the city is crime more likely to happen?

- Which districts are more potentially dangerous?

# Data and methods

The analysis was done using a sample of the [crime dataset from the Chicago Police Department](https://data.cityofchicago.org/Public-Safety/Crimes-2001-to-present/ijzp-q8t2) which contained all the crime incidents that occurred in the city of Chicago from 2001 to October 2016.

To perform the analysis, we first prepared the data. For this, we created, transformed and cleaned the variables we considered relevant. Then, we plotted these variables through line, bar graphs and heat-maps. Finally, the interpretation of these visualisations led us the answer our questions. 

The whole process was done using the software `R` and `Rmarkdown`, and it is reproducible based on code. 

# Results

## Data preparation

First, we activated the libraries needed to set up the project.

```{r, echo=TRUE, message=FALSE, warning=FALSE}
# Activate libraries
library(ggplot2)
library(lubridate)
library(zoo)
library(dplyr)
```

Second, we loaded the data into the `R` environment.

```{r eval=TRUE, echo=TRUE}
# Read csv in R
dd=read.csv("http://www1.maths.leeds.ac.uk/~charles/math5741/crime.csv",header=T)
```

\pagebreak

Third, we created two new variables,`Count` and `Hour`, and made some transformations in the `Date` variable format in order to make `R` understand it right. 

```{r, echo=TRUE, message=FALSE, warning=FALSE}
# Create a variable count with value 1
dd$Count <- 1
# Convert Date from factor to date
dd$Date <- mdy_hms(dd$Date)
# Extract hour from Date
dd$Hour <- substring(dd$Date, 12,13)
# Drop time from Date
dd$Date <- as.Date(dd$Date, format="%m/%d/%Y")
```

Fourth, we grouped the existent categories in the variables `Primary.Type` and `Location.Description` in larger ones, and called them `Type_grouped` and `Location_grouped` respectively.

```{r, include=FALSE}
# Group Primary.Type categories
dd$Type_grouped[dd$Primary.Type == "THEFT" | dd$Primary.Type == "MOTOR VEHICLE THEFT" ] <- "Theft"

dd$Type_grouped[dd$Primary.Type == "BATTERY"] <- "Batery"

dd$Type_grouped[dd$Primary.Type == "CRIMINAL DAMAGE"] <- "Criminal damage"

dd$Type_grouped[dd$Primary.Type == "NARCOTICS" | dd$Primary.Type == "OTHER NARCOTIC VIOLATION"] <- "Narcotics"

dd$Type_grouped[dd$Primary.Type == "ASSAULT"] <- "Assault"

dd$Type_grouped[dd$Primary.Type == "BURGLARY"] <- "Burglary"

dd$Type_grouped[dd$Primary.Type == "ROBBERY" ] <- "Robery"

dd$Type_grouped[dd$Primary.Type == "ARSON" | dd$Primary.Type == "CONCEALED CARRY LICENSE VIOLATION" | dd$Primary.Type == "CRIMINAL TRESPASS" | dd$Primary.Type == "GAMBLINGS" | dd$Primary.Type == "HUMAN TRAFFICKING" | dd$Primary.Type == "INTERFERENCE WITH PUBLIC OFFICER" | dd$Primary.Type == "INTIMIDATION" | dd$Type == "KIDNAPPING" | dd$Type == "LIQUOR LAW VIOLATION" | dd$Primary.Type == "NON-CRIMINAL" | dd$Primary.Type == "NON - CRIMINAL" | dd$Primary.Type == "OBSCENITY" | dd$Primary.Type == "OFFENSE INVOLVING CHILDREN"| dd$Primary.Type == "PROSTITUTION"| dd$Primary.Type == "PUBLIC INDECENCY"| dd$Primary.Type == "PUBLIC PEACE VIOLATION"| dd$Primary.Type == "STALKING"| dd$Primary.Type == "WEAPONS VIOLATION"| dd$Primary.Type == "HOMICIDE"| dd$Primary.Type == "CRIM SEXUAL ASSAULT" | dd$Primary.Type == "SEX OFFENSE"| dd$Primary.Type == "DECEPTIVE PRACTICE" | dd$Primary.Type == "OTHER OFFENSE"] <- "Others"
```

```{r, include=FALSE}
# Group Location.Description categories
dd$Location_grouped[dd$Location.Description == "\"SCHOOL- PRIVATE- BUILDING\"" | dd$Location.Description ==  "\"SCHOOL- PRIVATE- GROUNDS\"" | dd$Location.Description == "\"SCHOOL- PUBLIC- BUILDING\"" | dd$Location.Description == "\"SCHOOL- PUBLIC- GROUNDS\"" | dd$Location.Description == "COLLEGE/UNIVERSITY GROUNDS" | dd$Location.Description == "COLLEGE/UNIVERSITY RESIDENCE HALL"| dd$Location.Description == "SCHOOL YARD"]<- "School/university"

dd$Location_grouped[dd$Location.Description == "" | dd$Location.Description == "ABANDONED BUILDING"| dd$Location.Description == "AIRCRAFT" | dd$Location.Description == "ANIMAL HOSPITAL" | dd$Location.Description == "ATHLETIC CLUB" | dd$Location.Description == "AUTO" | dd$Location.Description == "BASEMENT"  | dd$Location.Description == "BOAT/WATERCRAFT" | dd$Location.Description == "CHA GROUNDS" | dd$Location.Description == "CHA HALLWAY"  | dd$Location.Description == "CHA HALLWAY/STAIRWELL/ELEVATOR" | dd$Location.Description == "CHURCH" | dd$Location.Description == "CHURCH/SYNAGOGUE/PLACE OF WORSHIP"| dd$Location.Description == "COIN OPERATED MACHINE"| dd$Location.Description == "CONSTRUCTION SITE"| dd$Location.Description == "OTHER"| dd$Location.Description == "CHURCH"| dd$Location.Description == "OTHER RAILROAD PROP / TRAIN DEPOT" | dd$Location.Description =="SEWER" | dd$Location.Description =="STAIRWELL"| dd$Location.Description == "VACANT LOT" | dd$Location.Description =="VACANT LOT/LAND"| dd$Location.Description == "VESTIBULE" | dd$Location.Description =="WOODED AREA" | dd$Location.Description == "CTA STATION" | dd$Location.Description == "CTA BUS STOP"| dd$Location.Description == "CTA TRACKS - RIGHT OF WAY"  | dd$Location.Description == "AIRPORT BUILDING NON-TERMINAL - NON-SECURE AREA" | dd$Location.Description == "AIRPORT BUILDING NON-TERMINAL - SECURE AREA" | dd$Location.Description == "AIRPORT EXTERIOR - NON-SECURE AREA" | dd$Location.Description == "AIRPORT EXTERIOR - SECURE AREA" | dd$Location.Description == "AIRPORT PARKING LOT" | dd$Location.Description ==  "AIRPORT TERMINAL LOWER LEVEL - NON-SECURE AREA" | dd$Location.Description == "AIRPORT TERMINAL LOWER LEVEL - SECURE AREA" | dd$Location.Description == "AIRPORT TERMINAL MEZZANINE - NON-SECURE AREA"| dd$Location.Description == "AIRPORT TERMINAL UPPER LEVEL - NON-SECURE AREA"| dd$Location.Description == "AIRPORT TERMINAL UPPER LEVEL - SECURE AREA"| dd$Location.Description == "AIRPORT TRANSPORTATION SYSTEM (ATS)"| dd$Location.Description == "AIRPORT VENDING ESTABLISHMENT" | dd$Location.Description == "AIRPORT/AIRCRAFT" |dd$Location.Description == "DAY CARE CENTER"| dd$Location.Description == "COMMERCIAL / BUSINESS OFFICE"| dd$Location.Description == "FACTORY" | dd$Location.Description =="FACTORY/MANUFACTURING BUILDING" | dd$Location.Description =="FEDERAL BUILDING"| dd$Location.Description == "FIRE STATION"| dd$Location.Description == "GOVERNMENT BUILDING/PROPERTY"| dd$Location.Description == "HOSPITAL"| dd$Location.Description == "HOSPITAL BUILDING/GROUNDS"| dd$Location.Description == "JAIL / LOCK-UP FACILITY"| dd$Location.Description == "LIBRARY"| dd$Location.Description == "MOVIE HOUSE/THEATER"  | dd$Location.Description =="NURSING HOME/RETIREMENT HOME"| dd$Location.Description == "POOL ROOM" | dd$Location.Description =="SPORTS ARENA/STADIUM"| dd$Location.Description == "WAREHOUSE" | dd$Location.Description == "BANK"| dd$Location.Description == "CREDIT UNION"| dd$Location.Description == "CURRENCY EXCHANGE"| dd$Location.Description == "SAVINGS AND LOAN"]<- "Others"

dd$Location_grouped[dd$Location.Description == "ALLEY" | dd$Location.Description == "BOWLING ALLEY" | dd$Location.Description == "CHA BREEZEWAY" | dd$Location.Description =="HALLWAY"]<- "Alley" 

dd$Location_grouped[dd$Location.Description == "APARTMENT"| dd$Location.Description == "CHA APARTMENT"]<- "Apartment"

dd$Location_grouped[dd$Location.Description == "APPLIANCE STORE" | dd$Location.Description == "BARBERSHOP" | dd$Location.Description == "CAR WASH" | dd$Location.Description == "CLEANING STORE" | dd$Location.Description ==  "CONVENIENCE STORE" | dd$Location.Description =="DEPARTMENT STORE" | dd$Location.Description =="DRUG STORE"| dd$Location.Description == "GARAGE/AUTO REPAIR"| dd$Location.Description == "GAS STATION"| dd$Location.Description == "GAS STATION DRIVE/PROP." | dd$Location.Description =="GROCERY FOOD STORE"| dd$Location.Description == "MEDICAL/DENTAL OFFICE" | dd$Location.Description =="NEWSSTAND"| dd$Location.Description == "OFFICE"| dd$Location.Description == "PAWN SHOP" | dd$Location.Description =="RETAIL STORE"| dd$Location.Description == "SMALL RETAIL STORE"]<- "Store/small business"

dd$Location_grouped[dd$Location.Description == "ATM (AUTOMATIC TELLER MACHINE)" | dd$Location.Description == "BRIDGE"| dd$Location.Description == "DRIVEWAY"| dd$Location.Description == "GANGWAY"| dd$Location.Description == "HIGHWAY/EXPRESSWAY"| dd$Location.Description == "LAKEFRONT/WATERFRONT/RIVERBANK"| dd$Location.Description == "SIDEWALK" | dd$Location.Description == "STREET"]<- "Street"

dd$Location_grouped[dd$Location.Description == "BAR OR TAVERN"| dd$Location.Description == "HOTEL"| dd$Location.Description == "HOTEL/MOTEL"| dd$Location.Description == "RESTAURANT"| dd$Location.Description == "TAVERN"| dd$Location.Description == "TAVERN/LIQUOR STORE"]<- "Restaurant/bar/hotel"

dd$Location_grouped[dd$Location.Description == "CHA PARKING LOT" | dd$Location.Description =="PARK PROPERTY" | dd$Location.Description =="PARKING LOT"| dd$Location.Description == "PARKING LOT/GARAGE(NON.RESID.)" | dd$Location.Description =="POLICE FACILITY/VEH PARKING LOT"]<- "Parking lot"

dd$Location_grouped[dd$Location.Description == "TRAILER" | dd$Location.Description == "TRUCK" | dd$Location.Description == "VEHICLE - DELIVERY TRUCK"| dd$Location.Description ==  "VEHICLE - OTHER RIDE SERVICE"| dd$Location.Description ==  "VEHICLE NON-COMMERCIAL" | dd$Location.Description == "VEHICLE-COMMERCIAL" | dd$Location.Description == "CTA BUS" | dd$Location.Description =="DELIVERY TRUCK" | dd$Location.Description =="TAXICAB" | dd$Location.Description == "CTA TRAIN" | dd$Location.Description == "OTHER COMMERCIAL TRANSPORTATION"]<- "Vehicle" 

dd$Location_grouped[dd$Location.Description == "CTA GARAGE / OTHER PROPERTY" | dd$Location.Description =="DRIVEWAY - RESIDENTIAL" | dd$Location.Description =="GARAGE" | dd$Location.Description =="HOUSE" | dd$Location.Description == "PORCH" | dd$Location.Description =="RESIDENCE" | dd$Location.Description == "RESIDENCE PORCH/HALLWAY"| dd$Location.Description == "RESIDENCE-GARAGE"| dd$Location.Description == "RESIDENTIAL YARD (FRONT/BACK)"| dd$Location.Description == "YARD"]<- "Residence"
```

The next step was to drop all those columns we did not need to answer our research questions.

```{r, echo=TRUE}
# Drop all variables we are not interested in
dd <- dd[, -c(1:2, 4:11, 13:15, 17:18)]
```

Then, we cleaned the dataset of missing values and removed all values from 2016 as this last year was not complete.

```{r, echo=TRUE}
# Remove NAs
dd <- dd[complete.cases(dd),]
# Remove 2016 rows
dd <- dd[!dd$Year > 2015,]
```

Finally, here we can see an abstract of the final dataset ready for exploration.

```{r, echo=TRUE}
# Show first 5 records
head(dd)
```

## Data exploration

### How has crime evolved over time in the city of Chicago?  

To answer the first question we plotted Figures 1 and 2. Figure 1 shows the number of crimes per year from 2001 to 2015. As it can be seen in the graph, crime in the city of Chicago has been decreasing year after year, with a clear and continuous decline. 

```{r fig, fig.cap="Crimes evolution 2001-2015", echo=FALSE, message=FALSE, warning=FALSE, out.extra='',fig.align='center'}
# Create aggregated object
dd_aggr <- aggregate(Count ~ Year, data = dd, FUN = sum)
# Plot the graph 
ggplot(dd_aggr, aes(x=Year, y= Count)) + geom_line(colour = "steelblue") + geom_point(colour = "steelblue") + theme_minimal() + theme(axis.title.x=element_blank()) + theme(axis.title.y=element_blank()) 
```

Figure 2 depicts the same information as the previous one but dividing the crimes by type. From the graph it is clear that the most common types of crimes over the whole period of time are Theft and Batery; and that all type of crimes have been falling since 2001 to a greater or lesser extent. 

```{r fig2, fig.cap="Crimes evolution per type of crime 2001-2015", echo=FALSE, message=FALSE, warning=FALSE, out.extra='',fig.align='center'}
# Create aggregated object
dd_aggr2 <- aggregate(Count ~ Type_grouped + Year, data = dd, FUN = sum)
# Plot the graph
ggplot(data=dd_aggr2, aes(x=Year, y=Count, group = Type_grouped, colour = Type_grouped)) +
    geom_line() + geom_point() + theme_minimal() + theme(axis.title.x=element_blank()) + theme(axis.title.y=element_blank())+ theme(legend.title=element_blank())
```

\pagebreak

### What time of day does most crime occur?

Figures 3 and 4 lead us to anwer question 2. Figure 3 indicates that the number of crimes increases gradually from 05:00 in the morning (the hour with less crimes) until 20:00 in the evening (the hour with the most crimes). The hours of 12:00 and 00:00 are exceptionally high, both at a similar level as 20:00.

```{r, fig3, fig.cap="Crimes per hour", echo=FALSE, out.extra='',fig.align='center'}
# Plot the graph 
ggplot(dd, aes(x=Hour))+geom_bar(stat="Count", width=0.8, fill = "steelblue")+ theme(axis.text.x = element_text(angle = 0, hjust = 1)) + labs(x = "Hour", y = "Number of crimes") + theme_minimal()+ theme(axis.title.x=element_blank()) + theme(axis.title.y=element_blank())
```

Figure 4 shows in a heat-map the distribution of number of crimes per hour and type. From this visualisaition we can highlight, for example, that whereas the peak hours of Theft and Others are at 00:00, 09:00 and 12:00, Narcotics concentrate between 10:00 to 14:00 and 19:00 to 22:00. Other types of crime are more evenly distributed throughout the day.

```{r fig4, fig.cap="Type of crime vs hour", echo=FALSE, out.extra='', fig.align='center'}
# Create aggregated object
dd_aggr3 <- aggregate(Count ~ Type_grouped + Hour, data = dd, FUN = sum)
# Plot graph
p1 <- ggplot(data = dd_aggr3, aes(x = Hour, y = Type_grouped)) + geom_tile(aes(fill = Count), color = "white") +
  scale_fill_gradient(low = "white", high = "steelblue") 
p1 + theme_minimal()+ theme(axis.title.x=element_blank()) + theme(axis.title.y=element_blank()) +
  theme(legend.title = element_text(size = 10),
        legend.text = element_text(size = 6),
        axis.text.y = element_text(size= 8),
        axis.text.x = element_text(size = 8, angle = 45, hjust = 1)) 
```

\pagebreak

### In which locations of the city is crime more likely to happen? 

Figure 5 and 6 indicate the location of crime. As is illustrated by Figure 5, most crimes happen in the Street, followed by Residences and Apartments.

```{r fig5, fig.cap="Crimes per location", echo=FALSE, out.extra='',fig.align='center'}
# Create aggregated object
dd_aggr4 <- aggregate(Count ~ Location_grouped, data = dd, FUN = sum)
# Order values
dd_aggr4$Location_grouped <- factor(dd_aggr4$Location_grouped, levels = dd_aggr4$Location_grouped[order(-dd_aggr4$Count)])
# Plot the graph 
ggplot(dd_aggr4, aes(x = Location_grouped, y = Count)) + theme_minimal() + geom_bar(stat="identity", width=0.7, fill = "steelblue") + theme(axis.text.x = element_text(angle = 45, hjust = 1)) + labs(x = "Location", y = "Number of crimes")+ theme(axis.title.x=element_blank()) + theme(axis.title.y=element_blank())
```

Figure 6, shows that some types of crime tend to occur in specific locations. For instance, Robery is recorded almost entirely in the Street, as well as Narcotics. However, Burglary and Others are registered particularly in Residences or Apartments - What makes sense.

```{r fig6, fig.cap="Type of crime vs location", echo=FALSE, out.extra='',fig.align='center'}
# Create aggregated object
dd_aggr5 <- aggregate(Count ~  Type_grouped + Location_grouped, data = dd, FUN = sum)
# Plot the graph
p2 <- ggplot(data = dd_aggr5, aes(x = Location_grouped, y = Type_grouped)) + geom_tile(aes(fill = Count), color = "white") +
  scale_fill_gradient(low = "white", high = "steelblue") 
p2+ theme_minimal()+ theme(axis.title.x=element_blank()) + theme(axis.title.y=element_blank()) +
  theme(legend.title = element_text(size = 10),
        legend.text = element_text(size = 6),
        axis.text.y = element_text(size= 8),
        axis.text.x = element_text(size = 8, angle = 45, hjust = 1)) 
```

\pagebreak

### Which districts are more potentially dangerous?

Finally, Figure 7 and 8 gives us the anwer for the last question. In Figure 7 we visualise the number of crimes per districts. The most dangerous district seems to be number 8, with more than 30,000 records in the 15 years, while district 20 with less than 10,000 seems the safest. 

```{r fig7, echo=FALSE, fig.cap="Crimes per district"}
# Remove values districts 21 and 31
dd$District<- as.factor(dd$District)
dd_sub <- subset(dd, District!="21" & District!="31")
# Create aggregated object
dd_aggr6 <- aggregate(Count ~ District, data = dd_sub, FUN = sum)
# Order values
dd_aggr6$District <- factor(dd_aggr6$District, levels = dd_aggr6$District[order(-dd_aggr6$Count)])
# Plot the graph 
ggplot(dd_aggr6, aes(x=District, y = Count)) + theme_minimal() + geom_bar(stat="identity", width=0.7, fill = "steelblue") + theme(axis.text.x = element_text(angle = 45, hjust = 1)) + labs(x = "District", y = "Number of crimes") + theme(axis.title.x=element_blank()) + theme(axis.title.y=element_blank())
```

Figure 8 illustrates some interesting findings in the relations between the type of crime and districts where they occurred. For example, we can see that districts 1, 8, 12 and 18 are particularly dangerous in terms of Theft, that districts 7 and 11 stand out in terms of Batery, and how clearly Narcotics crime concentrates in district 11 and 15.

```{r fig8, fig.cap="Type of crime vs district", echo=FALSE, out.extra='',fig.align='center'}
# Create aggregated object
dd_aggr7 <- aggregate(Count ~ Type_grouped + District, data = dd_sub, FUN = sum)
# Plot the graph
p3<-ggplot(data = dd_aggr7, aes(x = District, y = Type_grouped)) +
  geom_tile(aes(fill = Count), color = "white") +
  scale_fill_gradient(low = "white", high = "steelblue")  
p3+ theme_minimal()+ theme(axis.title.x=element_blank()) + theme(axis.title.y=element_blank()) +
  theme(legend.title = element_text(size = 10),
        legend.text = element_text(size = 6),
        axis.text.y = element_text(size= 8),
        axis.text.x = element_text(size = 8, angle = 45, hjust = 1)) 
```
