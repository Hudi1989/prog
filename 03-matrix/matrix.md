

# Általánosítunk tovább

Előző óra végén eljutottunk egy olyan beolvasási függvényhez, ami folyam illetve típusfüggetlen és ráadásul >> operátorként használhatjuk.
 
```c++
template <typename T>
istream& operator>>(istream& in,vector<T> &myvector)
{
    unsigned int n=0;
    in >> n;
    myvector.resize(n);
    for(int i=0;i<n;i++)
    {
        in >> myvector[i];
    }
    return in;
}
```

Ennek a beolvasás függvénynek van egy nagy ereje: alapvetően ez a függvény működik mindenre ami egy vectort akar beolvasni függetlenül attól, hogy mi van benne. A gondolatmenetet követve feltehetünk egy kérdést, hogy: akkor vector is lehet benne? Amire természetesen helyeslő a válasz.

Ha csinálunk egy olyan vectort ami vectorokat tartalmaz, akkor máris eljutottunk egy mátrixhoz. 
```c++
vector< vector<int> > v;
vector< vector<float> > v2;
```
Ez kb. úgy néz ki, hogy van egy vector amiben vannak vectorok és azokban a számok. Így természetesen a sorok szélességei lehetnek különbözőek, de ez minket nem zavar. De akkor nézzük a beolvasó operátorunkat a fejezet elején: Ha egy ilyen vector-vector-int-re (olyan vector, ami olyan vectorokat tartalmaz ami inteket tartalmaz vagy inteket tartalmazó vectorok vectora) ráhívjuk a >> operátorunkat akkor vajon, befogja tudni olvasni?
Előző fejezetben beszéltünk, hogy mindenre jó a beolvasó függvényünk a template miatt azokra amire az istream képes, tehát amiely típusokhoz van >> operátor. Viszont a mátrix beolvasásához szükségünk lenne arra, hogy vectort be tudjunk olvasni, viszont ugyanezt a függvényt már használtuk vector beolvasásához és most ugyanezt akarjuk használni vector-vector-int beolvasására.  Ezt nyugodt szívvel meg is tehetjük, mert ha a kódban írunk egy olyat, hogy:

```c++
vector< vector<int> > v;
cin >> v;
```
Ekkor a ```c++cin >> v```-t a fordító átalakítja ```c++operator>>(cin,v)```-re, ezután megpróbál keresni egy olyan függvényt aminek azonos a szignatúrájá. Az általunk írt ```c++operator>>(istream&,vector<T>)```-t meg is találja úgy, hogy a ```c++T``` típus ```c++vector<int>```. Ebben a függvényben viszont viszont meghívódik egy ```c++T```-re a ```c++operator>>(istream&,vector<int>)```, ebben az esetben meg a template a már eddig látott módon elkészül és az fog lefutni.

Viszont ebben az esetben milyen bemeneti formátum lesz szükséges? 
```
3 
3 1 2 3 
3 2 3 4
3 4 5 6
```

