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
