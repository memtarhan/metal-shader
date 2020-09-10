# Metal Shader for ARKit
[Metal]: https://developer.apple.com/documentation/metal	"Apple's Metal"

Apple's Metal framework provides a simple way to build a custom rendering engine for augmented reality experiences. ARKit works perfectly with Metal, while displaying custom rendered content and/or custom view. 

A basic flow for ARKit is shown below: 

![img](https://docs-assets.developer.apple.com/published/ffb3831f78/c162c528-dc03-494d-a5da-c23a8691a98e.png)

AR experiences follows folllowing path: 

1. Retriving video frames 
2. Tracking information from the current session(or selected one)
3. Rendering the frames retrieved from the video as a background task 
4. Using tracked real world coordinates and drawing content on the camera

AR experience is mostly opening your camera and tracking the real world. It detects real world objects, faces, objects, surfaces, etc. 

For details below, you might want to check the source code provided. 



Step 1 & 2 

```swift
func updateGameState() {        
    guard let currentFrame = session.currentFrame else {
        return
    }
    
    updateSharedUniforms(frame: currentFrame)
    updateAnchors(frame: currentFrame)
    updateCapturedImageTextures(frame: currentFrame)
    
    if viewportSizeDidChange {
        viewportSizeDidChange = false
        
        updateImagePlane(frame: currentFrame)
    }
}
```

Step 3 & 4 

```swift
func updateCapturedImageTextures(frame: ARFrame) {
    // Create two textures (Y and CbCr) from the provided frame's captured image
    let pixelBuffer = frame.capturedImage
    if (CVPixelBufferGetPlaneCount(pixelBuffer) < 2) {
        return
    }
    capturedImageTextureY = createTexture(fromPixelBuffer: pixelBuffer, pixelFormat:.r8Unorm, planeIndex:0)!
    capturedImageTextureCbCr = createTexture(fromPixelBuffer: pixelBuffer, pixelFormat:.rg8Unorm, planeIndex:1)!
}

func createTexture(fromPixelBuffer pixelBuffer: CVPixelBuffer, pixelFormat: MTLPixelFormat, planeIndex: Int) -> MTLTexture? {
    var mtlTexture: MTLTexture? = nil
    let width = CVPixelBufferGetWidthOfPlane(pixelBuffer, planeIndex)
    let height = CVPixelBufferGetHeightOfPlane(pixelBuffer, planeIndex)
    
    var texture: CVMetalTexture? = nil
    let status = CVMetalTextureCacheCreateTextureFromImage(nil, capturedImageTextureCache, pixelBuffer, nil, pixelFormat, width, height, planeIndex, &texture)
    if status == kCVReturnSuccess {
        mtlTexture = CVMetalTextureGetTexture(texture!)
    }
    
    return mtlTexture
}
```



###### Rendering with Realistic Lighting  

```swift
var ambientIntensity: Float = 1.0
if let lightEstimate = frame.lightEstimate {
    ambientIntensity = Float(lightEstimate.ambientIntensity) / 1000.0
}
let ambientLightColor: vector_float3 = vector3(0.5, 0.5, 0.5)
uniforms.pointee.ambientLightColor = ambientLightColor * ambientIntensity
```

