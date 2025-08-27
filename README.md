# I. Introduction
Báo cáo tập trung phân tích lượng khí thải carbon nhằm đánh giá dấu chân carbon của nhiều ngành công nghiệp theo quốc gia và giai đoạn. Khí thải carbon chiếm hơn 75% tổng phát thải toàn cầu, là nguyên nhân chính dẫn đến biến đổi khí hậu và các thảm họa môi trường. Kết quả phân tích giúp xác định ngành có mức phát thải cao nhất, nhận diện xu hướng, và hỗ trợ đưa ra quyết định hướng tới phát triển bền vững.
# II. Data Overview
## a/ Where does our data come from?
Bộ dữ liệu được tổng hợp từ nguồn công khai trên nature.com, bao gồm dấu chân carbon (PCF) của nhiều công ty. PCF thể hiện lượng khí thải nhà kính gắn liền với từng sản phẩm cụ thể, được đo lường bằng đơn vị CO₂ tương đương.
## b/ Data Structure
![](https://github.com/BangPham-ux/Project-2---Carbon-Emission-Analysis/blob/main/Database%20diagram.png)
## c/ Table review
```
SELECT *
FROM product_emissions
LIMIT 5
```
| id           | company_id | country_id | industry_group_id | year | product_name                                                    | weight_kg | carbon_footprint_pcf | upstream_percent_total_pcf | operations_percent_total_pcf | downstream_percent_total_pcf | 
| -----------: | ---------: | ---------: | ----------------: | ---: | --------------------------------------------------------------: | --------: | -------------------: | -------------------------: | ---------------------------: | ---------------------------: | 
| 10056-1-2014 | 82         | 28         | 2                 | 2014 | Frosted Flakes(R) Cereal                                        | 0.7485    | 2                    | 57.50                      | 30.00                        | 12.50                        | 
| 10056-1-2015 | 82         | 28         | 15                | 2015 | "Frosted Flakes, 23 oz, produced in Lancaster, PA (one carton)" | 0.7485    | 2                    | 57.50                      | 30.00                        | 12.50                        | 
| 10222-1-2013 | 83         | 28         | 8                 | 2013 | Office Chair                                                    | 20.68     | 73                   | 80.63                      | 17.36                        | 2.01                         | 
| 10261-1-2017 | 14         | 16         | 25                | 2017 | Multifunction Printers                                          | 110       | 1488                 | 30.65                      | 5.51                         | 63.84                        | 
| 10261-2-2017 | 14         | 16         | 25                | 2017 | Multifunction Printers                                          | 110       | 1818                 | 25.08                      | 4.51                         | 70.41                        | 
# Pre-processing
Xử lý giá trị trùng lặp
- Trước khi xử lý

|COUNT(*)|
|--------|
|1037|
- Sau khi xử lý

```
WITH handled_duplicates AS (
SELECT *,
ROW_NUMBER() OVER (PARTITION BY id,product_name, company_id, country_id, industry_group_id, year) AS rn
FROM product_emissions
)
SELECT COUNT(*)
FROM handled_duplicates
WHERE rn = 1
```

|COUNT(*)|
|--------|
|866|
# Analyse
Q1: Which products contribute the most to carbon emissions?
```
WITH handled_duplicates AS (
SELECT *,
ROW_NUMBER() OVER (PARTITION BY id,product_name, company_id, country_id, industry_group_id, year) AS rn
FROM product_emissions
)
SELECT product_name,
       ROUND(AVG(carbon_footprint_pcf), 2) AS avg_emission
FROM handled_duplicates
WHERE rn = 1
GROUP BY product_name
ORDER BY avg_emission DESC
LIMIT 10;
```
|product_name|avg_emission|
|------------|------------|
|Wind Turbine G128 5 Megawats|3718044.00|
|Wind Turbine G132 5 Megawats|3276187.00|
|Wind Turbine G114 2 Megawats|1532608.00|
|Wind Turbine G90 2 Megawats|1251625.00|
|Land Cruiser Prado. FJ Cruiser. Dyna trucks. Toyoace.IMV def unit.|191687.00|
|Retaining wall structure with a main wall (sheet pile): 136 tonnes of steel sheet piles and 4 tonnes of tierods per 100 meter wall|167000.00|
|TCDE|99075.00|
|Mercedes-Benz GLE (GLE 500 4MATIC)|91000.00|
|Mercedes-Benz S-Class (S 500)|85000.00|
|Mercedes-Benz SL (SL 350)|72000.00|

Q2: What are the industry groups of these products?
```
WITH handled_duplicates AS (
SELECT *,
ROW_NUMBER() OVER (PARTITION BY id,product_name, company_id, country_id, industry_group_id, year) AS rn
FROM product_emissions
)
SELECT 
	product_name,
	ig.industry_group,
	ROUND(AVG(carbon_footprint_pcf), 2) AS avg_emission
FROM handled_duplicates
JOIN industry_groups ig ON handled_duplicates.industry_group_id = ig.id
WHERE rn = 1
GROUP BY product_name, ig.industry_group
ORDER BY avg_emission DESC
LIMIT 10;
```
|product_name|industry_group|avg_emission|
|------------|--------------|------------|
|Wind Turbine G128 5 Megawats|Electrical Equipment and Machinery|3718044.00|
|Wind Turbine G132 5 Megawats|Electrical Equipment and Machinery|3276187.00|
|Wind Turbine G114 2 Megawats|Electrical Equipment and Machinery|1532608.00|
|Wind Turbine G90 2 Megawats|Electrical Equipment and Machinery|1251625.00|
|Land Cruiser Prado. FJ Cruiser. Dyna trucks. Toyoace.IMV def unit.|Automobiles & Components|191687.00|
|Retaining wall structure with a main wall (sheet pile): 136 tonnes of steel sheet piles and 4 tonnes of tierods per 100 meter wall|Materials|167000.00|
|TCDE|Materials|99075.00|
|Mercedes-Benz GLE (GLE 500 4MATIC)|Automobiles & Components|91000.00|
|Mercedes-Benz S-Class (S 500)|Automobiles & Components|85000.00|
|Mercedes-Benz SL (SL 350)|Automobiles & Components|72000.00|

Q3: What are the industries with the highest contribution to carbon emissions?
```
WITH handled_duplicates AS (
SELECT *,
ROW_NUMBER() OVER (PARTITION BY id,product_name, company_id, country_id, industry_group_id, year) AS rn
FROM product_emissions
)
SELECT 
	ig.industry_group,
	ROUND(SUM(carbon_footprint_pcf), 2) AS total_emission
FROM handled_duplicates
JOIN industry_groups ig ON handled_duplicates.industry_group_id = ig.id
WHERE rn = 1
GROUP BY ig.industry_group
ORDER BY total_emission DESC
```
|industry_group|total_emission|
|--------------|--------------|
|Electrical Equipment and Machinery|9801558.00|
|Automobiles & Components|2582264.00|
|Materials|430199.00|
|Technology Hardware & Equipment|278650.00|
|Capital Goods|258633.00|
|"Food, Beverage & Tobacco"|109132.00|
|"Pharmaceuticals, Biotechnology & Life Sciences"|72486.00|
|Software & Services|46533.00|
|Chemicals|44939.00|
|Media|23017.00|
|Energy|10774.00|
|"Forest and Paper Products - Forestry, Timber, Pulp and Paper, Rubber"|8909.00|
|"Mining - Iron, Aluminum, Other Metals"|8181.00|
|Consumer Durables & Apparel|7097.00|
|Commercial & Professional Services|4925.00|
|Containers & Packaging|2988.00|
|Tires|2022.00|
|Food & Staples Retailing|1481.00|
|"Consumer Durables, Household and Personal Products"|931.00|
|Telecommunication Services|418.00|
|Trading Companies & Distributors and Commercial Services & Supplies|239.00|
|"Textiles, Apparel, Footwear and Luxury Goods"|228.00|
|Food & Beverage Processing|138.00|
|Utilities|122.00|
|Gas Utilities|61.00|
|Semiconductors & Semiconductor Equipment|52.00|
|Retailing|22.00|
|Semiconductors & Semiconductors Equipment|3.00|
|Tobacco|1.00|
|Household & Personal Products|0.00|

Q4: What are the companies with the highest contribution to carbon emissions?
```
WITH handled_duplicates AS (
SELECT *,
ROW_NUMBER() OVER (PARTITION BY id,product_name, company_id, country_id, industry_group_id, year) AS rn
FROM product_emissions
)
SELECT 
	c.company_name,
	ROUND(SUM(carbon_footprint_pcf), 2) AS total_emission
FROM handled_duplicates
JOIN companies c  ON handled_duplicates.company_id = c.id
WHERE rn = 1
GROUP BY c.company_name
ORDER BY total_emission DESC
LIMIT 10;
```
|company_name|total_emission|
|------------|--------------|
|"Gamesa Corporación Tecnológica, S.A."|9778464.00|
|Daimler AG|1594300.00|
|Volkswagen AG|655960.00|
|"Hino Motors, Ltd."|191687.00|
|Arcelor Mittal|167007.00|
|Weg S/A|160655.00|
|General Motors Company|137007.00|
|"Mitsubishi Gas Chemical Company, Inc."|106008.00|
|"Daikin Industries, Ltd."|105600.00|
|CJ Cheiljedang|94817.00|

Q5: What are the countries with the highest contribution to carbon emissions?
```
WITH handled_duplicates AS (
SELECT *,
ROW_NUMBER() OVER (PARTITION BY id,product_name, company_id, country_id, industry_group_id, year) AS rn
FROM product_emissions
)
SELECT 
	c.country_name,
	ROUND(SUM(carbon_footprint_pcf), 2) AS total_emission
FROM handled_duplicates
JOIN countries c  ON handled_duplicates.country_id = c.id
WHERE rn = 1
GROUP BY c.country_name
ORDER BY total_emission DESC
LIMIT 10;
```
|country_name|total_emission|
|------------|--------------|
|Spain|9786127.00|
|Germany|2251225.00|
|Japan|519348.00|
|USA|451867.00|
|Brazil|167587.00|
|Luxembourg|167007.00|
|South Korea|140995.00|
|Netherlands|70417.00|
|Taiwan|61511.00|
|India|24574.00|

Q6: What is the trend of carbon footprints (PCFs) over the years?
```
WITH handled_duplicates AS (
SELECT *,
ROW_NUMBER() OVER (PARTITION BY id,product_name, company_id, country_id, industry_group_id, year) AS rn
FROM product_emissions
)
SELECT 
	year,
	ROUND(SUM(carbon_footprint_pcf), 2) AS total_emission
FROM handled_duplicates
WHERE rn = 1
GROUP BY year
```
|year|total_emission|
|----|--------------|
|2013|496076.00|
|2014|548229.00|
|2015|10810407.00|
|2016|1612760.00|
|2017|228531.00|

Q7: Which industry groups has demonstrated the most notable decrease in carbon footprints (PCFs) over time?
```
WITH handled_duplicates AS (
SELECT *,
ROW_NUMBER() OVER (PARTITION BY id,product_name, company_id, country_id, industry_group_id, year) AS rn
FROM product_emissions
)
SELECT
	ig.industry_group,
	year,
	ROUND(SUM(carbon_footprint_pcf), 2) AS total_emission
FROM handled_duplicates
JOIN industry_groups ig ON handled_duplicates.industry_group_id = ig.id
WHERE rn = 1
GROUP BY ig.industry_group, year
ORDER BY ig.industry_group, year, total_emission DESC
```
|industry_group|year|total_emission|
|--------------|----|--------------|
|"Consumer Durables, Household and Personal Products"|2015|931.00|
|"Food, Beverage & Tobacco"|2013|4308.00|
|"Food, Beverage & Tobacco"|2014|2023.00|
|"Food, Beverage & Tobacco"|2015|0.00|
|"Food, Beverage & Tobacco"|2016|99639.00|
|"Food, Beverage & Tobacco"|2017|3162.00|
|"Forest and Paper Products - Forestry, Timber, Pulp and Paper, Rubber"|2015|8909.00|
|"Mining - Iron, Aluminum, Other Metals"|2015|8181.00|
|"Pharmaceuticals, Biotechnology & Life Sciences"|2013|32271.00|
|"Pharmaceuticals, Biotechnology & Life Sciences"|2014|40215.00|
|"Textiles, Apparel, Footwear and Luxury Goods"|2015|228.00|
|Automobiles & Components|2013|130189.00|
|Automobiles & Components|2014|230015.00|
|Automobiles & Components|2015|817227.00|
|Automobiles & Components|2016|1404833.00|
|Capital Goods|2013|60117.00|
|Capital Goods|2014|93699.00|
|Capital Goods|2015|3505.00|
|Capital Goods|2016|6369.00|
|Capital Goods|2017|94943.00|
|Chemicals|2015|44939.00|
|Commercial & Professional Services|2013|817.00|
|Commercial & Professional Services|2014|477.00|
|Commercial & Professional Services|2016|2890.00|
|Commercial & Professional Services|2017|741.00|
|Consumer Durables & Apparel|2013|2860.00|
|Consumer Durables & Apparel|2014|3123.00|
|Consumer Durables & Apparel|2016|1114.00|
|Containers & Packaging|2015|2988.00|
|Electrical Equipment and Machinery|2015|9801558.00|
|Energy|2013|750.00|
|Energy|2016|10024.00|
|Food & Beverage Processing|2015|138.00|
|Food & Staples Retailing|2014|773.00|
|Food & Staples Retailing|2015|706.00|
|Food & Staples Retailing|2016|2.00|
|Gas Utilities|2015|61.00|
|Household & Personal Products|2013|0.00|
|Materials|2013|194464.00|
|Materials|2014|66719.00|
|Materials|2016|61887.00|
|Materials|2017|107129.00|
|Media|2013|9645.00|
|Media|2014|9645.00|
|Media|2015|1919.00|
|Media|2016|1808.00|
|Retailing|2014|11.00|
|Retailing|2015|11.00|
|Semiconductors & Semiconductor Equipment|2014|50.00|
|Semiconductors & Semiconductor Equipment|2016|2.00|
|Semiconductors & Semiconductors Equipment|2015|3.00|
|Software & Services|2013|3.00|
|Software & Services|2014|143.00|
|Software & Services|2015|22851.00|
|Software & Services|2016|22846.00|
|Software & Services|2017|690.00|
|Technology Hardware & Equipment|2013|60539.00|
|Technology Hardware & Equipment|2014|101153.00|
|Technology Hardware & Equipment|2015|93807.00|
|Technology Hardware & Equipment|2016|1285.00|
|Technology Hardware & Equipment|2017|21866.00|
|Telecommunication Services|2013|52.00|
|Telecommunication Services|2014|183.00|
|Telecommunication Services|2015|183.00|
|Tires|2015|2022.00|
|Tobacco|2015|1.00|
|Trading Companies & Distributors and Commercial Services & Supplies|2015|239.00|
|Utilities|2013|61.00|
|Utilities|2016|61.00|
