# Refleksja

## O repozytorium
W tym repozytorium opisałem podstawy teoretyczne oraz praktyczne zastosowania mechanizmu refleksji w języku Java.
Staram się, by notatki te były *Junior Software Developer friendly*, więc zacząłem od najbardziej podstawowych 
informacji, by później, krok po kroku, dojść do tych faktycznie ciekawych. :)

## 1. Mechanizm Refleksji
Mechanizm refleksji to proces, dzięki któremu program komupterowy może podglądać, analizować i modyfikować kod źródłowy w
trakcie działania aplikacji. Wyobraź sobie aplikację która przygląda się twojemu kodowi i w trakcie działania decyduje
o tym, które metody uruchomić. W praktyce będzie to chociażby biblioteka testowa (np. TestNG lub JUnit) - przeglądają 
aplikację w poszukiwaniu metod o adnotacji @Test, a następnie takowe metody uruchamiają.

## 2. Wprowadzenie do refleksji w Javie
Zaczniemy od klas, metod i pól. Java pozwala nam wyszukiwać klasy, tworzyć ich instancje, odczytować wartości pól 
i uruchamiać metody. Posługując się refleksją nie musimy znać konkretnych nazw. Innymi słowy -
mając obiekt posiadający 2 metody których nazw nie znam, jestem w stanie wyciągnąć informacje o każdej z nich (w tym 
ich nazwy).

Jak zrobić to w praktyce? Mamy klasę "Sokrates" w pakiecie "com.github.wojtechm.refleksja.rozdzial_02". Sokrates ma 2
metody - publiczną i prywatną. Metody te nie są statyczne, więc aby je wywołać, musimy stworzyć nową instancję klasy
Sokrates.

### 2.1. Klasa *Class* - wrota do refleksji.
Punktem wejścia do świata refleksji będzie klasa *java.lang.Class<T\>*. Jest to specjalny kawałek kodu reprezentujący
klasy i interfejsy w aplikacji Javowej. Przechowuje ona wszystkie niezbędne nam informacje i udostępnia je poprzez
bardzo sympatyczne, choć rozbudowane API.

Zgodnie z wprowadzeniem - mamy klasę *Sokrates* i chcemy stworzyć jej instancję. Aby to zrobić będziemy potrzebować
referencji do klasy *Class* reprezentującej klasę *Sokrates*. Możemy zrobić to na dwa sposoby.
1. Bezpieczny: jeszcze na poziomie kompilacji wiemy, że klasa ta istnieje i możemy się do niej odnieść bezpośrednio.
Dodatkowo możemy zdefiniować konkretny typ naszej klasy. Przekonasz się o tym że to przydatne pod koniec następnego 
podrozdziału.
    ```jshelllanguage
    public static void main(String[] args) {
        Class<Sokrates> klasaSokratesa = Sokrates.class;
    }
    ```
2. Niebezpieczny: klasy szukamy w trakcie działania aplikacji, co może skutkować niechcianym wyjątkiem. Dodatkowo nie
definiujemy typu generycznego.
    ```jshelllanguage
    public static void main(String[] args) throws ClassNotFoundException {
        Class<?> klasaSokratesa = Class.forName("com.github.wojtechm.refleksja.rozdzial_02.Sokrates");
    }
    ```
    ***ClassNotFoundException*** jest wyjątkiem rzucanym przez metodę *Class.forName(String className)*. Niezbyt odkrywczym będzie
    stwierdzenie, że zostaje on rzucony gdy podanej klasy nie udało się namierzyć. Podstawowym błędem popełnianym przez 
    ludzi podejmujących się walki z refleksją, jest podanie jedynie nazwy klasy, kiedy (zgodnie z dokumentacją) powinno
    się dostarczyć w pełni kwalifikowaną nazwę (ang. FQN), a więc nazwę klasy poprzedzoną jej pakietem.<br/>
    ```
    Parameters: 
    className - the fully qualified name of the desired class.
    ```
   
Jeśli mamy ten komfort i możemy posługiwać się pierwszym sposobem, to powinniśmy z tego korzystać. W ramach tego tekstu
planuję jednak używać tej dłuższej (niebezpiecznej) wersji, żebyś miał okazję napatrzeć się na tę składnię.

