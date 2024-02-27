# Housing-Data-Cleaning

### Goal: Explore Nashville Housing data and clean to make the dataset useful for futher analysis


-- Cleaning Data in SQL Queries --


-- Show all columns and rows from Nashville_Housing dataset
SELECT *
FROM Portfolio_Project.dbo.Nashville_Housing

----------------------------------------------------------------------------------------------------------------------------------


-- Standardize Data Format --


-- Display two date columns with the CONVERT function changing the format to YYYY-MM-DD format
SELECT SaleDate
FROM Portfolio_Project.dbo.Nashville_Housing
SELECT SaleDateConverted, CONVERT(Date,SaleDate)
FROM Portfolio_Project.dbo.Nashville_Housing

-- Update the SaleDate column to the converted column
UPDATE Nashville_Housing
SET SaleDate = CONVERT(Date,SaleDate)

-- Add SaleDateConverted column to table
ALTER TABLE Nashville_Housing
ADD SaleDateConverted Date;

-- Update SaleDateConverted to YYYY-MM-DD format
UPDATE Nashville_Housing
SET SaleDateConverted = CONVERT(Date, SaleDate)

----------------------------------------------------------------------------------------------------------------------------------


-- Populate Property Address data --


-- Display dataset by ParcelID to view duplicate ParcelID and explore differences in other columns
SELECT *
FROM Portfolio_Project.dbo.Nashville_Housing
ORDER BY ParcelID

-- Identify missing Propertyaddress information
-- Display rows where these conditions are met: PropertyAddress is Null, there is a duplicate in ParcelID, they have a unique UniqueID
-- ISNULL creates a column where a.PropertyAddress is NULL and fills it with corresponding data from b.PropertyAddress 
SELECT a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM Portfolio_Project.dbo.Nashville_Housing a
JOIN Portfolio_Project.dbo.Nashville_Housing b
	ON a.ParcelID = b.ParcelID
	AND a.[UniqueID] <> b.[UniqueID]
WHERE a.PropertyAddress IS NULL
	

-- Update Nashville_Housing a (must use alias to update) 
UPDATE a
SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM Portfolio_Project.dbo.Nashville_Housing a
JOIN Portfolio_Project.dbo.Nashville_Housing b
	ON a.ParcelID = b.ParcelID
	AND a.[UniqueID] <> b.[UniqueID]
WHERE a.PropertyAddress IS NULL

----------------------------------------------------------------------------------------------------------------------------------


-- Breaking out Address into Individual Columns (Address, City, State) --


-- PROPERTY ADDRESS --

-- Display PropertyAddress to visiaulize current format
SELECT PropertyAddress
FROM Portfolio_Project.dbo.Nashville_Housing

-- SUBSTRING 1: Print the values in PropertyAddress up until the first comma (exclude the comma) 
-- SUBSTRING 2: Print all values after the first comma in PropertyAddress 
SELECT 
SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1) as Address,
SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1, LEN(PropertyAddress)) as City
FROM Portfolio_Project.dbo.Nashville_Housing

-- Add PropertySplitAddress column to table
ALTER TABLE Nashville_Housing
ADD PropertySplitAddress Nvarchar(255);

-- Fill PropertySplitAddress column with SUBSTRING 1 function
UPDATE Nashville_Housing
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1)

-- Add PropertySplitCity column to table
ALTER TABLE Nashville_Housing
ADD PropertySplitCity Nvarchar(255);

-- Fill PropertySplitCity column with SUBSTRING 2 function 
UPDATE Nashville_Housing
SET PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1, LEN(PropertyAddress))

-- Check added columns
SELECT *
FROM Portfolio_Project.dbo.Nashville_Housing


-- OWNER ADDRESS --

-- Display OwnerAddress
SELECT OwnerAddress
FROM Portfolio_Project.dbo.Nashville_Housing

-- Segment OwnerAddress by delimiter using PARSENAME (only reads periods '.')
-- Use REPLACE function to change commas to periods
SELECT
PARSENAME(REPLACE(OwnerAddress, ',', '.'), 3),
PARSENAME(REPLACE(OwnerAddress, ',', '.'), 2),
PARSENAME(REPLACE(OwnerAddress, ',', '.'), 1)
FROM Portfolio_Project.dbo.Nashville_Housing

-- Add and Fill columns using PARSENAME function to OwnerSplit Address, City, State
ALTER TABLE Nashville_Housing
ADD OwnerSplitAddress Nvarchar(255);

ALTER TABLE Nashville_Housing
ADD OwnerSplitCity Nvarchar(255);

ALTER TABLE Nashville_Housing
ADD OwnerSplitState Nvarchar(255);

UPDATE Nashville_Housing
SET OwnerSplitAddress = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 3)

UPDATE Nashville_Housing
SET OwnerSplitCity = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 2)

UPDATE Nashville_Housing
SET OwnerSplitState = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 1)

-- Check added columns
SELECT *
FROM Portfolio_Project.dbo.Nashville_Housing

----------------------------------------------------------------------------------------------------------------------------------


-- Change Y and N to Yes and No in "Sold as Vacant" field --


-- Display SoldAsVacant column with counts for each distinct value
SELECT DISTINCT(SoldAsVacant), COUNT(SoldAsVacant)
FROM Portfolio_Project.dbo.Nashville_Housing
GROUP BY SoldAsVacant
ORDER BY 2
 
-- Case statement to change 'Y' to 'Yes' and 'N' to 'No'
 SELECT SoldAsVacant,
 CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
	  WHEN SoldAsVacant = 'N' THEN 'No'
	  ELSE SoldAsVacant
	  END
 FROM Portfolio_Project.dbo.Nashville_Housing

 -- Update SoldAsVacant column to 'Yes' and 'No' format
 UPDATE Nashville_Housing
 SET SoldAsVacant =  CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
	  WHEN SoldAsVacant = 'N' THEN 'No'
	  ELSE SoldAsVacant
	  END

----------------------------------------------------------------------------------------------------------------------------------


-- Remove Duplicates --


-- Use ROW_NUMBER() window function to assign a unique row number within each partition: (ParcelID, PropertyAddress, SalePrice, SaleDate, LegalReference)
-- SELECT from CTE to identify duplicate values where row_num is > 2
-- DELETE where row_num is > 2
WITH RowNumCTE AS(
SELECT *,
	ROW_NUMBER() OVER (
	PARTITION BY ParcelID,
				 PropertyAddress,
				 SalePrice,
				 SaleDate,
				 LegalReference
				 ORDER BY
					UniqueID
					) row_num
				 

FROM Portfolio_Project.dbo.Nashville_Housing
-- ORDER BY ParcelID
)
-- DELETE
SELECT *
FROM RowNUMCTE
WHERE row_num > 1
ORDER BY PropertyAddress


SELECT *
FROM Portfolio_Project.dbo.Nashville_Housing

----------------------------------------------------------------------------------------------------------------------------------


-- Delete Unused Columns --

SELECT *
FROM Portfolio_Project.dbo.Nashville_Housing

-- Remove OwnerAddress, TaxDistrict, PropertyAddress
ALTER TABLE Portfolio_Project.dbo.Nashville_Housing
DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress, SaleDate
