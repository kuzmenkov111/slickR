# slickR
slick carousel htmlwidget for R

![](https://raw.githubusercontent.com/yonicd/slickR/master/slickRexample.gif)

## Examples for script generated plots

```r
library(svglite)
library(lattice)
library(ggplot2)
library(slickR)

s.in=sapply(
  list(
     xmlSVG({plot(1:10)},standalone=TRUE)
    ,xmlSVG({print(xyplot(x~x,data.frame(x=1:10),type="l"))},standalone=TRUE)
    ,xmlSVG({show(ggplot(iris,aes(x=Sepal.Length,y=Sepal.Width,colour=Species))+geom_point())},standalone=TRUE)
    ,xmlSVG({
      print(
        dotplot(variety ~ yield | site , data = barley, groups = year,
                key = simpleKey(levels(barley$year), space = "right"),
                xlab = "Barley Yield (bushels/acre) ",
                aspect=0.5, layout = c(1,6), ylab=NULL)        
      )
    },standalone=TRUE
    )
  )
  ,function(sv){
    paste0(
      "data:image/svg+xml;utf8,"
      ,as.character(sv)
    )
  }
)
  

slickR(s.in,slideId = 'baseDiv',slideIdx = list(1:3))

slickR(s.in,slideId = c('up','down'),
       slideIdx = list(1:3,1:3),
       synchSlides = c('up','down'),
       slickOpts = list(list(slidesToShow=1,slidesToScroll=1),
                        list(dots=T,slidesToScroll=1,slidesToShow=3,
                             centerMode=T,focusOnSelect=T)
                        )
       )
  
```

##Working with Dots and multiple sliders

Reading in the starting five for each team and showing them in the slider 5 at a time. We'll show the defaults dots attribute, replacing the dots with numbers and finally replacing the dots with icons (thanks to [\@ramnathv](https://github.com/ramnathv) for guidance). 

![](https://raw.githubusercontent.com/yonicd/slickR/master/slickRdotIconsexample.gif)

```r
library(slickR)
library(rvest) 
library(reshape2)
library(dplyr)
library(htmlwidgets)

a=c("ATL","BOS","BKN","CHA","CHI","CLE","DAL","DEN","DET","GSW",
    "HOU","IND","LAC","LAL","MEM","MIA","MIL","MIN","NOP","NYK",
    "OKC","ORL","PHI","PHX","POR","SAC","SAS","TOR","UTA","WAS")
teamImg=sprintf("https://i.cdn.turner.com/nba/nba/.element/img/4.0/global/logos/512x512/bg.white/svg/%s.svg",a)

a1=read_html('http://www.espn.com/nba/depth')%>%html_nodes(css = '#my-teams-table a')
a2=a1%>%html_attr('href')
a3=a1%>%html_text()
team_table=read_html('http://www.espn.com/nba/depth')%>%html_table()
team_table=team_table[[1]][-c(1,2),]
playerTable=team_table%>%melt(,id='X1')%>%arrange(X1,variable)
playerName=a2[grepl('[0-9]',a2)]
playerId=do.call('rbind',lapply(strsplit(playerName,'[/]'),function(x) x[c(8,9)]))
playerId=playerId[playerId[,1]!='phi',]
playerTable$img=sprintf('http://a.espncdn.com/combiner/i?img=/i/headshots/nba/players/full/%s.png&w=350&h=254',playerId[,1])


cP1=JS("function(slick,index) {return '<a>'+(index+1)+'</a>';}")
cP2=JS("function(slick,index) {return '<a><img src= ' + dotImages[index] + '  width=100% height=100%></a>';}")

#Defaults Dots
slickR(
  images = playerTable$img,
  slickOpts = list(
    initialSlide = 0,
    slidesToShow = 5,
    slidesToScroll = 5,
    focusOnSelect = T,
    dots = T
  )
)

#Replace dots with Numbers
slickR(
  images = playerTable$img,
  dotImages = teamImg,
  slickOpts = list(
    initialSlide = 0,
    slidesToShow = 5,
    slidesToScroll = 5,
    focusOnSelect = T,
    dots = T,
    customPaging = cP1
  )
)

#Replace dots with Images
s1 <- slickR(
  images = playerTable$img,
  dotImages = teamImg,
  slickOpts = list(
    initialSlide = 0,
    slidesToShow = 5,
    slidesToScroll = 5,
    focusOnSelect = T,
    dots = T,
    customPaging = cP2
  )
)


#Putting it all together in one slickR call
s2 <- htmltools::tags$script(
  sprintf("var dotImages = %s", 
          jsonlite::toJSON(teamImg))
)

htmltools::browsable(htmltools::tagList(s2, s1))

s3 <- slickR(
  images = rep(playerTable$img,3),
  slideId = c('dots','numbers','icons'),
  slideIdx = list(1:150,151:300,301:450),
  dotImages = teamImg,
  slickOpts = list(list(
    initialSlide = 0,slidesToShow = 5,
    slidesToScroll = 5,focusOnSelect = T,
    dots = T),
  list(
    initialSlide = 0, slidesToShow = 5,
    slidesToScroll = 5, focusOnSelect = T,
    dots = T, customPaging = cP1),
  list(
    initialSlide = 0, slidesToShow = 5,
    slidesToScroll = 5, focusOnSelect = T,
    dots = T, customPaging = cP2)
),height='0px')

htmltools::browsable(htmltools::tagList(s2, s3))

```

