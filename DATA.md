# Data

Every source, unit and cleaning decision, recorded as it is made. Where and when each step ran, and its record, is under "The runs" at the end.

## Sources

### Anonymised MOT tests and results

The **MOT test**, named after the Ministry of Transport that introduced it, is the periodic roadworthiness and environmental check required of cars, light goods vehicles, motorcycles and some passenger vehicles once they pass a prescribed age, at least once a year from then on. Tests are carried out by garages and some local authorities authorised by DVSA, known as **Vehicle Testing Stations**. Most vehicles take their first test at three years; taxis, ambulances and passenger vehicles with nine or more seats at one year. The vehicle classes, and the age at which each is first tested, are tabled under "Test Classes" in the user guide.

Every MOT test carried out in Great Britain since 2005, anonymised, together with the defects and advisories each test recorded. Published under the UK Open Government Licence (OGL).

**One dataset, two publishers.** The Department for Transport (DfT) published the years 2005 to 2023, catalogued on data.gov.uk as [`anonymised_mot_test`](https://ckan.publishing.service.gov.uk/dataset/anonymised_mot_test), with the files at `data.dft.gov.uk`. The Driver and Vehicle Standards Agency (DVSA) publishes from 2023 onwards, catalogued as [`anonymised-mot-tests-and-results`](https://ckan.publishing.service.gov.uk/dataset/anonymised-mot-tests-and-results) and on its own portal, [open.data.dvsa.gov.uk/mot-anonymised](https://open.data.dvsa.gov.uk/mot-anonymised/index.html). The portal is the only place the 2025 files appear: they were added on 22 June 2026, together with a second release of 2024.

**Documentation.** Two user guides, one for [tests before May 2018](https://edh-dvsa-data-gov-uk-files-prod.s3.eu-west-1.amazonaws.com/DfT+MOT+Anonymised+Data+Set+Content+2.docx) and one for [tests after](https://edh-dvsa-data-gov-uk-files-prod.s3.eu-west-1.amazonaws.com/mot-testing-data-user-guide-v5.1.odt), version 5.1. The second describes the files up to 2023 and not the layout used from 2024, which is described below from the files themselves.

#### The files

Each year is published as two files, **test results** and **failure items**, plus one set of lookup tables shared by all years. The layout has changed three times. Every size, compression method and header below is produced by [`evidence/01_source_files.py`](evidence/01_source_files.py), which reads the files over HTTP without downloading them in full, and, for the files downloaded, by [`evidence/03_archive_layout.py`](evidence/03_archive_layout.py), which reads the copies in `data/raw/`.

| Years | Container | Inside | Separator | Columns |
|---|---|---|---|---|
| 2005 to 2016 | gzip, `.txt.gz` | one file per table per year | `\|` | old set |
| 2017 | ZIP | not checked, and not used | | |
| 2018 to 2020 | ZIP | four quarterly CSVs per table; in 2018 and 2019, also folder and `__MACOSX/` entries carrying no data | `,` | old set |
| 2021 | ZIP | twelve CSVs per table, of equal size rather than one a month; the header in double quotes, and in the first row every value but `test_id`, `vehicle_id`, `rfr_id` and `location_id` | `,` | old set |
| 2022 and 2023 | ZIP | one CSV per table; in the 2022 failure items, also a `__MACOSX/` entry carrying no data | `\|` | old set |
| 2024, first release, March 2025 | ZIP | twelve monthly CSVs per table, plus folder and `__MACOSX/` entries carrying no data | `,` | new set |
| 2024 and 2025, extracts, June 2026 | ZIP, stored without compression | twelve monthly CSVs per table | `,` | new set |

The **new column set** adds `completed_date` to both tables and renames the failure item's `location_id` to `mot_test_rfr_location_type_id`. Checked: 2016, the first release of 2024, and every file from 2018 on.

Every file from 2018 on, at its size as published. The extracts are stored without compression, which is why each of their years takes about 9 GB:

| Year | Test results | Failure items | Total |
|---|---|---|---|
| 2018 | 1,098 MB | 384 MB | 1,481 MB |
| 2019 | 1,114 MB | 412 MB | 1,527 MB |
| 2020 | 1,079 MB | 385 MB | 1,463 MB |
| 2021 | 1,246 MB | 631 MB | 1,876 MB |
| 2022 | 1,165 MB | 435 MB | 1,600 MB |
| 2023 | 1,193 MB | 460 MB | 1,653 MB |
| 2024, first release | 1,531 MB | 680 MB | 2,211 MB |
| 2024, extracts | 4,798 MB | 4,330 MB | 9,127 MB |
| 2025, extracts | 4,836 MB | 4,360 MB | 9,195 MB |

The years 2018 to 2021 are served only by the DfT server, which does not resume an interrupted download; 2022 onwards are also on the DVSA server, which does.

**The fetch.** We downloaded the sixteen files used, results and failure items for 2018 to 2023 and the 2024 and 2025 extracts, 27.9 GB in all, and the lookup tables, `lookup.zip`, untouched into `data/raw/`. Each is logged with its size, SHA-256 and source address in `data/raw/fetch_log.tsv`, and a copy of that log is committed as [`data/fetch_log.tsv`](data/fetch_log.tsv), so that the repository shows exactly which files every figure comes from.

**The copy in Azure.** We landed the same 17 files, untouched, in the Blob Storage container `raw`, by [`src/motrisk/fetch.py`](src/motrisk/fetch.py) run on a virtual machine in Azure, so that each file went from the publisher's server straight into storage. The fetcher computes each file's SHA-256 from the bytes as it lands them and stores it with the file, together with the address it came from. Every file's size, SHA-256 and address match its line in `data/fetch_log.tsv`, and the container holds no other file, as [`evidence/08_azure_copy.py`](evidence/08_azure_copy.py) shows.

**The CSVs.** We extracted the 106 CSVs of the 17 archives with [`src/motrisk/extract.py`](src/motrisk/extract.py), in Azure and into `data/extracted/`, which git ignores, locally. Each is named after its archive and its own name, without the folders inside the archive, such as `dft_test_item_2022/test_item.csv`; folder and `__MACOSX/` entries are left out. Each is kept only if it reads to its end, where its CRC-32 is checked, with as many bytes as the archive records. Read with the separator and quoting set out below, the 106 CSVs hold 999,154,614 rows, none skipped for not splitting into as many values as their header has columns, as [`evidence/09_csv_from_azure.py`](evidence/09_csv_from_azure.py) shows. The build reads them through Parquet copies, as set out under "Decisions".

**The watcher.** From 7 to 18 November 2026, an Azure Function, [`function/function_app.py`](function/function_app.py), ran [`src/motrisk/fetch_scheduled.py`](src/motrisk/fetch_scheduled.py) every day at 00:00 UTC. It read DVSA's portal page and took each file whose name starts with `dft_test_result` or `dft_test_item`, ends in `.zip` and holds a year from 2026 onward. It downloaded each such file to compute its SHA-256, and landed in `raw`, as the fetcher above does, every one the container did not hold or held with a different SHA-256. It ran 12 times, each landing nothing, since DVSA had published no file from 2026.

**The 2022 and 2023 results files are compressed with Deflate64**, which Python's `zipfile` cannot open. We read them through [`src/motrisk/archives.py`](src/motrisk/archives.py), which reads Deflate64 with the `zipfile-deflate64` package, in constant memory, and checks each member against the CRC-32 the archive records.

**How each file quotes its values decides how it is read.** Only `make` and `model`, in the test results, hold a comma or a double quote.
- The comma-separated results of 2018 to 2020 and from 2024 enclose a value holding a comma in double quotes, such as the model `"SERIES 1, 80 INCH"`, and escape a double quote inside it with a backslash, such as `"SWB 88\""`. Read that way, every row that holds a double quote splits into as many values as its header has columns, while doubled quotes, the CSV standard, misread 138, 100 and 71 rows of 2018 to 2020.
- The 2021 results enclose most values in double quotes and double a quote inside one. Read that way every row splits correctly, while a backslash as the escape misreads one row.
- The `|`-separated files of 2022 and 2023 need no quoting: the few double quotes in their results sit inside values, so every row splits on the separator alone.
- The failure items of 2018 to 2021 split correctly under either reading, and those from 2022 hold no double quote.

We therefore read the 2021 results with doubled quotes, every other comma-separated file with a backslash as the escape, and the `|`-separated files on the separator alone. We set the quote and escape characters explicitly, rather than leave them to DuckDB's automatic detection, which missed the quoting of January 2025: that month's first quoted row is on line 57,369. These counts are produced by [`evidence/02_csv_quoting.py`](evidence/02_csv_quoting.py) and [`evidence/06_csv_quoting_2018_2023.py`](evidence/06_csv_quoting_2018_2023.py).

#### Test results: one row per test

| Column | Meaning |
|---|---|
| `test_id` | the test; primary key |
| `vehicle_id` | the vehicle, the same across all its tests, derived by the publisher from the registration and the VIN |
| `test_date` | date of the test |
| `test_class_id` | vehicle class tested: 1 and 2 motorcycles, 3 light three-wheelers, 4 cars and light vans, 5 passenger vehicles with thirteen or more seats, 7 goods vehicles of 3,000 to 3,500 kg |
| `test_type` | `NT` normal test, `RT` re-test, `PL` and `PV` partial retests, `ES` statutory appeal; decoded by `mdr_test_type` |
| `test_result` | `P` pass, `F` fail, `PRS` pass after rectification at the station, `ABA` abandoned, `ABR` aborted; decoded by `mdr_test_outcome` |
| `test_mileage` | odometer reading in miles, as entered or validated by the tester; zero or blank where none was taken |
| `postcode_area` | postcode area of the testing station, such as `OX` |
| `make`, `model` | as held in DVSA's vehicle dataset |
| `colour` | as entered or validated by the tester |
| `fuel_type` | fuel code, such as `PE` petrol or `DI` diesel, as entered or validated by the tester; decoded by `mdr_fuel_types` |
| `cylinder_capacity` | engine size, as entered or validated by the tester; the guide states no unit, and values such as 1600 read as cubic centimetres |
| `first_use_date` | the vehicle's date of first use |
| `completed_date` | from 2024 only: when the test was completed, as a UTC timestamp; not described in the guide |

**PRS** is a test that begins in a failing condition and is put right within one hour, before the result is entered, so it is recorded as a pass of a vehicle that arrived failing. DVSA accordingly reports two failure rates: an **initial failure rate** that counts PRS as a failure, and a **final failure rate** that does not. Both are defined under "Comparison with DVSA published statistics" in the user guide.

#### Failure items: one row per defect or advisory recorded in a test

| Column | Meaning |
|---|---|
| `test_id` | the test it was recorded in; many items to one test, and none for a clean pass |
| `rfr_id` | the defect, as a **reason for rejection** (RfR) code; decoded by `item_detail`, together with the test's `test_class_id` |
| `rfr_type_code` | `F` failure, `P` failure rectified at the station, `A` advisory, `M` minor defect |
| `location_id`, from 2024 `mot_test_rfr_location_type_id` | where on the vehicle, by side, end and position; decoded by `mdr_rfr_location` |
| `dangerous_mark` | the tester's mark that a failure was dangerous, used until 19 May 2018 |
| `completed_date` | from 2024 only: when the test was completed, as on the test result |

An **advisory** is a tester's warning about an item that does not fail the test.

#### The lookup tables

The two main tables store codes, not descriptions. The lookup tables translate them, and two of them carry information found nowhere else.

| File | Rows | What it holds |
|---|---|---|
| `item_detail.csv` | 21,069 | one row per defect code per vehicle class: its description, its advisory wording, its **severity** under the 2018 rules, and the group it belongs to |
| `item_group.csv` | 4,136 | the defect groups (brakes, lighting, …) as a tree: each group names its parent, up to `Vehicle` at the top |
| `mdr_rfr_location.csv` | 129 | location code → side, end and position on the vehicle |
| `mdr_fuel_types.csv` | 14 | fuel code → fuel name |
| `mdr_test_outcome.csv` | 7 | result code → outcome |
| `mdr_test_type.csv` | 6 | test type code → kind of test |

Severity and grouping are the two found nowhere else: whether a failure was major or dangerous, and that two different defect codes concern the same part of the vehicle, are recorded only in `item_detail` and `item_group`.

**Every coded column takes its values from a finite list.** `test_type` has the 6 codes of `mdr_test_type` and `test_result` the 7 of `mdr_test_outcome`; three of those the guide does not describe: `EI` inverted appeal, `ABRVE` aborted by VE, and `R` refused. `rfr_type_code` has 4 values, `F`, `P`, `A` and `M`. `item_detail` holds 7,505 distinct defect codes over 21,069 rows, because the guide defines a defect as a code together with a vehicle class, and a code is listed once for each class it applies to. Of the 4,432 codes for class 4, 2,511 are categorised Pre-EU Directive, the codes used before 20 May 2018, and 1,921 belong to the 2018 rules. These counts are produced by [`evidence/01_source_files.py`](evidence/01_source_files.py). Nine codes in the data files are not in the lookup, all recorded only in 2024 and 2025, as [`evidence/05_code_coverage.py`](evidence/05_code_coverage.py) shows.

**Every code's group is listed, and a group number means the same group in every class.** Over all classes, 2,574 code numbers belong to the 2018 rules and 4,931 to the earlier ones, and no number is used by both. Every code-and-class row of `item_detail` finds its group in `item_group` for that class, and no group number carries another name or another parent in another class. Of the 2,574 codes of the 2018 rules, 2,557 are listed in more than one class, and each sits under the same group in every class. The groups directly above those codes are 456, carrying 304 names: 92 names belong to more than one group, *Condition* and *Switch* to nine each, so a group is identified by its number, not its name. These figures are produced by [`evidence/04_lookup_groups.py`](evidence/04_lookup_groups.py).

#### How the tables link

```
 test_result ──────────────── one row per test; test_id, vehicle_id, test_class_id, ...
      │  one test has zero or many items
      ▼
 test_item ────────────────── one row per defect or advisory; test_id, rfr_id, location id
      │  rfr_id from the item, test_class_id from its test
      ▼
 item_detail ──────────────── one row per defect code per vehicle class
      │  test_item_id, test_class_id
      ▼
 item_group ───────────────── the tree of defect groups; parent_id points at the parent

 test_item.location id      → mdr_rfr_location
 test_result.fuel_type      → mdr_fuel_types
 test_result.test_result    → mdr_test_outcome
 test_result.test_type      → mdr_test_type
```

### MOT History API

Not used so far. The [MOT History API](https://documentation.history.mot.api.gov.uk/mot-history-api/download-vehicle-mot-history-data) publishes a full snapshot every Sunday and a file of changes every day, each an archive of gzip-compressed JSON, one record per vehicle carrying its whole test history. Keys are issued only to approved applicants. It is the only source of the MOT data that updates more often than once a year.

## What the publisher says the data does and does not contain

From the post-2018 user guide, unless marked as from the portal.

- **A test describes the vehicle on the day of the test only.** The certificate is not evidence of its condition at any other time, of its general mechanical condition, or of full legal compliance.
- **Only tests that could have ended in a pass are included.**
- **2005 to March 2006 is incomplete**, because computerisation was not complete across Great Britain until 1 April 2006.
- **The rules changed on 20 May 2018**, when EU Directive 2014/45 introduced dangerous, major and minor defects. The portal warns that the change "affected the information presented in these datasets". The `dangerous_mark` used before it is not comparable with the later dangerous category.
- **The defect category was added from the 2022 data set** and has not been added to 2018 to 2021.
- **Mileage before 2022 is sometimes kilometres.** From 2022, readings taken in kilometres are converted to miles; earlier years are not corrected.
- **The location code is wrong in data sets up to 2021**, and correct from 2022.
- **Fuel type is often wrong for hybrids**, such as the Toyota Prius in the 2000s, recorded as some other fuel. Corrected from 2022 only.
- **An unknown date of manufacture is recorded as a first use of 1 January 1971**, so 1971 shows anomalies.
- **A small proportion of vehicles have make and model `UNCLASSIFIED`**, having no valid record in DVSA's vehicle dataset.
- **Postcode areas with fewer than five active testing stations are merged into `XX`**, so that no station can be identified.
- **Test class 0** marks duplicate or replacement certificates for tests made before computerisation.
- **Two years have been reissued** (portal): 2017 in June 2022, to add missing results, and 2022 in December 2023, to correct fuel type codes for a small number of vehicles.

## What the data can and cannot support

- **There is no vehicle table.** Make, model, fuel and first use are repeated on every test row, and `vehicle_id` is the only link between one vehicle's tests.
- **A failure item carries no vehicle class**, and `item_detail` is keyed on defect code and class together, so decoding a defect goes through the item's test.
- **The 2020 extension moved some tests by six months.** Cars, motorcycles and vans whose MOT was due between 30 March and 31 July 2020 were given a six-month extension, and testing became mandatory again for those due from 1 August 2020 ([Which?, 30 June 2020](https://www.which.co.uk/news/2020/06/government-cuts-short-mot-exemption-is-your-cars-mot-due)). A vehicle tested on the extension has a longer gap than a year between that test and the one before, and one due late in July could be put back to January 2021, leaving no test in 2020. We build the rows of 2020 and 2021 as those of every other year: the model is given the time between the two slots, which records the longer interval, and nothing else is corrected. The extension lengthened about 4.3 million training rows, 2.15% of them: of the rows whose previous slot started from 30 March to 31 July 2019, 49.70% have more than 430 days between the two slots' first tests, against 6.28% to 7.15% for the same window of 2021 to 2023, as [`evidence/19_covid_extension.py`](evidence/19_covid_extension.py) shows.
- **Not in the data at all:** repairs made between tests, what any repair cost, which testing station carried out a test beyond its postcode area, the owner, and accidents.
- **The guide and the files disagree on format.** The guide gives dates as `DD-MM-YYYY` and every file as `|`-separated; the files checked carry ISO dates, `2016-01-01`, and those of 2018 to 2021 and from 2024 are comma-separated.

## Decisions

### The three layouts are read into one, through Parquet

We convert each of the 106 CSVs once to a Parquet file, with [`src/motrisk/convert.py`](src/motrisk/convert.py). Each is read with the separator and quoting set out under "The files" above, and every column is kept as text, so that only the container changes and no value is reinterpreted. An empty value is stored as missing, and rows may be stored in another order, which carries no meaning. Each Parquet file is kept only if it holds exactly the rows [`evidence/09_csv_from_azure.py`](evidence/09_csv_from_azure.py) counted for its CSV; the 106 files hold all 999,154,614 rows in under a quarter of the CSVs' size, as [`evidence/11_parquet_in_azure.py`](evidence/11_parquet_in_azure.py) shows. The build then reads the test results of every year in one query and the failure items in another, into two staged tables with the same columns for every year. The new column set's added and renamed columns are among those dropped below. The cleaning rules below are stated on values rather than on years, so they apply to every year alike.

### Every row of the files is accounted for

We build the staged tables over every year with dbt in DuckDB. Every row of the files is either kept or removed by exactly one rule, each counted on its own, and the counts add up to the rows of the files, as the model [`staging_row_counts`](transform/models/staging/staging_row_counts.sql) records in [`data/staging_row_counts_2026-11-12.log`](data/staging_row_counts_2026-11-12.log). A row that meets several rules is counted under the first, in the order of [`transform/macros/classified.sql`](transform/macros/classified.sql), which is why the counts of the bounds differ slightly from those of evidence 13.

| Table | Kept, or removed because | Rows |
|---|---|---|
| tests | kept | 308,127,375 |
|  | before 20 May 2018 | 15,665,371 |
|  | aborted, `ABR` | 1,598,012 |
|  | engine size outside its class's range | 341,084 |
|  | age at the test over 50 years | 249,746 |
|  | abandoned, `ABA` | 189,605 |
|  | mileage above 900,000 | 10,451 |
|  | aborted by a vehicle examiner, `ABRVE` | 234 |
|  | **total, the rows of the files** | **326,181,878** |
| failure items | kept | 633,027,171 |
|  | of a removed test | 30,820,336 |
|  | on a code not kept | 9,074,846 |
|  | on a code not listed for its vehicle's class | 24,846 |
|  | whose test is in no file | 175 |
|  | recorded in a way the rules exclude | 1 |
|  | **total, the rows of the files** | **672,947,375** |

**A failure item whose test is in no file is removed.** 175 items belong to tests that appear in no results file of any year; with no vehicle, date or class, they cannot be placed in any slot.

Every test of the build passes. The vehicles tested in more than one class are reported as a warning rather than a failure, since each test's class is kept as recorded (under "What the model predicts" below).

### Tests from 20 May 2018 to 31 December 2025

We use tests from 20 May 2018, the day the 2018 rules came into force, to the end of 2025, the latest year published. The recent years describe modern cars and the current test, and starting on that date puts every test under one set of rules and one list of defect codes: for class 4, the 1,921 codes of the 2018 rules, rather than those and the 2,511 used before them. The tests of 1 January to 19 May 2018 are in the 2018 files and are excluded. The period cannot start earlier: none of the 29,051,275 failure items recorded before 20 May 2018 carries a code of the 2018 rules. These counts, and those of the next section, are produced by [`evidence/05_code_coverage.py`](evidence/05_code_coverage.py), over all tests, aborted and abandoned ones included, since a failure item carries no test result.

### Which defect codes are kept: those in use from 2018 to 2025, renumbered ones joined

We keep, as data and as the problems the model gives a probability for, the largest set of defects in use throughout the period. A defect is kept when it is first recorded in 2018, from 20 May, and last recorded in 2025; a year without a record in between is allowed, since a rare defect can miss a year by chance. A defect that starts later or stops earlier is dropped, in every year.

**Some defects changed code number during the period, and we join the two numbers into one.** The lookup links no old number to a new one, so we pair them ourselves. Two codes are one defect when they have the same category and are listed for the same vehicle classes, the old one is last recorded in the year the new one is first recorded or the year before, and either they have the same short description and the same group or the same inspection-manual wording, or they have the same group and manual wording and are the only two codes in the lookup with that group, wording and category. The manual wording alone does not identify a defect: 2,332 of the 2,574 codes share theirs with another code. The two rules find 40 pairs of codes, among them the lamps renumbered in 2019, the shock absorber's serious fluid leak in 2021, and the four *not tested* codes that became *unable to be tested* or *unable to be completed* in 2023. Each pair takes the number used first. Of the 40, 36 belong to defects kept under the rule below, and are the pairs in [`transform/seeds/code_dictionary.csv`](transform/seeds/code_dictionary.csv).

Of the 2,559 codes of the 2018 rules recorded from 20 May 2018, 168 are first recorded after 2018 and 159 last recorded before 2025. After joining, 2,305 defects are in use throughout and are kept; 218 codes are dropped, carrying between 1.0% and 1.5% of each year's failure items from 2019 and 2.8% of those of 2018 from 20 May. They include codes the 2018 rules split from an older one, such as the shock absorber's *negligible damping effect*, from 2021. Fifteen codes are never recorded. Of the codes of the earlier rules, 16 are recorded in every year from 2019, and only as advisories: the *Non-component advisories*, such as *Nail in tyre*, 3.5% to 4.2% of each year's items. Those 16 are kept, which makes 2,321 defects. The nine codes missing from the lookup are dropped.

**The build reads these rules from one place.** They are applied by [`src/motrisk/codes.py`](src/motrisk/codes.py), which writes the two dbt seeds the build reads: [`transform/seeds/code_dictionary.csv`](transform/seeds/code_dictionary.csv), each of the 2,357 kept codes with the defect it belongs to, numbered by the code used first, and [`transform/seeds/year_vector_entries.csv`](transform/seeds/year_vector_entries.csv), the record's 4,821 entries in order, each with the vehicle classes its defect is listed for. The evidence file above imports the same rules, so the figures and the build cannot disagree.

### 2024 comes from the extracts

2024 is published twice. We use the extracts of June 2026, published together with 2025 and in the same layout, rather than the first release of March 2025. The first release holds more test-result bytes for the year than the extracts: each of its twelve monthly files is 461 to 746 MB uncompressed, against 312 to 475 MB for the monthly files of the 2025 extracts. Whether that is more rows or a different formatting, such as mileage written as `71614.0`, is not checked.

### `dangerous_mark` is dropped, in every year

The guide says the tester's dangerous mark, used until 19 May 2018, is not directly comparable with the dangerous category the 2018 rules introduced. Keeping it would therefore not make the years before and after that date consistent with each other, so we drop it from the failure items of every year, including those before May 2018. Severity after 20 May 2018 remains available from `item_detail`; codes used before that date carry only "Pre-EU Directive" there.

### `completed_date` is dropped from both tables

The question is which problems a vehicle's tests will record, and what bears on it is when the vehicle was presented, which `test_date` records. When the test was completed adds nothing about the condition of the vehicle, so we drop `completed_date` from the test results and the failure items. It exists only from 2024, so dropping it also leaves every year with the same date columns.

### What is kept from each file

From the test results we keep `test_id` and `vehicle_id`, which link the tables and separate the vehicles but are not given to the model, and `test_date`, `test_class_id`, `test_type`, `test_result`, `test_mileage`, `make`, `model`, `fuel_type`, `cylinder_capacity` and `first_use_date`. We drop `postcode_area` and `colour`, and `completed_date` as above. From the failure items we keep `test_id`, `rfr_id` and `rfr_type_code`, and drop the location and `dangerous_mark`. Without the location, a defect found at two places in one test gives two identical rows.

### Every vehicle class is used

We keep tests of every class, from motorcycles to goods vehicles of up to 3,500 kg, not cars alone. Each class has its own list of defect codes and its own age at first test, as set out under "The lookup tables" and "Test results: one row per test" above.

### Tests stopped before their end are removed

**The model estimates the probability of the problems a vehicle actually has, not of how its tests end.** A test stopped before its end carries no information on the problem itself: it says only that this test did not finish, while we train the model to estimate the probability of the vehicle's problems, which a fleet manager needs in order to decide which parts to inspect. A vehicle whose test is stopped is normally tested again, and that complete test finds what is wrong. A stopped test also leaves parts uninspected, and the absence of a record for them would read as nothing found. So we remove the three kinds of stopped test, with the failure items recorded in them:

- `ABR`, aborted because of a problem with the testing equipment or the tester, not with the vehicle;
- `ABA`, abandoned by the tester because it was unsafe to continue or because some items could not be inspected;
- `ABRVE`, aborted by a DVSA vehicle examiner, a result the user guide does not describe.

They are removed before the tests are grouped into slots (below), so they neither start nor extend one.

### What the model estimates: the probability of every problem in the vehicle's next slot of tests

**We group each vehicle's tests into slots.** Its tests are taken in date order, with aborted and abandoned ones removed and every other type alike: normal tests, retests and appeals. A gap of more than 20 weeks between two consecutive tests starts a new slot. A failure and its retests, days apart, therefore stay in one slot, even across 1 January, while annual tests, about a year apart, fall in different slots. A retest more than 20 weeks after the test before it starts a new slot like any other test. A slot belongs to the year of its first test. The cutoff of 20 weeks falls in the sparse stretch between the two kinds of gap: 21.0% of consecutive tests are less than two weeks apart, the retests and repeated tests of one visit, and 64.5% are 46 to 58 weeks apart, the next annual test, while each four-week band from 8 to 24 weeks holds under 0.4%, so moving the cutoff four weeks either way would move fewer than 0.4% of the pairs, as [`evidence/14_test_gaps.py`](evidence/14_test_gaps.py) shows on 5% of the vehicles.

**A slot's first test is the one the vehicle arrives for, the moment of decision, so on a day with several tests we take the test completed first.** The files give only the day of a test. We take the normal test first, then the lowest mileage, then the lowest `test_id`, since a retest follows the test it retests and an odometer only goes up. The files from 2024 on record when each test was completed, and over a 5% sample of vehicles the normal test was completed first in 297,173 of 298,458 same-day pairs with a test of another type, and the lower of two mileages in 760 of 808 pairs of normal tests, while the lower `test_id` came first only half the time, as [`evidence/16_same_day_tests.py`](evidence/16_same_day_tests.py) shows. The order decides which test's details a row takes, and nothing else, since tests of one day are always in the same slot.

**The probability of each problem in a slot is estimated from the slot before it.** The label is every problem recorded on any test of the slot: each defect code, together with the way it was recorded, as a failure, a failure rectified at the station, a minor defect or an advisory. The inputs are:
- the same record of the previous slot, however long before;
- the groups of that record: for each group directly above a kept defect's code, 1 if the previous slot recorded any entry in it, whatever the way;
- the vehicle's mileage and age at the target slot's first test, since both are known when the vehicle arrives for it, the moment of decision; and its other details as recorded at the previous slot's first test;
- the gap between the first tests of the two slots, in bands (under "The models").

**The groups of the previous slot enter as inputs**, because:
- a vehicle's problems tend to recur within a component, not only as the same defect;
- a group pools its defects, so that the rare ones, too few on their own to carry a weight, still count;
- a group sees a component's condition whichever wording the tester chose among similar defects;
- the small models keep only two facts about the previous slot, to save time, and the group is one of them.

**A row's class is that of the previous slot's first test**, since that slot is what the row learns from. When a vehicle's class differs between the two slots, a defect listed only for the target slot's class cannot be predicted for that row.

**A row whose two slots differ in class is learnt from and scored only for the defects listed for its previous slot's class**, taken as a marginal limit, because:
- about 169,000 rows have their two slots in different classes, 0.08% of the training table, as [`evidence/12_class_changes.py`](evidence/12_class_changes.py) shows;
- most entries are listed for both classes such a vehicle moves between, mostly 4 and 7, so for them these rows already count; only the entries listed for one of the two and not the other lose them, 152 of the 4,821 for classes 4 and 7;
- counting such a row for either class would take about half a day of rebuilds and reruns, for an effect no score would show.

The prediction is made when the vehicle arrives for the target slot's first test, when that test's date is known. A vehicle's first slot serves only as input, and a vehicle with no later slot, whether scrapped, exported or off the road, gives no further row. A vehicle with more than one slot starting in a year has a row for each. The outcome of a test is not predicted: what matters is the problems the vehicle had, not how its tests ended.

**Rows go to training or test by the year of the target slot's first test.** A slot that starts in December can hold a retest from the following January, so a fold's training rows can hold a few tests from the first weeks of its test year. None of them is a test label, since each belongs to a slot that starts before the test year. We accept this. At either end of the period a slot can be cut: a slot under way on 20 May 2018 loses its tests before that date, so a retest from 20 May starts the vehicle's first slot, and a slot that starts late in 2025 loses any test in 2026, which the data does not hold.

**The record is a list of 4,821 entries, each 0 or 1, one for each defect and way of recording it, which `REPORT.md` calls a problem, and we keep only entries whose value is not fixed in advance.** The 2018 rules fix how a defect can be recorded: a dangerous or major defect fails the test, as a failure or as a failure put right at the station, and is never a minor defect; a minor defect is only ever a minor defect; an advisory is recorded only under a code that carries advisory wording; and the 16 earlier advisories only as advisories. That allows 4,961 of the 2,321 × 4 entries, and we leave the rest out. One item, in 2022, falls on a code of the 2018 rules in a way the rules exclude, and we remove it with the failure items. We also leave out the 140 allowed entries never recorded in the period, 125 of them failures put right at the station. These counts are produced by [`evidence/05_code_coverage.py`](evidence/05_code_coverage.py).

**A vehicle's class fixes which defects it can have.** No code of the 2018 rules is listed both for motorcycles, classes 1 and 2, and for another class, as [`evidence/04_lookup_groups.py`](evidence/04_lookup_groups.py) shows. An entry is therefore 0 for a vehicle whose class its defect is not listed for, and is learnt only from vehicles of the classes it is listed for: 1,251 entries for a motorcycle, 3,572 for a car or light van, class 4. An item on a code not listed for its vehicle's class is dropped.

**Each entry falls in one group, so that a defect has one group throughout the period, as it has one number.** The group is the one directly above the defect's code in `item_detail`, which is the same in every class (evidence 04). The 2,321 kept defects fall under 452 groups. A renumbered defect takes the group of the code it is numbered by, the code used first: 8 of the 36 renumbered pairs kept put their two codes under different groups, such as excessive corrosion of the structure or chassis, which moved from *Integral vehicle structure condition* to *Chassis condition*, as [`evidence/17_entry_groups.py`](evidence/17_entry_groups.py) shows. The build reads each entry's group from [`int_entry_groups`](transform/models/intermediate/int_entry_groups.sql).

**We keep each test's class as recorded, although a vehicle's class sometimes changes, because the class is important data, noisy as it is.** It is the test's own record of what was checked: it fixes which defects can be recorded, and it separates versions of one model that make and model cannot, such as a Ford Transit van, minibus or heavier van, since the files record neither weight nor seats. The tester chooses it at each test, so some changes are recorded rather than real, and the data cannot tell which. 0.33% of vehicles change class at least once, 99.5% of the changes at the next annual test and most of them between classes 4 and 7 (evidence 12).

### The vehicle columns, cleaned

The vehicle columns of each entry test are cleaned as below before they enter the model. The counts are over the normal tests of each year, aborted and abandoned ones removed, and are produced by [`evidence/07_vehicle_columns.py`](evidence/07_vehicle_columns.py) unless another file is named.

- **Fuel is recorded by name as well as by code from 2024**: beside the codes of `mdr_fuel_types`, some tests carry `HYBRID ELECTRIC (CLEAN)` or `ELECTRIC`, the names that table gives `HY` and `EL`. We map each name to its code, ignoring case, so that one fuel is one value.
- **Missing mileage is blank before 2024 and zero from 2024.** It is blank on 0.25% to 0.32% of normal tests a year to 2023, except 2.54% in 2020, and zero on 0.24% and 0.23% in 2024 and 2025. We treat both as missing. Readings taken in kilometres before 2022 cannot be told from miles, as "What the publisher says the data does and does not contain" sets out, and are kept as recorded.
- **Engine size is blank on about as many tests as are electric from 2022**, 740,156 against 735,598 in 2025, and on 0.15% to 0.25% of normal tests before 2022. It is zero on under 0.01% in every year. We treat blank and zero as missing, and leave the fuel code to say that a vehicle is electric.
- **Age is unknown where the first-use date is.** The first-use date is 1 January 1971, which the guide uses for an unknown date, on at most 0.03% of normal tests a year; it falls after the test on 6,392 in 2021 and on at most 10 in any other year; it is blank on at most 631. We treat all three as an unknown age.
- **A test with a value past one of the bounds below is removed, with its failure items, because filling the value in could compromise the training.** A row given the average in place of an extreme value, while its outcome reflects the extreme one, lowers that input's weight, and a correlated input takes up the difference; removing rows chosen by an input's value leaves the relation in the remaining rows intact. The cost is the tests removed, 601,281, 0.19% of those kept, and, for engine size and age, whole vehicles, since those values repeat on every test of a vehicle: of the 190,149 vehicles with a test past a bound, 154,825 have every test past one and leave the data, as [`evidence/13_value_bounds.py`](evidence/13_value_bounds.py) shows, with the counts for each class and bound.
- **A test whose mileage is above 900,000 is removed.** The highest reading, 999,999, is probably the most the system can record, and readings that high stop meaning anything. Mileage means the same for every class, so one bound serves them all. It is set where it drops 0.1% of the buses' mileages, class 5, whose mileages run highest: their 99.9% point is 896,760, rounded up to 900,000. A bad mileage is an accident of one test: only 625 of the 5,677 vehicles concerned have one at every test.
- **A test whose engine size lies outside its class's usual range is removed.** A size such as 1 or 99,999 cc is a mistake, and what is plausible depends on the class. For each class the range runs from the 0.1% to the 99.9% point of its sizes, rounded outward so as to remove no more than those points do, as staging reads from [`transform/seeds/engine_size_bounds.csv`](transform/seeds/engine_size_bounds.csv): 600 to 6,000 cc for class 4, for example. The size repeats at every test of a vehicle, so 75,558 of the 77,443 vehicles concerned leave the data.
- **A test at which the vehicle is over 50 years old is removed.** The earliest first-use date is 1 January of year 1, which no vehicle can have, while a vehicle of 50 can be tested. 0.08% of known ages are over 50 years, and 80,985 of the 110,463 vehicles concerned are over 50 at every test and leave the data, so the model covers no vehicle over 50.
- **An electric vehicle's missing engine size is 0**, since it has no engine. The fuel input marks every electric vehicle, so a value the same for all of them adds nothing beside it and distorts no weight. Of the 2,038,463 kept tests without an engine size, 1,905,012 are of electric vehicles, as [`evidence/15_missing_values.py`](evidence/15_missing_values.py) shows.
- **Any other missing number is dirty data, and the training row that needs it is left out of the training table**, while the test and its defects stay: a mileage or an age missing at the target slot's first test, or an engine size missing at the previous slot's first test on a vehicle that is not electric. Leaving out only the row, rather than the test, keeps the test's defects wherever they serve as the input of another row.
- **Make and model are trimmed and put in capitals**, which changes at most 0.003% of makes. `UNCLASSIFIED`, which the publisher gives a vehicle with no valid record in its vehicle dataset, is kept as a value of its own: about 0.3% of models a year.
- **A make or model ending in a backslash is kept, without the backslash.** Four rows of the 2018 to 2020 results end a value with a backslash just before the separator, such as the make `MOTOGUZZ\`. DuckDB, which the build reads with, applies the escape only inside double quotes and splits all four correctly, where Python's `csv` would join two values into one, as [`evidence/10_backslash_rows.py`](evidence/10_backslash_rows.py) shows.

### The training table: every possible row accounted for

**Every possible training row is either kept or left out by exactly one rule, each counted on its own, so that the rows the model learns from trace back to every pair of consecutive slots**, as the rows of the staged tables trace back to the files. A possible row is two consecutive slots of a vehicle, which makes as many rows as there are slots that are not a vehicle's first. The build's tests check that each staged test is in exactly one slot, with the slots split where and only where two consecutive tests are more than 20 weeks apart; that the table holds one row per vehicle and target slot; that every entry of an input or a label is one of the 4,821 and every group one of theirs; that the counts below add up to the possible rows and their kept rows to the table; and that every kept row has a mileage, age and engine size within its bounds, and a class among 1, 2, 3, 4, 5 and 7. Every test passes.

The model [`training_row_counts`](transform/models/marts/training_row_counts.sql) counts the rows per year of the target slot, as recorded in [`data/training_row_counts_2026-11-12.log`](data/training_row_counts_2026-11-12.log). A row that meets several rules is counted under the first, in the order of [`transform/macros/training_pairs.sql`](transform/macros/training_pairs.sql).

| Target year | Kept | Mileage missing at the target slot's first test | Age unknown at the target slot's first test | Engine size missing at the previous slot's first test, not electric | Possible rows |
|---|---|---|---|---|---|
| 2019 | 16,129,855 | 41,254 | 2,590 | 8,840 | 16,182,539 |
| 2020 | 27,230,182 | 67,635 | 3,163 | 12,736 | 27,313,716 |
| 2021 | 29,290,438 | 75,707 | 291 | 12,706 | 29,379,142 |
| 2022 | 30,581,637 | 77,719 | 215 | 11,917 | 30,671,488 |
| 2023 | 31,654,601 | 78,192 | 179 | 12,132 | 31,745,104 |
| 2024 | 31,942,228 | 77,179 | 156 | 10,926 | 32,030,489 |
| 2025 | 32,244,374 | 73,787 | 142 | 9,812 | 32,328,115 |
| **2019 to 2025** | **199,073,315** | **491,473** | **6,736** | **79,069** | **199,650,593** |

A further 52,824 pairs have a target slot that starts in 2018, from 20 May, and are counted under their own rule, since the first year predicted is 2019: 199,703,417 possible rows in all.

**Each vector is stored as the list of its entries that are 1**, every other entry being 0, and the groups the same way: the previous slot's entries, its groups and the label are three such lists on each row, sorted, with no repeats, and empty for a slot whose tests recorded nothing.

**The table is copied out of the build** by [`src/motrisk/export.py`](src/motrisk/export.py), as one Parquet file per target year, each holding the kept rows `training_row_counts` counts for its year, into `data/training_rows/`, which git ignores.

### The baselines

**Two simple rules, fixed and scored before any model is trained, which the model has to beat to show that it has learnt anything a rule does not already know.** Both are scored as the model is, on every test year from 2020 to 2025, by [`src/motrisk/baselines.py`](src/motrisk/baselines.py), which writes the true positives, false positives and false negatives of each baseline, test year and entry to [`data/processed/baseline_counts.csv`](data/processed/baseline_counts.csv).

- **Repeat** predicts, for each row, the entries of its previous slot, however long before: the problems recur exactly. It uses the same input as the model.
- **Most popular** predicts, for every row of a test year, the same list: the *k* entries found on the most slots starting in the year before, where *k* is the median number of entries per slot of that year. Every slot counts, a vehicle's first slot included, although it is never a row's label, and a vehicle with two slots starting in a year counts twice. A tie at the *k*-th place goes to the lower entry number. The lists are built from every slot by the dbt model [`most_popular`](transform/models/marts/most_popular.sql), since a vehicle with a single slot appears in no training row. In every year from 2018 to 2025, *k* is 1 and the list is a single entry, the tyre advisory *worn close to legal limit / worn on edge*, listed for classes 3, 4 and 7 and found on 15.3% to 16.8% of each year's slots, as [`data/processed/most_popular.csv`](data/processed/most_popular.csv) shows.

**An entry is scored only on rows of the classes it is listed for, by the baselines as by the models, so that the three are counted on the same ground.** A row's class is its previous slot's, so a defect recorded in the target slot but listed only for another class, after a change of class, is not scored for that row. For every test year and entry, each baseline's true positives plus false negatives equal the ones of the labels counted on their own, or the run stops.

### The models

**One logistic regression per entry, with an L2 penalty and $C = 1$, as large as its training rows can carry**: about ten ones per weight, so that rare entries are not fitted beyond what their data supports (Peduzzi et al., 1996). The fold of test year *Y* trains on the rows of target years 2019 to *Y* − 1, and an entry's ones and rows are counted on rows of the classes it is listed for only. In each fold an entry gets one of three models, written by [`src/motrisk/folds.py`](src/motrisk/folds.py):

- **The full model**, when it has at least 10(*d* + 1) ones, where *d* is the number of inputs of the fold: one for each of the 4,821 entries of the previous slot, each of its 452 groups, each entry crossed with each band of the gap (below), and each class, fuel and make-and-model pair found in the fold's training rows, then mileage, age and engine size, each standardised with the mean and deviation of the fold's training rows, and one input per band of the gap. A pair, fuel or class not found in the training rows has no input, so a test row carrying one has none of them set. *d* grows from 69,756 in the fold of 2020 to 114,060 in that of 2025.
- **The small model**, when it has at least 150 ones, ten for each of its 15 weights: whether the previous slot recorded the entry, whether it recorded any entry of its group, the bands of the gap, and the bands crossed with each of the two, so that each kind of vehicle has a curve of its own.
- **The base rate** below 150 ones: the share of its training rows that hold the entry, the same for every row, which is what a model with an intercept alone gives; 0 when no training row holds it.

**The gap between the two slots enters as bands, each kind of vehicle with a curve of its own across them, because the share of vehicles with a problem rises to a peak at about a year after it was last recorded and falls after it, while it keeps rising for vehicles that never had it.** Any one curve shared by every kind of vehicle is set by those that never had the problem, by far the most numerous, and over-predicts at long gaps for those that had it. Pooled over every entry in the training rows of the fold of 2025, the share holding the entry is 9.19%, 18.10%, 14.21%, 12.81% and 10.84% for vehicles whose previous slot had it, from under 300 days to over 3 years, and 0.021%, 0.041%, 0.053%, 0.066% and 0.068% for vehicles that had neither it nor anything of its group, as [`evidence/18_gap_by_kind.py`](evidence/18_gap_by_kind.py) shows. The bands are under 300 days, 300 to 430, about a year and the reference, 431 to 730, 731 to 1,095, and over 1,095: 4.74%, 86.48%, 8.06%, 0.55% and 0.18% of the fold's training rows. For entry 2546, among vehicles that had it before, the gap as one straight line predicts 19.8% to 66.4% from the shortest band to the longest against 15.1% to 13.6% observed, while bands with a curve per kind give the observed shares (evidence 18). In the small models the bands are crossed with the entry and with its group; in the full models, with each entry of the previous slot. The models fitted with the gap as one straight line are kept, for comparison, in `data/processed/train_straight_line/`.

**The small models are fitted on counts, which gives the same weights as every row at a fraction of the cost.** Rows with the same three facts, the entry before, its group before and the gap in days, and the same label add the same term to the loss, so [`src/motrisk/small_models.py`](src/motrisk/small_models.py) counts the rows of each combination once per target year and fits each entry on its counts weighted by them; the same counts score the test year, since a small model's prediction depends on those three facts alone. On the first fold, three entries chosen to cover the cases, the one whose rows most often have its group without itself, one listed only for motorcycles and the one with the fewest ones, get the same weights to six decimals fitted on counts and on every row, as [`data/check_counts_run_2026-11-13.log`](data/check_counts_run_2026-11-13.log) shows.

**The small models are run to a tight tolerance, $10^{-10}$, because scikit-learn's default stops a fit before the weights of rare inputs settle**: it divides the loss by the rows' total weight, so the slope in the weights of inputs few rows carry falls below its default threshold, $10^{-4}$, early. Fitted to the default, the small model of entry 2546 with a curve per kind gives 26.0% for vehicles that had it over 3 years before, against 13.6% observed; to $10^{-10}$, 13.7% (evidence 18).

**The full models' inputs are a sparse matrix of 32-bit floats**, by [`src/motrisk/full_models.py`](src/motrisk/full_models.py): the fold of 2025 sets 1,579,006,374 values, about 13 GB in that form against twice as much in 64-bit floats, and under the 2,147,483,647 that 32-bit offsets can address. An entry's model is trained on the rows of its classes by giving the other rows a weight of 0, which leaves the fit as if they were not there without copying the matrix. Both are choices of memory and time, and change no model. The full models' weights, 150 MB, are not in the repository; the figures computed from them, by `rescore.py` and evidence 21 and 22, are.

**The full models are fitted at scikit-learn's default tolerance, $10^{-4}$, each fold starting from the weights of the fold before**, because:
- fitted to $10^{-5}$, the folds of 2022 to 2025 were projected to take 5 to 6 days, against 6.4 days left before the Azure credit was to end on 21 November 2026;
- a larger machine was not available: the subscription allows 4 vCPUs in the region, all in use;
- starting from the previous fold's weights saved about a third of the iterations for the entry with the most flags, *Rigid brake pipes*, and little for the others, so it shortens the run without changing what a fit converges to;
- every fold is fitted alike, so the folds compare with one another.

**The gap from a tighter fit is small in the early folds.** The folds of 2020 and 2021 were also fitted to $10^{-5}$, 23 full entries in all: at $p_{\text{crit}} = 0.5$ over their full entries, precision is 46.61% against 46.36% in 2020 and 47.08% against 47.40% in 2021, and recall 2.29% against 2.33% and 2.34% against 2.38%, for about a tenth of the iterations. The models retrained on shuffled labels flag nothing at the default, and 307 and 23 cells at $10^{-5}$, as [`evidence/20_full_model_tolerance.py`](evidence/20_full_model_tolerance.py) shows; their results are in `data/processed/train_tol1e-5/`. From the fold of 2023 the fits stop much sooner (under "The second model" below).

**The shuffled-label check retrains every model on labels permuted among the rows of each target year and class**, by an order fixed by a hash of each row's key, so that the permutation is the same on every machine and every run. Each entry keeps its ones in every year and class, and so its model's tier, while the pairing of inputs and labels is broken: in each year, the share of rows that keep their own label is within 0.0001 of the share for two rows drawn at random within a class, 15.16% against 15.15% in 2019, nearly all of them two empty labels ([`data/small_counts_run_2026-11-13.log`](data/small_counts_run_2026-11-13.log)). Its scores are taken on the true labels of the test year, as the model's are.

### How the models are scored

**Every model is scored as the baselines are**: true positives, false positives and false negatives at $p_{\text{crit}} = 0.5$ for each entry and test year, on rows of its classes, and the rows and ones in each band of probability 0.01 wide, from which precision and recall follow at any $p_{\text{crit}}$ in steps of 0.01. A run stops unless each entry's ones in the training years are those counted from the rows, its true positives plus false negatives are the baselines', and its bands agree with its counts at $p_{\text{crit}}$. Each run's counts, bands and weights are in `data/processed/train/` for the first model and `data/processed/train_pooled/` for the second, the full models' weights excepted.

**We report the money saved, with precision and recall beside it**, because:
- a fleet manager decides in money, and it gives one number for each cut-off;
- precision and recall at a cut-off judge the decision rather than the probabilities, and on their own stay low where checks pay, which reads as a failure.

The **money saved** at $p_{\text{crit}} = t/m$ is the share of what leaving every problem to the test costs that checking the flags saves, $(m\,TP - t\,(TP + FP))/(m\,(TP + FN))$, which is $\text{recall}\,(1 - p_{\text{crit}}/\text{precision})$ and never exceeds recall. It is the net benefit of decision curves (Vickers and Elkin, 2006) multiplied by $1 - p_{\text{crit}}$ and divided by the share of cells that hold a problem. [`src/motrisk/evaluate.py`](src/motrisk/evaluate.py) computes it.

**We judge the probabilities with the Brier skill**, $1 - \text{BS}/\text{BS}_{\text{base}}$, where BS is the Brier score, the mean of $(p - y)^2$ over every cell, and the base gives each entry its share of ones in the training rows, because:
- the Brier score is lowest on average when the probabilities are true;
- it is twice the money lost to wrong flags and missed problems against knowing the outcome, averaged over every cut-off from 0 to 1, in units of a missed problem's cost;
- its lowest value is the noise the inputs leave, not 0, so it is read against the base rate.

A model is **still predictive** when its skill is above 0 in every test year, and **improving** when its skill rises across them.

**The Brier score is computed from the weights each run saved, without fitting anything again**, because:
- it needs each entry's sums of $p$, of $p^2$ and of $p$ over the cells that hold the entry, which the bands 0.01 wide do not keep;
- the band under 0.01 holds 99% of the cells, and its midpoint, 0.005, is about thirty times the rate observed in it, 0.00017 in 2025, so reliability and resolution read from those bands would measure the band's width rather than the model;
- every fold's weights were saved, so each model's probabilities can be recomputed as its run computed them.

[`src/motrisk/rescore.py`](src/motrisk/rescore.py) recomputes them on each test year's rows and keeps, for each entry, its rows, its ones and the three sums in each of 140 fine bands: one under $10^{-6}$, ten a decade from $10^{-6}$ to 0.01, and the run's bands 0.01 wide above, so that the fine bands add up to the run's. It stops unless each entry's rows and ones are its run's, and unless at most one row in a million is in another band than in its run; in every fold of both models, no row is. Beside them it writes the base rate each model is read against: each entry's rows and ones in the fold's training rows. The sums' last digits, about $10^{-14}$ of their value, differ from one run to the next, since DuckDB returns the rows in a different order each time; rows and ones are exact, and no figure reported moves.

### The second model

**A second model gives a column only to the makes and models common enough to learn a weight from, and is compared with the first, which is kept**. A make and model with at least 10,000 rows in the fold's training rows keeps a column of its own; the rarer models of a make share one column, when they add up to 10,000 rows; every other make and model shares one last column; and a make and model found only in the test year goes where a rare model of its make would, by [`src/motrisk/folds.py`](src/motrisk/folds.py). Nothing else changes, and both models are scored alike; its small and base-rate models read no make or model, so for them only which entries they cover changes. In the fold of 2025 this leaves 779 make-and-model columns in place of 89,476: 701 of their own, with 95.1% of the training rows, 77 makes' shared columns with 4.5%, and the last column with 0.3%; *d* falls from 114,060 to 25,363, as [`evidence/21_make_model_columns.py`](evidence/21_make_model_columns.py) shows. Because:
- most makes and models are too rare to learn a weight from: in the fold of 2025, the median one has 5 training rows, and 80,435 of the 89,476 have fewer than 100;
- their weights do almost nothing: 98.9% of the first model's make-and-model weights in the fold of 2025 are under 0.01 in absolute value;
- they decide which entries get the full model: they are 78% of *d*, so they set the threshold of 10(*d* + 1) ones that left 64 entries with a full model in the fold of 2025, holding 72.0% of the training ones, while with 25,363 inputs 150 clear it, holding 88.1%; from the fold of 2020 to that of 2025, 23, 70, 101, 121, 130 and 150 entries get the full model, against 6, 17, 27, 41, 52 and 64 in the first model (evidence 21);
- 10,000 rows hold about ten ones of the rarest entry with a full model, whose ones are about 0.15% of the rows, so each column's weight meets the rule of about ten ones per weight;
- pooling keeps one column per group of makes and models, rather than a single number per make and model (below).

**The second model is better than the first only if, in every test year, its Brier skill is higher, its money saved is higher at $p_{\text{crit}} = 0.1$ and at 0.25, and its shuffled-label check passes; otherwise the first is kept.** The rule was fixed before either model's Brier skill or the second model's money saved was computed, because:
- a challenger has to beat the model in place, not tie it;
- at 0.5 both models lose a fraction of a percent, so a comparison there would be decided by noise; 0.5 is reported, not compared.

**From the fold of 2023, the full fits stop short, and that costs the entries the second model adds part of their skill.** The entries with a full model in the second model and a small one in the first, 17 in the fold of 2020 and 86 in that of 2025, score a Brier skill of 0.0455 as full models against 0.0377 as small ones in 2020, ahead to 2022, and 0.0196 against 0.0228 in 2025, behind from 2023. The median iterations of their full fits fall from 43 in 2020 to 2 in 2025, and those of the first model's full fits from 92 to 5.5, 32 of 64 with 5 or fewer: each fold starts close to the weights of the fold before, and at the default tolerance a fit stops almost at once. We fitted two entries of the fold of 2025 further, from the weights the run saved, in 64-bit floats so that rounding cannot be what stops a fit: entry 2475, which loses the most skill, and entry 2546, the one followed above. We tested two causes:
- **the fits stopped too early**: 300 more iterations at the run's penalty raise the skill from 0.0699 to 0.0762 and from 0.0453 to 0.0505, about half and over a third of the way to the small models' 0.0830 and 0.0592, and no further after 200, though no fit converged;
- **too many weights for too few cases**: with the penalty 10 to 10,000 times the run's, the skill is at best 0.0768 and 0.0508, and falls at 10,000 times, short of the gain of 0.002 set before the test as the sign of this cause.

**We leave the rest of the gap untested**, because:
- the next suspect needs a model of its own: the full model has a column for the entry's group, but not the small model's curve over the gap for vehicles with something of the group and not the entry itself;
- which model is kept is decided by the comparison, not by why the second scores lower.

The tests are [`evidence/22_tight_refit.py`](evidence/22_tight_refit.py), and their results are in `data/processed/train_pooled/tight_2025_fine.csv`.

### What we do not do

**We do not over-sample the rare problems, with SMOTE or otherwise**, because:
- in a logistic regression, adding positive rows moves mainly the intercept, raising every probability, while the weights that rank one vehicle above another stay nearly the same (King and Zeng, 2001);
- calibration would then bring the probabilities back down to their true, small values, so the model would end where it started;
- the fleet manager's rule, to check a vehicle for a problem when $p \ge t/m$, needs probabilities that mean what they say, which over-sampling breaks until it is undone;
- the problems it could apply to, those with a full model, have hundreds of thousands of cases each, while the rare ones have a small model on 0/1 inputs or a base rate, with nothing to interpolate.

**We do not reduce the inputs with principal component analysis (PCA)**, because:
- on the make-and-model columns, its first components come out close to the most frequent makes and models, which already have a column of their own;
- it chooses its directions without looking at the label, so it can drop a direction that predicts a problem;
- each weight would belong to a mix of thousands of inputs, and could no longer be read as the effect of one defect or one make and model;
- it would turn the sparse inputs, about 1.6 billion values in the fold of 2025, into a dense matrix too large to hold.

**We do not give each make and model a number in a single column (label encoding)**, because:
- a logistic regression multiplies that number by one weight, so it would assume an order and a scale among makes and models, those of their numbering, alphabetical or any other, which no real effect follows;
- it suits tree models, which split a number into ranges, not a linear model;
- it would not make the inputs less sparse in any way that matters: each row sets one make-and-model value under either encoding, and what made the first model costly was its columns too rare to learn a weight from, which pooling removes;
- every category column enters one-hot, so that no order among values is assumed.

## The runs

Where and when each step ran, and its record. The steps in Azure ran in the Azure Function or on one virtual machine, small until the conversion to Parquet and resized to 4 vCPUs and 32 GB for the staging build and every step after it; every other step ran on our computer, with 7 GB. Every resource in Azure was deleted on 18 November 2026, once its results were copied to our computer ([`data/azure_deletion_run_2026-11-18.log`](data/azure_deletion_run_2026-11-18.log)).

| Step | When, 2026 | Where | Record |
|---|---|---|---|
| Download the 17 files | 31 October to 4 November | our computer | [`data/fetch_log.tsv`](data/fetch_log.tsv) |
| Land the 17 files in Azure | 7 November | Azure | [`fetch_run_2026-11-07`](data/fetch_run_2026-11-07.log); checked on 18 November by evidence 08, [`azure_copy_run_2026-11-18`](data/azure_copy_run_2026-11-18.log) |
| The watcher, daily | 7 to 18 November | Azure Function | [`watcher_runs_2026-11-18`](data/watcher_runs_2026-11-18.log), read from the Function's record by [`azure/watcher_runs.sh`](azure/watcher_runs.sh) |
| Extract the CSVs | 10 November | Azure; our computer | [`extract_run_2026-11-10`](data/extract_run_2026-11-10.log), out of memory at the first Deflate64 archive, and [`extract_run_2026-11-10_resumed`](data/extract_run_2026-11-10_resumed.log) for the five left once `archives.py` read it in constant memory; [`extract_local_run_2026-11-10`](data/extract_local_run_2026-11-10.log) |
| Read the CSVs, evidence 09 | 10 November | Azure | [`csv_from_azure_run_2026-11-10`](data/csv_from_azure_run_2026-11-10.log) |
| Convert to Parquet | 12 November | Azure; our computer | [`convert_run_2026-11-12`](data/convert_run_2026-11-12.log), out of memory five files in, and [`convert_run_2026-11-12_resumed`](data/convert_run_2026-11-12_resumed.log), with a swap file; [`convert_local_run_2026-11-12`](data/convert_local_run_2026-11-12.log); checked by evidence 11, [`parquet_in_azure_run_2026-11-12`](data/parquet_in_azure_run_2026-11-12.log) |
| Staging build | 12 November | Azure | [`dbt_build_run_2026-11-12`](data/dbt_build_run_2026-11-12.log), which filled the disk running four steps at once; [`dbt_retry_run_2026-11-12`](data/dbt_retry_run_2026-11-12.log), one step at a time; [`dbt_items_run_2026-11-12`](data/dbt_items_run_2026-11-12.log), the failure items rebuilt with the rule for items whose test is in no file |
| Training table, 35 minutes | 12 November | Azure | [`dbt_mart_run_2026-11-12`](data/dbt_mart_run_2026-11-12.log) |
| Export of the training table and Most popular | 13 November | Azure, then copied to our computer, each file's SHA-256 matching | [`training_rows_export_run_2026-11-13`](data/training_rows_export_run_2026-11-13.log), [`dbt_most_popular_run_2026-11-13`](data/dbt_most_popular_run_2026-11-13.log) |
| Baselines | 13 November | our computer | [`baselines_run_2026-11-13`](data/baselines_run_2026-11-13.log) |
| Small and base-rate models: counts, check, fits | 13 November, 26 and 24 minutes | our computer | [`small_counts_run_2026-11-13`](data/small_counts_run_2026-11-13.log), [`check_counts_run_2026-11-13`](data/check_counts_run_2026-11-13.log), [`small_models_run_2026-11-13`](data/small_models_run_2026-11-13.log) |
| The gap as one straight line, kept for comparison | 13 November | our computer; Azure | [`small_models_straight_line_run_2026-11-13`](data/small_models_straight_line_run_2026-11-13.log), [`check_counts_straight_line_run_2026-11-13`](data/check_counts_straight_line_run_2026-11-13.log), [`full_models_straight_line_run_2026-11-13`](data/full_models_straight_line_run_2026-11-13.log), [`full_memory_straight_line_run_2026-11-13`](data/full_memory_straight_line_run_2026-11-13.log) |
| Measuring the full models' memory, precision and tolerance | 13 and 14 November | Azure | [`full_measure_run_2026-11-13`](data/full_measure_run_2026-11-13.log), [`full_measure_dtype_run_2026-11-13`](data/full_measure_dtype_run_2026-11-13.log), [`full_measure_tol_run_2026-11-13`](data/full_measure_tol_run_2026-11-13.log), [`full_measure_bands_run_2026-11-13`](data/full_measure_bands_run_2026-11-13.log), [`full_measure_default_run_2026-11-14`](data/full_measure_default_run_2026-11-14.log) |
| Full models of 2020 and 2021 at $10^{-5}$ | 14 November | Azure | [`full_models_tol1e-5_run_2026-11-14`](data/full_models_tol1e-5_run_2026-11-14.log), [`full_memory_tol1e-5_run_2026-11-14`](data/full_memory_tol1e-5_run_2026-11-14.log); compared by evidence 20, [`full_model_tolerance_run_2026-11-16`](data/full_model_tolerance_run_2026-11-16.log) |
| Full models | 14 to 16 November | Azure | [`full_models_run_2026-11-14`](data/full_models_run_2026-11-14.log), stopped when DuckDB's spill filled the disk and resumed from the folds already written; [`full_memory_run_2026-11-14`](data/full_memory_run_2026-11-14.log) |
| Evaluation | 16 November | our computer | [`evaluate_run_2026-11-16`](data/evaluate_run_2026-11-16.log) |
| Second model: small and base-rate models, 17 minutes, and full models | 17 and 18 November | our computer; Azure | [`small_models_pooled_run_2026-11-17`](data/small_models_pooled_run_2026-11-17.log), [`full_models_pooled_run_2026-11-17`](data/full_models_pooled_run_2026-11-17.log), [`full_memory_pooled_run_2026-11-17`](data/full_memory_pooled_run_2026-11-17.log); its columns by evidence 21, [`make_model_columns_run_2026-11-17`](data/make_model_columns_run_2026-11-17.log) |
| Brier score, both models | 18 November | our computer for the small models; Azure for the full ones, which need 9 to 11 GB a test year | [`rescore_small_run_2026-11-18`](data/rescore_small_run_2026-11-18.log), [`rescore_full_run_2026-11-18`](data/rescore_full_run_2026-11-18.log), [`rescore_full_memory_run_2026-11-18`](data/rescore_full_memory_run_2026-11-18.log) |
| Evaluation of both models | 18 November | our computer | [`evaluate_run_2026-11-18`](data/evaluate_run_2026-11-18.log), [`evaluate_pooled_run_2026-11-18`](data/evaluate_pooled_run_2026-11-18.log) |
| Why the second model scores lower, evidence 22 | 18 November | Azure | [`moved_entries_run_2026-11-18`](data/moved_entries_run_2026-11-18.log), [`tight_refit_run_2026-11-18`](data/tight_refit_run_2026-11-18.log), [`penalty_refit_run_2026-11-18`](data/penalty_refit_run_2026-11-18.log), [`tight_refit_score_run_2026-11-18`](data/tight_refit_score_run_2026-11-18.log), [`tight_refit_memory_run_2026-11-18`](data/tight_refit_memory_run_2026-11-18.log), [`penalty_refit_memory_run_2026-11-18`](data/penalty_refit_memory_run_2026-11-18.log) |
| Evaluation of both models again, adding the ways a problem is recorded and the slots with a failure | 19 November | our computer | [`evaluate_run_2026-11-19`](data/evaluate_run_2026-11-19.log), [`evaluate_pooled_run_2026-11-19`](data/evaluate_pooled_run_2026-11-19.log) |

The output of evidence 12 to 19 is kept in `data/` beside the runs: 12, [`class_changes_run_2026-11-12`](data/class_changes_run_2026-11-12.log); 13, [`value_bounds_run_2026-11-12`](data/value_bounds_run_2026-11-12.log); 14, [`test_gaps_run_2026-11-12`](data/test_gaps_run_2026-11-12.log); 15, [`missing_values_run_2026-11-12`](data/missing_values_run_2026-11-12.log); 16, [`same_day_tests_run_2026-11-12`](data/same_day_tests_run_2026-11-12.log); 17, [`entry_groups_run_2026-11-12`](data/entry_groups_run_2026-11-12.log); 18, [`gap_by_kind_run_2026-11-13`](data/gap_by_kind_run_2026-11-13.log); 19, [`covid_extension_run_2026-11-16`](data/covid_extension_run_2026-11-16.log).
