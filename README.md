### Detecting Crop Harvest Using Sentinel Imagery | Zonal Statistics in R | David Smith | IDCE30274 Finial Project | 26 May 2021. 

This repository contains the README explanation of accessing Sentinial-1 Radar imagery for free using Amazon Web Services (AWS), calculating zonal using R, and a quick start guide for using Tableau to display excel data. The README will walk a user through data download, data preparation, zonal statistics using R, and loading excel files into Tableau for graphic display. A Command Line Interface (CLI) script for browsing AWS buckets, two examples .tiff files, R script, study area .shp file, and excel workbook are contained in this [google drive](https://drive.google.com/drive/folders/1eB11vdeKu7GY5kuYM66aIo1W-ajp_c0q?usp=sharing). 

## What is Sentinel- 1 imagery, and why is it important?
[<img width="701" alt="Screen Shot 2021-05-28 at 3 20 07 PM" src="https://user-images.githubusercontent.com/73979215/120032509-4b24cf00-bfc8-11eb-868c-9bf9fbbb6a6b.png">](https://www.esa.int/Enabling_Support/Operations/Sentinel-1_operations)

Sentinel 1 is a part of the European Space Agency’s satellite suite. Launched in 2014, Sentinel 1 was the first mission of the European Union’s Copernicus Programme and activated the public use of moderate resolution radar imagery. Onboard is a C- band duel Polarized Synthetic Aperture Radar that provides ground images in all light and weather conditions. Sentinel -1 C-band radar imagery is an active remote sensing platform that measures the amount of backscatter from a surface in decibels (dB). Backscatter is a measure of how smooth in texture a surface is; the higher the value in backscatter, the more edges and reflection points a surface has, i.e., rough in texture, while the lower the value in backscatter, the smoother a surface. C-band imagery has particular application in the agricultural industry, and recent interest has explored the use of radar to detect crop harvesting. The idea is that a field of, say, corn or soy will have a more significant amount of backscatter before being harvested than after harvest (Kavats et al., 2019). With enough training data on the difference in backscatter values, an analyst can train a neural network to detect crop harvest in real-time, over many Sentinel scenes, in any weather condition. Agricultural consulting firms can then use the harvest data to provide feedback to farmers concerning market value and harvest timing and other data-influenced suggestions. This tutorial will walk you through this concept, from data downloading to conducting analysis.  

## Tutorial Summary 
In this tutorial, we will be downloading Sentinel -1 C – Band radar imagery hosted in AWS for public use by [Indigo Ag](indigoag.com). We will then calculate the change in median backscatter value for a cornfield before and after harvest to see if our hypothesis holds. Hypothesis: *If there is a significant change in backscatter values of a crop surface, this should indicate the harvesting of the crop.* Finally, we’ll walk through how to load our resulting excel files into Tableau for clean graphic display.

## Software Needed   
The software needed for this tutorial is:    
•	A copy of R Statistical Analysis Software (I use the free [R Studio](https://www.rstudio.com/products/rstudio/)).   
•	ArcGIS Pro or QGIS (Optional for viewing your scene imagery).      
•	Tableau.      

To download R Studio, visit (https://www.rstudio.com/products/rstudio/), download the .zip file, and follow through with the R Studio set up wizard. If you would like further explanation or help setting up R, check out this Github blog post by [Software Carpentry](https://jennybc.github.io/2014-05-12-ubc/r-setup.html). 

For a free on-year student license to Tableau, follow this link (https://www.tableau.com/academic/students). If you’re not a student – don’t worry about getting Tableau. Excel makes good graphs on the cheap! I wanted to include Tableau here to help those with access but not the know-how. 

## Data 
In this tutorial, we will use three sets of example data contained in the provided data folder ` harvest_detection_data.zip` :
1.	Sentinel .tiffs from October 2020, C-Band radar imagery.   
2.	A study area .shp file.  
3.	An excel spreadsheet of VV polarization and VH polarization median values (explained later on).  

#### Geo tiffs 
Our imagery is accessed through AWS CLI cloud shell; you’ll need a free [AWS](https://aws.amazon.com/free/?trk=ps_a131L0000085DvcQAE&trkCampaign=acq_paid_search_brand&sc_channel=ps&sc_campaign=acquisition_US&sc_publisher=google&sc_category=core&sc_country=US&sc_geo=NAMER&sc_outcome=acq&sc_detail=aws%20account&sc_content=Account_e&sc_segment=432339156165&sc_medium=ACQ-P|PS-GO|Brand|Desktop|SU|AWS|Core|US|EN|Text&s_kwcid=AL!4422!3!432339156165!e!!g!!aws%20account&ef_id=CjwKCAjwqcKFBhAhEiwAfEr7zUchttWpTWHrMuxNXpe0JNMcbMdZKzzcnUIpel-q4XtLkmGSBfRjEhoC5WEQAvD_BwE:G:s&s_kwcid=AL!4422!3!432339156165!e!!g!!aws%20account&all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all) account to access the cloud and imagery buckets. Provided [here]( https://sentinel-s1-rtc-indigo-docs.s3-us-west-2.amazonaws.com/index.html) is documentation on imagery access. I have also included a .txt file with the needed command line inputs to access the imagery used in this tutorial.  


#### Study area .shp file 
I have selected a friend’s cornfield in New Mexico, drawn a polygon in ArcGIS pro around the cornfield, and added a date of harvest attribute. The date of the harvest will help us select our imagery and test our hypothesis. 

#### Excel Spreadsheet 
I most often interact with tables for graphic display in excel, but we will simply use the provided excel spreadsheet as a launching point into Tableau. 

## Getting Started
The first thing is first, let's set up our data folder. Start by downloading the harvest_detection_data folder contained in this [google drive](https://drive.google.com/drive/folders/1eB11vdeKu7GY5kuYM66aIo1W-ajp_c0q?usp=sharing). This folder includes:

•	Two .tiff images.    
•	One R script.  
•	A shapefile and associated metafiles.   
•	A .txt file.   
•	An excel spreadsheet  

Your data environment should look something like this. 

<img width="593" alt="Screen Shot 2021-05-29 at 4 30 39 PM" src="https://user-images.githubusercontent.com/73979215/120084063-3cf3b300-c09b-11eb-8c1d-1577053d304a.png">.    

If you decide you want to try and download the Sentinel imagery yourself, read on, or skip this next section and go right to calculating zonal statistics in R using the data provided. 

#### Image access through AWS 
Once you've made a free account with AWS, use the search bar at the top of the webpage to search for "CLI Cloud Shell" and click its link. A new page will open that looks relatively blank; wait for the connection to be established. 
(1)	paste `aws s3 ls s3://sentinel-s1-rtc-indigo/tiles/RTC/1/IW/13/S/CU/2020/` into the command line. A list of images from location UTM 13S, tile CU in 2020 will display. Notice the `ls` command in the line of code; this tells AWS to list the items inside the bucket. We're looking for an image before the 10/16/2020 harvest date and for one after. Looking through the list of images, I see `S1B_20201014_13SCU_ASC` and `S1B_20201019_13SCU_ASC`. Let's get both of these for our analysis. 
(2)	Copy the name of one of the images, then push the up arrow on your keyboard to insert the last command you typed and paste the image name after `/2020/`, add a backslash to the line that follows the image name you pasted, and press enter. More options will display; we have to choose between the gamma VH or gamma VV polarization. Khabbazan et al., 2019 explains why VV polarization is best for monitoring crops; let's grab the gamma_VV image.
(3)	Press the up arrow, copy the gamma0_VV.tif image name, paste it behind `S1B_20201014_13SCU_ASC/` within the command line, and press enter. Nothing should happen. We don't want to use the `ls` command; instead, we want to use the command for downloading, which is `cp`. Press the up arrow on the keyboard, scroll to the left, change `ls` to `cp`, scroll back to the right and add the name you want to call the file to the end of the line by putting a space after `.tiff` and adding a file name that ends in .tiff. Your line should look like this
 ```
aws s3 cp s3://sentinel-s1-rtc-indigo/tiles/RTC/1/IW/13/S/CU/2020/S1B_20201014_13SCU_ASC/Gamma0_VV.tif 20201014_13SCU.tif
```
Press enter, and you’ll begin to see the download progress; when the file is ready to be downloaded, you’ll see the file name you entered as `./20201014_13SCU.tif` copy this name and navigate to the AWS options tab in the top right of the page, click download file and paste `./20201014_13SCU.tif` into the box. You will then be prompted to download the image onto your local drive. 
(4)	Once successful, you can press the up arrow on the keyboard and change the command line to the next image date we want to download by scrolling through and changing the 14 within the image name to 19; press enter and download this image. 
When both images are download to your local drive, place them both into the folder that contains the `isleta_UTM13.shp` and the R script. The Raw VV backscatter images will look link this, notice any features?   
<img width="616" alt="Screen Shot 2021-05-29 at 4 51 08 PM" src="https://user-images.githubusercontent.com/73979215/120084571-5565cc80-c09f-11eb-9c5a-ad283bcb03d6.png">.  

Now we're ready for statistical analysis.

#### Zonal Statistics in R 
Open the R script provided in this tutorial and take a look around; it's short and sweet! Huge shout out to Su Ye at the University of Connecticut for helping me get this script up and running. I’ve since made adjustments to fit my needs by Su Ye got me going. Let’s take a look. 

1. We need to begin by importing a few different libraries; you’ll see them listed at the top of the script. The first library is `sf` or [simple features]( https://r-spatial.github.io/sf/) that will help us place our study area in the correct location for analysis. Then, we’ll add the `dplyr` [library]( https://www.rdocumentation.org/packages/dplyr/versions/0.7.8) , which lets us use a few verb conventions, later on, to add a new field to the resulting shapefile. Last on the library list is `raster`. Raster is the most important library because it enables us to run statistics on our raster .tiff images. Unfortunately, there isn’t much documentation on this [library]( https://www.rdocumentation.org/packages/raster/versions/3.4-10). In case you’re following along, copy this code block to install these packages. 
```
library(sf)
library(dplyr)
library(raster)
```
2. Now, we will establish our directories that house our files. The early directory should link the folder path to the sentinel image taken before harvesting. In our case, this will be the image `20201014_13SCU.tif`. The later directory should link the folder path to the sentinel image taken after harvest. In our case, this will be the image `20201019_13SCU.tif`. The polygon directory links the script to the shapefile we want to run statistics within, and the out directory identifies a new name for the resulting shapefile. I’ve chosen to name my output Isleta_1014_1019_new to reference the location and date range of comparison. The cornfield is on Isleta Street in Albuquerque, NM, so I’ve named the shapefile `Isleta_UTM13` to represent the location and projection of the shapefile contents. Your file paths will be different from mine, but I'll include abundance to see what an outline of yours should look like. **Notice the \\\\** .In your file paths, either add a backslash to the path you provide or change the backslash to a single forward slash. I prefer the double backslash.  
```
# change the below four lines to point to your local directory
early.dir <- 'F:\\spring 21\\programming\\Finialproject\\20201014_13SCU.tif'
late.dir <- 'F:\\spring 21\\programming\\Finialproject\\20201019_13SCU.tif'
poly.dir <- 'F:\\spring 21\\programming\\Finialproject\\Isleta_UTM13.shp'
out.dir <- 'F:\\spring 21\\programming\\Finialproject\\Isleta_1014_1019_new.shp'
```
3. Next, we’ll define our variables.
```
early.s1.raster <- raster(early.dir)
late.s1.raster <- raster(late.dir)
polys <- st_read(poly.dir)
```
4. Run the statistics: I’ve chosen to extract the median backscatter value instead of the mean or a different descriptive statistic. Outliers have less influence on medians, and for that reason, the median is more valuable for me to know and compare. 
```
early.vals <- extract(early.s1.raster, st_transform(polys,crs(early.s1.raster)), 
                      fun=median, na.rm=TRUE)
late.vals <- extract(late.s1.raster, st_transform(polys,crs(late.s1.raster)), 
                     fun=median, na.rm=TRUE)
```
Running the above lines of code will result in an error. 

When making feature classes in ArcPro, like the provided polygon, a vertical attribute is automatically generated and linked to the shapefile. The raster package we are using to extract zonal statistics in R only works on XY plane features. We need to coerce the shapefile into the XY plane by adding `st_zm` to our `polys` variable. Information on how I found this out is [here]( https://gis.stackexchange.com/questions/346855/remove-z-value-from-xyz-point-in-sf) on GIS Stack Exchange. 

```
early.vals <- extract(early.s1.raster, st_transform(st_zm(polys),crs(early.s1.raster)), 
                      fun=median, na.rm=TRUE)
late.vals <- extract(late.s1.raster, st_transform(st_zm(polys),crs(late.s1.raster)), 
                     fun=median, na.rm=TRUE)
```
5.  Finally, we’ll write our results into new fields within a new shapefile. 
```
new.polys <- st_zm(polys) %>% mutate(bfr_hr = early.vals) %>% 
  mutate(aft_hr = late.vals) %>% mutate(diffMed = early.vals - late.vals)
st_write(new.polys, out.dir, delete_dsn = TRUE)
```
Here’s the entire code block in case you need a clean start. 
```
library(sf)
library(dplyr)
library(raster)

# change the below four lines to point to your local file directory
early.dir <- 'F:\\spring 21\\programming\\Finialproject\\20201014_13SCU.tif'
late.dir <- 'F:\\spring 21\\programming\\Finialproject\\20201019_13SCU.tif'
poly.dir <- 'F:\\spring 21\\programming\\Finialproject\\Isleta_UTM13.shp'
out.dir <- 'F:\\spring 21\\programming\\Finialproject\\Isleta_1014_1019_new.shp'

#define variables 
early.s1.raster <- raster(early.dir)
late.s1.raster <- raster(late.dir)
polys <- st_read(poly.dir)
early.vals <- extract(early.s1.raster, st_transform(st_zm(polys),crs(early.s1.raster)), 
                      fun=median, na.rm=TRUE)
late.vals <- extract(late.s1.raster, st_transform(st_zm(polys),crs(late.s1.raster)), 
                     fun=median, na.rm=TRUE)

#write new shapefile
new.polys <- st_zm(polys) %>% mutate(bfr_hr = early.vals) %>% 
  mutate(aft_hr = late.vals) %>% mutate(diffMed = early.vals - late.vals)
st_write(new.polys, out.dir, delete_dsn = TRUE)
```
6. Check the folder you linked for the output and make sure you see a new shapefile. After running the last line of code, a GDAL error will pop up saying the new shapefile doesn’t appear to be a file or directory. The error isn’t a problem, and R will write the new file. What it’s telling you is there wasn’t a file with the same name to delete-  `delete_dsn` tells R to overwrite a file of the same output name; if you didn’t have one, it throws an error. 

## Migrate to ArcPro or QGIS to view statistics 
If you don’t have access to ArcPro or QGIS, I will include screenshots of the viewing results. 
1.  Open ArcPro or QGIS and start a new project. Define the home folder as our data folder we’ve worked from and open the ` Isleta_1014_1019_new.shp` shapefiles attribute table. It will look something like this. 
  
 <img width="508" alt="Screen Shot 2021-05-29 at 6 33 58 PM" src="https://user-images.githubusercontent.com/73979215/120086629-6f5adb80-c0ae-11eb-807b-e0bb658066ab.png">.  

The attribute field bfr_hr is the median backscatter value of the study area from before the harvest date, aft_hr is the median backscatter value from after the harvest date, and diffMed is the difference between bfr_hr and aft_hr. The value of diffMed is exciting! The positive value shows us there was a measurable decline in backscatter from before harvest to after harvest. The raw backscatter images look like this. Note: white pixels indicate higher backscatter than black ones.  

#### Before Harvest 10/14/2020
 <img width="813" alt="Screen Shot 2021-05-29 at 6 40 44 PM" src="https://user-images.githubusercontent.com/73979215/120086634-7d106100-c0ae-11eb-9b8f-d018d2acb9ca.png">.  


#### After Harvest 10/19/2020
 <img width="815" alt="Screen Shot 2021-05-29 at 6 41 20 PM" src="https://user-images.githubusercontent.com/73979215/120086660-b34de080-c0ae-11eb-8506-515348fc3017.png">.  

Here’s an image of the field itself. 

<img width="811" alt="Screen Shot 2021-05-29 at 6 53 02 PM" src="https://user-images.githubusercontent.com/73979215/120086692-2e16fb80-c0af-11eb-8f6a-3887bf0aedad.png">.  


Sometime between October 14th, 2020, and October 19th, 2020, farmers at Isleta harvested the cornfield. Since this is a control example to train a neural network to identify crop harvest, we know farmers at Isleta harvested the cornfield on October 16th, 2020. It appears our hypothesis is sound!

## Viewing data from excel using Tableau 
To train a neural network, we would need hundreds of training sites like this from multiple years. We should be sure our methods are sound before getting into the nitty-gritty of calculating median values for many fields. Let’s use Tableau to check the recommendation of Khabbazan et al. 2019 in using VV polarization instead of VH polarization using pretty graphics.
This tutorial includes a folder called VV_VH_compare, which contains an excel file of median backscatter values in both VV and VH polarization. We want to use the polarization that shows the most discernability from before and after harvest so we don’t waste our time! The median values calculated should show us the field was harvested between the image dates of 10/23/2018 and 10/30/2018. Let us check it out! 

1. Open Tableau and go to file <- new. Save the new workbook into the data folder with the VV_VH_compare excel workbook.
 
2. Click “New data source” in the top ribbon, and under “To File” select Microsoft Excel.    

 <img width="322" alt="Screen Shot 2021-05-29 at 7 29 00 PM" src="https://user-images.githubusercontent.com/73979215/120087420-c0220280-c0b5-11eb-8d16-9820e59508d4.png">.  


3. Navigate to where you’ve saved the provided .xls workbook titled “VV_VH_compare” and click OK. Inside this workbook are two sheets, one for VV statistics and one for VH statistics. Each sheet has dates of image acquisition, median backscatter values, change in median backscatter values, and the date range of comparison called “change type.   
Your Tableau window will look like this. 

<img width="1076" alt="Screen Shot 2021-05-29 at 7 34 38 PM" src="https://user-images.githubusercontent.com/73979215/120087427-cca65b00-c0b5-11eb-888d-3ac0efbb8c8f.png">.  
 

4. Drag VHstats and VVstats to the area that says “Drag tables here” and, when prompted, select the date fields as the connecting attribute between the two tables. The result will look like this. 

 <img width="1074" alt="Screen Shot 2021-05-29 at 7 36 51 PM" src="https://user-images.githubusercontent.com/73979215/120087429-d4fe9600-c0b5-11eb-959f-baf703f97a7a.png">.  

5.  At the bottom of the Tableau window, click on sheet1. 
6. On the left-hand side, you should see both of your sheets, VHstats, and VVstats, stacked on top of each other. 
7.  Drag Change Type for VHstats into the columns area and Change in Median for VHstats into the rows area. Our Graphic is coming together! 
8. Now, do the same for VVstats. Drag the Change Type for VVstats into the columns area and place it to the right of the VHstats. Then, drag the Change in Median for VVstats into the rows area and put it to the right of VHstats. Your frame will now look like this. 

<img width="1078" alt="Screen Shot 2021-05-29 at 7 52 21 PM" src="https://user-images.githubusercontent.com/73979215/120087838-94a11700-c0b9-11eb-8686-8c212cbae6a2.png">. 

9. Without doing anything else, we can see that the VV polarization is more sensitive to changes in the backscatter. This field was harvested on 10/17/2018, but you wouldn’t know that from looking at the VHstats. We can conclude the recommendation by Khabbazan et al., 2019 is correct and only collect the VV polarization images for crop monitoring from here on out. 

## Conclusion 
In this tutorial, we have learned how to download images using the AWS command-line interface. CLI took me some time to get used to, so don’t worry if it doesn’t come second nature off the cusp. Hopefully, this tutorial will help you out! Then we explored a new concept in remote sensing that uses radar imagery to detect changes in backscatter for application in the agricultural industry. Our initial working hypothesis was *If there is a significant change in backscatter values of a crop surface, this should indicate the harvesting of the crop.* From this preliminary analysis in R, it looks like this hypothesis could be a fact – that’s exciting going forward. Then we made sure our methods for approaching this hypothesis were sound by checking the discriminability of different data sources. We can see VV polarization is more sensitive to changes in backscatter when looking at crop surfaces so that we won’t waste our time with VH polarization for this application. A stakeholder would like to know they’ll be getting the best product, and you can show them a great graphic from Tableau. 
Happy Coding :) 

## Citations 
Kavats, O., Khramov, D., Sergieieva, K., & Vasyliev, V. (2019). Monitoring Harvesting by Time Series of Sentinel-1 SAR Data. Remote Sensing, 11(21), 2496. https://doi.org/10.3390/rs11212496

Khabbazan, S., Vermunt, P., Steele-Dunne, S., Ratering Arntz, L., Marinetti, C., van der Valk, D., Iannini, L., Molijn, R., Westerdijk, K., & van der Sande, C. (2019). Crop Monitoring Using Sentinel-1 Data: A Case Study from The Netherlands. Remote Sensing, 11(16), 1887. https://doi.org/10.3390/rs11161887

McNairn, H., & Brisco, B. (2004). The application of C-band polarimetric SAR for agriculture: A review. Canadian Journal of Remote Sensing, 30(3), 525–542. https://doi.org/10.5589/m03-069

Moran, M. S., Hymer, D. C., Qi, J., & Kerr, Y. (n.d.). Comparison of ERS-2 SAR and Landsat TM imagery for monitoring agricultural crop and soil conditions, Remote Sens. Environ, 2002.

Veloso, A., Mermoz, S., Bouvet, A., Le Toan, T., Planells, M., Dejoux, J.-F., & Ceschia, E. (2017).Understanding the temporal behavior of crops using Sentinel-1 and Sentinel-2-like data for agricultural applications. Remote Sensing of Environment, 199, 415–426. https://doi.org/10.1016/j.rse.2017.07.015




















