---
Date: 2025-03-16
Title: Building a Real-Time Audio Visualizer in SwiftUI
---

*A step-by-step guide to creating a radial audio visualizer that responds to microphone input*

## Introduction

In this tutorial, we'll build "AudioWave" - a SwiftUI app that creates a beautiful radial visualization of audio captured from the device's microphone. This project demonstrates several important iOS development concepts:

- Working with microphone permissions and audio recording
- Processing real-time audio data
- Creating custom visualizations with SwiftUI
- Handling iOS permissions properly

Let's dive in and see how we can bring audio to life visually!

## Project Setup

First, create a new SwiftUI project in Xcode:

1. Open Xcode and select "Create a new Xcode project"
2. Choose "App" as the template
3. Name your project "AudioWave"
4. Select "SwiftUI" for the interface and "Swift" for the language
5. Choose a location to save your project

## Step 1: Setting Up Microphone Permissions

iOS requires explicit permission to access the microphone. We need to:

1. Add a usage description to our Info.plist
2. Request permission at runtime

Create or modify your Info.plist file to include:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>NSMicrophoneUsageDescription</key>
    <string>AudioWave needs microphone access to visualize audio.</string>
    <key>CFBundleVersion</key>
    <string>1</string>
    <key>CFBundleShortVersionString</key>
    <string>1.0</string>
    <key>CFBundleIdentifier</key>
    <string>com.example.AudioWave</string>
    <key>CFBundleName</key>
    <string>AudioWave</string>
    <key>CFBundleExecutable</key>
    <string>$(EXECUTABLE_NAME)</string>
    <key>CFBundlePackageType</key>
    <string>APPL</string>
</dict>
</plist>
```

The `NSMicrophoneUsageDescription` key is crucial - it provides the message shown to users when requesting microphone access.

## Step 2: Creating the Audio Manager

Next, we'll create an `AudioManager` class to handle microphone access and process audio data:

```swift
// AudioManager.swift
import Foundation
import AVFoundation
import Accelerate

class AudioManager: ObservableObject {
    private var audioRecorder: AVAudioRecorder?
    private var timer: Timer?
    
    @Published var soundSamples: [CGFloat] = Array(repeating: 0.0, count: 30)
    
    init() {
        setupAudioRecorder()
    }
    
    func setupAudioRecorder() {
        let audioSession = AVAudioSession.sharedInstance()
        
        do {
            try audioSession.setCategory(.playAndRecord, mode: .default)
            try audioSession.setActive(true)
            
            let url = URL(fileURLWithPath: "/dev/null", isDirectory: true)
            let recorderSettings: [String: Any] = [
                AVFormatIDKey: Int(kAudioFormatAppleLossless),
                AVSampleRateKey: 44100.0,
                AVNumberOfChannelsKey: 1,
                AVEncoderAudioQualityKey: AVAudioQuality.high.rawValue
            ]
            
            audioRecorder = try AVAudioRecorder(url: url, settings: recorderSettings)
            audioRecorder?.isMeteringEnabled = true
            audioRecorder?.record()
            
            startMonitoring()
        } catch {
            print("AudioManager initialization failed: \(error.localizedDescription)")
        }
    }
    
    func startMonitoring() {
        timer = Timer.scheduledTimer(withTimeInterval: 0.03, repeats: true) { [weak self] _ in
            guard let self = self else { return }
            self.audioRecorder?.updateMeters()
            
            // Get the power level from the recorder
            let power = self.audioRecorder?.averagePower(forChannel: 0) ?? -160
            // Convert to a 0-1 scale (dB is logarithmic, typically -160 to 0)
            let normalizedPower = (power + 50) / 50
            
            // Add the new value and remove the oldest one
            self.soundSamples.removeFirst()
            self.soundSamples.append(CGFloat(max(0, min(normalizedPower, 1))))
        }
    }
    
    func stopMonitoring() {
        timer?.invalidate()
        timer = nil
        audioRecorder?.stop()
    }
    
    deinit {
        stopMonitoring()
    }
}
```

This class:
- Sets up an audio session and recorder
- Creates a timer to regularly sample audio levels
- Normalizes the audio power levels to a 0-1 scale
- Maintains an array of recent samples for visualization
- Provides methods to start and stop monitoring

## Step 3: Building the Radial Visualizer

Now, let's create a custom SwiftUI view to visualize our audio data:

```swift
// RadialVisualizerView.swift
import SwiftUI

struct RadialVisualizerView: View {
    var samples: [CGFloat]
    var color: Color = .blue
    
