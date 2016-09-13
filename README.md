Skip to content
This repository
Search
Pull requests
Issues
Gist
 @ishanchaturvedi
Configure email settings 
Please verify your email address to access all of GitHub's features.
An email containing verification instructions was sent to ish.chat85@gmail.com.
 Watch 1
  Star 0
  Fork 0 ishchat/technicalanalysislibrary
 Code  Issues 0  Pull requests 0  Wiki  Pulse  Graphs
Branch: master Find file Copy pathtechnicalanalysislibrary/C++ code for Visual Studio
2275b2e  16 days ago
@ishchat ishchat Create C++ code for Visual Studio
1 contributor
RawBlameHistory     270 lines (216 sloc)  11.4 KB
#include "stdafx.h"

#include <vector>
#include <string>
#include <iostream>
#include <sstream>
#include <fstream>
#include <time.h>
//#include "C:\ta-lib-0.4.0-msvc\ta-lib\c\include\ta_libc.h"
#include "C:\Users\IshanC\Desktop\readingdatainvisualc++\csv.h"
#include "boost\date_time\gregorian\gregorian.hpp"
#include "boost\date_time\posix_time\posix_time.hpp"
#include <string>
#include <iomanip>
#include <locale>
#include <stdexcept>

// for mmap:
#include <boost/iostreams/device/mapped_file.hpp> // for mmap
#include <algorithm>  // for std::find
#include <cstring>

//Strategies to code : SMA, EMA, FRMA, RSI, CCI, Supertrend, ATR, ROC, MACD, n-bar rules
//Bulkowski patterns - How to code the pattern : http://thepatternsite.com/HTFStudy.html, http://thepatternsite.com/SmallPatterns.html , http://thepatternsite.com/id75.html, http://thepatternsite.com/id74.html,  http://thepatternsite.com/Trendiness.html, 
//Short term trading strategies that work - RSI-2, TRIN etc
//IBS
//Linda Raschke : 2-period ROC modeling, Taylor modeling, and volatility breakout methods -- http://lindaraschke.net/research/
//VIX/VXV ratio (godotfinance working papers)


//Use pure C++ as much as possible and Boost as less as possible
//https://google.github.io/styleguide/cppguide.html#General_Naming_Rules - Naming conventions

//Read large files into C++
//http://stackoverflow.com/questions/22541801/handling-large-amounts-of-data-in-c-need-approach -- SQlite
//http://stackoverflow.com/questions/27401401/parse-very-large-csv-files-with-c -- QT


//Faster C++ code :
//https://people.cs.clemson.edu/~dhouse/courses/405/papers/optimize.pdf
//https://www.quora.com/How-can-I-reduce-execution-time-on-my-C-C++-code
//Use loops within function instead of function within loops to avoid repeatred calling of function - this has overhead
//Use reference (pointers) for passing containers (including vectors) to functions
//Use inline functions where possible as it has no overhead
//Preinitialized/Preallocated vector is one of the fastest containers - reserve preallocates memory but not initialize
//http://en.cppreference.com/w/cpp/types/numeric_limits/quiet_NaN and http://en.cppreference.com/w/cpp/types/numeric_limits/signaling_NaN

//Use STL algorithms instead of for loops for speed:
//https://msdn.microsoft.com/en-IN/library/hh438471.aspx
//https://meetingcpp.com/index.php/br/items/raw-loops-vs-stl-algorithms.html
//http://programmers.stackexchange.com/questions/37047/prefer-algorithms-to-hand-written-loops
//http://stackoverflow.com/questions/135129/should-one-prefer-stl-algorithms-over-hand-rolled-loops
//http://www.drdobbs.com/stl-algorithms-vs-hand-written-loops/184401446
//http://www.drdobbs.com/stl-algorithms-vs-hand-written-loops/184401446

//Advantage of calling functions like SMA on full data is that overhead is avoided by repeated calling of function within main()
//Also, if multiple such functions are there, they can be run parallely in separate threads and then joined within main()

//Since it is better to pass containers by reference to functions to reduce overhead, if we are using functions in separate threads, 
//then normal pass by reference will not work - you need to use std::ref for this
//http://stackoverflow.com/questions/8250932/what-is-the-overhead-of-passing-a-reference
//http://stackoverflow.com/questions/5116756/difference-between-pointer-and-reference-as-thread-parameter
//http://stackoverflow.com/questions/8299545/why-does-passing-object-reference-arguments-to-thread-function-fails-to-compile
//http://jakascorner.com/blog/2016/01/arguments.html
//http://www.bogotobogo.com/cplusplus/multithreaded4_cplusplus11.php

using namespace std;

//https://github.com/ben-strasser/fast-cpp-csv-parser
//https://code.google.com/archive/p/fast-cpp-csv-parser/issues/6

