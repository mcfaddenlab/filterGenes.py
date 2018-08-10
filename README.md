import sys
import csv
"""
Author: Brooks Macdonald
Date: 2018 July 23
Description: Filter tab delimited files with gene annotations
to yield unique genes with the largest bitscore

"""
def load(filename):
    """
    Input: filename of tab delimited file
    Returns: dictionary containing only unique genes (filtering out 
    repeats) using max bitscore as the filter. Length is the tiebreaker
    if bitscores are identical.
    """
    
    genes = {}
    #Dictionary. Format: geneID is the key, the full line of the tab delimited
    #file is the data.

    geneName = ""
    #hardcoded positions of relevant data
    geneIDPos = 1
    bitScorePos = 13
    lengthPos = 5
    with open(filename, "r") as file:
        reader = csv.reader(file, delimiter = '\t')
        #read in original file
        for line in reader:
            geneName = line[geneIDPos]
            if geneName in genes:#gene already in dictionary, need to check
            #bitscore
                print("checking bitscore for gene " + geneName)
                currentBitScore = line[bitScorePos]
                oldBitScore = genes[geneName][bitScorePos]
                #print("current: " + currentBitScore)
                #print("old: " + oldBitScore)
                #cast bitscores to float to avoid weird errors
                if float(currentBitScore) > float(oldBitScore):
                    #print(currentBitScore + ">" +oldBitScore)
                    genes[geneName] = line
                    #print("using current")
                #use length for a tiebreaker
                elif currentBitScore == oldBitScore:
                    #print("length tiebreaker!")
                    currentLength = line[lengthPos]
                    oldLength = genes[geneName][lengthPos]
                    #print("current length " + currentLength)
                    #print("old length" + oldLength)
                    if float(currentLength) > float(oldLength):
                        genes[geneName] = line
                        #print("using current")
                    #else:#do nothing
                        #print("keeping old")
                #else:#do nothing
                    #print(currentBitScore + "<" +  oldBitScore)
                    #print("keeping old")
            else:#first time this gene has appeared, put it in dictionary

                genes[geneName] = line
            
    return genes

def writeNew(geneDict, filename):
    """
        Input: a dictionary containing the output of load(fn).
        Output: a tab delimited file containing the information stored
        in that dictionary.
    """
    with open("filtered" + filename, 'w') as newTable:
        writer = csv.writer(newTable, delimiter = "\t", lineterminator= "\n" )
        #extract header line from dict
        firstLine = geneDict.pop("gene id")
        writer.writerow(firstLine)
        
        for key in geneDict:
            data = geneDict[key]        
            writer.writerow(data)
        print("new file complete.")



def main():
    """Usage: ipython filterGenes.py filename.extension
       Output: filteredfilename.extension, a tab delimited table
       with the data extracted from the load() function.
       
    """
    if len(sys.argv) == 2:
        filename = sys.argv[1]
        output = load(filename)
        writeNew(output, filename)
        
    else:
        raise Exception("%s: error: invalid command" % sys.argv[0])
if __name__ == '__main__':
    sys.exit(main())
# filterGenes.py
