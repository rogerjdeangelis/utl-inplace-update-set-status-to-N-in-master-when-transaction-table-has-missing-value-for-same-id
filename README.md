# utl-inplace-update-set-status-to-N-in-master-when-transaction-table-has-missing-value-for-same-id
Inplace update set status to N in master when transaction table has missing value for same idInplace update set status to N in master when transaction table has missing value for same id
    %let pgm=utl-inplace-update-set-status-to-N-in-master-when-transaction-table-has-missing-value-for-same-id;

    %stop_submission;

    Inplace update set status to N in master when transaction table has missing value for same id

         CONTENTS
            1 sas sql left join
            2 sas sql update
            3 r sql left join
              sqldf does supprt update but I could not figuer it out.
              also update is less usefull when dropping down to sqldf
              because you probably want a r dataframe
              same code in python and sql
              see
              https://tinyurl.com/4e6yaap8

    inplace update set status to N in master when transaction table has missing for same id

    SOAPBOX ON
    No need for two sorts and a merge?
    All soltions with a single sql query;
    SOAPBOX OFF;

    github
    https://tinyurl.com/bdhxz2tn
    https://github.com/rogerjdeangelis/utl-inplace-update-set-status-to-N-in-master-when-transaction-table-has-missing-value-for-same-id

    communities.sas
    https://tinyurl.com/2rwvjpjb
    https://communities.sas.com/t5/SAS-Programming/Update-one-dataset-with-same-value-for-selected-field-for-only/m-p/808451#M318787

    /**************************************************************************************************************************/
    /*         INPUT                |         PROCESS                                |     OUTPUT                             */
    /*                              |  BEST WAY TO DOCUMENT SHOW SQL                 |                                        */
    /*                              |                                                |                                        */
    /* SD1.MASTER obs=4             |  Left join master to  trans on id              |    ID   TYPE STATUS                    */
    /*                              |  choose non mssing value( "N",trans.status)    |                                        */
    /*     ID      TYPE    STATUS   |                                                | 1234567 Pub    N missing in trans      */
    /*                              |  select                                        | 1245678 Pub    N missing in trans      */
    /*  1234567    Pub              |     m.id                                       | 2349878 Pri    N                       */
    /*  1245678    Pub              |    ,m.type                                     | 3948573 Priv   Y                       */
    /*  2349878    Pri       N      |    ,coalescec("N",m.status,) as status         |                                        */
    /*  3948573    Priv      Y      |  from                                          |                                        */
    /*                              |     master as m left join trans as t           |                                        */
    /* SD1.TRANS obs=2              |  on                                            |                                        */
    /*                              |    m.id = t.id                                 |                                        */
    /*     ID      TYPE             |                                                |                                        */
    /*                              |  All trans record are missing in this case     |                                        */
    /*  1234567    Pub              |                                                |                                        */
    /*  1245678    Pub              |                                                |                                        */
    /*                              |-----------------------------------------------------------------------------------------*/
    /* options validvarname=upcase; | 1 SAS SQL LEFT JOIN                            |   ID       TYPE    STATUS              */
    /* libname sd1 "d:/sd1";        | ====================                           |                                        */
    /* data sd1.master;             |                                                | 1234567    Pub       N                 */
    /*    input id type$ status$;   | proc sql;                                      | 1245678    Pub       N                 */
    /* cards4;                      |  create                                        | 2349878    Pri       N                 */
    /* 1234567 Pub .                |     table want as                              | 3948573    Priv      Y                 */
    /* 1245678 Pub .                |  select                                        |                                        */
    /* 2349878 Pri N                |     m.id                                       |                                        */
    /* 3948573 Priv Y               |    ,m.type                                     |                                        */
    /* ;;;;                         |    ,coalescec(m.status,"N") as status          |                                        */
    /* run;quit;                    |  from                                          |                                        */
    /*                              |     master as m left join trans as t           |                                        */
    /* data sd1.trans;              |  on                                            |                                        */
    /*    input id type$;           |    m.id = t.id                                 |                                        */
    /* cards4;                      | ;quit;                                         |                                        */
    /* 1234567 Pub                  |                                                |                                        */
    /* 1245678 Pub                  |-----------------------------------------------------------------------------------------*/
    /* ;;;;                         | 2 SAS SQL UPDATE                               |   ID       TYPE    STATUS              */
    /* run;quit;                    | ================                               |                                        */
    /*                              |                                                | 1234567    Pub       N                 */
    /*                              | proc sql;                                      | 1245678    Pub       N                 */
    /*                              |     update                                     | 2349878    Pri       N                 */
    /*                              |       sd1.master m                             | 3948573    Priv      Y                 */
    /*                              |     set status = coalescec(                    |                                        */
    /*                              |       m.status                                 |                                        */
    /*                              |      ,(select                                  |                                        */
    /*                              |         "N" as status                          |                                        */
    /*                              |       from                                     |                                        */
    /*                              |         sd1.trans t                            |                                        */
    /*                              |       where t.id = m.id))                      |                                        */
    /*                              | ;quit;                                         |                                        */
    /*                              |                                                |                                        */
    /*                              |                                                |                                        */
    /*                              |-----------------------------------------------------------------------------------------*/
    /*                              | 3 R SQL LEFT JOIN                              | > want                                 */
    /*                              | =================                              |        ID TYPE status                  */
    /*                              |                                                | 1 1234567  Pub      N                  */
    /*                              | %utl_rbeginx;                                  | 2 1245678  Pub      N                  */
    /*                              | parmcards4;                                    | 3 2349878  Pri      N                  */
    /*                              | library(haven)                                 | 4 3948573 Priv      Y                  */
    /*                              | library(sqldf)                                 |                                        */
    /*                              | source("c:/oto/fn_tosas9x.R")                  | SAS                                    */
    /*                              | options(sqldf.dll = "d:/dll/sqlean.dll")       |    ID   TYPE STATUS                    */
    /*                              | trans<-read_sas("d:/sd1/trans.sas7bdat")       |                                        */
    /*                              | master<-read_sas("d:/sd1/master.sas7bdat")     | 1234567 Pub    N                       */
    /*                              | # convert emty string to null                  | 1245678 Pub    N                       */
    /*                              | master$STATUS[master$STATUS == ""] <- NA       | 2349878 Pri    N                       */
    /*                              | want<-sqldf('                                  | 3948573 Priv   Y                       */
    /*                              |  select                                        |                                        */
    /*                              |     m.id                                       |                                        */
    /*                              |    ,m.type                                     |                                        */
    /*                              |    ,coalesce(m.status,"N") as status           |                                        */
    /*                              |  from                                          |                                        */
    /*                              |     master as m left join trans as t           |                                        */
    /*                              |  on                                            |                                        */
    /*                              |    m.id = t.id                                 |                                        */
    /*                              | ')                                             |                                        */
    /*                              | fn_tosas9x(                                    |                                        */
    /*                              |       inp    = want                            |                                        */
    /*                              |      ,outlib ="d:/sd1/"                        |                                        */
    /*                              |      ,outdsn ="want"                           |                                        */
    /*                              |      )                                         |                                        */
    /*                              | ;;;;                                           |                                        */
    /*                              | %utl_rendx;                                    |                                        */
    /*                              |                                                |                                        */
    /*                              | proc print data=sd1.want;                      |                                        */
    /*                              | run;quit;                                      |                                        */
    /**************************************************************************************************************************/

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.master;
       input id type$ status$;
    cards4;
    1234567 Pub .
    1245678 Pub .
    2349878 Pri N
    3948573 Priv Y
    ;;;;
    run;quit;

    data sd1.trans;
       input id type$;
    cards4;
    1234567 Pub
    1245678 Pub
    ;;;;
    run;quit;

    /**************************************************************************************************************************/
    /* SD1.MASTER obs=4            |  SD1.TRANS obs=2                                                                         */
    /*     ID      TYPE    STATUS  |      ID      TYPE                                                                        */
    /*                             |                                                                                          */
    /*  1234567    Pub             |   1234567    Pub                                                                         */
    /*  1245678    Pub             |   1245678    Pub                                                                         */
    /*  2349878    Pri       N     |                                                                                          */
    /*  3948573    Priv      Y     |                                                                                          */
    /**************************************************************************************************************************/

    /*                   _       __ _       _       _
    / |  ___  __ _ ___  | | ___ / _| |_    (_) ___ (_)_ __
    | | / __|/ _` / __| | |/ _ \ |_| __|   | |/ _ \| | `_ \
    | | \__ \ (_| \__ \ | |  __/  _| |_    | | (_) | | | | |
    |_| |___/\__,_|___/ |_|\___|_|  \__|  _/ |\___/|_|_| |_|
                                         |__/
    */

    proc sql;
     create
        table want as
     select
        m.id
       ,m.type
       ,coalescec(m.status,"N") as status
     from
        master as m left join trans as t
     on
       m.id = t.id
    ;quit;

    /**************************************************************************************************************************/
    /*  WORK. WANT total obs=4                                                                                                */
    /*     ID       TYPE    STATUS                                                                                            */
    /*                                                                                                                        */
    /*   1234567    Pub       N                                                                                               */
    /*   1245678    Pub       N                                                                                               */
    /*   2349878    Pri       N                                                                                               */
    /*   3948573    Priv      Y                                                                                               */
    /**************************************************************************************************************************/

    /*___                              _                   _       _
    |___ \   ___  __ _ ___   ___  __ _| |  _   _ _ __   __| | __ _| |_ ___
      __) | / __|/ _` / __| / __|/ _` | | | | | | `_ \ / _` |/ _` | __/ _ \
     / __/  \__ \ (_| \__ \ \__ \ (_| | | | |_| | |_) | (_| | (_| | ||  __/
    |_____| |___/\__,_|___/ |___/\__, |_|  \__,_| .__/ \__,_|\__,_|\__\___|
                                    |_|         |_|
    */

    proc sql;
        update
          sd1.master m
        set status = coalescec(
          m.status
         ,(select
            "N" as status
          from
            sd1.trans t
          where t.id = m.id))
    ;quit;

    /**************************************************************************************************************************/
    /*  WORK. WANT total obs=4                                                                                                */
    /*     ID       TYPE    STATUS                                                                                            */
    /*                                                                                                                        */
    /*   1234567    Pub       N                                                                                               */
    /*   1245678    Pub       N                                                                                               */
    /*   2349878    Pri       N                                                                                               */
    /*   3948573    Priv      Y                                                                                               */
    /**************************************************************************************************************************/

    /*____                    _
    |___ /   _ __   ___  __ _| |
      |_ \  | `__| / __|/ _` | |
     ___) | | |    \__ \ (_| | |
    |____/  |_|    |___/\__, |_|
                           |_|
    */

    /*---- you need to reload input due to previos in plce update ----*/

    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.master;
       input id type$ status$;
    cards4;
    1234567 Pub .
    1245678 Pub .
    2349878 Pri N
    3948573 Priv Y
    ;;;;
    run;quit;

    data sd1.trans;
       input id type$;
    cards4;
    1234567 Pub
    1245678 Pub
    ;;;;
    run;quit;

    %utl_rbeginx;
    parmcards4;
    library(haven)
    library(sqldf)
    source("c:/oto/fn_tosas9x.R")
    options(sqldf.dll = "d:/dll/sqlean.dll")
    trans<-read_sas("d:/sd1/trans.sas7bdat")
    master<-read_sas("d:/sd1/master.sas7bdat")
    # convert emty string to null
    master$STATUS[master$STATUS == ""] <- NA
    want<-sqldf('
     select
        m.id
       ,m.type
       ,coalesce(m.status,"N") as status
     from
        master as m left join trans as t
     on
       m.id = t.id
    ')
    fn_tosas9x(
          inp    = want
         ,outlib ="d:/sd1/"
         ,outdsn ="want"
         )
    ;;;;
    %utl_rendx;

    /**************************************************************************************************************************/
    /* R                      |   SAS                                                                                         */
    /*        ID TYPE status  |      ID   TYPE STATUS                                                                         */
    /*                        |                                                                                               */
    /* 1 1234567  Pub      N  |   1234567 Pub    N                                                                            */
    /* 2 1245678  Pub      N  |   1245678 Pub    N                                                                            */
    /* 3 2349878  Pri      N  |   2349878 Pri    N                                                                            */
    /* 4 3948573 Priv      Y  |   3948573 Priv   Y                                                                            */
    /**************************************************************************************************************************/

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
