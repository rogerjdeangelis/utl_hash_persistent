# utl_hash_persistent.sas
The results of a HASH lookup are passed via arrays to a DOSUBL  and two separate reports are created.

    ```  SAS-L: Academic: Persistent HASH: Results persist across two reports all in one datastep                                                                     ```
    ```                                                                                                                                                               ```
    ```  For Win 7 workstations and laptops virtual and physical addresses are limited to 44 bits?                                                                    ```
    ```  This means a SAS float can represent these addresses exactly?                                                                                                ```
    ```  Not sure about server operating system address limits.                                                                                                       ```
    ```                                                                                                                                                               ```
    ```   This lack of functionality complicated the solution( serious bug?)                                                                                          ```
    ```                                                                                                                                                               ```
    ```    These work                                                                                                                                                 ```
    ```                                                                                                                                                               ```
    ```     32 bit                                                                                                                                                    ```
    ```      ans=peek (&adrans + (idx-1)*8,8 );                                                                                                                       ```
    ```      que=peekc(&adrque + (idx-1)*15,15);                                                                                                                      ```
    ```                                                                                                                                                               ```
    ```    These do not                                                                                                                                               ```
    ```                                                                                                                                                               ```
    ```     64 bit                                                                                                                                                    ```
    ```      ans=peeklong (&adrans + (idx-1)*8,8 );                                                                                                                   ```
    ```      que=peekclong(&adrque + (idx-1)*15,15);                                                                                                                  ```
    ```                                                                                                                                                               ```
    ```                                                                                                                                                               ```
    ```    WORKING CODE                                                                                                                                               ```
    ```    ============                                                                                                                                               ```
    ```                                                                                                                                                               ```
    ```       LOAD HASH TABLE                                                                                                                                         ```
    ```                                                                                                                                                               ```
    ```          rc=lookUp.defineKey('question');                                                                                                                     ```
    ```          rc=lookUp.defineData('answer');                                                                                                                      ```
    ```          rc=lookUp.defineDone();                                                                                                                              ```
    ```          do until(eof1);                                                                                                                                      ```
    ```            set have end=eof1;                                                                                                                                 ```
    ```            rc=lookUp.add();                                                                                                                                   ```
    ```          end;                                                                                                                                                 ```
    ```                                                                                                                                                               ```
    ```       DO HASH LOOKUP AND LOAD RESULT INTO ARRAYS                                                                                                              ```
    ```       (Variable answer was added)                                                                                                                             ```
    ```                                                                                                                                                               ```
    ```          array que[&_nobs] $15 _temporary_;                                                                                                                   ```
    ```          array ans[&_nobs] _temporary_;                                                                                                                       ```
    ```          do _i_=1 by 1 until ( dne );                                                                                                                         ```
    ```             set lookup end=dne;                                                                                                                               ```
    ```             que[_i_]=question;                                                                                                                                ```
    ```             rc=lookUp.find();                                                                                                                                 ```
    ```             ans[_i_]=answer;                                                                                                                                  ```
    ```          end;                                                                                                                                                 ```
    ```                                                                                                                                                               ```
    ```        DOSUBL (Pass array to DOSUBL)                                                                                                                          ```
    ```                                                                                                                                                               ```
    ```          * workaround for a SAS bug? need blobs?;                                                                                                             ```
    ```                                                                                                                                                               ```
    ```          blobn=peekclong ("&adrAns"x, 8*&_nobs) ;                                                                                                             ```
    ```          blobc=peekclong ("&adrQue"x, 15*&_nobs) ;                                                                                                            ```
    ```                                                                                                                                                               ```
    ```          do _i_=1 to &_nobs;                                                                                                                                  ```
    ```            blob=substr(blobc,(_i_-1)*15+1,15);                                                                                                                ```
    ```            occupation = substr(blob,1,3);                                                                                                                     ```
    ```            state      = substr(blob,4,2);                                                                                                                     ```
    ```            specialty  = substr(blob,6,6);                                                                                                                     ```
    ```            year       = substr(blob,12,4);                                                                                                                    ```
    ```            answer=input(substr(blobn,(_i_-1)*8+1,8),rb8.);                                                                                                    ```
    ```            output;                                                                                                                                            ```
    ```          end;                                                                                                                                                 ```
    ```                                                                                                                                                               ```
    ```          * Two reports;                                                                                                                                       ```
    ```          proc print data=dosub;                                                                                                                               ```
    ```          run;quit;                                                                                                                                            ```
    ```                                                                                                                                                               ```
    ```          ods exclude all;                                                                                                                                     ```
    ```          proc corresp data=dosub dim=1 observed;                                                                                                              ```
    ```          ods output observed=dosub_cnt(rename=label=state);                                                                                                   ```
    ```             tables state, occupation;                                                                                                                         ```
    ```             weight answer;                                                                                                                                    ```
    ```          run;quit;                                                                                                                                            ```
    ```          ods select all;                                                                                                                                      ```
    ```                                                                                                                                                               ```
    ```          proc print data=dosub_cnt;                                                                                                                           ```
    ```          run;quit;                                                                                                                                            ```
    ```                                                                                                                                                               ```
    ```  HAVE                                                                                                                                                         ```
    ```  ====                                                                                                                                                         ```
    ```                                                                                                                                                               ```
    ```  HASH TABLE                                                                                                                                                   ```
    ```                                                                                                                                                               ```
    ```  WORK.HAVE total obs=240                                                                                                                                      ```
    ```                                                                                                                                                               ```
    ```     Obs    QUESTION       ANSWER                                                                                                                              ```
    ```                                                                                                                                                               ```
    ```       1  DOCFLFAMLEE2000    2                                                                                                                                 ```
    ```       2  DOCFLFAMLEE2005   57                                                                                                                                 ```
    ```       3  DOCFLINTMED2000   39                                                                                                                                 ```
    ```       4  DOCFLINTMED2005   61                                                                                                                                 ```
    ```       5  DOCNYFAMLEE2000   70                                                                                                                                 ```
    ```       6  DOCNYFAMLEE2005   92                                                                                                                                 ```
    ```       7  DOCNYINTMED2000   81                                                                                                                                 ```
    ```       8  DOCNYINTMED2005   12                                                                                                                                 ```
    ```       9  DENFLFAMLEE2000   33                                                                                                                                 ```
    ```     ....                                                                                                                                                      ```
    ```     232  DENNYINTMED2005   77                                                                                                                                 ```
    ```     233  NURFLFAMLEE2000   66                                                                                                                                 ```
    ```     234  NURFLFAMLEE2005   24                                                                                                                                 ```
    ```     235  NURFLINTMED2000   34                                                                                                                                 ```
    ```     236  NURFLINTMED2005   78                                                                                                                                 ```
    ```     237  NURNYFAMLEE2000   78                                                                                                                                 ```
    ```     238  NURNYFAMLEE2005   34                                                                                                                                 ```
    ```     239  NURNYINTMED2000   38                                                                                                                                 ```
    ```     240  NURNYINTMED2005    6                                                                                                                                 ```
    ```                                                                                                                                                               ```
    ```                                                                                                                                                               ```
    ```  LOOKUP                                                                                                                                                       ```
    ```                                                                                                                                                               ```
    ```  WORK.LOOKUP total obs=13                                                                                                                                     ```
    ```                                                                                                                                                               ```
    ```  Up to 40 obs WORK.LOOKUP1 total obs=18                                                                                                                       ```
    ```                                                                                                                                                               ```
    ```  Obs    JOB    STATE    SPECIALTY    YEAR       QUESTION                                                                                                      ```
    ```                                                                                                                                                               ```
    ```    1    DOC     FL       FAMLEE      2000    DOCFLFAMLEE2000                                                                                                  ```
    ```    2    NUR     NY       INTMED      2000    NURNYINTMED2000                                                                                                  ```
    ```    3    DEN     NY       FAMLEE      2000    DENNYFAMLEE2000                                                                                                  ```
    ```    4    DOC     NY       FAMLEE      2000    DOCNYFAMLEE2000                                                                                                  ```
    ```    5    DOC     FL       FAMLEE      2000    DOCFLFAMLEE2000                                                                                                  ```
    ```    6    DOC     NY       FAMLEE      2000    DOCNYFAMLEE2000                                                                                                  ```
    ```    7    DEN     NY       INTMED      2000    DENNYINTMED2000                                                                                                  ```
    ```    8    NUR     FL       FAMLEE      2000    NURFLFAMLEE2000                                                                                                  ```
    ```                                                                                                                                                               ```
    ```                                                                                                                                                               ```
    ```                                                                                                                                                               ```
    ```  WANT (Add answer to lookup table and produce two reports)                                                                                                    ```
    ```  ==========================================================                                                                                                   ```
    ```                                                                                                                                                               ```
    ```  WORK.DOSUBL total obs=13                                                                                                                                     ```
    ```                                                                                                                                                               ```
    ```  Obs    OCCUPATION    STATE    SPECIALTY    YEAR    ANSWER                                                                                                    ```
    ```                                                                                                                                                               ```
    ```    1       DOC         FL       FAMLEE      2000       2                                                                                                      ```
    ```    2       DEN         NY       FAMLEE      2005       3                                                                                                      ```
    ```    3       DOC         NY       FAMLEE      2000      70                                                                                                      ```
    ```    4       DOC         NY       FAMLEE      2000      70                                                                                                      ```
    ```    5       DEN         NY       INTMED      2000      70                                                                                                      ```
    ```    6       NUR         NY       INTMED      2000       7                                                                                                      ```
    ```    7       DOC         NY       FAMLEE      2000      70                                                                                                      ```
    ```    8       DEN         FL       FAMLEE      2000      36                                                                                                      ```
    ```    9       DOC         FL       INTMED      2005      61                                                                                                      ```
    ```   10       DOC         NY       INTMED      2000      81                                                                                                      ```
    ```   11       DOC         FL       INTMED      2000      39                                                                                                      ```
    ```   12       DEN         NY       FAMLEE      2005       3                                                                                                      ```
    ```   13       NUR         NY       INTMED      2005       8                                                                                                      ```
    ```                                                                                                                                                               ```
    ```                                                                                                                                                               ```
    ```   WORK.DOSUB_CNT TOTAL OBS=13                                                                                                                                 ```
    ```                                                                                                                                                               ```
    ```   Obs    STATE    DEN    DOC    NUR      SUM                                                                                                                  ```
    ```                                                                                                                                                               ```
    ```    1      FL       36    102      0      138                                                                                                                  ```
    ```    2      NY       76    291     15      382                                                                                                                  ```
    ```                                                                                                                                                               ```
    ```    3      Sum     112    393     15      520                                                                                                                  ```
    ```                                                                                                                                                               ```
    ```  *                _               _       _                                                                                                                   ```
    ```   _ __ ___   __ _| | _____     __| | __ _| |_ __ _                                                                                                            ```
    ```  | '_ ` _ \ / _` | |/ / _ \   / _` |/ _` | __/ _` |                                                                                                           ```
    ```  | | | | | | (_| |   <  __/  | (_| | (_| | || (_| |                                                                                                           ```
    ```  |_| |_| |_|\__,_|_|\_\___|   \__,_|\__,_|\__\__,_|                                                                                                           ```
    ```                                                                                                                                                               ```
    ```  ;                                                                                                                                                            ```
    ```                                                                                                                                                               ```
    ```   * HASH TABLE;                                                                                                                                               ```
    ```                                                                                                                                                               ```
    ```   data have(drop=rep);                                                                                                                                        ```
    ```       length question $15;                                                                                                                                    ```
    ```       do rep=1 to 10;                                                                                                                                         ```
    ```        do job="DOC","DEN","NUR";                                                                                                                              ```
    ```         do State="FL",'NY';                                                                                                                                   ```
    ```           do specialty="FAMLEE","INTMED";                                                                                                                     ```
    ```             do year="2000","2005";                                                                                                                            ```
    ```               question=cats(job,state,specialty,year);                                                                                                        ```
    ```               answer=int(100*uniform(5731));                                                                                                                  ```
    ```               output;                                                                                                                                         ```
    ```             end;                                                                                                                                              ```
    ```           end;                                                                                                                                                ```
    ```         end;                                                                                                                                                  ```
    ```       end;                                                                                                                                                    ```
    ```      end;                                                                                                                                                     ```
    ```  run;quit;                                                                                                                                                    ```
    ```                                                                                                                                                               ```
    ```   * LOOKUP TABLE;                                                                                                                                             ```
    ```   data lookup(drop=rep);                                                                                                                                      ```
    ```       length question $15;                                                                                                                                    ```
    ```       do rep=1 to 10;                                                                                                                                         ```
    ```        do job="DOC","DEN","NUR";                                                                                                                              ```
    ```         do State="FL",'NY';                                                                                                                                   ```
    ```           do specialty="FAMLEE","INTMED";                                                                                                                     ```
    ```             do year="2000","2005";                                                                                                                            ```
    ```               question=cats(job,state,specialty,year);                                                                                                        ```
    ```               if uniform(5731) < .04 then output;                                                                                                             ```
    ```             end;                                                                                                                                              ```
    ```           end;                                                                                                                                                ```
    ```         end;                                                                                                                                                  ```
    ```       end;                                                                                                                                                    ```
    ```      end;                                                                                                                                                     ```
    ```  run;quit;                                                                                                                                                    ```
    ```                                                                                                                                                               ```
    ```  *          _       _   _                                                                                                                                     ```
    ```   ___  ___ | |_   _| |_(_) ___  _ __                                                                                                                          ```
    ```  / __|/ _ \| | | | | __| |/ _ \| '_ \                                                                                                                         ```
    ```  \__ \ (_) | | |_| | |_| | (_) | | | |                                                                                                                        ```
    ```  |___/\___/|_|\__,_|\__|_|\___/|_| |_|                                                                                                                        ```
    ```                                                                                                                                                               ```
    ```  ;                                                                                                                                                            ```
    ```                                                                                                                                                               ```
    ```  data _null_;                                                                                                                                                 ```
    ```                                                                                                                                                               ```
    ```      if _n_=0 then do;                                                                                                                                        ```
    ```         %let rc=%sysfunc(dosubl('                                                                                                                             ```
    ```           data _null_;                                                                                                                                        ```
    ```              set lookup nobs=_nobs;                                                                                                                           ```
    ```              call symputx("_nobs",_nobs);                                                                                                                     ```
    ```           run;quit;                                                                                                                                           ```
    ```         '));                                                                                                                                                  ```
    ```      end;                                                                                                                                                     ```
    ```      length question $15;                                                                                                                                     ```
    ```      declare hash lookUp();                                                                                                                                   ```
    ```      rc=lookUp.defineKey('question');                                                                                                                         ```
    ```      rc=lookUp.defineData('answer');                                                                                                                          ```
    ```      rc=lookUp.defineDone();                                                                                                                                  ```
    ```      do until(eof1);                                                                                                                                          ```
    ```        set have end=eof1;                                                                                                                                     ```
    ```        rc=lookUp.add();                                                                                                                                       ```
    ```      end;                                                                                                                                                     ```
    ```                                                                                                                                                               ```
    ```      array que[&_nobs] $15 _temporary_;                                                                                                                       ```
    ```      array ans[&_nobs] _temporary_;                                                                                                                           ```
    ```      do _i_=1 by 1 until ( dne );                                                                                                                             ```
    ```         set lookup end=dne;                                                                                                                                   ```
    ```         que[_i_]=question;                                                                                                                                    ```
    ```         rc=lookUp.find();                                                                                                                                     ```
    ```         ans[_i_]=answer;                                                                                                                                      ```
    ```      end;                                                                                                                                                     ```
    ```                                                                                                                                                               ```
    ```      adrQue=put(addrlong(que[1]),$hex16.);                                                                                                                    ```
    ```      adrAns=put(addrlong(ans[1]),$hex16.);                                                                                                                    ```
    ```      call symputx('adrQue',adrQue);                                                                                                                           ```
    ```      call symputx('adrAns',adrAns);                                                                                                                           ```
    ```                                                                                                                                                               ```
    ```      rc=dosubl('                                                                                                                                              ```
    ```                                                                                                                                                               ```
    ```      data dosub(keep=occupation -- answer);                                                                                                                   ```
    ```        blobn=peekclong ("&adrAns"x, 8*&_nobs) ;                                                                                                               ```
    ```        blobc=peekclong ("&adrQue"x, 15*&_nobs) ;                                                                                                              ```
    ```        do _i_=1 to &_nobs;                                                                                                                                    ```
    ```          blob=substr(blobc,(_i_-1)*15+1,15);                                                                                                                  ```
    ```          occupation = substr(blob,1,3);                                                                                                                       ```
    ```          state      = substr(blob,4,2);                                                                                                                       ```
    ```          specialty  = substr(blob,6,6);                                                                                                                       ```
    ```          year       = substr(blob,12,4);                                                                                                                      ```
    ```          answer=input(substr(blobn,(_i_-1)*8+1,8),rb8.);                                                                                                      ```
    ```          output;                                                                                                                                              ```
    ```        end;                                                                                                                                                   ```
    ```      run;quit;                                                                                                                                                ```
    ```                                                                                                                                                               ```
    ```      proc print data=dosub;                                                                                                                                   ```
    ```      run;quit;                                                                                                                                                ```
    ```                                                                                                                                                               ```
    ```      ods exclude all;                                                                                                                                         ```
    ```      proc corresp data=dosub dim=1 observed;                                                                                                                  ```
    ```      ods output observed=dosub_cnt(rename=label=state);                                                                                                       ```
    ```         tables state, occupation;                                                                                                                             ```
    ```         weight answer;                                                                                                                                        ```
    ```      run;quit;                                                                                                                                                ```
    ```      ods select all;                                                                                                                                          ```
    ```                                                                                                                                                               ```
    ```      proc print data=dosub_cnt;                                                                                                                               ```
    ```      run;quit;                                                                                                                                                ```
    ```     ');                                                                                                                                                       ```
    ```     stop;                                                                                                                                                     ```
    ```                                                                                                                                                               ```
    ```  run;quit;                                                                                                                                                    ```
    ```                                                                                                                                                               ```
    ```                                                                                                                                                               ```
    ```                                                                                                                                                               ```
    ```                                                                                                                                                               ```
    ```                                                                                                                                                               ```

