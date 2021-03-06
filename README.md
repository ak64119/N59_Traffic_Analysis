Traffic Analysis and Visualisation of N59, Galway City
================
Akshay Kochhar - 18230051
29th March 2019

**Introduction:**
-----------------

Galway City Council has shared real traffic data on traffic on the N59 just before and at the junction outside the Insight building in the IDA Business park in Dangan, Galway. Those of us who work in this area know that this junction suffers severe congestion in the morning and evening hours.

There are fairly infrequent public transport options on the N59 to and from its nearest town, Moycullen; the road is not terribly bicycle-friendly, as it does not have a bicycle path and is narrow with sharp turns in places. At the same time, Moycullen's population has significantly grown and contributes to the morning and evening commuter traffic.

The visualisation and analysis of the traffic data presented in the report below share suggestions on increasing the number of buses or introducing cycle path to and from Moycullen to Galway city via the IDA Business Park and the University.

**Deliverables:**
-----------------

#### **Load Libraries:**

``` r
library(lubridate) # date and time formatting
library(tidyr) # data formatting
library(ggplot2) # graphs
library(hms) # storing time values
library(dplyr) # data manipulation
library(xts) #creating time-series objects
library(lattice) # lattice graphics
library(RColorBrewer) # colourful graphs
library(gplots) # plotting Data
library(readxl) # read excel
library(ggridges) #ridgeline plot
library(scales) # graph layouts
library(ggpubr) # arranging plots
```

#### **Date Pre-Processing**

The original data was not in the R acceptable format and couldn't be imported into R. Each XLSX file had multiple sheets, and each sheet had multiple columns. The data was cleaned by removing the additional rows, transposing the rows, adding appropiate headers and columns. Eg: A column is added to distinguish the data as 'Eastbound' or 'Westbound'. After loading the data into R, it was further pre-processed to manipulate the data in desired format. The headers of the column were changed so that it could be analysed easily.

``` r
# Pre-Processing

setwd("E:/New Volume/Academic/NUIG_College/Data_Visulization/Assignment_5/")


data <- read.csv2("Duplicate_Data.csv",sep = ",", header=FALSE,stringsAsFactors = FALSE)

colnames(data) <- c("Location","Date","Time","V0-5","V5-10","V10-15","V15-20","V20-25","V25-30","V30-35","V35-40","V40-45","V45-50","V50-55","V55-60","V60-65","V65-70","V70-75","V75-80","V80-85","V85-90","V90-95","V95-100","V100-105","V105-110","V110-115","V115-120","V120-125","V125-145","V140-160","Cyclist", "M/Cycle", "Car","Van","Rigid_2_Axle","Rigid_3_Axle","Rigid_4_Axle","3_Axle_HGv","4_Axle_HGV","5_Axle_HGV","Bus","Mean","Vpp85","Vmax")

data <- data[, -c(42,43,44)]

data$Date <- as.Date(as.character(data$Date),format = "%Y-%m-%d")

data$Time <- as.hms(data$Time)

# now create a 'day' column to represent a day label "e.g. day 1"
data$day<-"NA"
data$day[data$Date == "2016-11-18"]<-"Friday"
data$day[data$Date == "2016-11-19"]<-"Saturday"
data$day[data$Date == "2016-11-20"]<-"Sunday"
data$day[data$Date == "2016-11-21"]<-"Monday"
data$day[data$Date == "2016-11-22"]<-"Tuesday"
data$day[data$Date == "2016-11-23"]<-"Wednesday"
data$day[data$Date == "2016-11-24"]<-"Thursday"

# create an ordering for the new day feature
data$day <-factor(data$day, levels=c("Friday", "Saturday", "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday"))
```

### **1. The Periods Of Traffic Congestion**

The data was manipulated on the basis of velocity ranges with different velocity ranges in 'Velocity Range' column, their respective values in 'Outcome' column and the count of vehicles grouped by Location, Date and Time in one column.

A geom line plot is used to represent the maximum number of vechiles in a particular time range with colours represting the direction of traffic and different facets representing the days of the week.

``` r
#Traffic Congestion

subset_1 <- data[,-c(31:41)]

subset_1 <- gather(subset_1, key=Velocity_Range, value=Outcome, 4:30)

subset_1 <- subset_1 %>%
                 group_by(Location, Date, Time) %>%
                 mutate(Time.Vehicle.Sum.per.Time= sum(Outcome))

traffic_cong <- ggplot(subset_1, aes(x = Time, y=Time.Vehicle.Sum.per.Time, colour = Location))

traffic_cong <- traffic_cong + geom_line() + 
  facet_wrap(~subset_1$day, nrow=4,ncol=2) +
  scale_colour_brewer(palette = "Dark2", name = "") +
  ylab("Total Number of Vehicles") + 
  labs(title = "Figure 1.1 | Periods of Traffic Congestion Per Day",
       subtitle = "7 Day Data | Coloured By Traffic Direction",
       color = "Traffic Direction",
       caption = "Galway City Council Traffic Data") +
  xlab("Time") + 
  theme_grey() +
  theme(
    axis.line = element_line(colour = "black", size = 0.25), 
    axis.title.x=element_text(), 
    legend.key = element_rect(fill = NA, colour = NA, size = 0.25)
    ) 
```

