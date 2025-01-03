AssignmentB-4
================
Jasper Zhao

## Option A – Strings and functional programming in R

First, let’s load the necessary libraries.

``` r
library(stopwords)
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.4     ✔ readr     2.1.5
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.1
    ## ✔ ggplot2   3.5.1     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.3     ✔ tidyr     1.3.1
    ## ✔ purrr     1.0.2     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(janeaustenr)
library(stringr)
library(purrr)
library(testthat)
```

    ## 
    ## Attaching package: 'testthat'
    ## 
    ## The following object is masked from 'package:dplyr':
    ## 
    ##     matches
    ## 
    ## The following object is masked from 'package:purrr':
    ## 
    ##     is_null
    ## 
    ## The following objects are masked from 'package:readr':
    ## 
    ##     edition_get, local_edition
    ## 
    ## The following object is masked from 'package:tidyr':
    ## 
    ##     matches

### Exercise 1

Reference: *stopwords: the R package* is used for identifying stop
words.

Link: <https://cran.r-project.org/web/packages/stopwords/>

We will use the text of Jane Austen’s novel “Emma” from package
‘janeaustenr’

``` r
# Load the English stop words from the stopwords library
stop_words <- stopwords("en")

# Process the words in the Emma
emma_words <- emma %>%
  # Convert every word to lowercase
  str_to_lower() %>%
  # Removes all characters other than lowercase letters spaces and apostrophe 
  # (to prevent separate words such as he's and it's into two words), replacing them with spaces
  str_replace_all("[^a-z\\s']", " ") %>%
  # Split into words
  str_split(" ") %>%                            
  unlist()

# Remove stop words according to the stopwords library, and empty strings
emma_words_filtered <- emma_words %>%
  tibble(word = .) %>%
  filter(!word %in% stop_words, word != "")

# Count word frequencies
word_freq <- emma_words_filtered %>%
  count(word, sort = TRUE)

# Select the top 30 most common words and arrange them according to frequency
top_words <- word_freq %>%
  slice_max(order_by = n, n = 30) %>%
  arrange(n)

# Plot the most common words
ggplot(top_words, aes(x = reorder(word, n), y = n)) +
  geom_bar(stat = "identity") +
  coord_flip() +
  labs(title = "Top 30 Most Common Words in Jane Austen's novel Emma w/o stop words",
       x = "Words",
       y = "Frequency") +
  theme_minimal()
```

![](AssignmentB-4_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

### Exercise 2

My modified rules to Pig Latin is:

- For words begin with a consonant sound, the initial consonant of the
  word is moved to the end.
- For words begin with a vowel sound, the first letter of the word is
  moved to the end.
- Add the suffix “eh” directly to the end of the rearranged word.

#### The function

``` r
#' Convert Words to Modified Pig Latin
#'
#' This function converts words into a modified version of Pig Latin. The conversion rule is defined as follow:
#'
#' - For words begin with a consonant sound, the initial consonant of the word is moved to the end.
#' - For words begin with a vowel sound, the first letter of the word is moved to the end.
#' - Add the suffix "eh" directly to the end of the rearranged word.
#'
#' @param words An alphabetical-characters-only vector of words to be converted to modified Pig Latin.
#'
#' @return A character vector of words converted into the modified Pig Latin.
#'
#' @examples
#' convert_to_pig_latin(c("string", "apple", "banana", "orange", "hello"))
#' # Returns: "ingstreh", "ppleaeh", "ananabeh", "rangeoeh", "elloheh"

convert_to_pig_latin <- function(words) {
  # Define vowels
  vowels <- c("a", "e", "i", "o", "u", "A", "E", "I", "O", "U")
  
  # Input validation
  if (!is.character(words)) {
    stop("Input 'words' must be a character vector.")
  }
  
  if (any(str_detect(words, "[^A-Za-z]"))) {
    stop("All elements in 'words' must contain only alphabetic characters.")
  }
  
  # Helper function to process each word in the input words vector
  process_word <- function(word) {
    # Check if the word starts with a vowel
    starts_with_vowel <- str_sub(word, 1, 1) %in% vowels
    
    if (starts_with_vowel) {
      # Word start with vowel: Move the first letter to the end
      if (str_length(word) == 1) {
        rearranged <- word
      } else {
        rearranged <- str_c(str_sub(word, 2, -1), str_sub(word, 1, 1))
      }
    } else {
      # Word start with consonant: Move the initial consonant cluster to the end
      
      # Find the position of the first vowel
      first_vowel_pos <- str_locate(word, "[AEIOUaeiou]")[1]
      
      if (is.na(first_vowel_pos)) {
        # No vowels in the word, treat the entire word as consonant cluster
        rearranged <- word
      } else {
        if (first_vowel_pos == 1) {
          rearranged <- word
        } else {
          consonant_cluster <- str_sub(word, 1, first_vowel_pos - 1)
          rest_of_word <- str_sub(word, first_vowel_pos, -1)
          rearranged <- str_c(rest_of_word, consonant_cluster)
        }
      }
    }
    
    # Add "eh" to the rearranged word and return it
    return(str_c(rearranged, "eh"))
  }
  
  # Apply the helper function to all words
  pig_latin_words <- map_chr(words, process_word)
  
  return(pig_latin_words)
}
```

#### Usage Examples

``` r
words_a <- c("string", "apple", "banana", "orange", "hello")
convert_to_pig_latin(words_a)
```

    ## [1] "ingstreh" "ppleaeh"  "ananabeh" "rangeoeh" "elloheh"

``` r
words_b <- c("UBC", "Holy")
convert_to_pig_latin(words_b)
```

    ## [1] "BCUeh"  "olyHeh"

#### Unite Tests

Test 1: The function can handle consonant-start words correctly

``` r
# Unit Tests
test_that("Function handles consonant-start words correctly", {
  expect_equal(convert_to_pig_latin("string"), "ingstreh")
  expect_equal(convert_to_pig_latin("hello"), "elloheh")
})
```

    ## Test passed 🌈

Test 2: The function can handle vowel-start words correctly

``` r
test_that("Function handles vowel-start words correctly", {
  expect_equal(convert_to_pig_latin("apple"), "ppleaeh")
  expect_equal(convert_to_pig_latin("orange"), "rangeoeh")
})
```

    ## Test passed 😀

Test 3: The function can handle single-letter words correctly

``` r
test_that("Function handles single-letter words correctly", {
  expect_equal(convert_to_pig_latin("a"), "aeh")
  expect_equal(convert_to_pig_latin("b"), "beh")
})
```

    ## Test passed 🌈

Test 4: The function can throw error for non-character inputs

``` r
test_that("Function throws error for non-character inputs", {
  expect_error(convert_to_pig_latin(123), "Input 'words' must be a character vector.")
  expect_error(convert_to_pig_latin(TRUE), "Input 'words' must be a character vector.")
})
```

    ## Test passed 😸

Test 5: The function can throw error for words with non-alphabetic
characters

``` r
test_that("Function throws error for words with non-alphabetic characters", {
  expect_error(convert_to_pig_latin("hello!"), "All elements in 'words' must contain only alphabetic characters.")
  expect_error(convert_to_pig_latin("good-day"), "All elements in 'words' must contain only alphabetic characters.")
 })
```

    ## Test passed 😸
