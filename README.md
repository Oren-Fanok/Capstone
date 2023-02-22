# Capstone: Stormwater Flow Rate Model for the City of Missoula

# Section 1

In this section I read in our sewer SCADA data to a pandas dataframe and do some quick formatting. I'll begin by converting the dateandtime column to datetime format, removing unneeded columns, and dropping any NA values. Finally I will remove erronious 0's. This data contains 1's and 0's in the value column. 1's signal a pump turning on while a 0 signals a pump turning off. The relationship needs to be 1:1, however SCADA inputs erronious zero's into the output every 15 minutes, so I will remove them. Once this process is finished we are ready to begin section 2.

# Section 2

In this section I will create a new dataframe called calc_df, which will hold all of our data going forward. In section 2 I'll begin by filling the new dataframe with only time stamps containing pumps turning on or off (Val column 1 for on and 0 for off). From there I will convert both on and off timestamps into Unix time and calculate the difference in time between the pump turning on and off. Remove the # in front of any of the calc_df lines to see a preview of the dataframe.

# Section 3

In section 3 I will add elevation difference, volume per foot, observed pumping flow rate, average flow rate per pump, and average observed pumping flow rate (gpm).

Elevation Diference - This value can be changed for the desired lift station. Please note that each lift station will have a different elevation level. To change the elevation for a different lift station, simply change the 2.2 in the first cell below.

Observed pumping flow rate - To calculate this value we first need to input the wetwell's diameter, again this value will differ per each lift station. Simply change the diameter in the diam_well = (5) statement to the desired value. From there the vol_per_foot calculation will update and be added to the calc_df. Now we have all the information in the calc_df to calculate observed pumping flow rate. I'll take the elevation difference value times the volume per foot value, and divide the result by our time difference. The resulting value will be added to the calc_df in the observed pumping flow rate column.

Average flow rate per pump - Average flow rate per pump is calculated by adding the values in the observed pumping flow rate column for each individual pump, and then dividing that sum by the count of values for that pump. So if I iterate over 4 rows of the calc_df, and pump 1 has 3 out of the 4 rows with an observed pumping flow rate of 100, 120, and 100, the formula will sum these three rows (320) and divide by the amount of rows that were related to pump 1 (3). The average flow rate per pump 1 in row 4 would be 106.

Average observed pumping flow rate (gpm) - to calculate this value I simply add all the average flow rates for both pumps and divide the sum by the count of rows I have iterated over. For example if I have iterated over 4 rows, and 2 of the rows are pump 1 and 2 are pump 2, I'll add up the values from both pumps (100,110,100,120) and divide the sum by 4 to calculate the average observed pumping flow rate in GPM of 107.5 in row 4.

# Section 4

In this section I will calculate the time to fill, inflow, and average inflow. These values need to be accounted for when calculating flow rate, as even when the pumps are turned on within a wet well and the level is dropping, water is still flowing through the sewer pipes into the well.

Time to fill - To calculate time to fill for each row in calc_df I take the stop unix time of the first row within calc_df (when the pump turned off) and subtract this value from the start unix time off the next row (when the pump turned back on). The difference between the pump turning off and the next pump turning on is my time to fill measure.

Inflow - To calculate my inflow value for each row in calc_df, I take the row's volume per foot value and multiply by the row's elevation difference value. I then divide that result by the row's time to fill value we just calculated.

Average Inflow - To calculate my average inflow for each row in calc_df, I simply add all the values for inflow in the row's I have iterated over and divide by the count of those rows. So if I have iterated over 4 rows so far in calc_df and the inflow values are (60,40,20,60) I sum these values (180) and divide by four to get my average inflow value in row 4, 45.

# Section 5

In this section I will calculate the actual pump flow rate in GPM, average flow rate per pump in GPM, and average pump flow rate in GPM.

Actual pump flow rate - To calculate actual pump flow rate for each row in calc_df, I simply take the row's observed pumping flow rate value we calculated earlier, and add the row's inflow value.

Average flow rate per pump - To calculate the average flow rate per pump I add the values in the actual pump flow rate column for each individual pump, and then divide that sum by the count of values for that pump. So if I iterate over 4 rows of the calc_df, and pump 1 has 3 out of the 4 rows with an actual pump flow rate of 100, 120, and 100, the formula will sum these three rows (320) and divide by the amount of rows that were related to pump 1 (3). The average flow rate per pump 1 in row 4 would be 106.

Average pump flow rate - to calculate this value I simply add all the actual pump flow rates for both pumps and divide the sum by the count of rows I have iterated over. For example if I have iterated over 4 rows, and 2 of the rows are pump 1 and 2 are pump 2, I'll add up the values from both pumps (100,110,100,120) and divide the sum by 4 to calculate the average pump flow rate in GPM of 107.5.
