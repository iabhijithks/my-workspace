# Data Visualization Lab Exam -- Complete Solutions

## All three question documents \| R, ggplot2 and Power BI

**Exam-use note:** These are simple, exam-oriented solutions using the
patterns practised: `data.frame()`, `read_excel()`, `dplyr`, `ggplot()`,
`geom_*()`, DAX measures and standard Power BI visuals.

**Before starting** - In RStudio, run `install.packages("ggplot2")`,
`install.packages("dplyr")`, `install.packages("readxl")`, and where
required `install.packages("reshape2")` once. - Load packages in each
script with `library(ggplot2)`, `library(dplyr)`, `library(readxl)`. -
Keep `Sample - Superstore.xlsx` in the working directory, or replace the
filename with its full path. - In Power BI, the imported table may be
named `Orders`, `Sheet1`, or something else. Replace `Orders` in DAX
with the actual table name shown in the Fields pane. - Check data types:
Order Date = Date; Sales, Profit, Discount and Quantity = numeric; City,
State, Region, Category and Sub-Category = text.

------------------------------------------------------------------------

# DOCUMENT 1 -- Internal Lab Exam

## Q1(A). Create a pie chart and bar plot showing sales/record count by Region using Sample Superstore in R

The wording says "sales count"; count the number of records per region.
If the examiner means total sales value, use `sum(Sales)` instead of
`n()`.

``` r
library(readxl)
library(dplyr)
library(ggplot2)

sales <- read_excel("Sample - Superstore.xlsx")

region_count <- sales %>%
  count(Region, name="SalesCount")

# Pie chart
ggplot(region_count, aes(x="", y=SalesCount, fill=Region)) +
  geom_col(width=1, colour="white") +
  coord_polar("y") +
  labs(title="Record Count by Region", fill="Region") +
  theme_void()

# Bar chart
ggplot(region_count, aes(x=Region, y=SalesCount, fill=Region)) +
  geom_col() +
  geom_text(aes(label=SalesCount), vjust=-0.3) +
  labs(title="Record Count by Region",
       x="Region", y="Number of Records") +
  theme_minimal() +
  theme(legend.position="none")
```

**Remember:** `count(Region)` counts rows. `geom_col()` plots the counts
already calculated.

## Q1(B). Line chart of Sales over time by Region and scatter plot of Quantity vs Profit coloured by Category

``` r
library(readxl)
library(ggplot2)
library(dplyr)

sales <- read_excel("Sample - Superstore.xlsx")
sales$`Order Date` <- as.Date(sales$`Order Date`)

# Sales over time by Region
ggplot(sales, aes(x=`Order Date`, y=Sales, colour=Region)) +
  geom_line(stat="summary", fun=sum) +
  labs(title="Sales Trend by Region",
       x="Order Date", y="Total Sales",
       colour="Region") +
  theme_minimal()

# Quantity vs Profit scatter plot
ggplot(sales, aes(x=Quantity, y=Profit, colour=Category)) +
  geom_point(alpha=0.65) +
  labs(title="Quantity vs Profit by Category",
       x="Quantity", y="Profit", colour="Category") +
  theme_minimal()
```

`geom_line(stat="summary", fun=sum)` adds the sales totals at each
date/region grouping. `geom_point()` displays individual records.

## Q2(A). Find maximum Sales for Furniture and Technology in each Region

``` r
library(readxl)
library(dplyr)
library(ggplot2)

sales <- read_excel("Sample - Superstore.xlsx")

max_sales <- sales %>%
  filter(Category %in% c("Furniture", "Technology")) %>%
  group_by(Region, Category) %>%
  summarise(MaxSales=max(Sales, na.rm=TRUE), .groups="drop")

print(max_sales)

ggplot(max_sales, aes(x=Region, y=MaxSales, fill=Category)) +
  geom_col(position="dodge") +
  labs(title="Maximum Sales by Region and Category",
       x="Region", y="Maximum Sales") +
  theme_minimal()
```

`filter()` keeps the two requested categories; `group_by()` creates
Region--Category groups; `max()` finds the largest sale in each group.

## Q2(B). Power BI dashboard with control-style charts, scatter plot, stacked bars and trend lines

1.  Open Power BI Desktop → **Home → Get data → Excel** → select the
    Superstore workbook → **Load**.
2.  Check column data types in Data view.
3.  Add a **Line chart**: X-axis = Order Date; Y-axis = Sales (Sum).
4.  Add a **Scatter chart**: X-axis = Quantity; Y-axis = Profit; Details
    = Order ID; Legend = Category.
5.  Add a **Stacked bar chart**: Y-axis/Axis = Region; Legend =
    Category; X-axis/Values = Sales (Sum).
6.  Select the scatter chart. Open the **Analytics** pane and add a
    Trend line if the installed Power BI version offers it.
7.  For a simple control-style chart, use a line chart and add
    average/upper/lower constant lines from Analytics where available. A
    true statistical control chart may require a custom visual.
8.  Add slicers for Region, Category and Order Date.
9.  Arrange and format the visuals; add a descriptive title and save the
    `.pbix` file.

## Q3(A). Create two diamond bar graphs: mean Price by Cut and minimum Price by Clarity

The `diamonds` dataset is included with ggplot2.

``` r
library(ggplot2)
library(dplyr)

# Mean price by cut
mean_cut <- diamonds %>%
  group_by(cut) %>%
  summarise(MeanPrice=mean(price, na.rm=TRUE))

ggplot(mean_cut, aes(x=cut, y=MeanPrice, fill=cut)) +
  geom_col() +
  labs(title="Mean Diamond Price by Cut",
       x="Cut", y="Mean Price") +
  theme_minimal()

# Minimum price by clarity
min_clarity <- diamonds %>%
  group_by(clarity) %>%
  summarise(MinPrice=min(price, na.rm=TRUE))

ggplot(min_clarity, aes(x=clarity, y=MinPrice, fill=clarity)) +
  geom_col() +
  labs(title="Minimum Diamond Price by Clarity",
       x="Clarity", y="Minimum Price") +
  theme_minimal()
```

## Q3(B). Power BI dashboard: mean, median, normal distribution, variation and uncertainty

1.  Load the Superstore Excel file and verify numeric data types.
2.  Choose **Modeling → New measure** and create:

``` dax
Mean Sales = AVERAGE(Orders[Sales])
Median Sales = MEDIAN(Orders[Sales])
Sales SD = STDEV.P(Orders[Sales])

Mean Profit = AVERAGE(Orders[Profit])
Median Profit = MEDIAN(Orders[Profit])
Profit SD = STDEV.P(Orders[Profit])
```

3.  Add Card visuals for the six measures.
4.  Create a histogram for Sales: create bins for Sales (right-click
    Sales in Fields → New group/Bins, if available), then use Sales bins
    on the X-axis and count of Sales on the Y-axis.
5.  Use a box plot custom visual if permitted, or compare standard
    deviation by Category in a column chart.
