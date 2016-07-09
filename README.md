# Single Table Facets

This class is intended as a simple faceted search solution for PHP applications where the data source is a single MySQL table. It does not require any joins or relationships. If you want faceted search, but you want the data source to be as simple as an Excel spreadsheet imported into a MySQL database, this class should help.

## Dependencies

* PHP 5.3.2 or higher
* MySQL 5.5 or higher
* jQuery

## Dev dependencies

* Composer

## Installation

Use composer to bring this into your PHP project. The composer.json should look like this:

```
{
    "require": {
        "usdoj/singletablefacets": "dev-master"
    },
    "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/usdoj/singletablefacets.git"
        }
    ]
}
```

After creating a composer.json similar to the above, do a "composer install".

## Usage

To use the library you need to include the autoloader. For an example of this, see docs/example.index.php. The various parts of the page can be rendered individually using these methods: renderStyles(), renderJavascript(), renderFacets(), renderKeywordSearch(), renderResults(), and renderPager().

## Database table

It is up to you to create the database table that you will be using. Some notes:

1. The column names of the database should be the same as the headers (first row) that will be in the Excel/CSV source file.
2. At least one column must be set in MySQL as a unique index. If the data does not naturally have any unique columns, add an auto-increment column to the database and set it as a unique index.
3. For keyword searches you must add a column to the database. This columns should be able to hold a lot of text. (Recommend using the "longtext" column type.) The name of the column must be `stf_keywords`.
4. The `stf_keywords` column mentioned above, as well as any other columns that you would like to include in the keyword search, must belong to a FULLTEXT index on the table. Note that this has ramifications about the storage engine the table uses: on MySQL 5.6 or higher, you can use InnoDB or MyISAM, but for MySQL 5.5 you must use MyISAM.

## Importing source data

The library includes a command-line tool for re-importing data from a CSV file. That tool can be run with:
```
./vendor/bin/singletablefacets [path-to-config-file] [path-to-source-data]
```
Note that the source data file must be a CSV file.

Tip: You'll probably usually be getting the CSV file from an XLS file. Since Excel has a problem with special characters, a useful command-line tool is "xls2csv" from the "catdoc" library. To install:

* Linux: `apt-get install catdoc`
* Babun: `pact install catdoc`

When using xls2csv, to ensure you don't get encoding issues, specify the destination encoding like so:
```
xls2csv -d utf-8 file.xls > file-utf-8.csv
```

## Configuration

