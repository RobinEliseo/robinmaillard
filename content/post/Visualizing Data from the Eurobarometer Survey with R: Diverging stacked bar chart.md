---
title: "Visualizing Data from the Eurobarometer Survey with R: Diverging Stacked Bar Chart"
author: "Robin Maillard"
date: "23 May 2020"
comments: yes
output:
  html_document:
    df_print: paged
  pdf_document: default
  word_document: default
header:
  caption: ''
  image: ''
profile: yes
reading_time: no
share: yes
tags:
- Eurobarometer
- Public Opinion
- R
- Data Visualisation
commentable: yes

---

In April, I found out a very interesting tutorial written by Dimiter Toshkov and published on his website.  
http://dimiter.eu/Visualizations_files/ESS/Visualizing_ESS_data.html   

The existence of such tutorials on open data and their visualisation are highly relevant today. Indeed, we live in a world where more and more data are available online but few people know how to deal with them. One of the main obstacle is, in my opinion, the lack of quick, easy and turnkey solutions in order to present those data.  
Following Dimiter, I've decided to adapt his work to the Eurobarometer survey. 

Earlier this month, I've published a post on my website on "How to correctly weight Eurobarometer data on R using the SPSS file?" available here:  
https://robinmaillard.netlify.app/post/how-to-correctly-weight-eurobarometer-data-on-r-using-the-spss-file/

The code presented in this article allows to produce a clean and correctly weighted dataframe of a question of the Eurobarometer survey with the SPSS file. 

```{r eval = FALSE} 
#Read the SPSS file with the labels
library(foreign)
dataSPSS<-read.spss("eb91_spss_en.sav", to.data.frame=TRUE)

library(tidyverse)

#Rename the countries with their ISO
dataSPSS <- dataSPSS %>% 
  mutate(b = recode(b, "LUXEMBOURG"="LU","DANMARK"="DK","NEDERLAND"="NL","SUOMI"="FI","ÖSTERREICH"="AT","DEUTSCHLAND WEST"="DE","CESKA REPUBLIKA"="CZ","EESTI"="EE","SVERIGE"="SE","IRELAND"="IE","DEUTSCHLAND OST"="DE","SLOVENIJA"="SI","POLSKA"="PL","MAGYARORSZAG"="HU","MALTA"="MT","BELGIQUE"="BE","TURKIYE"="TR","CRNA GORA"="ME","LIETUVA"="LT","KYPROS"="CY","PORTUGAL"="PT","LATVIA"="LV","SHQIPERIA"="AL","ROMANIA"="RO","SLOVENSKA REPUBLIC"="SK","SRPSKI"="RS","REPUBLIKA MAKEDONIJA"="MK","UNITED KINGDOM"="GB","BALGARIJA"="BG","FRANCE"="FR","HRVATSKA"="HR","ESPANA"="ES","ITALIA"="IT","ELLADA"="GR","KUZEY KIBRIS TÜRK CUMHURIYETI"="CY_tcc")) 

#Weighting of the data, and remove CY(tcc) and DE (wrong weighting here)
VolumeAA <- dataSPSS %>% 
  count(qa1a_1RGPS, b, wt = w1, sort = TRUE) %>% 
  spread(b, n) %>% 
  select(-qa1a_1RGPS) %>% 
  select(-CY_tcc) %>% 
  select(-DE)

#Compute DE with the right weithing
VolumeAA_DE <- dataSPSS %>% 
  count(qa1a_1RGPS, b, wt = w3, sort = TRUE) %>% 
  spread(b, n) %>% 
  select(-qa1a_1RGPS) %>% 
  select(DE)

#Add DE to dataframe used for the plot
VolumeAA$DE = VolumeAA_DE$DE

#Creation of the matrix for questionr 
VolumeA_matrix <- data.matrix(VolumeAA)
VolumeA_matrix[is.na(VolumeA_matrix)] <- 0

library(questionr)
#Create the matrix with the percentages
VolumeA_Perc<-questionr::cprop(VolumeA_matrix, digits=0, total=FALSE, n=FALSE, percent=TRUE)

## Transpose so that countries are rows, and make it a data frame (unexpectedly, it's not)
## Note that we need to use the special data.frame.matrix() function and not just data.frame()
VolumeA_Perc2<-as.data.frame.matrix(t(VolumeA_Perc)) 
names(VolumeA_Perc2) <- c("Total_Good", "Total_Bad", "DK")
VolumeA_Perc2 <- VolumeA_Perc2 %>% 
  select(-DK) 
VolumeA_Perc2 <- VolumeA_Perc2[order(VolumeA_Perc2$Total_Good, decreasing = TRUE),]
```

