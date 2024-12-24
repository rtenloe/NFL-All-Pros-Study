# NFL-All-Pros-Study
# [IN PROGRESS]
## Introduction
### What are we doing here?
The usage of data analytics in sports has increased exponentially in the past 15-20 years. I have always loved math, numbers, and solving problems or finding patterns. Sports is also something that I have always had a love for so those two areas mesh very well in sports analytics. I love baseball, basketball, auto racing, and football, but football is definitely my favorite sport to watch and study. In this particular study, I am analyzing the Associated Press (AP) All-Pro teams from the past 20 years to see if there are certain traits these players share that can be used going forward to find good players in free agency or the draft. On top of this, I am interested in how some of these traits could have possibly changed over time with the evolution of the game.
### Why the AP All-Pro teams?
In the NFL, there are two primary all-star teams that players can be selected for: the Pro Bowl team or the All-Pro team. The Pro Bowl is operated by the NFL and players are selected through a consensus of votes by players, coaches, and fans. Votes from each of these groups count one-third toward selecting players. There are two Pro-Bowl teams (one for each conference): the NFC team and the AFC team. The primary All-Pro team (split into first-team and second-team) is operated by the AP and is voted on my selected media members by the AP. Due to the fact that players, coaches, and even fans do not watch every player in the league and are especially susceptible to bias, the Pro Bowl teams are often considered less accurate than the all-pro teams. Additionally, players that were selected to the Pro Bowl, but make it to the Super Bowl have to have an alterante selected in their place. Also, a player can opt out of playing in the Pro Bowl and so alternates are selected for those players as well. This can lead to the number of players being selected to the Pro Bowl team to be quite large. Overall, being selected to an All-Pro is seen as a bigger honor than the Pro Bowl. There are several other organizations that select All-Pro teams (e.g. Football Writers of America (FWA), Pro Football Focus (PFF), The Sporting News (TSN), etc.), but the AP All-Pro team is widely considered to be the definitive all-pro team when a player's legacy is being considered. Therefore, I am only focusing on the AP All-Pro teams (1st and 2nd team) for this study.
## Data Sources
In order to complete this study, I need the list of players on the AP All-Pro teams from 2000-2023 as well as their statistics and physical measurements and testing numbers. Below is a list of where I obtained each piece of data:
|			Data		  	  |			    Source		   	   |
|-------------------------------------------------|--------------------------------------------------------|
| 	AP All-Pro team rosters (2000-2023) 	  |		Pro-Football-Reference.com      	   |
| 	 	AP All-Pro team stats 		  |		Pro-Football-Reference.com		   |
| Player physical measurements and testing scores |	Pro-Football-Reference.com & NFLCombineResults.com |
## Python Code
```
import csv
import glob
import os


def convert_to_inches(feet_inches):
    # Split the string by the dash to separate feet and inches
    try:
        feet, inches = feet_inches.split("-")
        feet = int(feet.strip())
        inches = int(inches.strip())
        total_inches = feet * 12 + inches  # Convert to total inches
        return total_inches
    except ValueError:
        # If there's an error in the format, return None or handle as needed
        print(f"Invalid format: {feet_inches}")
        return None

def convert_heights_in_csv(input_csv, output_csv):
    # Open the input CSV file
    with open(input_csv, mode='r', newline='') as infile:
        reader = csv.DictReader(infile)
        fieldnames = reader.fieldnames
        # Replace the 'Ht' field with heigh in inches
        fieldnames = [name for name in fieldnames if name != 'Ht']
        fieldnames.append('Ht')  # Overwrite the height column with the new format

        # Open the output CSV file
        with open(output_csv, mode='w', newline='') as outfile:
            writer = csv.DictWriter(outfile, fieldnames=fieldnames)
            writer.writeheader()
            
            # Iterate over each row in the input CSV
            for row in reader:
                height = row.get('Ht')
                if height:
                    # Convert the height to inches
                    height_in_inches = convert_to_inches(height)
                    if height_in_inches is not None:
                        row['Ht'] = height_in_inches
                # Write the row with the new height
                writer.writerow(row)

def process_multiple_csvs(directory):
    # Pattern to match the CSV files: '20?? Combine.csv'
    pattern = os.path.join(directory, '20?? Combine.csv')
    files = glob.glob(pattern)
    
    for file in files:
        # Get the year from the file name (e.g., '20?? Combine.csv')
        year = os.path.basename(file)[:4]  # Get '20??' from '20?? Combine.csv'
        output_file = os.path.join(directory, f"{year} Combine_converted.csv")
        
        print(f"Processing file: {file}")
        convert_heights_in_csv(file, output_file)
        print(f"Converted file saved as: {output_file}")

# Input CSV Directory
directory = '/Users/kierraenloe/Documents/All-Pros Project/CSV'
process_multiple_csvs(directory)
```
The combine data that I pulled from had the heights listed as "Feet-Inches". In order to get this into a format for SQL, I changed that to inches.
```
import csv
import glob
import os

def combine_csvs(directory, output_csv):
    # Pattern to match the converted CSV files: '20?? Combine_converted.csv'
    pattern = os.path.join(directory, '20?? Combine_converted.csv')
    files = glob.glob(pattern)
    
    # Open the output CSV file for writing
    with open(output_csv, mode='w', newline='') as outfile:
        writer = None
        
        # Iterate over each file to read and write its content
        for file in files:
            print(f"Processing file: {file}")
            
            with open(file, mode='r', newline='') as infile:
                reader = csv.DictReader(infile)
                
                # Write the header only once
                if writer is None:
                    writer = csv.DictWriter(outfile, fieldnames=reader.fieldnames)
                    writer.writeheader()
                
                # Write the rows from the current CSV file
                for row in reader:
                    writer.writerow(row)
    
    print(f"All files combined into: {output_csv}")

directory = '/Users/kierraenloe/Documents/All-Pros Project/CSV'  # Directory containing CSV files
output_csv = os.path.join(directory, 'combined_data.csv')  # The combined CSV file output path
combine_csvs(directory, output_csv)
```
Combines all the newly created CSVs to import into SQL.
```
import csv

def remove_invalid_rows(input_csv, output_csv, column_name='Player-additional', invalid_value=-9999):
    # Open the input CSV file
    with open(input_csv, mode='r', newline='') as infile:
        reader = csv.DictReader(infile)
        
        # Open the output CSV file for writing
        with open(output_csv, mode='w', newline='') as outfile:
            writer = csv.DictWriter(outfile, fieldnames=reader.fieldnames)
            writer.writeheader()
            
            # Iterate over each row in the input CSV
            for row in reader:
                try:
                    # Check if the value in the 'player-additional' column is -9999
                    if int(row[column_name]) != invalid_value:
                        writer.writerow(row)
                except ValueError:
                    # In case of an invalid value (not an integer), keep the row (or handle as needed)
                    writer.writerow(row)

    print(f"Filtered file saved as: {output_csv}")

input_csv = '/Users/kierraenloe/Documents/All-Pros Project/CSV/combined_data.csv'  # Combined CSV file path
output_csv = '/Users/kierraenloe/Documents/All-Pros Project/CSV/combined_data_filtered.csv' # The output filtered CSV file path
remove_invalid_rows(input_csv, output_csv)
```
Removes all players that have bad player ids.
## SQL Queries
```
CREATE TABLE AP2000 (
	pos varchar(255),
	player varchar(255),
	team varchar(255),
	age int,
	years_exp int,
	games_played int,
	games_started int,
	pass_comps int,
	pass_atts int,
	pass_yds int,
	Pass_TDs int,
	Off_Ints int,
	Rush_Att int,
	Rush_Yds int,
	Rush_TDs int,
	Receptions int,
	Receiving_Yds int,
	Receiving_TDs int,
	Solo_Tackles int,
	Sacks float,
	Def_Ints int,
	All_Pro_Team varchar(255),
	player_id varchar(255),
	PRIMARY KEY (player_id)
);
```
Creates tables to store all the data for all-pro players for each season since the year 2000. This query is repeated for each year (2000 season shown as example).
```
CREATE TABLE combined AS
SELECT * FROM ap2000
UNION ALL
SELECT * FROM ap2001
UNION ALL
SELECT * FROM ap2002
UNION ALL
SELECT * FROM ap2003
UNION ALL
SELECT * FROM ap2004
UNION ALL
SELECT * FROM ap2005
UNION ALL
SELECT * FROM ap2006
UNION ALL
SELECT * FROM ap2007
UNION ALL
SELECT * FROM ap2008
UNION ALL
SELECT * FROM ap2009
UNION ALL
SELECT * FROM ap2010
UNION ALL
SELECT * FROM ap2011
UNION ALL
SELECT * FROM ap2012
UNION ALL
SELECT * FROM ap2013
UNION ALL
SELECT * FROM ap2014
UNION ALL
SELECT * FROM ap2015
UNION ALL
SELECT * FROM ap2016
UNION ALL
SELECT * FROM ap2017
UNION ALL
SELECT * FROM ap2018
UNION ALL
SELECT * FROM ap2019
UNION ALL
SELECT * FROM ap2020
UNION ALL
SELECT * FROM ap2021
UNION ALL
SELECT * FROM ap2022
UNION ALL
SELECT * FROM ap2023
```
Creates a table of all the seasons combined.
```
CREATE TABLE ap_combined AS
SELECT player_id, player, pos, team, age, years_exp, games_played, games_started, pass_comps, pass_atts, pass_yds, pass_tds, off_ints, rush_att, rush_yds, rush_tds, receptions, receiving_yds, receiving_tds, COALESCE(solo_tackles, 0) AS solo_tackles, sacks, def_ints, all_pro_team
FROM combined
WHERE all_pro_team LIKE '%AP%'
ORDER BY player ASC
```
Creates a table where only the AP all-pro teams are considered.