    var body: some View {
        GeometryReader { geometry in
            let size = min(geometry.size.width, geometry.size.height)
            let middle = CGPoint(x: geometry.size.width / 2, y: geometry.size.height / 2)
            let radius = size / 2.5
            
            ZStack {
                // Background circle
                Circle()
                    .fill(color.opacity(0.1))
                    .frame(width: radius * 2, height: radius * 2)
                
                // Visualization bars
                ForEach(0..<samples.count, id: \.self) { index in
                    let angle = 2 * .pi / CGFloat(samples.count) * CGFloat(index)
                    let barHeight = radius * samples[index] * 0.8
                    
                    Rectangle()
                        .fill(color)
                        .frame(width: 3, height: barHeight)
                        .cornerRadius(1.5)
                        .offset(y: -radius - barHeight / 2)
                        .rotationEffect(.radians(Double(angle)))
                }
            }
            .position(middle)
        }
    }
}

#Preview {
    RadialVisualizerView(samples: Array(repeating: CGFloat.random(in: 0.1...1.0), count: 30), color: .blue)
        .frame(width: 300, height: 300)
        .background(Color.black.opacity(0.1))
}
```

This view:
- Takes an array of audio samples and a color as input
- Creates a circular background
- Draws bars radiating outward from the center
- Positions each bar at the correct angle
- Sizes each bar based on the corresponding audio sample value

## Step 4: Putting It All Together in ContentView

Finally, let's update our ContentView to integrate the AudioManager and RadialVisualizerView:

```swift
// ContentView.swift
import SwiftUI
import AVFoundation
import AVFAudio

struct ContentView: View {
    @StateObject private var audioManager = AudioManager()
    @State private var isRecording = false
    
    var body: some View {
        VStack {
            Text("AudioWave")
                .font(.largeTitle)
                .fontWeight(.bold)
                .padding()
            
            RadialVisualizerView(samples: audioManager.soundSamples, color: .blue)
                .frame(height: 300)
                .padding()
                .background(
                    RoundedRectangle(cornerRadius: 16)
                        .fill(Color.black.opacity(0.05))
                )
                .padding()
            
            Button(action: {
                if isRecording {
                    audioManager.stopMonitoring()
                } else {
                    audioManager.setupAudioRecorder()
                }
                isRecording.toggle()
            }) {
                Text(isRecording ? "Stop Listening" : "Start Listening")
                    .padding()
                    .background(isRecording ? Color.red : Color.blue)
                    .foregroundColor(.white)
                    .cornerRadius(10)
            }
            .padding()
        }
        .onAppear {
            // Request microphone permission when the view appears
            if #available(iOS 17.0, *) {
                AVAudioApplication.requestRecordPermission(completionHandler: { granted in
                    if granted {
                        print("Microphone access granted")
                    } else {
                        print("Microphone access denied")
                    }
                })
            } else {
                // Fallback for iOS versions before 17.0
                AVAudioSession.sharedInstance().requestRecordPermission { granted in
                    if granted {
                        print("Microphone access granted")
                    } else {
                        print("Microphone access denied")
                    }
                }
            }
        }
    }
}
```

This view:
- Creates an instance of our AudioManager
- Displays the app title
- Shows the RadialVisualizerView with current audio samples
- Provides a button to start/stop audio monitoring
- Requests microphone permission when the view appears
- Handles compatibility with different iOS versions

## Troubleshooting Common Issues

During development, we encountered several issues that you might also face:

### 1. Missing AVFoundation Import

If you see an error like `Cannot find 'AVAudioSession' in scope`, make sure to import AVFoundation:

```swift
import AVFoundation
```

### 2. Multiple Info.plist Files

If you get an error about multiple Info.plist files, you need to configure your project settings:

1. Go to your project settings
2. Set `GENERATE_INFOPLIST_FILE = NO`
3. Set `INFOPLIST_FILE = AudioWave/Info.plist`

### 3. Missing Required Info.plist Keys

iOS requires certain keys in your Info.plist:

- `CFBundleVersion` - The build number
- `CFBundleShortVersionString` - The version number
- `CFBundleIdentifier` - The bundle identifier
- `NSMicrophoneUsageDescription` - Explanation for microphone use

### 4. Deprecated API Methods

For iOS 17+, use the updated microphone permission request method:

```swift
if #available(iOS 17.0, *) {
    AVAudioApplication.requestRecordPermission(completionHandler: { granted in
        // Handle permission result
    })
} else {
    AVAudioSession.sharedInstance().requestRecordPermission { granted in
        // Handle permission result
    }
}
```

## Conclusion

Congratulations! You've built a real-time audio visualizer in SwiftUI. This project demonstrates:

- Working with device sensors (microphone)
- Processing real-time data
- Creating custom visualizations
- Handling permissions properly
- Supporting different iOS versions

You can extend this project in many ways:
- Add different visualization styles
- Allow users to customize colors and sensitivity
- Save and share visualizations
- Add audio effects or filters

The combination of SwiftUI's powerful drawing capabilities with AVFoundation's audio processing makes it possible to create beautiful, responsive audio visualizations with relatively little code.

Happy coding!

---

