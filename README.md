# Aquarium-Species-Data-Analysis-Project
Data Analysis Case Project on aquarium fish species available in the fishkeeping hobby.

## Project Overview
Collected, organised and analyzed data on aquarium fish species available in the fishkeeping hobby. Result was a dashboard allowing users to filter out species based on their budget, experience, available aquarium tank size, desired habitat type and desired origin (Distribution). Dashboard also showcases photos of the selected fish species (where licensed photos were available).

## Link to the dashboard: 
https://public.tableau.com/views/AquariumSpeciesandSetupHelper/AquariumSpeciesandSetupHelper?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link

## Project Goals:
- Make it easier for beginner hobbyists to decide on aquisitions and plan for their aquarium stocking and design.
- Allow experienced aquarists to decide on stocking options when designing "biotope" aquariums.
- Provide an easy to use resource for aquarists when reviewing available species and varieties.
- Promote "themed" and "biotope" aquarium setups by making them easier to plan.
- Provide online stores with a template for better tailororing inventory and recommendations to customers.
- Promote informed purchasing decisions among costumers when purchasing pets.

## Datas Description:
- Collected and reviewed data on ~300 species with regards to: Distribution, Natural Habitat, Aquarium Size Requirements, Aquarium Parameters Requirements, Aquarium Setup Preferences, Overall Difficulty to Care for, Dietary Requirements, Same-Species Cohabitation, Overall Behaviour in Aquarium.
- See below list of websites used for gathering information:
- SeriouslyFish.com
- https://www.fishi-pedia.com
- https://aquariumbreeder.com
- https://aquadiction.world
- https://www.fishbase.se
- https://dwarfcichlid.com
- https://www.planetcatfish.com
- https://www.tropicalfishsite.com
- https://en.aqua-fish.net

Images collected from the following websites with credit attribution provided in the dashboard popups:
- https://commons.wikimedia.org/wiki/Main_Page
- https://www.inaturalist.org/
- https://www.fishbase.se/search.php

## Tools Used:
- Google Sheets (data cleaning, data validation, junction tables)
- bigQuery (data restructuring, JOINS)
- Tableau (dashboard creation)
- Google Slides (presentation)

## Files Included:
- "Fish Data - Webshop - fish_webstore.csv" - Data collected on webstore available fish species and variety together with prices in July 2025.
- "Fish Data - Details - region_breakdown" - Small companion table to assist with region categorization during the analysis.
- "Fish Data - Details - fish_species" - Table with main data collected on species.
- "Fish Data - Details - Google Docs" - Table with main data collected on species as worked on using Google Suit.
- "Fish Data - Details - habitat_junction" - Junction table created out of the "Habitat" column of the fish_species table for multi-value cell transformation.
- "Fish Data - Details - compatibility_junction" - Junction table created out of the "Compatibility" column of the fish_species table for multi-value cell transformation.
"Export Fish Project - Final" - Table exported after querying in bigQuery and utilized for the dashboard.
"Aquarium Setups - Presentation" - Overall presentation of the project.

## Key Transformations:
Created junction tables out of the multi-value cell columns (Habitat and Compatibility) using the following formula in Google Sheets: "=TRANSPOSE(SPLIT(REPT(A2 & "♦", COUNTA(SPLIT(D2, ", "))) & SPLIT(D2, ", "), "♦"))"

