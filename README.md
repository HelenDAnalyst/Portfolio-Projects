# Project 1: Exploratory Analysis - Covid 19

The is a data exploration project designed to interpret and analyze information about Covid-19 pandemic worldwide. The project uses two datasets, CovidDeaths.xlsx and CovidVaccinations.xlsx, which have been cleaned in Excel and loaded into a Microsoft SQL Server database for data exploration.

## Files
The project includes the following files:

- **CovidDeaths.xlsx** - the dataset containing information on Covid-19 deaths worldwide

- **CovidVaccinations.xlsx** - the dataset containing information on Covid-19 vaccinations worldwide


- **Covid Portfolio Project.sql** - the SQL code used to explore the data in the database


## Data Exploration
The **Covid Portfolio Project.sql** file contains SQL queries for exploring the Covid-19 data. The following queries have been used in this project:

These SQL queries are used to explore and analyze COVID-19 data for a data analysis project. They were executed in Microsoft SQL Server Management Studio and were used to generate visualizations and insights about the COVID-19 pandemic.

### Part - 1


The first two queries are simply selecting and ordering the data from the CovidDeaths and CovidVaccinations tables.

The next several queries are looking at different aspects of the CovidDeaths data. 
- looking at total cases vs population
- identifying the countries with the highest infection rates compared to population,
- looking at total cases vs total death
- looking at total cases vs total population in a country
- identifying countries with the highest death counts per population.
- looking at contintents with the highest death count per population
- looking at global numbers
- looking at total population vs total tests in continents
- Looking at total population vs vaccination in continents
- Using CTE to perform calculation on partition by in previous queries
- Using temp table to perform calculation on partition by in previous query
- Creating view for Data visualization


Overall, these queries are useful for gaining insights and understanding trends in COVID-19 data for analysis and visualization purposes.

```sql
Select *
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
Order By 3,4 

Select *
FROM PortfolioProject..CovidVaccinations
WHERE continent IS NOT NULL
Order By 3,4

--Select the data that we going to be using

Select Location, date, total_cases, new_cases, total_deaths, population
FROM PortfolioProject..CovidDeaths
order by 1,2


--Looking at total cases vs population

Select Location, date, population, total_cases, (total_cases/population)*100 as PercentPopulationInfected
FROM PortfolioProject..CovidDeaths
Where location like '%desh%'
order by 1,2


--Looking at countries with highest infection rate compared to population

Select Location, population, MAX (total_cases) as HighestInfectionCOunt, MAX ((total_cases/population))*100 as PercentPopulationInfected
FROM PortfolioProject..CovidDeaths
--Where continent is not null
Group By Location, population
order by PercentPopulationInfected desc


--Looking at total cases vs total deaths

Select Location, date, total_cases, total_deaths,(total_deaths/total_cases)*100 as DeathPercentage
FROM PortfolioProject..CovidDeaths
Where continent is not null
order by 1,2


--Looking at total cases vs total deaths in Nigeria

Select Location, date, total_cases, total_deaths,(total_deaths/total_cases)*100 as DeathPercentage
FROM PortfolioProject..CovidDeaths
Where location like '%Nigeria%'
order by 1,2


--Showing countries with highest death count per population 

Select Location, MAX (cast (Total_deaths as int)) as TotalDeathCount
FROM PortfolioProject..CovidDeaths
--Where location like '%desh%'
Where continent is not null
Group By Location
order by TotalDeathCount desc


-- BREAKING THINGS DOWN BY CONTINENT

-- Showing contintents with the highest death count per population

Select continent, MAX(cast(Total_deaths as int)) as TotalDeathCount
From PortfolioProject..CovidDeaths
--Where location like '%states%'
Where continent is not null 
Group by continent
order by TotalDeathCount desc


-- GLOBAL NUMBERS

Select location, date, total_cases, total_deaths, (CONVERT(FLOAT, total_deaths)/NULLIF(CONVERT(FLOAT, total_cases), 0))*100 AS DeathPercentage
From PortfolioProject..CovidDeaths
--Where location like '%states%'
where continent is not null 
order by 1,2


-- Total Population vs Total Tests

Select dea.continent, dea.location, dea.date, dea.population, vac.new_tests
SUM(CAST(vac.new_tests AS INT)) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date)
AS RollingPeopleTested
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
order by 1,2,3


-- Total Population vs Vaccinations

Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
SUM(CAST(vac.new_vaccinations AS INT)) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date)
AS RollingPeopleVaccinated
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
order by 1,2,3


-- Using CTE to perform Calculation on Partition By in previous queries

With PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleTested)
as
(
Select dea.continent, dea.location, dea.date, dea.population, vac.new_tests
, SUM(CONVERT(bigint,vac.new_tests)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopletested
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
--order by 2,3
)
Select *, (RollingPeopleTested/Population)*100
From PopvsVac


With PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated)
as
(
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(bigint,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
--order by 2,3
)
Select *, (RollingPeopleVaccinated/Population)*100
From PopvsVac



-- Using Temp Table to perform Calculation on Partition By in previous query

DROP Table if exists #PercentPopulationVaccinated
Create Table #PercentPopulationVaccinated
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population numeric,
New_vaccinations numeric,
RollingPeopleVaccinated numeric
)

Insert into #PercentPopulationVaccinated
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
--where dea.continent is not null 
--order by 2,3

Select *, (RollingPeopleVaccinated/Population)*100
From #PercentPopulationVaccinated



-- Creating View to store data for later visualizations

Create View CaseFatalityRate as
Select Location, date, total_cases, total_deaths,(total_deaths/total_cases)*100 as DeathPercentage
FROM PortfolioProject.dbo.CovidDeaths
Where continent is not null


Create View TotalDeathCount as
Select Location, MAX (cast (Total_deaths as int)) as TotalDeathCount
FROM PortfolioProject.dbo.CovidDeaths
--Where location like '%desh%'
Where continent is not null
Group By Location


Create View PopulationTested
Select dea.continent, dea.location, dea.date, dea.population, vac.new_tests
, SUM(CONVERT(bigint,vac.new_tests)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopletested
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null


Create View PercentPopulationVaccinated as
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 

```
Select all data from the CovidDeaths table and order it by date and location.
<img src = "/Screenshots/sql-1.JPG">
-----------------------------------------------------------------------------------

