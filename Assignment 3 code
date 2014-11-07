/*********************************
Gilbert Duenas
Assignment Number 3
*********************************/

/*
1-17 – Include your corrected code for Assignments 1 & 2.
*/
proc format;
	value sex 	1 = 'Female'
				2 = 'Male';
	value race	1 = 'Asian'
				2 = 'Black'
				3 = 'Caucasian'
				4 = 'Other';
run;

data STUDY;
	infile '/courses/dc4508e5ba27fe300/c_629/suppTRP-1062.txt' dsd missover;
	input Site : $1.
		Pt : $2.
		Sex : 8.
		Race : 8.
		Dosedate : mmddyy10.
		Height : 8.
		Weight : 8.
		Result1 : 8.
		Result2 : 8.
		Result3 : 8.;

	format Dosedate mmddyy10.
		Sex sex.
		Race race.;

	label	Site	= 'Study Site'
		Pt 			= 'Patient'
		Dosedate 	= 'Dose Date'
		Doselot 	= 'Dose Lot'
		prot_amend 	= 'Protocol Amendment'
		Limit 		= 'Lower Limit of Detection'
		site_name 	= 'Site Name';

	length Doselot $5;
	if Dosedate ge '1JAN1997'd and (Dosedate le '31DEC1997'd) then Doselot = 'S0576';
	else if Dosedate ge '1JAN1998'd and (Dosedate le '10JAN1998'd) then Doselot = 'P1122';
	else if Dosedate ge '11JAN1998'd and (Dosedate le '31DEC1998'd) then Doselot = 'P0526';
	else Doselot = ' ';

	length prot_amend $1;
	format Limit 3.2;

	if Doselot = 'P0526' then do;
		prot_amend = 'B';
		if Sex = 1 then Limit = 0.03;
		if Sex = 2 then Limit = 0.02;
	end;
	else if Doselot = 'S0576' or Doselot = 'P1122' then do;
		prot_amend = 'A';
		Limit = 0.02;
	end;
	else if missing(Doselot) then do;
		prot_amend = ' ';
		Limit = ' ';
	end;

	length site_name $26;
	select(Site);
		when('J') site_name = 'Aurora Health Associates';
		when('Q') site_name = 'Omaha Medical Center';
		when('R') site_name = 'Sherwin Heights Healthcare';
		otherwise;
	end;
run;

libname tempData "/courses/dc4508e5ba27fe300/c_629/saslib" access=readonly;

proc sort data=tempData.DEMOG1062 out=DEMOG1062new;
	by pt site; 
run;

proc sort data=STUDY out=STUDYnew;
	by pt site;
run;

data PAT_INFO;
	merge DEMOG1062new STUDYnew;
	by pt site;

	pt_id = Site || '-' || Pt;
	if missing(site) or missing(pt) then pt_id = ' ';
	label pt_id = 'Site-Patient';

	dose_qtr = 'Q' || put(qtr(Dosedate), z1.);
	if missing(Dosedate) then dose_qtr = ' ';

	if n(of result1-result3) then mean_result = mean(of result1-result3);
	format mean_result 3.2;

	BMI = Weight / (Height ** 2) * 703;
	if missing(Weight) or missing(Height) then BMI = ' ';
	format BMI 4.1;

	if prot_amend = 'A' then do;
		est_end = Dosedate + 120;
	end;
	else if prot_amend = 'B' then do;
		est_end = Dosedate + 90;
	end;
	else if missing(prot_amend) then do;
		est_end = ' ';
	end;
	format est_end mmddyy10.;
	label est_end = 'Estimated Termination Date';
run;

options nocenter;

proc sort data=pat_info out=pat_infoPrint;
	by site site_name;
run;

proc print data=pat_infoPrint label double;
	title 'Listing of Baseline Patient Information for Patients Having Weight > 250';
	where weight gt 250;
	by site site_name;
	id site site_name;
	var pt age Sex race height Weight Dosedate Doselot;
	label age = 'Age'
		dosedate = 'Date of First Dose'
		doselot = 'Dose Lot Number';
	format Dosedate mmddyy8.;
