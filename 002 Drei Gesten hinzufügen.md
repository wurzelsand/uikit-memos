# Füge einem ImageView drei Gesten hinzu

## Aufgabe

<a><img src="media/add-three-gestures-to-imageview.gif" width=300></a>

## Ausführung

```swift
import UIKit

class ViewController: UIViewController {
    @IBOutlet weak var imageView: UIImageView!
    
    var angle: Float = 0
    var scale: Float = 1
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        imageView.isUserInteractionEnabled = true
        
        let dragged = #selector(ViewController.dragged(dragGesture:))
        let drag = UIPanGestureRecognizer(target: self, action: dragged)
        imageView.addGestureRecognizer(drag)
        
        let rotated = #selector(ViewController.rotated(rotationGesture:))
        let rotationGesture = UIRotationGestureRecognizer(target: self, action: rotated)
        imageView.addGestureRecognizer(rotationGesture)
        
        let pinched = #selector(ViewController.pinched(pinchGesture:))
        let pinchGesture = UIPinchGestureRecognizer(target: self, action: pinched)
        imageView.addGestureRecognizer(pinchGesture)
    }
    
    @objc func dragged(dragGesture: UIPanGestureRecognizer) {
        switch dragGesture.state {
        case .began, .changed:
            var newPosition = dragGesture.translation(in: view)
            newPosition.x += dragGesture.view!.center.x
            newPosition.y += dragGesture.view!.center.y
            dragGesture.view?.center = newPosition
            // reset:
            dragGesture.setTranslation(CGPoint.zero, in: view)
        default:
            ()
        }
    }
    
    @objc func rotated(rotationGesture: UIRotationGestureRecognizer) {
        switch rotationGesture.state {
        case .began, .changed:
            angle += Float(rotationGesture.rotation)
            let transform = CGAffineTransform.identity
                .rotated(by: CGFloat(angle))
                .scaledBy(x: CGFloat(scale), y: CGFloat(scale))
            rotationGesture.view?.transform = transform
            // reset:
            rotationGesture.rotation = .zero
        default:
            ()
        }
    }

    @objc func pinched(pinchGesture: UIPinchGestureRecognizer) {
        switch pinchGesture.state {
        case .began, .changed:
            scale *= Float(pinchGesture.scale)
            let transform = CGAffineTransform.identity
                .rotated(by: CGFloat(angle))
                .scaledBy(x: CGFloat(scale), y: CGFloat(scale))
            pinchGesture.view?.transform = transform
            // reset:
            pinchGesture.scale = 1
        default:
            ()
        }
    }
}
```
