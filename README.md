# CBT553
Converted to GitHub via [cbt2git](https://github.com/wizardofzos/cbt2git)

This is still a work in progress. 
Due to amazing work by Alison Zhang and Jake Choi repos are no longer deleted.

```
//***FILE 553 is from David Alcock and contains a sophisticated     *   FILE 553
//*           edit macro, written in Assembler, for submitting      *   FILE 553
//*           jobs from TSO.  There are many bells and whistles     *   FILE 553
//*           in this program, and especially if you need to enter  *   FILE 553
//*           passwords from within a jobstream, this program is    *   FILE 553
//*           worth looking into.                                   *   FILE 553
//*                                                                 *   FILE 553
//*           email:   dave@planetmvs.com                           *   FILE 553
//*                                                                 *   FILE 553
//*     Edit macro that submits the member (or selected lines).     *   FILE 553
//*     and does inserts of passwords and translates of system      *   FILE 553
//*     symbolics.                                                  *   FILE 553
//*                                                                 *   FILE 553
//*     SUBMITC is NOT robust.  It was designed for specific        *   FILE 553
//*     types of jobstreams, not every possibility is supported.    *   FILE 553
//*     Your mileage may vary.                                      *   FILE 553
//*                                                                 *   FILE 553
//*     - - - - - - - - - - - - - - - - - - - - - - - - - - - -     *   FILE 553
//*                                                                 *   FILE 553
//*     Information on the SUBMITC ISPF edit macro - 2002-02-15     *   FILE 553
//*     -------------------------------------------------------     *   FILE 553
//*                                                                 *   FILE 553
//*     SUBMITC is an ISPF edit macro that submits the file (or     *   FILE 553
//*     selected lines) to the JES2/JES3 internal reader.           *   FILE 553
//*                                                                 *   FILE 553
//*     Oh big deal you say, but wait there's more:                 *   FILE 553
//*                                                                 *   FILE 553
//*     Passwords:                                                  *   FILE 553
//*     ---------                                                   *   FILE 553
//*     When reading the input, it looks for certain password       *   FILE 553
//*     locations in the jobsteam and prompts you for them so       *   FILE 553
//*     you don't have keep passwords in datasets (bad) and         *   FILE 553
//*     don't have to remember to cancel out or change the          *   FILE 553
//*     password before saving.                                     *   FILE 553
//*                                                                 *   FILE 553
//*     Passwords are specifed as "?" and are only looked for in    *   FILE 553
//*     specific locations. See Process_Line for more details.      *   FILE 553
//*                                                                 *   FILE 553
//*     Passwords are put into the jobstream asis (mixed case)      *   FILE 553
//*     except for jobcard passwords which are folded to            *   FILE 553
//*     uppercase.                                                  *   FILE 553
//*                                                                 *   FILE 553
//*     For passwords on the jobcard, USER= and PASSWORD= must      *   FILE 553
//*     be on the same line and in that order.  In this case, I     *   FILE 553
//*     will append a comma after the password if found in the      *   FILE 553
//*     input stream otherwise I just overlay password beginning    *   FILE 553
//*     at the question mark.                                       *   FILE 553
//*                                                                 *   FILE 553
//*     You must leave enough room after the "?" where the          *   FILE 553
//*     password is to go because it is *assumed* the jobstream     *   FILE 553
//*     was prepped for SUBMITC usage.                              *   FILE 553
//*                                                                 *   FILE 553
//*     Usually you will need to put an exclamation point before    *   FILE 553
//*     SUBMITC so ISPF will find it.  For example:                 *   FILE 553
//*                                                                 *   FILE 553
//*     EDIT       IBMUSER.IN.CNTL(TCPFTP)                          *   FILE 553
//*     Command ===> !SUBMITC                                       *   FILE 553
//*     ****** ********************************                     *   FILE 553
//*     000001 //IBMUSERF  JOB  (ACCT#),'MY NAM                     *   FILE 553
//*                                                                 *   FILE 553
//*     An allocation to the JES INTRDR is dynamically allocated    *   FILE 553
//*     and SUBMITC writes directly from the ISPF line variables    *   FILE 553
//*     to the JES input queue.  We do NOT write any data to a      *   FILE 553
//*     DASD dataset (unlike IBM's ISPF submit). WHO HA!            *   FILE 553
//*                                                                 *   FILE 553
//*     Sample jobstream showing the passwords we try to detect     *   FILE 553
//*     and change:                                                 *   FILE 553
//*                                                                 *   FILE 553
//*       //IBMUSERF  JOB  (ACCT#),'MY NAME HERE',                  *   FILE 553
//*       //          USER=IBMUSER,PASSWORD=?,                 (1)  *   FILE 553
//*       //          MSGCLASS=X,CLASS=U                            *   FILE 553
//*       //*                                                       *   FILE 553
//*       //PS0130  EXEC PGM=FTP,PARM='FIREWALL (EXIT TI 720'       *   FILE 553
//*       //NETRC     DD *                                          *   FILE 553
//*       MACHINE FIREWALL LOGIN ibmuser PASSWORD ?            (2)  *   FILE 553
//*       //SYSPRINT  DD SYSOUT=*                                   *   FILE 553
//*       //OUTPUT    DD SYSOUT=*                                   *   FILE 553
//*       //INPUT     DD *                                          *   FILE 553
//*       user bozo@clown.net                                  (3)  *   FILE 553
//*       ?                                                    (3)  *   FILE 553
//*       dir                                                       *   FILE 553
//*       quit                                                      *   FILE 553
//*                                                                 *   FILE 553
//*       (1) USER=user,PASSWORD=?  on the jobcard                  *   FILE 553
//*       (2) Instream machine statements in //NETRC file           *   FILE 553
//*       (3) Instream user statements followed by ? in //INPUT     *   FILE 553
//*           file                                                  *   FILE 553
//*                                                                 *   FILE 553
//*     Symbols                                                     *   FILE 553
//*     -------                                                     *   FILE 553
//*     IBM currently does support system symbolics in batch        *   FILE 553
//*     jobs.  You can optionally use SUBMITC to translate          *   FILE 553
//*     system symbolics in your instream jobstream using two       *   FILE 553
//*     methods:                                                    *   FILE 553
//*                                                                 *   FILE 553
//*        1) Use the parm SYM when invoke SUBMITC to translate     *   FILE 553
//*           all lines (or until a SUBMITC::NOSYM control card     *   FILE 553
//*           is reached).                                          *   FILE 553
//*                                                                 *   FILE 553
//*        2) Use the special control cards in the jobstream to     *   FILE 553
//*           turn on and off translation.  Use the SYM or NOSYM    *   FILE 553
//*           on one of the 3 types of control card formats:        *   FILE 553
//*                                                                 *   FILE 553
//*           //*SUBMITC::SYM                                       *   FILE 553
//*           /* SUBMITC::SYM                                       *   FILE 553
//*           *SUBMITC::SYM                                         *   FILE 553
//*                                                                 *   FILE 553
//*           Line(s) to translate with symbolics here              *   FILE 553
//*                                                                 *   FILE 553
//*           *SUBMITC::SYM                                         *   FILE 553
//*           /* SUBMITC::SYM                                       *   FILE 553
//*           //*SUBMITC::SYM                                       *   FILE 553
//*                                                                 *   FILE 553
//*     See member $SAMPSYM for a job that has symbolics.           *   FILE 553
//*                                                                 *   FILE 553
//*     BTW: SUBMITC calls the IBM system symbolic routine          *   FILE 553
//*     ASASYMBM to do the translation.                             *   FILE 553
//*                                                                 *   FILE 553
```