run;

title 'Use the data set PAT_INFO and one PROC MEANS';

proc sort data=pat_info out=pat_infoMeans;
	by sex;
run;

proc means data=pat_infoMeans n mean std min max maxdec=1 median;
	class sex;
	var result1 result2 result3 height weight;
	output out=medianWeight
		median(weight)= median_weight;
run;

title '15 Combine data sets and create wt_cat';

proc format;
	value Wtcat 1 = '<= Median Weight'
				2 = '> Median Weight';
run;

proc sort data=medianWeight;
	by sex;
run;

data mergeSex;
	merge pat_infoMeans medianWeight;
	by sex;
	if weight le median_weight then wt_cat=1;
	if weight gt median_weight then wt_cat=2;
	format wt_cat Wtcat.;
	label wt_cat='Median Weight Category';
run;

title '16 Using your data set from Item 15 and one PROC FREQ to do the following';

proc format;
	value raceGroup
		3 = 'White'
		other = 'Other';
	value weightGroup
		low-<200 = '< 200'
		200-<300 = '200 to < 300'
		300-high = '>= 300'
		. = 'Missing';
run;

proc freq data=mergeSex;
	tables doselot wt_cat / nocum;
	tables race * weight / nocum missing;
	format race raceGroup.
		weight weightGroup.;
run;

title '17 Using your data set from Item 15 and one PROC UNIVARIATE to do the following';

proc univariate data=mergeSex;
	class wt_cat;
	var height;
	id pt_id;
run;

/*
18 – Create this summary table using a single PROC REPORT. Do not use a data step or any other procedures (PROCs) for
this item.
*/
options missing='';

title 'Summary of Mean Analyte Results by Weight Category and Sex';

proc report data=mergeSex nowd headline;
	column wt_cat sex (site_name,(result1 result2 result3)); 
	define wt_cat		/ group left 'Weight Category' width=15;
	define sex			/ group left width=10;	
	
	define site_name	/ across '- Site -';
	define result1		/ analysis mean 'Mean Result1' width=8 format=4.3;
	define result2		/ analysis mean 'Mean Result2' width=8 format=4.3;
	define result3		/ analysis mean 'Mean Result3' width=8 format=4.3;
	break after wt_cat	/ skip;
run;

/*
19 – Create this listing using a single PROC REPORT. Do not use a data step or any other procedures (PROCs) for this item.

Use the following information to create the columns BMI Category and Absolute Change.
BMI Category

< 18.5 Underweight
18.5 to < 25 Normal
25 to < 30 Overweight
30 or more Obese

Absolute Change is the difference between Analyte Result 2 and Analyte Result 1.
*/
proc report data=mergeSex nowd headskip;
	column site_name pt_id dosedate age sex race wt_cat BMI bmiCat result1 result2 absChange;
	define site_name/ group noprint;
	define pt_id 	/ order 'Patient' width=7;
	define sex		/ left width=6;
	define age		/ left 'Age' width=3;
	define race		/ left width=9;
	define wt_cat	/ 'Weight Category' width=15;
	define BMI		/ display width=3;
	define bmiCat	/ computed 'BMI Category' width=12;
	compute bmiCat	/ character length=12;
		if BMI ge 30 then bmiCat = 'Obese';
		else if BMI ge 25 then bmiCat = 'Overweight';
		else if BMI ge 18.5 then bmiCat = 'Normal';
		else if not missing(BMI) then bmiCat = 'Underweight';
	endcomp;
		
	define result1	/ display 'Analyte Result 1' width=8;
	define result2	/ display 'Analyte Result 2' width=8;
	define absChange / computed 'Absolute Change' width=8 format=4.1;
	compute absChange;
		absChange = result2 - result1;
	endcomp;
	title 'Listing of Baseline Patient Characteristics ';
	break after site_name	/ skip;
run;
