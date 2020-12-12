# 003 Sprite Atlas

## Aufgabe

Die *Textures* sollen in einen *Texture Atlas* geschrieben werden, aber ein *Asset* soll dafür sorgen, dass es verschiedene Größen der *Textures* passend zum Handy-Typ gibt.

<a><img src="media/sprite-atlas-ipod-touch.png" width="240"></a>
<a><img src="media/sprite-atlas-iphone-8-plus.png" width="320"></a>

## Umsetzung

* Stelle im Storyboard die Klasse des Views von *UIView* auf *SKView* um:
  
  <a><img src="media/storyboard-skview.png" width="250"></a>

* Erstelle eine neue Datei, z.B. *"SKAssets.xcassets"*:

  <a><img src="media/asset-catalog.png" width="200"></a>
  
* Erstelle darin ein neues *Texture Set*:

  <a><img src="media/create-texture-set.png" width="300"></a>
  
* Füge erst mal ein einzelnes Sprite in drei Versionen hinzu:

  <a><img src="media/asset-catalog-circle.png" width="400"></a>
  
  Eigentlich reicht *2x* und *3x*, da *1x* nur von wirklich alten Handys benutzt werden: [iOS Resolution](https://ios-resolution.com), [iOS Device Display Summary](https://developer.apple.com/library/archive/documentation/DeviceInformation/Reference/iOSDeviceCompatibility/Displays/Displays.html)
