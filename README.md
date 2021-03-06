# utl-shift-up-non-missing-values-read-backwards-dow
Shift up non missing values read backwards dow.
    Shift up non missing values read backwards dow
    
      A grand tour of the interaction of mutiple set statements, HASH, array, do until/while ,                                                 set point and read backwards.       

       WOW see #7 for Mark Keintz five statement solution below
       
       SYNOPSIS
       set have (keep=var2) end=end_of_have;                                                             
       if var2^=. or end_of_have then                                                                    
         do _i=1 to coalesce(dif(_n_),_n_);                                                              
            set have ;                                                                        
         output;                                                                                         
       end; 
       
       The dif function calculates the number of '.'s .                                                  
       Dropping var2 retains the non-missing.                                                            
       Very minor nickpick. I think you can remove the 'drop=var2?
       
       All Solutions
       
       1. DOW datastep - read ahead
       2. read backward then carry forward and fip again
       3  Interpolating values in a timeseries when some first,last and middle values are missing
          see https://goo.gl/iQn3nU
          https://communities.sas.com/t5/SAS-Procedures/shifts-up-not-missing-variables/m-p/515208          
       4. HASH soultion
       5. Array Solution
       6. Paul Dorfmans simple elegant dow solution (on end - final with enhancement)
       
       7. Five statement solution Dif function and interaction of set statements (brilliant) by            
          Keintz, Mark                                                                                     
          mkeintz@wharton.upenn.edu   
       
    Who knew it could could be done in 5 lines of code(#7)                                              
                                                                                                        
      Keintz, Mark                                                                                      
      mkeintz@wharton.upenn.edu                                                                         
                                                                                                        
      set have (keep=var2) end=end_of_have;                                                             
      if var2^=. or end_of_have then                                                                    
        do _i=1 to coalesce(dif(_n_),_n_);                                                              
           set have (drop=var2);                                                                        
        output;                                                                                         
      end;                                                                                              
                                                                                                        
      The dif function calculates the number of '.'s .                                                  
      Dropping var2 retains the non-missing.                                                            
      Very minor nickpick. I think you can remove the 'drop=var2?                                                           
                               
     Recent solutions on end by

     Bartosz Jablonski
     yabwon@gmail.com
    
     Paul Dorfman
     sashole@bellsouth.net

     Thanks to the Op, Paul,  Bartosz and the op for a interesting question
     and innovative answers.

     See Paul Dorfman's 'best?' solution on end

     Paul Dorfman
     sashole@bellsouth.net

     Nice Paul

     The simplest and elegant solution is odten the best.

     I made a slight change to Pauls solution to cover the 0 case(var2 ne .).
     Note 'do while' does not work, until gets the 'next' value
     but does not go back and increment _n_ even though it has the
     next var2.

    This is a nice example of the difference between
    'do until' and 'do while.

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
    
        *____             _
    | __ )  __ _ _ __| |_
    |  _ \ / _` | '__| __|
    | |_) | (_| | |  | |_
    |____/ \__,_|_|   \__|

    ;

    Recent solutions on end by

    Bartosz Jablonski
    yabwon@gmail.com

     4. HASH soultion
     5. Array Solution


    4. HASH soultion
    ----------------

    data _null_;
      declare hash H(ordered:"a");
      H.defineKey("curobs");
      H.defineData("curobs");
      H.defineData("var1");
      H.defineData("var2");
      H.defineDone();
      declare hiter I("H");

      do until(EOF);
       set have end=EOF curobs=curobs;
       H.add();
      end;

      do while(I.prev()=0);
        if var2 ne . then _t2 = var2;
                     else var2 = _t2;
        H.replace();
      end;

      H.output(dataset:"want(drop=curobs)");
      stop;
    run;


    5. Array Solution
    -----------------

    data _null_;
      set have nobs=nobs;
      call symputX("nobs", nobs, "G");
      stop;
    run;

    data want2;
      array arr1_[&nobs.] _temporary_;
      array arr2_[&nobs.] _temporary_;

      do until(EOF);
       set have end=EOF curobs=curobs;
       arr1_[curobs] = var1;
       arr2_[curobs] = var2;
      end;

      do _N_ = dim(arr1_) to 1 by -1; drop _t2;
        if arr2_[_N_] ne . then _t2 = arr2_[_N_];
                           else arr2_[_N_] = _t2;
      end;

      do _N_ = 1 to dim(arr1_) by 1;
        var1 = arr1_[_N_];
        var2 = arr2_[_N_];
        output;
      end;

      stop;
    run;


    *____             _
    |  _ \ __ _ _   _| |
    | |_) / _` | | | | |
    |  __/ (_| | |_| | |
    |_|   \__,_|\__,_|_|

    ;

        5. Paul Dorfmans simple elegant dow solution

     Thanks to the Op, Paul,  Bartosz and the op for a interesting question
    and innovative answers.

    See Paul Dorfman's 'best?' solution on end

    Paul Dorfman
    sashole@bellsouth.net

    Nice Paul

    The simplest and elegant solution is odten the best.

    I made a slight change to Pauls solution to cover the 0 case(var2 ne .).
    Note 'do while' does not work, until gets the 'next' value
    but does not go back and increment _n_ even though it has the
    next var2.

    This is a nice example of tje difference between
    'do until' and 'do while.


    data have ;
     input var1 var2 ;
    cards ;
    1 .
    2 .
    4 .
    5 4
    3 9
    3 .
    3 .
    6 0
    run ;

    data want (drop = _:) ;
      do _n_ = 1 by 1 until (var2 ne .) ;
        set have ;
      end ;
      _fill = var2 ;
      do _n_ = 1 to _n_ ;
        set have ;
        var2 = _fill ;
        output ;
      end ;
    run ;

    * enhanced;
    
        Thanks! and I agree with your amendment.
    Perhaps for generality, NOT MISSING() would be even better,
    since it makes the code independent of the var2 data type.
    Nitpicking further, it's possible to both make the step a
    bit more efficient and still terser, plus retain the
    (var1, var2) original PDV order:

    data have ;
     input var1 var2 ;
    cards ;
    1 .
    2 .
    4 .
    5 4
    3 9
    3 .
    3 .
    6 5
    run ;

    data want (drop = _:) ;
      do _n_ = 1 by 1 until (not missing (_fill)) ;
        set have (rename=(var2=_fill)) ;
      end ;
      do _n_ = 1 to _n_ ;
        set have (drop=var2) ;
        var2 = _fill ;
        output ;
      end ;
    run ;


    *__  __            _                                                                                
    |  \/  | __ _ _ __| | __                                                                            
    | |\/| |/ _` | '__| |/ /                                                                            
    | |  | | (_| | |  |   <                                                                             
    |_|  |_|\__,_|_|  |_|\_\                                                                            
                                                                                                        
    ;                                                                                                   
                                                                                                        
                                                                                                        
    A grand tour of the interaction of mutiple set statements, HASH, array, do until/while ,            
    set point and read backwards.                                                                     
                                                                                                        
    github                                                                                              
                                                                                                    
    WOW Mark                                                                                            
                                                                                                        
    7. Five statement solution Dif function and interaction of set statements (brilliant) by            
       Keintz, Mark                                                                                     
       mkeintz@wharton.upenn.edu   
       
      Who knew it could could be done in 5 lines of code(#7)                                              
                                                                                                        
      Keintz, Mark                                                                                      
      mkeintz@wharton.upenn.edu                                                                         
                                                                                                        
      set have (keep=var2) end=end_of_have;                                                             
      if var2^=. or end_of_have then                                                                    
        do _i=1 to coalesce(dif(_n_),_n_);                                                              
           set have (drop=var2);                                                                        
        output;                                                                                         
      end;                                                                                              
                                                                                                        
      The dif function calculates the number of '.'s .                                                  
      Dropping var2 retains the non-missing.                                                            
      Very minor nickpick. I think you can remove the 'drop=var2?                                                           
                                                                                                        
                                                                                                        
      So is the DIF function, even when evaluated only                                                  
      when an if condition is satisfied.                                                                
                                                                                                        
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
     6 .                                                                                                
     7 0                                                                                                
    ;;;;                                                                                                
    run;                                                                                                
                                                                                                        
    data want (drop=_i);                                                                                
      if 0 then set have;             /*Just to preserve variable order */                              
      set have (keep=var2) end=end_of_have;                                                             
      if var2^=. or end_of_have then do _i=1 to coalesce(dif(_n_),_n_);                                 
        set have (drop=var2);                                                                           
            output;                                                                                     
      end;                                                                                              
    run;                                                                                                
                                                                                                        
    data want (drop=_i);                                                                                
      if 0 then set have;             /*Just to preserve variable order */                              
      set have  end=end_of_have;                                                                        
      if var2^=. or end_of_have then do _i=1 to coalesce(dif(_n_),_n_);                                 
        set have (drop=var2);                                                                           
            output;                                                                                     
      end;                                                                                              
    run;                                                                                                
                                                                                                        
    The "or end_of_have" condition is to accommodate missing values at the end of have.                 
                                                                                                        
                                                                                                        
                                                                                                        
                                                                                                        



