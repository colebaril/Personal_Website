---
title: "Foodnetwork Search: How accurate is it?"
date: 2023-08-08T06:00:20+06:00
hero: hero.jpg
description: Evaluating the search feature on Foodnetwork (Canada)
menu:
  sidebar:
    name: "Foodnetwork Search"
    identifier: foodnetwork
    parent: data_analysis
    weight: 10
tags: ["Spotify","R","ggplot2", "Foodnetwork", "Recipes"]
draft: true
---

<!--More-->

After spending more time than I care to admit figuring out how the search features on popular recipe websites work to develop my recipe scraping tool (rRecipes), I began to notice a pattern in search results. Over time, the search results became less and less relavent over each iteration of the search. This prompted me to investigate what's going on in order to answer one question: how much does search accuracy degrade over each page of results, and over each individual result. Is there a point at which search results become obsolete? Does this differ by search? 

## Retrieving the data

I retrieved the data for this analysis using my recipe scraping package, `rRecipes`, available for download via GitHub [here]() or by running the following code:

```r
devtools::install_git("https://github.com/colebaril/rRecipes")
library(rrecipes)
```

I will demonstrate the use of some key functions used to obtain the data. 

### Search Functions

The first function involved is called `search_recipes`. It takes in a site, search query (e.g., a food) as well as the number of pages worth of results you want to obtain and passes the information to the relavent search function based on site. The next function (e.g., search_foodnetwork) will crawl through each page of search results and retrieve recipe URLs from each page. The benefit of this function is that it removes all non-recipe posts (e.g., blog posts, articles) that don't contain any actual recipes. 

#### Search Recipes

```r
search_recipes <- function(query, site, pages){

  message(paste0("Searching ", site, " for recipes... Please wait."))

  if("allrecipes" == site){
    search_allrecipes(query, number)
  } else if ("foodnetwork" == site){
    search_foodnetwork(query, pages)
  } else if ("pioneerwoman" == site){
    search_pioneerwoman(query, number)
  }

  else stop('No correct recipe sites entered. Supported sites are "allrecipes" and "foodnetwork".')
}
```

#### Search Foodnetwork

```r
search_foodnetwork <- function(query, pages) {



  landing <- 'https://www.foodnetwork.ca/search/' # This recently changed on the website.

  page <- "/?page="

  query <- gsub(" ", "%20", query) # Change spaces in search query to `%20` to work in URL.

  page_numbers <- paste0("{1:", pages, "}")

  urls <- glue(landing, query, page, page_numbers, sep = "") # Combines base URL and query together.

  if(pages > 50){
    message(paste0("Your query has been rate limited. Estimated run time: ", pages*5, " seconds"))
  }

counter <- 0

all_url_list <- list()

tic()

for(url in urls) {
  site <- rvest::read_html(url) # Extract


  url_list <- site %>%
    rvest::html_elements("a") %>%
    rvest::html_attr("href") %>%
    enframe("id", "url") %>%
    filter(grepl("/recipe/", url)) %>%
    filter(grepl("www", url)) %>%
    distinct(url)

  all_url_list[[length(all_url_list) + 1]] <- url_list

  if(nrow(url_list) == 0) {
    counter <<- counter + 1
    if(counter >= 3) {
      break
    }
  } else {
    counter <<- 0
  }

# Rate limit large requests

        if(pages > 50){
           Sys.sleep(2)
          }
}

toc()

url_list_long <- do.call(rbind, all_url_list)

url_list_long <- as.character(url_list_long$url)

message("Search complete: ", paste0(length(url_list_long), " recipe URLs obtained."))

return(url_list_long)

}
```

### Scrape Functions 

The next set of functions will crawl through each recipe URL obtained by the search function and return the list of recipes either as text or as a data frame saved in comma separated values (csv) format. If a large request is passed through the function, the process will be rate-limited such as to not bombard the site with requests. 

#### Scrape 

```r
scrape <- function(recipe_urls, format) {

  `%rin%` <- function (pattern, list) {
    vapply(pattern, function (p) any(grepl(p, list)), logical(1L), USE.NAMES = FALSE)
  }

  if(length(recipe_urls) > 0) {
  message("Obtaining recipe data.")
  }

  if(length(recipe_urls) > 50) {
    message(paste0("Your request is large and has been rate limited. Estimated run time: ", length(recipe_urls)*4), " seconds")
  }


  tic()
  for (recipe_url in recipe_urls) {
    if ("allrecipes.*" %rin% recipe_urls){
      scrape_allrecipes(recipe_url, format)
    }
    if("foodnetwork.*" %rin% recipe_urls){

      suppressWarnings(scrape_foodnetwork(recipe_url, format))

      if(length(recipe_urls) > 50) {
        Sys.sleep(2)
      }
    }
    if("tasty.*" %rin% recipe_urls){
      scrape_tasty(recipe_url, format)
    }
    if("thepioneerwoman.*" %rin% recipe_urls){
      scrape_pioneerwoman(recipe_url, format)
    }
  }
  message("✓ Recipes successfully saved in your working directory.")
  toc()
}
```
#### Scrape Foodnetwork

