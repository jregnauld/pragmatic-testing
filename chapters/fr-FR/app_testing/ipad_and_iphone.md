### Support du Multi-Device

Il y a deux principales manières de mettre en place une suite de tests gérant plusieurs types de devices et d'orientations. 

La manière facile: Lancez votre suite de tests plusieurs fois sur plusieurs devices, simulateurs et orientations.

La manière difficile: Mock et Stub de façon à prendre en charge le multi-device en une seule suite de tests.

#### Device Fakes

Comme pour beaucoup de choses, c'était plus simple avant. Quand vous pouvez, configurez juste une taille de device and partez de là. Allez jeter un coup d'œil dans le projet Eigen - ARTestContext.m

TODO - Link ^

``` objc
static OCMockObject *ARPartialScreenMock;

@interface UIScreen (Prvate)
- (CGRect)_applicationFrameForInterfaceOrientation:(long long)arg1 usingStatusbarHeight:(double)arg2 ignoreStatusBar:(BOOL)ignore;
@end

+ (void)runAsDevice:(enum ARDeviceType)device
{
  [... setup]

  ARPartialScreenMock = [OCMockObject partialMockForObject:UIScreen.mainScreen];
  NSValue *phoneSize = [NSValue valueWithCGRect:(CGRect)CGRectMake(0, 0, size.width, size.height)];

  [[[ARPartialScreenMock stub] andReturnValue:phoneSize] bounds];
  [[[[ARPartialScreenMock stub] andReturnValue:phoneSize] ignoringNonObjectArgs] _applicationFrameForInterfaceOrientation:0 usingStatusbarHeight:0 ignoreStatusBar:NO];
}
```
Cela permet d'être certain que tous les ViewControllers sont créés à la taille attendue. Vous pouvez alors utiliser votre propre logique pour déterminer iPhone vs iPad. Ca marche pour des cas simples, mais ce n'est pas optimal dans le paysage actuel des applications iOS.

#### Trait Fakes

Les collections de trait sont maintenant la manière recommandée pour distinguer les devices, car un iPad peut maintenant afficher votre app dans un écran de la taille d'un iPhone. Vous ne pouvez pas vous baser sur le fait d'avoir une application à la même taille de l'écran. Cela devient plus complexe. 🎉

Ce n'est pas une partie où j'ai passé beaucoup de temps, donc considérez cette section comme une beta. Si quelqu'un veut aller plus loin, je serais intéréssé de savoir quel est l'élément clé à connaitre pour se former sur les collections et pour stubbé comme je l'ai fait avec `_applicationFrameForInterfaceOrientation:usingStatusbarHeight:ignoreStatusBar:`.

Chaque View ou View Controller (V/VC) a des traits de collections en read-only, les V/VCs peut écouter les traits qui changent et les re-organiser: Par exemple, nous avons une vue qui s'alloue elle-même lors d'un changement de collection:

TODO: Lier à AuctionBannerView.swift

``` swift
class AuctionBannerView: UIView {

  override func traitCollectionDidChange(previousTraitCollection: UITraitCollection?) {
    super.traitCollectionDidChange(previousTraitCollection)

    // Remove all subviews and call setupViews() again to start from scratch.
    subviews.forEach { $0.removeFromSuperview() }
    setupViews()
  }
}
```

Quand nous testons cette vue, nous stubbons le tableau de `traitCollection` et le trigger `traitCollectionDidChange`, nous l'avons fait dans notre librairie [Forgeries](https://github.com/ashfurrow/forgeries). Ca y ressemble beaucoup, avec l'environnement étant la V/VC.

``` objc
void stubTraitCollectionInEnvironment(UITraitCollection *traitCollection, id<UITraitEnvironment> environment) {
    id partialMock = [OCMockObject partialMockForObject:environment];
    [[[partialMock stub] andReturn:traitCollection] traitCollection];
    [environment traitCollectionDidChange:nil];
}
```
Essayons d'appliquer notre réfléxion sur les V/VCs sur chaque type d'environnement où nous voulons y écrire des tests.