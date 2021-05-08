## Drip Monthly Data Tape Preparation
1.	Download  all system generated data tapes from the auto-email sent on month end. 
2.	Concatenate all the tapes together to form “Drip All”. 
3.	Move “Exporter Country” field from end of tape to between “Buyer” and “Buyer Country”.
4.	Move “Exporter ID” and “Buyer ID” fields from end of tape to between “Buyer” and “Exporter Country”
5.	Move “Buyer Grade” field from end of tape to between “Buyer Country” and “Product Exported Desc”.
6.	Move ‘Invoice Date’ field from end to between ‘Grace Period’ and ‘Anchor Event Date’
7.	Move “Other Adjustments” between “Setup Fee Charged” and “Total Fees” 
8.	Remove following unnecessary columns from “Drip All”:
    -Product Operations Category
b.	Expected Tenor including Grace Period
c.	Comments for 30+ DPD RF Invoices
d.	Comments(Other Adjustments)
e.	Set offs
f.	Set off comments
g.	Sum of daily outstanding balance
h.	Native values – Invoice value, Advance Value, Payment Received
i.	Approved LTM
j.	Weighted Average Invoice Duration
k.	Invoice IRR (Column to be added later from metabase query) 

9.	Add in following new derived columns to “Drip All”:
a.	Create In Vasco? column
i.	Between ‘Reference’ and ‘Exporter Name’ 
ii.	Fill In Vasco? by vlookup from Vasco tracker
b.	Advance Rate (Actual) = Advanced Value USD / Net Invoice Value USD 
i.	Between ‘Advanced Value USD’ and ‘Payment Received USD’ 
c.	Open or Repaid Flag 
i.	Base it on invoice stage, not outstanding balance [Categorisation – All except “Settled” and “Pending Settlement” are open] Also "Auto-Settled" -🡪[OPEN]”Auto-Settled” stage did not exist in any of the 6 raw tapes THIS IS OKAY
ii.	Between ‘LTM’ and ‘IRR’/’Yield’ *
d.	Check “Payment Received USD” for blanks and make 0
e.	The below 6 after 'Payment Received USD' : 
f.	Invoice Value Received USD = MIN(Payment Received USD, Net Invoice Value USD) 
g.	Principal/Advance Value Received USD = MIN(Payment Received USD, Advance Value USD) 
h.	Dilution on Invoice Value USD = IF(Open,0,Net Invoice Value – Invoice Value Received) 
i.	Advance Value Dilution = IF(Open,0,Advanced Value USD – Advanced Value Received)
j.	Outstanding Invoice Value USD = (Net Invoice Value USD - Invoice Value Received USD – Dilution on Invoice Value USD) 
k.	Outstanding Principal Balance USD = (Advance Value Received USD – Advanced Value USD – Dilution on Advance Balance USD) 
l.	Create Invoice Yield at the very end 
m.	Add “Modified Due Date Flag” field after “Insurable Flag” field[Added this column but not sure how it is to be populated] SAME AS 09/30 TAPE FOR NOW
n.	Add “Modified Due Date” and “New Due Date” fields after “Due Date” field [Added these two columns but not sure how they’re to be populated] SAME AS 09/30 TAPE FOR NOW

