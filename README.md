# SQL-Zoo-Query
SQL analysis of a zoo management database using Microsoft SQL Server. The database tracks animals, zoos, employees, and species across multiple facilities in different countries. 20 queries answer operational and business questions using joins across 3+ tables, subqueries, HAVING clauses, self-joins, and aggregate functions.

Zoo: ZooName (PK), City, Country
Animal: AnimalId (PK), ZooName (FK), SpeciesName (FK), MotherAnimalId (FK, self-referencing), FatherAnimalId (FK, self-referencing), DateOfBirth
Employee: EmpNo (PK), EmpName, ZooName (FK), SpeciesExpertise (FK), Salary, Gender
Species: SpeciesName (PK), Status (E = endangered, T = threatened), WorldBestExpertEmpNo (FK to Employee)

1. Species count per zoo, ranked

SELECT A.ZooName, COUNT(DISTINCT A.SpeciesName) AS NumberOfSpecies 
FROM Animal A
GROUP BY A.ZooName
ORDER BY NumberOfSpecies DESC

_Counts distinct species held at each zoo and ranks zoos from most to least diverse. Uses GROUP BY with COUNT DISTINCT._

2. Zoos with more than 2 Pandas (correlated subquery with HAVING)

SELECT * FROM Zoo Z
WHERE Z.ZooName IN
(SELECT A.ZooName FROM Animal A
WHERE A.SpeciesName = 'Panda'
GROUP BY A.ZooName
HAVING COUNT(*) > 2)

_Filters zoos down to only those holding more than 2 pandas. The subquery does the counting, the outer query pulls full zoo detail for matches only._

3. Animals whose mother lives in a different zoo (self-join)

SELECT DISTINCT A.* FROM Animal AM 
INNER JOIN Animal A ON AM.AnimalId = A.MotherAnimalId
WHERE A.ZooName != AM.ZooName AND A.MotherAnimalId IS NOT NULL

_Joins the Animal table to itself to compare each animal's zoo against its mother's current zoo. Surfaces cases where an animal was relocated separately from its mother, useful for tracking transfers._

4. Fathers with more than 2 offspring in Canadian zoos (nested subquery, 3 levels)

SELECT F.* FROM Animal F
WHERE F.AnimalId IN
(SELECT FatherAnimalId FROM Animal O
WHERE O.ZooName IN
(SELECT ZooName FROM Zoo WHERE Country = 'Canada')
GROUP BY FatherAnimalId
HAVING COUNT(*) > 2)

_Three nested levels: innermost finds Canadian zoo names, middle groups offspring by father and counts them, outer pulls the father's full record. Demonstrates handling multi-step business logic in a single query._

5. Species expert count by species, filtered to USA (3-table join)

SELECT COUNT(E.EmpNo) AS "Number of experts", S.SpeciesName 
FROM Employee E 
INNER JOIN Species S ON S.SpeciesName = E.SpeciesExpertise 
INNER JOIN Zoo Z ON E.ZooName = Z.ZooName
WHERE Country = 'USA'
GROUP BY S.SpeciesName

_Joins Employee, Species, and Zoo to count how many experts per species work specifically at US zoos. Shows a standard 3-table join with a geographic filter._
