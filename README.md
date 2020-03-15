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
Zaczniemy od klas, metod i pól. Java pozwala nam wyszukiwać klasy, tworzyć ich instancje, ustalać wartości pól (w tym
tych prywatnych) i finalnie uruchamiać metody. Posługując się refleksją nie musimy znać konkretnych nazw. Innymi słowy -
mając obiekt posiadający 2 metody których nazw nie znam, jestem w stanie wyciągnąć informacje o każdej z nich (w tym 
ich nazwy).

Jak zrobić to w praktyce? Mamy klasę "Sokrates" w pakiecie "com.github.wojtechm.refleksja.rozdzial_02". Sokrates ma 2
metody - publiczną i prywatną. Metody te nie są statyczne, więc aby je wywołać, musimy stworzyć nową instancję klasy
Sokrates.
### 2.1. Tworzenie nowej instancji
```jshelllanguage
public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
    Class<?> klasaSokratesa = Class.forName("com.github.wojtechm.refleksja.rozdzial_02.Sokrates");
    Object sokrates = klasaSokratesa.getConstructor().newInstance();
}
```
Jak widzisz w przykładzie powyżej, w naprawdę niewielkim kawałku kodu operującym refleksją, naprawdę wiele rzeczy może
pójść nie po naszej myśli. 2 linijki kodu i 5 kontrolowanych wyjątków. Tym tytułem od teraz wyjątki które już zostały 
już opisane, będą przeze mnie ignorowane, i nie będą przewijać się przez przykłady (w imię ich czytelności). 
A skoro o omawianiu wyjątków mowa:

* ***ClassNotFoundException*** jest wyjątkiem rzucanym przez metodę Class.forName(String className). Niezbyt odkrywczym będzie
    stwierdzenie, że zostaje on rzucony gdy podanej klasy nie udało się namierzyć. Podstawowym błędem popełnianym przez 
    ludzi podejmujących się walki z refleksją, jest podanie jedynie nazwy klasy, kiedy (zgodnie z dokumentacją) powinno
    się dostarczyć w pełni kwalifikowaną nazwę (ang. FQN), a więc nazwę klasy poprzedzoną jej pakietem.<br/>
     ```
    Parameters: 
    className - the fully qualified name of the desired class.
    ```
* ***NoSuchMethodException*** rzucany jest przez *Class.getConstructor(Class<?>... parameterTypes)*. Tutaj coś może pójść
    nie tak na 2 sposoby.
    1. Niezgodność parametrów - *getConstructor()* jest metodą o zmiennej liczbie argumentów (varargs). W przykładzie 
    nie podałem żadnych parametrów, a więc poszukuję bezparametrowego konstruktora. Nie ma takiego? No to wyjątek.
    2. Modyfikator dostępu - nawet jeśli parametry się zgadzają, konstruktor musi być zadeklarowany jako publiczny by
    odszukać go metodą getConstructor().
* ***IllegalAccessException***, podobnie jak kolejne 2 wyjątki które opiszę, jest rzucany przez wywołanie
    *Constructor.newInstance(Object... initargs)*. IllegalAccessException zgłaszany jest w momencie, kiedy konstruktor
    jest niedostępny (zgodnie z modyfikatorami dostępu). Jeśli czytasz ten tekst uważnie (w co niestety wątpię), to
    zauważyłeś zapewne pewien problem. Jakim cudem mogę nie mieć dostępu do konstruktora, skoro *Class.getConstructor()*
    zwraca tylko publicze konstruktory? Bardzo dobre spostrzeżenie. Fakt, nasz konstruktor z całą pewnością będzie publiczny,
    ale nie zapominajmy o tym, że istnieją inne sposoby na uzyskanie referencji do konstruktorów niepublicznych.
    Dokładniej opiszę ten precedens w jednym z późniejszych rozdziałów.
* ***InvocationTargetException*** zgłaszany jest, gdy konstruktor który wywołujemy rzuci wyjątek.
* ***InstantiationException*** zgłaszany jest, gdy próbujemy stworzyć instancję klasy abstrakcyjnej.

### 2.2. Odnoszenie się do metod, oraz ich wywoływanie.
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
