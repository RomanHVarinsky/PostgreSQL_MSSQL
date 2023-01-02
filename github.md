### Importer adresser med bredbåndstype til LOIS

ogr2ogr -overwrite -f MSSQLSpatial MSSQL:"server=lois.sql.kerteminde.local;driver=SQL Server Native Client 11.0;database=LOIS;trusted_connection=yes" -nln bredbaand1_xls -lco schema=k440 "[filimport]" -progress -skipfailures -nlt none

### Denne forespørgsel opsummerer befolkningen i aldersgrupper:
SELECT

SUM(CASE when pers.alder between 0 and 120 then 1 else 0 end) as total,

SUM(CASE WHEN pers.Alder between 0 and 6 then 1 else 0 end) as antal_0_6,

SUM(CASE WHEN pers.Alder between 7 and 17 then 1 else 0 end) as antal_7_17,

SUM(CASE WHEN pers.Alder between 18 and 24 then 1 else 0 end) as antal_18_24,

SUM(CASE WHEN pers.Alder between 25 and 39 then 1 else 0 end) as antal_25_39,

SUM(CASE WHEN pers.Alder between 40 and 64 then 1 else 0 end) as antal_40_64,

SUM(CASE WHEN pers.Alder >= 65 then 1 else 0 end) as antal_over_65

FROM dbo.CPR_AktivKom_GeoView pers  

### Forespørgslen kan tilpasses så den ikke viser antallet af borgere i de enkelte celler, hvis der findes færre end ti personer i en bestemt kategori:
SELECT 
geom.gid,
geom.kn1kmdk,
CASE WHEN SUM(CASE WHEN cpr.Alder between 0 and 120 THEN 1 ELSE 0 END)>10 THEN SUM(CASE WHEN cpr.Alder between 0 and 120 THEN 1 ELSE 0 END) ELSE NULL END as Antal,
CASE WHEN SUM(CASE WHEN cpr.Alder between 0 and 6 THEN 1 ELSE 0 END)>10 THEN SUM(CASE WHEN cpr.Alder between 0 and 6 THEN 1 ELSE 0 END) ELSE NULL END as Antal_0_6, 
CASE WHEN SUM(CASE WHEN cpr.Alder between 7 and 17 THEN 1 ELSE 0 END)>10 THEN SUM(CASE WHEN cpr.Alder between 7 and 17 THEN 1 ELSE 0 END) ELSE NULL END as Antal_7_17, 
CASE WHEN SUM(CASE WHEN cpr.Alder between 18 and 24 THEN 1 ELSE 0 END)>10 THEN SUM(CASE WHEN cpr.Alder between 18 and 24 THEN 1 ELSE 0 END) ELSE NULL END as Antal_18_24, 
CASE WHEN SUM(CASE WHEN cpr.Alder between 25 and 39 THEN 1 ELSE 0 END)>10 THEN SUM(CASE WHEN cpr.Alder between 25 and 39 THEN 1 ELSE 0 END) ELSE NULL END as Antal_25_39, 
CASE WHEN SUM(CASE WHEN cpr.Alder between 40 and 64 THEN 1 ELSE 0 END)>10 THEN SUM(CASE WHEN cpr.Alder between 40 and 65 THEN 1 ELSE 0 END) ELSE NULL END as Antal_40_64, 
CASE WHEN SUM(CASE WHEN cpr.Alder >65 THEN 1 ELSE 0 END)>10 THEN SUM(CASE WHEN cpr.Alder > 65 THEN 1 ELSE 0 END) ELSE NULL END as Antal_over_65, 
geometry::STGeomFromText(geom.ogr_geometry.STAsText(),25832) as geometri
FROM dbo.CPR_AktivKom_GeoView cpr, k440.dkn_1km_euref89 geom WHERE geom.ogr_geometry.STContains(cpr.geometri)=1
GROUP BY geom.gid, geom.kn1kmdk, geom.ogr_geometry.STAsText()
ORDER BY geom.kn1kmdk

