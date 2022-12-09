# Basic Data Cleaning
### Aims
1. Learn how to look at data to identify which changes and cleaning needs to be done
2. Learn the nessesary code to clean data
3. Establish a general workflow for data cleaning

### Introduction
Data cleaning is the first and one of the most important steps in any sort of data analysis. A lot of real-world datasets are very messy, they might have typos, missing data, incorrect data and might not even be prepared for analysis in R. This is especially true with datasets that have been copied/entered **by hand** or those whose responses are not **standardized**. Data cleaning is the process of 'preparing' this raw data for manipulation and analysis. It often involves **reformatting**, **correcting** and **removing** inaccurate or unwanted parts of this data. 

This tutorial is doable for begginers or people with an intermediate understanding who want to develop good general practice for data cleaning. It does require at least basic understanding of R, especially how objects work and how to call values and so forth.

![](https://github.com/EdDataScienceEES/tutorial-Mikael54/blob/master/images/clean-data.jpg)

Image from [here](http://www.polarisstrategic.com/spring-cleaning-your-data/)

While each dataset will need to be cleaned in its own way, there is a general workflow we can follow to clean data:

1. [Investigate and understand the data](#investigate-and-understand-the-data)
2. [Investigate and deal with structural issues](#investigate-and-deal-with-structural-issues)
    + [Column names](#column-names) 
    + [Data type](#data-type)
3. [Investigate and deal with data issues](#investigate-and-deal-with-data-issues)
    + [How to fix string inconsistencies](#how-to-fix-string-inconsistencies)
    + [How to check for duplicated data](#how-to-check-for-duplicated-data)
    + [How to deal with missing data](#how-to-deal-with-missing-data) 
    + [Verifying numerical data](#verifying-numerical-data) 

Note: the dataset for this tutorial can be found [here](https://github.com/EdDataScienceEES/tutorial-Mikael54/blob/master/Final%20Files/dragon_data.csv)

## Importing the data and loading libraries 

Start by setting your work directory and loading the `tidyverse` package we need for this tutorial, remember to follow good ethiquette and say what this code is at the start using `#` 
```
# Script title/purpose
# Date
# Your name
# Contact details (ie. email)

# set your working directory (the file path will depend on you computer/if it is linked to github):
setwd("C:/Users.../Data cleaning")

# Libraries, if you have not used these before you can install them by running the install.packages("package_name") command
library(tidyverse) # Tidyverse is a collection of many different packages and is one of the most commonly used packages in data science
```
Now, lets load the data, in this case we're gonna be looking at data collected by Hogwarths University, researchers are interested in the effect dragon species has on wingspan.
```
# Load data (again, the file path will depend on you computer/if it is linked to github):
dragon_data <- read.csv("...dragon_data.csv")
```

## Investigate and understand the data 
Ok, now we've got our data loaded, we can finally get to examining the structure and what we'll need to do to clean it, lets have a look at our data.

The `View()` function is part of base R and is pretty useful if you just want to explore a data set as it opens a whole new tab with the data:
``` 
# Using the View() command:
View(dragon_data)
``` 
However, for data cleaning purposes the `glimpse()` function is far more useful, `glimpse()` shows us the first few values of every column, but in addition to this, it also tells us about the data classes which we will get to [later](#data-type):
``` 
# Using the glimpse() command:
glimpse(dragon_data)
``` 
![glimpse](https://github.com/EdDataScienceEES/tutorial-Mikael54/blob/master/images/glimpse.PNG)

So based off this, we can see there are quiet a few problems with the data, lets start off by dealing with structural issues.
## Investigate and deal with structural issues
Structural issues generally deal with column names and data type, if these aren't correct and standardized, you may run into problems later on in your code.
### Column names
``` 
# We can call a list of the column names like this:
names(dragon_data)
``` 
![](https://github.com/EdDataScienceEES/tutorial-Mikael54/blob/master/images/names.PNG)

The first problem with this is that the names are not consistent in formatting. For example, it would be simpler for column names to be lowercase (but formatting can be a lot more than this, think about spaces being defined by `_` vs `.`. Consistent formating makes it easier to avoid making typos later, especially when you have a large number of variables. We can use `tolower` to remove capitals:

```
# Removing the capitals from columns
names(dragon_data) <- tolower(names(dragon_data))

# And now if we check the names we can see it's all lowercase
names(dragon_data)
```
![](https://github.com/EdDataScienceEES/tutorial-Mikael54/blob/master/images/names2.PNG)

### Changing column names
Some of the column names also seem quiet weird, specifically if we look at the first column which says `ï..id`, the `ï..` is actually created by an hidden character in the csv file, luckily it's pretty easy to rename columns in using the `rename` function from the `tidyverse`
```
# We can change the names of a colunm names like this:
dragon_data <- rename(dragon_data, "id" = "ï..id")
# And again we can check the names we can see that its changed
names(dragon_data)
```
![](https://github.com/EdDataScienceEES/tutorial-Mikael54/blob/master/images/names3.PNG)

### Data type
So now that we made sure all the columns are properly names, we need to make sure all our columns are the right type of data.

***
### What exactly is a data classes
Data classes are basically what R thinks your data is, R has five basic data classes:
- Character (chr) `"a"`, `"red"` : R basically views this as a string of words and not much more. This also means that R will not view this as a categorical or 'grouping' variable, so in this example, R doesn't understand that dragons with the same species name should be the considered the same.
- Numeric (num) `11.5`, `5`: R just views this as any number.
- Integer (int) `2L`, `2`: R views this a whole numbers, the L can be used to tell R to store this as an integer.
- Logical (log) `TRUE`, `FALSE`: R  views this as a True/False, this is also often how R checks conditions.
- Complex (com) `i`, `5+3i`: This is how R stores data involving complex numbers (numbers with the square root of negatives), in most data analysis you will not have to use this so don't worry much about it.

And finally, a bonus data type, factors:
- Technically factors aren't a data class, they're a data structure, which is how R organizes data. This structure is used to categorically group variables and if often used in data analesis. 

There's a lot about data classes structures that I didn't cover in this quick explination, if you want to know more, have a look at [this link](https://swcarpentry.github.io/r-novice-inflammation/13-supp-data-structures/).
***

So now that we know what data types are, lets check if they're correct in this dataset. We can run the `glimpse` function again to see the data types and think about if this data type is accurate:
```
glimpse(dragon_data) 
```
***
Results:

Rows: 17

Columns: 5

- id (integer): id should technically be a character, the id should be unique value given to each observation. It shouldn not tell us about where one observation stands in relationship to others. ID being an intiger means `1` is smaller than `2`, which is smaller than `3`, which is not the case for our data. It also means that dragons `2 + 3` would give us dragon `5`, which doesn't make any sense!

- sex (character): The sex is a grouping variable, which means it should be a factor, right now if we were to run any code with `sex`, R would not consider one female to be in the same group as another female, and the same goes for the males.  

- wingspan (character):  R views this as a character, however this should be a numeric variable. For example, right now R does not understand that 15 meters is 3 meters more than 12 meters. We'll get to it very soon, but have a look at the results of the `glimpse` and think about why R might have classified this as a character as opposed to numeric or integer...

- dragon_age_group (integer): So the researchers decided to put the age group as `1`, `2` and `3`, but this is the age group they belong to, not their actual age. Maybe `1` was something like 'child' or 'dragons between the ages of one and five years old', who knows how dragon researchers think! So, while age group two is higher than age group one, it is not nessesarily one year or one set 'value' older, and it moreso refers to a different lifestage rather than a strict numerical age difference. And so, this should be viewed as a catagorical variable, which means... yep, it's a factor! Decisions like this show why you should think carefully about your data and usually require a good understanding of the study methodology.

- dragon_species (character): Same as before, species is a catagorical variable, and when doing analesis with dragon species as a character, R will not view dragons from the same species as being in the same group. So once again, this needs to be a factor.   

***

So how do we actually go about changing how R views these columns, that's actually quiet easy using the `as.datatype`: 
```
# To change just a single column to a factor we could use:
dragon_data$sex <- as.factor(dragon_data$sex)

# But what if we had a dataset with a bunch of variables and didn't want to do this over and over again
# In that case we could do something like this:
factors <- c("sex", "dragon_age_group", "dragon_species")

# Then we can use the lapply function, the lapply function takes an input, applies a function and outputs it as a list.
# In this case it takes the dragon data whose columns corresponde with the previously defined 'factors'.
# Then we turn it into a factor using the 'factor' function
dragon_data[factors] <- lapply(dragon_data[factors], factor) 
```
Now all we need to do is change the `id` and `wingspan` columns and all our data will be in the right format:
```
# Changing the id column to character
dragon_data$id <- as.character(dragon_data$id)
``` 
However, changing the  wingspan to numeric is slightly more complicated, this is because all the data was entered with an `m` at the end (ie `15m`), so R cannot read that as a number. Luckily, we can use `parse_number` function from `tidyverse`, which selects **only** the numbers in our data.
```
# Changing the wingspan to numeric
dragon_data$wingspan <- as.numeric(parse_number(dragon_data$wingspan))
```
Now we can check that everything worked as expected:
```
# Now we can check that it worked using:
glimpse(dragon_data)
```
![](https://github.com/EdDataScienceEES/tutorial-Mikael54/blob/master/images/glimpse2.PNG)

Everything looks good, note that dbl (or double) means that R views this as a number with decimal points. In practice these two are basically the same.

## Investigate and deal with data issues

Great, now that we've gotten all the column names figured out we can move on to cleaning the actual data, from our initial `glimpse` function we can already see quiet a lot wrong with our data (however, the `summary()` and `View()` functions are usually better when dealing with data from one column). Lets start by dealing with **string inconsistencies**.

### How to fix string inconsistencies
String inconsistencies are issues with the consistent formatting of our data. This could be anything from typos to not having a set standardization when inputing data. For this example, lets have a look at the `dragon_species` column:
```
# when it's a factor, we can use summary() to get a quick look at what the different levels are and how many there are
summary(dragon_data$dragon_species)
```
![](https://github.com/EdDataScienceEES/tutorial-Mikael54/blob/master/images/summary_dragon.PNG)

Ok so we can see that there are many dragon species, but they're no all **formated the same**, some use a `_` and some simply use a space. Both these issues can be solved using `gsub()`, this will take **one character** and **replace it**, or 'sub' it out for **another**:
```
# Using gsub() to replace all spaces with underscores
# The first value in the parenthesis is what you want to replace
# The second it what you want to make it into
# The third is which 'string' (in this case, column) you want to use 
# Unfortunately, using gsub() turns the output into characters, so we have to use the as.factor function again
dragon_data$dragon_species <- as.factor(gsub(" " , "_", dragon_data$dragon_species))

# Now using summary() shows us all spaces have been replaced
summary(dragon_data$dragon_species)
```
![](https://github.com/EdDataScienceEES/tutorial-Mikael54/blob/master/images/summary%20dragon%202.PNG)

### How to check for duplicated data
Ok, now we've cleared up all the string inconsistencies in your data, let's have a look at how we can **identify** and **remove** any duplicates that may have slipped in there. We can identify them using `duplicate()` and remove them using the `distinct` function from `tidyverse`:
```
# First lets check if there are any duplicated datasets, this code will return the id of any rows that are duplicated
dragon_data$id[duplicated(dragon_data)]
# We can see that both id number 4 and 10 have been duplicated.
# We could double check if this was just a mistake (if we still have the original recording sheet for example)
```
![](https://github.com/EdDataScienceEES/tutorial-Mikael54/blob/master/images/duplicate.PNG)
```
# If the data is in fact duplicated we can remove it with the distinct() function
# This code looks at the entire row in dragon_data, 
# In this code, the '.keep_all = TRUE' tells it to keep all values that fit the criteria (ie. they are distict):
dragon_data_no_dup <-  distinct(dragon_data, .keep_all = TRUE)

# If we want it to check just a specific column, we can just add a comma and put something like id
dragon_data_no_dup <-  distinct(dragon_data, id, .keep_all = TRUE)
```
### How to deal with missing data
#### Creating NA values
The next issue you would commonly face is **missing values** or values that are **not available (NA)**. This can be dealt with in multiple ways. Lets have a look at the `sex` column to see how to deal with this:
```
# We can, once again use the summary function to examine the column 
summary(dragon_data_no_dup$sex)
```
![](https://github.com/EdDataScienceEES/tutorial-Mikael54/blob/master/images/summary_sex.PNG)

Based off the results it seems like we have **one blank** and **one NA**, to avoid problems with any future data manipulations, it would be best to change the **blank to NA**. This can be done using the `na_if` function from `tidyverse`:
```
# Convert all blank values to NA
# The first thing in the parenthesis is where you want them to look to apply the function
# The second is under what conditions you want it to become NA
# We have nothing between the brackets, so it will convert blanks to NA
dragon_data_no_dup$sex <- na_if(dragon_data_no_dup$sex,"")

# We can still use the 'normal' way of replacing variables as well
# You might get a warning with this code since you've already removed blank spaces in the previous line
dragon_data_no_dup[dragon_data_no_dup$sex == "",]$sex <- NA

# Then we can check with another summary function
summary(dragon_data_no_dup$sex)
```
Hold on a second! The results of the summary seem a bit weird:

![](https://github.com/EdDataScienceEES/tutorial-Mikael54/blob/master/images/summary_sex2.PNG)

It looks like, because of an extra space before the NA, R didn't proccess the original NA in the dataframe as 'Not Available' but just another factor. This is something you should watch out for when doing data analesis, real NA's will show up as `NA's`. Luckily this is pretty easily fixed:
```
# Making sure all NA values are recognized by R
dragon_data_no_dup$sex <- na_if(dragon_data_no_dup$sex," NA")
```
#### Dealing with NA values
Some of the future data manipulation we could do might not work if we have NA's, but there's a pretty easy was to ignore them using the `is.na()` function, here we create a completely new dataset without the NA's, but this can also used with specific functions:
```
# Creating a new object with the sex values without any NA's
# The is.na test to see if it is NA and the ! makes sure to pick those that are not (it means 'not equal to')
no_na_dragons <- dragon_data_no_dup[!is.na(dragon_data_no_dup$sex),]

# We can also use it in functions like this
summary(dragon_data_no_dup[!is.na(dragon_data_no_dup$sex),]$sex)

# You'll notice that the previous output does not include NAs, unlike this one
summary(dragon_data_no_dup$sex)
```
![](https://github.com/EdDataScienceEES/tutorial-Mikael54/blob/master/images/summary_na.PNG)
### Verifying numerical data
#### Check for incorrect data
It's always good practice to check the data to make sure everything actually makes sense and is accurate. For this example we can look at dragon wingspan. There two main ways to check numerical data like this, histograms and the `summary` function. Using `summary` will give you quartiles, minimum, maximum and mean. While `hist` will automatically generate a histogram for you. Note that there are much more beatiful histograms you can create with R, this histogram is not really meant to be shown, but rather to identify any potentially incorrect data for data cleaning purposes. If you are interested in making beatifull histograms or graphs in general, check [this tutorial](https://ourcodingclub.github.io/tutorials/datavis/) on  how to use ggplot.
```
# Generate a histogram to identify any incorrect data
# The first part of the parenthesis refers to what data you want to plot
# The second 'breaks' refers to how many 'pieces' or 'bars' you want to break your histogram into
hist(dragon_data_no_dup$wingspan, breaks=20)

# Create a summary of the data
summary(dragon_data_no_dup$wingspan)
```
![image](https://github.com/EdDataScienceEES/tutorial-Mikael54/blob/master/images/histogram1.png)
![](https://github.com/EdDataScienceEES/tutorial-Mikael54/blob/master/images/summary_wings.PNG)

Ok so based off this summary we can see that some values might not make sense. Most values seem within reason, however, the minimum value is 0 and the maximum is 350, based off our understanding of dragon biology, this is impossible and was most likely an error when the data was entered. So, assuming we can't track down the original values, we'll have to change these to NA.
```
# Changing the impossible values to NA
# This places NA on the values whose wingspan in 0 and 350
# If you aren't aware, in %in% means to belong to one of the following
dragon_data_no_dup[dragon_data_no_dup$wingspan %in% c(0, 350),]$wingspan <- NA
```
So now if we run the code from before it should make a much better hsitogram and summary:
```
# Generate a histogram to identify any incorrect data
hist(dragon_data_no_dup$wingspan, breaks=20)
# Create a summary of the data
summary(dragon_data_no_dup$wingspan)
```
![](https://github.com/EdDataScienceEES/tutorial-Mikael54/blob/master/images/wings2.PNG)
![hist2](https://github.com/EdDataScienceEES/tutorial-Mikael54/blob/master/images/histogran2.png)

#### A word on outliers

It seems we do still have one somewhat extreme value in our data, a dragon with a wingspan of 52 meters seems to be heavily skewing our data. However, the scientist at hogwarts university know that this is biologically possible, but is just a very extreme case. Sometimes outliers can severely effect and skew data, but they often can also represent natural variation in data. This is a controversial topic and before you do delete or alter your data to minimize the effect of outliers, think carefully about if you should. [This article](http://www.asasrms.org/Proceedings/y2012/Files/304068_72402.pdf) goes a bit more in depth about the effects of outliers and how to identify them, [this article](https://www.theanalysisfactor.com/outliers-to-drop-or-not-to-drop/) explains in somewhat simple terms when you can change the outliers.

Given the fact that this is such a small dataset, I would advise against removing or altering this outlier. However, if we did want to do so, we could use the same code as above to make the values NA.

### Conclusion
Great job! Now you're ready to move on to analysing your data. Rememeber, each dataset will need to be cleaned in its own way and you need to have some knowledge about the data itself to decide wether or not it makes sense. Since we've gone through the trouble of cleaning it, lets have a look at if dragon species does effect wingspan, again, this is a basic graph so check out [this tutorial](https://ourcodingclub.github.io/tutorials/datavis/index.html) if you want to know more.
```
boxplot(wingspan ~ dragon_species, dragon_data_no_dup)
```
![boxplot](https://github.com/EdDataScienceEES/tutorial-Mikael54/blob/master/images/boxplot.png)
***
## Challenge yourself!
Now that you know how to clean data, why don't you practice doing it with [this imaginary dataset](https://github.com/EdDataScienceEES/tutorial-Mikael54/blob/master/Final%20Files/Challenge.csv) about people's ages and reading habits. While you have all the code you need, have a think about which ways you could reformat the data, and come back here for a hint if you're stuck (particularly in the fourth column). 

<details><summary>Hint</summary>  In the fourth column some people put a single number of hours read, while other put a range. There's two ways you can go about this, you can either make all the values a range and have it a a factor, instead of a number, or you can just put NA on the people that gave a range and keep it as a number. The choice you go with will ultimately depend on what kind of analesis you want to do (which is admitedly a bit unfair to you, as we aren't actually doing any analesis). </details>

And here is a list of issues and how to fix it if you want to check you've done it all:

<details><summary>List</summary>
    
Data Structure:
    
- The title's (particularly column 3 and 4) arent great, use `to.lower` and `rename` to change them.

- Data type: id should be a `character`, sex a `factor`, age should be `numeric`, hours read should be a `factor` or `numeric` (see the previous hint). Use `as.datatype` to fix this.
    
- Use `parse_number` to remove text characters from the age column.
    
Data issues:
    
- Many string inconsistencies in the `sex` column, using `g_sub` can fix this, at the end you want two factors (`male` and `female` for example).
    
- id number 2 is dupliated, use `distict` to remove it.
    
- The sex column has a lot of missing data, use `na_if` or `<- NA`.
                                                                   
- The age column has NAs but R might not have recognized them as such, use `na_if` or `<- NA`
    
- One age value is or `9999`, that seems to have been a mistake, use `na_if` or `<- NA`
                                                                                        
</details>
    