#### **Analysis**

As per the plots below, we can observe that from Monday to Friday, interestingly, the traffic pattern is similar for both 'eastbound' and 'westbound' directions. The traffic starts increasing steadily from about 05:00 AM in the morning, and is at its peak in between 08:00 to 09:00, which is the prime office going hours. After 09:00 AM the traffic goes down and remains steady with some peaks till 15:00 in the evening. This traffic can be due to commuters between cities or people coming for lunch breaks. The traffic starts to increase after 15:00 and reaches at its peak between 17:00 and 18:00 which is due to people returning back from work. After that, till 24:00 the traffic drops to minimum.

On weekends, the traffic remains low as compared to normal working days. The only increase in traffic is bewteen 12:00 and 18:00 which can be explained that people come out to visit and buy groceries.

Interestingly, there is a very sharp increase in traffic on wednesday between 06:00 to 10:00 in the morning. This could be because of any event in Galway city which could have caused a sharp increase in traffic.

``` r
traffic_cong
```

<img src="Assignment_5_final_files/figure-markdown_github/traffic_cong-1.png" style="display: block; margin: auto;" />

### **2. Daily Traffic Vehicle Distribution**

The different types of vehicle were filtered from the main data set with location and day. There total count of each vehicle type was taken per day and location-wise.

``` r
#Stacked Bar Chart Data Preparation

subset_2 <- data[, c(1,31,32,33,34,35,36,37,38,39,40,41,42)]

subset_2 <- subset_2 %>%
                 group_by(Location, day) %>%
                 summarise_each(funs(sum))

subset_2 <- gather(subset_2, key=Vehicle_Type, value= Number, 3:13)


#Stacked Bar Chart Data Preparation

subset_3 <- data[, c(1,31,32,33,34,35,36,37,38,39,40,41,42)]

subset_3 <- subset_3 %>% group_by(Location, day) %>% summarise_each(funs(sum))

subset_3 <- subset_3[,c(1,2,5,6,7)]

subset_3 <- gather(subset_3, key=Vehicle_Type, value= Share, 3:5)

subset_3$day <- NULL

subset_3 <- subset_3 %>% 
  group_by(Location, Vehicle_Type) %>% 
  summarise(Vechile_Dist = round(mean(Share),0))

subset_3 <- subset_3 %>%
                 group_by(Location) %>%
                 mutate(Vehicle.share = mean(Vechile_Dist, na.rm = T))

subset_3$Vehicle.share <- as.double(c("0.4426","0.0340","0.5320","0.4049","0.0287","0.5749"))
```

``` r
# Define the number of colors to be filled in sub-groups
nb.cols_1 <- 11
mycolors_1 <- colorRampPalette(brewer.pal(8, "Dark2"))(nb.cols_1)

stack_bar <- ggplot(subset_2, aes(x= day, y= Number,fill= Vehicle_Type)) + 
     geom_bar( stat="identity", position="fill") +
     scale_y_continuous(labels = scales::percent) +
     facet_wrap(~Location, nrow=1,ncol=2,scales = "free") +
     ylab("Vehicle Distribuiton (%) per Day") +
     labs(title = "Figure 2.1 | Distribution of Daily Traffic",
       subtitle = "7 Day Data | Coloured By Vehicle Type",
       fill = "Vehicle Type",
       caption = "Galway City Council Traffic Data") +
     scale_fill_manual(values = mycolors_1,name = "Vehicle Types") +
      theme_minimal() +
      theme(
        axis.text.x = element_text(vjust = 5, size=8, face="bold"),
        strip.text.x = element_text(size = 10, face = "bold"),
        axis.line.y= element_line(colour = "grey", size = 0.1),
        axis.text.y = element_text(size=8)) + 
  coord_flip()


#Pie Chart

# Create a basic bar
pie <- ggplot(subset_3, aes(x="", y=Vehicle.share, fill=Vehicle_Type)) + geom_bar(stat="identity", width=1)

# Convert to pie (polar coordinates) and add labels
pie <- pie + coord_polar("y", start=0) + geom_text(aes(label = paste0(round(Vehicle.share*100), "%")), position = position_stack(vjust = 0.5))

# Add color scale (hex colors)
pie <- pie + scale_fill_manual(values = c("#33658A", "#F6AE2D", "#F26419"), name = "Vehicle Type") 

# Facet wrap to separate data location wise
pie <- pie + facet_wrap(~Location, nrow=4,ncol=2)

# Remove labels and add title
pie <- pie + labs(x = NULL, y = NULL, fill = NULL, title = "Figure 2.2 | Major Contributors to Daily Traffic",
                  subtitle = "7 Days Average Data | Coloured By Vehicle Type",caption = "Galway City Council Traffic Data")

# Tidy up the theme
pie <- pie + theme_classic() + theme(axis.line = element_blank(),
          axis.text = element_blank(),
          axis.ticks = element_blank())
```

