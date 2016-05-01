### Support du Multi-Device

Il y a deux principales manières de mettre en place une suite de tests gérant plusieurs types de devices et d'orientations. 

La manière facile: Lancez votre suite de tests plusieurs fois sur plusieurs devices, simulateurs et orientations.

La manière difficile: Mock et Stub de façon à prendre en charge le multi-device en une seule suite de tests.

#### Les Faux Devices

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
Cela permet d'être certain que tous les ViewControllers sont créés à la taille attendue. Vous pouvez alors utiliser votre propre logique pour déterminer iPhone vs iPad. Cq marche pour des cas simples, mais ce n'est pas optimal dans le paysage actuel des applications iOS.

#### Les Faux Traits