### 2.2. Tworzenie nowej instancji.
Mamy już naszą klasę - zmienna *klasaSokratesa*. Czas na tworzenie nowej instancji. Oto jak to robimy:
```jshelllanguage
public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
    Class<?> klasaSokratesa = Class.forName("com.github.wojtechm.refleksja.rozdzial_02.Sokrates");
    Object sokrates = klasaSokratesa.getConstructor().newInstance();
}
```
Jak widzisz w przykładzie powyżej, w naprawdę niewielkim kawałku kodu operującym refleksją, naprawdę wiele rzeczy może
pójść nie po naszej myśli. 1 nowa linijka kodu i 4 nowe kontrolowane wyjątki. Tym tytułem od teraz wyjątki które zostały 
już opisane, będą przeze mnie pomijane, i nie będą przewijać się przez przykłady (w imię ich czytelności). 
A skoro o omawianiu wyjątków mowa:

* ***NoSuchMethodException*** rzucany jest przez *Class.getConstructor(Class<?>... parameterTypes)*. Tutaj coś może pójść
    nie tak na 2 sposoby.
    * Niezgodność parametrów - *getConstructor()* jest metodą o zmiennej liczbie argumentów (varargs). W przykładzie 
    nie podałem żadnych parametrów, a więc poszukuję bezparametrowego konstruktora. Nie ma takiego? No to wyjątek.
    * Modyfikator dostępu - nawet jeśli parametry się zgadzają, konstruktor musi być zadeklarowany jako publiczny by
    odszukać go metodą *getConstructor()*.
* ***IllegalAccessException***, podobnie jak kolejne 2 wyjątki które opiszę, jest rzucany przez wywołanie
    *Constructor.newInstance(Object... initargs)*. IllegalAccessException zgłaszany jest w momencie, kiedy konstruktor
    jest niedostępny (zgodnie z modyfikatorami dostępu). Jeśli czytasz ten tekst uważnie (w co niestety wątpię), to
    zauważyłeś zapewne pewien problem. Jakim cudem mogę nie mieć dostępu do konstruktora, skoro *Class.getConstructor()*
    zwraca tylko publicze konstruktory? Bardzo dobre spostrzeżenie. Fakt, nasz konstruktor z całą pewnością będzie publiczny,
    ale nie zapominajmy o tym, że istnieją sposoby na uzyskanie referencji do konstruktorów niepublicznych.
    Dokładniej opiszę ten precedens w jednym z późniejszych rozdziałów.
* ***InvocationTargetException*** zgłaszany jest, gdy konstruktor który wywołujemy rzuci wyjątek.
* ***InstantiationException*** zgłaszany jest, gdy próbujemy stworzyć instancję klasy abstrakcyjnej.

Jeśli żaden wyjątek nie został zgłoszony, to otrzymujemy nowy obiekt, który jest instancją klasy Sokrates.

No i wszystko super, ale dlaczego *sokrates* jest zmienną typu *Object*, a nie *Sokrates*? Odpowiedź jest odrobinę skomplikowana. 

Szybkie i niedokładne wprowadzenie do generyków: w przykładzie zmienna *klasaSokratesa* jest typu *Class<?\>*. *Wildcard*
(przedstawiany znakiem zapytania) oznacza "nieznane" - typ klasy *klasaSokratesa* jest nieznany. Mógłby to być *String*,
*Sokrates*, czy nawet *ArrayIndexOutOfBoundsException* - nie zagwarantowaliśmy typu... więc dlaczego *Object*?

Mamy tu do czynienia z 
uproszczeniem - skoro w Javie prawie wszystko jest obiektem (pozdrowienia dla prymitywów), to i tu będziemy mieć do 
czynienia z czymś, co dziedziczy po *Object*. Gdybyśmy mieli gwarancję typu, nie byłoby problemu z przypisaniem 
*sokratesa* do zmiennej typu *Sokrates*. Oczywiście da się to zrobić -> przykład poniżej.
```jshelllanguage
public static void main(String[] args) {
    Class<Sokrates> klasaSokratesa = Sokrates.class;
    Sokrates sokrates = klasaSokratesa.getDeclaredConstructor().newInstance();
}
```

