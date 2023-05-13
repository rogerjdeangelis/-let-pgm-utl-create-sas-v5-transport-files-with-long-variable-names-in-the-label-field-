# -let-pgm-utl-create-sas-v5-transport-files-with-long-variable-names-in-the-label-field-
Create sas v5 transport files with long variable names in the label field   
    %let pgm=utl-create-sas-v5-transport-files-with-long-variable-names-in-the-label-field;

    Ptthon: create sas v5 transport files with long variable names in the label field

    github
    https://tinyurl.com/35xuwu2y
    https://github.com/rogerjdeangelis/-let-pgm-utl-create-sas-v5-transport-files-with-long-variable-names-in-the-label-field-

    Switch the xport labels with the xport variable names

    I could not figure out how to create a V5 transport file using pyreadstat, only v8 transport file.
    V5 is more universal. I also provide sas code to rename the 8 character names using the xport label.

    I also prefer not to use the sas only xpt2loc macro to read v8 transport datasets,

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    libname sd1 "d:/sd1";
    data sd1.have;

      input
        Sequence_by_10_units
        Students_in_Math_Class $
      ;

    cards4;
    10 Roger
    20 Jamed
    30 Mary
    ;;;;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* Up to 40 obs from SD1.HAVE total obs=3 12MAY2023:16:23:51                                                              */
    /*                                                                                                                        */
    /*        SEQUENCE_    STUDENTS_                                        Variables in Creation Order                       */
    /*          BY_10_     IN_MATH_                                                                                           */
    /* Obs      UNITS        CLASS                                  #    Variable                  Type    Len                */
    /*                                                                                                                        */
    /*  1         10         Roger                                  1    SEQUENCE_BY_10_UNITS      Num       8                */
    /*  2         20         Jamed                                  2    STUDENTS_IN_MATH_CLASS    Char      8                */
    /*  3         30         Mary                                                                                             */
    /*                                                                                                                        */
    /**************************************************************************************************************************/
    /*           _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| `_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    */

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*                                                                                                                        */
    /*  THE CONTENTS PROCEDURE XPORT FILE                                                                                     */
    /*                                                                                                                        */
    /*  Data Set Name        XPT.WANT                                    Observations          .                              */
    /*  Member Type          DATA                                        Variables             2                              */
    /*  Engine               XPORT                                       Indexes               0                              */
    /*                                                                                                                        */
    /*        Alphabetic List of Variables and Attributes                                                                     */
    /*                                                                                                                        */
    /*  #    Variable    Type    Len    Label                                                                                 */
    /*                                                                                                                        */
    /*  1    SEQUENCE    Num       8    SEQUENCE_BY_10_UNITS                                                                  */
    /*  2    STUDENTS    Char      5    STUDENTS_IN_MATH_CLASS                                                                */
    /*                                                                                                                        */
    /*   dataset xpt.want                                                                                                     */
    /*                                                                                                                        */
    /*   Obs    SEQUENCE    STUDENTS                                                                                          */
    /*                                                                                                                        */
    /*     1       10        Roger                                                                                            */
    /*     2       20        Jamed                                                                                            */
    /*     3       30        Mary                                                                                             */
    /*                                                                                                                        */
    /*  AFTER USING THE LABELS TO RENAME THE VARIABLES                                                                        */
    /*                                                                                                                        */
    /*  Data Set Name        WORK.WANT_R_LONG_NAMES        Observations          3                                            */
    /*                                                                                                                        */
    /*                                                                                                                        */
    /*                 Variables in Creation Order                                                                            */
    /*                                                                                                                        */
    /*   #    Variable                  Type    Len    Label                                                                  */
    /*                                                                                                                        */
    /*   1    SEQUENCE_BY_10_UNITS      Num       8    SEQUENCE ** THESE ARE THE ORIGINAL XPORT FILE VARIABLE NAMES           */
    /*   2    STUDENTS_IN_MATH_CLASS    Char      5    STUDENTS                                                               */
    /*                                                                                                                        */
    /*                                                                                                                        */
    /*  FINAL SAS TABLE                                                                                                       */
    /*                                                                                                                        */
    /*  Up to 40 obs from WANT_R_LONG_NAMES total obs=3 12MAY2023:17:43:01                                                    */
    /*                                                                                                                        */
    /*         SEQUENCE_    STUDENTS_                                                                                         */
    /*           BY_10_     IN_MATH_                                                                                          */
    /*  Obs      UNITS        CLASS                                                                                           */
    /*                                                                                                                        */
    /*   1         10         Roger                                                                                           */
    /*   2         20         Jamed                                                                                           */
    /*   3         30         Mary                                                                                            */
    /*                                                                                                                        */
    */ /*************************************************************************************************************************

    /*         _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _ \| | | | | __| |/ _ \| `_ \
    \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|

    */

    proc datasets lib=work mt=data mt=view;
      delete want want_r_long_names namLbl;
    run;quit;

    %utlfkil(d:/xpt/want.xpt);

    %utl_pybegin;
    parmcards4;
    import pandas as pd
    import xport
    import xport.v56
    import numpy as np
    import pyreadstat as ps
    df, meta  = ps.read_sas7bdat("d:/sd1/have.sas7bdat")
    print(df)
    lbl = list(df.columns.values)
    df  = df.rename(columns={k: k.upper()[:8] for k in df});
    print(lbl);
    print(df);
    ds = xport.Dataset(df, name='want')
    idx=-1;
    for k, v in ds.items():
        idx=idx+1
        print(idx)
        v.label = lbl[idx]
    print(ds)
    library = xport.Library({'want': ds})
    with open('d:/xpt/want.xpt', 'wb') as f:
        xport.v56.dump(library, f)
    ;;;;
    %utl_pyend;

    libname xpt xport "d:/xpt/want.xpt";
    ods output variables=namLbl;
    proc contents data=xpt._all_;
    run;quit;

    %array(_var _lbl,data=namLbl,var=variable label);

    %put &_var1;
    %put &_varn;

    %put &_lbl1;
    %put &_lbln;

    proc print data=xpt.want;
    run;quit;

    data want_r_long_names; /*---- cannot use name want because a view named want exists ----*/
      label
         %do_over(_var _lbl,phrase=%str(?_lbl = "?_var"))
      ;
      %utl_rens(xpt.want) ;
      set want;
    run;quit;

    /*---- if you want the generated code ----*/
    %utlnopts;
    data _null_;
    %do_over(_var _lbl,phrase=%str(put "?_lbl = '?_var'";))
    ;run;quit;
    %utlopts;

    /*---- SEQUENCE_BY_10_UNITS = 'SEQUENCE'   ----*/
    /*---- STUDENTS_IN_MATH_CLASS = 'STUDENTS' ----*/

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
