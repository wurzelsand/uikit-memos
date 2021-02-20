# Swipe Pages

## Aufgabe:

<img src="media/swipe-pages.gif">

Ich habe einen View-Typ für Monate und einen für Wochentage. Sie sollen eine horizontale Reihe bilden, die ich mit dem Finger verschieben kann.

## Ausführung:

* Entferne aus **Info.plist** *Application Scene Manifest* und *Main storyboard file base name*.
* Entferne **SceneDelegate.swift**.
* Entferne **Main.storyboard**.
* Entferne **ViewController.swift**.

**AppDelegate.swift**:

```swift
import UIKit

@main
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?

    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
        window = UIWindow()
        window?.makeKeyAndVisible()
        
        let layout = UICollectionViewFlowLayout()
        layout.scrollDirection = .horizontal
        let swipingController = SwipingController(collectionViewLayout: layout)
        window?.rootViewController = swipingController
        
        return true
    }

}
```

**SwipingController.swift**:

```swift
import UIKit

class SwipingController: UICollectionViewController, UICollectionViewDelegateFlowLayout {
    
    let months = ["January", "February"]
    let days = ["Monday", "Tuesday", "Wednesday"]
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        collectionView?.backgroundColor = .red
        collectionView?.register(DayCell.self, forCellWithReuseIdentifier: "days")
        collectionView?.register(MonthCell.self, forCellWithReuseIdentifier: "months")
        collectionView?.isPagingEnabled = true
    }
    
    func collectionView(_ collectionView: UICollectionView,
                        layout collectionViewLayout: UICollectionViewLayout,
                        minimumLineSpacingForSectionAt section: Int) -> CGFloat {
        return 0
    }
    
    override func numberOfSections(in collectionView: UICollectionView) -> Int {
        return 2
    }
    
    override func collectionView(_ collectionView: UICollectionView,
                                 numberOfItemsInSection section: Int) -> Int {
        switch section {
        case 0:
            return months.count
        case 1:
            return days.count
        default:
            return 0
        }
    }
    
    override func collectionView(_ collectionView: UICollectionView,
                                 cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        
        switch indexPath.section {
        case 0:
            if let cell = collectionView.dequeueReusableCell(
                withReuseIdentifier: "months",
                for: indexPath) as? MonthCell {
                    // `indexPath.item` identical to `indexPath.row`; `row`
                    // should be used in the context of tables (in terms of
                    // programming style):
                    cell.text = months[indexPath.item]
                    return cell
                }
        case 1:
            if let cell = collectionView.dequeueReusableCell(
                withReuseIdentifier: "days",
                for: indexPath) as? DayCell {
                    cell.text = days[indexPath.item]
                    return cell
                }
        default:
            ()
        }
        // empty cell:
        return UICollectionViewCell()
    }
    
    func collectionView(_ collectionView: UICollectionView,
                        layout collectionViewLayout: UICollectionViewLayout,
                        sizeForItemAt indexPath: IndexPath) -> CGSize {
        return CGSize(width: view.frame.width, height: view.frame.height)
    }
    
    override func viewWillTransition(to size: CGSize,
                                     with coordinator: UIViewControllerTransitionCoordinator) {
        collectionViewLayout.invalidateLayout()
    }
}
```

**DayCell.swift**:

```swift
import UIKit

class DayCell: UICollectionViewCell {
    
    var text = "" {
        didSet {
            label.text = text
            label.sizeToFit()
        }
    }
    
    private var label = UILabel()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        backgroundColor = .yellow
        addSubview(label)
        label.textColor = .black
        label.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate(
            [label.topAnchor.constraint(equalTo: safeAreaLayoutGuide.topAnchor),
             label.leadingAnchor.constraint(equalTo: safeAreaLayoutGuide.leadingAnchor)])
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

**MonthCell.swift** (hier fast identisch mit **DayCell.swift**):

```swift
import UIKit

class MonthCell: UICollectionViewCell {
    
    var text = "" {
        didSet {
            label.text = text
            label.sizeToFit()
        }
    }
    
    private var label = UILabel()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        backgroundColor = .green
        addSubview(label)
        label.textColor = .black
        label.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate(
            [label.topAnchor.constraint(equalTo: safeAreaLayoutGuide.topAnchor),
             label.leadingAnchor.constraint(equalTo: safeAreaLayoutGuide.leadingAnchor)])
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```