### 2.3. Odnoszenie się do metod, oraz ich wywoływanie.
Mamy już klasę oraz jej instancję. Kolejnym krokiem jest wyciągnięcie informacji o metodach. Najbardziej oczywistym będzie
użycie metody *Class.getMethods()* na naszej klasie Sokratesa.
```jshelllanguage
public static void main(String[] args) {
    Class<?> klasaSokratesa = Class.forName("com.github.wojtechm.refleksja.rozdzial_02.Sokrates");
    Object sokrates = klasaSokratesa.getDeclaredConstructor().newInstance();
    Method[] metodySokratesa = klasaSokratesa.getMethods();
    for (Method metoda : metodySokratesa) {
        System.out.println(metoda.getName());
    }
}
```
##### *Wspominałem, że opisane wcześniej wyjątki będą pomijane i tak też się stało (brak klauzuli **throws**).*

Wykonanie tego programu skutkuje następującym wydrukiem do konsoli:
```text
powiedzCośMądrego
wait
wait
wait
equals
toString
hashCode
getClass
notify
notifyAll
```
Wiemy już, że nasz filozof może *powiedziećCośMądrego()* oraz że odziedziczył podstawowe metody po klasie *Object*.
Ma to sens. Wszak *prawie* wszystko w Javie jest obiektem (pozdrowienia dla prymitywów). Jest jednak jeden problem.
Wspomniałem, że Sokrates ma dwie metody - publiczą i prywatną. Jak zapewne się domyślasz, *Sokrates.powiedzCośMądrego()*
jest metodą publiczną. Jak więc dostać się do metod prywatnych? Z pomocą przychodzi metoda *Class.getDeclaredMethods()*.
```jshelllanguage
public static void main(String[] args) {
    Class<?> klasaSokratesa = Class.forName("com.github.wojtechm.refleksja.rozdzial_02.Sokrates");
    Object sokrates = klasaSokratesa.getDeclaredConstructor().newInstance();
    Method[] metodySokratesa = klasaSokratesa.getDeclaredMethods(); // Zmiana z .getMethods() na .getDeclaredMethods().
    for (Method metoda : metodySokratesa) {
        System.out.println(metoda.getName());
    }
}
```
Drobna zmiana w kodzie i mamy nowy wynik:
```text
pomyślOCzymśMądrym
powiedzCośMądrego
```
Różnice które od razu rzucają się w oczy. Metoda *Class.getDeclaredMethods()* zwraca tylko i wyłącznie metody, które 
zdefiniowane zostały w danej klasie, kiedy *Class.getMethods()* udostępnia **jedynie publiczne** metody włącznie z tymi
odziedziczonymi z interfejsów czy klasy bazowej. Warto zwrócić również uwagę na fakt, że żadna z tych metod nie 
udostępnia konstruktorów.

