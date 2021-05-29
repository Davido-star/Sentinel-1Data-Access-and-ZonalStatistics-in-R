### Detecting Crop Harvest Using Sentinel Imagery | Zonal Statistics in R | David Smith | IDCE30274 Finial Project | 26 May 2021. 

This repository contains the README explanation of accessing Sentinial-1 Radar imagery for free using Amazon Web Services (AWS), running spatial statistics on the imagery using R, and a quick start guide for using Tableau to display your data. The README will walk a user through data download, data preparation, zonal statistics using R, and loading excel files into Tableau for graphic display. A Command Line Interface (CLI) script for browsing AWS buckets, two examples .tiff files, R script, and a study area .shp file are contained in this tutorial repository. 

## What is Sentinel- 1 imagery, and why is it important?
[<img width="701" alt="Screen Shot 2021-05-28 at 3 20 07 PM" src="https://user-images.githubusercontent.com/73979215/120032509-4b24cf00-bfc8-11eb-868c-9bf9fbbb6a6b.png">](https://www.esa.int/Enabling_Support/Operations/Sentinel-1_operations)

Sentinel 1 is a part of the European Space Agency’s satellite suite. Launched in 2014, Sentinel 1 was the first mission of the European Union’s Copernicus Programme and activated the public use of moderate resolution radar imagery. Onboard is a C- band duel Polarized Synthetic Aperture Radar that provides ground images in all light and weather conditions. Sentinel -1 C-band radar imagery is an active remote sensing platform that measures the amount of backscatter from a surface in decibels (dB). Backscatter is a measure of how smooth texture a surface is; the higher the value in backscatter, the more edges and reflection points a surface has, i.e., rough in texture, while the lower the value in backscatter, the smoother a surface. C-band imagery has particular application in the agricultural industry, and recent interest has explored the use of radar to detect crop harvesting. The idea is that a field of, say, corn or soy will have a more significant amount of backscatter before being harvested than after harvest (Kavats et al., 2019). With enough training data on the difference in backscatter values, an analyst can train a neural network to detect crop harvest in real-time, over many Sentinel scenes, in any weather condition. Agricultural consulting firms can then use the harvest data to provide feedback to farmers concerning market value and harvest timing and other data-influenced suggestions. This tutorial will walk you through this concept, from data downloading to conducting analysis.  

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
In this tutorial, we will use three sets of example data:  
1.	Two .tiffs of C-Band radar imagery.     
2.	A study area .shp file.    
3.	An excel spreadsheet.   

#### Geo tiffs 
Our imagery is accessed through AWS CLI cloud shell; you’ll need a free [AWS](https://aws.amazon.com/free/?trk=ps_a131L0000085DvcQAE&trkCampaign=acq_paid_search_brand&sc_channel=ps&sc_campaign=acquisition_US&sc_publisher=google&sc_category=core&sc_country=US&sc_geo=NAMER&sc_outcome=acq&sc_detail=aws%20account&sc_content=Account_e&sc_segment=432339156165&sc_medium=ACQ-P|PS-GO|Brand|Desktop|SU|AWS|Core|US|EN|Text&s_kwcid=AL!4422!3!432339156165!e!!g!!aws%20account&ef_id=CjwKCAjwqcKFBhAhEiwAfEr7zUchttWpTWHrMuxNXpe0JNMcbMdZKzzcnUIpel-q4XtLkmGSBfRjEhoC5WEQAvD_BwE:G:s&s_kwcid=AL!4422!3!432339156165!e!!g!!aws%20account&all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all) account to access the cloud and imagery buckets. 

#### Study area .shp file 
I have selected a friend’s cornfield in New Mexico, drawn a polygon in ArcGIS pro around the cornfield, and added a date of harvest attribute. The date of the harvest will help us select our imagery and test our hypothesis. 

#### Excel Spreadsheet 
I most often interact with tables for graphic display in excel, but we will simply use the provided excel spreadsheet as a launching point into Tableau. 




