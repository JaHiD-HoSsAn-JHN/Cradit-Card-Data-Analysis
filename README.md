# Cradit Card Transaction and Customer Dynamic & Interactive Dashboard Using PowerBI And data input from PostgreSQL Database


# Dax functions that I have applied in This Project:
# 01:
Calendar = ADDCOLUMNS(
    CALENDAR(MIN('craditcard card_data'[Date]), MAX('craditcard card_data'[Date])), 
    "Year", YEAR([Date]),
    "Month", FORMAT([Date], "MMM"),
    "MonthNum", MONTH([Date]),
    "Day", DAY([Date]),
    "Weekday", FORMAT([Date], "ddd"),
    "WeekOfMonth", 
        INT((DAY([Date]) - 1) / 7) + 1,  -- Integer week number within the month
    "Week", WEEKNUM([Date], 1),  -- Assuming you want to keep the original week number within the year
    "Qtr", "Q" & QUARTER([Date])
)
# 02:
Age_group = SWITCH(
    TRUE(),
    'craditcard customer_data'[customer_age]< 30, "20-30",
    'craditcard customer_data'[customer_age]>= 30 && 'craditcard customer_data'[customer_age]< 40,"30-40",
    'craditcard customer_data'[customer_age] >= 40 && 'craditcard customer_data'[customer_age]<50, "40_50",
    'craditcard customer_data'[customer_age] >= 50 &&'craditcard customer_data'[customer_age]<60, "50-60",
    'craditcard customer_data'[customer_age]>=60, "60+",
    "Unknown"
    )
# 03:
Income_Group = SWITCH(
    TRUE(),
    'craditcard customer_data'[income] >= 50000 && 'craditcard customer_data'[income] <= 80000, "High income",
    'craditcard customer_data'[income] >= 25000 && 'craditcard customer_data'[income] < 50000, "Medium income",
    'craditcard customer_data'[income] < 25000, "Low income",
    "Unknown"
)
# 04:
% = DIVIDE(([Current_Week]-[previous_week_]),[previous_week_])
05:
% Current_week vs Previous_week = 
DIVIDE([Current_Week], [previous_week_], 0)
# 06:
Current_Week = CALCULATE(
    SUM('craditcard card_data'[Revenue]), 
    FILTER(
        ALL('Calendar'), 
        'Calendar'[Week] = MAX('Calendar'[Week])
    )
)
# 07:
previous_week_ = CALCULATE(
    SUM('craditcard card_data'[Revenue]), 
    FILTER(
        ALL('Calendar'), 
        'Calendar'[Week] = MAX('Calendar'[Week]) - 1
    )
)
# 08:
Filters = 
VAR marital_status_top = RANKX(ALL('craditcard customer_data'[marital_status]), [Total_Revenue])  // Defaults to descending order
VAR marital_status_bottom = RANKX(ALL('craditcard customer_data'[marital_status]), [Total_Revenue], , ASC)

VAR customer_job_top = RANKX(ALL('craditcard customer_data'[customer_job]), [Total_Revenue])  // Defaults to descending order
VAR customer_job_bottom = RANKX(ALL('craditcard customer_data'[customer_job]), [Total_Revenue], , ASC)

VAR state_top = RANKX(ALL('craditcard customer_data'[state_cd]), [Total_Revenue])  // Defaults to descending order
VAR state_bottom = RANKX(ALL('craditcard customer_data'[state_cd]), [Total_Revenue], , ASC)

VAR income_group_top = RANKX(ALL('craditcard customer_data'[Income_Group]), [Total_Revenue])  // Defaults to descending order
VAR income_group_bottom = RANKX(ALL('craditcard customer_data'[Income_Group]), [Total_Revenue], , ASC)

VAR education_top = RANKX(ALL('craditcard customer_data'[education_level]), [Total_Revenue])  // Defaults to descending order
VAR education_bottom = RANKX(ALL('craditcard customer_data'[education_level]), [Total_Revenue], , ASC)