#### **Analysis**

As per figure 2.1, we can see that Car, Van and Rigid\_2\_Axle vehicle contribute to around 97% of the daily traffic from both eastbound and westbound directions. The contribution of the remaining vehicle types have been negligible over all the days of the week.

The pie chart below(figure 2.2) was created to know the excat proportion of major contributors to the daily traffic. It was found that out of major contributors, Car and Van adds to around 95% of the total traffic.

``` r
stack_bar
```

<img src="Assignment_5_final_files/figure-markdown_github/vehicle_dist_plots_2-1.png" style="display: block; margin: auto;" />

``` r
pie
```

<img src="Assignment_5_final_files/figure-markdown_github/vehicle_dist_plots_3-1.png" style="display: block; margin: auto;" />

### **3. Galway City Council Bus Recommendations**

The data was taken from 'junction turning counts' excel in which the data was segregated for each route.

``` r
setwd("E:/New Volume/Academic/NUIG_College/Data_Visulization/Assignment_5/")

#Read route Data
B_A_route <- read_excel("Route_data.xlsx", sheet=3)
B_C_route <- read_excel("Route_data.xlsx", sheet=4)
A_C_route <- read_excel("Route_data.xlsx", sheet=5)
C_A_route <- read_excel("Route_data.xlsx", sheet=6)
C_B_route <- read_excel("Route_data.xlsx", sheet=7)
A_B_route <- read_excel("Route_data.xlsx", sheet=8)

colnames(B_A_route) <- c("Date_Time","PCL","MCL","Car","Taxi","CDB","BEB","OB")
colnames(B_C_route) <- c("Date_Time","PCL","MCL","Car","Taxi","CDB","BEB","OB")
colnames(A_C_route) <- c("Date_Time","PCL","MCL","Car","Taxi","CDB","BEB","OB")
colnames(C_A_route) <- c("Date_Time","PCL","MCL","Car","Taxi","CDB","BEB","OB")
colnames(C_B_route) <- c("Date_Time","PCL","MCL","Car","Taxi","CDB","BEB","OB")
colnames(A_B_route) <- c("Date_Time","PCL","MCL","Car","Taxi","CDB","BEB","OB")

B_A_route$Time <- format(B_A_route$Date_Time,"%H:%M:%S")
B_A_route$Date_Time <- NULL
B_C_route$Time <- format(B_C_route$Date_Time,"%H:%M:%S")
B_C_route$Date_Time <- NULL
A_C_route$Time <- format(A_C_route$Date_Time,"%H:%M:%S")
A_C_route$Date_Time <- NULL
C_A_route$Time <- format(C_A_route$Date_Time,"%H:%M:%S")
C_A_route$Date_Time <- NULL
C_B_route$Time <- format(C_B_route$Date_Time,"%H:%M:%S")
C_B_route$Date_Time <- NULL
A_B_route$Time <- format(A_B_route$Date_Time,"%H:%M:%S")
A_B_route$Date_Time <- NULL
```

#### **Pre-Recommendation Processing**

To make recommendations about bus services, it is important to know which route has the maximum traffic, maximum number of commuters, peak hours of traffic, and the number of people ready to take bus transportation.

In order to get all this data, I have plotted heat map for the route 'Business Park To Moycullen', 'Business Park To Galway City and University', 'Moycullen To Business Park', 'Moycullen To Galway City and University', 'Galway City and University To Business Park' and 'Galway City and University To Moycullen'.

