# ct_infos
Information file about dose report for CT scanners. 

This a data file for [YADER](https://maryknollrad.github.io/yader).

## How to add a new entry for CT scanner
1. Check DICOM header information of CT examinations with your PACS viewer for Manufacturer (0008,0070) and Manfacturer's Model Name (0008, 1090) fields
1. Find out which is the dose report series. YADER supports 3 methods for series selection.
    1. Series number : each series have associated number, dose report may have specific number
    1. Series position : 0 to (number of series - 1), -1 for last one
    1. Series name : part of series name, YADER selects first series that contains specified string value.
1. Write Regular expression for the target positions of dose value.  
    for further information about regular expression, see [Oracle's page about Pattern](https://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html)
1. Add 3 lines to ct-info.conf
    1. Manufacturer.Model.dose-series-by : one of "number", "index", or "name"
    1. Manufacturer.Model.dose-series-value : target value
    1. Manufacturer.Model.dose-extraction-reg : regular expression

## Example
* Scanners from Toshiba use series number 9000 for dose report series
* Dose values are located at either #P1 or #P2 within following text   
`Total DLP(mGy.cm): (Head): #P1  (Body): #P2`
    * Possible target positions are marked with capturing groups, using parentheses
    * Need to escape actual parenthesis using backslash, `\\(` or `\\)`
    * Various numbers of spaces can be expressed by `\\s*`
    * #P1 and #P2 may be either `-` or float number, so `[-.0-9]+` is used for capturing both values
    * "(mGy.cm)" may be misrecognized by OCR, so replaced with various number of characters, `.+`
    * Final regular expression is `Total DLP.+\\(Head\\):\\s*([-.0-9]+)\\s*\\(Body\\):\\s*([-.0-9]+)`
* We can add following 3 lines within `CTINFO {}` as follows.
```
CTINFO {
    TOSHIBA."AQUILION PRIME".dose-series-by : "number" # number, index, name
    TOSHIBA."AQUILION PRIME".dose-series-value : 9000
    TOSHIBA."AQUILION PRIME".dose-extraction-reg : "Total DLP.+\\(Head\\):\\s*([-.0-9]+)\\s*\\(Body\\):\\s*([-.0-9]+)"
}
```