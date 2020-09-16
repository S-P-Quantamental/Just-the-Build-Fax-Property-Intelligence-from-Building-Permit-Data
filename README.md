# Just-the-Build-Fax-Property-Intelligence-from-Building-Permit-Data
Just the (Build)Fax: Property Intelligence from Building Permit Data


The following code generates scores per REIT in accordance with the formulae provided in the paper:

```SQL
SELECT
	PRAT2019.INSTITUTIONID AS INSTITUTIONID
	,PRAT2019.nmanyPermitProperties2019 AS nmanyPermitProperties2019
	,PRAT2019.nmanyProps2019 AS nmanyProps2019
	,nmanyPermitProperties2019/nmanyProps2019 AS Pratio2019
	,PRAT2020.nmanyPermitProperties2020 AS nmanyPermitProperties2020
	,PRAT2020.nmanyProps2020 AS nmanyProps2020
	,nmanyPermitProperties2020/nmanyProps2020 AS Pratio2020
	,Pratio2020-Pratio2019 as REIT_Permit_Score
FROM
(
	SELECT
		PermitProp2019.INSTITUTIONID
		,PermitProp2019.nmanyPermitProperties2019
		,TotalProps2019.nmanyProps2019
	FROM
	(
	-- Counts the properties with at least 1 permit filed in 2019
	SELECT 
		S.INSTITUTIONID														                			-- snl ID (specifies the REIT)
		, COUNT( DISTINCT P.PROPERTYID) as nmanyPermitProperties2019		-- count # of properties that filed a permit
	FROM "MP_DB"."BUILDFAX"."BUILDFAXPERMITINSIGHT" P									-- BuildFax Permit Table 
	JOIN "MI_XPRESSCLOUD"."XPRESSFEED"."SNLPROPOWNERSHIP" S           -- XF SNLpropOwnership
	ON S.propertyID=P.propertyID					                            -- JOIN on the propertyID
	WHERE 1=1
	AND (cast(permitDate AS DATE) >= ownershipStartDate or ownershipStartDate is NULL)	
                                                                    -- The property's permit was filed after the REIT owned it (assumes ownership if start is NULL)
	AND (cast(permitDate as DATE) <= ownershipEndDate or currentOwner = 1)				
                                                                    -- The property's permit was filed before the REIT stopped owning it (assumes ownership if end is NULL)
	AND Month(cast(PermitDate as date)) in (1,2,3,4,5,6,7)						-- The permit was filed in the first 7 months of the year
	AND Year(cast(PermitDate as Date))=2019												    -- The permit was filed in 2019
	GROUP BY S.INSTITUTIONID			
	) PermitProp2019
	JOIN
	(
	-- Select the number of properties owned by the REIT between 1/1 and 7/31 in 2019
	SELECT
		INSTITUTIONID																	                  -- SNL ID (REIT)
		, count(distinct PROPERTYID) as nmanyProps2019									-- Counts Distinct Properties Per REIT	
	from "MI_XPRESSCLOUD"."XPRESSFEED"."SNLPROPOWNERSHIP" S 
	where (ownershipStartDate < '07/31/2019' or ownershipStartDate is NULL)				
                                                                    -- Where the REIT Owned the Property before end of July 2019 (assumes ownership if start is NULL)
	and (ownershipEndDate > '01/01/2019' or currentOwner=1)						-- And didn't stop owning the property before start of 2019 (assumes ownership if end if NULL. With previous condition this means property was owned during 2019)
	--and institutionID = 13485338 -- DEBUG
	group by S.INSTITUTIONID		
	) TotalProps2019
	ON PermitProp2019.INSTITUTIONID=TotalProps2019.INSTITUTIONID
) PRat2019
JOIN
(        
	SELECT
	  PermitProp2020.INSTITUTIONID
	  ,PermitProp2020.nmanyPermitProperties2020
	  ,TotalProps2020.nmanyProps2020
	FROM
	(
	-- Counts the properties with at least 1 permit filed in 2020
	SELECT 
		S.INSTITUTIONID																	                -- snl ID (specifies the REIT)
		, COUNT( DISTINCT P.PROPERTYID) as nmanyPermitProperties2020		-- count # of properties that filed a permit
	FROM "MP_DB"."BUILDFAX"."BUILDFAXPERMITINSIGHT" P									-- BuildFax Permit Table 
	JOIN "MI_XPRESSCLOUD"."XPRESSFEED"."SNLPROPOWNERSHIP" S           -- XF SNLpropOwnership
	ON S.propertyID=P.propertyID					                            -- JOIN on the propertyID
	WHERE 1=1
	AND (cast(permitDate AS DATE) >= ownershipStartDate or ownershipStartDate is NULL)	
                                                                     -- The property's permit was filed after the REIT owned it (assumes ownership if start is NULL)
	AND (cast(permitDate as DATE) <= ownershipEndDate or currentOwner = 1)				
                                                                     -- The property's permit was filed before the REIT stopped owning it (assumes ownership if end is NULL)
	AND Month(cast(PermitDate as date)) in (1,2,3,4,5,6,7)						 -- The permit was filed in the first 7 months of the year
	AND Year(cast(PermitDate as Date))=2020												     -- The permit was filed in 2020
	GROUP BY S.INSTITUTIONID			
	) PermitProp2020
	JOIN
	(
	-- Select the number of properties owned by the REIT between 1/1 and 7/31 in 2020
	SELECT
		INSTITUTIONID																	                   -- SNL ID (REIT)
		, count(distinct PROPERTYID) as nmanyProps2020									 -- Counts Distinct Properties Per REIT	
	from "MI_XPRESSCLOUD"."XPRESSFEED"."SNLPROPOWNERSHIP" S 
	where (ownershipStartDate < '07/31/2020' or ownershipStartDate is NULL)				
                                                                     -- Where the REIT Owned the Property before end of July 2020 (assumes ownership if start is NULL)
	and (ownershipEndDate > '01/01/2020' or currentOwner=1)						 -- And didn't stop owning the property before start of 2020 (assumes ownership if end if NULL. With previous condition this means property was owned during 2020)
	--and institutionID = 13485338 -- DEBUG
	group by S.INSTITUTIONID		
	) TotalProps2020
	ON PermitProp2020.INSTITUTIONID=TotalProps2020.INSTITUTIONID
) PRat2020
ON PRAT2019.INSTITUTIONID=PRAT2020.INSTITUTIONID

-- Goodbye!
```