``` r
# Heatmaps

a <- c("#e7f0fa", "#c9e2f6", "#95cbee", "#0099dc","#4ab04a", "#ffd73e")
b <- c("#eec73a", "#e29421", "#e29421", "#f05336","#f05336")

#####################

cols <- c(colorRampPalette(a)(10),colorRampPalette(b, bias=2)(90))

A_B_route <- gather(A_B_route, key=Vehicle_Type, value=Outcome, 1:7)

heat_mp_1 <- ggplot(data = A_B_route, aes(x = Vehicle_Type, y = Time)) +
  geom_tile(aes(fill = Outcome)) +
  labs(x = "Vehicle Type", y = "Time",title = "Figure 3.1 | Distribution Of Traffic Turning from N59",
       subtitle = "IDA Business Park To Moycullen | 22nd November 2016",
       fill = "Number of Vehicles",
       caption = "Galway City Council Traffic Data") +
  scale_fill_gradientn(colours=cols, limits=c(0, 60),
                        breaks=seq(0, 50, by=10), 
                        na.value=rgb(246, 246, 246, max=255),
                        labels=c("0", "10", "20", "30", "40", "50"),
                        guide=guide_colourbar(ticks=T, nbin=50,
                               barheight=5, label=T, 
                               barwidth=0.5)) +
  theme(axis.text.y = element_text(size = 6))


#####################

A_C_route <- gather(A_C_route, key=Vehicle_Type, value=Outcome, 1:7)

heat_mp_2 <- ggplot(data = A_C_route, aes(x = Vehicle_Type, y = Time)) +
  geom_tile(aes(fill = Outcome)) +
    labs(x = "Vehicle Type", y = "Time", title = "Figure 3.2 | Distribution Of Traffic Turning from N59",
       subtitle = "IDA Business Park To Galway City and University | 22nd November 2016",
       fill = "Number of Vehicles",
       caption = "Galway City Council Traffic Data") +
  scale_fill_gradientn(colours=cols, limits=c(0, 60),
                        breaks=seq(0, 50, by=10), 
                        na.value=rgb(246, 246, 246, max=255),
                        labels=c("0", "10", "20", "30", "40", "50"),
                        guide=guide_colourbar(ticks=T, nbin=50,
                               barheight=5, label=T, 
                               barwidth=0.5)) +
  theme(axis.text.y = element_text(size = 6))


#####################

B_A_route <- gather(B_A_route, key=Vehicle_Type, value=Outcome, 1:7)

heat_mp_3 <- ggplot(data = B_A_route, aes(x = Vehicle_Type, y = Time)) +
  geom_tile(aes(fill = Outcome)) +
    labs(x = "Vehicle Type", y = "Time", title = "Figure 3.3 | Distribution Of Traffic Turning from N59",
       subtitle = "Moycullen To IDA Business Park | 22nd November 2016",
       fill = "Number of Vehicles",
       caption = "Galway City Council Traffic Data") +
  scale_fill_gradientn(colours=cols, limits=c(0, 60),
                        breaks=seq(0, 50, by=10), 
                        na.value=rgb(246, 246, 246, max=255),
                        labels=c("0", "10", "20", "30", "40", "50"),
                        guide=guide_colourbar(ticks=T, nbin=50,
                               barheight=5, label=T, 
                               barwidth=0.5)) +
  theme(axis.text.y = element_text(size = 6))


#####################

B_C_route <- gather(B_C_route, key=Vehicle_Type, value=Outcome, 1:7)

heat_mp_4 <- ggplot(data = B_C_route, aes(x = Vehicle_Type, y = Time)) +
  geom_tile(aes(fill = Outcome)) +
  labs(x = "Vehicle Type", y = "Time", title = "Figure 3.4 | Distribution Of Traffic Turning from N59",
       subtitle = "Moycullen To Galway City and University | 22nd November 2016",
       fill = "Number of Vehicles",
       caption = "Galway City Council Traffic Data") +
  scale_fill_gradientn(colours=cols, limits=c(0, 140),
                        breaks=seq(0, 140, by=20), 
                        na.value=rgb(246, 246, 246, max=255),
                        labels=c("0","20","40","60","80","100","120","140"),
                        guide=guide_colourbar(ticks=T, nbin=50,
                               barheight=5, label=T, 
                               barwidth=0.5)) +
  theme(axis.text.y = element_text(size = 6))

#####################

C_A_route <- gather(C_A_route, key=Vehicle_Type, value=Outcome, 1:7)

heat_mp_5 <- ggplot(data = C_A_route, aes(x = Vehicle_Type, y = Time)) +
  geom_tile(aes(fill = Outcome)) +
  labs(x = "Vehicle Type", y = "Time", title = "Figure 3.5 | Distribution Of Traffic Turning from N59",
       subtitle = "Galway City and University To IDA Business Park | 22nd November 2016",
       fill = "Number of Vehicles",
       caption = "Galway City Council Traffic Data") +
  scale_fill_gradientn(colours=cols, limits=c(0, 70),
                        breaks=seq(0, 60, by=10), 
                        na.value=rgb(246, 246, 246, max=255),
                        labels=c("0", "10", "20", "30", "40", "50", "60"),
                        guide=guide_colourbar(ticks=T, nbin=50,
                               barheight=5, label=T, 
                               barwidth=0.5)) +
  theme(axis.text.y = element_text(size = 6))

#####################

C_B_route <- gather(C_B_route, key=Vehicle_Type, value=Outcome, 1:7)

heat_mp_6 <- ggplot(data = C_B_route, aes(x = Vehicle_Type, y = Time)) +
  geom_tile(aes(fill = Outcome)) +
  labs(x = "Vehicle Type", y = "Time", title = "Figure 3.6 | Distribution Of Traffic Turning from N59",
       subtitle = "Galway City and University To Moycullen | 22nd November 2016",
       fill = "Number of Vehicles",
       caption = "Galway City Council Traffic Data") +
  scale_fill_gradientn(colours=cols, limits=c(0, 120),
                        breaks=seq(0, 120, by=20), 
                        na.value=rgb(246, 246, 246, max=255),
                        labels=c("0","20","40","60","80","100","120"),
                        guide=guide_colourbar(ticks=T, nbin=50,
                               barheight=5, label=T, 
                               barwidth=0.5)) +
  theme(axis.text.y = element_text(size = 6))
```

#### **Pre-Recommendation Analysis**

