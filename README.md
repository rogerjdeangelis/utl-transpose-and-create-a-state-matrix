# utl-transpose-and-create-a-state-matrix
    Transpose and create a a true false state matrix

     Three Solutions

         1. Single proc report.
         2. Proc corresp then sql
         3. View and transpose

    github
    https://github.com/rogerjdeangelis/utl-transpose-and-create-a-state-matrix

    SAS-L
    https://listserv.uga.edu/cgi-bin/wa?A2=SAS-L;eaee8806.1902a

    INPUT
    =====

    data have;
    informat id $1.  project_name $5.;
    input id project_name ;
    cards4;
    1 CLAIM
    2 CLAIM
    3 RETRO
    4 IHA
    5 CLAIM
    ;;;;
    run;quit;

    /*

     WORK.HAVE total obs=5

            PROJECT_
      ID      NAME

      1      CLAIM
      2      CLAIM
      3      RETRO
      4      IHA
      5      CLAIM
     */

    RULES
    =====

    Proc report  cross tab of project_name x project_name

    EXAMPLE OUTPUT
    --------------

    WORK.WANT total obs=5

      ID    CLAIM    RETRO    IHA

      1       1        0       0
      2       1        0       0
      3       0        0       1
      4       0        1       0
      5       1        0       0


    PROCESS
    =======


    1. Single proc report.
    ---------------------

    * bug in ods output. Names are not honored;
    proc report data=have missing nowd out=want (rename=(
      _c2_=claim _c3_=retro _c4_=iha));;
    cols  id project_name  /*project_name=pname* calc */;
    define id / group;
    define project_name/group;
    define project_name/across;
    define project_name/across;
    run;quit;



    Up to 40 obs from WANT total obs=5

    Obs    ID    _C2_    _C3_    _C4_    _BREAK_

     1     1       1       0       0
     2     2       1       0       0
     3     3       0       0       1
     4     4       0       1       0
     5     5       1       0       0



    2. Proc corresp then sql
    --------------------------

    ods exclude all;
      ods output rowprofilespct=wantpre ;
    proc corresp data=have dim=1 observed missing all print=both;
    tables project_name, project_name;
    run;quit;
    ods select all;

    /* intermediate result
    WORK.WANTPRE total obs=3

      LABEL    CLAIM    IHA    RETRO

      CLAIM     100       0       0
      IHA         0     100       0
      RETRO       0       0     100

    */

    proc sql;
       create
          table want(drop=label) as
       select
          l.id
         ,l.project_name
         ,r.*
       from
         have as l left join wantpre as r
       on
         l.project_name = r.label
       order
         by id
    ;quit;

    /*
    Percentage state matrix

     WORK.WANT total obs=5

            PROJECT_
      ID      NAME      CLAIM    IHA    RETRO

      1      CLAIM       100       0       0
      2      CLAIM       100       0       0
      3      RETRO         0       0     100
      4      IHA           0     100       0
      5      CLAIM       100       0       0
    */




    data havVue/view=havVue;
      set have;
      val=1;
    run;quit;

    proc transpose data=havVue out=want(drop=_name_ project_name);
    by id project_name;
    id project_name;
    var val;
    run;quit;

    WORK.WANT total obs=5

       ID    CLAIM    RETRO    IHA

       1       1        .       .
       2       1        .       .
       3       .        1       .
       4       .        .       1
       5       1        .       .


