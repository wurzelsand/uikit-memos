# Ein ImageView soll sich wie ein Button verhalten

## Aufgabe

<a><img src="media/button-from-image.gif" width=300></a>

## Ausführung

Zunächst fügen wir im *Storyboard* zwei *ImageViews* hinzu und und verbinden sie mit zwei *Outlets*: `saveImageView` und `cancelImageView`. Mit diesen *ImageViews* werden `ButtonFromImage`-Objekte erzeugt. `ButtonFromImage`-Objekte reagieren auf `touchesBegan`- und `touchesEnded`-Aufrufe des `ViewControllers`. Außerdem erhält das *ImageView* in `ButtonFromImage` einen `UITapGestureRecognizer`. So können die *ImageViews* darauf reagieren, wenn sie berührt und losgelassen wurden.

```swift
import UIKit
import QuartzCore
import Combine

class ButtonFromImage: NSObject, CAAnimationDelegate {
    private let imageView: UIImageView
    private let touchesBeganObserver: AnyCancellable
    private let touchesEndedObserver: AnyCancellable
    private let completion: (() -> Void)?
    
    init(_ imageView: UIImageView,
         in viewController: ViewController,
         completion: (() -> Void)? = nil)
    {
        self.imageView = imageView
        self.completion = completion
        self.touchesBeganObserver = viewController.$beganTouches
            .sink(receiveValue: { touches in
                if let touch = touches.first, touch.view == imageView {
                    imageView.transform = CGAffineTransform(scaleX: 0.9, y: 0.9)
                }
            })
        self.touchesEndedObserver = viewController.$endedTouches
            .sink(receiveValue: { touches in
                if let touch = touches.first, touch.view == imageView {
                    imageView.transform = CGAffineTransform(scaleX: 1, y: 1)
                }
            })
        super.init()
        let tapGestureRecognizer = UITapGestureRecognizer(
            target: self,
            action: #selector(imageTapped(tapGestureRecognizer:)))
        self.imageView.addGestureRecognizer(tapGestureRecognizer)
        self.imageView.isUserInteractionEnabled = true
    }
    
    @objc func imageTapped(tapGestureRecognizer: UITapGestureRecognizer) {
        guard let image = tapGestureRecognizer.view as? UIImageView else {
            return
        }
        let keyframeAnimation = CAKeyframeAnimation(keyPath: "transform.scale")
        keyframeAnimation.keyTimes = [0.2, 0.4, 1.0]
        keyframeAnimation.values = [0.9, 1.2, 1.0]
        keyframeAnimation.duration = 0.3
        keyframeAnimation.delegate = self
//      keyframeAnimation.timingFunction = CAMediaTimingFunction(name: .easeOut)
        image.layer.add(keyframeAnimation, forKey: "pop")
        image.transform = CGAffineTransform(scaleX: 1, y: 1)
    }
    
    func animationDidStop(_ anim: CAAnimation, finished flag: Bool) {
        completion?()
    }

}

class ViewController: UIViewController {
    @IBOutlet weak var saveImageView: UIImageView!
    @IBOutlet weak var cancelImageView: UIImageView!
    @Published private(set) var beganTouches = Set<UITouch>()
    @Published private(set) var endedTouches = Set<UITouch>()
    
    var saveButton: ButtonFromImage?
    var cancelButton: ButtonFromImage?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        self.saveButton = ButtonFromImage(saveImageView, in: self) { [weak self] in
            let alert = UIAlertController(title: "Saved",
                                          message: nil,
                                          preferredStyle: .alert)
            alert.addAction(UIAlertAction(title: "OK", style: .cancel))
            self?.present(alert, animated: true)
        }
        self.cancelButton = ButtonFromImage(cancelImageView, in: self) { [weak self] in
            let alert = UIAlertController(title: "Cancelled",
                                          message: nil,
                                          preferredStyle: .alert)
            alert.addAction(UIAlertAction(title: "OK", style: .cancel))
            self?.present(alert, animated: true)
        }
    }
    
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        super.touchesBegan(touches, with: event)
        beganTouches = touches
    }
    
    override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
        super.touchesEnded(touches, with: event)
        endedTouches = touches
    }
}
```

## Diskussion

* Der *Button* reagiert nicht genauso wie ein `UIButton`: Wenn der *Button* z.B. zu lange gedrückt wird, wird kein *Tap* ausgelöst.
* Gezeigt wird, dass man mit `CAKeyframeAnimation` *Views* animieren kann.
* Mit `Published` Variablen, kann man Werteänderungen, ähnlich wie in SwiftUI, an *Subscriber* weitergeben. Alternativ hätte man auch ein `PassthroughSubject` verwenden können: [swiftbysundell](https://www.swiftbysundell.com/articles/building-custom-combine-publishers-in-swift/)

