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
metody - publiczną i prywatną. Metody te nie są statyczne, więc musimy uzyskać referencję do klasy Sokrates, a następnie
stworzyć nową instancję.
```jshelllanguage
public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
    Class<?> klasaSokratesa = Class.forName("com.github.wojtechm.refleksja.rozdzial_02.Sokrates");
    Object sokrates = klasaSokratesa.getConstructor().newInstance();
}
```
Jak widzisz w przykładzie powyżej, w naprawdę niewielkim kawałku kodu operującym refleksją, naprawdę wiele rzeczy może
pójść nie po naszej myśli. 2 linijki kodu i 5 kontrolowanych wyjątków. Tym tytułem od teraz wyjątki które już zostały 
już opisane, będą przeze mnie ignorowane. A skoro o omawianiu wyjątków mowa:

* **ClassNotFoundException** jest wyjątkiem rzucanym przez metodę Class.forName(String className). Niezbyt odkrywczym będzie
 stwierdzenie, że zostaje on rzucony gdy podanej klasy nie udało się namierzyć. Podstawowym błędem popełnianym przez 
 ludzi podejmujących się walki z refleksją, jest podanie jedynie nazwy klasy, kiedy (zgodnie z dokumentacją) powinno
 się dostarczyć w pełni kwalifikowaną nazwę (ang. FQN), a więc nazwę klasy poprzedzoną jej pakietem.<br/>
 ```
Parameters: 
className - the fully qualified name of the desired class.
```