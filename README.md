Waldo examples
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

Another example with metadata this time. Subsetting samples sometimes
gets us into trouble if we have also subsetted a matrix of counts.

``` r
sample_subset1 <- my_samples %>%
  filter(study_group %in% "Blue")

my_matrix <- my_data %>%
  filter(SampleID %in% sample_subset1$SampleID) %>%
  pivot_wider(names_from = SampleID, values_from = count) %>%
  column_to_rownames(var = "geneID") %>%
  as.matrix()

head(my_matrix)
```

    ##        D15.1N2T29 D16.1N2T29 D16.1N4T29 D16.1N5T29 D6.3N4T42 D6.3N5T42
    ## K00001        110        116        118        119       179        83
    ## K00002         14         32         22         31        30        60
    ## K00003        635        894        699        665       932       940
    ## K00004          0          1          0          1         2         0
    ## K00005          5         35         21         26        26        72
    ## K00007          0          0          0          0         0         0
    ##        D6.3N7T42 D6.6N3T82 D6.6N5T82 D6.6N6T82 D6.7N2T70
    ## K00001       179       201       111       158       158
    ## K00002        10        31        60        10        28
    ## K00003       876       816       922       705       643
    ## K00004         1         2         2         0         1
    ## K00005         5        20        48         9        23
    ## K00007         0         0         0         1         0

``` r
sample_subset2 <- my_samples %>%
  filter(study_group %in% "Yellow")

my_error <- try(my_matrix[,sample_subset2$SampleID])
```

    ## Error in my_matrix[, sample_subset2$SampleID] : subscript out of bounds

Again, you can quickly find which samples changed:

``` r
compare(colnames(my_matrix), sample_subset2$SampleID, max_diffs = Inf)
```

    ##      old          | new             
    ##  [1] "D15.1N2T29" - "D15.1N6T29" [1]
    ##  [2] "D16.1N2T29" - "D16.1N1T29" [2]
    ##  [3] "D16.1N4T29" - "D16.1N3T29" [3]
    ##  [4] "D16.1N5T29" - "D6.3N1T42"  [4]
    ##  [5] "D6.3N4T42"  - "D6.3N6T42"  [5]
    ##  [6] "D6.3N5T42"  - "D6.6N2T82"  [6]
    ##  [7] "D6.3N7T42"  - "D6.6N4T82"  [7]
    ##  [8] "D6.6N3T82"  - "D6.7N1T70"  [8]
    ##  [9] "D6.6N5T82"  - "D6.7N4T70"  [9]
    ## [10] "D6.6N6T82"  -                 
    ## [11] "D6.7N2T70"  -