This is the ouput we get from this code: a clean matrix with the weighted results of this question - here I've chosen the totals of the results.  

```
   Total_Good Total_Bad         DK
LU   93.45266  5.267743 1.27959358
NL   92.48828  7.369534 0.14218636
FI   90.38298  8.044121 1.57289565
DK   90.24181  8.769949 0.98824382
AT   83.26907 15.451636 1.27929325
SE   81.46648 18.230996 0.30252235
DE   78.05275 20.810070 1.13717827
MT   76.20189 16.807272 6.99084001
EE   74.20940 22.425339 3.36526004
IE   74.03719 24.635998 1.32681386
CZ   67.60729 30.873856 1.51885171
PL   67.38727 27.311859 5.30086788
BE   67.03006 32.606737 0.36320678
SI   64.52626 34.109190 1.36455238
LT   61.38024 36.449714 2.17004215
HU   60.11313 38.922472 0.96440260
SK   52.43486 44.663066 2.90207473
ME   51.28096 44.836791 3.88224889
CY   49.96497 49.195507 0.83952309
LV   48.51482 46.947289 4.53789550
AL   47.07051 52.929488 0.00000000
MK   46.70557 51.546679 1.74774605
PT   44.48121 53.524681 1.99410751
TR   40.01730 58.510491 1.47220544
RS   39.30711 56.984344 3.70854747
RO   36.16685 62.128999 1.70415327
FR   35.53125 62.250376 2.21837227
GB   33.60111 60.635439 5.76345257
ES   32.41223 66.082274 1.50549201
BG   29.37130 65.782597 4.84610233
IT   26.58340 71.781431 1.63516521
HR   24.59170 73.858967 1.54933551
GR   14.41550 85.491345 0.09315223
```

At this point, we "only" have to apply the various steps presented by Dimiter. I will not comment this part of the code as it is mainly an adaptation of Dimiter's work: if you want to enter into details of the code I strongly advise you to go on his website - see the link above - and follow his great tutorial. 

