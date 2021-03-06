---
layout: page
title: Headcount
sidebar: true
---

Federal and state governments publish a huge amount of data. You can find
a large collection of it on [Data.gov](http://data.gov) -- everything from land surveys
to pollution to census data.

As programmers, we can use those data sets to ask and answer questions. Starting
with CSV data we will:

* build a "Data Access Layer" which allows us to query/search the underlying data
* build a "Relationships Layer" which creates connections between related data
* build an "Analysis Layer" which uses the data and relationships to draw conclusions

We'll build upon a dataset centered around schools in Colorado provided by the Annie E. Casey foundation.

## Project Overview

### Goals

* Use tests to drive both the design and implementation of code
* Decompose a large application into components such as parsers, repositories, and analysis tools
* Use test fixtures instead of actual data when testing
* Connect related objects together through references

### Getting Started

1. One team member clones the repository at https://github.com/turingschool-examples/headcount.
2. `cd headcount`
3. `git remote rm origin`
4. Create your own new `headcount` repository on GitHub, then add that remote to your `headcount` from the command line.
5. Push your repository to the new remote origin.
6. Add your teammate(s) as collaborators.
7. Create a [Waffle.io](http://waffle.io) account for project management.

## Base Expectations

### Data Access Layer

#### `DistrictRepository`

The `DistrictRepository` is responsible for holding and searching our `District`
instances. It offers the following methods:

* `find_by_name` - returns either `nil` or an instance of `District` having done a *case insensitive* search
* `find_all_matching` - returns either `[]` or one or more matches which contain the supplied name fragment, *case insensitive*

There is no one data file that contains just the districts. The data must be extracted from one of the other files.

#### `District`

The `District` is the top of our data hierarchy. It has the following methods:

* `name` - returns the upcased string name of the district
* `statewide_testing` - returns an instance of `StatewideTesting`
* `enrollment` - returns an instance of `Enrollment`

#### `StatewideTesting`

The instance of this object represents data from the following files:

* `3rd grade students scoring proficient or above on the CSAP_TCAP.csv`
* `4th grade students scoring proficient or above on the CSAP_TCAP.csv`
* `8th grade students scoring proficient or above on the CSAP_TCAP.csv`
* `Average proficiency on the CSAP_TCAP by race_ethnicity_Math.csv`
* `Average proficiency on the CSAP_TCAP by race_ethnicity_Reading.csv`
* `Average proficiency on the CSAP_TCAP by race_ethnicity_Writing.csv`

##### `.initialize(name)`

An instance this class can be initialized by supplying the name of the district
which is then used to find the matching data in the data files.

That instance then offers the following methods:

##### `.proficient_by_grade(grade)`

This method takes one parameter:

* `grade` as an integer from the following set: `[3, 4, 8]`

A call to this method with an unknown grade should raise a `UnknownDataError`.

The method returns a hash grouped by year referencing percentages by subject all
as three digit floats.

*Example*:

```
statewide_testing.proficient_by_grade(3)
# => {2008 => {:math => 0.857, :reading => 0.866, :writing => 0.671},
      2009 => {:math => 0.824, :reading => 0.862, :writing => 0.706},
      2010 => {:math => 0.849, :reading => 0.864, :writing => 0.662},
      2011 => {:math => 0.819, :reading => 0.867, :writing => 0.678},
      2012 => {:math => 0.870, :reading => 0.830, :writing => 0.655},
      2013 => {:math => 0.855, :reading => 0.859, :writing => 0.639},
      2014 => {:math => 0.834, :reading => 0.831, :writing => 0.668},
     }
```

##### `.proficient_by_race_or_ethnicity(race)`

This method takes one parameter:

* `race` as a symbol from the following set: `[:asian, :black, :pacific_islander, :hispanic, :native_american, :two_or_more, :white]`

A call to this method with an unknown race should raise a `UnknownDataError`.

The method returns a hash grouped by race referencing percentages by subject all
as truncated three digit floats.

*Example*:

```
statewide_testing.proficient_by_race_or_ethnicity(:asian)
# => {2011 => {:math => 0.816, :reading => 0.897, :writing => 0.826},
      2012 => {:math => 0.818, :reading => 0.893, :writing => 0.808},
      2013 => {:math => 0.805, :reading => 0.901, :writing => 0.810},
      2014 => {:math => 0.800, :reading => 0.855, :writing => 0.789},
     }
```

##### `.proficient_for_subject_by_grade_in_year(subject, grade, year)`

This method takes three parameters:

* `subject` as a symbol from the following set: `[:math, :reading, :writing]`
* `grade` as an integer from the following set: `[3, 4, 8]`
* `year` as an integer for any year reported in the data

A call to this method with any invalid parameter (like subject of `:science`) should raise a `UnknownDataError`.

The method returns a truncated three-digit floating point number representing a percentage.

*Example*:

```
statewide_testing.proficient_for_subject_by_grade_in_year(:math, 3, 2008) # => 0.857
```

##### `.proficient_for_subject_by_race_in_year(subject, race, year)`

This method take three parameters:

* `subject` as a symbol from the following set: `[:math, :reading, :writing]`
* `race` as a symbol from the following set: `[:asian, :black, :pacific_islander, :hispanic, :native_american, :two_or_more, :white]`
* `year` as an integer for any year reported in the data

A call to this method with any invalid parameter (like subject of `:history`) should raise a `UnknownDataError`.

The method returns a truncated three-digit floating point number representing a percentage.

*Example*:

```
statewide_testing.proficient_for_subject_by_race_in_year(:math, :asian, 2012) # => 0.818
```

##### `.proficient_for_subject_in_year(subject, year)`

This method take two parameters:

* `subject` as a symbol from the following set: `[:math, :reading, :writing]`
* `year` as an integer for any year reported in the data

A call to this method with any invalid parameter (like subject of `:history`) should raise a `UnknownDataError`.

The method returns a truncated three-digit floating point number representing a percentage.

*Example*:

```
statewide_testing.proficient_for_subject_in_year(:math, 2012) # => 0.680
```

#### `Enrollment`

The instance of this object represents data from the following files:

* `Dropout rates by race and ethnicity.csv`
* `High school graduation rates.csv`
* `Kindergartners in full-day program.csv`
* `Online pupil enrollment.csv`
* `Pupil enrollment by race_ethnicity.csv`
* `Pupil enrollment.csv`
* `Special education.csv`
* `Remediation in higher education.csv`

##### `.initialize(name)`

An instance this class can be initialized by supplying the name of the district
which is then used to find the matching data in the data files.

That instance then offers the following methods:

##### `.dropout_rate_in_year(year)`

This method takes one parameter:

* `year` as an integer for any year reported in the data

A call to this method with any unknown `year` should return `nil`.

The method returns a truncated three-digit floating point number representing a percentage.

*Example*:

```
enrollment.dropout_rate_in_year(2012) # => 0.680
```

##### `.dropout_rate_by_gender_in_year(year)`

This method takes one parameter:

* `year` as an integer for any year reported in the data

A call to this method with any unknown `year` should return `nil`.

The method returns a hash with genders as keys and three-digit floating point number representing a percentage.

*Example*:

```
enrollment.dropout_rate_by_gender_in_year(2012)
# => {:female => 0.002, :male => 0.002}
```

##### `.dropout_rate_by_race_in_year(year)`

This method takes one parameter:

* `year` as an integer for any year reported in the data

A call to this method with any unknown `year` should return `nil`.

The method returns a hash with race markers as keys and a three-digit floating point number representing a percentage.

*Example*:

```
enrollment.dropout_rate_by_race_in_year(2012)
# => {
  :asian => 0.001,
  :black => 0.001,
  :pacific_islander => 0.001,
  :hispanic => 0.001,
  :native_american => 0.001,
  :two_or_more => 0.001,
  :white => 0.001
}
```

##### `.dropout_rate_for_race_or_ethnicity(race)`

This method takes one parameter:

* `race` as a symbol from the following set: `[:asian, :black, :pacific_islander, :hispanic, :native_american, :two_or_more, :white]`

A call to this method with any unknown `race` should raise an `UnknownRaceError`.

The method returns a hash with years as keys and a three-digit floating point number representing a percentage.

*Example*:

```
enrollment.dropout_rate_for_race_or_ethnicity(:asian)
# => {
  2011 => 0.047,
  2012 => 0.041
}
```

##### `.dropout_rate_for_race_or_ethnicity_in_year(race, year)`

This method takes two parameters:

* `race` as a symbol from the following set: `[:asian, :black, :pacific_islander, :hispanic, :native_american, :two_or_more, :white]`
* `year` as an integer for any year reported in the data

A call to this method with any unknown `year` should return `nil`.

The method returns a truncated three-digit floating point number representing a percentage.

*Example*:

```
enrollment.dropout_rate_for_race_or_ethnicity_in_year(:asian, 2012) # => 0.001
```

##### `.graduation_rate_by_year`

This method returns a hash with years as keys and a truncated three-digit floating point number representing a percentage.

*Example*:

```
enrollment.graduation_rate_by_year
# => {2010 => 0.895,
      2011 => 0.895,
      2012 => 0.889,
      2013 => 0.913,
      2014 => 0.898,
     }
```

##### `.graduation_rate_in_year(year)`

This method takes one parameter:

* `year` as an integer for any year reported in the data

A call to this method with any unknown `year` should return `nil`.

The method returns a truncated three-digit floating point number representing a percentage.

*Example*:

```
enrollment.graduation_rate_in_year(2010) # => 0.895
```

##### `.kindergarten_participation_by_year`

This method returns a hash with years as keys and a truncated three-digit floating point number representing a percentage.

*Example*:

```
enrollment.kindergarten_participation_by_year
# => {2010 => 0.391,
      2011 => 0.353,
      2012 => 0.267,
      2013 => 0.487,
      2014 => 0.490,
     }
```

##### `.kindergarten_participation_in_year(year)`

This method takes one parameter:

* `year` as an integer for any year reported in the data

A call to this method with any unknown `year` should return `nil`.

The method returns a truncated three-digit floating point number representing a percentage.

*Example*:

```
enrollment.kindergarten_participation_in_year(2010) # => 0.391
```

##### `.online_participation_by_year`

This method returns a hash with years as keys and an integer as the value.

*Example*:

```
enrollment.online_participation_by_year
# => {2010 => 16,
      2011 => 18,
      2012 => 24,
      2013 => 32,
      2014 => 40,
     }
```

##### `.online_participation_in_year(year)`

This method takes one parameter:

* `year` as an integer for any year reported in the data

A call to this method with any unknown `year` should return `nil`.

The method returns a single integer.

*Example*:

```
enrollment.online_participation_in_year(2013) # => 33
```

##### `.participation_by_year`

This method returns a hash with years as keys and an integer as the value.

*Example*:

```
enrollment.participation_by_year
# => {2009 => 22620,
      2010 => 22620,
      2011 => 23119,
      2012 => 23657,
      2013 => 23973,
      2014 => 24578,
     }
```

##### `.participation_in_year(year)`

This method takes one parameter:

* `year` as an integer for any year reported in the data

A call to this method with any unknown `year` should return `nil`.

The method returns a single integer.

*Example*:

```
enrollment.participation_in_year(2013) # => 23973
```

##### `.participation_by_race_or_ethnicity(race)`

This method takes one parameter:

* `race` as a symbol from the following set: `[:asian, :black, :pacific_islander, :hispanic, :native_american, :two_or_more, :white]`

A call to this method with any unknown `race` should raise an `UnknownRaceError`.

The method returns a hash with years as keys and a three-digit floating point number representing a percentage.

*Example*:

```
enrollment.participation_by_race_or_ethnicity(:asian)
# => {
  2011 => 0.047,
  2012 => 0.041,
  2013 => 0.052,
  2014 => 0.056
}
```

##### `.participation_by_race_or_ethnicity_in_year(year)`

This method takes one parameter:

* `year` as an integer for any year reported in the data

A call to this method with any unknown `year` should return `nil`.

The method returns a hash with race markers as keys and a three-digit floating point number representing a percentage.

*Example*:

```
enrollment.participation_by_race_or_ethnicity_in_year(2012)
# => {
  :asian => 0.036,
  :black => 0.029,
  :pacific_islander => 0.118,
  :hispanic => 0.003,
  :native_american => 0.004,
  :two_or_more => 0.050,
  :white => 0.756
}
```

##### `.special_education_by_year`

This method returns a hash with years as keys and an floating point three-significant digits representing a percentage.

*Example*:

```
enrollment.participation_by_year
# => {2009 => 0.075,
      2010 => 0.078,
      2011 => 0.072,
      2012 => 0.071,
      2013 => 0.070,
      2014 => 0.068,
     }
```

##### `.special_education_in_year(year)`

This method takes one parameter:

* `year` as an integer for any year reported in the data

A call to this method with any unknown `year` should return `nil`.

The method returns a single three-digit floating point percentage.

*Example*:

```
enrollment.special_education_in_year(2013) # => 0.105
```

##### `.remediation_by_year`

This method returns a hash with years as keys and an floating point three-significant digits representing a percentage.

*Example*:

```
enrollment.participation_by_year
# => {2009 => 0.232,
      2010 => 0.251,
      2011 => 0.278
     }
```

##### `.remediation_in_year(year)`

This method takes one parameter:

* `year` as an integer for any year reported in the data

A call to this method with any unknown `year` should return `nil`.

The method returns a single three-digit floating point percentage.

*Example*:

```
enrollment.remediation_in_year(2010) # => 0.250
```

#### `EconomicProfile`

The instance of this object represents data from the following files:

* `Median household income.csv`
* `School-aged children in poverty.csv`
* `Students qualifying for free or reduced price lunch.csv`
* `Title I students.csv`

An instance of this class represents the data for a single district and offers the following methods:

##### `.free_or_reduced_lunch`

##### `.free_or_reduced_lunch_in_year(year)`

##### [TODO: more methods for this category]

### Relationships Layer

*TODO: These are just a sketch of where we're going.*

Assume we start with loading our data and finding a school district like this:

```
dr = DistrictRepository.new
dr.load('./data')
district = dr.find_by_name("ACADEMY 20")
```

Then each `district` has several child objects loaded with data allowing us to ask questions like this:

```
district.enrollment.in_year(2009) # => 22620
district.enrollment.graduation_rate.for_high_school_in_year(2010) # => 0.895
district.statewide_testing.proficient_for_subject_by_grade_in_year(:math, 3, 2008) # => 0.857
```

### Analysis Layer

*TODO: Coming soon!*

## References

### Data Sources

* [Search Index](http://datacenter.kidscount.org/data#CO/10/0)
* [Median Household Income](http://datacenter.kidscount.org/data/tables/6228-median-household-income?loc=7&loct=10#detailed/10/1278-1457/false/1376,1201,1074,880,815/any/12960)
* [School Aged Children in Poverty](http://datacenter.kidscount.org/data/tables/435-school-aged-children-in-poverty?loc=7&loct=10#detailed/10/1278-1457/false/36,868,867,133,38/any/11775,1084)
* [Pupil Enrollment](http://datacenter.kidscount.org/data/tables/5678-pupil-enrollment?loc=7&loct=10#detailed/10/1278-1457/false/869,36,868,867,133/any/12280)
* [Special Education](http://datacenter.kidscount.org/data/tables/5314-special-education?loc=7&loct=10#detailed/10/1278-1457/false/869,36,868,867,133/any/14675,11828)
* [Title 1 Students](http://datacenter.kidscount.org/data/tables/5325-title-i-students?loc=7&loct=10#detailed/10/1278-1457/false/869,36,868,867,38/any/11841)
* [Students Qualifying for Free and Reduced Price Lunch](http://datacenter.kidscount.org/data/tables/469-students-qualifying-for-free-or-reduced-price-lunch?loc=7&loct=10#detailed/10/1278-1457/false/869,36,868,867,133/109,110,111/11515,7665)
* [Kindergarteners in Full-Day Program](http://datacenter.kidscount.org/data/tables/449-kindergartners-in-full-day-program?loc=7&loct=10#detailed/10/1278-1457/false/869,36,868,867,133/any/11012)
* [High School Graduation Rates](http://datacenter.kidscount.org/data/tables/6134-high-school-graduation-rates?loc=7&loct=10#detailed/10/1278-1457/false/869,36,868,867,133/any/12806)
* [Dropout Rates by Race and Ethnicity](http://datacenter.kidscount.org/data/tables/7296-dropout-rates-by-race-and-ethnicity?loc=7&loct=10#detailed/10/1278-1457/false/868,867/785,786,787,788,789,790,791,792,3494,2302/14353)
* [Online Pupil Enrollment](http://datacenter.kidscount.org/data/tables/7141-online-pupil-enrollment?loc=7&loct=10#detailed/10/1278-1457/false/36,868,867/any/14171)
* [Remediation in Higher Education](http://datacenter.kidscount.org/data/tables/7663-remediation-in-higher-education?loc=7&loct=10#detailed/10/1278-1457/false/867,133,38/any/14818)
* [3rd Grade Students Scoring Proficient or Above on the CSAP/TCAP](http://datacenter.kidscount.org/data/tables/5651-3rd-grade-students-scoring-proficient-or-above-on-the-csap-tcap?loc=7&loct=10#detailed/10/1278-1457/false/869,36,868,867,133/129,130,145/12252)
* [4rd Grade Students Scoring Proficient or Above on the CSAP/TCAP](http://datacenter.kidscount.org/data/tables/7081-4th-grade-students-scoring-proficient-or-advanced-on-csap-tcap?loc=7&loct=10#detailed/10/1278-1457/false/869,36,868,867,133/any/14099)
* [8th Grade Students Scoring Proficient or Above on the CSAP/TCAP](http://datacenter.kidscount.org/data/tables/5657-8th-grade-students-scoring-proficient-or-above-on-the-csap-tcap?loc=7&loct=10#detailed/10/1278-1457/false/869,36,868,867,133/129,130,145/12262)
* [Pupil Enrollment by Race & Ethnicity](http://datacenter.kidscount.org/data/tables/3736-pupil-enrollment-by-race-ethnicity?loc=7&loct=10#detailed/10/1278-1457/false/869,36,868,867,133/84,87,86,85,88,1849,185,13/11661,7630)
* [AVERAGE PROFICIENCY ON THE CSAP/TCAP BY RACE/ETHNICITY: READING](http://datacenter.kidscount.org/data/tables/6727-average-proficiency-on-the-csap-tcap-by-race-ethnicity-reading?loc=7&loct=10#detailed/10/1278-1457/false/869,36,868,867/2756,2161,2159,2819,2157,2158,2820,2160/13821)
* [AVERAGE PROFICIENCY ON THE CSAP/TCAP BY RACE/ETHNICITY: MATH](http://datacenter.kidscount.org/data/tables/6567-average-proficiency-on-the-csap-tcap-by-race-ethnicity-math?loc=7&loct=10#detailed/10/1278-1457/false/869,36,868,867/2756,2161,2159,2819,2157,2158,2820,2160/13563)
* [AVERAGE PROFICIENCY ON THE CSAP/TCAP BY RACE/ETHNICITY: WRITING](http://datacenter.kidscount.org/data/tables/6729-average-proficiency-on-the-csap-tcap-by-race-ethnicity-writing?loc=7&loct=10#detailed/10/1278-1457/false/869,36,868,867/2756,2161,2159,2819,2157,2158,2820,2160/13823)
