---
title: "rRecipes"
date: 2022-10-10T06:00:20+06:00
hero: hero.jpg
description: R package for scraping recipes off the web.
menu:
  sidebar:
    name: "rRecipes"
    identifier: rRecipes
    parent: R
    weight: 10
tags: ["rRecipes","R","Package"]
---

R package for scraping recipes off the web.

<!---More--->

![](https://img.shields.io/badge/Status-Functional-brightgreen) ![](https://img.shields.io/badge/Version-0.2.0-informational) 

<img src='logo.png' align="right" height="210" />
This package contains some simple functions to scrape recipes from the web using the [`rvest`](https://rvest.tidyverse.org/) package. This package is useful if you wish to extract multiple recipes from the web at once. It can also be used to convert recipe text on a website into a easily usable format (e.g., if you wish to copy and paste into another document or recipe book, as I am doing). 

## Download

To download and load the package, run the following code in R: 

{{< highlight R >}}
devtools::install_git("https://github.com/colebaril/rRecipes")
library(rrecipes)
{{< /highlight >}}

# Version 0.2.0

- Added more error messages
- Removed messages printed in console
- Cleaned up the output .txt file to read better

# Functions


## `scrape()`

Use the `scrape()` function employs a variety of other functions to extract recipes from the web. Internet connection required. This function prints recipes to the console and saves a file called `scraped_recipes.txt` in your working directory. 

### Arguments

`recipe_urls = `:  Takes in 1 or more recipe URLs. Make sure the URL is for a single recipe, not a list.

For example:

{{< highlight R >}}
scrape(recipe_urls = c(
  "https://www.allrecipes.com/recipe/25080/mmmmm-brownies/?internalSource=hub%20recipe&referringContentType=Search",
  "https://www.foodnetwork.ca/recipe/the-pioneer-woman-bbq-pork-walking-tacos-are-the-ultimate-snack/",
  "https://tasty.co/recipe/slow-cooker-ribs")) 
{{< /highlight >}}
## Search Recipe Websites 

For each supported website, a search feature has been or is being implemented.

The `search_recipes()` function works by taking in an argument `query =` which can be any food you want to search for and `site =`, wwhich can be any of the supported sites, and will return the top 10 URLs for your query by default. You can also specify the number of URLs returned as demonstrated below. The number returned may be less than the number requested where there are no more recipes. 

### Arguments

`query =`: The search term (e.g., "apple pie"). 

`site =`: The site you wish to search (e.g., "allrecipes"). 

**The following commands may be entered for `site`**

- Foodnetwork.ca: `foodnetwork`
- allrecipes.com: `allrecipes`
- thepioneerwoman.com: `pioneerwoman`


`number =`: The number of URLs you wish to return. There may be less results than requested.

### Example

For example, running this:

{{< highlight R >}}
search_recipes(query = "apple pie",
               site = "allrecipes",
               number = 10)
{{< /highlight >}}
Yields this:
{{< highlight R >}}
 [1] "https://www.allrecipes.com/recipe/283215/salted-caramel-apple-pie/"             
 [2] "https://www.allrecipes.com/recipe/15806/chemical-apple-pie-no-apple-apple-pie/" 
 [3] "https://www.allrecipes.com/recipe/235346/apple-pie-moonshine/"                  
 [4] "https://www.allrecipes.com/recipe/261468/apple-pie-dip/"                        
 [5] "https://www.allrecipes.com/recipe/234166/apple-pie-liquor/"                     
 [6] "https://www.allrecipes.com/recipe/213569/grandmas-iron-skillet-apple-pie/"      
 [7] "https://www.allrecipes.com/recipe/255040/awesome-apple-pie-cookies/"            
 [8] "https://www.allrecipes.com/recipe/239918/apple-jam-apple-pie-in-a-jar/"         
 [9] "https://www.allrecipes.com/recipe/15683/dutch-apple-pie-with-oatmeal-streusel/" 
[10] "https://www.allrecipes.com/recipe/218330/grandmas-apple-pie-ala-mode-moonshine/"     
{{< /highlight >}}

# Use Pipes 

The functions are designed to work with pipes. For example, see below as we search for chocolate banana recipes and scrape. In this example, The outputs are saved  in a file called `scraped_recipes.txt` in your file directory.

Running this code:
{{< highlight R >}}
search_recipes(query = "chocolate banana",
               site = "allrecipes",
               number = 20) %>%
  scrape(format = "text")
{{< /highlight >}}

Yields this in the console:

{{< highlight R >}}
> search_recipes(query = "chocolate banana",
+                site = "allrecipes",
+                number = 20) %>%
+   scrape()
Retrieving recipe data...
Searching the web for recipes... Please wait.
✓ Recipes saved (appended) to scraped_recipes.txt in your working directory.
{{< /highlight >}}

Recipes are exported in a .txt file in the following format for easy viewing and copy-pasting:

{{< highlight R >}}
Chocolate Banana Muffins
 
Ingredients
1 ½ cups all-purpose flour
¼ cup unsweetened cocoa powder
1 ½ teaspoons baking powder
½ teaspoon baking soda
½ teaspoon salt
½ cup white sugar
½ cup canola oil
¼ cup milk
1  egg
1 teaspoon vanilla extract
2 large very ripe bananas, mashed
¾  mini semisweet chocolate chips
 
Directions
Preheat the oven to 350 degrees F (175 degrees C). Line a12-cup muffin tin with paper liners.
Combine flour, cocoa, baking powder, baking soda, and salt together in a large bowl.
Whisk sugar, oil, milk, egg, and vanilla extract together in a separate bowl until smooth; stir into flour mixture until just moistened. Fold in bananas and chocolate chips.
Divide batter among the muffin cups, filling each about 3/4 full.
Bake in the preheated oven until a toothpick inserted into the center comes out clean, 20 to 25 minutes.
 
--
{{< /highlight >}}

# Supported Sites

1. allrecipes.com
2. foodnetwork.ca (Canadian version)
3. tasty.co (not for searching)
4. thepioneerwoman.com

# In Development 

Currently exploring ways to:

- Format the .txt differently depending on desired output (e.g., Markdown formatting)
- Return recipes as a data frame 
- Return recipes in a nice looking table

# References 

Wickham H, François R, Henry L, Müller K (2022). _dplyr: A Grammar of Data Manipulation_. R package version 1.0.10,
<https://CRAN.R-project.org/package=dplyr>.

Wickham H (2022). _rvest: Easily Harvest (Scrape) Web Pages_. R package version 1.0.3, <https://CRAN.R-project.org/package=rvest>.

Wickham H, Girlich M (2022). _tidyr: Tidy Messy Data_. R package version 1.2.1, <https://CRAN.R-project.org/package=tidyr>.

Bache S, Wickham H (2022). _magrittr: A Forward-Pipe Operator for R_. R package version 2.0.3,
<https://CRAN.R-project.org/package=magrittr>.

Dowle M, Srinivasan A (2021). _data.table: Extension of `data.frame`_. R package version 1.14.2,
<https://CRAN.R-project.org/package=data.table>.

Yu G (2020). _hexSticker: Create Hexagon Sticker in R_. R package version 0.4.9, <https://CRAN.R-project.org/package=hexSticker>.

Qiu Y, details. aotifSfAf (2022). _sysfonts: Loading Fonts into R_. R package version 0.8.8, <https://CRAN.R-project.org/package=sysfonts>.

Ooms J (2021). _magick: Advanced Graphics and Image-Processing in R_. R package version 2.7.3, <https://CRAN.R-project.org/package=magick>.

References code:

{{< highlight R >}}
library(purrr)
c("dplyr", "rvest", "tidyr", "magrittr", "data.table", "hexSticker", "sysfonts", "magick") %>%
  map(citation) %>%
  print(style = "text")
{{< /highlight >}}

Logo Code:

{{< highlight R >}}
library(hexSticker)

sysfonts::font_add_google("Zilla Slab", "pf", regular.wt = 500)

hexSticker::sticker(
  subplot = ~ plot.new(), s_x = 1, s_y = 1, s_width = 0.1, s_height = 0.1,
  package = "rrecipes", p_x = 1, p_y = 1, p_size = 30, h_size = 1.2, p_family = "pf",
  p_color = "#00738c", h_fill = "#FFF9F2", h_color = "#00738c",
  dpi = 320, filename = "man/figures/logo.png"
)

magick::image_read("man/figures/logo.png")
{{< /highlight >}}