Mamy już interesujące nas metody (tablica *Method[] metodySokratesa*) - czas je wywołać. Kluczem do wykonania tej operacji
jest metoda *Method.invoke(Object obj, Object... args)*. Pierwszy parametr to obiekt na którym zostanie wywołana nasza
metoda (w przypadku metod statycznych możemy tam wrzucić *nulla*). Parametr *args* reprezentuje argumenty które zostaną 
wykorzystane do uruchomienia metody. Mając tę wiedzę oczywiście chcemy ustalić ile parametrów przyjmują nasze metody.
Nowy kawałek kodu:
```jshelllanguage
public static void main(String[] args) {
    Class<?> klasaSokratesa = Class.forName("com.github.wojtechm.refleksja.rozdzial_02.Sokrates");
    Object sokrates = klasaSokratesa.getDeclaredConstructor().newInstance();
    Method[] metodySokratesa = klasaSokratesa.getMethods();
    for (Method metoda : metodySokratesa) {
        System.out.printf("Metoda %s(). Parametry: %d => %s\n",
                metoda.getName(),
                metoda.getParameterCount(),
                Arrays.toString(metoda.getParameterTypes()));
    }

}
```
I nowe wyniki:
```text
Metoda powiedzCośMądrego(). Parametry: 0 => []
Metoda wait(). Parametry: 1 => [long]
Metoda wait(). Parametry: 2 => [long, int]
Metoda equals(). Parametry: 1 => [class java.lang.Object]
// i kilka innych
```
Podmieniłem metodę *.getDeclaredMethods()* na *getMethods()* aby pokazać metody bezargumentowa, przyjmujące obiekty i 
typy prymitywne. Jak widzisz mamy konkretnie zapisane jakich typów są oczekiwane przez metodę parametry, oraz ich ilość.
Tutaj drobny spoiler niezbędny do pójścia do przodu => prywatna metoda *Sokrates.pomyślOCzymśMądrym()* również jest
bezargumentowa.
```text
Metoda powiedzCośMądrego(). Parametry: 0 => []
Metoda pomyślOCzymśMądrym(). Parametry: 0 => []
```
No to już mamy wszystko. Czas na wywołanie naszych metod.
```jshelllanguage
public static void main(String[] args) throws IllegalAccessException, InvocationTargetException {
    Class<?> klasaSokratesa = Class.forName("com.github.wojtechm.refleksja.rozdzial_02.Sokrates");
    Object sokrates = klasaSokratesa.getDeclaredConstructor().newInstance();
    Method[] metodySokratesa = klasaSokratesa.getDeclaredMethods();
    for (Method metoda : metodySokratesa) {
        System.out.printf("Metoda %s().\n", metoda.getName());
        metoda.invoke(sokrates);
    }
}
```
Jedna nowa linia, dwa nowe kontrolowane wyjątki. Pierwszy z nich to IllegalAccessException, który faktycznie pojawia się
po uruchomieniu kodu:
```text
Metoda powiedzCośMądrego().
Szczęście jest przyjemnością wolną od wyrzutów sumienia.
Metoda pomyślOCzymśMądrym().
Exception in thread "main" java.lang.IllegalAccessException: class com.github.wojtechm.refleksja.rozdzial_02.Podstawy cannot access a member of class com.github.wojtechm.refleksja.rozdzial_02.Sokrates with modifiers "private"
	at java.base/jdk.internal.reflect.Reflection.newIllegalAccessException(Reflection.java:361)
	at java.base/java.lang.reflect.AccessibleObject.checkAccess(AccessibleObject.java:591)
	at java.base/java.lang.reflect.Method.invoke(Method.java:558)
	at com.github.wojtechm.refleksja.rozdzial_02.Podstawy.main(Podstawy.java:32)
```
Zgodnie z treścią błędu - nie możemy uruchomić prywatnej metody ot tak. Java poprzez zgłoszenie wyjątku sugeruje, że
aktualnie próbujemy podkopać jeden z filarów programowania obiektowego w języku programowania 
prawie całkowicie obiektowym (pozdrowienia dla prymitywów). Jesteśmy jednak oddani sprawie i poradzimy sobie z tą
niedogodnością za pomocą metody *Method.setAccessible(boolean flag)*, która (jak wskazuje nazwa) może magicznie sprawić, 
że metoda staje się dostępna. Zmienna *flag* oznacza bezpośrednio *"czy ograniczenia dostępu języka Java powinny być 
zignorowane?"*.
```jshelllanguage
public static void main(String[] args) throws InvocationTargetException {
    Class<?> klasaSokratesa = Class.forName("com.github.wojtechm.refleksja.rozdzial_02.Sokrates");
    Object sokrates = klasaSokratesa.getDeclaredConstructor().newInstance();
    Method[] metodySokratesa = klasaSokratesa.getDeclaredMethods();
    for (Method metoda : metodySokratesa) {
        System.out.printf("Metoda %s().\n", metoda.getName());
        metoda.setAccessible(true);
        metoda.invoke(sokrates);
    }
}
```
Pozostaje jeszcze *InvocationTargetException*. Podobnie jak w przypadku wywołania *Constructor.newInstance(Object... 
initargs)* - wyjątek ten zostaje zgłoszony, gdy metoda którą uruchamiamy zgłosi wyjątek.

