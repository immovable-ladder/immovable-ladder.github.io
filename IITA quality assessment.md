#IITA Impact Evaluation Quality Assessment

- dataset name: Impact evaluation questionnaire of IITA technologies in the Great Lakes Region
- source: International Institute of Tropical Agriculture
- date of survey: 2014
- countries: Burundi, DRC (South Kivu), Rwanda
- Number of households in dataset: 504
- Number of variables, all datasets (minus repeated): 537 

##Data Overview

###Questionnaire Modules

1. Part A: Respondent's Identification/Background
2. Part B: Household Composition and Characteristics
3. Part C1: Household Assets
4. Part C2: Other household asset
5. Part D: Household Expenditure
	1. Food consumption
	2. Non-food items
6. Part E: Food Security
7. Part F: Improved crop varieties/Technology Adoption
8. Part G: Land use and management (*by Season A and Season B)
	1. G5a/G6a: Land use and conservation practices
	2. G5b/G6b: Input Use
	3. G5c/G6c: Crop Harvested and Utilization
	4. G7: Use of warrantage
	5. G8: Market access
11. Part H: Livestock ownership
12. Part J: Livestock products
13. Part K: Other Sources of income in the last 12 months
14. Part L: Access to extension/training and credit services
15. Part M: Communal Property Resources
16. Part N: Multi-stakeholder platforms and intervention landscape 

###Data file names

1. Agricultural_activities
1. Asset_roaster
1. Asset_roaster2
1. Communal _property _resources
1. Credit_source
1. Crop _harvestd _utilizatn
1. Fruit_beverages
1. HH _compostion _characteristic
1. HH _compostion _characteristic _b
1. Household_diseases
1. Improved__crop _management
1. Improved__crop _varieties
1. Input__short _rainy _season
1. Input_use
1. IPM
1. Land _use _management
1. Land _use _mgt _short _rainy
1. Livestock _ownership
1. Livestock _products
1. Market _access
1. Meat _animalproducts
1. Month _food _scarcity
1. Nonfood _items1
1. Nonfood _items2
1. Other _Income
1. Other _livestock
1. Others
1. Platform _type
1. Staple _food
1. UtilzationshortRainySeason
1. Vegetable _fat _others
1. Warrantage _use


###Comments

*The names of modules and the names of files are not well-matched. This makes it very difficult for outside users to locate variables or parts of the questionnaire they may be looking for.

*Many data files contain incomplete modules, or parts of multiple modules, or are replications of modules but for a different season, etc. These files could be reorganized and renamed in ways that make them easier to understand.

*The organization of questionnaire modules is not always intuitive. Some modules contain a very limited set of questions on a narrow scope, others contain a wide variety of questions with a broad scope (esp. Part G).

##aWhere Data cleaning process

###Metadata construction
- R code was used to extract the names of all variables in the datasets and write them to a codebook along with the file they are drawn from.
- aWhere data analysts filled the codebook by matching each variable in the dataset to the exact question from the questionnaire.
- Each variable was also matched with a unit and an original questionnaire module. 

###Data vetting
- Examine datafiles and note any:
	- missing or inconsistent location or time values
	- unconverted enumerated values
	- datafile variables not reflected in questionnaire
	- questions in questionnaire not included in datafiles
	- missing units in questionnaire
	- inconsistencies between question asked and answers given in files

###Data Action 1: Location cleaning
Inconsistency in reporting of the latitude and longitude variables is demonstrated with this sample:

		HHID		Latitude		longtitude
		2301		2168076			28530328
		2327		2173481			28527708
		2573		217439			2174394
		11			02 14 7761		028 48 6173
		12			02 14 7512		028 48 6243
		2577		215.5989075		2853.238525
		2575		216.2319946		2853.474854
		2261		215.7144928		2853.109375
		2645		02 16 .2726		028 53 1054
		35			02 15.5898		028 53 . 4474
		2526		02  16  6328	028 53. 550
		2613		02 17.0651		028 52. 7919
		2328		211.3433		028.50.5221
		2284		2116.28			28531554
		2314		2,175,806		295,412
		2463		2,379,487		28,451,993
		2226		24.5312			028'38.9395
		2655		21.47888		17'12588 767