## Final SQL query used to create the table used for the dashboard:
```sql

SELECT
  fs.`Scientific Name`,
  ws.Classification,
  ws.`Lowest Price RON`,
  ws.`Highest Price RON`,
  fs.`Water Level Area`,
  fs.`Temp C Low `,
  fs.`Temp C High `,
  fs.`Hardness dH Low`,
  fs.`Hardness dH High`,
  fs.`pH Low`,
  fs.`pH High`,
  CONCAT(fs.`Common Name`, ' (', ws.Variety, ')') AS `Display Name`,

  -- Budget classification based on grouping and lowest price
  CASE
    WHEN fs.Grouping IN ('Loner', 'Pair') AND ws.`Lowest Price RON` < 30 THEN 'Low Budget'
    WHEN fs.Grouping IN ('Loner', 'Pair') AND ws.`Lowest Price RON` >= 30 THEN 'Medium Budget'
    WHEN fs.Grouping = 'Small Groups' AND ws.`Lowest Price RON` < 20 THEN 'Low Budget'
    WHEN fs.Grouping = 'Small Groups' AND ws.`Lowest Price RON` >= 20 AND ws.`Lowest Price RON` < 35 THEN 'Medium Budget'
    WHEN fs.Grouping = 'Small Groups' AND ws.`Lowest Price RON` >= 35 THEN 'High Budget'
    WHEN fs.Grouping IN ('Groups', 'Shoaling', 'Schooling') AND ws.`Lowest Price RON` < 10 THEN 'Low Budget'
    WHEN fs.Grouping IN ('Groups', 'Shoaling', 'Schooling') AND ws.`Lowest Price RON` >= 10 AND ws.`Lowest Price RON` < 15 THEN 'Medium Budget'
    WHEN fs.Grouping IN ('Groups', 'Shoaling', 'Schooling') AND ws.`Lowest Price RON` >= 15 THEN 'High Budget'
    ELSE 'High Budget'
  END AS `Budget Category`,

  -- Habitat type classification
  CASE
    WHEN hj.Habitat = 'Brackish' THEN 'Brackish Environment'
    WHEN hj.Habitat IN ('Rivers', 'Streams', 'Canals') AND fs.`Aquarium Setup` = 'Flowing River' THEN 'Fast Flowing River'
    WHEN hj.Habitat = 'Rivers' AND fs.`Aquarium Setup` = 'Dedicated' THEN 'Fast Flowing River'
    WHEN hj.Habitat IN ('Rivers', 'Streams', 'Canals') THEN 'Flowing River'
    WHEN hj.Habitat IN ('Lakes', 'Ponds', 'Pools') THEN 'Still Water'
    WHEN hj.Habitat IN ('Swamp', 'Flooded Areas') THEN 'Flooded Areas'
    ELSE 'Other Environments'
  END AS `Habitat Type`,

  -- Experience level classification
  CASE
    WHEN fs.Difficulty IN ('Very Easy', 'Easy', 'Moderately Easy') THEN 'Beginner'
    WHEN fs.Difficulty = 'Intermediate' AND (fs.Diet = 'Unfussy' OR fs.`Aquarium Setup` = 'Unfussy') THEN 'Beginner'
    WHEN fs.Difficulty = 'Intermediate' AND (fs.Diet <> 'Unfussy' OR fs.`Aquarium Setup` <> 'Unfussy') THEN 'Intermediate'
    WHEN fs.Difficulty IN ('Difficult', 'Moderately Difficult') THEN 'Advanced'
    ELSE fs.Difficulty
  END AS `Experience Level`,

  -- Tank size classification
  CASE
    WHEN fs.`Minimum Aquarium Size in Litres` < 60 THEN 'Small'
    WHEN fs.`Minimum Aquarium Size in Litres` BETWEEN 60 AND 120 THEN 'Medium'
    WHEN fs.`Minimum Aquarium Size in Litres` > 120 THEN 'Large'
    ELSE 'Unknown'
  END AS `Tank Size Category`,

  fs.Distribution,
  rm.MacroRegion

FROM `aquarium-project-466714.aquarium_data.fish_species` AS fs
JOIN `aquarium-project-466714.aquarium_data.fish webstore` AS ws 
  ON fs.`Scientific Name` = ws.`Scientific Name`
LEFT JOIN `aquarium-project-466714.aquarium_data.habitat_junction` AS hj 
  ON fs.`Scientific Name` = hj.`Scientific Name`
LEFT JOIN `aquarium-project-466714.aquarium_data.region_mapping` AS rm 
  ON fs.Distribution = rm.Area