Warto mieć tu na uwadze jeszcze jedną rzecz. Niekontrolowany wyjątek *IllegalArgumentException* może zostać zgłoszony
przez metodę *Method.invoke(Object obj, Object... args)* jeśli spróbujemy podstawić pod zmienną *obj* obiekt, który
nie jest instancją klasy do której nasza metoda należy. Bardziej po ludzku - jeśli mamy metodę *M* którą wyciągneliśmy
z klasy *K*, to wywołanie *M.invoke(O)* zgłosi wyjątek, jeśli nasz obiekt *O* **nie jest** instancją klasy *K*.
```jshelllanguage
metodaSokratesa.invoke("Jakiś losowy string, który (rzecz jasna) nie jest instancją klasy Sokrates");
```

### 2.4. Praca z polami.
##### *Poprzedni podrozdział odrobinę się rozciągnął. Tym razem będzie krótko... chyba. Mam nadzieję...*
Zacznijmy od uproszczenia naszego kodu. Na ten moment nie będziemy zajmować się metodami, więc uproszczę kod z poprzedniej
sekcji.
```jshelllanguage
public static void main(String[] args) {
    Class<?> klasaSokratesa = Class.forName("com.github.wojtechm.refleksja.rozdzial_02.Sokrates");
    Object sokrates = klasaSokratesa.getDeclaredConstructor().newInstance();
}
```
Podobnie jak w przypadku wyciągania metod, dobieranie się do informacji o polach można zrobić na dwa sposoby:
* *Class.getFields()* - zwraca tablicę pól publicznych włącznie z tymi odziedziczonymi po klasie bazowej.
* *Class.getDeclaredFields()* - zwraca tablicę wszystkich pól zdefiniowanych w danej klasie (ignorując pola klasy bazowej).
Nowy kod:
```jshelllanguage
public static void main(String[] args) {
    Class<?> klasaSokratesa = Class.forName("com.github.wojtechm.refleksja.rozdzial_02.Sokrates");
    Object sokrates = klasaSokratesa.getDeclaredConstructor().newInstance();
    for (Field pole : klasaSokratesa.getDeclaredFields()) {
        System.out.println(pole.toGenericString());
    }
}
```
Nowe wyniki:
```text
public static final java.lang.String com.github.wojtechm.refleksja.rozdzial_02.Sokrates.EPOKA
public final java.lang.String com.github.wojtechm.refleksja.rozdzial_02.Sokrates.zawód
private final java.lang.String com.github.wojtechm.refleksja.rozdzial_02.Sokrates.ulubionyCytat
```
Aby uzyskać wartości danych pól wykorzystujemy metodę *Field.get(Object obj)*. Zmienna *obj* to wskaźnik do obiektu
którego pola nas interesują; jeśli pole jest statyczne, *obj* może przyjąć wartość *null*. Podobnie jak w przypadku
metod, chcemy wywołać *Field.setAccessible(boolean flag)*, aby móc grzebać w prywatnych rzeczach Sokratesa.
```jshelllanguage
public static void main(String[] args) {
    Class<?> klasaSokratesa = Class.forName("com.github.wojtechm.refleksja.rozdzial_02.Sokrates");
    Object sokrates = klasaSokratesa.getDeclaredConstructor().newInstance();
    for (Field pole : klasaSokratesa.getDeclaredFields()) {
        pole.setAccessible(true);
        System.out.printf("Pole '%s' ma wartość '%s'\n", pole.getName(), pole.get(sokrates));
    }
}
```
```text
Pole 'EPOKA' ma wartość 'Starożytność.'
Pole 'zawód' ma wartość 'Filozof'
Pole 'ulubionyCytat' ma wartość 'Strzeż się ludzi, którzy są pewni tego, że mają rację.'
```

### 2.5. Podsumowanie i źródła
W tym rozdziale poznałeś podstawowe techniki tworzenia i inspekcji obiektów za pomocą refleksji. Oczywiście poznałeś
również mój ulubiony cytat Sokratesa, w imię którego bardzo chciałbym, żebyś postanowił przekonać się o tym, czy mam rację.

Zachęcam Cię do zrobienia dwóch rzeczy:
1. Napisz sobie prostą klaskę podobną do Sokratesa i postaraj się wyciągnąć z niej informacje. Eksperymentuj!
2. Zerknij do dokumentacji - podaję nazwy klas i metod dla łatwiejszego ich namierzenia. Klasy które wykorzystaliśmy
do naruszenia przastrzeni osobistej Sokratesa znajdują się w [pakiecie java.lang.reflect](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/package-summary.html).