6.  Add slicers for Region, Category and Order Date.
7.  Interpret the histogram: a bell-shaped and approximately symmetric
    pattern may suggest normality; skewness or outliers suggest
    otherwise. Do not claim data is normally distributed without
    inspecting the chart.
8.  Explain variation using standard deviation and uncertainty as the
    spread/variability of values.

## Q4(A). Histogram using the Iris dataset in ggplot2

``` r
library(ggplot2)

ggplot(iris, aes(x=Sepal.Length)) +
  geom_histogram(binwidth=0.3,
                 fill="skyblue", colour="black") +
  labs(title="Distribution of Iris Sepal Length",
       x="Sepal Length (cm)", y="Frequency") +
  theme_minimal()
```

A histogram groups continuous values into intervals (bins) and displays
their frequency.

## Q4(B). Power BI proportions and percentages of Sales and Profit by Region, Category and Sub-Category

Create measures:

``` dax
Total Sales = SUM(Orders[Sales])
Total Profit = SUM(Orders[Profit])

Sales % Total = 
DIVIDE(
    [Total Sales],
    CALCULATE([Total Sales], ALL(Orders))
)

Profit % Total = 
DIVIDE(
    [Total Profit],
    CALCULATE([Total Profit], ALL(Orders))
)
```

Steps: 1. Select **Modeling → New measure** and create the measures
above. 2. Select `Sales % Total` and `Profit % Total`; set Format to
**Percentage**. 3. Add a **Matrix** visual. 4. Rows: Region, Category,
Sub-Category. 5. Values: Total Sales, Sales % Total, Total Profit,
Profit % Total. 6. Expand/collapse the hierarchy to view overall and
category/sub-category proportions.

The denominator removes filters from those three grouping columns, so
each row is compared with the overall total under the remaining report
filters.

## Q5(A). Group by Region, Category and Sub-Category and calculate total Sales

``` r
library(readxl)
library(dplyr)

sales <- read_excel("Sample - Superstore.xlsx")

sales_summary <- sales %>%
  group_by(Region, Category, `Sub-Category`) %>%
  summarise(TotalSales=sum(Sales, na.rm=TRUE),
            .groups="drop")

print(sales_summary)
```

## Q5(B). Power BI report comparing current vs historical performance and actual vs target

1.  Load Superstore and ensure Order Date is Date.
2.  Create a date table:

``` dax
Calendar =
CALENDAR(
    MIN(Orders[Order Date]),
    MAX(Orders[Order Date])
)
```

3.  In Table tools, mark Calendar as the date table. Create a
    relationship from `Calendar[Date]` to `Orders[Order Date]`.
4.  Create measures:

``` dax
Total Sales = SUM(Orders[Sales])

Sales Previous Year =
CALCULATE(
    [Total Sales],
    SAMEPERIODLASTYEAR(Calendar[Date])
)

YoY Change = [Total Sales] - [Sales Previous Year]

YoY % =
DIVIDE([YoY Change], [Sales Previous Year])
```

5.  For actual vs target, use the target values supplied by the
    question/instructor. Do not invent target values. If a Targets table
    is provided:

``` dax
Sales Target = SUM(Targets[Target])
Target Variance = [Total Sales] - [Sales Target]
Target Achievement % = DIVIDE([Total Sales], [Sales Target])
```

6.  Use a line chart by month with current and previous-year sales. Use
    a clustered column chart for Actual Sales vs Sales Target. Add cards
    for YoY Change and YoY %; add date/category slicers.

## Q6(A). Bar plot of total Quantity by Category and box plot with one categorical and one continuous variable

``` r
library(readxl)
library(dplyr)
library(ggplot2)

sales <- read_excel("Sample - Superstore.xlsx")

# Bar plot: total Quantity by Category
qty <- sales %>%
  group_by(Category) %>%
  summarise(TotalQuantity=sum(Quantity, na.rm=TRUE))

ggplot(qty, aes(x=Category, y=TotalQuantity, fill=Category)) +
  geom_col() +
  labs(title="Total Quantity by Category",
       x="Category", y="Total Quantity") +
  theme_minimal()

# Box plot: categorical Category vs continuous Sales
ggplot(sales, aes(x=Category, y=Sales, fill=Category)) +
  geom_boxplot() +
  labs(title="Sales Distribution by Category",
       x="Category", y="Sales") +
  theme_minimal()
```

## Q6(B). Power BI dashboard showing Mean, Count, Minimum and Maximum by Category

Create measures:

``` dax
Mean Sales = AVERAGE(Orders[Sales])
Order Row Count = COUNTROWS(Orders)
Minimum Sales = MIN(Orders[Sales])
Maximum Sales = MAX(Orders[Sales])
```

Steps: 1. Load Superstore. 2. Create the four measures using **Modeling
→ New measure**. 3. Add a clustered column chart: X-axis = Category;
Y-axis = Mean Sales. 4. Add a Table visual with Category and the four
measures, or add four Card visuals and a Category slicer. 5. Format
titles and numeric values.

## Q7(A). Pie chart of Iris species with title, legend and specific colours

``` r
library(ggplot2)

ggplot(iris, aes(x = "", fill = Species)) +
  geom_bar(color = "white") +
  coord_polar("y") +
  scale_fill_manual(values = c("tomato", "gold", "steelblue")) +
  labs(title = "Iris Flowers by Species") +
  theme_void()
```

Each species has 50 records in the built-in Iris dataset, so the three
slices are equal.

## Q7(B). Power BI mean/median Sales and Profit, normal distribution, variation and uncertainty

Use the same measures from Q3(B): Mean Sales, Median Sales, Sales SD,
Mean Profit, Median Profit and Profit SD.

1.  Add Cards for mean and median Sales and Profit.
2.  Create a Sales histogram using Sales bins and count of Sales.
3.  Create a Profit histogram similarly.
4.  Compare standard deviation by Category using a column chart or use a
    box plot custom visual if permitted.
5.  Add Region, Category and Order Date slicers.
6.  Describe whether the observed distributions are symmetric, skewed or
    contain outliers. Standard deviation describes spread; do not assume
    normality.

## Q8(A). Scatter plot of Iris with colour and shape by Species

``` r
library(ggplot2)

ggplot(iris, aes(x=Sepal.Length, y=Sepal.Width,
                 colour=Species, shape=Species)) +
  geom_point(size=3) +
  labs(title="Sepal Length vs Sepal Width",
       x="Sepal Length (cm)", y="Sepal Width (cm)",
       colour="Species", shape="Species") +
  theme_minimal()
```

## Q8(B). Power BI dashboard with buttons, bookmarks, slicers and measures

Create measures:

``` dax
Total Sales = SUM(Orders[Sales])
Total Profit = SUM(Orders[Profit])
Total Quantity = SUM(Orders[Quantity])
Profit Margin = DIVIDE([Total Profit], [Total Sales])
```

