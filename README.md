# utl-count-distinct-compound-keys-using-sql-and-hash-algorithms
utl-count-distinct-compound-keys-using-sql-and-hash-algorithms
   %let pgm=utl-count-distinct-compound-keys-using-sql-and-hash-algorithms;

    Count distinct compound keys using sql and hash algorithms

    stackoverflow
    https://tinyurl.com/3jeua6tt
    https://stackoverflow.com/questions/75714051/sas-translate-code-from-proc-sql-regarding-counter-of-distinct-pair-of-variabl

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    /*---- not necessary for the data to be sorted. Sorted solely for documentation purposes ----*/

    data have;
    input key1 key2;
    cards4;
    1  1
    1  2
    1  3
    2  2
    2  3
    3  1
    3  1
    3  2
    3  6
    3  6
    3  6
    3  7
    3  7
    ;;;;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*  Up to 40 obs from last table WORK.HAVE total obs=13 06APR2023:13:27:14                                                */
    /*                                                                                                                        */
    /*  Obs    KEY1    KEY2   |  RULES                                                                                        */
    /*                        |                                                                                               */
    /*    1      1       1    |                                                                                               */
    /*    2      1       2    |                                                                                               */
    /*    3      1       3    |  3 distinct compound keys                                                                     */
    /*                                                                                                                        */
    /*    4      2       2    |                                                                                               */
    /*    5      2       3    |  2 distinct compound keys                                                                     */
    /*                                                                                                                        */
    /*    6      3       1    |                                                                                               */
    /*    7      3       1    |                                                                                               */
    /*    8      3       2    |                                                                                               */
    /*    9      3       6    |                                                                                               */
    /*   10      3       6    |                                                                                               */
    /*   11      3       6    |                                                                                               */
    /*   12      3       7    |                                                                                               */
    /*   13      3       7    |  4 distinct compound keys                                                                     */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*                         _
     ___  __ _ ___   ___  __ _| |
    / __|/ _` / __| / __|/ _` | |
    \__ \ (_| \__ \ \__ \ (_| | |
    |___/\__,_|___/ |___/\__, |_|
                            |_|
    */

    proc sql noprint;
       create
           table want as
       select
           key1
          ,count(distinct key2) as Uniques
       from
           have
       group
           by key1;
    quit;

    /*                              _
    __      ___ __  ___   ___  __ _| |
    \ \ /\ / / `_ \/ __| / __|/ _` | |
     \ V  V /| |_) \__ \ \__ \ (_| | |
      \_/\_/ | .__/|___/ |___/\__, |_|
             |_|                 |_|
    */

    %let work=%sysfunc(pathname(work));

    %utl_submit_wps64("

       options validvarname=any;

       libname wrk '&work';

       proc sql noprint;
          create
              table want as
          select
              key1
             ,count(distinct key2) as Uniques
          from
              wrk.have
          group
              by key1;
       quit;

       proc print;
       run;quit;

    ");

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* The WPS System                                                                                                         */
    /*                                                                                                                        */
    /* Obs    KEY1    Uniques                                                                                                 */
    /*                                                                                                                        */
    /*  1       1        3                                                                                                    */
    /*  2       2        2                                                                                                    */
    /*  3       3        4                                                                                                    */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*               _               _
     ___  __ _ ___  | |__   __ _ ___| |__
    / __|/ _` / __| | `_ \ / _` / __| `_ \
    \__ \ (_| \__ \ | | | | (_| \__ \ | | |
    |___/\__,_|___/ |_| |_|\__,_|___/_| |_|

    */


    data _null_;
       dcl hash h ();
       h.definekey ("key1");
       h.definedata ("key1", "Uniques");
       h.definedone ();

       dcl hash u ();
       u.definekey ("key1", "key2");
       u.definedone ();

       do until (dne);
          set have end = dne;
          if h.find() ne 0 then call missing (Uniques);
          if u.check() ne 0 then do;
             Uniques = sum (Uniques, 1);
             u.add();
          end;
          h.replace();
       end;

       h.output (dataset: "want_sas_hash");
       stop;
    run;

    /*                    _               _
    __      ___ __  ___  | |__   __ _ ___| |__
    \ \ /\ / / `_ \/ __| | `_ \ / _` / __| `_ \
     \ V  V /| |_) \__ \ | | | | (_| \__ \ | | |
      \_/\_/ | .__/|___/ |_| |_|\__,_|___/_| |_|
             |_|
    */

    %let work=%sysfunc(pathname(work));

    %utl_submit_wps64("

       options validvarname=any;

       libname wrk '&work';

     data _null_;
       dcl hash h ();
       h.definekey ('key1');
       h.definedata ('key1', 'Uniques');
       h.definedone ();

       dcl hash u ();
       u.definekey ('key1', 'key2');
       u.definedone ();

       do until (dne);
          set wrk.have end = dne;
          if h.find() ne 0 then call missing (Uniques);
          if u.check() ne 0 then do;
             Uniques = sum (Uniques, 1);
             u.add();
          end;
          h.replace();
       end;

       h.output (dataset: 'want_sas_hash');
       stop;
      run;

      proc print data=want_sas_hash;
      run;quit;

    ");


    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*  The WPS System                                                                                                        */
    /*                                                                                                                        */
    /*  Obs    KEY1    Uniques                                                                                                */
    /*                                                                                                                        */
    /*   1       1        3                                                                                                   */
    /*   2       3        4                                                                                                   */
    /*   3       2        2                                                                                                   */
    /*                                                                                                                        */
    /**************************************************************************************************************************/


    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
