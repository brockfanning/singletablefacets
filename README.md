# Single Table Facets

This class is intended as a simple faceted search solution for PHP applications where the data source is a single MySQL table. It does not require any joins or relationships. If you want faceted search, but you want the data source to be as simple as an Excel spreadsheet imported into a MySQL database, this class should help.

## Dependencies

* PHP/MySQL
* jQuery
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
    ],
    "minimum-stability": "dev"
}
```

After creating a composer.json similar to the above, do a "composer install".

## Usage

To use the library you need to include the autoloader. For an example of this, see docs/example.index.php. The various parts of the page can be rendered individually using these methods: renderStyles(), renderJavascript(), renderFacets(), renderKeywordSearch(), renderResults(), and renderPager().

## Database table

It is up to you to create the database table that you will be using. Note that at least one column MUST be a unique index. Further, you MUST have a blank column in the database called "stf_keywords", which should be able to hold a lot of text. (Recommend using the "blob" column type.) Make sure that the column names of the database are the same as the headers that you have in the Excel spreadsheet source.'

## Importing source data

The library includes a command-line tool for re-importing data from a CSV file. That tool can be run with: ./vendor/bin/singletablefacets [path-to-config-file] [path-to-source-data]. Note that the soure data feel must be a CSV file.

## Configuration

The library depends on configuration in a separate YAML file. See singletablefacets.yml.dist for an example. Here is that example config:

```
# Configuration file for SingleTableFacets.
#
# This should be copied somewhere OUTSIDE of the publicly available files. Ie,
# if the docroot is /var/www/htdocs, this file should be somewhere above that,
# such as /var/www, or adjacent, such as /var/www/libraries/singletablefacets.
#
# You can have any number of these files, named differently, to set up multiple
# instances of SingleTableFacets. Each time you invoke SingleTableFacets, either
# on a web page or on the command line (for refreshing the data) you reference
# this configuration file.
#
# Because this is a YAML file, remember that whitespace is important.

# Database credentials
database name: myDatabase
database user: myUser
database password: myPassword
database host: localhost

# Per the name of this library, we look at only a single table.
database table: myTable

# Provide information about the database columns here. In this example file,
# all options are shown for one database column.
database columns:

    myDatabaseColumn1:
        # Do not display results unless they have data in this column.
        is required: false
        # This column should show up as a facet.
        is a facet: true
        # Above the facet, show this label.
        label for facet: Filter by something
        # The label to display when displaying search results.
        label for search results: Something
        # Only show this many items and then display a "show more" button.
        collapse facet items after: 5
        # This facet is dependent on the another facet. Ie, it will not show up
        # unless the other facet is also active.
        depends on: myDatabaseColumn2
        # Data in this column should be consulted during keyword searches.
        consult during keyword searches: true
        # This columns has URLs pointing to files (PDF, HTML) with keyword data.
        contains URLs to files for indexing keywords: false
        # This column should be used for sorting.
        use for sorting: true
        # The default direction, ASC or DESC, when used for sorting.
        sort direction: ASC
        # Specify another column that this column has additional values for.
        # This allows for one row to have multiple values for a facet, since
        # each column can only have one value per row.
        contains additional values for: myDatabaseColumn2
        # When data from this column is rendered, try to format it as a link
        # pointing to a URL from another column.
        output as link to URL from: myDatabaseColumn2

    myDatabaseColumn2:
        # etc....

# Choose the single column to use as a default for sorting.
default sort column: myDatabaseColumn1

# Choose the order that you would like the columns to show in the facet list.
order for displaying facets:
    - myDatabaseColumn1
    - myDatabaseColumn2

# Choose the order that you would like the fields to show in search results.
order for displaying results:
    - myDatabaseColumn2
    - myDatabaseColumn1

# Additional options

# Do not consider keywords shorter than this number.
minimum valid keyword length: 3

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

# Indent dependents to the right and hide their titles. This gives the effect
# that they are being shown in a hierarchical way.
show dependents indented to the right: true

# Show this blurb in an expandable section beneath the keyword search.
keyword help: |
    <ul>
        <li>Use the checkboxes on the left to refine your search, or enter new keywords to start over.</li>
        <li>Enter multiple keywords to get fewer results, eg: cat dogs</li>
        <li>Use OR to get more results, eg: cats OR dogs</li>
        <li>Put a dash (-) before a keyword to exclude it, eg: dogs -lazy</li>
        <li>Use "" (double-quotes) to match specific phrases, eg: "the brown fox"</li>
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

# Normally keywords are processed to remove common words and such. If you would
# like the search to be very precise, set this to false.
remove common keywords: true
```