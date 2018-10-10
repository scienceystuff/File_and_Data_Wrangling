### Fastq FileManipulation
Listed are several commands for file manipulation.  Unless specifically noted, all commands will work on any flavor of *nix.
Bullet points that are labeled with "grep" use grep within a Bash command.  

Bullet points that are labeled with "awk" either call awk directly, or use an awk command file that is also described.  

Bullet points that are labeled with "sed" use sed within a Bash command.  

Bullet points that are labeled with "perl" call perl from the commandline. The syntax is the same as if a perl script were written and called but with all commands completed in a single line of code which can be a bit confusing.

Bullet points that are labeled simply as "bash" do not use awk, sed, or grep directly.

#### Perl commandline Notes

Perl commands via commandline are set up with the syntax:

        perl OPTIONS COMMANDS file 
        
The OPTIONS for perl are:

        -e sends the perl command to the compiler to execute

        -p creates a while (<>) {} loop, and prints the output of the loop

        -n creates a while (<>) {} loop

        -i edit in-place, optionally add argument -i.bak to create backup

        -l with no option: chomp input and add newline

        -l with octal number, chomp input, set output record separator to 0, example: -l0

        -F'' set field separator

        -a turn on autosplit mode, store elements in specified array

        -M use module specified

        LINE: directions to script for line of text, example: next LINE unless /pattern/

        END: execute instruction at end of loop

The COMMANDS for perl follow perl syntax.  An excellent overview (and credit for the examples) is [here](https://www.perl.com/pub/2004/08/09/commandline.html/).

#### Awk commandline Notes

The basic awk syntax is

        awk COMMANDS file

Note that commands can also be piped to awk:

        some_command | awk COMMANDS 

An excellent but verbose overview is [here](http://linuxcommand.org/lc3_adv_awk.php)

#### Sed commandline notes

The basic sed syntax is

        sed OPTIONS COMMANDS file

A good overview of sed commands can be found [here](https://unix.stackexchange.com/a/26290)
sed has many possible OPTIONS that can be utilized; a few are:

        -e *execute* command
        -i *execute* command to modify the existing file, a backup file wit hthe suffix '.bak' will be automatically created
        # IMPORTANT: 
        # on GNU sed, the *-i* option can be called without additional arguments
        # On MacOSX sed, you MUST specify an output backup file like this:
        -i.bak 

Note that commands can also be piped to sed:

        some_command | sed OPTIONS COMMANDS
 
#### File manipulation commands

1) Count unique fastq headers
* awk

        # gz-compressed (Note: do not use this particular command with tar.gz files!)
        # explanation: This uncompresses a fastq.gz file, then looks for unique fastq headers and returns the count of each
        gzcat fqfile.fastq.gz | awk '/^@A[0-9]+:14:H3[A-Z,0-9]+/{nr[NR]}; NR in nr' | cut -d:  -f1-3 | sort -n | uniq -c >> readNameListOfCounts.txt
        # 
        # already-uncompressed fastq files
        # explanation: This looks for unique fastq headers and returns the count of each
        awk '/^@A[0-9]+:14:H3[A-Z,0-9]+/{nr[NR]}; NR in nr' fqfile.fastq | cut -d:  -f1-3 | sort -n | uniq -c >> readNameListOfCounts.txt

* grep

        # count the number of headers that match REGEX using the -c option
        grep -c REGEX fileToParse.txt

2) remove the first line and/or last line of a file
* sed

        # remove first and last line
        sed '1d;$d' fileToParse.txt > fileThatWasParsed.txt
        # remove first line only
        sed '1d' fileToParse.txt > fileThatWasParsed.txt
        # remove last line only
        sed '$d' fileToParse.txt > fileThatWasParsed.txt

* bash

        # remove first and last line
        export V=$(wc -l fileToParse.txt | rev | cut -d\  -f2 | rev) ; let TRIMCT=$V-1 ; let TRIMSTOP=$V-2 ; tail -n $TRIMCT fileToParse.txt | head -n $TRIMSTOP > fileThatWasParsed.txt
        # remove first line only
        export WCOUNT=$(wc -l fileToParse.txt | rev | cut -d\  -f2 | rev) ; let TRIMCT=$WCOUNT-1 ; head -n $TRIMCT fileToParse.txt
        # remove last line only
        export WCOUNT=$(wc -l fileToParse.txt | rev | cut -d\  -f2 | rev) ; let TRIMCT=$WCOUNT-1 ; head -n $TRIMCT fileToParse.txt
        
3) insert newline at the beginning of file
* sed

        # insert blank newline
        # credit mojuba StackExchange
        sed -e '1s/^/\'$'\n/' fileToInsertNewline.txt > fileWithInsertedNewline.txt
        # modify in-place
        sed -i.bak '1s/^/\'$'\n/' fileToInsertNewline.txt

* bash

        # insert blank newline at beginning of file
        echo '' > rmMeFile ; cat rmMeFile fileToParse.txt > fileThatWasParsed.txt ; rm rmMeFile
        
4) Insert line with/without text at the beginning/end of a file
* sed

        # insert line with the text '[HelloWorld' at the beginning of file
        sed -e '1s/^/\[HelloWorld\'$'\n/' fileToParse.txt > fileThatWasParsed.txt
        # insert line with the text 'HelloWorld]' at the end of file, *$* directs sed to the end of the file
        sed -e '$ s/HelloWorld/HelloWorld\]/g' fileToParse.txt > fileThatWasParsed.txt
        # alternative way to do this, *a* appends a line at the end *$* of the file with the text 'HelloWorld]'
        sed -e '$a\'$'\nHelloWorld\]' fileToParse.txt > fileThatWasParsed.txt

* bash 

        # insert line with text '[HelloWorld'
        echo '[HelloWorld' > rmMeFile ; cat rmMeFile fileToParse.txt > fileThatWasParsed.txt ; rm rmMeFile

5) Look for a match to STRING in a file, then print all lines after that match

* awk 

        # search for match to STRING, then print all lines until STRING2 is encountered
        awk '/STRING/{flag=1;next}/STRING2/{flag=0}flag' file

* sed

        # search for a match to i in a file, then modify that value
        # note to be extremely wary of the value of the contents of the variable *i* for security reasons; sed will expand the variable blindly
        export i='SOMEVALUE' ; sed -e "s/AC_000${i}.1/AC_000${i}/g" fastaFileWithIntegersListed.fasta > fastaFileWithIntegersListedModified.fasta

* perl

        # search for STRING then replace with REPLSTRING
        perl -pe 's/STRING/REPLSTRING/' unparsedFasta.txt > parsedFasta.txt