As per below heat maps from 3.1 to 3.6, I have the correct observation that maximum number of commuters travel from and to Galway City-University-Moycullen by cars. The traffic on n59 is majorly because of these type of commuters from 07:00 in the morning till 18:45 in the evening. They lead to heavy traffic on n59 around 07:45 till 09:15 in the morning and 13:00 till 18:30 in the evening (ref fig\#3.4 and 3.6). Out of all the routes shared below, it is beneficial to run bus service on to-and-from "Galway City-University-Moycullen"."

``` r
heat_mp_1
```

<img src="Assignment_5_final_files/figure-markdown_github/pre_recomdation plots-1.png" style="display: block; margin: auto;" />

``` r
heat_mp_2
```

<img src="Assignment_5_final_files/figure-markdown_github/pre_recomdation plots-2.png" style="display: block; margin: auto;" />

``` r
heat_mp_3
```

<img src="Assignment_5_final_files/figure-markdown_github/pre_recomdation plots-3.png" style="display: block; margin: auto;" />

``` r
heat_mp_4
```

<img src="Assignment_5_final_files/figure-markdown_github/pre_recomdation plots-4.png" style="display: block; margin: auto;" />

``` r
heat_mp_5
```

<img src="Assignment_5_final_files/figure-markdown_github/pre_recomdation plots-5.png" style="display: block; margin: auto;" />

``` r
heat_mp_6
```

<img src="Assignment_5_final_files/figure-markdown_github/pre_recomdation plots-6.png" style="display: block; margin: auto;" />

#### **Bus Recommendation Processing**

The data was taken from 'junction turning counts' excel in which the data was segregated for each route. The count of cars and taxis are added in 'Car' column, and the count of all the buses (CDB, BEB and OB) are added in bus column.

``` r
setwd("E:/New Volume/Academic/NUIG_College/Data_Visulization/Assignment_5/")

#Preparing Data

B_C_route_indi <- read_excel("Route_data.xlsx", sheet=4)
C_B_route_indi <- read_excel("Route_data.xlsx", sheet=7)

colnames(B_C_route_indi) <- c("Date_Time","PCL","MCL","Car","Taxi","CDB","BEB","OB")
colnames(C_B_route_indi) <- c("Date_Time","PCL","MCL","Car","Taxi","CDB","BEB","OB")

B_C_route_indi$Time <- format(B_C_route_indi$Date_Time,"%H:%M:%S")
B_C_route_indi$Date_Time <- NULL
B_C_route_indi$Car <- (B_C_route_indi$Car+B_C_route_indi$Taxi)
B_C_route_indi$Bus <- (B_C_route_indi$CDB+B_C_route_indi$BEB+B_C_route_indi$OB)
B_C_route_indi$Taxi <- NULL
B_C_route_indi$MCL <- NULL
B_C_route_indi$CDB <- NULL
B_C_route_indi$BEB <- NULL
B_C_route_indi$OB <- NULL
B_C_route_indi <- B_C_route_indi[,c(3,1,2,4)]

C_B_route_indi$Time <- format(C_B_route_indi$Date_Time,"%H:%M:%S")
C_B_route_indi$Date_Time <- NULL
C_B_route_indi$Car <- (C_B_route_indi$Car+C_B_route_indi$Taxi)
C_B_route_indi$Bus <- (C_B_route_indi$CDB+C_B_route_indi$BEB+C_B_route_indi$OB)
C_B_route_indi$Taxi <- NULL
C_B_route_indi$MCL <- NULL
C_B_route_indi$CDB <- NULL
C_B_route_indi$BEB <- NULL
C_B_route_indi$OB <- NULL
C_B_route_indi <- C_B_route_indi[,c(3,1,2,4)]

  
B_C_route_indi <- gather(B_C_route_indi, key=Vehicle_Type, value=Outcome, 2:4)
C_B_route_indi <- gather(C_B_route_indi, key=Vehicle_Type, value=Outcome, 2:4)

B_C_route_indi$Time <- as.hms(B_C_route_indi$Time)
C_B_route_indi$Time <- as.hms(C_B_route_indi$Time)
```

As we know the potential route which could be beneficial to run bus service, we need to plot the present traffic distribution on that route to know the current state of traffic, and the number of buses already in running.

``` r
#Traffic Distribution Plots on n59

bus_plot_1 <- ggplot(B_C_route_indi, aes(x=Time, y=0, height = Outcome, fill= (Outcome))) +
  
  geom_ridgeline_gradient(show.legend = TRUE, size = 0.2) +
  
  scale_y_continuous(name = "Number of Vehicles", position = "right") + 
  
  scale_fill_distiller( type = "seq",  palette = "Spectral", direction = -1, 
                        values = NULL, space = "Lab", na.value = "grey50", guide = "colourbar", aesthetics = "fill") +
  
  labs(title = "Figure 3.7 | Distribution Of Traffic Turning from N59",
       subtitle = "Moycullen To Galway City and University | 22nd November 2016",
       fill = "Vehicle Count",
       caption = "Galway City Council Traffic Data") +
  
  theme_grey() +
  
  theme(
        axis.title.x=element_blank(), 
        axis.text.x = element_text(vjust = 0.5, size=7, face="bold"),
        axis.text.y = element_text(size=5),
        axis.title.y = element_text(size=8, face="bold"),
        panel.grid.major.y =element_line(colour = "gray95", size =0.2),
        panel.grid.major.x =element_line(colour = "gray95", size =0.2),
        panel.grid.minor.y =element_blank(),
        panel.grid.minor.x =element_blank(),
        plot.title=element_text(hjust=0.00, face='bold', size=11),
        strip.text.y = element_text(angle=180, size=7.5, colour = "black", face="bold"),
        strip.background = element_blank(),
        strip.placement = "outside"
        )  +

    facet_grid(Vehicle_Type ~ ., scales="free", switch="y")


################

bus_plot_2 <- ggplot(C_B_route_indi, aes(x=Time, y=0, height = Outcome, fill= (Outcome))) +
  
  geom_ridgeline_gradient(show.legend = TRUE, size = 0.2) +
  
  scale_y_continuous(name = "Number of Vehicles", position = "right") + 
  
  scale_fill_distiller( type = "seq",  palette = "Spectral", direction = -1, 
                        values = NULL, space = "Lab", na.value = "grey50", guide = "colourbar", aesthetics = "fill") +
  
    labs(title = "Figure 3.8 | Distribution Of Traffic Turning from N59",
       subtitle = "Galway City and University To Moycullen | 22nd November 2016",
       fill = "Vehicle Count",
       caption = "Galway City Council Traffic Data") +
  
  theme_grey() +
  
  theme(
        axis.title.x=element_blank(), 
        axis.text.x = element_text(vjust = 0.5, size=7, face="bold"),
        axis.text.y = element_text(size=5),
        axis.title.y = element_text(size=8, face="bold"),
        panel.grid.major.y =element_line(colour = "gray95", size =0.2),
        panel.grid.major.x =element_line(colour = "gray95", size =0.2),
        panel.grid.minor.y =element_blank(),
        panel.grid.minor.x =element_blank(),
        plot.title=element_text(hjust=0.00, face='bold', size=11),
        strip.text.y = element_text(angle=180, size=7.5, colour = "black", face="bold"),
        strip.background = element_blank(),
        strip.placement = "outside"
        )  +
    facet_grid(Vehicle_Type ~ ., scales="free", switch="y")
```

#### **Traffic Analysis from N59**

As per fig. 3.7, the route 'Moycullen To Galway City and University' is the bussiest from 07:00 till 09:15 in the morning, but surprisingly there are hardly any buses available at this time.

As per fig. 3.8, the route 'Galway City and University To Moycullen' has the traffic increasing from 13:00 and reaches its peak around 18:15, but there is not even one bus service available for the commuters.

``` r
bus_plot_1
```

<img src="Assignment_5_final_files/figure-markdown_github/bus_recomdation plots-1.png" style="display: block; margin: auto;" />

``` r
bus_plot_2
```

<img src="Assignment_5_final_files/figure-markdown_github/bus_recomdation plots-2.png" style="display: block; margin: auto;" />

#### **Recommendations**

As per fig. 3.7 and 3.8, it was observed that even-though, the traffic is maximum at the office going hours on the route to-and-from 'Moycullen To Galway City and University, there is no bus service available to carry the commuters around.

**Assumptions:**

1.  One standard bus can carry around 35 passengers\[1\].(taking 35 rather than 40 to have realistic assumptions.)
2.  Only one person travels in the whole car while commuting.
3.  The bus will go via. IDA Business Park while travelling on the route 'Moycullen To Galway City and University'.
4.  The distance is reachable from the IDA bus junction to the IDA business park by foot.
5.  Most of the commuters will travel to university center and Galway city, which is the path B-A-C.

**a) Moycullen To Galway City and University:**