The code for the small size PNG:
```
png ('./figures/F1_small_eb91.png', width=1280, height=906, res=96)

library(emojifont)

library(png)
temp.flag <- png::readPNG('./197373-countrys-flags/png/albania.png')
dim(temp.flag)
## [1] 512 512   4

## Function to place images by center points on the two axes rather than corners by Stack Overflow user 'Marc in the box', retrieved from: https://stackoverflow.com/questions/27800307/adding-a-picture-to-plot-in-r
addImg <- function(
  obj, # an image file imported as an array (e.g. png::readPNG, jpeg::readJPEG)
  x = NULL, # mid x coordinate for image
  y = NULL, # mid y coordinate for image
  width = NULL, # width of image (in x coordinate units)
  interpolate = TRUE # (passed to graphics::rasterImage) A logical vector (or scalar) indicating whether to apply linear interpolation to the image when drawing. 
){
  if(is.null(x) | is.null(y) | is.null(width)){stop("Must provide args 'x', 'y', and 'width'")}
  USR <- par()$usr # A vector of the form c(x1, x2, y1, y2) giving the extremes of the user coordinates of the plotting region
  PIN <- par()$pin # The current plot dimensions, (width, height), in inches
  DIM <- dim(obj) # number of x-y pixels for the image
  ARp <- DIM[1]/DIM[2] # pixel aspect ratio (y/x)
  WIDi <- width/(USR[2]-USR[1])*PIN[1] # convert width units to inches
  HEIi <- WIDi * ARp # height in inches
  HEIu <- HEIi/PIN[2]*(USR[4]-USR[3]) # height in units
  rasterImage(image = obj, 
              xleft = x-(width/2), xright = x+(width/2),
              ybottom = y-(HEIu/2), ytop = y+(HEIu/2), 
              interpolate = interpolate)
}

library(countrycode)

library(extrafont) # to embed extra fonts
library(sysfonts) # to check available fonts and download fonts from google
library(showtext) # to use the extra fonts

font_add_google('Quattrocento') #get the fonts 
font_add_google('Quattrocento Sans')
font_families() #check that the fonts are installed and available

showtext_auto() #this is to turn on the custom fonts availability
showtext_opts(dpi = 96) #set the resolution: 96 is default

####Improving the visualization
#Color palette
## color settings
background.color = rgb(248, 244, 255, max=255) # color for the background: magnolia
dark.color = rgb(24, 24, 38, max=255) # dark color: almost black

red.1 = rgb(228, 32, 50, max=255) # default red (Pantone)
blue.1 = rgb(11, 97, 146, max=255) # default blue (Pantone)

blue.twitter = rgb (29, 161, 242, max=255) # twitter blue

#Axes
library(plyr)
y.min <- plyr::round_any(max(VolumeA_Perc2[, 'Total_Bad']), accuracy = 10, f = ceiling)
y.max <- plyr::round_any(max(VolumeA_Perc2$Total_Good), accuracy = 10, f = ceiling)
#Titles
offset = 0.01 # distance from the corners of the plotting area
mtext.title = 1.8 # scaling factor for the font size
mtext.subtitle = 1.5

par(mfrow=c(1,1), # number and distribution of plots
    oma=c(1,0,3,0), # size of the outer margins in lines of text (can be specified in inches as well with `omi`)
    mar=c(1,4,1,1), # number of lines of margin to be specified on the four sides of the plot (can be specified in inches as well with `mai`) 
    bty='n', # no box
    cex = 1.25, # magnification of text and symbols
    xpd = FALSE, # clipping of plotting to the figure region
    ann = FALSE, # switch off titles,
    #yaxt = 'n', # switch off y axis, do not do this here, it cannot be overriden with axis() 
    #xaxt = 'n', # switch off x axis
    bg=background.color, # background color
    family='Quattrocento' # font family
)

#visualisation the data first cut
plot(NULL, # start with an empty plot
     xlim=c(1, dim(VolumeA_Perc2)[1]), ylim=c(-y.min, y.max), yaxt = 'n', xaxt = 'n') # the x axis should extend from 1 to the nubmer of rows in the data
ylim=c(-max(VolumeA_Perc2[, "Total_Bad"]), max(VolumeA_Perc2$Total_Good)) # the y axis should extend from the 
# negative of the maximum of the 'Total_Godd' category to the maximum of Total_Good 

# specify vertical axis
axis (2, # this indicates which axis to draw: 1 is bottom and it goes clockwise from there, so 2 is left
      line = 0, # position in terms of the plotting region
      #lwd = 1, # width of axis lines
      tck = -0.01, # length of axis tick marks
      col = dark.color, # color of the actual axis (line) 
      col.axis = dark.color, # colors of the actual labels
      cex.axis = 1, # font size of of the axis lables
      font=2, # font type (bold)
      at=seq(-y.min, y.max, 10), # where to put the labels  
      labels= paste0(c(rev(seq(0, y.min, 10)), seq(10, y.max,10)), "%"), # text of labels 
      las=1 # orientation of the labels
)

for (i in 1:dim(VolumeA_Perc2)[1]){ # for each row in the data
  rect(xleft = i - 0.25, xright = i + 0.25, ybottom = 0 - VolumeA_Perc2[i , "Total_Bad"], ytop =  0, col=red.1) # plot a red rectangle going down from zero for the 'Allow none' category
  rect(xleft = i - 0.25, xright = i + 0.25, ybottom = 0, ytop =  0 + VolumeA_Perc2$Total_Good[i], col=blue.1) # plot blue rectangles going up for the rest of the categories 
}

#Gridlines
abline(h=seq(-100,100,10), col='white', lwd=1)
abline(h=0, col='white', lwd=3)

for (i in 1:dim(VolumeA_Perc2)[1]){ # for each row in the data
  text (rownames(VolumeA_Perc2)[i], x = i, y = 0 + VolumeA_Perc2$Total_Good[i] + 4) # add the names of the countries above each set of bars
  
  addImg(readPNG(paste0('./197373-countrys-flags/png/', 
                        gsub(" ","-", tolower(countrycode(rownames(VolumeA_Perc2)[i], origin = 'iso2c', destination = 'country.name'))), 
                        '.png')), 
         x = i, y =  0 - VolumeA_Perc2[i , 'Total_Bad'] - 7, width = 0.6)
}

#title
mtext(expression(bold("The situation in (OUR COUNTRY) in general")), # the text
      side = 3, # on which side of the plot to include; 3 is top
      line = 2, # position of the text in terms of the figure region; 2 is two lines above the top of the figure
      adj = 0, # horizontal adjustment of the text; 0 is left
      padj = 1, # vertical adjustment of the text; 1 is top
      outer = TRUE, # whether the plot in the margins (outside the figure region)
      at = offset, # offset position from the corners of the plotting area
      font=1,  # font type
      col=dark.color,  # font color
      cex = mtext.title # font size
)

#legend
mtext(expression(italic("Share of people who answer:                   'Total Good'                   'Total Bad'")),
      side = 3, line = 0, adj = 0, padj = 1, outer = TRUE, at = offset, 
      font=1, col=dark.color, cex = mtext.subtitle)

par(xpd = TRUE) # turn on plotting outside the figure region
points(x = 8.5, y = 123, pch = 15, cex = 10.4, col=blue.1) # add small rectangles with the respective color
points(x = 8.5 + 7.18, y = 123, pch = 15, cex = 10.4, col=red.1)

#data statement
mtext.sign = 1.2
mtext.sign.emo = 1.5
mtext(text = fontawesome('fa-table'), side=1, line=-1, outer=T, adj=0, padj=0.8,
      col=red.1, cex=mtext.sign.emo, at = offset, font=1, family='fontawesome-webfont')

mtext(text=expression("Data: " * phantom("Standard Eurobarometer 91 (Sp. 2019)")), 
      side=1, line=-1, outer=T, at = offset + 0.03, col=dark.color, cex=mtext.sign, font=1, family='Quattrocento Sans', adj=0, padj=1)
mtext(text=expression(phantom("Data: ") * "Standard Eurobarometer 91 (Sp. 2019)"), 
      side=1, line=-1, outer=T, at = offset + 0.03, col=red.1, cex=mtext.sign, font=1, family='Quattrocento Sans', adj=0, padj=1)

#signature
mtext(text=expression(phantom("@R0binh093     ") * " https://robinmaillard.netlify" * phantom(".app")), 
      side=1, line=-1, outer=T, at = 1 - offset - 0.02, col=dark.color, cex=mtext.sign, font=1, family='Quattrocento Sans', adj=1, padj=1)

mtext(text=expression(phantom("@R0binh093      https://robinmaillard.netlify") * ".app"),
      side=1, line=-1, outer=T, at = 1 - offset - 0.02, col=red.1, cex=mtext.sign, font=1, family='Quattrocento Sans', adj=1, padj=1)

mtext(text=expression("@R0binh093     " * phantom(" https://robinmaillard.netlify.app")), 
      side=1, line=-1, outer=T, at = 1 - offset - 0.02, col=blue.twitter, cex=mtext.sign,font=1, family='Quattrocento Sans', adj=1, padj=1)

mtext(text= fontawesome('fa-twitter'), # the icon or emoji we want
      side=1, line=-1, outer=T, col=blue.twitter, cex=mtext.sign.emo, at = 1 - 0.24, adj=1, padj=0.8,
      font=1, family='fontawesome-webfont') # the Font Awesome font

mtext(text= fontawesome('fa-creative-commons'), 
      side=1, line=-1, outer=T, col=dark.color, cex=mtext.sign.emo, at = 1 - 0.35, font=1, family='fontawesome-webfont', adj=1, padj=0.8)

mtext(text= fontawesome('fa-rss'), 
      side=1, line=-1, outer=T, adj=1, padj=0.8, col=red.1, cex=mtext.sign.emo, at = 1 - offset, font=1, family='fontawesome-webfont')

#end of Small size PNG
dev.off()
```