```r
scrape_foodnetwork <- function(URL, format) {
 # URL <- "https://www.foodnetwork.ca/recipe/dumpwing/"
recipe <- rvest::read_html(URL)

title <- rvest::html_nodes(recipe, ".article-title") %>%
  rvest::html_text()

rating <- rvest::html_nodes(recipe, ".blog-author .rate-recipe-summary") %>%
  rvest::html_text()
rating <- gsub("\\(", " (", rating)

date <- rvest::html_nodes(recipe, ".article-date") %>%
  rvest::html_text()
date <- gsub("Updated ", "", date)

ingredientsq <- rvest::html_nodes(recipe, ".recipe-ingredients-title+ .subrecipe .subrecipe-ingredients--quantity") %>%
  rvest::html_text()

ingredientsi <- rvest::html_nodes(recipe, ".subrecipe-ingredients--name") %>%
  rvest::html_text()

ingredients <- paste(ingredientsq, ingredientsi)
ingredients <- gsub("\"", " inch", ingredients) # Inches include an escape

ingredients <- gsub("\\u215B", "1/8", ingredients)
ingredients <- gsub("\\u2153", "1/3", ingredients)

directionss <- rvest::html_nodes(recipe, ".recipe-step-content p") %>%
  rvest::html_text()

directionsn <- rvest::html_nodes(recipe, ".recipe-step-title") %>%
  rvest::html_text()

directions <- paste(directionsn, directionss)
directions <- gsub("\\\\", "", directions) # Inches include an escape



if(format == "text"){

  lbreak <- " "
  data <- paste(rating, "|", date)
  ingredients_title <- "Ingredients"
  directions_title <- "Directions"

  sep <- "--"


  utils::write.table(c(title, data, lbreak, ingredients_title, ingredients,
                       lbreak, directions_title, directions, lbreak, sep, lbreak),
                     file = "scraped_recipes.txt",
                     append = TRUE,
                     row.names = FALSE,
                     fileEncoding = "UTF-8",
                     quote = FALSE,
                     col.names = FALSE)

  } else if(format == "csv"){

    title <- paste0(title)

    rating <- rating

    n_ingredients <- length(ingredients)
    n_directions <- length(directions)
    ingredients <- str_c(c(ingredients), collapse = "\n")
    directions <- str_c(c(directions), collapse = "\n")

    df <- data.frame(title = title,
                     rating = rating,
                     date = date,
                     n_ingredients = n_ingredients,
                     n_directions = n_directions,
                     ingredients = ingredients,
                     directions = directions,
                     URL = URL) %>%
      separate_wider_delim(cols = rating, delim = " (", names = c("rating", "n_ratings"), too_few = "align_start") %>%
      mutate(n_ratings = gsub(")", "", n_ratings))

    write.table(df,
                file = "scraped_recipes.csv",
                append = TRUE,
                sep = ",",
                row.names = FALSE,
                col.names = !file.exists("scraped_recipes.csv")
                ) %>%
      suppressWarnings()

    }

}
```

### Putting it Together

Putting all these functions together looks like this:

```r
search_recipes(query = "chicken",
               site = "foodnetwork",
               pages = 51) %>%
  scrape(., format = "csv")

Searching foodnetwork for recipes... Please wait.
Your query has been rate limited. Estimated run time: 255 seconds
140.11 sec elapsed
Search complete: 373 recipe URLs obtained.
Obtaining recipe data.
Your request is large and has been rate limited. Estimated run time: 1492 seconds
✓ Recipes successfully saved in your working directory.
1357.33 sec elapsed
```

The data is now saved as a csv file. Upon reading into R, this is the data structure:

```r
# A tibble: 373 × 9
   title                                                   rating n_ratings date  n_ingredients n_directions ingredients directions URL  
   <chr>                                                    <dbl> <chr>     <chr>         <dbl>        <dbl> <chr>       <chr>      <chr>
 1 Air Fryer Hoisin Glazed Chicken Thighs                     5   3 ratings Apri…             9            4 "½ cup hoi… "Step 1 W… http…
 2 Guy’s Citrus Chicken                                       5   2 ratings Apri…            17            5 "4 lb(s) b… "Step 1 C… http…
 3 This Healthy Sriracha-Honey Oven-Fried Chicken is Simp…    4.6 11 ratin… Octo…             9            7 "2 cups al… "Step 1 P… http…
 4 Traditional Acadian Chicken Fricot                         4.8 205 rati… Dece…            14           10 "3 lb whol… "Step 1 I… http…
 5 Crispy Chicken Thighs Over Vinegar Beans                   5   3 ratings Apri…            14            4 "6 bone-in… "Step 1 P… http…
 6 Easiest Ever Air Fryer Whole Chicken                       5   2 ratings Apri…             9            7 "1 3-4 lb … "Step 1 P… http…
 7 Oven-Fried Honey Garlic Chicken Bites                      4.9 11 ratin… Janu…            14            7 "1 lb bone… "Step 1 P… http…
 8 This Chicken Caesar Salad Burger is the Best of Both W…    5   6 ratings Augu…            16            5 "1 lb grou… "Step 1 F… http…
 9 Spicy Fried Chicken Sandwich (AKA Nima Sandwich)           4.3 3 ratings May …            18           13 "1 cup but… "Step 1 F… http…
10 West Indian Curry Chicken Poutine is a Must-Try            5   1 ratings Febr…            15            9 "1 ½ lbs f… "Step 1 P… http…
# ℹ 363 more rows
```

## Analysis

### Cleaning 