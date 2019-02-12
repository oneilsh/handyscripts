#!/usr/bin/env Rscript

# don't show warnings...
# this is to silenze a warning about the input mode,
# this is probably not the best way to do that...
options(warn = -1)

suppressMessages(library(dplyr))
suppressMessages(library(tidyr))
suppressMessages(library(ggplot2))

con <- file("stdin")
load(con)

# dd is the dataframe holding rdat-deconstructed data frame input

cmd <- paste0("dd %>% ", commandArgs(trailingOnly = TRUE)[1])

exp <- parse(text = cmd)

dd <- eval(exp)

con <- pipe("cat")
save(dd, file = con)
