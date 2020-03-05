# Maryland child support caseload data

## Baltimore Sun analysis

By [Christine Zhang](https://twitter.com/christinezhang)

In August 2019, The Baltimore Sun received a response to a public records request for a database of child support cases in the public child support system (also known as IV-D cases) across Maryland during the 2018 federal fiscal year (which ran from October 1, 2017 to September 30, 2018). The data was provided by the Maryland Department of Human Services in response to a public records request and used in a March 5, 2020 story titled ["At what cost? For Baltimore’s poorest families, the child support system exacts a heavy price — and it’s hurting whole communities"](https://www.baltimoresun.com/news/investigations/bs-md-baltimore-sun-child-support-project-20200305-cddqvji4m5dlvd3n27mnq4e3by-htmlstory.html).

The Sun's findings and analysis are available in the "analysis" markdown file in this repository: `analysis.md`. The pre-processing code is in `cleaning.Rmd`.

If you'd like to run the code yourself in R, you can download the R Markdown files `cleaning.Rmd` and `analysis.Rmd` along with the data in the `input` folder.

The raw datasets are saved in the `input` folder.  The cleaned caseload data is in the `output` folder under `caseload.rds` and `baci_caseload.rds`.

https://twitter.com/baltsundata

## Licensing

All code in this repository is available under the [MIT License](https://opensource.org/licenses/MIT). The data files are available under the [Creative Commons Attribution 4.0 International](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0) license.