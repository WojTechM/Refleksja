# Refleksja

## O repozytorium
W tym repozytorium opisałem podstawy teoretyczne oraz praktyczne zastosowania mechanizmu refleksji w języku Java.
Staram się, by notatki te były *Junior Software Developer friendly*, więc zacząłem od najbardziej podstawowych informacji, by później, krok po kroku, dojść do tych faktycznie ciekawych. :)

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
    Constructor.newInstance(Object... initargs). IllegalAccessException zgłaszany jest w momencie, kiedy konstruktor
    jest niedostępny (zgodnie z modyfikatorami dostępu). Jeśli czytasz ten tekst uważnie (w co niestety wątpię), to
    zauważyłeś zapewne pewien problem. Jakim cudem mogę nie mieć dostępu do konstruktora, skoro *Class.getConstructor()*
    zwraca tylko publicze konstruktory? Bardzo dobre spostrzeżenie. Fakt, nasz konstruktor z całą pewnością będzie publiczny,
    ale nie zapominajmy o tym, że istnieją inne sposoby na uzyskanie referencji do konstruktorów niepublicznych.
    Dokładniej opiszę ten precedens w jednym z późniejszych rozdziałów.
* ***InvocationTargetException*** zgłaszany jest, gdy konstruktor który wywołujemy rzuci wyjątek.
* ***InstantiationException*** zgłaszany jest, gdy próbujemy stworzyć instancję klasy abstrakcyjnej.