Steps: 1. Load Superstore. 2. Add Card visuals for Total Sales, Total
Profit and Profit Margin. 3. Add a column chart for Sales by Category, a
bar chart for Profit by Region and a line chart for Sales over Order
Date. 4. Add slicers for Region, Category and Order Date. 5. Open **View
→ Bookmarks** and **View → Selection** panes. 6. Arrange visuals/slicers
for a Sales view; select **Add** in Bookmarks and rename it
`Sales View`. 7. Change the visual arrangement or selections to make a
Profit view; add another bookmark named `Profit View`. 8. Insert →
Buttons → Blank. In Format, turn Action On; set Type = Bookmark and
Bookmark = Sales View. Repeat for Profit View. 9. Test buttons using
Ctrl+click in Power BI Desktop or Reading view. Save the report.

## Q9(A). Scatter plot of two continuous Superstore variables with colour, size and shape

``` r
library(readxl)
library(ggplot2)

sales <- read_excel("Sample - Superstore.xlsx")

ggplot(sales, aes(x=Quantity, y=Profit,
                  colour=Category, size=Sales,
                  shape=Category)) +
  geom_point(alpha=0.6) +
  labs(title="Quantity and Profit by Category",
       x="Quantity", y="Profit",
       colour="Category", size="Sales",
       shape="Category") +
  theme_minimal()
```

Quantity and Profit are the two numeric axes; Category controls colour
and shape; Sales controls point size; alpha controls transparency.

## Q9(B). Power BI dashboard with image navigation and live webpage

1.  Create report pages, for example Overview, Regional Analysis and
    Category Analysis.
2.  Insert → Image; select an image from the computer. If the
    visual/version supports it, use an image URL.
3.  Select the image → Format → Action On → Type = Page navigation →
    Destination = the target report page.
4.  To embed a webpage, use the Web content visual if available and
    allowed by security settings. Enter the webpage URL. Some sites
    block embedding; if so, use a button with a Web URL action to open
    the page externally.
5.  Add Superstore charts and slicers to the pages.
6.  Arrange the pages as a story: overview first, then regional
    performance, then category details.
7.  Test the image/page navigation and save the `.pbix` file.

## Q10(A). Linear and polynomial regression with and without confidence interval using Iris

``` r
library(ggplot2)

# Linear regression WITH confidence interval
ggplot(iris, aes(x=Sepal.Length, y=Petal.Length)) +
  geom_point() +
  geom_smooth(method="lm", se=TRUE) +
  labs(title="Linear Regression with Confidence Interval")

# Linear regression WITHOUT confidence interval
ggplot(iris, aes(x=Sepal.Length, y=Petal.Length)) +
  geom_point() +
  geom_smooth(method="lm", se=FALSE) +
  labs(title="Linear Regression without Confidence Interval")

# Polynomial regression WITH confidence interval
ggplot(iris, aes(x=Sepal.Length, y=Petal.Length)) +
  geom_point() +
  geom_smooth(method="lm", formula=y ~ poly(x, 2), se=TRUE) +
  labs(title="Polynomial Regression with Confidence Interval")

# Polynomial regression WITHOUT confidence interval
ggplot(iris, aes(x=Sepal.Length, y=Petal.Length)) +
  geom_point() +
  geom_smooth(method="lm", formula=y ~ poly(x, 2), se=FALSE) +
  labs(title="Polynomial Regression without Confidence Interval")
```

`method="lm"` fits the model. `y ~ x` is linear; `y ~ poly(x, 2)` is a
second-degree polynomial; `se=TRUE/FALSE` shows/hides the confidence
band.

## Q10(B). Power BI story: Profit vs Order Date by Category and average/minimum/maximum Profit

Create measures:

``` dax
Average Profit = AVERAGE(Orders[Profit])
Minimum Profit = MIN(Orders[Profit])
Maximum Profit = MAX(Orders[Profit])
Total Profit = SUM(Orders[Profit])
```

Steps: 1. Load Superstore and check Order Date and Profit data types. 2.
Add a line chart: X-axis = Order Date (Month or Year-Month); Y-axis =
Profit (Sum); Legend = Category. 3. Add a Table visual: Category,
Average Profit, Minimum Profit and Maximum Profit. 4. Add Cards for
Total Profit and Average Profit. 5. Add slicers for Region and Order
Date. 6. Explain the story by comparing category trends over time and
their average and extreme profit values.

------------------------------------------------------------------------

# DOCUMENT 2 -- DV LAB-internal.docx

## Q1(A). Explain basic components of R programming, data types and data structures

**Basic components** - Variables: store values, e.g. `x <- 10`. -
Operators: arithmetic (`+`, `-`, `*`, `/`), relational (`>`, `<`, `==`)
and logical (`&`, `|`, `!`). - Functions: reusable operations,
e.g. `mean(x)`, `sum(x)`. - Control statements: `if`, `else`, `for`,
`while`. - Packages: collections of functions, e.g. ggplot2 and dplyr.

**Data types** - Numeric: `x <- 10.5` - Integer: `x <- 10L` - Character:
`name <- "R"` - Logical: `flag <- TRUE` - Complex: `z <- 2 + 3i`

**Data structures** - Vector: one-dimensional, same basic type:
`c(1,2,3)`. - Matrix: two-dimensional, same type:
`matrix(1:6, nrow=2)`. - Array: multiple dimensions, same type. - List:
can contain different types: `list(10, "R", TRUE)`. - Data frame:
tabular rows/columns; columns can have different types.

## Q1(B). Steps to install ggplot2

``` r
install.packages("ggplot2")
library(ggplot2)
```

Installation is generally done once. `library(ggplot2)` is run whenever
starting a new R session that needs the package.

## Q1(C). Box plot of Iris: one continuous and one discrete variable; fill, flipped coordinates and border colour

``` r
library(ggplot2)

ggplot(iris, aes(x=Species, y=Sepal.Length, fill=Species)) +
  geom_boxplot(colour="black") +
  coord_flip() +
  labs(title="Box Plot of Iris Species",
       x="Species", y="Sepal Length") +
  theme_minimal()
```

Species is categorical/discrete; Sepal.Length is continuous. `fill`
colours boxes, `colour` changes the border, and `coord_flip()` flips the
axes.

## Q2(A). Syntax for loading Excel and CSV files and basic graph plotting syntax

``` r
# CSV
data1 <- read.csv("data.csv")

# Excel
install.packages("readxl")  # once
library(readxl)
data2 <- read_excel("data.xlsx")

# Basic base-R plot
plot(data2$x, data2$y,
     main="Basic Scatter Plot",
     xlab="X values", ylab="Y values",
     col="blue", pch=19)
```

General ggplot2 syntax:

``` r
ggplot(data, aes(x=variable1, y=variable2)) +
  geom_point() +
  labs(title="Title", x="X label", y="Y label") +
  theme_minimal()
```

## Q2(B). Functionality and functions of dplyr

`dplyr` is used to manipulate data frames.