1.  As per figure 3.7 around 760 vehicles travel in the time-frame of 07:30 to 09:15 with 15 minutes difference slab.
2.  A bus carries around 35 passengers (approx.). Therefore, a bus takes around 35 vehicles off the road while moving in single direction.
3.  A bus takes around 20 minutes from 'Moycullen' to 'Galway City'.
4.  The traffic starts to rise around 07:30 AM.
5.  Therefore, taking in account the total number of Vehicles within this time-frame, a bus at every 8th minute (14 buses in total) starting from 07:15 AM till 09:00 AM will help in controlling the traffic during this time.
6.  The before and after bus service vehicle count (fig. 3.9) is demonstrated in the figure below taking in account the above assumptions and calculations.

**b) Galway City and University To Moycullen:**

1.  As per figure 3.8 around 1940 vehicles travel in the time-frame of 13:00 to 18:30 with 15 minutes difference slab.
2.  A bus carries around 35 passengers (approx.). Therefore, a bus takes around 35 vehicles off the road while moving in single direction.
3.  A bus takes around 20 minutes from 'Galway City' to 'Moycullen'.
4.  The traffic starts to rise around 13:00 PM.
5.  Therefore, taking in account the total number of Vehicles within this time-frame, a bus at every 11th minute starting from 12:45 PM till 18:15 PM helps in controlling the traffic during this time.
6.  The before and after bus service vehicle count (fig. 3.10) is demonstrated in the figure below taking in account the above assumptions and calculations.

