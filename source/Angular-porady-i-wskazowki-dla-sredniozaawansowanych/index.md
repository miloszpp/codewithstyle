---
title: Angular - porady i wskazówki dla średniozaawansowanych
url: 560.html
id: 560
comments: false
date: 2017-11-29 22:08:51
---

Angular - porady i wskazówki dla średniozaawansowanych
======================================================

Przeszedłeś przez kurs lub szkolenie z Angulara i zbudowałeś już swoją pierwszą aplikację? Być może znasz już dobrze AngularJS (czyli _starego_ Angulara w wersji 1.x), łatwo Ci było ogarnąć podstawy _nowego_ Angulara, ale nie wiesz za bardzo co dalej? Ten artykuł jest właśnie dla Ciebie! Przedstawię w nim kilka technik i podejść, które warto poznać żeby móc wykorzystać w pełni moc tego wspaniałego frameworka.

Poznaj Angulara od środka
-------------------------

Frameworki powstają żeby ułatwić nam życie i abyśmy nie musieli za każdym razem implementować od zera wszystkich funkcjonalności. Nie znaczy to jednak, że framework jest czarną skrzynką, której działania nie musimy rozumieć. Dlatego jeśli chciałbyś stać się lepszym programistą Angular to powinieneś zacząć od zastanowienia się jak niektóre mechanizmy są w nim zaimplementowane.

Wykrywanie zmian
----------------

Jedną z podstawowych funkcjonalności Angulara jest **data binding**. To za jego sprawą zmiany w modelu danych są automatycznie odzwierciedlane w widoku. Mechanizmem odpowiedzialnym za data binding jest **change detection**. W dużym uproszczeniu, change detection polega na porównaniu wartości użytych w property bindings przed zajściem jakiegoś wydarzenia (np. kliknięcia przycisku lub nadejścia odpowiedzi z serwera aplikacyjnego) z wartościami po zajściu tego wydarzenia. Jeśli istniał handler, który zaktualizował jakąś wartość, to framework wykrywa takę zmianę i musi nanieść ją na widok. Dla każdego komponentu w naszej aplikacji Angular generuje **Change Detector**, którego rolą jest właśnie porównywanie wartości.

### Ręczne wywoływanie Change Detectora

Łatwo dojść do wniosku, że bardzo wiele cykli change detection wykonuje się niepotrzebnie. Przykładowo, jeśli danym wydarzeniem jest zainteresowany tylko jeden komponent, to nie ma potrzeby wywoływania **Change Detectorów** dla wszystkich komponentów za każdym razem. Angular pozwala nam w prosty sposób _odłączyć_ Change Detector od komponentu. Wystarczy wstrzyknąć obiekt `ChangeDetectorRef` i wywołać na nim metodę `detach`.

    constructor(ref: ChangeDetectorRef) {
      ref.detach();
    }
    

Musimy jednak pamiętać, że w takim przypadku widok nie będzie reagował na zmiany w modelu danych. Jeśli chcemy odświeżyć widok musimy wywołać wykrywanie zmian ręcznie. Poniższy przykład pokazuje jak to zrobić. Widok zostanie odświeżony tylko po kliknięciu przycisku _Detect changes_.

    @Component({
      selector: 'app-change-detection',
      template: `
        <p>
          Counter: { { counter }}
          <button (click)="increment()">Increment</button>
          <button (click)="detect()">Detect changes</button>
        </p>
      `,
    })
    export class ChangeDetectionComponent {
      counter = 0;
    
      constructor(private ref: ChangeDetectorRef) { ref.detach(); }
      increment() { this.counter++; }
      detect() { this.ref.detectChanges(); }
    }
    

Dzięki takiemu podejściu możemy znacząco poprawić wydajność naszej aplikacji poprzez uniknięcie niepotrzebnych wykonań **Change Detectorów**.

### Strategia `OnPush`