``` r
install.packages("dplyr")  # once
library(dplyr)

select(data, Name, Marks)       # select columns
filter(data, Marks > 80)        # filter rows
mutate(data, Bonus=Marks+5)     # create column
arrange(data, Marks)            # sort ascending
arrange(data, desc(Marks))      # sort descending
group_by(data, Department)      # group rows
summarise(data, Avg=mean(Marks))# calculate summary
count(data, Department)         # count records by group
```

## Q2(C). Power BI cards for Profit Margin and Cities Count

Create measures:

``` dax
Total Sales = SUM(Orders[Sales])
Total Profit = SUM(Orders[Profit])
Profit Margin = DIVIDE([Total Profit], [Total Sales])
Cities Count = DISTINCTCOUNT(Orders[City])
```

Steps: 1. Get Data → Excel → load Superstore. 2. Modeling → New measure;
create the four measures. 3. Select Profit Margin and set its format to
Percentage. 4. Insert a Card visual; drag Profit Margin into it. 5.
Insert another Card visual; drag Cities Count into it. 6. Add titles and
format the cards.

## Q3(A). Different types of graphs in R with representation

-   **Bar chart:** compares categories; `barplot()` or ggplot2
    `geom_bar()`/`geom_col()`.
-   **Histogram:** displays frequency distribution of continuous data;
    `hist()` or `geom_histogram()`.
-   **Pie chart:** displays proportions; in ggplot2 use `geom_bar()`
    with `coord_polar("y")`.
-   **Line graph:** shows trends over an ordered variable/time;
    `plot(type="l")` or `geom_line()`.
-   **Scatter plot:** shows relationship between two numeric variables;
    `plot()` or `geom_point()`.
-   **Box plot:** shows median, quartiles, spread and outliers;
    `boxplot()` or `geom_boxplot()`.

Example ggplot2 syntax:

``` r
ggplot(iris, aes(x=Sepal.Length, y=Petal.Length)) +
  geom_point()
```

## Q3(B). Explain data reshaping with example

Data reshaping changes data from wide to long or long to wide format.

``` r
install.packages("reshape2")  # once
library(reshape2)

df <- data.frame(
  n=c(1,1,2,2),
  time=c(1,2,1,2),
  x=c(6,3,2,5),
  y=c(1,4,6,9)
)

# Wide to long
melted <- melt(df, id.vars=c("n","time"))
print(melted)

# Long to wide
casted <- dcast(melted, n + time ~ variable,
                value.var="value")
print(casted)
```

`melt()` retains identifier columns and stacks the measured columns into
`variable` and `value`. `dcast()` spreads the variable names back into
columns.

## Q3(C). Power BI field parameters for City and State; clustered column chart of Sales

1.  Load Superstore.
2.  In Power BI Desktop, select **Modeling → New parameter → Fields**.
3.  Select City and State fields; create the parameter. Power BI creates
    a parameter field and a slicer.
4.  Insert a Clustered column chart.
5.  Drag the generated field parameter into the X-axis and Sales into
    Y-axis/Values; set Sales aggregation to Sum.
6.  Use the parameter slicer to switch between City and State.
7.  Give the chart a title such as "Sum of Sales by City/State".

If the Fields parameter option is unavailable, create two separate
charts or use a field parameter after updating Power BI Desktop.

## Q4. Power BI dashboard: Sales by Sub-Category, Quantity over Order Date, Quantity by Region, Sales map by State, Sales/Discount/Profit cards, average Sales by Category

Load Superstore and create these visuals on one report page:

  Visual             Fields
  ------------------ ----------------------------------------------------
  Clustered column   X-axis = Sub-Category; Y-axis = Sales (Sum)
  Line chart         X-axis = Order Date; Y-axis = Quantity (Sum)
  Pie chart          Legend = Region; Values = Quantity (Sum)
  Map                Location = State; Bubble size/Values = Sales (Sum)
  Card 1             Sales (Sum)
  Card 2             Discount (Sum)
  Card 3             Profit (Sum)
  Clustered column   X-axis = Category; Y-axis = Sales (Average)

Steps: 1. Home → Get data → Excel → Load. 2. Select each visual from
Visualizations. 3. Drag the fields into the wells shown in the table. 4.
For the last chart, change Sales aggregation from Sum to Average. 5. Add
titles, format values and arrange the visuals. 6. If the map does not
locate states correctly, set State data category to State/Province and
Country to United States where appropriate for the standard Superstore
dataset.

## Q5(A). Differentiate discrete and continuous variables

  -----------------------------------------------------------------------
  Discrete                            Continuous
  ----------------------------------- -----------------------------------
  Countable values                    Measurable values

  Usually whole-number counts         Can take decimal values

  Gaps between possible values        Any value within a range is
                                      possible

  Example: number of students, cars   Example: height, weight,
                                      temperature
  -----------------------------------------------------------------------

In charts, discrete/categorical values often form bar-chart groups;
continuous values commonly appear on scatter plots, histograms and box
plots. This is a common pattern, not an absolute X/Y rule.

## Q5(B). Scatter plot of Iris using colour, size, shape and transparency

``` r
library(ggplot2)

ggplot(iris, aes(x=Sepal.Length, y=Petal.Length,
                 colour=Species, size=Sepal.Width,
                 shape=Species)) +
  geom_point(alpha=0.7) +
  labs(title="Iris Scatter Plot",
       x="Sepal Length", y="Petal Length",
       colour="Species", shape="Species",
       size="Sepal Width") +
  theme_minimal()
```

Colour and shape identify species; point size maps Sepal.Width; alpha
controls transparency.

## Q5(C). Power BI overall/category proportions and percentage of Sales

Create measures:

``` dax
Total Sales = SUM(Orders[Sales])
Sales % of Overall =
DIVIDE(
    [Total Sales],
    CALCULATE([Total Sales], REMOVEFILTERS(Orders))
)
```

If the question asks percentage within the selected category/region
context, use the appropriate denominator and filter-removal columns. For
example, for percentage of total within each Category while rows show
Sub-Category:

``` dax
Sales % of Category =
DIVIDE(
    [Total Sales],
    CALCULATE([Total Sales], REMOVEFILTERS(Orders[Sub-Category]))
)
```

Steps: 1. Format the percentage measures as Percentage. 2. Add a Matrix
visual. 3. Rows = Region, Category and/or Sub-Category. 4. Values =
Total Sales and percentage measures. 5. Add slicers if required. The
denominator determines whether the percentage is of overall total or
within a category.

## Q6(A). Histogram using mtcars

``` r
library(ggplot2)

ggplot(mtcars, aes(x=mpg)) +
  geom_histogram(binwidth=5,
                 fill="skyblue", colour="black") +
  labs(title="Histogram of MPG",
       x="Miles Per Gallon", y="Frequency") +
  theme_minimal()
```

## Q6(B). Pie chart using Iris with title, legend and specific colours

``` r
library(ggplot2)

ggplot(iris, aes(x="", fill=Species)) +
  geom_bar() +
  coord_polar("y") +
  scale_fill_manual(values=c(
    "setosa"="red",
    "versicolor"="green",
    "virginica"="blue"
  )) +
  labs(title="Iris Species Distribution", fill="Species") +
  theme_void()
```