int LinesCount()
{
	//http://stackoverflow.com/questions/17925051/fast-textfile-reading-in-c

	//boost::iostreams::mapped_file mmap("C:/Users/IshanC/Desktop/Subsectors/Subsectors_test2.csv", boost::iostreams::mapped_file::readonly);
	boost::iostreams::mapped_file mmap("C:/Users/IshanC/Desktop/Subsectors/Topix_Subsectors_5min_2.csv", boost::iostreams::mapped_file::readonly);
	auto f = mmap.const_data();
	auto l = f + mmap.size();

	uintmax_t m_numLines = 0;
	while (f && f != l)
	if ((f = static_cast<const char*>(memchr(f, '\n', l - f))))
		m_numLines++, f++;

	std::cout << "m_numLines = " << m_numLines << "\n";
	return m_numLines;
}

vector<double> RollingSMA(vector<double> price_vector, int period){
	//http://stackoverflow.com/questions/18741651/should-i-use-size-type-in-my-code -- size_type return type
	std::vector<double>::size_type price_vector_size = price_vector.size(); //This is the number of actual objects held in the vector, which is not necessarily equal to its storage capacity.
	//http://stackoverflow.com/questions/8480640/how-to-throw-a-c-exception -- throw statement
	if (period > price_vector_size) { throw std::invalid_argument("period too small for vector size"); } // period can't be greater than the length of data itself

	vector<double> mRollingSMA;
	mRollingSMA.reserve(price_vector_size);

	for (size_t i = period; i < price_vector_size; i++)
	{
		double sum = 0;
		for (size_t j = i - period; j < i; j++)
		{
			sum = sum + price_vector[j];
		}

		mRollingSMA[i] = sum / period;
	}

	return mRollingSMA;
}

vector<double> RollingEMA(vector<double> price_vector, int period){
	//http://stackoverflow.com/questions/18741651/should-i-use-size-type-in-my-code -- size_type return type
	std::vector<double>::size_type price_vector_size = price_vector.size(); //This is the number of actual objects held in the vector, which is not necessarily equal to its storage capacity.
	//http://stackoverflow.com/questions/8480640/how-to-throw-a-c-exception -- throw statement
	if (period < price_vector_size) { throw std::invalid_argument("period too small for vector size"); }

	vector<double> RollingEMA;
	RollingEMA.reserve(price_vector_size);

	for (size_t i = period; i < price_vector_size; i++)
	{
		double sum = 0;
		for (size_t j = i - period; j < i; j++)
		{
			sum = sum + price_vector[j];
		}

		RollingEMA[i] = sum / period;
	}

	return RollingEMA;
}

