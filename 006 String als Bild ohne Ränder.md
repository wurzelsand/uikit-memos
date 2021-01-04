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
        let attributedString = NSAttributedString(
            string: "Zapfino",
            attributes: [NSAttributedString.Key.font: UIFont(name: "Zapfino",
                                                             size: 65)!])
        let textConverter = AttributedStringToImageConverter(singleLineText: attributedString)
        let image = textConverter?.createImage()!
        let imageView = UIImageView(image: image)
        imageView.backgroundColor = .yellow
        imageView.center = view.center
        view.addSubview(imageView)
    }

}

/**
 Wrapper for a single line NSAttributedString to create a frameless or centered image.
 */
class AttributedStringToImageConverter {
    
    struct FrameError: Error {}
    
    private let text: NSAttributedString
    private var textBounds = CGRect()
    private var canvas = CGRect()
    
    init?(singleLineText: NSAttributedString) {
        self.text = singleLineText
        self.textBounds = text.boundingRect(
            with: .zero,
            options: NSStringDrawingOptions.usesLineFragmentOrigin,
            context: nil)
        self.increaseTextBoundsForSpecialFonts()
    }
    
    /// Fonts like Zapfino need some extra space
    private func increaseTextBoundsForSpecialFonts() {
        // trial and error:
        textBounds.size.width += textBounds.height
        textBounds.size.height *= 1.5
    }

    /**
     Create a UIImage without margins, or centered if size explicitly defined.
     - Parameter singleLine: Single line of `NSAttributedString` or
     `NSMutableAttributedString`. May consists of variously formatted words.
     - Parameter size: If defined, the image is centered within size.
     - Returns: Text as image.
     */
    func createImage(size: CGSize? = nil) -> UIImage? {
        let isBookedForCropping = (size == nil)
        if let size = size {
            canvas.size = size
        } else {
            canvas.size = textBounds.size
        }
        UIGraphicsBeginImageContext(canvas.size)
        guard let context = UIGraphicsGetCurrentContext() else { return nil }
        flipCanvasVertical(context: context)
        guard let _ = try? drawToCanvasCenter(in: context) else { return nil }
        let image = UIGraphicsGetImageFromCurrentImageContext()
        UIGraphicsEndImageContext()
        if isBookedForCropping {
            let croppingBounds = getCroppingBounds()
            let croppedImage = image?.cgImage?.cropping(to: croppingBounds)
            return croppedImage.map { UIImage(cgImage: $0) }
        }
        return image
    }
    
    private func flipCanvasVertical(context: CGContext) {
        context.textMatrix = .identity
        context.translateBy(x: 0, y: canvas.height)
        context.scaleBy(x: 1, y: -1)
    }
    
    private func drawToCanvasCenter(in context: CGContext) throws {
        context.addRect(textBounds)
        let ctFrame = createCTFrame()
        try moveToCanvasCenter(context, ctFrame)
        CTFrameDraw(ctFrame, context)
    }
    
    private func getCroppingBounds() -> CGRect {
        return CGRect(x: (canvas.width - textBounds.width) / 2,
                      y: (canvas.height - textBounds.height) / 2,
                      width: textBounds.width,
                      height: textBounds.height)
    }
    
    private func createCTFrame() -> CTFrame {
        let framesetter = CTFramesetterCreateWithAttributedString(text)
        let path = CGMutablePath()
        path.addRect(textBounds)
        let ctFrame = CTFramesetterCreateFrame(framesetter, CFRange(), path, nil)
        return ctFrame
    }
    
    private func moveToCanvasCenter(_ context: CGContext, _ ctFrame: CTFrame) throws {
        try shrinkTextBounds(context, ctFrame)
        let dx = canvas.midX - textBounds.midX
        let dy = canvas.midY - textBounds.midY
        context.translateBy(x: dx, y: dy)
    }
    
    private func shrinkTextBounds(_ context: CGContext, _ ctFrame: CTFrame) throws {
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
    
}
```

## Diskussion

Statt `CTLineGetBoundsWithOptions` ginge auch `CTLineGetImageBounds`.
