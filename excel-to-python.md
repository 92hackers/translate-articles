# Excel to Python

## Building front-end Excel workbooks for Python tools

![](https://static.breword.com/38261595751986_.pic_hd.jpg)

During the Build 2016 conference, Microsoft announced that 1.2 billion people around the globe were using Excel [1]. That same year, the estimated population of Earth was 7.4 billion [2].

That is **16.2% of all people on Earth**.

Python comparatively boasts a mere 8.2 million active developers according to a 2019 report [3] — which is 0.001% of the Earth’s population.

With those numbers in mind, it may benefit us to encourage more interactivity between Excel and Python — opening up the floodgates to a swarm of new users for Python-built tools.

The opportunity for Excel front-end’s for Python is huge. In this article, we will take a look at how to do this, and implement a ‘typical’ finance Excel setup sheet.



## Tool First, Excel Later

In almost every scenario I can think of, it is more convenient to build the Python portion of the tool first. However, we must maintain flexibility in the formatting of the ‘inputs’ of the tool.

![](https://static.breword.com/2.gif)

What I mean by this is, if we are for example reading one or two CSV/Excel sheets using Pandas — for the first prototype, we may rely on a given set of column names.

But, if thousands of lines into the code, we are relying on the same hard-coded values, we will run into problems when we attempt to make these input column names dynamic with Excel.

So, in the initial prototyping stages, **if there is no Excel sheet yet**, use the initial section of the code to rename column labels to their internal (and hopefully more descriptive) label names:

At a later point, this will be replaced by our Excel sheet mappings.



## Excel Front-End

Once the Python tool has been built out into a more substantial prototype, it is time for us to begin building the Excel front-end. Firstly, we must decide what variables can be adjusted from the Excel worksheet.

Always build these types of tools under the assumption that the format of the input data will change.

Depending on where you work, and the tool you are developing, this is either very important, or not. Some processes are simply well defined and data formatting is unlikely to change.

But, I would always err on the side of caution and include more, rather than less flexibility via the Excel front-end. Just don’t over-complicate.

![](https://static.breword.com/38281595752119_.pic.jpg)

Using an internal naming system, and allowing the Excel user to specify column mappings is a great example of maintaining flexibility. Now, rather than relying on hard-coded column names, the Excel user can adjust these mappings, without ever touching Python.

## Mappings

The centerpiece of integration is the `mappings` dictionary. This will take an Excel tab containing the ‘settings’ of the tool (I usually name it Mapping).

To populate the mappings dictionary, we need functions for reading the Excel mapping tab. For this, we use **openpyxl**.

We can read a value from a given cell in Excel like so:

Using this method, we can now begin populating our mappings dictionary. We will adjust the code above to add a local path to a ‘tool_setup’ Excel workbook.

Let’s also assume that the initially active sheet may not be the Mapping sheet, and in the case of tabs being added, deleted, or moved, we use a list comprehension to find the ‘Mapping’ tab index:

Now we can add some mappings:

```javascript
mappings = {}
mappings['Amount'] = ws["E4"].value
mappings['Term'] = ws["E5"].value
```

![](https://static.breword.com/38301595752120_.pic_hd.jpg)



## Keeping it Flexible

In the case of rows being added or deleted from our Excel mapping tab, this approach will build an incorrect `mapping` dictionary. To avoid this, we use the `search_col` function. This will iteratively search every cell within a column until finding the cell containing the value we want (or exceeding the row `limit`).

At this point, search_col returns the column and row containing the data we wanted to find.

![](https://static.breword.com/38291595752119_.pic.jpg)



![](https://static.breword.com/38311595752120_.pic.jpg)

This allows us to search for the column mappings table by searching column B for **‘Internal’**, like so:

```javascript
search_col(ws, 'B', 'Internal')
```

`[Out]: ('B', 12)`

From here, we can then create a loop to add mappings from column C to column E to our `mappings` dictionary. Once seeing two or more blank cells, we can be confident the mapping table has ended, and thus we can break from the loop:

Once running this piece of code, we will have a Python dictionary `mappings `that looks like this:

```javascript
{
    'Loan ID': 'loan identifier',
    'Product': 'product type',
     ...
    'Initial Fees': 'init fees'
}
```

If we also want to bring in other variables, for example the filepath, which in the Mapping sheet screenshots is shown to be `data/loanbook.csv`. We simply find the row containing *‘*Filepath’ and extract the corresponding value in column D:

```javascript
row, _ = search_col(ws, 'C', 'Filepath')
mappings['filepath'] = ws[f'D{row}].value
```

![](https://static.breword.com/38321595752120_.pic_hd.jpg)



## Integration

The final step is also the easiest, integrating these new column names inside our Python scripts.

Let’s use the mapping sheet above to read in our data, and convert the input column labels to their internal labels.

```javascript
data = pd.read_csv(mappings['Filepath'])
```

Before converting input column labels to their internal labels, we must swap the key-value pairs around to value-key pairs.

```javascript
# invert the dictionary
inv_mappings = {mappings[key]: key for key in mappings}
```

Although for this simple example it may seem more convenient to do this whilst building the `mappings` dictionary. For more involved tools, I have always found it better to maintain the internal: external mapping format that we use here. Nonetheless, this detail is yours to decide.

Finally, convert input labels, to internal labels:

```javascript
data.rename(inv_mappings, axis=1, inplace=True)
```

We can add more flexibility here. To avoid the potential for leading/trailing white-space or lowercase/uppercase typos, we rewrite this part of the code:

Another optional part that I like to include. When we show the internal column labels in the Excel sheet, they are capitalized and contain normal spacing. However, as a personal preference, I maintain [snake_case](https://en.wikipedia.org/wiki/Snake_case) formatting internally, converting:

```javascript
"Loan ID" -> "loan_id"
"Initial Rate" -> "initial_rate"
```

![](https://static.breword.com/38331595752120_.pic_hd.jpg)

I have seen countless Excel-heavy offices that could save on hundreds of hours spent checking boxes, keying in values, or waiting for Excel models to process even the smallest of datasets.

Although the age of automation and machine learning is quickly automating many Excel heavy domains, Excel isn’t going anywhere quickly.

For now, many industries could reap huge benefits through a tighter integration between the world’s fastest growing programming language and the world’s most used software.

Thanks for reading!



## References

[1] J. Osborne, [Build 2016: the biggest news from Day 1 and 2](https://www.techradar.com/uk/news/computing/pc/build-2016-1318027) (2016), techradar

[2] The World Bank, [Total Population](https://data.worldbank.org/indicator/sp.pop.totl) (2019), The World Bank Open Data

[3] M. Carraz, J. Stichbury, S. Schuermans, P. Crocker, K. Korakitis, C. Voskoglou, [Developer Economis: State of the Developer Nation](https://slashdata-website-cms.s3.amazonaws.com/sample_reports/ZAamt00SbUZKwB9j.pdf) (2019), SlashData