**Before and After Bus Service Plots:**

As per the plots below, we can see that the traffic is under significant control with adding multiple buses on this route.

``` r
setwd("E:/New Volume/Academic/NUIG_College/Data_Visulization/Assignment_5/")

#### Moycullen To Galway City and University

Before_After_MG <- read_excel("Moycullen_To_Galway.xlsx", sheet=1)
Before_After_MG <- gather(Before_After_MG, key=Time_frame, value= Number, 2:3)
Before_After_MG$Time <- as.hms(Before_After_MG$Time)

before_after_1 <- ggplot(Before_After_MG, aes(x = factor(format(Time, format = "%b")), y=Number, fill=Time_frame)) +
  geom_bar(position="dodge", stat="identity")  +
  scale_y_continuous(name = "Number of Uploads", 
                     limits=c(0,140), breaks=c(0,20,40,60,80,100,120,140)) + 
  scale_fill_manual(name="Colour Tag",labels = c("Without Buses", "With Buses"),
                    values = c("#0072b2", "#D55E00"),guide = guide_legend(reverse = TRUE)) +
  labs(
    x = "", y = "Vehicle Count",
    title = "Figure 3.9 | Before and After Bus Service Traffic Distribution",
    subtitle = "Moycullen To Galway City and University | 22nd November 2016",
       fill = "With and Without Bus",
       caption = "Galway City Council Traffic Data") +
  
  theme_classic() +
  
  theme(
        axis.ticks.x = element_blank(),
        axis.line.y= element_line(colour = "black", size = 0.1),
        axis.title.x=element_blank(), 
        axis.text.x = element_text(size=9, face="bold"),
        axis.text.y = element_text( size=7),
        axis.title.y = element_text(size=9, face="bold"),
        legend.text = element_text(size=8),
        legend.key.size = unit(0.8,"line"),
        plot.title=element_text( hjust=0.00, face='bold', size=11)
        ) + 
  coord_flip()

before_after_1
```

<img src="Assignment_5_final_files/figure-markdown_github/befre_after-1.png" style="display: block; margin: auto;" />

``` r
####Galway City and University To Moycullen

Before_After_MG <- read_excel("Moycullen_To_Galway.xlsx", sheet=2)
Before_After_MG <- gather(Before_After_MG, key=Time_frame, value= Number, 2:3)
Before_After_MG$Time <- as.hms(Before_After_MG$Time)

before_after_2 <- ggplot(Before_After_MG, aes(x = factor(format(Time, format = "%b")), y=Number, fill=Time_frame)) +
  geom_bar(position="dodge", stat="identity")  +
  scale_y_continuous(name = "Number of Uploads", limits=c(0,110), breaks=c(0,10,20,30,40,50,60,70,80,90,100,110)) + 
  scale_fill_manual(name="Colour Tags",labels = c("Without Buses", "With Buses"),
                    values = c("#0072b2", "#D55E00"),guide = guide_legend(reverse = TRUE)) +
    labs(
    x = "", y = "Vehicle Count",
    title = "Figure 3.10 | Before and After Bus Service Traffic Distribution",
    subtitle = "Galway City and University To Moycullen | 22nd November 2016",
       fill = "With and Without Bus",
       caption = "Galway City Council Traffic Data") +
  
  theme_classic() +
  
  theme(
        axis.ticks.x = element_blank(),
        axis.line.y= element_line(colour = "black", size = 0.1),
        axis.title.x=element_blank(), 
        axis.text.x = element_text(size=9, face="bold"),
        axis.text.y = element_text(size=7),
        axis.title.y = element_text(size=9, face="bold"),
        legend.text = element_text(size=8),
        legend.key.size = unit(0.8,"line"),
        plot.title=element_text( hjust=0.00, face='bold', size=11)
        ) + 
  coord_flip()

before_after_2
```

<img src="Assignment_5_final_files/figure-markdown_github/befre_after-2.png" style="display: block; margin: auto;" />

### **4. Galway City Council Bike Lane Recommendations**

The data was taken from 'junction turning counts' excel in which the data was segregated for each route.

