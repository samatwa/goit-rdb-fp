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



 