Ręczne sterowanie mechanizmem wykrywania zmian może okazać się nie lada wyzwaniem. Innym pomysłem na usprawnienie tego procesu, który jednak nie zmusza nas do rezygnacji z automatycznego wykrywania zmian jest użycie strategii wykrywania zmian `OnPush`. Wyobraźmy sobie, że mamy komponent `BookDetailsComponent`, który zawiera wewnątrz komponent `AuthorDetailsComponent`, jak poniżej.

    { { book.title }}
    <app-author-details [author]="author"></app-author-details>
    <button (click)="changeName()">Change author name</button>
    

Domyślnie po wykrywaniu zmian dla komponentu `BookDetailsComponent` Angular uruchomi wykrywanie zmian dla komponentu `AuthorDetailsComponent`. Jednakże po włączeniu strategii `OnPush`, Angualr uruchomi Change Detector dla zagnieżdżonego komponentu tylko i wyłącznie gdy wykryje zmianę w polu `author`. Pozwoli nam to oszczędzić sporo czasu na wykrywaniu zmian.

    @Component({
      selector: 'app-author-details',
      changeDetection: ChangeDetectionStrategy.OnPush
    })
    

Z takim podejściem wiąże się jednak zmiana sposobu pisania kodu w naszej aplikacji. Po włączeniu strategii `OnPush` poniższy kod nie spowoduje odświeżenia widoku:

    changeName() {
      this.author.name = "GEORGE";
    }
    

Nie zmieniła się wartość pola `author` \- zmieniło się tylko pole w tym obiekcie. Musimy zatem inaczej wyrazić naszą intencję:

    changeName() {
      this.author = { ...this.author, name: "GEORGE" };
    }
    

Takie podejście do pisania kodu wiąże się z pojęciem **niemutowalności**. Zamiast modyfikować obiekty, tworzymy ich nowe wersje.

Dependency Injection
--------------------

Kolejnym mechanizmem w Angularze wartym zgłębienia jest **Dependency Injection**. Zakładam, że rozumiesz na podstawowym poziomie do czego służy Dependency Injection.

### Drzewo Injectorów