VAR _CheckRank = 
    IF(
        CONTAINSSTRING(SELECTEDVALUE(Selectedfield[Selectedfield Fields]), "marital_status"),
        IF(SELECTEDVALUE('Select'[Select]) = "Top", marital_status_top, marital_status_bottom),
        IF(
            CONTAINSSTRING(SELECTEDVALUE(Selectedfield[Selectedfield Fields]), "customer_job"),
            IF(SELECTEDVALUE('Select'[Select]) = "Top", customer_job_top, customer_job_bottom),
            IF(
                CONTAINSSTRING(SELECTEDVALUE(SelectField[SelectField Fields]), "state"),
                IF(SELECTEDVALUE('Select'[Select]) = "Top", state_top, state_bottom),
                    IF(
                        CONTAINSSTRING(SELECTEDVALUE(Selectedfield[Selectedfield Fields]), "income_group"),
                        IF(SELECTEDVALUE('Select'[Select]) = "Top", income_group_top, income_group_bottom),
                        IF(
                            CONTAINSSTRING(SELECTEDVALUE(Selectedfield[Selectedfield Fields]), "education"),
                            IF(SELECTEDVALUE('Select'[Select]) = "Top", education_top, education_bottom),
                            BLANK()  // Default case
                        )
                    )
                
            )
        )
    )

RETURN
IF(
    _CheckRank <= 'Choose Rank'[Choose Rank Value], 
    [Total_Revenue],  // Return Total Revenue if within the rank
    BLANK()  // Hide row if not within rank
)
# 09:

Ranking = VAR _top_expend = RANKX(ALL('craditcard card_data'[expenditure_type]), [Total_Revenue])  // Defaults to descending order
VAR _bottom_expend = RANKX(ALL('craditcard card_data'[expenditure_type]), [Total_Revenue], , ASC)

VAR _top_customer_job = RANKX(ALL('craditcard customer_data'[customer_job]), [Total_Revenue])  // Defaults to descending order
VAR _bottom_customer_job = RANKX(ALL('craditcard customer_data'[customer_job]), [Total_Revenue], , ASC)

VAR _top_Education = RANKX(ALL('craditcard customer_data'[education_level]), [Total_Revenue])  // Defaults to descending order
VAR _bottom_Education = RANKX(ALL('craditcard customer_data'[education_level]), [Total_Revenue], , ASC)

VAR _CheckRank = 
    IF(
        CONTAINSSTRING(SELECTEDVALUE(SelectField[SelectField Fields]), "expenditure_type"),
        IF(SELECTEDVALUE('Select'[Select]) = "Top", _top_expend , _bottom_expend),
        IF(
            CONTAINSSTRING(SELECTEDVALUE(SelectField[SelectField Fields]), "customer_job"),
            IF(SELECTEDVALUE('Select'[Select]) = "Top", _top_customer_job, _bottom_customer_job),
            IF(SELECTEDVALUE('Select'[Select]) = "Top", _top_Education, _bottom_Education)
        )
    )

RETURN
IF(
    _CheckRank <= 'Choose Rank'[Choose Rank Value], [Total_Revenue],  // Return Total revenue if within rank
    BLANK()  // Hide row if not within rank
)
# 10:
Selectedfield = {
    ("customer_job", NAMEOF('craditcard customer_data'[customer_job]), 0),
    ("dependent_count", NAMEOF('craditcard customer_data'[dependent_count]), 1),
    ("education_level", NAMEOF('craditcard customer_data'[education_level]), 2),
    ("Income_Group", NAMEOF('craditcard customer_data'[Income_Group]), 3),
    ("marital_status", NAMEOF('craditcard customer_data'[marital_status]), 4)
}
# 11:
SelectField = {
    ("exp_type", NAMEOF('craditcard card_data'[expenditure_type]), 0),
    ("customer_job", NAMEOF('craditcard customer_data'[customer_job]), 1),
    ("education_level", NAMEOF('craditcard customer_data'[education_level]), 2)
}
# 12:
total interst_earn = SUM('craditcard card_data'[interest_earned])
# 13:
total transaction = SUM('craditcard card_data'[total_trans_amt])
# 14:
Total_Revenue = SUM('craditcard card_data'[Revenue])
# 15:
Weekly Revenue = 
FORMAT([Current_Week] - [previous_week_], "$#,##0;-$#,##0")