The library depends on configuration in a separate YAML file. See singletablefacets.yml.dist for an example. Here is that example config:
```
# Database credentials: this is the only required section.
database name: myDatabase
database user: myUser
database password: myPassword
database host: localhost
database table: myTable

# Everything else in this document is optional.

# Indicate the columns that are required to have data in order for a row to
# appear in the search results. For example, if you don't want any rows to show
# up without titles, make your title column a required column here.
required columns:
    - myDatabaseColumn1

# Choose the order that you would like the columns to show in the facet list,
# and indicate the human-readable labels to display above each one.
facet labels:
    myDatabaseColumn1: Filter by something
    myDatabaseColumn2: Filter by something else

# Choose the order that you would like the fields to show in search results,
# and indicate the human-readable labels to display above each one. There is
# one special field called "stf_score". This is not an actual column in your
# database/spreadsheet, but you can include it here to display a "relevance"
# field in your search results.
search result labels:
    myDatabaseColumn2: Something
    myDatabaseColumn1: Something Else
    stf_score: Keyword Relevance

# Choose the priority (order) and default direction for the sortable columns.
# As with search result labels, there is a special field called "stf_score",
# which you can include to allow users to sort by (keyword) relevance.
# ASC = ascending, DESC = descending
sort directions:
    myDatabaseColumn1: ASC
    myDatabaseColumn2: DESC
    stf_score: DESC

# List the columns that contain URLs pointing to files with keywords. The
# keywords in these files will be crawled and indexed. Note, this only works on
# PDF and HTML files.
keywords in files:
    - myDatabaseColumn2

# List the columns that should be output as links, using another columns to
# get the destination URLs. For example, if you wanted to display the row's
# title as a link to a document, you might do something like this:
# (The format should be: Link label : Link URL)
    output as links:
    myTitleField: myDocumentURLField

# List the facet columns that should be collapsed at a certain point. Use 0 to
# collapse all items, or for example, 5 to collapse items in excess of 5.
collapse facet items:
    myDatabaseColumn1: 0
    myDatabaseColumn2: 5

# List the columns that should function as additional values for another facet.
# For example, if you have a Tag and Tag2 column, you could indicate that Tag2
# is just additional values for Tag, and they would both appear together.
# This is the ONLY way to give one item multiple values in a single facet.
columns for additional values:
    # Extra column: Main column
    myDatabaseColumn1: myDatabaseColumn2

# List the facet columns that depend on other facets. For example, if you don't
# want "Sub Category" to appear unless "Category" is active, you can set that
# here. This is the only way to imitate a hierarchical setup, and works well
# with "show dependents indented to the right" below.
dependent columns:
    # Child column: Main column
    myDatabaseColumn1: myDatabaseColumn2

# Indent dependents to the right and hide their titles. This gives the effect
# that they are being shown in a hierarchical way.
show dependents indented to the right: true

# List the HTML table columns you would like to give a minimum width (CSS).
# This can be used to cut down on undesirable text wrapping.
minimum column widths:
    myDatabaseColumn1: 75px

# Next to facet items, show the totals in parenthesis.
show counts next to facet items: true

# Choose the text for the keyword search button.
search button text: Search

# Choose the text for the message that shows when there are no results.
no results message: |
    <p>
        Sorry, no results could be found for those keywords.
    </p>

# Display the facet items as checkboxes instead of links.
use checkboxes for facets instead of links: true

# For each page of results, show this many results.
number of items per page: 20

# In the pager, show direct links to this many pages. (Besides the normal
# "Next" and "Previous" buttons.)
number of pager links to show: 5

# Show this blurb in an expandable section beneath the keyword search.
keyword help: |
    <ul>
        <li>Use the checkboxes on the left to refine your search, or enter new keywords to start over.</li>
        <li>Enter multiple keywords to get fewer results, eg: cat dogs</li>
        <li>Use OR to get more results, eg: cats OR dogs</li>
        <li>Put a dash (-) before a keyword to exclude it, eg: dogs -lazy</li>
        <li>Use "" (double-quotes) to match specific phrases, eg: "the quick brown fox"</li>
    </ul>

# Users will click this label to expand the help text above.
keyword help label: "Need help searching?"

# The items within a given facet are normally sorted alphabetically, but setting
# this to true will sort them by their counts, in descending order.
sort facet items by popularity: false

# When crawling remote URLs for keywords, add this prefix to any relative URLs.
# For example, if this is set to: "http://example.com/files/", then a relative
# URL of "mydoc.pdf" will be fetched from "http://example.com/files/mydoc.pdf".
prefix for relative keyword URLs: http://example.com/files/

# Normally when users do a keyword search, the full text (crawled) data is
# included. However if you would like to exclude the full text by default, and
# give the user the option to include it, set this to true.
allow user to exclude full text from keyword search: false

# Out of the box, the system uses MySQL's "Boolean" full-text search system.
# You can read about its supported operators here:
# https://dev.mysql.com/doc/refman/5.5/en/fulltext-boolean.html
# You may find you want different behavior, though, from what MySQL decided on.
# Here a couple of ways to tweak the behavior.

# By default, multiple keywords are treated as an "OR" query. If users want to
# get "AND" behavior, they have to put "+" in front of each word. If you would
# like the default to be "AND", set this value to true. To demonstrate this
# behavior, the following two keyword searches would give identical results:
# - this setting set to false: +dogs +cats
# - this setting set to true:  dogs cats
# As another demonstration, the following two keyword searches would also give
# identical results:
# - this setting set to false: dogs cats
# - this setting set to true:  dogs OR cats
use AND for keyword logic by default: false

# By default the partial word matches will not give results. For example, if
# a row's keywords are all "cats", a search for "cat" will not find results.
# Users can deal with this by putting asterisks at the end of words. For example
# a search for "cat*" would match both "cat" and "cats" (and any other word
# that starts with c-a-t). If you would like all user keywords to be treated
# in this way (automatically getting an asterisk at the end) then set this value
# to true.
automatically put wildcards on keywords entered: false

# If the environment needs to use a proxy, uncomment and fill out this section.
# proxy: 192.168.1.1:8080
# To prevent the use of the proxy for certain URLs, enter partial patterns here.
# proxy exceptions:
#    - .example.com

# If there are any special characters or phrases that need to be altered when
# importing the data from the CSV file, indicate those here. For example, to
# change all occurences of § with &#167; uncomment the lines below.
#text alterations:
#    "§": "&#167;"
```

## Scale limits

Because this solution relies on MySQL's FULLTEXT capabilities, it should scale reasonably well. A Solr implementation would surely perform better though, and might make a good future improvement.
