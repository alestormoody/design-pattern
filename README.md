# Design Patterns in PHP

Questo repository contiene esempi di implementazioni di vari Design Patterns in PHP. 
Ogni pattern include una breve descrizione, vantaggi, svantaggi e un esempio di codice commentato.

## Indice

1. [Singleton](#1-singleton)
2. [Factory](#2-factory)
3. [Observer](#3-observer)
4. [Strategy](#4-strategy)
5. [Decorator](#5-decorator)
6. [Adapter](#6-adapter)
7. [Command](#7-command)
8. [Proxy](#8-proxy)
9. [Facade](#9-facade)
10. [Composite](#10-composite)
11. [Builder](#11-builder)

## 1. Singleton

**Descrizione**: Garantisce che una classe abbia una sola istanza e fornisce un punto di accesso globale a quella istanza.

**Vantaggi**:
- Controllo centralizzato dell'istanza.
- Evita la creazione di più istanze di un oggetto.

**Svantaggi**:
- Può diventare difficile da testare.
- Potenziale violazione del principio di Single Responsibility.

**Esempio**:

```php
class Database {
    private static $instance = null;
    private $connection;

    private function __construct() {
        // Costruttore privato per prevenire l'istanziazione diretta
        $this->connection = new PDO('mysql:host=localhost;dbname=test', 'user', 'password');
    }

    public static function getInstance() {
        if (self::$instance === null) {
            // Crea l'istanza solo se non esiste già
            self::$instance = new self();
        }
        return self::$instance;
    }

    public function query($sql) {
        // Metodo per eseguire query
        return $this->connection->query($sql);
    }
}

// Uso
$db1 = Database::getInstance();
$db2 = Database::getInstance();

// $db1 e $db2 sono la stessa istanza
```

## 2. Factory

**Descrizione**: Definisce un'interfaccia per creare oggetti, ma lascia alle sottoclassi la decisione di quale tipo di oggetto istanziare.

**Vantaggi**:
- Centralizzazione della logica di creazione.
- Facilità di estensione.

**Svantaggi**:
- Aumenta il numero di classi nel progetto.
- Può complicare la comprensione del flusso del programma.

**Esempio**:

```php
interface Vehicle {
    public function drive();
}

class Car implements Vehicle {
    public function drive() {
        return "Guido un'auto";
    }
}

class Motorcycle implements Vehicle {
    public function drive() {
        return "Guido una moto";
    }
}

class VehicleFactory {
    public static function createVehicle($type) {
        switch ($type) {
            case 'car':
                return new Car();
            case 'motorcycle':
                return new Motorcycle();
            default:
                throw new Exception("Tipo di veicolo non supportato");
        }
    }
}

// Uso
$car = VehicleFactory::createVehicle('car');
echo $car->drive(); // Output: Guido un'auto

$moto = VehicleFactory::createVehicle('motorcycle');
echo $moto->drive(); // Output: Guido una moto
```

## 3. Observer

**Descrizione**: Permette a un oggetto (soggetto) di notificare automaticamente gli oggetti osservatori quando il suo stato cambia.

**Vantaggi**:
- Promuove il loose coupling tra il soggetto e gli osservatori.
- Facile aggiungere nuovi osservatori.

**Svantaggi**:
- Può causare una notevole quantità di notifiche in sistemi complessi.
- Le dipendenze possono diventare difficili da tracciare.

**Esempio**:

```php
interface Observer {
    public function update($message);
}

class NewsPublisher {
    private $observers = [];
    private $latestNews = "";

    public function attach(Observer $observer) {
        $this->observers[] = $observer;
    }

    public function setNews($news) {
        $this->latestNews = $news;
        $this->notifyObservers();
    }

    private function notifyObservers() {
        foreach ($this->observers as $observer) {
            $observer->update($this->latestNews);
        }
    }
}

class NewsSubscriber implements Observer {
    private $name;

    public function __construct($name) {
        $this->name = $name;
    }

    public function update($message) {
        echo $this->name . " ha ricevuto la notizia: " . $message . "\n";
    }
}

// Uso
$publisher = new NewsPublisher();

$subscriber1 = new NewsSubscriber("Alice");
$subscriber2 = new NewsSubscriber("Bob");

$publisher->attach($subscriber1);
$publisher->attach($subscriber2);

$publisher->setNews("Nuova notizia importante!");
// Output:
// Alice ha ricevuto la notizia: Nuova notizia importante!
// Bob ha ricevuto la notizia: Nuova notizia importante!
```

## 4. Strategy

**Descrizione**: Definisce una famiglia di algoritmi, incapsula ciascuno di essi e li rende intercambiabili.

**Vantaggi**:
- Isola il comportamento dal contesto.
- Facilita l'aggiunta di nuovi algoritmi.

**Svantaggi**:
- Aumento del numero di classi.
- Potenziale aumento della complessità nella gestione degli algoritmi.

**Esempio**:

```php
interface SortStrategy {
    public function sort(array $data): array;
}

class BubbleSort implements SortStrategy {
    public function sort(array $data): array {
        // Implementazione del Bubble Sort
        $n = count($data);
        for ($i = 0; $i < $n - 1; $i++) {
            for ($j = 0; $j < $n - $i - 1; $j++) {
                if ($data[$j] > $data[$j + 1]) {
                    $temp = $data[$j];
                    $data[$j] = $data[$j + 1];
                    $data[$j + 1] = $temp;
                }
            }
        }
        return $data;
    }
}

class QuickSort implements SortStrategy {
    public function sort(array $data): array {
        // Implementazione del Quick Sort (semplificata)
        if (count($data) <= 1) {
            return $data;
        }
        $pivot = $data[0];
        $left = $right = [];
        for ($i = 1; $i < count($data); $i++) {
            if ($data[$i] < $pivot) {
                $left[] = $data[$i];
            } else {
                $right[] = $data[$i];
            }
        }
        return array_merge($this->sort($left), [$pivot], $this->sort($right));
    }
}

class Sorter {
    private $sortStrategy;

    public function __construct(SortStrategy $sortStrategy) {
        $this->sortStrategy = $sortStrategy;
    }

    public function sort(array $data): array {
        return $this->sortStrategy->sort($data);
    }
}

// Uso
$data = [64, 34, 25, 12, 22, 11, 90];

$bubbleSorter = new Sorter(new BubbleSort());
print_r($bubbleSorter->sort($data));

$quickSorter = new Sorter(new QuickSort());
print_r($quickSorter->sort($data));
```

## 5. Decorator

**Descrizione**: Permette di aggiungere funzionalità a un oggetto in modo dinamico, senza modificarne la struttura originale.

**Vantaggi**:
- Permette di aggiungere nuove funzionalità senza modificare il codice esistente.
- Consente di combinare diversi comportamenti in modo flessibile.

**Svantaggi**:
- La combinazione di più decoratori può rendere il codice difficile da comprendere.
- Aumento del numero di classi.

**Esempio**:

```php
interface Coffee {
    public function getCost();
    public function getDescription();
}

class SimpleCoffee implements Coffee {
    public function getCost() {
        return 10;
    }

    public function getDescription() {
        return "Caffè semplice";
    }
}

abstract class CoffeeDecorator implements Coffee {
    protected $coffee;

    public function __construct(Coffee $coffee) {
        $this->coffee = $coffee;
    }

    public function getCost() {
        return $this->coffee->getCost();
    }

    public function getDescription() {
        return $this->coffee->getDescription();
    }
}

class Milk extends CoffeeDecorator {
    public function getCost() {
        return $this->coffee->getCost() + 2;
    }

    public function getDescription() {
        return $this->coffee->getDescription() . ", con latte";
    }
}

class Sugar extends CoffeeDecorator {
    public function getCost() {
        return $this->coffee->getCost() + 1;
    }

    public function getDescription() {
        return $this->coffee->getDescription() . ", con zucchero";
    }
}

// Uso
$coffee = new SimpleCoffee();
echo $coffee->getCost() . ": " . $coffee->getDescription() . "\n";

$coffee = new Milk($coffee);
echo $coffee->getCost() . ": " . $coffee->getDescription() . "\n";

$coffee = new Sugar($coffee);
echo $coffee->getCost() . ": " . $coffee->getDescription() . "\n";
```

## 6. Adapter

**Descrizione**: Consente di adattare un'interfaccia di una classe a un'altra interfaccia, rendendo classi incompatibili compatibili.

**Vantaggi**:
- Permette di usare classi esistenti senza modificarle.
- Aumenta la riusabilità del codice.

**Svantaggi**:
- Aumenta la complessità del sistema introducendo nuovi adattatori.
- Richiede una comprensione più profonda della struttura delle classi.

**Esempio**:

```php
interface MediaPlayer {
    public function play($audioType, $fileName);
}

interface AdvancedMediaPlayer {
    public function playVlc($fileName);
    public function playMp4($fileName);
}

class VlcPlayer implements AdvancedMediaPlayer {
    public function playVlc($fileName) {
        echo "Riproduzione file VLC: " . $fileName . "\n";
    }

    public function playMp4($fileName) {
        // Non supportato
    }
}

class Mp4Player implements AdvancedMediaPlayer {
    public function playVlc($fileName) {
        // Non supportato
    }

    public function playMp4($fileName) {
        echo "Riproduzione file MP4: " . $fileName . "\n";
    }
}

class MediaAdapter implements MediaPlayer {
    private $advancedMusicPlayer;

    public function __construct($audioType) {
        if ($audioType === "vlc") {
            $this->advancedMusicPlayer = new VlcPlayer();
        } elseif ($audioType === "mp4") {
            $this->advancedMusicPlayer = new Mp4Player();
        }
    }

    public function play($audioType, $fileName) {
        if ($audioType === "vlc") {
            $this->advancedMusicPlayer->playVlc($fileName);
        } elseif ($audioType === "mp4") {
            $this->advancedMusicPlayer->playMp4($fileName);
        }
    }
}

class AudioPlayer implements MediaPlayer {
    private $mediaAdapter;

    public function play($audioType, $fileName) {
        if ($audioType === "mp3") {
            echo "Riproduzione file MP3: " . $fileName . "\n";
        } elseif ($audioType === "vlc" || $audioType === "mp4") {
            $this->mediaAdapter = new MediaAdapter($audioType);
            $this->mediaAdapter->play($audioType, $fileName);
        } else {
            echo "Formato audio non supportato: " . $audioType . "\n";
        }
    }
}

// Uso
$audioPlayer = new AudioPlayer();
$audioPlayer->play("mp3", "song.mp3");
$audioPlayer->play("vlc", "movie.vlc");
$audioPlayer->play("mp4", "video.mp4");
$audioPlayer->play("avi", "video.avi");
```

## 7. Command

**Descrizione**: Incapsula una richiesta come oggetto, permettendo di parametrizzare gli oggetti con richieste.

**Vantaggi**:
- Supporta operazioni di undo/redo.
- Separa la logica di invocazione dalla logica di esecuzione.

**Svantaggi**:
- Introduce una complessità significativa se usato in modo errato.
- Aumento del numero di classi.

**Esempio**:

```php
interface Command {
    public function execute();
}

class Light {
    public function turnOn() {
        echo "La luce è accesa\n";
    }

    public function turnOff() {
        echo "La luce è spenta\n";
    }
}

class LightOnCommand implements Command {
    private $light;

    public function __construct(Light $light) {
        $this->light = $light;
    }

    public function execute() {
        $this->light->turnOn();
    }
}

class LightOffCommand implements Command {
    private $light;

    public function __construct(Light $light) {
        $this->light = $light;
    }

    public function execute() {
        $this->light->turnOff();
    }
}

class RemoteControl {
    private $command;

    public function setCommand(Command $command) {
        $this->command = $command;
    }

    public function pressButton() {
        $this->command->execute();
    }
}

// Uso
$light = new Light();
$lightOn = new LightOnCommand($light);
$lightOff = new LightOffCommand($light);

$remote = new RemoteControl();

$remote->setCommand($lightOn);
$remote->pressButton(); // Output: La luce è accesa

$remote->setCommand($lightOff);
$remote->pressButton(); // Output: La luce è spenta
```

## 8. Proxy

**Descrizione**: Fornisce un sostituto di un oggetto per controllare l'accesso a un altro oggetto.

**Vantaggi**:
- Ottimizza l'accesso a oggetti remoti o costosi.
- Può gestire la sicurezza, il caching, o il logging senza modificare l'oggetto reale.

**Svantaggi**:
- Introduce un livello di astrazione che può ridurre la performance se non ben progettato.
- Aumenta la complessità del sistema.

**Esempio**:

```php
class RealImage implements Image {
    private $filename;

    public function __construct($filename) {
        $this->filename = $filename;
        $this->loadFromDisk();
    }

    private function loadFromDisk() {
        echo "Caricamento dell'immagine " . $this->filename . "\n";
    }

    public function display() {
        echo "Visualizzazione dell'immagine " . $this->filename . "\n";
    }
}

class ProxyImage implements Image {
    private $realImage;
    private $filename;

    public function __construct($filename) {
        $this->filename = $filename;
    }

    public function display() {
        if ($this->realImage == null) {
            $this->realImage = new RealImage($this->filename);
        }
        $this->realImage->display();
    }
}

// Uso
$image = new ProxyImage("foto.jpg");
// L'immagine verrà caricata e visualizzata
$image->display();
// L'immagine non verrà caricata di nuovo, ma solo visualizzata
$image->display();
```

## 9. Facade

**Descrizione**: Fornisce un'interfaccia semplificata per un insieme di interfacce di un sottosistema.

**Vantaggi**:
- Riduce la complessità esposta agli utenti.
- Facilita l'uso di sistemi complessi, facendo da punto di accesso centralizzato.

**Svantaggi**:
- Potenziale abuso che può mascherare la necessità di un design più modulare.
- Può diventare un punto di bottleneck se non ben progettato.

**Esempio**:

```php
// Sottosistema complesso
class SubsystemA {
    public function operationA() {
        return "Operazione A\n";
    }
}

class SubsystemB {
    public function operationB() {
        return "Operazione B\n";
    }
}

class SubsystemC {
    public function operationC() {
        return "Operazione C\n";
    }
}

// Facade
class Facade {
    private $subsystemA;
    private $subsystemB;
    private $subsystemC;

    public function __construct() {
        $this->subsystemA = new SubsystemA();
        $this->subsystemB = new SubsystemB();
        $this->subsystemC = new SubsystemC();
    }

    // Metodo semplificato che usa i sottosistemi
    public function operation() {
        $result = "Facade inizializza i sottosistemi:\n";
        $result .= $this->subsystemA->operationA();
        $result .= $this->subsystemB->operationB();
        $result .= $this->subsystemC->operationC();
        return $result;
    }
}

// Uso
$facade = new Facade();
echo $facade->operation();
```

## 10. Composite

**Descrizione**: Consente di trattare gli oggetti singoli e i gruppi di oggetti in modo uniforme.

**Vantaggi**:
- Facilita la gestione di strutture ad albero.
- Può essere utilizzato per rappresentare sia oggetti singoli che composti senza dover cambiare il codice che li manipola.

**Svantaggi**:
- Può risultare eccessivamente generico e poco flessibile per strutture non gerarchiche.
- Potrebbe complicare il codice con una logica di gestione complessa.

**Esempio**:

```php
// Componente
interface Component {
    public function operation(): string;
}

// Foglia
class Leaf implements Component {
    private $name;

    public function __construct(string $name) {
        $this->name = $name;
    }

    public function operation(): string {
        return "Leaf " . $this->name . "\n";
    }
}

// Composito
class Composite implements Component {
    private $children = [];

    public function add(Component $component): void {
        $this->children[] = $component;
    }

    public function operation(): string {
        $results = [];
        foreach ($this->children as $child) {
            $results[] = $child->operation();
        }
        return "Composite (" . implode("+", $results) . ")\n";
    }
}

// Uso
$leaf1 = new Leaf("1");
$leaf2 = new Leaf("2");
$leaf3 = new Leaf("3");

$composite = new Composite();
$composite->add($leaf1);
$composite->add($leaf2);

$composite2 = new Composite();
$composite2->add($leaf3);
$composite2->add($composite);

echo $composite2->operation();
```

## 11. Builder

**Descrizione**: Consente di costruire un oggetto complesso passo per passo, separando la costruzione dall'oggetto stesso.

**Vantaggi**:
- Permette di creare oggetti complessi senza costringere il costruttore a gestire troppe variabili.
- Flessibilità nel costruire oggetti con molte opzioni.

**Svantaggi**:
- Introduce una complessità aggiuntiva nella costruzione dell'oggetto.
- Necessita di un numero maggiore di classi.

**Esempio**:

```php
// Prodotto
class Product {
    private $parts = [];

    public function add(string $part): void {
        $this->parts[] = $part;
    }

    public function show(): void {
        echo "Parti del prodotto: " . implode(', ', $this->parts) . "\n";
    }
}

// Builder
interface Builder {
    public function buildPartA(): void;
    public function buildPartB(): void;
    public function getResult(): Product;
}

// ConcreteBuilder
class ConcreteBuilder implements Builder {
    private $product;

    public function __construct() {
        $this->product = new Product();
    }

    public function buildPartA(): void {
        $this->product->add("Parte A");
    }

    public function buildPartB(): void {
        $this->product->add("Parte B");
    }

    public function getResult(): Product {
        return $this->product;
    }
}

// Director
class Director {
    public function construct(Builder $builder): void {
        $builder->buildPartA();
        $builder->buildPartB();
    }
}

// Uso
$director = new Director();
$builder = new ConcreteBuilder();
$director->construct($builder);
$product = $builder->getResult();
$product->show();
```