`geom_bar()` counts species; `coord_polar("y")` converts the stacked bar
into a pie chart.

## Q6(C). Power BI mean and median cards

Create measures:

``` dax
Mean Sales = AVERAGE(Orders[Sales])
Median Sales = MEDIAN(Orders[Sales])
```

Steps: 1. Modeling → New measure → create both measures. 2. Insert two
Card visuals. 3. Put Mean Sales in the first card and Median Sales in
the second. 4. Add titles and format decimal places.

## Q7. Comprehensive Power BI dashboard with buttons, bookmarks, slicers and measures

Create measures:

``` dax
Total Sales = SUM(Orders[Sales])
Total Profit = SUM(Orders[Profit])
Total Quantity = SUM(Orders[Quantity])
Total Discount = SUM(Orders[Discount])
Profit Margin = DIVIDE([Total Profit], [Total Sales])
```

Steps: 1. Load Superstore using Get Data → Excel. 2. Add cards for Total
Sales, Total Profit, Total Quantity, Total Discount and Profit Margin.
3. Add a column chart for Sales by Category/Sub-Category. 4. Add a line
chart for Sales over Order Date. 5. Add a bar chart for Profit by Region
and a pie chart for Quantity by Region. 6. Add slicers for Region,
Category, Segment and Order Date. 7. Open View → Bookmarks and Selection
panes. 8. Create a Sales-focused view; add a bookmark named Sales View.
9. Create a Profit-focused view; add a bookmark named Profit View. 10.
Insert → Buttons → Blank; turn Action On; choose Bookmark as the action
type and select Sales View. Repeat for Profit View. 11. Test buttons
with Ctrl+click in Desktop; save the `.pbix` file.

## Q8(A). Functions in R for plotting

Common base R plotting functions:

-   `plot()` -- general plot/scatter plot.
-   `barplot()` -- bar chart.
-   `hist()` -- histogram.
-   `pie()` -- pie chart.
-   `boxplot()` -- box plot.
-   `lines()` -- adds lines to an existing plot.
-   `points()` -- adds points to an existing plot.
-   `legend()` -- adds a legend.
-   `title()` -- adds title and labels.

ggplot2 equivalents include `geom_point()`, `geom_bar()`, `geom_col()`,
`geom_histogram()`, `geom_boxplot()`, `geom_line()`.

## Q8(B). Explain the structure/seven core elements of ggplot2

The seven common components are:

1.  **Data:** dataset used for plotting, e.g. `iris`.
2.  **Aesthetics (`aes()`):** maps variables to x, y, colour, size,
    shape, etc.
3.  **Geometries (`geom_*()`):** visual marks such as points, bars,
    lines and boxes.
4.  **Facets:** split one plot into multiple panels.
5.  **Statistics (`stat_*()`):** statistical transformations such as
    counts or summaries.
6.  **Scales:** control mapping of data to visual properties, colours,
    axes and legends.
7.  **Coordinate system (`coord_*()`):** controls how data are
    positioned, e.g. `coord_flip()` or `coord_polar()`.

Additional formatting is commonly done with `labs()` and `theme()`.

General structure:

``` r
ggplot(data, aes(x=variable1, y=variable2)) +
  geom_point() +
  labs(title="Title", x="X label", y="Y label") +
  theme_minimal()
```

## Q8(C). Linear and polynomial regression without confidence interval

``` r
library(ggplot2)

# Linear regression without confidence interval
ggplot(iris, aes(x=Sepal.Length, y=Petal.Length)) +
  geom_point() +
  geom_smooth(method="lm", formula= y~x, se=FALSE) +
  labs(title="Linear Regression without CI")

# Polynomial regression without confidence interval
ggplot(iris, aes(x=Sepal.Length, y=Petal.Length)) +
  geom_point() +
  geom_smooth(method="lm",
              formula=y ~ poly(x, 2),
              se=FALSE) +
  labs(title="Polynomial Regression without CI")
```

## Q9. Control aesthetics: colour, size, legend and facets using Iris

``` r
library(ggplot2)

ggplot(iris, aes(x=Sepal.Length, y=Petal.Length,
                 colour=Species, size=Sepal.Width)) +
  geom_point() +
  facet_wrap(~Species) +
  labs(title="Iris Scatter Plot by Species",
       x="Sepal Length", y="Petal Length",
       colour="Species", size="Sepal Width") +
  theme_minimal()
```

-   `colour` maps species to point colour.
-   `size` maps Sepal.Width to point size.
-   `facet_wrap()` creates one panel per species.
-   `labs()` names the legends and axes.

## Q10. Power BI: embed pictures, insert picture URL, page navigation, live webpage and visualization story

1.  Create multiple report pages, e.g. Overview, Sales by Region and
    Category Analysis.
2.  Insert → Image → select a local picture.
3.  If supported, use an image URL in the image visual or a table/image
    URL field; set the field's Data category to Image URL where
    applicable.
4.  For page navigation, insert a Button or select an image and turn
    Action On → Type = Page navigation → Destination = the target page.
5.  For a live webpage, use a Web content visual if available and
    permitted. Enter the webpage URL. Some websites block embedding; use
    a Web URL button if embedding is blocked.
6.  Build the story with a clear sequence: overall sales → regional
    performance → category/sub-category details.
7.  Add captions, titles, slicers and navigation buttons. Test each
    action and save.

------------------------------------------------------------------------

# DOCUMENT 3 -- 2024--2025 Sec-A Print

## Q1(A). List and briefly describe the three main views in Power BI Desktop

1.  **Report view:** Create report pages and visuals such as charts,
    cards, maps and slicers.
2.  **Data view:** View table data, inspect columns and create
    calculated columns.
3.  **Model view:** View and manage relationships between tables and
    build the data model.

## Q1(B). Create a data frame for 30 attendance values and generate a customized histogram

``` r
library(ggplot2)

attendance <- c(
  97,93,65,78,88,92,96,71,84,90,
  67,99,73,86,81,94,76,89,70,83,
  87,95,77,68,93,91,85,79,74,97
)

df <- data.frame(Attendance=attendance)

ggplot(df, aes(x=Attendance)) +
  geom_histogram(binwidth=5,
                 fill="skyblue", colour="black") +
  labs(title="Attendance Distribution",
       x="Attendance Percentage",
       y="Frequency") +
  theme_minimal()
```

## Q1(C). Pie chart and bar plot for record count by Region in Superstore

Use the same solution as Document 1 Q1(A): load Superstore, run
`count(Region, name="SalesCount")`, then create a pie chart with
`geom_col() + coord_polar("y")` and a bar chart with `geom_col()`. The
complete code is:

``` r
library(readxl)
library(dplyr)
library(ggplot2)

sales <- read_excel("Sample - Superstore.xlsx")
region_count <- sales %>% count(Region, name="SalesCount")

# Pie
ggplot(region_count, aes(x="", y=SalesCount, fill=Region)) +
  geom_col(width=1, colour="white") +
  coord_polar("y") +
  labs(title="Record Count by Region", fill="Region") +
  theme_void()

# Bar
ggplot(region_count, aes(x=Region, y=SalesCount, fill=Region)) +
  geom_col() +
  geom_text(aes(label=SalesCount), vjust=-0.3) +
  labs(title="Record Count by Region",
       x="Region", y="Record Count") +
  theme_minimal()
```

## Q1(D). Power BI image embedding, picture URL, page navigation and visualization story

Follow Document 2 Q10. In short: insert image; use an image URL where
supported; set an image/button Action to Page navigation; use Web
content visual for a live webpage only if allowed; create an Overview →
Region → Category story with charts and slicers. If a site blocks
embedding, use a Web URL action.

## Q2(A). List the three main Power BI views

Report view -- create visuals and report pages. Data view -- inspect
table data and columns. Model view -- manage table relationships.

## Q2(B). Create a data frame and line chart for 30 SSC CGPA values

``` r
library(ggplot2)

cgpa <- c(
  10,10,7.28,8.91,6.52,9.86,7.12,6.78,9.01,8.47,
  9.42,7.65,8.03,6.95,9.15,8.58,7.89,6.27,9.63,7.34,
  8.26,9.79,8.68,7.03,6.49,9.28,8.14,7.52,9.94,9.8
)

df <- data.frame(
  StudentIndex=1:length(cgpa),
  CGPA=cgpa
)

ggplot(df, aes(x=StudentIndex, y=CGPA)) +
  geom_line(colour="blue") +
  geom_point(colour="red") +
  labs(title="SSC CGPA by Student Index",
       x="Student Index", y="CGPA") +
  theme_minimal()
```

## Q2(C). Bar graph of mean diamond price by Cut and minimum price by Clarity

Use the two programs in Document 1 Q3(A):
`group_by(cut) %>% summarise(MeanPrice=mean(price))` and
`group_by(clarity) %>% summarise(MinPrice=min(price))`, followed by
`geom_col()` for each summary.

## Q2(D). Power BI field parameters for City and State; clustered column chart for Sum of Sales

Use Document 2 Q3(C): Modeling → New parameter → Fields → select City
and State → Create. Add a clustered column chart; use the parameter on
the X-axis and Sales (Sum) as Values. Use the generated slicer to switch
fields.

## Q3(A). Explain basic components, data types and data structures in R

Use Document 2 Q1(A). Key points: variables, operators, functions,
control statements and packages; types include numeric, integer,
character, logical and complex; structures include vector, matrix,
array, list and data frame.

## Q3(B). Create a data frame of 30 roll numbers and CGPAs; generate a bar plot

``` r
cgpa <- c(
  10,10,7.28,8.91,6.52,9.86,7.12,6.78,9.01,8.47,
  9.42,7.65,8.03,6.95,9.15,8.58,7.89,6.27,9.63,7.34,
  8.26,9.79,8.68,7.03,6.49,9.28,8.14,7.52,9.94,9.8
)

roll <- c(
  53,30,1,5,27,18,64,46,14,50,
  39,31,9,8,11,33,2,29,7,42,
  58,12,48,36,41,49,34,28,63,244
)

df <- data.frame(RollNo=roll, CGPA=cgpa)

barplot(df$CGPA,
        names.arg=df$RollNo,
        col="skyblue",
        main="CGPA by Roll Number",
        xlab="Roll Number",
        ylab="CGPA",
        las=2, cex.names=0.6)
```
``` r
library(ggplot2)

# Your data
cgpa <- c(10,10,7.28,8.91,6.52,9.86,7.12,6.78,9.01,8.47,
          9.42,7.65,8.03,6.95,9.15,8.58,7.89,6.27,9.63,7.34,
          8.26,9.79,8.68,7.03,6.49,9.28,8.14,7.52,9.94,9.8)

roll <- c(53,30,1,5,27,18,64,46,14,50,
          39,31,9,8,11,33,2,29,7,42,
          58,12,48,36,41,49,34,28,63,244)

df <- data.frame(RollNo = roll, CGPA = cgpa)

# Create the ggplot
ggplot(df, aes(x = factor(RollNo), y = CGPA)) +
  geom_col(fill = "skyblue", color = "black", width = 0.7) +
  labs(title = "CGPA by Roll Number",
    x = "Roll Number",
    y = "CGPA") +
  theme_minimal()
```
## Q3(C). Group by Region, Category and Sub-Category; calculate total Sales

``` r
library(readxl)
library(dplyr)

sales <- read_excel("Sample - Superstore.xlsx")

result <- sales %>%
  group_by(Region, Category, `Sub-Category`) %>%
  summarise(TotalSales=sum(Sales, na.rm=TRUE),
            .groups="drop")

print(result)
```

## Q3(D). Power BI cards for Profit Margin and Cities Count

``` dax
Total Sales = SUM(Orders[Sales])
Total Profit = SUM(Orders[Profit])
Profit Margin = DIVIDE([Total Profit], [Total Sales])
Cities Count = DISTINCTCOUNT(Orders[City])
```

Add two Card visuals; one displays Profit Margin (formatted as
Percentage), the other displays Cities Count.

## Q4(A). Explain basic components, data types and data structures in R

Use Document 2 Q1(A). Include variables, operators, functions, control
statements, packages; numeric/integer/character/logical/complex types;
vector/matrix/array/list/data frame structures.

## Q4(B). Create attendance and Roll Number data frame; create a pie chart with roll-number labels

``` r
library(ggplot2)

attendance <- c(
  97,93,65,78,88,92,96,71,84,90,
  67,99,73,86,81,94,76,89,70,83,
  87,95,77,68,93,91,85,79,74,97
)

roll <- c(
  53,30,1,5,27,18,64,46,14,50,
  39,31,9,8,11,33,2,29,7,42,
  58,12,48,36,41,49,34,28,63,244
)

df <- data.frame(RollNo=roll, Attendance=attendance)

ggplot(df, aes(x="", y=Attendance, fill=factor(RollNo))) +
  geom_col(width=1, colour="white") +
  coord_polar("y") +
  labs(title="Attendance by Roll Number",
       fill="Roll Number") +
  theme_void()
```

The legend identifies each slice by roll number. If literal labels on
the slices are required, add
`geom_text(aes(label=RollNo), position=position_stack(vjust=0.5))`.

## Q4(C). Linear and polynomial regression with and without confidence interval using Iris

Use the four programs in Document 1 Q10(A): linear formula `y ~ x`;
polynomial formula `y ~ poly(x, 2)`; `se=TRUE` for confidence interval
and `se=FALSE` to hide it.

## Q4(D). Power BI clustered column chart showing Average Sales by Category

1.  Load Superstore.
2.  Add Clustered column chart.
3.  X-axis = Category.
4.  Y-axis/Values = Sales.
5.  Open the Sales field dropdown in the visual well and select
    **Average** (not Sum).
6.  Add title "Average Sales by Category".

## Q5(A). Explain different types of graphs available in R with diagrams