These inconsistencies were cleaned with R code that assumed all coordinates were originally numerical degrees-minutes-seconds coordinates corresponding to the South Kivu area (roughly at 2 degrees latitude South, and 28 degrees longitude East). The code performed the following actions:

1.  Removed all spaces and any initial "0" integers
2.  Separated into columns the degrees, minutes and seconds.
	- For latitude, the first integer (e.g. "2") represented the degrees. For longitude, the first two integers (e.g. "28") represented degrees.
	- The next two integers after the degrees were placed into the minutes column.
	- All remaining integers were placed in the seconds column.  
3.  Calculated decimal degree coordinates using only the degrees and minutes extracted for each location

Example of cleaning process:

		HHID	Latitude.original	longtitude.original	Latitude.awhere	longtitude.awhere	lat_d	lat_m	lat_o		lat_decimal		lon_d	lon_m	lon_o		lon_decimal
		1		02 50 7519			028 47 4250			2507519000		2847425000			2		50		7519000		-2.833333333	28		47		7425000		28.78333333
		2		2157599				28532968			2157599000		2853296800			2		15		7599000		-2.25			28		53		3296800		28.88333333

aWhere decided to use only degrees and minutes to calculate approximate decimal degree coordinates because of uncertainty in how to interpret the seconds column. Because decimals were inconsistently reported, it was impossible to know whether any group of integers beginning with the numbers 0-5 was meant to be in the form X.XXX or XX.XX. To avoid false specificity, the locations therefore exclude the seconds.


###Data Action 2: Streamline columns and records

- Match cleaned locations to original files by Household IDs
- Remove records with no locations
- Remove duplicated unique IDs. There were 3 duplicated household IDs, which generally appeared similarly to the following:

		2531|Nsimire M'Mushagalusa|Danane Birego|Spouse|0|-2.6|28.75|132201|28/10/2014|15:31:12|Arsene
		2531|bahizire muhendwa|bahizire muhendwa|Head|2.43993E+11|-2.6|28.75|103145|28/10/2014|09:39:45|Thithy

	
	- These appear to be different households accidentally assigned the same ID
	- Strategy: Create new ID for one - Nsimire M'Mushagalusa becomes 2530 (unused ID)
	- Other duplicates: ID 2578 (one reassigned to 2580); ID 2627 (one reassigned to 2625)


- Remove bad characters and numbers from names (ex. FRANÃ‡OISE M' SINDANI)
- Simplify Respondent and Household head columns (remove "Is respondent the household head" variable, fill in Relationship to household head column with "Head" value for all empty cells)
- Relation to HH Head variable: Replace all empty cells with "Head", allows users to quickly visualize how many respondents were the HH Head and what was the distribution of non-head respondents

###Remaining Issues

The units many variables are not specified. Ex: altitude, though the values in the dataset indicate it is meters.

There is sometimes ambiguity in question wording that causes potential misinterpretation:


- Example: Part B, Question 9: "Own farm labour contribution (in terms of % e.g. 10%, 25%, 40%)"
- This could mean either "percent of your labour that is spent on your own farm" or "percent of all labour on your farm performed by you"
- Other examples of ambiguity: Part F, Question 4: "Ever used?" in reference to a particular technology. While this seems to be asking whether the person has ever used the technology, it could be interpreted as asking whether the person has ever seen the technology used in his/her community or on his/her farm.  

The questionnaire sometimes appears to mistakenly cite questions out of order. Ex. Part F, Questions 10 and 11.

The PDF of the questionnaire cuts off the right side of a handful of tables.

It is clear in the dataset that there is a "short rainy season" which is enumerated separately from the main season, but the questionnaire refers exclusively to Season A, Season B, and the dry season, and does not specify which is the short rainy season.

The datasets include records for numerous crops, assets, and other attributes which are inapplicable to many of the households. As a result there are many unnecessary rows of NAs. 

##Recommendations


