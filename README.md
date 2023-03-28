

# Introduction



# Task
* Fixing Data Column e.g., Date Format
* Importing
* EDA

## Data Source

[Our World in Data](https://ourworldindata.org/covid-deaths)

# Data Preprocessing

## Importing

* We are inserting two csv files, each has around 270K rows, then joining them based on location (country)
``` 
="INSERT INTO TABLE_NAME VALUES('"&A2&"','"&B2&"','"&C2&"','"&D2&"','"&E2&"','"&F2&"', '"&G2&"','"&H2&"','"&I2&"','"&J2&"','"&K2&"','"&L2&"', '"&M2&"','"&N2&"','"&O2&"','"&P2&"','"&Q2&"','"&R2&"', '"&S2&"','"&T2&"','"&U2&"','"&V2&"','"&W2&"','"&X2&"','"&Y2&"','"&Z2&"');"
```

### For faster insert, use the command line 
```
mysql -u database_user -p database_password database_name < data.sql
```



## Fix Date Format

* Common Issues

1. Single quotes " ' " cause error while inserting, for a quick workaround, replace it with double quotion " in excel



2. MySQL standard format is [yyyy-mm-dd], so we need to convert it, the current date is dd-mm-yyyy
```
update covid_deaths set valid_date = concat(substring(date,7,4),'-',substring(date,4,2),'-',substring(date,1,2));
```

## Exploratory Data Analysis (EDA) - Queries

### Covid Death/Covid Cases
```
SELECT location, date, total_deaths, total_cases, (total_deaths/total_cases)*100 as death_rate
FROM `covid_deaths`
where location like '%Mexico%'
order by date
```

### Covid Cases/Population
```
SELECT location, date, population, total_cases, (total_cases/population)*100 as cases_population_rate
FROM `covid_deaths`
where location like '%states%'
order by date
```

### Order Deaths by Countries
```
-- Order Deaths by Countries
SELECT location, MAX(cast(total_deaths as INT)) as death_rate
FROM `covid_deaths`
where continent != '' -- some are empties and don't have "Null" inside
group by location
order by death_rate desc
```

### Order Deaths by Continent
```
-- Order Deaths by Continent
SELECT continent, MAX(cast(total_deaths as INT)) as continent_death_rate
FROM `covid_deaths`
where continent != '' -- some are empties and don't have "Null" inside
group by continent
order by continent_death_rate desc
```

### Global

```
--- global cases & deaths per day
SELECT date, SUM(CAST(new_cases as INT)) as global_new_cases, SUM(CAST(new_deaths as INT)) as global_new_deaths, (SUM(CAST(new_deaths as INT))/SUM(CAST(new_cases as INT)))*100 as global_death_percentage
FROM covid_deaths
where continent != ''
GROUP BY date
ORDER BY date ASC
```

```
-- show vaccinations per country
SELECT cod.continent, cod.location, cod.date, cod.population, cov.new_vaccinations
FROM covid_deaths cod
JOIN covid_vaccinations cov
on cod.location = cov.location and 
cod.date = cov.date
where cod.continent != '' and cod.location like '%CountryName%'
order by cod.date
limit 350
```

```
-- showing each country vaccinations daily growth
SELECT cod.continent, covid_deaths.location, cod.date, cod.population, covid_vaccinations.new_vaccinations
, SUM(cov.new_vaccinations) OVER(PARTITION BY cod.location order by cod.location, cod.date)
FROM covid_deaths AS cod
JOIN covid_vaccinations AS cov
on cod.location = cov.location and 
cod.date = cov.date
where cod.continent != '' and cod.location REGEXP 'Canada|Albania'
-- order by cod.date
limit 2000


-- SUM(cov.new_vaccinations) OVER(Partition by cod.location order by cod.location, cod.date) as cumulative_vaccinations
```


### Stored Procedures

* Show Covid Death/Covid Cases rates for a specific location (country)
```
DELIMITER $$
CREATE DEFINER=`UserName`@`HOST` PROCEDURE `get_covid_deaths_by_country`(IN `country` VARCHAR(30))
BEGIN

	SELECT location, date, total_deaths, total_cases,(total_deaths/total_cases)*100 as death_rate
    FROM `covid_deaths`
    where location like CONCAT('%', TRIM(IFNULL(country, '')), '%')
    order by date;

END$$
DELIMITER ;
```



### Tableau Viz Queries

```
-- 1
Select SUM(new_cases) as total_cases, SUM(cast(new_deaths as INT)) as total_deaths, SUM(cast(new_deaths as INT))/SUM(New_Cases)*100 as death_percentage
From covid_deaths
where continent is not null

```

```
-- 2
Select location as continent, SUM(cast(new_deaths as INT)) as total_deaths
From covid_deaths
Where continent = '' -- is null 
and location not in ('World', 'European Union', 'International') and location not like '%income%'
Group by location
order by total_deaths desc
```

```
-- 3
Select location, population, MAX(total_cases) as highest_infection,  Max((total_cases/population))*100 as percent_population_infected
From covid_deaths
Group by location, population
order by percent_population_infected desc
```

```
-- 4
Select location, population, date, MAX(total_cases) as highest_infection,  Max((total_cases/population))*100 as percent_population_infected
From covid_deaths
Group by location, population, date
order by percent_population_infected desc
```



## Credits
* Thanks to **[Alex The Analyst](https://www.youtube.com/@AlexTheAnalyst)** for proposing such insightful project


## Resources

### SQL Regex Functions
* https://www.tutorialspoint.com/mysql/mysql-regexps.htm