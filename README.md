# utl-shift-up-non-missing-values-read-backwards-dow
Shift up non missing values read backwards dow.
    Shift up non missing values read backwards dow

       1. DOW datastep - read ahead
       2. read backward then carry forward and fip again
       3  Interpolating values in a timeseries when some first,last and middle values are missing
          see https://goo.gl/iQn3nU
          https://communities.sas.com/t5/SAS-Procedures/shifts-up-not-missing-variables/m-p/515208

    github
    https://tinyurl.com/ycwdt2kf
    https://github.com/rogerjdeangelis/utl-shift-up-non-missing-values-read-backwards-dow

    SAS Forum
    https://communities.sas.com/t5/SAS-Procedures/shifts-up-not-missing-variables/m-p/515208

    R has a function to do this lead and lag?


    INPUT
    =====

     WORK.HAVE total obs=8

                    |  RULES
     VAR1    VAR2   |  VAR2
                    |
       1       .    |   4
       2       .    |   4
       4       .    |   4
       5       4    |   4  ** carry 4 backwards

       3       9    |   9  ** leave alone

       3       .    |   5
       3       .    |   5
       6       5    |   5  ** carry 5 backwards
                    |

    EXAMPLE OUTPUT
    ---------------

    WORK WANT total obs=8

     VAR1    VAR2

       1       4
       2       4
       4       4
       5       4
       3       9
       3       5
       3       5
       6       5



    PROCESS
    =======


    1. DOW datastep - read ahead
    -----------------------------

    data want;

      retain var2Sav pnt 0;

      do until (last.var2);

        set have;

        by var2 notsorted;

        pnt=pnt+1;

        if last.var2 and var2=. then do;
           * get value after last missing;
           pnt=pnt+1;
           set have point=pnt;
           var2Sav=var2;
           pnt=pnt-1; * RESET FOR NEXT ITERATIONS;
        end;

       end;

       do until (last.var2);
          set have;
          by var2 notsorted;
          if var2=. then var2=Var2Sav;
          output;
       end;

       var2Sav=0;

    run;quit;


    2. read backward then carry forward and fip again
    --------------------------------------------------

    proc datasets lib=work;
     delete havRev havFil want:;
    run;quit;

    data want(drop=var2 rename=varSav=var2);

      * read backards and carr forward;
      if _n_=0 then do; %let rc=%sysfunc(dosubl('
         data havRev; * unable to mahe this a view;
            if _n_=0 then set have nobs=numObs;
            set have point=numObs;
            if numObs=0 then stop;
            output;
            numObs=numObs-1;
         run;quit;
         data havFil;
           retain varSav;
           set havRev end=dne;
           if var2 ne . then varSav=var2;
        run;quit;
        '));
      end;

      if _n_=0 then set have nobs=numObs;

      set havFil point=numObs;
      if numObs=0 then stop;
      output;

      numObs=numObs-1;

    run;quit;


    3  Interpolating values in a timeseries when some first,last and middle values are missing
    -------------------------------------------------------------------------------------------

       see https://goo.gl/iQn3nU


    OUTPUT
    ======

    see above

    *                _               _       _
     _ __ ___   __ _| | _____     __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \   / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/  | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|   \__,_|\__,_|\__\__,_|

    ;

    data have;
     input var1 var2;
    cards4;
    1 .
    2 .
    4 .
    5 4
    3 9
    3 .
    3 .
    6 5
    ;;;;
    run;quit;


