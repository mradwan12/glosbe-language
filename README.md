# glosbe-language
Scrape sentences from Glosbe dictionary

The code aims to collect example sentences in two different languages for desired words.

I wrote this code to collect example sentences in German and Hungarian for the top 500 used German verbs. 

You need to provide the desired words in an CSV file. The scraped sentences will be stored in a text file called qoutes.txt. Each line in the text file is containing three things, the search word, the German sentence, the Hungarian sentence. Other languages can be used. You need to look for the suitable abbreviation of the desired language on Glosbe. 

All the words must be stored in "A" column in CSV file called "words.csv" in the same directory of the sc2.py file.

The code is tested and working up to September 2022.

I hope it will be useful. 


#!/usr/bin/python
# -*- coding: utf-8 -*-
import requests
from bs4 import BeautifulSoup
import csv
import time


def beautifulSoapPrepare(sourceLang,destLang,phrase):
    headers = {
            'User-Agent': 'My User Agent 1.0',
            'From': 'youremail@domain.example'  # This is another valid field
        }
    # headers = {
    # 'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/104.0.5112.79 Safari/537.36'
    # }
    
    url='https://glosbe.com/'+sourceLang+'/'+destLang+'/'+phrase
    r = requests.get(url,headers=headers)
    soup = BeautifulSoup(r.content,"html.parser")
    return soup

def phraseSearcher(soup):

    table = soup.findAll('li', attrs = {'class':'translation__item'}) 

    #print(table)
    sentences=[]
    if len(table)>0:
        for tablei in table:
            text=tablei.find('div', attrs = {'text-sm text-gray-700'})
            if text != None:
                div=text.find('div',attrs = {'translation__example'})
                p=div.findAll('p')
                intSentences=[]
                for count,pi in enumerate(p):
                    sentence=""
                    for word in pi:
                        sentence=sentence+str(word)
                    intSentences.append(sentence)
                sentences.append(intSentences)
        #print(sentences)
    return sentences

def phraseSearcher2(soup):
    table = soup.find('div', attrs = {'id':'tmem_first_examples'}) 
    counter=0
    intSentences=[]
    if table != None:
        firstDiv=table.find('div')
        if len(firstDiv)>0:
            divs=firstDiv.findAll('div',attrs={'class':'tmem__item'})
            for count,div in enumerate(divs):
                subDivs=div.findAll('div')
                for ssDiv in (subDivs):
                    sen=ssDiv.text.strip('\n')
                    if not(sen in intSentences):
                        intSentences.append(ssDiv.text.strip('\n'))
    
    return intSentences

def SentenceSorter(intSentences):
    Sentences=[]
    counter=0
    for count,sent in enumerate(intSentences):
        if counter<2  and count<(len(intSentences)-1):
            Sentences.append([intSentences[count],intSentences[count+1]])
            counter=counter+1
        elif counter==2:
            counter=0

    #print (Sentences)
    return Sentences

def chooseBestSentence(mainList):
    
    if len(mainList)>0:
        shortestSenIndex=0
        shortestSenLength=len(mainList[0][0])
        for count,sen in enumerate(mainList):
            if len(sen[0])<shortestSenLength:
                shortestSenIndex=count
    return shortestSenIndex



def excelRead():
    examples=[]
    with open('words.csv', newline='') as csvfile:
        spamreader = csv.reader(csvfile, delimiter=',')
        counter=0
        for row in spamreader:
            try:
                if counter<220:
                    time.sleep(2)
                    if row == 's.':
                        examples.append(["","",""])
                        counter=counter+1
                    else:
                        soup=beautifulSoapPrepare("de","hu",row[0])
                        mainExamples=phraseSearcher(soup)
                        phraseSentences=phraseSearcher2(soup)
                        secondaryExamples=SentenceSorter(phraseSentences)
                        if len(mainExamples)>0:
                            examples.append([row[0],mainExamples[chooseBestSentence(mainExamples)][0],mainExamples[chooseBestSentence(mainExamples)][1]])
                        elif len(secondaryExamples)>0 and len(mainExamples)==0:
                            examples.append([row[0],secondaryExamples[chooseBestSentence(secondaryExamples)][0],secondaryExamples[chooseBestSentence(secondaryExamples)][1]])
                        counter=counter+1
                        print(counter)
                else:
                    break
            except Exception:
                print(row)
                print('fail')
    print(examples)
    return examples

def fileWrite():
    header = ['word', 'deutsch', 'magyar']  
    rows=excelRead()
    with open('quotes.txt', 'w', encoding='utf-8') as f:
        for row in rows:
            print(row)
            f.write(str(row[0])+';'+str(row[1]+';'+str(row[2])+'\n'))

fileWrite()