``` r
setwd("E:/New Volume/Academic/NUIG_College/Data_Visulization/Assignment_5/")

#### Moycullen To Galway City and University

Before_After_MG_C <- read_excel("Moycullen_To_Galway.xlsx", sheet=3)
Before_After_MG_C <- gather(Before_After_MG_C, key=Time_frame, value= Number, 2:3)
Before_After_MG_C$Time <- as.hms(Before_After_MG_C$Time)

before_and_after_c_1 <- ggplot(Before_After_MG_C, aes(x = factor(format(Time, format = "%b")), y=Number, fill=Time_frame)) +
  geom_bar(position="dodge", stat="identity")  +
  scale_y_continuous(name = "Number of Uploads", 
                     limits=c(0,140), breaks=c(0,20,40,60,80,100,120,140)) + 
  scale_fill_manual(name="Colour Tag",labels = c("Without Bicycles", "With Bicycles"),
                    values = c("#0072b2", "#D55E00"),guide = guide_legend(reverse = TRUE)) +
    labs(
    x = "", y = "Vehicle Count",
    title = "Figure 4.1 | Before and After Cycle Lane Traffic Distribution",
    subtitle = "Moycullen To Galway City and University | 22nd November 2016",
       fill = "With and Without Bicycles",
       caption = "Galway City Council Traffic Data") +
  theme_classic() +
  theme(
        axis.ticks.x = element_blank(),
        axis.line.y= element_line(colour = "black", size = 0.1),
        axis.title.x=element_blank(), 
        axis.text.x = element_text(size=9, face="bold"),
        axis.text.y = element_text( size=7),
        axis.title.y = element_text(size=9, face="bold"),
        legend.text = element_text(size=8),
        legend.key.size = unit(0.8,"line"),
        plot.title=element_text( hjust=0.00, face='bold', size=11)
        ) + 
  coord_flip()


####Galway City and University To Moycullen

Before_After_MG_C <- read_excel("Moycullen_To_Galway.xlsx", sheet=2)
Before_After_MG_C <- gather(Before_After_MG_C, key=Time_frame, value= Number, 2:3)
Before_After_MG_C$Time <- as.hms(Before_After_MG_C$Time)

before_and_after_c_2 <- ggplot(Before_After_MG_C, aes(x = factor(format(Time, format = "%b")), y=Number, fill=Time_frame)) +
  geom_bar(position="dodge", stat="identity")  +
  scale_y_continuous(name = "Number of Uploads", 
                     limits=c(0,110), breaks=c(0,10,20,30,40,50,60,70,80,90,100,110)) + 
  scale_fill_manual(name="Colour Tag",labels = c("Without Bicycles", "With Bicycles"),
                    values = c("#0072b2", "#D55E00"),guide = guide_legend(reverse = TRUE)) +
     labs(
    x = "", y = "Vehicle Count",
    title = "Figure 4.2 | Before and After Cycle Lane Traffic Distribution",
    subtitle = "Galway City and University To Moycullen | 22nd November 2016",
       fill = "With and Without Bicycles",
       caption = "Galway City Council Traffic Data") +
  
  theme_classic() +
  
  theme(
        axis.ticks.x = element_blank(),
        axis.line.y= element_line(colour = "black", size = 0.1),
        axis.title.x=element_blank(), 
        axis.text.x = element_text(size=9, face="bold"),
        axis.text.y = element_text( size=7),
        axis.title.y = element_text(size=9, face="bold"),
        legend.text = element_text(size=8),
        legend.key.size = unit(0.8,"line"),
        plot.title=element_text(hjust=0.00, face='bold', size=11)
        ) + 
  coord_flip()
```

#### **Recommendations**

As per the initial description, we can make out that the route does not have a bicycle path and is narrow with sharp turns. Adding to it from figures 3.7 and 3.8, it can be seen that only a small fraction of commuters prefer travel by bi-cycle.

**Assumptions:**

1.  Around 40% of the total daily commuters will be ready to travel by bi-cycle, if given the choice.
2.  The is only one person travelling in the whole car while commuting.
3.  The cost of constructing and maintaining a bi-cycle lane is profitable for the government.
4.  The cycle lane will add to road safety.
5.  There will be a shower facility at every office.
6.  Companies will promote this move in order to bring the carbon footprint down by sharing the cost of the bi-cycle with the employees and helping the government in maintaining the lanes in their area.

**Before and After Bicycle lane Plots:**

1.  As per figure 4.1 and 4.2, we can observe that even if 40% of the total commuters during peak hours commute by bi-cycle, there is a significant reduction noticed in the count of vehicles on both the routes.
2.  As we can observe from fig. 3.9 and 3.10, the buses have significantly brought the number of vehicles down during traffic hours. The addition of bi-cycle lane will be benifical to decrease the load on bus transport by sharing the traffic and bringing down the number of buses required.

Therefore as per the facts presented by data, the introduction of a cycle lane will help to promote the improvement in traffic conditions on n59 and to bring down the load on bus transport.

``` r
before_and_after_c_1
```

<img src="Assignment_5_final_files/figure-markdown_github/cycle_plots-1.png" style="display: block; margin: auto;" />

``` r
before_and_after_c_2
```

<img src="Assignment_5_final_files/figure-markdown_github/cycle_plots-2.png" style="display: block; margin: auto;" />

**References:**
---------------

\[1\] Bic.asn.au. (2019). Climate Change and Public Transport. \[online\] Available at: <http://bic.asn.au/information-for-moving-people/climate-change-and-public-transport> \[Accessed 28 Mar. 2019\].