Describe: - Bar chart -- compare category values/counts. - Histogram --
frequency distribution of continuous data. - Pie chart -- proportions of
a whole. - Line graph -- trends over time/order. - Scatter plot --
relationship between two numeric variables. - Box plot -- median,
quartiles, spread and outliers.

Mention the functions: `barplot()`, `hist()`, `pie()`, `plot(type="l")`,
`plot()`/`geom_point()`, `boxplot()`/`geom_boxplot()`.

## Q5(B). Create a data frame of 30 CGPA values and scatter plot against Student Index

``` r
library(ggplot2)

cgpa <- c(
  10,10,7.28,8.91,6.52,9.86,7.12,6.78,9.01,8.47,
  9.42,7.65,8.03,6.95,9.15,8.58,7.89,6.27,9.63,7.34,
  8.26,9.79,8.68,7.03,6.49,9.28,8.14,7.52,9.94,9.8
)

df <- data.frame(
  StudentIndex=1:length(cgpa),
  CGPA=cgpa
)

ggplot(df, aes(x=StudentIndex, y=CGPA)) +
  geom_point(colour="blue", size=3) +
  labs(title="CGPA of 30 Students",
       x="Student Index", y="CGPA") +
  theme_minimal()
```

## Q5(C). Bar plot of maximum Furniture or Technology Sales for each Region

``` r
library(readxl)
library(dplyr)
library(ggplot2)

sales <- read_excel("Sample - Superstore.xlsx")

result <- sales %>%
  filter(Category %in% c("Furniture", "Technology")) %>%
  group_by(Region, Category) %>%
  summarise(MaxSales=max(Sales, na.rm=TRUE),
            .groups="drop")

ggplot(result, aes(x=Region, y=MaxSales, fill=Category)) +
  geom_col(position="dodge") +
  labs(title="Maximum Sales by Region",
       x="Region", y="Maximum Sales") +
  theme_minimal()
```

## Q5(D). Power BI pie chart of Sum of Quantity by Region

1.  Add a Pie chart visual.
2.  Legend = Region.
3.  Values = Quantity; aggregation = Sum.
4.  Add title "Total Quantity by Region" and data labels.

## Q6(A). Describe seven core elements of ggplot2

1.  Data -- dataset being plotted.
2.  Aesthetics -- map variables to x, y, colour, size, shape.
3.  Geometries -- points, bars, lines, boxes, etc.
4.  Facets -- split plot into panels.
5.  Statistics -- statistical transformations/summaries.
6.  Scales -- map values to colours, sizes, axes and legends.
7.  Coordinates -- control the coordinate system and axis orientation.

Example:

``` r
ggplot(iris, aes(x=Sepal.Length, y=Petal.Length)) +
  geom_point() +
  labs(title="Iris Scatter Plot") +
  theme_minimal()
```

## Q6(B). Bar plot using ggplot2 and mtcars

``` r
library(ggplot2)

ggplot(mtcars, aes(x=factor(cyl))) +
  geom_bar(fill="skyblue", colour="black") +
  labs(title="Number of Cars by Cylinders",
       x="Number of Cylinders", y="Count") +
  theme_minimal()
```

`factor(cyl)` treats cylinder counts as categories; `geom_bar()` counts
cars in each category.

## Q6(C). Multiple scatter subplots using facets and Iris

``` r
library(ggplot2)

ggplot(iris, aes(x=Sepal.Length, y=Petal.Length,
                 colour=Species)) +
  geom_point() +
  facet_wrap(~Species) +
  labs(title="Iris Scatter Plots by Species",
       x="Sepal Length", y="Petal Length") +
  theme_minimal()
```

## Q6(D). Power BI cards for Sum of Sales, Sum of Discount and Sum of Profit

Create measures:

``` dax
Total Sales = SUM(Orders[Sales])
Total Discount = SUM(Orders[Discount])
Total Profit = SUM(Orders[Profit])
```

Add three Card visuals and place one measure in each. Title each card
and format the values.

## Q7(A). Explain different types of graphs in R with diagram representation

Use Document 3 Q5(A): explain bar chart, histogram, pie chart, line
graph, scatter plot and box plot, including each graph's purpose and R
function.

## Q7(B). Create a graph using the given CGPA and Roll Numbers

The extracted question text does not specify the graph type clearly (it
refers to a graph image). A useful plot of CGPA against Roll Number is:

``` r
library(ggplot2)

cgpa <- c(
  10,10,7.28,8.91,6.52,9.86,7.12,6.78,9.01,8.47,
  9.42,7.65,8.03,6.95,9.15,8.58,7.89,6.27,9.63,7.34,
  8.26,9.79,8.68,7.03,6.49,9.28,8.14,7.52,9.94,9.8
)

roll <- c(
  53,30,1,5,27,18,64,46,14,50,
  39,31,9,8,11,33,2,29,7,42,
  58,12,48,36,41,49,34,28,63,244
)

df <- data.frame(RollNo=roll, CGPA=cgpa)

ggplot(df, aes(x=RollNo, y=CGPA)) +
  geom_line(colour="blue") +
  geom_point(colour="red") +
  labs(title="CGPA by Roll Number",
       x="Roll Number", y="CGPA") +
  theme_minimal()
```

If the image in the original paper shows a different chart type, keep
the same `df` and change the geometry (e.g. `geom_point()` for scatter
or `geom_col()` for bars). Do not plot Roll Number and CGPA as two
unrelated lines on the same Y-axis because they have very different
scales.

## Q7(C). Scatter plot using Iris with title, axis labels and colour

``` r
library(ggplot2)

ggplot(iris, aes(x=Sepal.Length, y=Petal.Length,
                 colour=Species)) +
  geom_point() +
  labs(title="Iris Scatter Plot",
       x="Sepal Length", y="Petal Length",
       colour="Species") +
  theme_minimal()
```

## Q7(D). Power BI mean, median Sales/Profit, normal distribution, variation and uncertainty

Use the measures and process in Document 1 Q3(B)/Q7(B): AVERAGE, MEDIAN
and STDEV.P measures for Sales and Profit; cards for mean/median;
histograms for distributions; standard deviation or box plot for
variation; slicers for Region, Category and Date. Interpret the actual
chart rather than assuming normality.

## Q8(A). Differentiate discrete and continuous variables

Use Document 2 Q5(A): discrete values are countable (e.g. number of
students); continuous values are measurable and can take decimal values
(e.g. height, weight, temperature).

## Q8(B). Create the graph using the given CGPA and Roll Numbers

The extracted text does not state the graph type, so use the data frame
and CGPA-by-RollNo plot from Q7(B) unless the original paper's embedded
graph specifies another geometry. The data frame and chart code are
given in Q7(B).

## Q8(C). Histogram using ggplot2 and mtcars

``` r
library(ggplot2)

ggplot(mtcars, aes(x=mpg)) +
  geom_histogram(binwidth=5,
                 fill="skyblue", colour="black") +
  labs(title="Histogram of MPG",
       x="Miles Per Gallon", y="Frequency") +
  theme_minimal()
```

