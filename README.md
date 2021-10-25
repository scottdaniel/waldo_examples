Waldo exmaples
================

# Introduction

Waldo was originally created as a tool for testing equality of objects
when developing R packages (see `testthat::expect_equal()`). The purpose
of the main function `compare()` is to determine the difference between
two R objects and display them. That way, if there are differences, the
test will fail and one can determine the source of the bug. However,
could we not use this function to ascertain difference in data sets?

## Metadata

A common problem is that we have a matrix of say, KEGG ortholog counts,
and a data-frame full of samples. NB: This is example data from a mouse
project.

``` r
my_data <- read_tsv("data_matrix.tsv", show_col_types = FALSE)
my_samples <- read_tsv("sample_sheet.tsv", show_col_types = FALSE)
```

### Sub-setting

A common practice is to subset the data to “drill-down” to pertinent
details. We might filter out the controls or just look at the top 10
genes in a dataset.

Like so:

``` r
my_subset <- my_data %>%
  group_by(geneID) %>%
  summarise(sum_count = sum(count)) %>%
  slice(1:10) %>%
  left_join(my_data, by = "geneID") %>%
  select(SampleID, count, geneID)

my_subset %>%
  ggplot(aes(x = SampleID, y = count, fill = geneID)) +
  geom_bar(stat = "identity", position = "fill") +
  theme_bw() +
  theme(axis.text.x = element_blank(), strip.background = element_blank()) +
  scale_fill_brewer(palette = "Paired") +
  scale_y_continuous(labels = scales::percent) +
  labs(title = "Top 10 KEGG orthologs",
       y = "Proportion among the top 10",
       x = "")
```

![](README_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

## Which ones were left out?

With Waldo, we can quickly determine which genes were left out of the
graph:

``` r
waldo::compare(unique(my_data$geneID), unique(my_subset$geneID))
```

    ##      old      | new                     
    ##  [8] "K00009" | "K00009" [8]            
    ##  [9] "K00010" | "K00010" [9]            
    ## [10] "K00012" | "K00012" [10]           
    ## [11] "K00013" -                         
    ## [12] "K00014" -                         
    ## [13] "K00015" -                         
    ## [14] "K00016" -                         
    ## [15] "K00018" -                         
    ## [16] "K00019" -                         
    ## [17] "K00020" -                         
    ##  ... ...        ...      and 83 more ...

``` r
# compare(my_data, my_subset)
# can not directly compare dataframes!
# gives error: Error in compare_proxy.spec_tbl_df(x, paths[[1]]) : unused argument (paths[[1]])
```

Another example with metadata this time:
