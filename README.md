![image](https://user-images.githubusercontent.com/112598531/220499284-95bbe4cb-8a46-4f4e-af7a-5f9231087459.png)


# Capstone: Stormwater Flow Rate Model for the City of Missoula

# Introduction

The City of Missoula is located in the Northwestern corner of Montana. It's admiringly called the garden city, due to its milder climate and the abundance of water in the valley. The Clarks Fork River and the Bitterroot River join the city in the valley, while 5 different mountain ranges surround the city from above. As a result of these favorable traits the city has seen a large increase in population, and to support this growth, development has increased as well. As a result the city's infrastructure now has a larger burden.

The focus of this project will be on the City of Missoula's sewer infrastructure. The city has a large array of sewer pipes that traverse the city toward treatment plants. These pipes are built in descending lines, to allow gravity to move the water toward these treatment plants. However, there are places in these sewer lines where it's no longer economic to dig any deeper to continue the descending grade. Here is where the city's lift stations step in. Lift stations serve as an intermediary, lifting water up from the end of sewer pipes that can go no deeper to a fresh pipe much nearer the surface. The new pipe closer to the surface begins the descending grade again for gravity to step back in.

There are a multitude of lift stations throughout the valley but for this project, I will focus on 20 stations that are outfitted with SCADA. SCADA is a data collection service that provides real-time data from each lift station such as when the wet well's pumps are turned on and off along with a timestamp for each event. Using this data I built a model to calculate the Flow Rate in Gallons Per Minute from each of these lift stations at any point in time. This flow rate can then be used for diurnal modeling and visualizations, which provides invaluable impact when assessing the city's sewer system limits.

In the links above you will find the model, the function, and the sample output. The model will show each step in this process individually from reading in the SCADA data to calculating inflow, and finally finding the Average Flow Rate in GPM. The function combines all of these steps into an easy 5-step program that allows the user to convert raw SCADA data into a meaningful calculated excel sheet with just a few clicks. The sample data is a sample of the output from the function for a lift station.

# Section 1

In this section, I read my Lift Station SCADA data and Lift Station Legend data to a pandas data frames and do some quick formatting. I'll begin by converting the date and time column to DateTime format, dropping unneeded columns, and dropping any rows that are missing pump start or stop timestamps. Once this process is finished we are ready to begin section 2.

# Section 2

In this section, I will create a new data frame called calc_df, which will hold all of our calculated data going forward. I'll begin by assigning the original df to my calculated df, and reformatting the columns (names and order). Next I will remove any erroneous rows containing invalid pump numbers. Then I'll use the legend dataframe I read into the program earlier to add the correct lift station name to each row in calc_df. Finally, I will convert both on and off timestamps into Unix time and calculate the difference in time between the pump turning on and off.

# Section 3

In this section, I will add elevation difference, volume per foot, observed pumping flow rate, the average flow rate per pump, and average observed pumping flow rate (GPM).

Elevation Difference - This value is unique to each lift station. Using the information in the legend data frame I'll assign each row an elevation difference based on the lift station.

Wet well diameter - This value is unique to each lift station. Using the information in the legend data frame I'll assign each row a wet well diameter based on the lift station.

Volume per foot - I'll calculate the volume per foot of each lift station by using this calculation, (wet_well_diam^2 * 3.14) / 4 * 7.48.

Observed pumping flow rate - To calculate this value we first need to input the wet wells diameter, again this value will differ per each lift station. I'll take the row's elevation difference value times the row's volume per foot value, and divide the result by the row's time difference. The resulting value will be added to the row in the observed pumping flow rate column.

Average flow rate per pump - Average flow rate per pump is calculated by adding the values in the observed pumping flow rate column for each pump # and then dividing that sum by the count of values for that pump. So if I iterate over 4 rows of the calc_df, and pump 1 has 3 out of the 4 rows with an observed pumping flow rate of 100, 120, and 100, the formula will sum these three rows (320) and divide by the number of rows that were related to pump 1 (3). The average flow rate per pump 1 in row 4 would be 106.

Average observed pumping flow rate (GPM) - to calculate this value I simply add all the average flow rates for all pumps per lift station and divide the sum by the count of rows I have iterated over. For example, if I have iterated over 4 rows, and 2 of the rows are pump 1 and 2 are pump 2, I'll add up the values from both pumps (100,110,100,120) and divide the sum by 4 to calculate the average observed pumping flow rate in GPM of 107.5 in row 4.

# Section 4

In this section, I will calculate the time to fill, inflow, and average inflow. These values need to be accounted for when calculating flow rate, as even when the pumps are turned on within a wet well and the level is dropping, water is still flowing through the sewer pipes into the well.

Time to fill - To calculate the time to fill for each row in calc_df, I take the stop Unix time of the first row within calc_df (when the pump turned off) and subtract this value from the start Unix time of the next row (when the pump turned back on). The difference between the pump turning off and the next pump turning on is my time to fill the measure.

Inflow - To calculate my inflow value for each row in calc_df, I take the row's volume per foot value and multiply it by the row's elevation difference value. I then divide that result by the row's time to fill value we just calculated.

Average Inflow - To calculate my average inflow for each row in calc_df, I simply add all the values for inflow in the rows I have iterated over and divide by the count of those rows. So if I have iterated over 4 rows so far in calc_df and the inflow values are (60,40,20,60) I sum these values (180) and divide by four to get my average inflow value in row 4, 45.

# Section 5

In this section, I will calculate the actual pump flow rate in GPM, the average flow rate per pump in GPM, and the average pump flow rate in GPM.

Actual pump flow rate - To calculate the actual pump flow rate for each row in calc_df, I simply take the row's observed pumping flow rate value we calculated earlier, and add the row's inflow value.

Average flow rate per pump - To calculate the average flow rate per pump for each lift station, I add the values in the actual pump flow rate column for each pump, and then divide that sum by the count of values for that pump. So if I iterate over 4 rows of the calc_df, and pump 1 has 3 out of the 4 rows with an actual pump flow rate of 100, 120, and 100, the formula will sum these three rows (320) and divide by the number of rows that were related to pump 1 (3). The average flow rate per pump 1 in row 4 would be 106.

Average pump flow rate - to calculate this value I simply add all the actual pump flow rates for all pumps within an individual lift station and divide the sum by the count of rows I have iterated over. For example, if I have iterated over 4 rows, and 2 of the rows are pump 1 and 2 are pump 2, I'll add up the values from both pumps (100,110,100,120) and divide the sum by 4 to calculate the average pump flow rate in GPM of 107.5.

# Creating a dashboard to visualize the model via PowerBI

<img width="1029" alt="image" src="https://user-images.githubusercontent.com/112598531/224856405-9f03e0d6-ae7a-42b5-8eca-c1053cd396e7.png">