## Q8(D). Power BI clustered column chart of Sum of Sales by Sub-Category

1.  Add a Clustered column chart.
2.  X-axis = Sub-Category.
3.  Y-axis/Values = Sales; aggregation = Sum.
4.  Add title "Total Sales by Sub-Category".
5.  Sort by Sales descending if asked.

## Q9(A). Differentiate discrete and continuous variables

Use Document 2 Q5(A)/Document 3 Q8(A): discrete variables are countable;
continuous variables are measurable and can take any value within a
range.

## Q9(B). Iris pie chart with title, legend and specific colours

``` r
library(ggplot2)

ggplot(iris, aes(x="", fill=Species)) +
  geom_bar() +
  coord_polar("y") +
  scale_fill_manual(values=c(
    setosa="red", versicolor="green", virginica="blue"
  )) +
  labs(title="Iris Species Distribution",
       fill="Species") +
  theme_void()
```

## Q9(C). Create a data frame and perform melt and cast

``` r
install.packages("reshape2")  # once
library(reshape2)

df <- data.frame(
  n=c(1,1,2,2),
  time=c(1,2,1,2),
  x=c(6,3,2,5),
  y=c(1,4,6,9)
)

melted <- melt(df, id.vars=c("n","time"))
print(melted)

casted <- dcast(melted, n + time ~ variable,
                value.var="value")
print(casted)
```

Melt changes wide data to long data, retaining `n` and `time` as
identifiers. Cast converts the long data back to wide format, with `x`
and `y` as columns.

## Q9(D). Power BI line chart for Sum of Sales and Sum of Profit with dual Y-axes by Year

1.  Load Superstore and ensure Order Date is Date.
2.  Create a calculated column:

``` dax
Year = YEAR(Orders[Order Date])
```

3.  Insert a **Line and clustered column chart** or another combo visual
    that supports a secondary axis.
4.  X-axis = Year.
5.  Column Y-axis = Sales (Sum).
6.  Line Y-axis = Profit (Sum).
7.  In Format → Y-axis/Secondary axis, enable the secondary axis for the
    line series if available.
8.  Sort Year ascending. Add title "Sales and Profit by Year".
9.  If the exact Power BI version does not offer a secondary axis in the
    selected visual, use a supported combo visual or a custom visual.

## Q10(A). Describe seven core elements of ggplot2

Use Document 3 Q6(A): Data, Aesthetics, Geometries, Facets, Statistics,
Scales and Coordinates. Explain each role and write the general
`ggplot(data, aes(...)) + geom_*()` structure.

## Q10(B). Box plot using ggplot2 and Iris

``` r
library(ggplot2)

ggplot(iris, aes(x=Species, y=Sepal.Length, fill=Species)) +
  geom_boxplot(colour="black") +
  labs(title="Box Plot of Iris Species",
       x="Species", y="Sepal Length") +
  theme_minimal()
```

Species is categorical and Sepal.Length is continuous. For flipped axes,
add `coord_flip()`.

## Q10(C). Power BI comprehensive dashboard with buttons, bookmarks, slicers and measures

Create measures:

``` dax
Total Sales = SUM(Orders[Sales])
Total Profit = SUM(Orders[Profit])
Total Quantity = SUM(Orders[Quantity])
Total Discount = SUM(Orders[Discount])
Profit Margin = DIVIDE([Total Profit], [Total Sales])
```

Steps: 1. Load Sample Superstore from Excel. 2. Create the measures
above. 3. Add Cards for Sales, Profit, Quantity, Discount and Profit
Margin. 4. Add charts: Sales by Category/Sub-Category, Profit by Region,
Quantity by Region and Sales over Order Date. 5. Add slicers for Region,
Category, Segment and Order Date. 6. Open View → Bookmarks and Selection
panes. 7. Create a Sales view and add bookmark `Sales View`. 8. Create a
Profit view and add bookmark `Profit View`. 9. Insert blank buttons;
turn Action On; set Type = Bookmark; assign the appropriate bookmark.
10. Test with Ctrl+click, format the dashboard, and save the `.pbix`
file.

------------------------------------------------------------------------

# Quick exam revision sheet

## R / ggplot2 syntax to remember

  Task                   Core syntax
  ---------------------- -----------------------------------------------
  Create data frame      `data.frame(x, y)`
  Load CSV               `read.csv("file.csv")`
  Load Excel             `read_excel("file.xlsx")`
  First rows             `head(data)`
  Structure              `str(data)`
  Summary                `summary(data)`
  Bar counts             `geom_bar()`
  Pre-calculated bars    `geom_col()`
  Histogram              `geom_histogram()`
  Scatter                `geom_point()`
  Line                   `geom_line()`
  Box plot               `geom_boxplot()`
  Pie                    `geom_bar() + coord_polar("y")`
  Regression             `geom_smooth(method="lm")`
  Hide confidence band   `se=FALSE`
  Facets                 `facet_wrap(~group)`
  Flip axes              `coord_flip()`
  Group and summarise    `group_by(...) %>% summarise(...)`
  Count by group         `count(group)`
  Melt                   `melt(data, id.vars=...)`
  Cast                   `dcast(data, rows ~ variable, value.var=...)`

## ggplot2 seven elements

1.  Data
2.  Aesthetics
3.  Geometries
4.  Facets
5.  Statistics
6.  Scales
7.  Coordinates

## Power BI field wells -- quick guide

  Visual             Fields
  ------------------ -----------------------------------------------------------
  Card               One measure/value
  Clustered column   Axis = category; Values = numeric measure
  Line chart         X-axis = date/order; Y-axis = numeric measure
  Pie chart          Legend = category; Values = numeric measure
  Map                Location = State; Size/Values = Sales
  Scatter            X = numeric; Y = numeric; Details = ID; Legend = category
  Matrix             Rows = hierarchy; Values = measures
  Slicer             Field to filter, e.g. Region/Category/Date

## DAX functions to remember

-   `SUM()` -- total.
-   `AVERAGE()` -- mean.
-   `MEDIAN()` -- median.
-   `MIN()` -- minimum.
-   `MAX()` -- maximum.
-   `COUNTROWS()` -- row count.
-   `DISTINCTCOUNT()` -- unique count.
-   `DIVIDE()` -- safe division.
-   `CALCULATE()` -- evaluates an expression under modified filter
    context.
-   `REMOVEFILTERS()` -- removes specified filters.
-   `STDEV.P()` -- population standard deviation.

**Final practical reminders** - Use `geom_bar()` when ggplot2 should
count rows automatically. - Use `geom_col()` when you have already
calculated the values. - Use `sum(Quantity)` for total quantity; use
`count(Region)` for number of records. - In Power BI, check the
aggregation (Sum/Average/Count) in each visual's field well. - Replace
table/column names in DAX with the exact names in your imported model. -
If a question refers to an embedded graph image that is not visible in
the text, match the geometry to the original image; the provided data
frame can be reused.
