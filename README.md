# c2bluetooth

[![Dart](https://github.com/CrewLAB/c2bluetooth/actions/workflows/dart.yml/badge.svg)](https://github.com/CrewLAB/c2bluetooth/actions/workflows/dart.yml)

A flutter package designed to sit on top of [FlutterBleLib](https://github.com/dotintent/FlutterBleLib) and provide an easy API for accessing data from Concept2 PM5 Indoor rowing machines. This library implements the [Concept2 Performance Monitor Bluetooth Smart Communications Interface Definition](https://www.concept2.com/files/pdf/us/monitors/PM5_BluetoothSmartInterfaceDefinition.pdf) Specification.

## Roadmap

Currently the library only supports retrieving simple workout summary information from the erg (date and time of workout, duration of workout, distance, average strokes per minute) after the conclusion of a workout. More features beyond that are currently planned.

## Installation

To install this package, just like any other flutter package, it needs to be included in your pubspec.yaml. Here are some templates for doing so:

**Pub.dev version**
To install from pub.dev, use the following snippet:

[Snippet TBD - Package not yet published.]

**From git**

To install as a [git dependency](https://dart.dev/tools/pub/dependencies#git-packages), use the following snippet:

```yaml
  c2bluetooth:
    git:
      url: https://github.com/CrewLab/c2bluetooth
      ref: v0.1.1
```

In this example, the value of the `ref` setting determines what branch/tag/commit it will use. this is useful if you want to lock your install to a particular version. If you want to use the bleeding-edge version, set this to `main`.

*Note*: This snippet assumes you have git configured correctly to be able to access the repository over SSH and have the correct auth (i.e. ssh keys) to access it without typing in your credentials. See the [dart docs](https://dart.dev/tools/pub/dependencies#git-packages) for more information 

**Locally**
For the most bleeding-edge experience - or if you plan to make and test changes to this library in realtime, it is recommended that you clone the library and use a relative path dependency as shown:

```yaml
  c2bluetooth:
    path: ../c2bluetooth
```

## Usage
Similar to how the underlying bluetooth library works, pretty much everything begins with an instance of `ErgBleManager()`. For a complete example, see the [example app](example/)

### Creating a manager

```dart
ErgBleManager bleManager = ErgBleManager();
bleManager.init(); //ready to go!
```
### Scanning for devices
Next, you need to start scanning for available devices. This uses a Stream that returns instances of the `Ergometer` class. Each of these instances represents an erg and can be saved for later reuse.

**Important:** This library does not handle things like permissions and many of the later steps, this one included, will fail if bluetooth is off or if permissions are not correct. It is the responsibility of users of this API to ensure permissions are correct before beginning a bluetooth scan.

```dart
bleManager.startErgScan().listen((erg) {
	//your code for detecting an erg here.
});
```
This block of code is where you can do things like:
 - determine what erg(s) you want to work with (this can be based on name, user choice, or basicaly anything)
 - store the erg instance somewhere more permanent, like a variable in a scope thats outside of this function (this allows you to access it after you stop scanning)
 - call `bleManager.stopErgScan()` if you are done. For example, one way to immediately connect to the first erg found is to unconditionally call `stopErgScan` within this function. Don't forget to close the stream too!


### Connecting to an erg
Once you have the `Ergometer` instance for the erg you want to connect to, you can call `connectAndDiscover()` on it to connect.

```dart
await myErg.connectAndDiscover();
```

### Getting workout summaries
To get data from the erg, use one of the methods available in the `Ergometer` class. Currently this is only `monitorForWorkoutSummary()`. This is a stream that returns a `WorkoutSummary` object that allows you to access the data from a completed workout (this includes programmed pieces as well as "Just row" pieces that are longer than 1 minute)

```dart
myErg.monitorForWorkoutSummary().listen((workoutSummary) {
  //do whatever here
});
```

### Disconnecting
When you are done, make sure to disconnect from your erg:
```dart
await myErg.disconnectOrCancel();
```

## Unit Testing
Tests can be run with `flutter test` or `flutter test --coverage` for coverage information.

Generate a HTML coverage report with `genhtml coverage/lcov.info -o coverage/html` (may require installing something. see [here](https://stackoverflow.com/questions/50789578/how-can-the-code-coverage-data-from-flutter-tests-be-displayed)). This `coverage` directory is gitignored already.
