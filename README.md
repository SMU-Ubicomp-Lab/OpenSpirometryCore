# OpenSpirometry Core: an Open Source Spirometer for iOS

This project is part of an open source project created by researchers at Southern Methodist University. Collaboration on the project also involves the Unversity of Washington. 
* http://www.smuubicomp.org
* https://ubicomplab.cs.washington.edu

Using a special whistle with this code base allows individuals to perform a forced expiratory maneuver while the smartphone microphone listens to the frequency of the whistle. This code is currently in alpha and is very much a work in progress. Many updates will be coming. This process is **NOT FDA approved** (yet) for providing Spirometric measures. We are currently underway using human subjects testing to verify the efficacy of this application. While these values might be close to the values you would receive from a Spirometry test, we make no legal guarantees about the measurements. Consult a Doctor if you think your pulmonary ailment is worsening. 

If you are the type of programmer that does best with reading the documents first, this is the place to start. If you would rather see an example of the app running in side a basic UI example, then head over to this repository:
* [Basic UI Example using OpenSpirometryCore]:https://github.com/SMU-Ubicomp-Lab/OpenSpirometryBasicExample

An extended example of using this code with more advanced interactivity and graphics is currently in development. 

## Spirometry Effort Analyzer for Vortex Whistles

In a nutshell, you can use this code in yours by placing all the code into a new group in Xcode. There are a decent amount of dependency classes, but the only one you will use is the "SpirometerEffortAnalyzer" and most of its processing power is exposed through delegation.  

The ViewController has an example of how to use every public function for the given model: make it a property in your class and set yourself as the delegate. If the delegate is a UIViewController, then some additional functionality is provided for asking permission to use the microphone (i.e., recording). Use the "askPermissionToUseAudioIfNotDone" function to access this.

You control when the effort is started using the function "beginListeningForEffort" and that kicks off the entire test. You can cancel the test at any time using the "requestThatCurrentEffortShouldCancel" function. It will clean up and try to cancel the effort (it might fail if the effort is already analyzing).

Since you are the delegate for the model (of type "SpirometerEffortDelegate") you get access to some notifications of the progress of the test. All notifications are given to you on the main thread because they are mostly meant to be used for updating UI. So if you want to do some intense processing from one of these notifications, you should do it on a separate queue (see Apple's documentation for *concurrency*, if you are unsure what I mean here). All of these functions are used in the example view controller:

``` objective-c
@protocol SpirometerEffortDelegate
-didFinishCalibratingSilence:
-didTimeoutWaitingForTestToStart:
-didStartExhaling: 
-willEndTestSoon: 
-didCancelEffort:
-didEndEffortWithResults: NSDictionary
-didUpdateFlow:(float)flow andVolume:(float)volume:
-didUpdateAudioBufferWithMaximum:(float)maxAudioValue

// With methods for initiating the effort:
-beginListeningForEffort
-askPermissionToUseAudioIfNotDone
-requestThatCurrentEffortShouldCancel

// And Methods for using and saving the audio of the effort:
-activateDebugAudioModeWithWAVFile:(NSString*)filenameAndPath
-shouldSaveSeparateEffortsToDocumentDirectory:(BOOL)should

// With various properties:
@property SpirometerEffortDelegate delegate;
@property SpirometryStage currentStage;
@property SpirometryWhistle* whistle;
@property float prefferredAudioMaxUpdateIntervalInSeconds;
```

**didFinishCalibratingSilence**: get notified when the background noise level has been calibrated. Example: Maybe tell the user they can start exhaling.

**didTimeoutWaitingForTestToStart**: get notified that no sound was heard for ~5 seconds after silence was calibrated and the effort canceled itself. Example: inform the user that this happened in the UI.

**didStartExhaling**: notified that the user started blowing. Example: Tell them to keep blasting out.

**willEndTestSoon**: notified that the test "might" end soon because the audio sounds are getting faint. You can get this notification multiple times, especially if the sound level is getting louder and softer periodically. Example: Tell the user to keep going because the effort is almost over.

**didCancelEffort**: Get notified that your request to cancel the effort was successful. Example: tell user test was cancelled.

**didEndEffortWithResults**: Get notified that the test is over and the results are passed in through a dictionary of key/value and key/array pairs. This part of the model is still getting built out so the dictionary only has keys, but the values and arrays are currently just empty NSArrays and NSNumbers. In the future the curves and the measures will be in this dictionary. The efforts will also have errors for the test (Cough is "semi-implemented" but that is the only error you might see--the detection may also be a bit sensitive right now).

**didUpdateFlow**:(float)flow andVolume:(float)volume: This function is called every time we detect a reasonable flow rate value. So it means you will get notified many times per second (but it not guaranteed to be consistent--if no valid frequency is found or the processing gets bogged down, this could get called at weird times). The flow rate will currently show up (not calibrated, it just moves in response to frequency). Volume will always currently be zero. Example: Update a part of your UI showing flow rate. The example has this connected to a slider and text label.

Right now this code is in beta, so if you post it on GitHub, be sure to say that it is not ready to be forked yet.

## Third Party Frameworks/Libraries

* [Novocaine]:https://github.com/alexbw/novocaine . Professor Larson manipulated this for iOS8 and built completely for objective-c in iOS. It isn't really backwards compatible with Alex's library anymore--although you should definitely check out Novocaine. He uses it in his iOS course here at SMU.  


## Building Vortex Whistles
Still to come: we are in the process of getting an open source page for building your owm custom whistles via 3D printing and providing calibration of the whistle coefficients and bias.