Soronként is meg kell adnunk, hogy hány elem van egy adott sorban (itt egy kicsit több a szabadságfok mint kéne, mivel egy mátrixban nem lehetnek különböző hosszú sorok, de ettől egyelőre eltekintünk.
Ha megnézzük valójában ez egy vector reprezentáció, a felépítése a rekurzíóval párhuzamba hozva megérthető: Az első hármas megadja, hogy hány vector-t akarunk beolvasni, ezután jönnek a vektorok. De hogy olvasunk be egy vektort, úgy hogy beolvasunk egy számot ami adja, hogy hány elem van a vectorban aztán beolvassuk a vectorokat.
Az első vector ```3 1 2 3``` lesz, ami egy olyan vector amiben 1,2 és 3 egész számok szerepelnek (az első a számosság) 

Ez a technika könnyen bővíthető a következő bementre is vector-vector-vector-int típusra:
```
2

3 
3 1 2 3 
3 2 3 4
3 4 5 6

3 
3 2 5 7 
3 1 3 2
3 7 5 3
```
Ez innen felfogható egy 3d-s mátrixként ami 3 széles 3 magas és 2 mély.



## Többszörös visszavezetés 

Azzal, hogy így rekurzíve betudunk olvasni több dimenziós adatot is (az operátorunk logikusan akármennyi egymásba ágyazott vektorra működne), gondolkodhatunk komplexebb problémákon is.

Hogyan keressük meg például, hogy *egy mátrixban melyik sor összege a maximális?*
Ha a mondatot elolvassuk magát, rögtön szembejön egy a két alkalmazandó tételünk neve: maximális, összege.
Persze ez a mondat lehet sokkal körmönfontabb is:

*Egy versenyen n darab versenyző m számban versenyez. Egy mátrixban feljegyeztük a ponteredményeiket egy-egy versenyszámban. Írjunk programot amely megmondja, hogy hanyadik versenyző lett a nyertes.*
Ez a feladat írandó program szempontjából teljesen ugyanaz.

Egy problémánk van: nekünk csak olyan programozási tételeink vannak, amik tömbökön (vektorokon) dolgozik, tehát ahhoz, hogy megoldjunk egy ilyen problémát módosítanunk kell egy kicsit az egyik tételünket amit alkalmazunk a megoldásnál.
Egy ilyen 2 dimenziós megoldásnál lesz egy külső tételünk meg egy belső tételünk. A belső tétel fog végrehajtódni a mostani példánk esetében egy sorra, ami összegezni fogja az adott sort. Tehetnénk azt, hogy végigmegyünk ciklussal a sorokon és mindre lefuttatjuk az összegzés tételét, majd az eredményeket tároljuk egy köztes vektorban és erre lefuttatjuk a külső tételt és így megtaláljuk a maimumot.
Ezzel az a probléma, hogy ha milliós nagyságrendű mátrixról van szó, akkor ez a mátrix is csak növeli a tárigényt, amikor könnyedén megoldható anélkül is.

Tehát akkor a szumma az marad ugyanaz ami normál esetben lenne:
```c++
int szum(const vector<int> &v){
    int s = 0;
    for(int a=0;a<v.size();a++){
        s += v[a];
    }
    return s;
}
```
vagy templatel:
```c++
template<typename T>
int szum(const vector<T> &v){
    int s = 0;
    for(int a=0;a<v.size();a++){
        s += v[a];
    }
    return s;
}
```
A templates esetben nyitva hagyjuk azt, hogy lebegőpontos vagy egész, mert mindkettőre működik és csak a bemenő paramétertől fog függeni.

A külső tétel, ami a maximumkeresés viszont igényelni fog egy kis váloztatást. Eddig a vectorban egy számérték volt amit tudtunk hasonlítani egymáshoz, most viszont egy teljes sor van ott aminek az összegével kéne számolnunk. 
Az alap maxker:
```c++
//Ez a függvény csak akkor működik, ha van elem a vectorban
int max(const vector<int> &v){
    int maxInd = 0; //ezzel jelöljük, hogy hol a maximális elem
    int max = v[0];
    for(int a=0;a<v.size();a++){
        if (max < v[a]){
            max = v[a];
            maxInd = a;
        }
    }
    return maxInd;
}
```
Az belső tétel bekötése után:
```c++
//Ez a függvény csak akkor működik, ha van elem a vectorban
int max(const vector<int> &v){
    int maxInd = 0; //ezzel jelöljük, hogy hol a maximális elem
    int max = szum(v[0]);
    for(int a=0;a<v.size();a++){
        if (max < szum(v[a])){
            max = szum(v[a]);
            maxInd = a;
        }
    }
    return maxInd;
}
```
Ezzel a legfőbb probléma, hogy mindig amikor talál új maximumot a függvény akkor újra kiszámolja a sorösszeget, ami egy nagyon széles mátrix esetén már jelentős számítási különbség lehet, tehát meg kéne jegyeznünk a ciklusmagban a sorösszeget. Egy lokális változó behozásával megoldható a dolog nagyon egyszerűen.
```c++
//Ez a függvény csak akkor működik, ha van elem a vectorban
int max(const vector<int> &v){
    int maxInd = 0; //ezzel jelöljük, hogy hol a maximális elem
    int max = szum(v[0]);
    for(int a=0;a<v.size();a++){
        int temp = szum(v[a]);
        if (max < temp){
            max = temp;
            maxInd = a;
        }
    }
    return maxInd;
}
```

De akár templatet is csinálhatunk belőle:
```c++
template<typename T>
int max(const vector<T> &v){
    int maxInd = 0; //ezzel jelöljük, hogy hol a maximális elem
    T max = szum(v[0]);
    for(int a=0;a<v.size();a++){
        int temp = szum(v[a]);
        if (max < temp){
            max = temp;
            maxInd = a;
        }
    }
    return maxInd;
}
```

A végső függvényünk már float,int-re is működő megoldás, de például string-el is működhetne a dolog, mivel az STL definiálja a ```c++ <``` (mint lexikografikus rendezés) és ```c++ +``` (mint konkatenáció) operátort, ezért valójában csak egy probléma van ami akadályoz minket, hogy az összegzés függvényünkben a kezdőérték 0-ként van definiálva ami viszont a stringek terén nem hasznáható. [TODO:hogy lehetne ezt tovább áltaálnosítani (null függvény)]

## Hogy találjunk ki feladatokat?

A félévközi ZH-n egy ilyen többszörös visszavezetéses feladat várható.
* Összegzés (sum)
* Számlálás (cnt)
* Lineáris keresés (ker)
* Kiválasztás (kiv)
* Maximum kiválasztás (max)
* Feltételes maximum keresés (ftm)

Hat alaptételünk van és a ZH-n várhatóan ezeknek valamilyen összekapcsolása lesz. Nézznk pár példát arra, hogy hogyan is lehet kreálni ilyen feladatokat.    

Jelöljük egymás után írással, kötőjellele elválasztva a két egymásba ágyazandó tételt úgy, hogy az első a külső tétel amibe beágyazzuk a 2.-at.

###Alaptételek kiterjesztése

A legegyszerűbb eset amikor az eredmény olyan mintha a mátrix minden elemén akarnánk futtatni az alaptételt. 

SUM-SUM 
* Adjuk össze az összes elemet

SUM-CNT
* Adjuk össze hogy a sorokban hány páros szám van! = Adjuk meg, hogy a mátriban hány páros szám van!
* Adjuk meg, hogy a mátrixban adott feltételnek megfelelő szám mennyi van?

KER-KER - Keressünk adott feltétlenek megfelelő elemet a mátrixban

KIV-KER - Kiválasztás a mátrix egy adott típusú elemére (Egy adott sorban nem tudjuk, hogy van-e így ott KER-t kell futtatnunk)

MAX-MAX - Válasszuk ki a mátrix maximális elemét.

FTM-FTM - Feltételesen keressük a maximumot (a külső ftm feltétle, hogy talált-e a belső FTM az érték meg a belő FTM által visszaadott maximum.)


### Valódi dupla tétel használat

CNT-SUM
* A mátrix mennyi sorának az összege pozitív?
* Legyenek a sorok versenyzők vagy tanulók az oszlopok meg versenyszámok vagy feladatok (mostantól: versenyző feladat).
    * Mennyi versenyző érte el a szükséges ponthatárt? = A mátrix hány sorösszege több mint egy adott konstans.
  
FTM-SUM,KER
* Egy mátrix melyik sorában maximimális az összeg, amelyikben csak negatív számok vannak. (ergo legközelebb a 0-hoz)
* Versenyző feladat
    * Melyik versenyző nyert? (Minimális versenyidő és minden versenyszámban érvényes eredmény, azaz nem 0)

# Feladat
1. Írjunk egy olyan programot ami beolvas egy mátrixot az ismeretetett formátumban majd megadja, hogy melyik sorban maximális a sorösszeg.
