# Read my notes thoroughly please

The below code in Megan's script said we need to check the chromosome names in our genome file which is the fasta file, and said that VEP only accepts numeric chromosome names which for example 01, 02, 03, ... as the chromosome names instead of other names. 

```sh
##### STEP 1: Remove all ISOLATE and chr names from the fasta file. VEP only accepts numeric chromosome names
cat IPO323.fasta | grep ">"

bgzip -c "$REFERENCE" > genome.fa.gz

echo "If you see anthing other than chromosome numbers above you need to fix your fasta file as the VEP tool only accepts 1, 2, 3...as chromosome names"
```

The code `cat genome.fasta | grep ">"` greps all the chromosome names in a fasta file. Which in our example, we will run:

```sh
cat WAI332.fasta | grep ">"
```

and the result is:

```
>WAI332_chr_01
>WAI332_chr_02
>WAI332_chr_03
>WAI332_chr_04
>WAI332_chr_05
>WAI332_chr_06
>WAI332_chr_07
>WAI332_chr_08
>WAI332_chr_09
>WAI332_chr_10
>WAI332_chr_11
>WAI332_chr_12
>WAI332_chr_13
>WAI332_chr_14
>WAI332_chr_15
>WAI332_chr_16
>WAI332_chr_19
>WAI332_chr_20
>WAI332_chr_21
```

Which our chromosome names in the fasta file contains texts such as `WAI` and `chr`. We want the chromosome names looks like:

```
>01
>02
>03
...
>21
```

To change our chromosome names to the correct format, we can do:

```sh
sed 's/^>WAI332_chr_\([0-9]\+\)/>\1/' WAI332.fasta > WAI332_CorrectNames.fasta
```

Then, we can run `ls -lh` to check if our input and output files are the same size we make sure we didn't accidentally cut anything off. The result of `ls -lh` shows:

```
total 88M
-rw-rw-r-- 1 u1133824 u1133824  39M Mar 25 15:10 WAI332_CorrectNames.fasta
-rw-rw---- 1 u1133824 u1133824 9.7M Mar 25 14:41 WAI332_curatedAvrs.gff
-rw-rw---- 1 u1133824 u1133824  39M Mar 25 14:41 WAI332.fasta
```

Looks like both `WAI332.fasta` and changed `WAI332_CorrectNames.fasta` are in the same size, which is good. Then we can grep all the chromosome names again in our new file to double check.

```sh
cat WAI332_CorrectNames.fasta | grep ">"
```

It returns us the result:

```
>01
>02
>03
>04
>05
>06
>07
>08
>09
>10
>11
>12
>13
>14
>15
>16
>19
>20
>21
```

Which means we have successfully change the names to the correct format. Then, next we can do the next step which is to compress our fasta file using `bgzip`:

```sh
bgzip -c WAI332_CorrectNames.fasta > genome.fa.gz
```

Then it's the below code from Megan we need to figure out where is wrong:

```sh
##### STEP 2: Format your gff3 file to fit the VEP rules and expectations..

sed 's/three_prime_UTR/3_prime_UTR/g; s/five_prime_UTR/5_prime_UTR/g' "$GFF" | grep -v "#" | sort -k1,1 -k4,4n -k5,5n -t$'\t' | bgzip -c > data.gff.gz
tabix -p gff data.gff.gz
```

As we have checked ealier, we don't have any information about UTRs in our GFF file, so we will skip the first part in the pipe. I will separate the pipe to make sure we did them step by step. The first step is to remove the header lines which containing `#`:

```sh
grep -v "#" WAI332_curatedAvrs.gff > WAI332_cleaned.gff
```

Then we can check our new GFF file by running `less WAI332_cleaned.gff`, and double check the header lines has been removed. Next we need to sort the GFF file:

```sh
sort -k1,1 -k4,4n -k5,5n -t$'\t' WAI332_cleaned.gff > WAI332_sorted.gff
```

Next, we need to check if we have empty lined in the sorted GFF file because that's where we have errors when running `tabix`: 

```sh
grep -n "^$" WAI332_sorted.gff 
```

The result shows us:

```
1:
2:
3:
4:
5:
6:
7:
8:
9:
10:
11:
12:
13:
14:
15:
16:
17:
18:
19:
20:
21:
22:
23:
24:
25:
26:
27:
28:
29:
30:
31:
32:
33:
34:
35:
36:
37:
38:
39:
40:
41:
42:
43:
44:
45:
46:
47:
48:
49:
50:
51:
52:
53:
54:
```

It seems from line 1 to line 54 are empty lines from our file `WAI332_sorted.gff`, which I also checked our original GFF file `grep -n "^$" WAI332_curatedAvrs.gff`, which gives us the result:

```
17056:
17057:
17058:
27123:
27124:
27125:
36626:
36627:
36628:
44339:
44340:
44341:
51646:
51647:
51648:
57781:
57782:
57783:
64953:
64954:
64955:
70794:
70795:
70796:
76059:
76060:
76061:
80746:
80747:
80748:
85043:
85044:
85045:
88940:
88941:
88942:
91989:
91990:
91991:
92716:
92717:
92718:
93737:
93738:
93739:
95012:
95013:
95014:
95925:
95926:
95927:
96774:
96775:
96776:
```

So these empty lines are existing in our original GFF file. Now, let's remove it:

```sh
sed -i '/^$/d' WAI332_sorted.gff
```

Then, let's check the new file if it has empty lines by running:

```
grep -n "^$" WAI332_sorted.gff
``` 

It returned us nothing which means we don't have empty lines anymore. Now, we can zip this GFF file:

```sh
bgzip -c WAI332_sorted.gff > data.gff.gz
```

Then we can run the `tabix` on our zipped GFF file:

```sh 
tabix -p gff data.gff.gz
```

This time I don't see any errors. I hope you go through this process and understand everything, then you can try to write the correct script to do this. 