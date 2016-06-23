## Patterns pour les Tester en Utilisant des Protocoles.

Une des choses vraiment bien à propos de l'utilisation des protocoles à la place des classes, c'est que ça définie une collection de méthodes à utiliser, mais tout en ne forçant pas la relation entre des implémentations spécifiques.

Cela fonctionne vraiment bien, parce que c'est très facile d'échanger l'objet dans le test, car il faut juste etre conforme au protocole voulu.
Jettons un coup d'oeil à un exemple astucieux à tester `UITableViewDataSource`.

Cet exemple a une classe qui doit être responsable de récupérer les données et de les fournir à une tableview.

``` swift
class ORArtworkDataSource, NSObject, UITableViewDataSource {
//Faire appel au réseau, récupérer des données; rendre possible la génération de cellules
    func getData() {
      [...]
    }
    [...]
}

class ORArtworkViewController: UITableViewController {
    var dataSource: ORArtworkDataSource!

    [...]
    override func viewDidLoad() {
        dataSource = ORArtworkDataSource()
        tableView.dataSource = dataSource
        dataSource.getData()
    }
}

Cette implémentation est bonne si vous ne voulez pas écrire de tests, mais cela devient compliqué pour trouver un moyen d'y insérer et effectuer vos tests tout en respectant le comportement voulu avec cette façon de faire.'


Une des plus simple approches de rendre ce type de code facile à tester est d'utiliser l'initialisation lazy et un protocole pour définir les attentes, mais pas l'implémentation.

Donc, définissons un protocole qui dit quelles méthodes le `ORArtworkDataSource` doit avoir et seulement alors laissons `ORArtworkViewController` savoir si ça communique avec quelque chose qui est conforme à ce protocole. 


```swift

// Ce protocole abstrait les détails de l'implémentation de l'appel au réseau
protocol ORArtworkDataSourcable {
    func getData()
}

class ORArtworkDataSource: NSObject, ORArtworkDataSourcable, UITableViewDataSource {
	//Faire appel au réseau, récupérer des données; rendre possible la génération de cellules
    func getData() {
        [...]
    }

    [...]
}

class ORArtworkViewController: UITableViewController {

    // Permettre une autre classe de changer le dataSource
    // mais aussi retournera à ORArtworkDataSource()
    // quand il n'est pas allouer.
    lazy var dataSource: ORArtworkDataSourcable = {
      return ORArtworkDataSource()
    }()

    [...]
    override func viewDidLoad() {
        tableView.dataSource = dataSource
        dataSource.getData()
    }
}

```

Cela vous permet de créer un nouvel objet qui est conforme à `ORArtworkDataSourcable` qui peut avoir différents comportements dans les tests. Par exemple:

```swift
it("shows a tableview cell") {
    subject = ORArtworkViewController()
    subject.dataSource = ORStubbedArtworkDataSource()
    // [...]
    expect(subject.tableView.cellForRowAtIndexPath(index)).to( beTruthy() )
}
```

Il y a une excellente vidéo d'Apple appelée [Protocol-Oriented Programming in Swift](https://developer.apple.com/videos/play/wwdc2015/408/) qui couvre le sujet et plus encore. La vidéo a un bon exemple qui montre comment vous pouvez tester une interface graphique en comparant les log des fichiers, car l'abstraction est couverte ici.

Les mêmes principes s'appliquent en Objective-C aussi, ne pensez pas que c'est une chose spéciale à Swift, le principal nouveau changement pour Swift est la possibilité pour un protocole d'offrir des méthodes permettant un étrange héritage multiple.

Exemples en pratique, la plupart sur des modèles NetworkM

* Eigen - [ARArtistNetworkModel](https://github.com/artsy/eigen/blob/da011cb4e0cd45e9148e89b92a4021ea3651753f/Artsy/Networking/Network_Models/ARArtistNetworkModel.h) is a protocol which `ARArtistNetworkModel` and `ARStubbedArtistNetworkModel` conform to. Here are some [tests using the technique.](https://github.com/artsy/eigen/blob/da011cb4e0cd45e9148e89b92a4021ea3651753f/Artsy_Tests/View_Controller_Tests/Artist/ARArtistViewControllerTests.m#L25)

* Eidolon - [BidderNetworkModel](https://github.com/artsy/eidolon/blob/16867a8de52fdf24db07937be003b6104c0ee5e9/Kiosk/Bid%20Fulfillment/BidderNetworkModel.swift) `BidderNetworkModelType` is a protocol that `BidderNetworkModel` and `StubBidderNetworkModel` conform to. Here are [tests using the technique](https://github.com/artsy/eidolon/blob/16867a8de52fdf24db07937be003b6104c0ee5e9/KioskTests/Bid%20Fulfillment/LoadingViewModelTests.swift)
