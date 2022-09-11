# glosbe-language
Scrape sentences from Glosbe dictionary

The code aims to collect example sentences in two different languages for desired words.

I wrote this code to collect example sentences in German and Hungarian for the top 500 used German verbs. 

You need to provide the desired words in an CSV file. The scraped sentences will be stored in a text file called qoutes.txt. Each line in the text file is containing three things, the search word, the German sentence, the Hungarian sentence. Other languages can be used. You need to look for the suitable abbreviation of the desired language on Glosbe. 

All the words must be stored in "A" column in CSV file called "words.csv" in the same directory of the sc2.py file.

The code is tested and working up to September 2022.

I hope it will be useful. 