### Ovenstående har den ulempe, at den både viser totalen og tallet for hver aldersinterval hvis den er større end ti. Så er der kun én aldersinterval med færre end ti personer, kan man godt regne den ud.
### Så hvis en aldersinterval indeholder færre end ti personer, vises kun tallet for totalen. Altså med udgangspunkt at totalen er større end ti:
SELECT
geom.gid,
geom.kn1kmdk,
--SUM(CASE WHEN cpr.Alder between 0 and 120 THEN 1 ELSE 0 END) as Antal_test,
case when SUM(CASE WHEN cpr.Alder between 0 and 120 THEN 1 ELSE 0 END)<10 then NULL else sum(CASE WHEN cpr.Alder between 0 and 120 THEN 1 ELSE 0 END) end as antal_total_test,
--case when ((SUM(CASE WHEN cpr.Alder between 0 and 6 THEN 1 ELSE 0 END)<10 and SUM(CASE WHEN cpr.Alder between 7 and 17 THEN 1 ELSE 0 END)<10 and sum(CASE WHEN cpr.Alder between 18 and 24 THEN 1 ELSE 0 END)<10 and sum(CASE WHEN cpr.Alder between 25 and 39 THEN 1 ELSE 0 END)<10 and sum(CASE WHEN cpr.Alder between 40 and 65 THEN 1 ELSE 0 END)<10 and sum(CASE WHEN cpr.Alder > 65 THEN 1 ELSE 0 END)<10) or sum(CASE WHEN cpr.Alder between 0 and 120 THEN 1 ELSE 0 END)<10) then NULL else sum(CASE WHEN cpr.Alder between 0 and 120 THEN 1 ELSE 0 END) end as antal_total,
case when (SUM(CASE WHEN cpr.Alder between 0 and 6 THEN 1 ELSE 0 END)<10 or SUM(CASE WHEN cpr.Alder between 7 and 17 THEN 1 ELSE 0 END)<10 or sum(CASE WHEN cpr.Alder between 18 and 24 THEN 1 ELSE 0 END)<10 or sum(CASE WHEN cpr.Alder between 25 and 39 THEN 1 ELSE 0 END)<10 or sum(CASE WHEN cpr.Alder between 40 and 65 THEN 1 ELSE 0 END)<10 or sum(CASE WHEN cpr.Alder > 65 THEN 1 ELSE 0 END)<10) then NULL else sum(CASE WHEN cpr.Alder between 0 and 6 THEN 1 ELSE 0 END) end as antal_0_6,
case when (SUM(CASE WHEN cpr.Alder between 0 and 6 THEN 1 ELSE 0 END)<10 or SUM(CASE WHEN cpr.Alder between 7 and 17 THEN 1 ELSE 0 END)<10 or sum(CASE WHEN cpr.Alder between 18 and 24 THEN 1 ELSE 0 END)<10 or sum(CASE WHEN cpr.Alder between 25 and 39 THEN 1 ELSE 0 END)<10 or sum(CASE WHEN cpr.Alder between 40 and 65 THEN 1 ELSE 0 END)<10 or sum(CASE WHEN cpr.Alder > 65 THEN 1 ELSE 0 END)<10) then NULL else sum(CASE WHEN cpr.Alder between 7 and 17 THEN 1 ELSE 0 END) end as antal_7_17,
case when (SUM(CASE WHEN cpr.Alder between 0 and 6 THEN 1 ELSE 0 END)<10 or SUM(CASE WHEN cpr.Alder between 7 and 17 THEN 1 ELSE 0 END)<10 or sum(CASE WHEN cpr.Alder between 18 and 24 THEN 1 ELSE 0 END)<10 or sum(CASE WHEN cpr.Alder between 25 and 39 THEN 1 ELSE 0 END)<10 or sum(CASE WHEN cpr.Alder between 40 and 65 THEN 1 ELSE 0 END)<10 or sum(CASE WHEN cpr.Alder > 65 THEN 1 ELSE 0 END)<10) then NULL else sum(CASE WHEN cpr.Alder between 18 and 24 THEN 1 ELSE 0 END) end as antal_18_24,
case when (SUM(CASE WHEN cpr.Alder between 0 and 6 THEN 1 ELSE 0 END)<10 or SUM(CASE WHEN cpr.Alder between 7 and 17 THEN 1 ELSE 0 END)<10 or sum(CASE WHEN cpr.Alder between 18 and 24 THEN 1 ELSE 0 END)<10 or sum(CASE WHEN cpr.Alder between 25 and 39 THEN 1 ELSE 0 END)<10 or sum(CASE WHEN cpr.Alder between 40 and 65 THEN 1 ELSE 0 END)<10 or sum(CASE WHEN cpr.Alder > 65 THEN 1 ELSE 0 END)<10) then NULL else sum(CASE WHEN cpr.Alder between 25 and 39 THEN 1 ELSE 0 END) end as antal_25_39,
case when (SUM(CASE WHEN cpr.Alder between 0 and 6 THEN 1 ELSE 0 END)<10 or SUM(CASE WHEN cpr.Alder between 7 and 17 THEN 1 ELSE 0 END)<10 or sum(CASE WHEN cpr.Alder between 18 and 24 THEN 1 ELSE 0 END)<10 or sum(CASE WHEN cpr.Alder between 25 and 39 THEN 1 ELSE 0 END)<10 or sum(CASE WHEN cpr.Alder between 40 and 65 THEN 1 ELSE 0 END)<10 or sum(CASE WHEN cpr.Alder > 65 THEN 1 ELSE 0 END)<10) then NULL else sum(CASE WHEN cpr.Alder between 40 and 64 THEN 1 ELSE 0 END) end as antal_40_64,
case when (SUM(CASE WHEN cpr.Alder between 0 and 6 THEN 1 ELSE 0 END)<10 or SUM(CASE WHEN cpr.Alder between 7 and 17 THEN 1 ELSE 0 END)<10 or sum(CASE WHEN cpr.Alder between 18 and 24 THEN 1 ELSE 0 END)<10 or sum(CASE WHEN cpr.Alder between 25 and 39 THEN 1 ELSE 0 END)<10 or sum(CASE WHEN cpr.Alder between 40 and 65 THEN 1 ELSE 0 END)<10 or sum(CASE WHEN cpr.Alder > 65 THEN 1 ELSE 0 END)<10) then NULL else sum(CASE WHEN cpr.Alder > 65 THEN 1 ELSE 0 END) end as antal_over_65,
geometry::STGeomFromText(geom.ogr_geometry.STAsText(),25832) as geometri
from dbo.CPR_AktivKom_GeoView cpr, k440.dkn_1km_euref89 geom where geom.ogr_geometry.STContains(cpr.geometri)=1
group by geom.gid, geom.kn1kmdk, geom.ogr_geometry.STAsText()
order by geom.kn1kmdk



### Og denne forespørgsel opsummerer befolkning i aldersgrupper baseret på bredbåndstyperne: 
SELECT
--ROW_NUMBER() OVER (order by pers.standardadresse) AS fid,
--pers.Standardadresse,
--pers.POSTNR,
--pers.Postdistrikt,
x.listetype,
SUM(CASE when pers.alder between 0 and 120 then 1 else 0 end) as total,
SUM(CASE WHEN pers.Alder between 0 and 6 then 1 else 0 end) as antal_0_6,
SUM(CASE WHEN pers.Alder between 7 and 17 then 1 else 0 end) as antal_7_17,
SUM(CASE WHEN pers.Alder between 18 and 24 then 1 else 0 end) as antal_18_24,
SUM(CASE WHEN pers.Alder between 25 and 39 then 1 else 0 end) as antal_25_39,
SUM(CASE WHEN pers.Alder between 40 and 64 then 1 else 0 end) as antal_40_64,
SUM(CASE WHEN pers.Alder >= 65 then 1 else 0 end) as antal_over_65
FROM dbo.CPR_AktivKom_GeoView pers  
right join k440.bredbaand1_xls x on pers.AdgAdr_id=x.adgadr_id
GROUP BY x.listetype
