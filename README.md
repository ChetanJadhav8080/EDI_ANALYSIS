# EDI_ANALYSIS
**A Live Dashboard connected to SAP BW Source of 48 different plants, in which  Demands, capacity, and sales analysis is done to meet customer demand and achieve its sales goals. It involves analyzing historical sales data, forecasting future demand, and assessing the business's production capacity.**






**SQL QUERY USED TO PULL DATA FROM SAP BW**
= SapHana.Database("s700s044.autoexpr.com:30241", [Query="select CAST(edi.""/BIC/ZMF_RLDAT"" AS DATE) AS EDI_DD,#(lf)(Week(CAST(edi.""/BIC/ZMF_RLDAT"" AS DATE))) AS EDI_week,#(lf)Year(CAST(edi.""/BIC/ZMF_RLDAT"" AS DATE)) AS EDI_year ,#(lf)CAST(Year(CAST(edi.""/BIC/ZMF_RLDAT"" AS DATE)) AS INT)*100 + CAST((Week(CAST(edi.""/BIC/ZMF_RLDAT"" AS DATE))-1) AS INT) as EDI_sum,#(lf)(concat((CAST(Year(CAST(edi.""/BIC/ZMF_RLDAT"" AS DATE)) AS varchar)),(CAST((Week(CAST(edi.""/BIC/ZMF_RLDAT"" AS DATE))-1) AS varchar))))as EDI_Calendarweek,#(lf)concat(edi.material,edi.SHIP_TO) as ediflag,#(lf)edi.""/BIC/ZMF_BTHDA"",#(lf)edi.PLANT,#(lf)edi.MATERIAL,#(lf)edi.""/BIC/ZMF_RLDAT"",#(lf)edi.CUSTOMER,#(lf)edi.SHIP_TO,#(lf)edi.""/BIC/ZMF_RLQTY"",#(lf)edi.""/BIC/ZMF_SCTYP"",#(lf)edi.""CALWEEK"",#(lf)edi.""/BIC/ZWEEKDAY""#(lf)#(lf)#(lf)from ""SAPSR3"".""/BIC/AZMFD02EUN6"" edi#(lf)#(lf)WHERE edi.PLANT = 2152  AND CAST(edi.""/BIC/ZMF_BTHDA"" AS DATE)>= ADD_DAYS(CURRENT_DATE,-100) ", Implementation="2.0"])





**M QUERY USED TO CREATE CUSTOM CALENDAR TABLE**
let
    Source = Table.FromRows(Json.Document(Binary.Decompress(Binary.FromText("i45WMtQ31DcyMDJU0oExTZViYwE=", BinaryEncoding.Base64), Compression.Deflate)), let _t = ((type nullable text) meta [Serialized.Text = true]) in type table [StartDate = _t, EndDate = _t]),
    #"Changed Type" = Table.TransformColumnTypes(Source,{{"StartDate", type date}, {"EndDate", type date}}),
    #"Added Custom" = Table.AddColumn(#"Changed Type", "Dates", each {Number.From([StartDate])..Number.From([EndDate])}),
    #"Expanded Date" = Table.ExpandListColumn(#"Added Custom", "Dates"),
    #"Changed Type1" = Table.TransformColumnTypes(#"Expanded Date",{{"Dates", type date}}),
    #"Removed Other Columns" = Table.SelectColumns(#"Changed Type1",{"Dates"}),
    #"Added Custom1" = Table.AddColumn(#"Removed Other Columns", "Year", each Date.Year([Dates])),
    #"Added Custom2" = Table.AddColumn(#"Added Custom1", "Month", each Date.Month([Dates])),
    #"Duplicated Column" = Table.DuplicateColumn(#"Added Custom2", "Dates", "Dates - Copy"),
    #"Calculated Week of Year" = Table.TransformColumns(#"Duplicated Column",{{"Dates - Copy", Date.WeekOfYear, Int64.Type}}),
    #"Added Custom3" = Table.AddColumn(#"Calculated Week of Year", "Week Num", each [#"Dates - Copy"]-1),
    #"Replaced Value" = Table.ReplaceValue(#"Added Custom3",0,1,Replacer.ReplaceValue,{"Week Num"}),
    #"Removed Other Columns1" = Table.SelectColumns(#"Replaced Value",{"Dates", "Year", "Week Num"}),
    #"Added Custom4" = Table.AddColumn(#"Removed Other Columns1", "Calendar weeknum", each [Year]*100 + [Week Num]),
    #"Inserted Merged Column" = Table.AddColumn(#"Added Custom4", "Merged", each Text.Combine({"W", Text.From([Week Num], "en-US")})),
    #"Inserted Last Characters" = Table.AddColumn(#"Inserted Merged Column", "Last Characters", each Text.End(Text.From([Year], "en-US"), 2), type text),
    #"Inserted Merged Column1" = Table.AddColumn(#"Inserted Last Characters", "YearC_WK", each Text.Combine({[Last Characters], "-", [Merged]}), type text)
in
    #"Inserted Merged Column1"


