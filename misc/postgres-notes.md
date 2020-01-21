# General
## import shapefile to pg table
shp2pgsql -I usmx84/usmx84.shp usmx84 | psql -U m -d fcc2

## query ranged within to geoms
SELECT *
FROM fcclicenses  
WHERE ST_DWithin(geom, 
(SELECT ST_Union(geom) from usmx84)::geography,
1000) limit 20;

## How to double check the projections of files
SELECT Find_SRID('public', 'usmx', 'geom');

# Reports:

## All experimental radios within 100km of border


COPY (
SELECT *
FROM fcclicenses  
WHERE ST_DWithin(geom, 
(SELECT ST_Union(geom) from usmx84)::geography,
25000) 
AND source_system='ELS'
) TO '/tmp/experimental-25km-border.csv' With CSV HEADER DELIMITER ',';




## Count of companies within 5km of border for all license types

COPY (
SELECT *
FROM fcclicenses  
WHERE ST_DWithin(geom, 
(SELECT ST_Union(geom) from usmx84)::geography,
5000)
) TO '/tmp/allradios-5km-border.csv' With CSV HEADER DELIMITER ',';



## Distribution of license types for 10km within border; comparison with whole country

COPY (
SELECT rollup_category_desc, count(rollup_category_desc)
FROM fcclicensens  
WHERE ST_DWithin(geom, 
(SELECT ST_Union(geom) from usmx84)::geography,
10000) 
GROUP BY rollup_category_desc ORDER BY count desc
) TO '/tmp/license-rollup-10km.csv' With CSV HEADER DELIMITER ',';

COPY (
SELECT rollup_category_desc, count(rollup_category_desc)
FROM fcclicenses  
GROUP BY rollup_category_desc ORDER BY count desc
) TO '/tmp/license-rollup-all-us.csv' With CSV HEADER DELIMITER ',';


## Zoom on particular companies for all license types

COPY (
select * from fcclicenses where lic_name='IMSAR LLC'
) TO '/tmp/license-imsar-all-us.csv' With CSV HEADER DELIMITER ',';

COPY (
select * from fcclicenses where lic_name='ELTA North America'
) TO '/tmp/license-etla-na-all-us.csv' With CSV HEADER DELIMITER ',';

COPY (
select * from fcclicenses where lic_name='DRS%'
) TO '/tmp/license-drs-all-us.csv' With CSV HEADER DELIMITER ',';


