1. Create a DAX measure that calculates month-over-month sales growth using the Total Sales measure and Dim_Date date column.

            VAR CurrentSales = [Total Sales]
        VAR PreviousSales =
            CALCULATE(
                [Total Sales],
                DATEADD(Dim_Date[date], -1, MONTH)
            )
        RETURN
            DIVIDE(CurrentSales - PreviousSales, PreviousSales)

2. Write a DAX measure for cumulative total sales over the selected date range using the Total Sales measure and Dim_Date.

        CALCULATE(
            [Total Sales],
            FILTER(
                ALLSELECTED(Dim_Date),
                Dim_Date[date] <= MAX(Dim_Date[date])
            )
        )

3. Create a DAX RANKX measure that ranks products by Total Sales in descending order.


        