Delete repeated invoice ID 72 (Keep the one with the NAs) 
10.	VLOOKUP CORRECT BUYER NAMES FROM IDs USING this query [buyer ID #106215 from prev tape] 
11.	VLOOKUP CORRECT EXPORTER NAMES FROM IDs USING this query
12.	Make invoice ID 49 Flag “Repaid” and Stage as “Settled” 
13.	For Mx invoices, override Interest Rate and Step-up Interest Rate as: 
a.	New Interest Rate = Old Interest Rate * (Net Invoice Value USD ou / Advanced Value USD) 
b.	New Step Up Interest Rate = Old Step Up Interest Rate * (Net Invoice Value USD / Advanced Value USD) 

14.	Fill In Invoice Yield by Metabase query (effective_interest_rate) followed by vlookup and division by 100 
a.	Make Yield = NA for “Open” invoices [Few Open invoices had >0 IRR] what is the "stage"?-🡪[OPEN]stages are: Advanced/Overdue/ETD/FDA/partial adv./partial payment and the count of such is 211 Make IRR NA [Add before Open/Repaid flag” column] 
15.	For invoices in “Pending Settlement” and “Auto-Settled” stage, change NAs in Fees and Interest (using metabase query) [Did not have “Auto-Settled” & “Auto-Advanced” stage in the raw data][Payment received as in Data Tape and as from metabase query for invoices 21069 & 109800 do not match] what about "Pending Settlement" -🡪 [OPEN]Raw tapes did have ‘Pending Settlement’ as a stage, and there were 29 such invoices for which the fees/interests were populated from metabase, but total payment received for 2/29 invoices did not match with the data tape. Also, there was no “Auto-Settled”  stage THIS IS OKAY
16.	Check for cases where factoring fee rate / interest rate / step-up interest rate is non-zero but corresponding actual $ value charged is 0 and vice-versa. (keeping in mind outstanding tenor, size etc.)
a.	Change Factoring Commission of invoices 18948, 17650,13196,11300, 11299, 9300, 9298, 3620, 2398 using this query. And dividing by 100 
b.	Sometimes for Repaid invoices show an interest/factoring fee charged and then increase adjustment amount (eg: 17421, 1757) 
17.	Make Invoice ID 9610 as ‘Repaid’ (stage = “Settled”, Yields = 0%) 
18.	Correct invoice dates of following invoices by reducing by 1 month: 
a.	Test failed: Invoice Date > First Advance Date
i.	4937
ii.	4779
iii.	4378
iv.	4319
v.	3270
vi.	3257
vii.	3256
viii.	2672
ix.	1137
x.	948
1.	10793 – NO CHANGE Done [Followed the doc although the inv. Date>adv. Date]
19.	Make Total LTM as ‘NA’ where 0 
20.	Change stage from NPA to Overdue where <90 and >0 days DQ from Due Date (New) [inv. #19345, #16544, #16482, #15441, #15413, #15333 moved to “Overdue”]
21.	Change stage from NPA/Overdue to Advanced/ETD Check /GUD Check where <=0 days DQ from Due Date (New)  [7 invoices (20681, 19337, 16415, 16835, 15684, 15683, 15634), both moved to “Advanced” stage for the Anchor Event being “Invoice Date”/”BL Date”] 
a.	Do ETD Check if Anchor Event is Discharge Date and dashboard agent check date > data tape date
b.	Do GUD Check if Anchor Event is GUD and dashboard agent check date > data tape date
22.	Change Stage from “Advanced/GUD/ETD” to “Partial Payment” if Open, not DQ and Payment >0 [No change here for  the “Payment Received” was 0]
23.	Make Buyer Grade = “Pending” where Buyer Grade = 11 and “NA” where Blank 
a.	If existing old NPA buyer, then 11 might need to become NA and not “Pending” (Eg: Bambino, East Ocean, Pure Life Foods)  
24.	Make Insurable Flag Blank where IF 
25.	Override columns Total Interest + Fees (With and Without Setup Fee) with summation formulae  
a.	Factoring + Interest + Step Up Interest + Setup + Adjustments [“NA”s in “Other Adjustments” column to be replaced with?] NAs should be for open invoices, so let it be. -🡪 [OPEN] The total fees (fees+interest etc.) calculation would show value error because text+string. Should I then add an ‘IFERROR’ clause? YES, MAKE NA
b.	Make NA for Open Invoices 
26.	Make Factoring Fee as ‘NA’ for invoices in stage “Open” 
27.	Manually populate Anchor Event Date where blank [done from prev. tape]
28.	Override Full Payment Date and Actual Outstanding Tenor for following two invoices: [Created a new col. “Actual Outstanding tenor as (full payment date-first adv. Date) and made changes in it] There should be a field already. 🡪[RESOLVED] Got it – In the ‘open’ raw tapes ‘Actual outstanding tenor’ column is named as ‘Expected outstanding tenor’ whereas, it goes by the required name in the ‘repaid’ data tapes. 
a.	2364: Full Payment Date = 10/25/2018, Actual Outstanding Tenor = 51
b.	7694: Full Payment Date = 09/23/2019, Actual Outstanding Tenor = 132
29.	Override column “Step Up Interest Rate Tenor including Holiday Grace” with below formula from dictionary:
a.	“NA” for Open
b.	Max(0, Outstanding Tenor – (Due Date (Old) – First Advance Date + Grace Period)) for “Repaid” 
30.	Rename currencies as usd -> USD, euro -> EUR and so on 
31.	Make “Release Net Payment to Exporter” and “Actual Outstanding Tenor” as NA (and not ‘-‘) for all “Open” invoices. 
32.	Make “Stage” as “Principal Repaid”, and populate Principal Payment Date from dashboard for invoices where principal is fully repaid but interest + fees in pending. [found 7 such invoices]
33.	Add “Expected Tenor From Invoice Date” and “Expected Tenor From Advance Date” fields (Due Date (New) – Invoice Date / First Advance Date ) 
34.	In the “Credit Limit” sheet: 
a.	Keep only “Insured Limit”
b.	Delete IF (Delete rows where “Insured Limit” is blank
c.	Add in Buyer ID column 
d.	Remove “Andre Prost Inc” and “Hathi Foods” (UAE invoice buyers) 
e.	Correct Sl Nos by dragging and autofill 

35.	Run the following tests on the data: 
a.	Ranges of all columns
b.	Nulls and negatives (refer to  the google sheet)
c.	Date chronology 
d.	Nothing in the future
e.	Total count of invoices, Mx/Ind split, RF/IF split with dashboard/metabase
f.	Anchor Event Dates when Anchor Event is BL/Invoice Date
g.	Fees/Interest, Yield, Outstanding Balance, Stage when invoice is “Open”
h.	Quick cross checks with past formatted data tapes to see increase/decrease etc. 

36.	Change formatting by copying pasting into latest “Formatted Tape Skeleton” file 

37.	In the “Cashflow Statement” sheet, add in:

a.	Cashflow USD = Cashflow*Conversion
b.	Cashflow Month = EOMONTH(Cashflow Date,-1)+1
c.	Origination Month = EOMONTH(First Invoice Advanced Date,-1)+1





