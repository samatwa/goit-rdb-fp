# goit-rdb-fp
1)
CREATE SCHEMA pandemic;
USE pandemic;
SELECT * FROM infectious_cases;

2)
CREATE TABLE entities (
    entity_id INT AUTO_INCREMENT PRIMARY KEY,
    Entity TEXT,
    Code TEXT
);

INSERT INTO entities (Entity, Code)
SELECT DISTINCT Entity, Code 
FROM infectious_cases;

CREATE TABLE normalized_cases (
    case_id INT AUTO_INCREMENT PRIMARY KEY,
    entity_id INT,
    Year INT,
    Number_yaws TEXT,
    polio_cases INT,
    cases_guinea_worm INT,
    Number_rabies TEXT,
    Number_malaria TEXT,
    Number_hiv TEXT,
    Number_tuberculosis TEXT,
    Number_smallpox TEXT,
    Number_cholera_cases TEXT,
    FOREIGN KEY (entity_id) REFERENCES entities(entity_id)
);

INSERT INTO normalized_cases (
    entity_id, Year, Number_yaws, polio_cases, cases_guinea_worm,
    Number_rabies, Number_malaria, Number_hiv, Number_tuberculosis,
    Number_smallpox, Number_cholera_cases
)
SELECT 
    e.entity_id, ic.Year,
    ic.Number_yaws, ic.polio_cases,
    ic.cases_guinea_worm, ic.Number_rabies,
    ic.Number_malaria, ic.Number_hiv,
    ic.Number_tuberculosis, ic.Number_smallpox,
    ic.Number_cholera_cases
FROM infectious_cases ic
JOIN entities e ON ic.Entity = e.Entity AND ic.Code = e.Code;

3)
SELECT 
    e.Entity,
    e.Code,
    ROUND(AVG(CAST(NULLIF(nc.Number_rabies, '') AS DECIMAL(10,2))),2) as avg_rabies,
    MIN(CAST(NULLIF(nc.Number_rabies, '') AS DECIMAL(10,2))) as min_rabies,
    MAX(CAST(NULLIF(nc.Number_rabies, '') AS DECIMAL(10,2))) as max_rabies,
    SUM(CAST(NULLIF(nc.Number_rabies, '') AS DECIMAL(10,2))) as total_rabies
FROM normalized_cases AS nc
JOIN entities e ON nc.entity_id = e.entity_id
WHERE nc.Number_rabies != ''
GROUP BY e.Entity, e.Code
ORDER BY avg_rabies DESC
LIMIT 10;

4)
SELECT 
    DISTINCT Year,
    MAKEDATE(Year, 1) AS year_date,
    CURDATE() AS today_date,
    TIMESTAMPDIFF(YEAR, MAKEDATE(Year, 1), CURDATE()) AS years_difference
FROM normalized_cases;

5)
5.1)
DELIMITER //

CREATE FUNCTION CalculateDateDiff(Year INT)
RETURNS INT
DETERMINISTIC 
NO SQL
BEGIN
	DECLARE result INT;
    SET result = TIMESTAMPDIFF(YEAR, MAKEDATE(Year, 1), CURDATE());
    RETURN result;
END //

DELIMITER ;

SELECT DISTINCT Year, CalculateDateDiff(Year) as years_diff
FROM normalized_cases;

5.2)
DELIMITER //

CREATE FUNCTION CalculatePeriodCases(cases_per_year TEXT, period_divider INT)
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE result DECIMAL(10,2);
    IF cases_per_year = '' OR cases_per_year IS NULL THEN
        RETURN NULL;
    END IF;
    SET result = CAST(cases_per_year AS DECIMAL(10,2)) / period_divider;
    RETURN result;
END //

DELIMITER ;

SELECT 
    Entity,
    Year,
    Number_rabies as yearly_cases,
    CalculatePeriodCases(Number_rabies, 12) as monthly_cases,
    CalculatePeriodCases(Number_rabies, 4) as quarterly_cases,
    CalculatePeriodCases(Number_rabies, 2) as semi_annual_cases
FROM normalized_cases nc
JOIN entities e ON nc.entity_id = e.entity_id
WHERE Number_rabies != ''
LIMIT 10;



 