Select all data from the CovidVaccinations table and order it by date and location.
<img src = "/Screenshots/sql-2.JPG">
-----------------------------------------------------------------------------------

Select the data that we are going to be using for the analysis, including location, date, total_cases, new_cases, total_deaths, and population. Order the data by location and date.

<img src = "/Screenshots/sql-3.JPG">
------------------------------------------------------------------------------------------------------------------------------------------

Look at total cases vs population to show what percentage of the population has been infected with Covid-19. Order the data by location and date.

<img src = "/Screenshots/sql-5.JPG">
----------------------------------------------------------------------------------------------------------------------------

Look at countries with the highest infection rate compared to population. Group the data by location and population and order it by the percentage of the population infected in descending order.

<img src = "/Screenshots/sql-6.JPG"

-----------------------------------------------------------------------------------

Look at total cases vs total deaths to show the likelihood of dying if you contract Covid-19 in your country. Order the data by location and date.

<img src = "/Screenshots/sql-4.JPG">
-----------------------------------------------------------------------------------

Look at total cases vs total deaths to show the likelihood of dying if you contract Covid-19 in Nigeria.

-----------------------------------------------------------------------------------

Show countries with the highest death count per population. Group the data by location and order it by the total death count in descending order.

<img src = "/Screenshots/sql-7.JPG">

-----------------------------------------------------------------------------------

#### Show continents with the highest death count per population.
```sql
--Break things down by continent
--Showing continent with highest death count per population

Select continent, MAX (cast (Total_deaths as int)) as TotalDeathCount
FROM PortfolioProject.dbo.CovidDeaths
--Where location like '%states%'
Where continent is not null
Group By continent
order by TotalDeathCount desc

```
This query extracts the total death count for each continent and displays the continent with the highest death count per population. 

- The MAX function is used to find the maximum value of the Total_deaths column for each continent. 
- The CAST function is used to convert the Total_deaths column to an integer data type. 
The result is ordered in descending order by the TotalDeathCount.

<img src = "/Screenshots/sql-8.JPG">

#### Global Numbers



### Part - 2


These SQL queries were used in the COVID-19 Portfolio Project for further deep analysis. 


SELECT location, date, total_cases, (convert(float, total_deaths)/nullif(convert(float, total_cases), 0)) *100 AS DeathPercentage
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
ORDER BY 1,2

```

These two queries extract global COVID-19 data. 
- The first query calculates the total number of cases, deaths, and the death percentage on each day. 

<img src = "/Screenshots/sql-9.JPG">

-----------------------------------------------------------------------------------

#### Total population vs tests
```sql
--Total population vs tests 

SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_tests,
SUM(CAST(vac.new_tests AS INT)) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date)
AS RollingPeopleTested
FROM PortfolioProject..CovidDeaths dea
JOIN  PortfolioProject..CovidVaccinations vac 
ON dea.location=vac.location
AND dea.date=vac.date
WHERE dea.continent IS NOT NULL
ORDER BY 1,2,3
```

#### Total population vs vaccination
```sql
--Total population vs vaccination 

SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
SUM(CAST(vac.new_vaccinations AS INT)) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date)
AS RollingPeopleVaccinated
FROM PortfolioProject..CovidDeaths dea
JOIN  PortfolioProject..CovidVaccinations vac 
ON dea.location=vac.location
AND dea.date=vac.date
WHERE dea.continent IS NOT NULL
ORDER BY 1,2,3
```

This query compares the total population of a location with the number of people who have been vaccinated. It uses the SUM function with the OVER clause to calculate the rolling sum of new_vaccinations for each location. The result is ordered by location and date.

<img src = "/Screenshots/sql-11.JPG">

-----------------------------------------------------------------------------------

#### Using CTE to perform calculation on partition by in previous queries
```sql

WITH PopvsVac(continent, location, date, population, new_tests, RollingPeopleTested)
AS
(
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_tests,
SUM(CAST(vac.new_tests AS BIGINT)) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date)
AS RollingPeopleTested
FROM PortfolioProject..CovidDeaths dea
JOIN  PortfolioProject..CovidVaccinations vac 
ON dea.location=vac.location
AND dea.date=vac.date


WITH PopvsVac(continent, location, date, population, new_vaccinations, RollingPeopleVaccinated)
AS
(
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
SUM(CAST(vac.new_vaccinations AS BIGINT)) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date)
AS RollingPeopleVaccinated
FROM PortfolioProject..CovidDeaths dea
JOIN  PortfolioProject..CovidVaccinations vac 
ON dea.location=vac.location
AND dea.date=vac.date
```
The above query uses a Common Table Expression (CTE) to define a temporary result set that can be used later in the query. 
In this case, 
- The CTE named "PopvsVac" combines data from two tables, CovidDeaths and CovidVaccinations, to calculate the rolling number of people tested and vaccinated in each location. 
- The CTE is then used in subsequent queries to calculate the percentage of population tested and vaccinated.

Using a CTE has several benefits for this project. 
- It allows us to break down complex queries into smaller, more manageable pieces, which can make them easier to read and understand. 
- It also helps to reduce code duplication, since the result set can be used multiple times in the query. 
- Finally, it can improve performance by allowing the database engine to optimize the execution plan for the entire query, rather than treating each subquery separately.

Overall, using a CTE can make queries more efficient, easier to read, and easier to maintain, which is especially important for complex data analysis projects.

<img src = "/Screenshots/sql-12.JPG">
<img src = "/Screenshots/sql-13.JPG">

-----------------------------------------------------------------------------------

#### Using a temp table
```sql
-- Create temporary table
DROP table if exists #PercentPopulationVaccinated
Create table #PercentPopulationVaccinated
(
  continent nvarchar(255),
  location nvarchar (255),
  date datetime,
  population numeric,
  new_vaccinations numeric,
  RollingPeopleVaccinated numeric
)

-- Insert data into temporary table
insert into #PercentPopulationVaccinated
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
SUM (cast(vac.new_vaccinations as int)) OVER (Partition by dea.location Order BY dea.location, dea.date)
as RollingPeopleVaccinated
From PortfolioProject..CovidDeaths dea
JOIN  PortfolioProject..CovidVaccinations vac 
On dea.location=vac.location
and dea.date=vac.date
--Where dea.continent is not null
--Order By 2,3

-- Calculate percentage of population vaccinated
Select *, (RollingPeopleVaccinated/population)*100
From #PercentPopulationVaccinated

-- Create view to store data for later visualizations
Create View PercentPopulationVaccinated as
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
SUM (cast(vac.new_vaccinations as int)) OVER (Partition by dea.location Order BY dea.location, dea.date)
as RollingPeopleVaccinated
From PortfolioProject..CovidDeaths dea
JOIN  PortfolioProject..CovidVaccinations vac 
On dea.location=vac.location
and dea.date=vac.date
Where dea.continent is not null
--Order By 2,3

Select *, (RollingPeopleVaccinated/Population)*100 AS PercentPeopleVaccinated
from #PercentPopulationVaccinated
```

This query is used to calculate the percentage of the population vaccinated against COVID-19 in different countries and continents. 
- It first creates a temporary table named #PercentPopulationVaccinated and inserts data into it by joining the CovidDeaths and CovidVaccinations tables on location and date. 
- It then calculates the rolling sum of new vaccinations for each location using the SUM function with the OVER clause. The results are stored in the RollingPeopleVaccinated column. 
- Finally, the query calculates the percentage of the population vaccinated by dividing RollingPeopleVaccinated by population and multiplying by 100.


These queries are useful for the project because they provide a way to track the progress of COVID-19 vaccination efforts in different regions and compare vaccination rates between countries and continents.

<img src = "/Screenshots/sql-14.JPG">

#### Data Visualizations
select*
from CaseFatalityRate

select*
from TotalDeathCount

select*
from PopulationTested

select*
from PopulationVaccinated

## Usage

To use this project, you will need to have Microsoft SQL Server installed. You can then download the **CovidDeaths.xlsx** and **CovidVaccinations.xlsx** datasets, clean them in Excel, and load them into the SQL Server. Finally, you can run the SQL queries from the **Covid Portfolio Project.sql** file to explore the data.
