---
layout: post
title: "Data structure Trie and its application"
categories: 
- Data Structure and Algorithms
tags: 
- Trie
- Dynamic Programming
close: 2014-02-20
---

a trie, also called digital tree and sometimes prefix tree (as they can be searched by prefixes), is a multi-way tree structure useful for storing strings over an alphabet. All the descendants of a node have a common prefix of the string associated with that node, and the root is associated with the empty string. Values are normally not associated with every node, only with leaves and some inner nodes that correspond to a word (a string given in the dictionary).

Given the dictionary:

an, ant, all, allot, alloy, aloe, are, ate, be

the corresponding trie would be:

<img src="/images/trie.gif" width="600" />

The term trie comes from retrieval. This term was coined by Edward Fredkin, who pronounces it /ˈtriː/ "tree" as in the word retrieval. However, other authors pronounce it /ˈtraɪ/ "try", in an attempt to distinguish it verbally from "tree".

Let's take a example here to demonstrate the application of trie. This is a interview question described as something like below:

"Write a program that reads a file containing a sorted list of words (one word per line, no spaces, all lower case), then identifies the longest word in the file that can be constructed by concatenating copies of shorter words also found in the file. The program should then go on to report the second longest word found as well as how many of the words in the list can be constructed of other words in the list."

# Analysis

The question can be generalized to finding compound words in a given dictionary (a sorted list of words), and report the longest one, the second longest one, and the amount of such words. A compound word is a word that constructed by concatenating copies of shorter words. A word is defined as "a string given in the dictionary". To be simple, we suppose all words consists of lower case letters, i.e., ‘a’ to ‘z’ only. An example is given as below:

Dictionary { cat, cats, catsdogcats, catxdogcatsrat, dog, dogcatsdog, hippopotamuses, rat, ratcatdogcat }
The longest compound word is “ratcatdogcat”.
The second longest compound word is “catsdogcats”.
The amount of compound words is 3.

## 1. The algorithm to determine a compound word

Lets consider the compound word “ratcatdogcat” given in the example. If we separate the suffix word as “ratcatdog|cat”, it is easy to find that the whole word is a compound can be determined by two points:

1. The suffix “cat” is a word.
2. The prefix “ratcatdog” is a word or a compound (not necessary to be a word).

By this means, the problem is broken down to its subproblems. Such kind of problem can be typically solved with Dynamic Programming as below: 

Algorithm 1:

Given W(n) denotes a word with length n and P(i) denotes its prefix with length i, then DP[i] == 1 denotes P(i) is a compound (not necessary a word), otherwise DP[i] == 0. 

Given S(j) denotes the shortest suffix word of P(i) and the corresponding prefix of P(i) is P(j), then

DP[i] = 1 if P(j) is a word or DP[j] == 1.

W(n) is a compound word only if DP[n] == 1.

## 2. The data structure to load dictionary

In Algorithm 1, we need to frequently find a suffix word, i.e., search a word in dictionary. Trie is one of the best data structure for dictionary to support searching. General Trie stores a single letter of words in a node. Each node holds a set of children nodes corresponding to the next letter in the words. That means Trie holds a single copy of the same prefix of different words. 

In Algorithm 1, we need to calculate and record a DP array for each word to be checked. It is easy to find that the same prefix of different words also has same DP values. Hence it is reasonable to store the DP values in Trie nodes. By this means, the repeat calculation of DP values for the same prefix of different words is eliminated. 

It is worth to mention that during calculating of DP value for a prefix we need to search the suffix word in the whole dictionary, so it is impossible to complete calculation of DP values during construction of the Trie. We have to load the words from file to construct the Trie and then traverse the Trie to calculate DP values for each node (corresponding to a prefix).

## 3. The algorithm to count compound words and search (n) longest one(s)

It is easy to understand that the node in Trie that marks as a word and as a compound denotes a compound word in the dictionary. All compound words can be counted and the longest n compound words can be tracked with a traversal of the Trie. 

Actually, during the traversal of DP calculation, we may find the compound words and complete counting and tracking simultaneously. 
We only need to track the longest and the second longest compound words here, so we may simply use two variables to track them during traversal. If more compound words need to be tracked, we may use a max heap to do the work. 

# Design

First we customize Trie to support our algorithm to find compound words. In addition to the regular member of a node of Trie, we store the DP value of prefix with a mark of “IsCompound”. In addition, to support back track the suffix word in dynamic programming algorithm, we store a pointer to parent node. So the structure of TrieNode is as below:

	struct TrieNode
	{
		bool isWord; 
		bool isCompound; 
		struct TrieNode* children[LETTER_NUMBER]; 
		struct TrieNode* parent; 
	}

Calculation of DP values and check of compound words are done in a traversal of Trie. Our traversal function store prefix in a string on each node. The algorithm to calculate DP and check compound word can be done with below code:

	size_t n = 0; // Length of suffix
	TrieNode* p = node; // Back track from current node
	while((p = p->parent) != NULL) // Back track
	{
		n++;
		if(p->isCompound || p->isWord) // Prefix is a compound or a word
		{
			if(search(prefix.substr(prefix.length() - n))) // suffix is a word
			{
				node->isCompound = true; // Mark prefix to this node as a compound
				if(node->isWord) // Find a compound word
				{
					_count++; // Total number of compound words
					if(prefix.length() > _longest.length()) // Track the longest one
					{
						_second = _longest;
						_longest = prefix;
					}
					else if(prefix.length() > _second.length()) // Track the second one
					{
						_second = prefix;
					}
				}
				break;
			}
		}
	}

Other functions like construction of Trie. seach in Trie, and traversal of a Trie have somehow standard implementations. Please refer the source code for details.
 	
# Implementation and testing

The design is implemented with C++. To be simple, all implementation is in a single C++ file. Please find a complete copy of the source code <a href="https://github.com/maoxuli/compounds">here</a>.

The source code is compiled on Mac OS X 10.8 with g++ 4.2 and on Windows XP with Visual C++ Express 2010. 
The executable program tries to load the dictionary from a file named “wordsforproblem.txt” located in the same folder of the program. If it is failed to load the file, the program load the words of the example given above.
 
The program is debugged with gdb 6.3 on Mac OS X 10.8 and Visual C++ Express 2010 on Windows XP. There is no memory leak or other running issues. 

With the dictionary given in the file, the program found 97107 compound words. The longest one is “ethylenediaminetetraacetates” and the second longest one is “electroencephalographically”. In one testing running on Mac OS X, the program takes 254 milliseconds to load the dictionary form the file to the Trie and takes 335 milliseconds to find all compound words in the dictionary. On average, Windows takes longer time (400- ms) to load the dictionary than Mac OS X, while takes shorter time (100+ ms) to find the compound words. 

Two screenshots of the program running on Mac OS X and Windows are given below:

<img src="/images/trie-screen-1.png" width="600" />

<img src="/images/trie-screen-2.png" width="600" />


# Reference

* [1] <a href="http://en.wikipedia.org/wiki/Trie">http://en.wikipedia.org/wiki/Trie</a>
* [2] <a href="http://www.allisons.org/ll/AlgDS/Tree/Trie/">http://www.allisons.org/ll/AlgDS/Tree/Trie</a>

 