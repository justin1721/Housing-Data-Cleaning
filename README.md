# 🏠Housing Data Cleaning

----------------------------------------------------------------------------------------------------------------------------------

## Business Task
Explore Nashville Housing data and clean it to make the dataset useful for further analysis.

----------------------------------------------------------------------------------------------------------------------------------

## Cleaning Data in SQL Queries

````sql
SELECT *
FROM Portfolio_Project.dbo.Nashville_Housing
````

#### Steps:
- Show all columns and rows from Nashville_Housing dataset.

----------------------------------------------------------------------------------------------------------------------------------

## Standardize Data Format 

````sql
SELECT SaleDate
FROM Portfolio_Project.dbo.Nashville_Housing
SELECT SaleDateConverted, CONVERT(Date,SaleDate)
FROM Portfolio_Project.dbo.Nashville_Housing

UPDATE Nashville_Housing
SET SaleDate = CONVERT(Date,SaleDate)

ALTER TABLE Nashville_Housing
ADD SaleDateConverted Date;

UPDATE Nashville_Housing
SET SaleDateConverted = CONVERT(Date, SaleDate)
````

#### Steps:
- Display two date columns with the CONVERT function changing the format to YYYY-MM-DD format.
- Add the SaleDateConverted column to the table.
- Update the SaleDate and SaleDateConverted columns to the converted format.
  
----------------------------------------------------------------------------------------------------------------------------------

## Populate Property Address data 

````sql
SELECT *
FROM Portfolio_Project.dbo.Nashville_Housing
ORDER BY ParcelID

SELECT a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM Portfolio_Project.dbo.Nashville_Housing a
JOIN Portfolio_Project.dbo.Nashville_Housing b
	ON a.ParcelID = b.ParcelID
	AND a.[UniqueID] <> b.[UniqueID]
WHERE a.PropertyAddress IS NULL
	
UPDATE a
SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM Portfolio_Project.dbo.Nashville_Housing a
JOIN Portfolio_Project.dbo.Nashville_Housing b
	ON a.ParcelID = b.ParcelID
	AND a.[UniqueID] <> b.[UniqueID]
WHERE a.PropertyAddress IS NULL
````

#### Steps:
- Identify missing Propertyaddress information.
- Display dataset by ParcelID to view duplicate ParcelID and explore differences in other columns.
- Display rows where these conditions are met: PropertyAddress is Null, there is a duplicate in ParcelID, they have a unique UniqueID.
- Utilize ISNULL function to create a column where a.PropertyAddress is NULL. Fill it out with corresponding data from b.PropertyAddress.
- Update Nashville_Housing a using the alias (a).
  
----------------------------------------------------------------------------------------------------------------------------------

## Part 1: Breaking out Address into Individual Columns (Address, City, State) 

### Property Address

````sql
SELECT PropertyAddress
FROM Portfolio_Project.dbo.Nashville_Housing

SELECT 
SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1) as Address,
SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1, LEN(PropertyAddress)) as City
FROM Portfolio_Project.dbo.Nashville_Housing

ALTER TABLE Nashville_Housing
ADD PropertySplitAddress Nvarchar(255);

ALTER TABLE Nashville_Housing
ADD PropertySplitCity Nvarchar(255);

UPDATE Nashville_Housing
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1)

UPDATE Nashville_Housing
SET PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1, LEN(PropertyAddress))
````

#### Steps:
- Display PropertyAddress to visualize the current format.
- Utilize the SUBSTRING function to filter specific values in the PropertyAddress Column. SUBSTRING 1: Print the values in PropertyAddress up until the first comma (exclude the comma). SUBSTRING 2: Print all values after the first comma in PropertyAddress.
- Add PropertySplitAddress and PropertySplitCity columns in the table.
- Fill PropertySplitAddress and PropertySplitCity columns using the SUBSTRING functions.

----------------------------------------------------------------------------------------------------------------------------------

## Part 2: Breaking out Address into Individual Columns (Address, City, State) 

### Owner Address

````sql
SELECT OwnerAddress
FROM Portfolio_Project.dbo.Nashville_Housing

SELECT
PARSENAME(REPLACE(OwnerAddress, ',', '.'), 3),
PARSENAME(REPLACE(OwnerAddress, ',', '.'), 2),
PARSENAME(REPLACE(OwnerAddress, ',', '.'), 1)
FROM Portfolio_Project.dbo.Nashville_Housing

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
````

#### Steps:
- Display OwnerAddress to visualize the current format.
- Segment OwnerAddress by delimiter using PARSENAME (only reads periods '.'). Use the REPLACE function to change commas to periods.
- Add and Fill columns using PARSENAME function to OwnerSplit Address, City, and State.

----------------------------------------------------------------------------------------------------------------------------------

## Change Y and N values to Yes and No in the "Sold as Vacant" field 

````sql
SELECT DISTINCT(SoldAsVacant), COUNT(SoldAsVacant)
FROM Portfolio_Project.dbo.Nashville_Housing
GROUP BY SoldAsVacant
ORDER BY 2
 
SELECT SoldAsVacant,
CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
	WHEN SoldAsVacant = 'N' THEN 'No'
	ELSE SoldAsVacant
	END
FROM Portfolio_Project.dbo.Nashville_Housing

UPDATE Nashville_Housing
SET SoldAsVacant =  CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
	WHEN SoldAsVacant = 'N' THEN 'No'
	ELSE SoldAsVacant
	END
````

#### Steps:
- Display SoldAsVacant column with counts for each distinct value.
- Use the CASE statement to change 'Y' to 'Yes' and 'N' to 'No'.
-  Update SoldAsVacant column to 'Yes' and 'No' format.
-  
----------------------------------------------------------------------------------------------------------------------------------

## Remove Duplicates in the dataset 

````sql 
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
````

#### Steps:
- Use ROW_NUMBER() window function to assign a unique row number within each partition: (ParcelID, PropertyAddress, SalePrice, SaleDate, LegalReference).
- Select from CTE to identify duplicate values where row_num is > 2. DELETE where the row_num is > 2.
  
----------------------------------------------------------------------------------------------------------------------------------

## Delete Unused Columns 

````sql
ALTER TABLE Portfolio_Project.dbo.Nashville_Housing
DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress, SaleDate

SELECT *
FROM Portfolio_Project.dbo.Nashville_Housing
````

#### Steps:
- Remove OwnerAddress, TaxDistrict, and PropertyAddress.
- Display dataset to visualize all changes made.
