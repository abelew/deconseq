# DeconSeq

DECONtamination of SEQuence data using a modified version of BWA-SW
(http://deconseq.sourceforge.net).

For more information about BWA-SW go to http://bio-bwa.sourceforge.net/


## SETUP

1. Create the databases used for contaminat screening

   Use command:

```bash
      bwaXXX index -p database_name -a bwtsw fasta_file_with_ref >out.txt 2>&1 &
```
      (XXX should be replaced by MAC or 64 based on your system architecture)

   For alternative system architectures, download BWA version 0.5.9 source code
   from http://bio-bwa.sourceforge.net/
   (http://sourceforge.net/projects/bio-bwa/files/bwa-0.5.9.tar.bz2/download);
   extract the file with "tar -xjf bwa-0.5.9.tar.bz2"; replace files in the
   extracted BWA directory with the files in bwasw_modified_source (distributed
   with DeconSeq); run "make" in the BWA directory.

   Notes:   (i) It is advised to remove very short or long sequences from files
                retrieved from public resources. Most likely, those sequences
                are a result of misannotations and should not be included in the
                database.
           (ii) Removing ducplicates may reduce the database size and speed up
                analysis. Tools such as PRINSEQ (http://prinseq.sourceforge.net)
                can assist with this task.
          (iii) There seem to be some issues with Mac OSX 10.6.8 that will
                require you to compile the program (the bwaMAC version might
                crash).

2. Change values in DeconSeqConfig.pm (if applicable)

```
   DB_DIR
   TMP_DIR
   OUTPUT_DIR
   PROG_NAME
   PROG_DIR
   DBS
   DBS_DEFAULT
```

   Notes: (i) If the bwaXXX program is located in the same directory as the
              deconseq.pl file, please specify ./ as program directory
              (use: ```constant PROG_DIR => './';```).

3. Setup databases

   Download databases from: ftp://edwards.sdsu.edu:7009/deconseq/db/

   Or create your own database following the steps at:
      http://deconseq.sourceforge.net/manual.html#DB

   In the DeconSeqConfig.pm, specify all your databases as follows:

```perl
use constant DBS => {hsref => { name => 'Human Reference GRCh37',
                                db   => 'hs_ref_GRCh37'},
                        vir => {name => 'Viral genomes',
                                db   => 'virDB'}
                    };
```

   In this example, you have two databases created/downloaded that start with
   hs_ref_GRCh37 and virDB. You can either give them the same name or specify a
   shorter/easier name for calling the databases in DeconSeq. Here, the
   databases are called from DeconSeq with hsref and vir
   (e.g. -dbs hsref -db_retain vir).

   If you have multiple chunks per database (as in the 1GB database chunks on
   the FTP site), you can specify all chunks directly in the config and call
   the database with a single name. In the config file, you would separate the
   database names by commas (no spaces):

```perl
use constant DBS => {hsref => { name => 'Human - Reference GRCh37',
                                db   => 'hs_ref_GRCh37_1,hs_ref_GRCh37_2,hs_ref_GRCh37_3'
                              },
                       vir => { name => 'Viral genomes',
                                  db => 'virDB'
                              }
                    };
```

   Here, the human reference genome is split up into three database chunks. In
   the command line, simply call the database by its name (hsref) and it will
   use the three databases defined in the config.

4. Processing large input files in a cluster environment

   Since mapping is embarrassingly parallel, there was no need to implement a
   parallel processing features into DeconSeq. In other words, splitting up the
   input file into chunks and processing them in parallel on the compute cores
   will get you the same results faster than using the multi-thread option in
   BWA. After processing the chunks, simply concatenate the result files with
   cat. If you have access to a big cluster, you can split large input data into
   smaller chunks and then submit them to the cluster. When done, simply
   concatenate the output files and you are done. (That is how DeconSeq
   processes the data for the web version.)

   A Perl script to split the input data based on file size or number of chunks
   can be found at: https://sourceforge.net/projects/deconseq/files/misc/

   The file is called splitFasta.pl and you can get details on its use with:
   perl splitFasta.pl -h

   Examples:

```bash
perl splitFasta.pl -verbose -i file.fasta -s 2     #chunks of 2MB
perl splitFasta.pl -verbose -i file.fasta -n 10    #10 chunks
```

## USAGE

Run as:

```bash
perl deconseq.pl [options] -f <file> -dbs <list> -dbs_retain <list> ...
```

or rename file and set chmod +x to run as:

```bash
./deconseq [options] -f <file> -dbs <list> -dbs_retain <list> ...
```

Try ```deconseq -h``` for more information on the options.


## DEPENDENCIES

The PERL script requires these other modules:

*   DeconSeqConfig  (included)
*   Data::Dumper
*   Getopt::Long
*   Pod::Usage
*   File::Path      >= 2.07
*   Cwd
*   FindBin


## BUG REPORTS

If you find a bug please email me at <rschmieder_at_gmail_dot_com> so that I can
make DeconSeq better.


## COPYRIGHT AND LICENSE

Copyright (C) 2010-2013  Robert SCHMIEDER

This program is free software: you can redistribute it and/or modify it under
the terms of the GNU General Public License as published by the Free Software
Foundation, either version 3 of the License, or (at your option) any later
version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY
WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE.  See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with
this program.  If not, see <http://www.gnu.org/licenses/>.