The code for the big size PNG:
```
s = 3 # scaling factor
mtext.title = 2*s
mtext.subtitle = 1.5*s
mtext.sign = 1.2*s
mtext.sign.emo = 1.5*s

showtext_opts(dpi = 96) #set the resolution: 96 is default

png ('./figures/F1_big_eb91.png', width=1280*s, height=905.5*s, res=96)

par(mfrow=c(1,1), # number and distribution of plots
    oma=c(1,0,3,0), # size of the outer margins in lines of text (can be specified in inches as well with `omi`)
    mar=c(1,4,1,1), # number of lines of margin to be specified on the four sides of the plot (can be specified in inches as well with `mai`) 
    bty='n', # no box
    cex = 1.25*s, # magnification of text and symbols
    xpd = FALSE, # clipping of plotting to the figure region
    ann = FALSE, # switch off titles,
    #yaxt = 'n', # switch off y axis, do not do this here, it cannot be overriden with axis() 
    #xaxt = 'n', # switch off x axis
    bg=background.color, # background color
    family='Quattrocento' # font family
)

#visualisation the data first cut
plot(NULL, # start with an empty plot
     xlim=c(1, dim(VolumeA_Perc2)[1]), ylim=c(-y.min, y.max), yaxt = 'n', xaxt = 'n') # the x axis should extend from 1 to the nubmer of rows in the data
ylim=c(-max(VolumeA_Perc2[, "Total_Bad"]), max(VolumeA_Perc2$Total_Good)) # the y axis should extend from the 
# negative of the maximum of the 'Total_Godd' category to the maximum of Total_Good 

# specify vertical axis
axis (2, # this indicates which axis to draw: 1 is bottom and it goes clockwise from there, so 2 is left
      line = 0, # position in terms of the plotting region
      lwd = 1*s, # width of axis lines
      tck = -0.01, # length of axis tick marks
      col = dark.color, # color of the actual axis (line) 
      col.axis = dark.color, # colors of the actual labels
      cex.axis = 1, # font size of of the axis lables
      font=2, # font type (bold)
      at=seq(-y.min, y.max, 10), # where to put the labels  
      labels= paste0(c(rev(seq(0, y.min, 10)), seq(10, y.max,10)), "%"), # text of labels 
      las=1 # orientation of the labels
)

for (i in 1:dim(VolumeA_Perc2)[1]){ # for each row in the data
  rect(xleft = i - 0.25, xright = i + 0.25, ybottom = 0 - VolumeA_Perc2[i , "Total_Bad"], ytop =  0, col=red.1) # plot a red rectangle going down from zero for the 'Allow none' category
  rect(xleft = i - 0.25, xright = i + 0.25, ybottom = 0, ytop =  0 + VolumeA_Perc2$Total_Good[i], col=blue.1) # plot blue rectangles going up for the rest of the categories 
}

#Gridlines
abline(h=seq(-100,100,10), col='white', lwd=1*s)
abline(h=0, col='white', lwd=3*s)

#Flags
for (i in 1:dim(VolumeA_Perc2)[1]){ # for each row in the data
  text (rownames(VolumeA_Perc2)[i], x = i, y = 0 + VolumeA_Perc2$Total_Good[i] + 4, font = 2) # add the names of the countries above each set of bars
  
  addImg(readPNG(paste0('./197373-countrys-flags/png/', 
                        gsub(" ","-", tolower(countrycode(rownames(VolumeA_Perc2)[i], origin = 'iso2c', destination = 'country.name'))), 
                        '.png')), 
         x = i, y =  0 - VolumeA_Perc2[i , 'Total_Bad'] - 7, width = 0.6)
}

#title
mtext(expression(bold("The situation in (OUR COUNTRY) in general")), # the text
      side = 3, # on which side of the plot to include; 3 is top
      line = 2, # position of the text in terms of the figure region; 2 is two lines above the top of the figure
      adj = 0, # horizontal adjustment of the text; 0 is left
      padj = 1, # vertical adjustment of the text; 1 is top
      outer = TRUE, # whether the plot in the margins (outside the figure region)
      at = offset, # offset position from the corners of the plotting area
      font=1,  # font type
      col=dark.color,  # font color
      cex = mtext.title # font size
)

#legend
mtext(expression(italic("Share of people who answer:                   'Total Good'                   'Total Bad'")),
      side = 3, line = 0, adj = 0, padj = 1, outer = TRUE, at = offset, 
      font=1, col=dark.color, cex = mtext.subtitle)

par(xpd = TRUE) # turn on plotting outside the figure region
points(x = 8.3, y = 123, pch = 15, cex = 10.4, col=blue.1) # add small rectangles with the respective color
points(x = 8.3 + 6.95, y = 123, pch = 15, cex = 10.4, col=red.1)

#data statement
mtext(text = fontawesome('fa-table'), side=1, line=-1, outer=T, adj=0, padj=0.8,
      col=red.1, cex=mtext.sign.emo, at = offset, font=1, family='fontawesome-webfont')

mtext(text=expression("Data: " * phantom("Standard Eurobarometer 91 (Sp. 2019)")), 
      side=1, line=-1, outer=T, at = offset + 0.03, col=dark.color, cex=mtext.sign, font=1, family='Quattrocento Sans', adj=0, padj=1)
mtext(text=expression(phantom("Data: ") * "Standard Eurobarometer 91 (Sp. 2019)"), 
      side=1, line=-1, outer=T, at = offset + 0.03, col=red.1, cex=mtext.sign, font=1, family='Quattrocento Sans', adj=0, padj=1)

#signature
mtext(text=expression(phantom("@R0binh093     ") * " https://robinmaillard.netlify" * phantom(" .app")), 
      side=1, line=-1, outer=T, at = 1 - offset - 0.02, col=dark.color, cex=mtext.sign, font=1, family='Quattrocento Sans', adj=1, padj=1)

mtext(text=expression(phantom("@R0binh093      https://robinmaillard.netlify") * " .app"),
      side=1, line=-1, outer=T, at = 1 - offset - 0.02, col=red.1, cex=mtext.sign, font=1, family='Quattrocento Sans', adj=1, padj=1)

mtext(text=expression("@R0binh093     " * phantom(" https://robinmaillard.netlify .app")), 
      side=1, line=-1, outer=T, at = 1 - offset - 0.02, col=blue.twitter, cex=mtext.sign, font=1, family='Quattrocento Sans', adj=1, padj=1)

mtext(text= fontawesome('fa-twitter'), # the icon or emoji we want
      side=1, line=-1, outer=T, col=blue.twitter, cex=mtext.sign.emo, at = 1 - 0.243, adj=1, padj=0.8,
      font=1, family='fontawesome-webfont') # the Font Awesome font

mtext(text= fontawesome('fa-creative-commons'), 
      side=1, line=-1, outer=T, col=dark.color, cex=mtext.sign.emo, at = 1 - 0.353, font=1, family='fontawesome-webfont', adj=1, padj=0.8)

mtext(text= fontawesome('fa-rss'), 
      side=1, line=-1, outer=T, adj=1, padj=0.8, col=red.1, cex=mtext.sign.emo, at = 1 - offset, font=1, family='fontawesome-webfont')

#end of Large size PNG
dev.off()
```
The final product:
{{< figure library="true" src="F1_big_eb91.png" title="" lightbox="true" >}}

You are now able to present the results of most questions of Eurobarometer surveys by using this script.  
My code is also available on Github. 