Za każdym razem gdy wstrzykujemy coś za pomocą Dependency Injection, Angular odwołuje się do obiektu `Injector`. Obiekt ten potrafi zwrócić nam instancję danej usługi gdy o nią poprosimy. Warto wiedzieć, że Injectorów w aplikacji jest więcej niż jeden. Każda aplikacja zawiera jeden **root Injector**. Dla każdego komponentu tworzony jest jednak **child Injector**. Mamy zatem w zasadzie drzewo **Injectorów** o kształcie przypominającym drzewo komponentów. Jeśli jakiś komponent pyta o usługę, która nie jest zarejestrowana we właściwym dla niego **Injectorze**, to zapytanie to jest przekazywane w górę drzewa. W większości przypadków odpowie na nie dopiero **root Injector**, w którym to zarejestrowane jest większość serwisów (za pomocą `providers` w dekoratorze głównego modułu). ![](https://codewithstyle.info/wp-content/uploads/2017/11/InjectorTree.png) Taki mechanizm umożliwia Angularowi wstrzykiwanie obiektów, które istnieją tylko w kontekście danego komponentu. Przykładowo, jeśli implementujemy własną dyrektywę, to może wstrzyknąć obiekt `ElementRef`, który daje nam dostęp do drzewa DOM związanego z dyrektywą. Oczywiście istnieje wiele innych zastosowań drzewa **Injectorów**. Przykładowo, możemy w komponencie za pomocą `providers` podmienić implementację danej usługi na bardziej wyspecjalizowaną. Możemy również ograniczyć dostęp do danej usługi tylko do jednego komponentu, ponieważ wiemy że tylko tam usługa będzie potrzebna. Warto zatem pamiętać, że usługi można rejestrować zarówno na poziomie modułu jak i komponentu.

### Moduły shared i core oraz lazy loading

Powyższa własność Angulara ma poważne konsekwencje, które mogą nas zaboleć jeśli używamy _lazy loadingu_. Okazuje się bowiem, że każdy moduł załadowany leniwie powoduje utworzenie nowego child injectora. Oznacza to zatem, że jeśli dana usługa jest rejestrowana zarówno przez moduł główny oraz moduł załadowany leniwie, to komponenty z modułu leniwego dostaną inną instancję usługi niż komponenty z modułu głównego! Na ogół jednak spodziewamy się, że serwis będzie **singletonem**. W związku z powyższym oficjalny Angular Style Guide sugeruje umieszczenie usług wspólnych dla wszystkich modułów w module `Core` (w przeciwieństwie do modułu `Shared`). Moduł ten jest importowany tylko raz, w module głównym. Moduł `Shared` jest za to importowany we wszystkich innych modułach, przez co nie powinien zawierać rejestracji usług. Jest za to dobrym miejscem na deklarowanie dyrektyw i komponentów wspólnych dla wszystkich modułów. W prostych słowach można ująć to następująco: * Jeśli chcesz uwspólnić dla wielu modułów komponent, dyrektywę lub pipe, to umieść ją w module `Shared` oraz zaimportuj ten moduł we wszystkich innych modułach. * Jeśli chcesz uwspólnić serwis to umieść go w module `Core` oraz zaimportuj ten moduł jedynie w głównym module.

RxJS
----

Ostatnie zagadnienie, które chciałbym poruszyć, to używanie `Observables`. Jak wiemy, obiekty typu `Observable` reprezentują strumień wartości, które zostaną wyemitowane w późniejszym czasie. `Observables` są obecne w Angularze w kilku miejscach. Jednym z nich jest klasa `HttpClient`, której używamy na co dzień do komunikacji z serwerem (tzw. backendem). W celu wysłania zapytania do serwera możemy wywołać metodę `HttpClient.get`, która zwróci nam właśnie `Observable`. W tym przypadku owym _strumieniem_ będzie pojedyncza wartość (lub błąd) zwrócona przez serwer. Podczas nauki Angulara zapewne używałeś klasy `HttpClient` w następujący sposób:

    ngOnInit() {
      this.httpClient.get<Post[]>(this.postsUrl).subscribe(posts => {
        this.posts = posts;
      });
    }
    

W podejściu tym subskrybujemy `Observable` podając funkcję która ma się wykonać gdy wyemituje on wartość. W naszym przypadku funkcja ta przypisze tablicę postów do pola `posts`, którego możemy użyć w szablonie.

    <ul>
      <li *ngFor="let post of posts">{ { post.title }}</li>
    </ul>
    

Warto wiedzieć, że Angular umożliwia uzyskanie takiego samego efektu w prostszy sposób. Z pomocą przychodzi `AsyncPipe`. Jest to _pipe_ dzięki któremu możemy w szablonie użyć bindingu bezpośrednio do `Observable` dzięki czemu `subscribe` nie jest potrzebnej.

    ngOnInit() {
      this.postsObservable = this.httpClient.get<Post[]>(this.postsUrl);
    }
    

Zauważ użycie `async` w szablonie:

    <ul>
      <li *ngFor="let post of postsObservable | async">{ { post.title }}</li>
    </ul>
    

Bardzo często mamy do czynienia z sytuacją, że chcielibyśmy wyświetlić w szablonie jakąś treść zanim dane z serwera zostaną załadowane. Dyrektywa `*ngIf` posiada użyteczną acz mniej znaną składnię dzięki której możemy w elegancki sposób zaimplementować taki scenariusz.

    <ul *ngIf="postsObservable | async as posts; else loading">
      <li *ngFor="let post of posts">{ { post.title }}</li>
    </ul>
    <ng-template #loading>Loading...</ng-template>
    

Jak widać, nie dość że `*ngIf` pozwala nam przypisać wartość _odpakowaną_ z `Observable` do nowej zmiennej (`posts`), to jeszcze umożliwia nam podanie szablonu, który wyświetli się w przypadku gdy wartość przed średnikiem nie konwertuje się do `true`.

Podsumowanie
------------

W artykule tym omówiłem kilka mechanizmów związanych z Angularem, które nie pojawiają się na ogół w kursach dla początkujących, a o których na pewno warto wiedzieć. Po raz kolejny podkreślę, że warto spędzić trochę czasu na próbie zrozumienia w jaki sposób Angular działa _pod spodem_. Dzięki głebokiemu zrozumieniu frameworka unikniemy błędów oraz będziemy lepiej rozumieć jak poprawić wydajność naszych aplikacji.