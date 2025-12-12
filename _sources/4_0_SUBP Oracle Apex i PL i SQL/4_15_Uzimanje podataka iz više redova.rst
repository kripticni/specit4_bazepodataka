Курсори - узимање података из више редова
=========================================

.. suggestionnote::

    У PL/SQL програму најчешће треба да искористимо упит који узима податке из више редова. У том случају је неопходно да употребимо курсор. Са курсором може да се ради експлицитно и имплицитно, и обавезно мора да се користи у комбинацији са циклусом који нам омогућава да идемо ред по ред кроз податке које обрађујемо.  

    Следе примери програма написаних у језику PL/SQL који користе курсоре. 

Програми се пишу у едитору у оквиру онлајн окружења *Oracle APEX*, а покрећу се кликом на дугме **Run**:

- https://apex.oracle.com/en/ (обавезно логовање на креирани налог)
- SQL Workshop
- SQL Commands

Креирати PL/SQL програме који узимају податке из базе података библиотеке. Следи списак свих табела са колонама. Примарни кључеви су истакнути болд, а страни италик. 

.. image:: ../../_images/slika_73a.jpg
   :width: 780
   :align: center

Вратимо се на проблем који смо већ раније решили. Приказати име, презиме и телефон члана библиотеке са бројем чланске карте 22. Име и презиме приказати спојено. За овај проблем користимо SELECT INTO зато што су нам потребни подаци из тачно једног реда, реда у којем се налазе подаци о члану са бројем чланске карте 22.  

::


    DECLARE
        v_clan VARCHAR2(150);
        v_telefon clanovi.telefon%TYPE;
    BEGIN
        SELECT ime||' '||prezime, telefon INTO v_clan, v_telefon
        FROM clanovi WHERE broj_clanske_karte=22;
        DBMS_OUTPUT.PUT_LINE('Ime i prezime clana: '||v_clan);
        DBMS_OUTPUT.PUT_LINE('Telefon: '|| v_telefon);
    END

Уколико желимо да прикажемо ове податке за све чланове, морамо да користимо **курсор**. Да бисмо радили са курсором, неопходни су следећи кораци:

1. курсор се у одељку за декларацију креира и веже за SELECT упит, 
2. курсор се отвори у телу PL/SQL програма и тада се изврши упит из декларације,,
3. у циклусу се помоћу курсора чита један по један ред док се не прочитају сви редови резултата одговарајућег SELECT упита,
4. курсор се затвори. 

Упознаћемо се са овом корацима кроз решење следећег задатка. Приказати имена, презимена и телефоне свих чланова библиотеке. Име и презиме приказати спојено. 

::


    DECLARE
        v_clan VARCHAR2(150);
        v_telefon clanovi.telefon%TYPE;
        CURSOR kursor_clan IS SELECT ime||' '||prezime, telefon FROM clanovi;
    BEGIN
        OPEN kursor_clan;
        LOOP
            FETCH kursor_clan INTO v_clan, v_telefon;
            EXIT WHEN kursor_clan%NOTFOUND;
            DBMS_OUTPUT.PUT_LINE('Ime i prezime clana: '||v_clan);
            DBMS_OUTPUT.PUT_LINE('Telefon: '|| v_telefon);
        END LOOP;
        CLOSE kursor_clan;
    END

.. image:: ../../_images/slika_81a.jpg
   :width: 600
   :align: center

.. image:: ../../_images/slika_81b.jpg
   :width: 300
   :align: center

Можемо да користимо променљиву сложеног типа да у њу учитамо цео ред. 

Променљива *v_red* има онолико поља колико има одговарајући SELECT упит. Како је прва колона добијена као израз, важно је да јој се додели име, у овом случају *clan*, тако да може да се приступа том пољу сложене променљиве на следећи начин: *v_red.clan* (назив променљиве, тачка, назив поља).

::


    DECLARE
        CURSOR kursor_clan IS SELECT ime||' '||prezime clan, telefon FROM clanovi;
        v_red kursor_clan%ROWTYPE;
    BEGIN
        OPEN kursor_clan;
        LOOP
            FETCH kursor_clan INTO v_red;
            EXIT WHEN kursor_clan%NOTFOUND;
            DBMS_OUTPUT.PUT_LINE('Ime i prezime clana: '||v_red.clan);
            DBMS_OUTPUT.PUT_LINE('Telefon: '|| v_red.telefon);
        END LOOP;
        CLOSE kursor_clan;
    END

Овакав облик рада са курсором се назива експлицитни и подразумева да експлицитно набројимо сваки корак који са курсором треба да се изврши. Курсор може имплицитно да се отвори, да се чита ред по ред и да се затвори, употребом циклуса FOR.

::

    DECLARE
        CURSOR kursor_clan IS SELECT ime||' '||prezime clan, telefon FROM clanovi;
        v_red kursor_clan%ROWTYPE;
    BEGIN
        FOR v_red IN kursor_clan LOOP
            DBMS_OUTPUT.PUT_LINE('Ime i prezime clana: '||v_red.clan);
            DBMS_OUTPUT.PUT_LINE('Telefon: '|| v_red.telefon);
        END LOOP;
    END 

Променљива које се користи у циклусу FOR не мора да се експлицитно декларише, тако да ће следећи блок кода такође радити. 

::

    DECLARE
        CURSOR kursor_clan IS SELECT ime||' '||prezime clan, telefon FROM clanovi;
    BEGIN
        FOR v_red IN kursor_clan LOOP
            DBMS_OUTPUT.PUT_LINE('Ime i prezime clana: '||v_red.clan);
            DBMS_OUTPUT.PUT_LINE('Telefon: '|| v_red.telefon);
        END LOOP;
    END

База података за библиотеку коју користимо нема превелики број података. Најчешће у базама имамо табеле са веома великим бројем редова и није могуће да све податке из табеле повучемо у програм. Из тог разлога можемо да ограничимо број редова из којих узимамо податке користећи у упиту FETCH FIRST ROWS ONLY уз навођење броја редова који нам је потребан. 

Следећи програм узима само податке о прва три члана. 

.. infonote::

    Како је пример базе података за библиотеку мали, ово нећемо употребљавати у програмима који следе, али би требало да увек имате у виду да се FETCH FIRST ROWS ONLY, или нека друга опција за ограничавање броја редова који се узимају, обавезно користи у већим базама података. 

::

    DECLARE
        CURSOR kursor_clan IS SELECT ime||' '||prezime clan, telefon 
        FROM clanovi FETCH FIRST 3 ROWS ONLY;
    BEGIN
        FOR v_red IN kursor_clan LOOP
            DBMS_OUTPUT.PUT_LINE('Ime i prezime clana: '||v_red.clan);
            DBMS_OUTPUT.PUT_LINE('Telefon: '|| v_red.telefon);
        END LOOP;
    END

У овим примерима смо у назив курсора ставили на почетак реч *kursor*, а промељиву за читање једног реда смо звали *v_red*. Могу, наравно, да се користе и другачији називи, и неки примери именовања курсора и одговарајуће променљиве ће бити приказани у задацима који следе. 
