
# **How are wealth and health related?**

## **Will Davis - IA626 Final Project - Spring 2024**

### **Premise**

In this project I explore the connections between wealth and health by comparing IRS tax return data and CDC health outcome data for the 2021 calendar year. Going into this project my hypothesis was that individuals and communities with more wealth would have less prevalence of negative health outcomes such as cancer, asthma, high blood pressure, and more. My reasoning was that these parties would have access to better medical treatment, have less stress, and generally have more healthy lifestyles due to their wealth. Conversely, poorer individuals and communities would have poorer medical treatment, worse nutrition options, higher stress, and generally unhealthy lifestyles due to their lack of resources.

### **Approach**

I have utilized two data sets in this project, both were found as U.S. government open data sets at [data.gov](https://data.gov/). Links to these data sets can be found here:

[https://catalog.data.gov/dataset/zip-code-data](https://catalog.data.gov/dataset/zip-code-data)  
[https://catalog.data.gov/dataset/places-local-data-for-better-health-zcta-data-2020-release-ea5f2](https://catalog.data.gov/dataset/places-local-data-for-better-health-zcta-data-2020-release-ea5f2)

#### **SOI Zip Code Data**

The first data set is the Statistics of Income (SOI) Division’s Zip Code Data - Tax Year 2021. The following is an overview of the data set included in the data guide:

>The Statistics of Income (SOI) Division’s ZIP code data is tabulated using individual income tax returns (Forms 1040) filed with the Internal Revenue Service (IRS)
during the 12-month period, January 1, 2022 to December 31, 2022. While the bulk of returns filed during this 12-month period are primarily for Tax Year 2021, the IRS
received a limited number of returns for tax years before 2021. These prior-year returns are used as a proxy for returns that are typically filed beyond the 12-month period
and have been included within the ZIP code data.

Additionally, some important definitions and qualifications included in the data guide:

>•	ZIP Code data are based on population data that was filed and processed by the IRS during the 2022 calendar year.   
•	Data do not represent the full U.S. population because many individuals are not required to file an individual income tax return.  
•	The address shown on the tax return may differ from the taxpayer’s actual residence.  
•	State codes were based on the ZIP code shown on the return.  
•	Tax returns filed without a ZIP code and returns filed with a ZIP code that did not match the State code shown on the return were excluded.  
•	Tax returns filed using Army Post Office (APO) and Fleet Post Office addresses, foreign addresses, and addresses in Puerto Rico, Guam, Virgin Islands, American Samoa, Marshall Islands, Northern Marianas, and Palau were excluded.
>
>SOI did not attempt to correct any ZIP codes listed on the tax returns; however, it did take the following precautions to avoid disclosing information about specific taxpayers:
>
>•	ZIP codes with less than 100 returns and those identified as a single building or nonresidential ZIP code were categorized as “other” (99999).  
•	Income and tax items with less than 20 returns for a particular AGI class were combined with another AGI class within the same ZIP Code. Collapsed AGI classes are identified with a double asterisk (**).  
•	All number of returns variables have been rounded to the nearest 10.  
•	Income and tax items with less than 20 returns within a ZIP code were excluded.  
•	Tax returns with a negative adjusted gross income were excluded.  

#### **CDC PLACES Health Data**

The second data set is the PLACES: Local Data for Better Health, ZCTA Data 2023 release from the Centers for Disease Control and Prevention (CDC). The following is an overview of the data set included in the data guide:

>This dataset contains model-based ZIP Code Tabulation Area (ZCTA) level estimates. PLACES covers the entire United States—50 states and the District of Columbia—at county, place, census tract, and ZIP Code Tabulation Area levels. It provides information uniformly on this large scale for local areas at four geographic levels. Estimates were provided by the Centers for Disease Control and Prevention (CDC), Division of Population Health, Epidemiology and Surveillance Branch. PLACES was funded by the Robert Wood Johnson Foundation in conjunction with the CDC Foundation. The dataset includes estimates for 36 measures: 13 for health outcomes, 9 for preventive services use, 4 for chronic disease-related health risk behaviors, 7 for disabilities, and 3 for health status. These estimates can be used to identify emerging health problems and to help develop and carry out effective, targeted public health prevention activities. Because the small area model cannot detect effects due to local interventions, users are cautioned against using these estimates for program or policy evaluations. Data sources used to generate these model-based estimates are Behavioral Risk Factor Surveillance System (BRFSS) 2021 or 2020 data, Census Bureau 2010 population data, and American Community Survey 2015–2019 estimates. The 2023 release uses 2021 BRFSS data for 29 measures and 2020 BRFSS data for 7 measures (all teeth lost, dental visits, mammograms, cervical cancer screening, colorectal cancer screening, core preventive services among older adults, and sleeping less than 7 hours) that the survey collects data on every other year. More information about the methodology can be found at [www.cdc.gov/places](https://www.cdc.gov/places/).

#### **Tools**

This project was coded almost entirely in Python with a small amount of JSON utilization. I used Python notebooks in the JupyterLab IDE to import, clean, transform, combine, and visualize the data.

### **Methodology**

To begin, I imported the IRS SOI Zip Code Data using the csv Python module. After consulting the data guide, I decided that the best way to represent community wealth was to determine the median adjusted gross income for each of the zip codes included in the data. This metric was already somewhat aggregated by SOI, with adjusted gross incomes being grouped into six categories under the "AGI_STUB" heading for each zip code. These categories are listed below:

* 1 = $1 under $25,000  
* 2 = $25,000 under $50,000  
* 3 = $50,000 under $75,000  
* 4 = $75,000 under $100,000  
* 5 = $100,000 under $200,000  
* 6 = $200,000 or more

Also included was the count of the number of tax returns submitted to the IRS with adjusted gross incomes in each category. I adjusted my initial hopes of finding median adjusted gross incomes in terms of dollars, and instead developed the following function to find the correct "AGI_STUB" category for each zip code:

```
def findMedianAGIStub(zip_dict):
    for zip in zip_dict:
        total = None
        index = 0
        try:
            total = zip_dict[zip]['1'] + zip_dict[zip]['2'] + zip_dict[zip]['3'] + zip_dict[zip]['4'] + zip_dict[zip]['5'] + zip_dict[zip]['6']
        except Exception as e:
            print('function error: ' + str(zip) + str(e))
            zip_dict[zip]['total'] = total
            zip_dict[zip]['median_index'] = total
            zip_dict[zip]['median_agi_stub'] = total
            continue
        zip_dict[zip]['total'] = total
        zip_dict[zip]['median_index'] = total / 2 # This value is rounded to ten, thus always even.
        if (total / 2) > zip_dict[zip]['1']:
            index += zip_dict[zip]['1']
        else:
            zip_dict[zip]['median_agi_stub'] = '1'
            continue
        if (total / 2) > zip_dict[zip]['2'] + index:
            index += zip_dict[zip]['2']
        else:
            zip_dict[zip]['median_agi_stub'] = '2'
            continue
        if (total / 2) > zip_dict[zip]['3'] + index:
            index += zip_dict[zip]['3']
        else:
            zip_dict[zip]['median_agi_stub'] = '3'
            continue
        if (total / 2) > zip_dict[zip]['4'] + index:
            index += zip_dict[zip]['4']
        else:
            zip_dict[zip]['median_agi_stub'] = '4'
            continue
        if (total / 2) > zip_dict[zip]['5'] + index:
            index += zip_dict[zip]['5']
        else:
            zip_dict[zip]['median_agi_stub'] = '5'
            continue
        zip_dict[zip]['median_agi_stub'] = '6'
    return zip_dict
```

This median "AGI_STUB" value and the corresponding index location of the same was stored in a dictionary, along with the initial tax return counts in each "AGI_STUB" category, and total tax return count by zip code. There were some zip codes that were missing counts for certain "AGI_STUB" categories. These gaps were addressed by inserting a "0" count for missing "AGI_STUB" categories. This allowed the function I had written to function for all zip codes, while not changing the resulting median adjusted gross income value:

```
# Patch solution - I am going to give each missing agi_stub category a 0.0 value. This should not change the median result, I'm not actually increasing the total returns.
for zip_key in zip_dict:
    if '1' not in zip_dict[zip_key]:
        zip_dict[zip_key]['1'] = 0.0
    if '2' not in zip_dict[zip_key]:
        zip_dict[zip_key]['2'] = 0.0
    if '3' not in zip_dict[zip_key]:
        zip_dict[zip_key]['3'] = 0.0
    if '4' not in zip_dict[zip_key]:
        zip_dict[zip_key]['4'] = 0.0
    if '5' not in zip_dict[zip_key]:
        zip_dict[zip_key]['5'] = 0.0
    if '6' not in zip_dict[zip_key]:
        zip_dict[zip_key]['6'] = 0.0

zip_dict = findMedianAGIStub(zip_dict)

for zip_key in zip_dict:
    if zip_key in ['95585','81029','83636','48109','87063','23708','53706']:
        print(zip_dict[zip_key])

output:
{'1': 70.0, '2': 40.0, '3': 20.0, '5': 0.0, '6': 0.0, 'total': 130.0, 'median_index': 65.0, 'median_agi_stub': '1', '4': 0.0}
{'1': 40.0, '2': 40.0, '3': 30.0, '4': 0.0, '6': 0.0, 'total': 110.0, 'median_index': 55.0, 'median_agi_stub': '2', '5': 0.0}
{'1': 40.0, '2': 30.0, '3': 30.0, '5': 0.0, '6': 0.0, 'total': 100.0, 'median_index': 50.0, 'median_agi_stub': '2', '4': 0.0}
{'1': 90.0, '2': 0.0, '3': 20.0, '5': 0.0, '6': 0.0, 'total': 110.0, 'median_index': 55.0, 'median_agi_stub': '1', '4': 0.0}
{'1': 50.0, '2': 60.0, '3': 0.0, '5': 0.0, '6': 0.0, 'total': 110.0, 'median_index': 55.0, 'median_agi_stub': '2', '4': 0.0}
{'1': 50.0, '2': 70.0, '3': 0.0, '5': 0.0, '6': 0.0, 'total': 120.0, 'median_index': 60.0, 'median_agi_stub': '2', '4': 0.0}
{'1': 80.0, '2': 30.0, '6': 0.0, 'total': 110.0, 'median_index': 55.0, 'median_agi_stub': '1', '3': 0.0, '4': 0.0, '5': 0.0}
```

Next, it was time to import the PLACES CDC data. This data set includes many varied health data, and it took some filtering to find the health outcomes categories I was looking for. Eventually, I found measures for 12 different health outcom metric as listed below:

* Arthritis among adults aged >=18 years  
* High blood pressure among adults aged >=18 years  
* Cancer (excluding skin cancer) among adults aged >=18 years  
* Current asthma among adults aged >=18 years  
* Coronary heart disease among adults aged >=18 years  
* Chronic obstructive pulmonary disease among adults aged >=18 years  
* Depression among adults aged >=18 years  
* Diagnosed diabetes among adults aged >=18 years  
* High cholesterol among adults aged >=18 years who have been screened in the past 5 years
* Chronic kidney disease among adults aged >=18 years  
* Obesity among adults aged >=18 years  
* Stroke among adults aged >=18 years  

I initially began working with 13 health outcome metrics, but later in the process I discovered that the "All teeth lost among adults aged >=65 years" metric did not include data from 2021. These metrics were extracted from the PLACES data set and stored in the same dictionary with the median adjusted gross income data. Stored data points included the measure, data value unit, data value type, and the data value itself. For efficiency and ease of access to the data later on, I used a nested dictionary approach. This was accomplished by using each zip code as a dictionary key, allowing both IRS and CDC data to be combined and stored for the same zip codes. An example of the final resulting dictionary entry for a given zip code is given below:

```
zip code: 35006
{'1': 370.0, '2': 340.0, '3': 200.0, '4': 150.0, '5': 170.0, '6': 30.0, 'total': 1260.0, 'median_index': 630.0, 'median_agi_stub': '2', 'Arthritis': {'Measure': 'Arthritis among adults aged >=18 years', 'Data_Value_Unit': '%', 'Data_Value_Type': 'Crude prevalence', 'Data_Value': '31.9'}, 'High_Blood_Pressure': {'Measure': 'High blood pressure among adults aged >=18 years', 'Data_Value_Unit': '%', 'Data_Value_Type': 'Crude prevalence', 'Data_Value': '39.1'}, 'Cancer': {'Measure': 'Cancer (excluding skin cancer) among adults aged >=18 years', 'Data_Value_Unit': '%', 'Data_Value_Type': 'Crude prevalence', 'Data_Value': '7.0'}, 'Asthma': {'Measure': 'Current asthma among adults aged >=18 years', 'Data_Value_Unit': '%', 'Data_Value_Type': 'Crude prevalence', 'Data_Value': '10.1'}, 'Heart_Disease': {'Measure': 'Coronary heart disease among adults aged >=18 years', 'Data_Value_Unit': '%', 'Data_Value_Type': 'Crude prevalence', 'Data_Value': '7.0'}, 'Pulmonary_Disease': {'Measure': 'Chronic obstructive pulmonary disease among adults aged >=18 years', 'Data_Value_Unit': '%', 'Data_Value_Type': 'Crude prevalence', 'Data_Value': '8.4'}, 'Depression': {'Measure': 'Depression among adults aged >=18 years', 'Data_Value_Unit': '%', 'Data_Value_Type': 'Crude prevalence', 'Data_Value': '25.4'}, 'Diabetes': {'Measure': 'Diagnosed diabetes among adults aged >=18 years', 'Data_Value_Unit': '%', 'Data_Value_Type': 'Crude prevalence', 'Data_Value': '12.3'}, 'High_Cholesterol': {'Measure': 'High cholesterol among adults aged >=18 years who have been screened in the past 5 years', 'Data_Value_Unit': '%', 'Data_Value_Type': 'Crude prevalence', 'Data_Value': '38.3'}, 'Kidney_Disease': {'Measure': 'Chronic kidney disease among adults aged >=18 years', 'Data_Value_Unit': '%', 'Data_Value_Type': 'Crude prevalence', 'Data_Value': '3.1'}, 'Obesity': {'Measure': 'Obesity among adults aged >=18 years', 'Data_Value_Unit': '%', 'Data_Value_Type': 'Crude prevalence', 'Data_Value': '37.6'}, 'Stroke': {'Measure': 'Stroke among adults aged >=18 years', 'Data_Value_Unit': '%', 'Data_Value_Type': 'Crude prevalence', 'Data_Value': '3.4'}}
```

Finally, it was time to create visualizations from this combined data and to evaluate the results. I decided to use multiple box plots on a single plot, each plot visualizing data for one health outcome. On the x-axis I have each "AGI_STUB" category, using the original labels and not the numerical ones. On the y-axis, I have the crude prevalence of the health outcome in terms of percentage. This was accomplished using the matplotlib Python module, and specifically the pyplot module within this package. An example of the extraction of the data for each health outcome and the creation of the box plots is given below:

##### **Extracting Arthritis Data**

```
# Now that we have our data stored on disk, we can start to make some visualizations.
from matplotlib import pyplot

arth_agi_1 = []
arth_agi_2 = []
arth_agi_3 = []
arth_agi_4 = []
arth_agi_5 = []
arth_agi_6 = []

for zip_code in data:
    if data[zip_code]['median_agi_stub'] == '1' and data[zip_code]['Arthritis']['Measure'] == 'Arthritis among adults aged >=18 years' and data[zip_code]['Arthritis']['Data_Value_Unit'] == '%' and data[zip_code]['Arthritis']['Data_Value_Type'] == 'Crude prevalence':
        arthritis_perc = None
        try:
            arthritis_perc = float(data[zip_code]['Arthritis']['Data_Value'])
        except Exception as e:
            print(str(zip_code) + str(e))
            continue
        arth_agi_1.append(arthritis_perc)
    if data[zip_code]['median_agi_stub'] == '2' and data[zip_code]['Arthritis']['Measure'] == 'Arthritis among adults aged >=18 years' and data[zip_code]['Arthritis']['Data_Value_Unit'] == '%' and data[zip_code]['Arthritis']['Data_Value_Type'] == 'Crude prevalence':
        arthritis_perc = None
        try:
            arthritis_perc = float(data[zip_code]['Arthritis']['Data_Value'])
        except Exception as e:
            print(str(zip_code) + str(e))
            continue
        arth_agi_2.append(arthritis_perc)
    if data[zip_code]['median_agi_stub'] == '3' and data[zip_code]['Arthritis']['Measure'] == 'Arthritis among adults aged >=18 years' and data[zip_code]['Arthritis']['Data_Value_Unit'] == '%' and data[zip_code]['Arthritis']['Data_Value_Type'] == 'Crude prevalence':
        arthritis_perc = None
        try:
            arthritis_perc = float(data[zip_code]['Arthritis']['Data_Value'])
        except Exception as e:
            print(str(zip_code) + str(e))
            continue
        arth_agi_3.append(arthritis_perc)
    if data[zip_code]['median_agi_stub'] == '4' and data[zip_code]['Arthritis']['Measure'] == 'Arthritis among adults aged >=18 years' and data[zip_code]['Arthritis']['Data_Value_Unit'] == '%' and data[zip_code]['Arthritis']['Data_Value_Type'] == 'Crude prevalence':
        arthritis_perc = None
        try:
            arthritis_perc = float(data[zip_code]['Arthritis']['Data_Value'])
        except Exception as e:
            print(str(zip_code) + str(e))
            continue
        arth_agi_4.append(arthritis_perc)
    if data[zip_code]['median_agi_stub'] == '5' and data[zip_code]['Arthritis']['Measure'] == 'Arthritis among adults aged >=18 years' and data[zip_code]['Arthritis']['Data_Value_Unit'] == '%' and data[zip_code]['Arthritis']['Data_Value_Type'] == 'Crude prevalence':
        arthritis_perc = None
        try:
            arthritis_perc = float(data[zip_code]['Arthritis']['Data_Value'])
        except Exception as e:
            print(str(zip_code) + str(e))
            continue
        arth_agi_5.append(arthritis_perc)
    if data[zip_code]['median_agi_stub'] == '6' and data[zip_code]['Arthritis']['Measure'] == 'Arthritis among adults aged >=18 years' and data[zip_code]['Arthritis']['Data_Value_Unit'] == '%' and data[zip_code]['Arthritis']['Data_Value_Type'] == 'Crude prevalence':
        arthritis_perc = None
        try:
            arthritis_perc = float(data[zip_code]['Arthritis']['Data_Value'])
        except Exception as e:
            print(str(zip_code) + str(e))
            continue
        arth_agi_6.append(arthritis_perc)
```

##### **Creating the box plots, and saving them as .png files**

```
# Removing outliers to get a clearer boxplot for each median_agi_stub category.
from matplotlib import pyplot

# Create dataframe
arth_data = [arth_agi_1,arth_agi_2,arth_agi_3,arth_agi_4,arth_agi_5,arth_agi_6]

# Plot the dataframe
fontdict_title = {'family':'serif','variant':'small-caps','weight':'medium','size':'x-large'}
fontdict_label = {'family':'serif','variant':'small-caps','weight':'light','size':'small'}
fontdict_ticks = {'family':'serif','variant':'small-caps','weight':'light','size':'x-small'}
pyplot.boxplot(arth_data,showfliers=False)
pyplot.xticks([1,2,3,4,5,6],['\$1-\$24,999','\$25,000-\$49,999','\$50,000-\$74,999','\$75,000-\$99,999','\$100,000-\$199,999','\$200,000 or more'],rotation=60,fontdict=fontdict_ticks)
pyplot.title('Arthritis among adults aged >=18 years',fontdict=fontdict_title)
pyplot.xlabel('Median Adjusted Gross Income (AGI) Category',fontdict=fontdict_label)
pyplot.ylabel('Crude Prevalence (%)',fontdict=fontdict_label)

# Display the plot
pyplot.savefig("arthritis_boxplot.jpg",bbox_inches='tight')
pyplot.show()
```

### **Results**

##### **An example box plot for the "Arthritis among adults aged >=18 years" measure**

![](..\IA626_FinalProject_Davis\arthritis_boxplot.jpg)

As exhibited in the plots included here, 10 out of the 12 selected health outcome measures showed a distinct decrease in crude prevalence as adjusted gross income increased for 27,605 U.S. zip codes. This supports my initial hypothesis that poorer communities generally lack the resources to support healthy lifestyles that wealthier communities come by easily. The two outlying measures included:

* Cancer (excluding skin cancer) among adults aged >=18 years  
* High cholesterol among adults aged >=18 years who have been screened in the past 5 years  

##### Cancer

The "Cancer (excluding skin cancer) among adults aged >=18 years" measure showed an increase in crude prevalence as adjusted gross income increased. This is counter to my hypothesis, and may be an indicator that the hypothesis is not true for all health outcomes. It is also possible that cancer may be a special case. It is possible that cancer is not being detected in poorer communities as effictively as in welthier communities. It is also possible that wealthier communities may be exposed to more carcinogenic materials in some capacity. More study is needed to see if this result is an outlier or is a true health outcome trend.

##### High Cholesterol

The "High cholesterol among adults aged >=18 years who have been screened in the past 5 years" measure showed mixed reaults in crude prevalence for all adjusted gross income categories. More study is needed to determine why this may be the case, however this may have to do with the nutritional choices we make as Americans when compared with the diets of those in other countries. Our diet tends to be more unhealthy than in other places, and this may have resulted in higher cholestrol levels in all Americans regardless of economic status. This would be a good study to follow this project.

It is also worth noting that some of the adjusted median gross income categories did not include many U.S. counties, especially the "$200,000 or more" category. The total U.S. counties that had a median adjusted gross income in each respective category are listed below:

* $1 under $25,000 -> 487 counties
* $25,000 under $50,000 -> 17,408 counties
* $50,000 under $75,000 -> 6,945 counties 
* $75,000 under $100,000 -> 1,235 counties
* $100,000 under $200,000 -> 588 counties
* $200,000 or more -> 14 counties

The measures that were collected for the "$1 under $25,000", "$100,000 under $200,000", and "$200,000 or more" categories will likely have more individual circumstance bias as there are less communities to collect data from. This enables regional health issues and concerns to skew the results of the project. A potential solution to this problem would be to continue this study over multiple years, to boost the amount of data collected in these categories.

Ultimately, this study provides strong evidence that my initial hypothesis may be true. Poor communities may face higher crude prevalence of negative health outcomes for a variety of reasons discussed in the "Premise". More study and data collection must be done to determine if these results are conclusive.