int main(int argc, char* argv[]){

	int m_numLines = LinesCount();

	//io::CSVReader<9> in("C:/Users/IshanC/Desktop/Subsectors/Topix_Subsectors.csv");

	//io::CSVReader<9> in("C:/Users/IshanC/Desktop/Subsectors/Topix_Subsectors_5min.csv");
	//in.read_header(io::ignore_extra_column, "#RIC", "Date[L]", "Time[L]", "Type", "Open", "High", "Low", "Last", "Volume");

	io::CSVReader<9> in("C:/Users/IshanC/Desktop/Subsectors/Topix_Subsectors_5min_2.csv");
	//io::CSVReader<9> in("C:/Users/IshanC/Desktop/Subsectors/Subsectors_test2.csv");
	in.read_header(io::ignore_extra_column, "RIC", "Date", "Time", "Type", "Open", "High", "Low", "Last", "Volume");
	//in.read_header("RIC", "Date", "Time", "Type", "Open", "High", "Low", "Last", "Volume");

	std::string RIC, Date, Time, Type;
	double Open, High, Low, Last, Volume;

	vector<std::string> vRIC, vDate, vTime, vType;
	vector<double> vOpen, vHigh, vLow, vLast, vVolume;
	//size_t filesize = 1048575;
	//size_t filesize = 8062915;
	//size_t filesize = 8388600;
	//size_t filesize = 5388600;

 	//Reserve preallocates memory for vector and speeds up later initialization to it
	//http://stackoverflow.com/questions/1461276/stdvector-reserve-and-push-back-is-faster-than-resize-and-array-index-w
	//http://baptiste-wicht.com/posts/2012/12/cpp-benchmark-vector-list-deque.html
	//http://www.liamdevine.co.uk/code/images/container.png
	vRIC.reserve(m_numLines);
	vDate.reserve(m_numLines);
	vTime.reserve(m_numLines);
	vType.reserve(m_numLines);
	vOpen.reserve(m_numLines);
	vHigh.reserve(m_numLines);
	vLow.reserve(m_numLines);
	vLast.reserve(m_numLines);
	vVolume.reserve(m_numLines);

	int count = 0;
	while (in.read_row(RIC, Date, Time, Type, Open, High, Low, Last, Volume)) {
		//vRIC[count] = RIC; vDate[count] = Date; vTime[count] = Time; vType[count] = Type;
		//vOpen[count] = Open; vHigh[count] = High; vLow[count] = Low; vLast[count] = Last; vVolume[count] = Volume;
		vRIC.push_back(RIC); vDate.push_back(Date); vTime.push_back(Time); vType.push_back(Type);
		vOpen.push_back(Open); vHigh.push_back(High); vLow.push_back(Low); vLast.push_back(Last); vVolume.push_back(Volume);
		count++;
	}

	cout << "File reading done!\n";
	//cout << double(clock() - startTime) / (double)CLOCKS_PER_SEC << " seconds." << endl;
	cout << count << endl;
	cout << RIC << Date << Time << Type << Open << High << Low << Last << Volume << endl;

	vector<double> rolling_sma = RollingSMA(vLast, 3);


	int current_position = 0, current_action = 0, current_signal = 0;
	double ltp;
	
	vector<double> position, action;

	clock_t startTime = clock();
	//PNL loop
	/*
	for (size_t i = 0; i < m_numLines; i++)
	{
		//http://stackoverflow.com/questions/3786201/how-to-parse-date-time-from-string
		//http://stackoverflow.com/questions/1904317/how-to-format-date-time-object-with-format-dd-mm-yyyy
		//http://www.boost.org/doc/libs/1_61_0/doc/html/date_time/posix_time.html
		namespace bt = boost::posix_time;
		Date = 
		//ltp = vLast[i];

	}
	*/
	cout << double(clock() - startTime) / (double)CLOCKS_PER_SEC << " seconds." << endl;
	
	/*
	cout << "myvector contains:";
	vector<string>::iterator it1 = --vDate.end();
	vector<string>::iterator it2 = --vTime.end();
	std::cout << ' ' << *it1 << ' ' << *it2;
	std::cout << '\n';
	*/

	//http://stackoverflow.com/questions/7274268/which-is-faster-vector-of-structs-or-a-number-of-vectors
	//According to link above, as we are going to just use the Date, Time, Last and Volume together, it makes sense to bundle them
	//in a struct together instead of keeping them as separate vectors. then make a vector of this struct to create the whole series
	//Do a timing test to check if this vector of structures is faster than separate vectors

	//It is appearing that sometimes iterators are faster and sometimes [] operator is faster on C++ vector
	//http://stackoverflow.com/questions/9506018/efficiency-of-vector-index-access-vs-iterator-access?noredirect=1&lq=1
	//http://stackoverflow.com/questions/16350518/c-vector-performance-iterator-access-versus-brackets-operator-access?noredirect=1&lq=1
	//Need to time both

	//using namespace boost::posix_time;
	//using namespace boost::gregorian;

	/*
	date d(2002, 02, 1); //an arbitrary date
	//construct a time by adding up some durations durations
	ptime t1(d, hours(5) + minutes(4) + seconds(2) + millisec(1));
	//construct a new time by subtracting some times
	ptime t2 = t1 - hours(5) - minutes(4) - seconds(2) - millisec(1);
	//construct a duration by taking the difference between times
	time_duration td = t2 - t1;

	std::cout << to_simple_string(t2) << " - "
		<< to_simple_string(t1) << " = "
		<< to_simple_string(td) << std::endl;
		*/
	//Percentile : https://www.informit.com/guides/content.aspx?g=cplusplus&seqNum=290
	//

	/*
	system.time(Subsectors <- read.csv(file="C:/Users/IshanC/Desktop/Subsectors/Topix_Subsectors_5min_2.csv", header=TRUE, sep=",", stringsAsFactors=FALSE))
	head(Subsectors)
	tail(Subsectors)
	Subsectors1 <- Subsectors
	Subsectors1 <- rbind(Subsectors1, Subsectors1)
	Subsectors1 <- rbind(Subsectors1, Subsectors1)
	write.table(Subsectors1, append = FALSE, file=paste("C:/Users/IshanC/Desktop/Subsectors/Subsectors_test2", ".csv", sep=""),  sep= ",",  quote=FALSE, row.names=FALSE, col.names = TRUE)
	rm(list=ls())
	gc()
	*/

	std::ofstream f("C:\Users\IshanC\Desktop\test.txt");
	for (vector<double>::const_iterator i = rolling_sma.begin(); i != rolling_sma.end(); ++i) {
		f << *i << '\n';
	}

	system("pause");

	return 0;
}
Contact GitHub API Training Shop Blog About
© 2016 GitHub, Inc. Terms Privacy Security Status Help
