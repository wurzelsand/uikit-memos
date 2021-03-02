# Verketten von affinen Transformationen

## Aufgabe

<a><img src="media/chain-transformations.gif"></a>

Wir haben eine Ableitung eines `UIViews`:

```swift
class RedView: UIView {
    override init(frame: CGRect) {
        super.init(frame: frame)
        self.isUserInteractionEnabled = true
        self.backgroundColor = .red
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        ...
    }
}
```

Wir wollen, dass alle sechs möglichen Kombinationsmöglichkeiten von 3 affinen Transformationen (Rotation um 90˚, X-Skalierung 50%, Verschiebung (x: 100, y: 200)) nacheinander durchlaufen werden, wenn auf das *View* geklickt wird.

## Ausführung

```swift
class RedView: UIView {
    override init(frame: CGRect) {
        super.init(frame: frame)
        self.isUserInteractionEnabled = true
        self.backgroundColor = .red
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    private struct Transformations {
        var transformation = CGAffineTransform.identity
        
        var rotate: Transformations {
            Transformations(transformation: transformation.rotated(by: .pi / 4))
        }
        
        var scale: Transformations {
            Transformations(transformation: transformation.scaledBy(x: 0.5, y: 1))
        }
        
        var translate: Transformations {
            Transformations(transformation: transformation.translatedBy(x: 100, y: 200))
        }
    }
    
    private static let identity = Transformations(transformation: CGAffineTransform.identity)
    let transformations = [
        identity.rotate.scale.translate,
        identity.rotate.translate.scale,
        identity.scale.rotate.translate,
        identity.scale.translate.rotate,
        identity.translate.rotate.scale,
        identity.translate.scale.rotate,
        identity
    ].map { $0.transformation }
    
    var count = 0
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        count %= transformations.count
        print(count)
        transform = transformations[count]
        count += 1
    }
}
```

## Diskussion

Die Zeile

```swift
identity.rotate.scale.translate
```

entspricht

1. Ausführungsreihenfolge rechts nach links(!):
  ```swift
  CGAffineTransform.identity.rotated(by: .pi / 4).scaledBy(x: 0.5, y: 1).translatedBy(x: 100, y: 200)
  ```

2. Ausführungsreihenfolge links nach rechts:
  ```swift
  CGAffineTransform(translationX: 100, y: 200)
              .concatenating(CGAffineTransform(scaleX: 0.5, y: 1))
              .concatenating(CGAffineTransform(rotationAngle: .pi / 4))
  ```
