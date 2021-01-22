# String als Bild ohne Ränder

## Aufgabe

<a><img src="media/zapfino-cropped.png" width = 400></a>

## Ausführung

```swift
import UIKit

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .blue
        let image = createImage(string: "Zapfino", font: UIFont(name: "Zapfino", size: 65)!)
        let imageView = UIImageView(image: image)
        imageView.backgroundColor = .yellow
        imageView.center = view.center
        view.addSubview(imageView)
    }

}

/**
 Create a UIImage without margins, or centered if size explicitly defined.
 - Parameter string: Single line of text.
 - Parameter font: Font for `string`
 - Parameter size: If defined, the image is centered within `size`. Otherwise the image is cropped to fit its text.
 - Returns: Text as image.
 */
func createImage(string: String, font: UIFont, size: CGSize? = nil) -> UIImage? {
    let attributedString = NSAttributedString(
        string: string,
        attributes: [NSAttributedString.Key.font: font])
    return createImage(with: attributedString, size: size)
}

/**
 - Parameter text: Single line of `NSAttributedString` or
 `NSMutableAttributedString`. May consists of variously formatted words.
 */
func createImage(with text: NSAttributedString, size: CGSize? = nil) -> UIImage? {
    var measurings = TextMeasurings(singleLine: text)
    if let size = size {
        measurings.canvas.size = size
    }
    UIGraphicsBeginImageContext(measurings.canvas.size)
    guard let context = UIGraphicsGetCurrentContext() else { return nil }
    measurings.flipCanvasVertical(context: context)
    guard let _ = try? measurings.drawToCanvasCenter(in: context) else { return nil }
    let image = UIGraphicsGetImageFromCurrentImageContext()
    UIGraphicsEndImageContext()
    if size == nil {
        let croppingBounds = measurings.croppingBounds
        let croppedImage = image?.cgImage?.cropping(to: croppingBounds)
        return croppedImage.map { UIImage(cgImage: $0) }
    }
    return image
}

private struct TextMeasurings {
    private struct FrameError: Error {}
    
    private var text = NSAttributedString()
    private var textBounds = CGRect()
    var canvas = CGRect()
    
    init(singleLine: NSAttributedString) {
        self.text = singleLine
        self.textBounds = self.text.boundingRect(
            with: .zero,
            options: NSStringDrawingOptions.usesLineFragmentOrigin,
            context: nil)
        self.increaseTextBoundsForSpecialFonts()
        self.canvas.size = self.textBounds.size
    }
    
    /// Fonts like Zapfino need some extra space
    private mutating func increaseTextBoundsForSpecialFonts() {
        // trial and error:
        textBounds.size.width += textBounds.height
        textBounds.size.height *= 1.5
    }
    
    func flipCanvasVertical(context: CGContext) {
        context.textMatrix = .identity
        context.translateBy(x: 0, y: canvas.height)
        context.scaleBy(x: 1, y: -1)
    }
    
    mutating func drawToCanvasCenter(in context: CGContext) throws {
        context.addRect(textBounds)
        let ctFrame = createCTFrame()
        try moveToCanvasCenter(context, ctFrame)
        CTFrameDraw(ctFrame, context)
    }
    
    private func createCTFrame() -> CTFrame {
        let framesetter = CTFramesetterCreateWithAttributedString(text)
        let path = CGMutablePath()
        path.addRect(textBounds)
        let ctFrame = CTFramesetterCreateFrame(framesetter, CFRange(), path, nil)
        return ctFrame
    }
    
    private mutating func moveToCanvasCenter(_ context: CGContext, _ ctFrame: CTFrame) throws {
        try shrinkTextBounds(context, ctFrame)
        let dx = canvas.midX - textBounds.midX
        let dy = canvas.midY - textBounds.midY
        context.translateBy(x: dx, y: dy)
    }
    
    private mutating func shrinkTextBounds(_ context: CGContext, _ ctFrame: CTFrame) throws {
        guard let lines = CTFrameGetLines(ctFrame) as? [CTLine],
              let firstLine = lines.first else { throw FrameError() }
        let textBoundsRelatedToLineOrigin =
            CTLineGetBoundsWithOptions(firstLine,
                                       CTLineBoundsOptions.useGlyphPathBounds)
        var origins = [CGPoint](repeating: .zero, count: lines.count)
        CTFrameGetLineOrigins(ctFrame, CFRange(), &origins)
        guard let firstOrigin = origins.first else { throw FrameError() }
        textBounds = CGRect(
            x: firstOrigin.x + textBoundsRelatedToLineOrigin.origin.x,
            y: firstOrigin.y + textBoundsRelatedToLineOrigin.origin.y,
            width: textBoundsRelatedToLineOrigin.width,
            height: textBoundsRelatedToLineOrigin.height)
    }
    
    var croppingBounds: CGRect {
        CGRect(x: (canvas.width - textBounds.width) / 2,
               y: (canvas.height - textBounds.height) / 2,
               width: textBounds.width,
               height: textBounds.height)
    }
}
```

## Diskussion

Statt `CTLineGetBoundsWithOptions` ginge auch `CTLineGetImageBounds`. Leider berücksichtigt keine der beiden Methoden Texteffekte wie z.B. Schatten.
