## Updating from 0.X or 1.X

This section holds info on major changes when moving from versions 0.X or 1.Y. If we've forgotten something important, please [file an issue](https://github.com/NetSparkleUpdater/NetSparkle/issues).

* Minimum .NET Framework requirement, if running on .NET Framework, is now .NET Framework 4.5.2 instead of 4.5.1. Otherwise, the core library is .NET Standard 2.0 compatible, which means you can use it in your .NET Core and .NET 5+ projects.
* Change of base namespace from `NetSparkle` to `NetSparkleUpdater`
* `Sparkle` renamed to `SparkleUpdater` for clarity
* More logs are now written via `LogWriter` to help you debug and figure out any issues that are going on
* The default `NetSparkleUpdater` package (`NetSparkleUpdater.SparkleUpdater`) **has no built-in UI**. Please use one of the NetSparkleUpdater packages with a UI if you want a built-in UI that is provided for you.
  * Note that if you do _not_ use a `UIFactory`, you **must** use the `CloseApplication` or `CloseApplicationAsync` events to close your application; otherwise, your downloaded update file will never be executed/read! The only exception to this is if you want to handle all aspects of installing the update package yourself.
* XML docs are now properly shipped with the code for all public and protected methods rather than being here in this README file
  * Enabled build time warnings for functions that need documentation that don't have it
* `SparkleUpdater` constructors now require an `ISignatureVerifier` in order to "force" you to choose your signature verification method
* `SecurityMode` is a new enum that defines which signatures are required and which signatures are not required
  * Added `SecurityMode.OnlyVerifySoftwareDownloads` if you want to verify only software download signatures and don't care about verifying your app cast or release notes
* UIs are now in different namespaces. If you want to use a UI, you must pass in a `UIFactory` that implements `IUIFactory` and handles showing/handling all user interface elements
  * `NetSparkleUpdater` offers basic, built-in UIs for WinForms, WPF, and Avalonia. Copy & paste these files to your project and modify them to make them work better in your project!
  * `SparkleUpdater` no longer holds its own `Icon`. This is now handled by the `UIFactory` object.
  * `HideReleaseNotes`, `HideRemindMeLaterButton`, and `HideSkipButton` are all handled by the `UIFactory` objects
* Added built-in UIs for WPF and [Avalonia](https://github.com/AvaloniaUI/Avalonia) 0.10.X.
* Localization capabilities are now non-functional and are expected to come back in a later version. See [this issue](https://github.com/NetSparkleUpdater/NetSparkle/issues/92). (Contributions are welcome!)
* Most `SparkleUpdater` elements are now configurable. For example, you can implement `IAppCastHandler` to implement your own app cast parsing and checking.
  * `IAppCastDataDownloader` to implement downloading of your app cast file
  * `IUpdateDownloader` to implement downloading of your actual binary/installer as well as getting file names from the server (if `CheckServerFileName` is `true`)
  * `IAppCastHandler` to implement your own app cast parsing
  * `ISignatureVerifier` to implement your own download/app cast signature checking. NetSparkleUpdater has built-in DSA and Ed25519 signature verifiers.
  * `IUIFactory` to implement your own UI
    * `IUIFactory` implementors must now have `ReleaseNotesHTMLTemplate` and `AdditionalReleaseNotesHeaderHTML` -- it's ok if these are `string.Empty`/`""`/`null`.
    * `IUIFactory` methods all now take a reference to the `SparkleUpdater` instance that called the method
  * `ILogger` to implement your own logger class (rather than being forced to subclass `LogWriter` like in previous versions)
  * `Configuration` subclasses now take an `IAssemblyAccessor` in their constructor(s) in order to define where assembly information is loaded from
  * Many `SparkleUpdater` functions are now virtual and thus more easily overridden for your specific use case
  * Several `ReleaseNotesGrabber` functions are now virtual as well.
* Many delegates, events, and functions have been renamed, removed, and/or tweaked for clarity and better use
  * `DownloadEvent` now has the `AppCastItem` that is being downloaded rather than being just the download path
  * `AboutToExitForInstallerRun`/`AboutToExitForInstallerRunAsync` has been renamed to `PreparingToExit`/`PreparingToExitAsync`, respectively
  * The `UserSkippedVersion` event has been removed. Use `UserRespondedToUpdate` instead.
  * The `RemindMeLaterSelected` event has been removed. Use `UserRespondedToUpdate` instead.
  * The `FinishedDownloading`/`DownloadedFileReady` events have been removed. Use `DownloadFinished` instead.
  * The `StartedDownloading` event has been removed and replaced with `DownloadStarted`
  * The `DownloadError` event has been removed and replaced with `DownloadHadError`
  * `Sparkle.RunUpdate` no longer exists. Use `SparkleUpdater.InstallUpdate` instead.
  * `Sparkle.DownloadPathForAppCastItem` -> `SparkleUpdater.GetDownloadPathForAppCastItem`
  * `AppCastItem.DownloadDSASignature` -> `AppCastItem.DownloadSignature`
  * `SilentModeTypes` enum renamed to `UserInteractionMode`
  * `Sparkle.SilentMode` renamed to `Sparkle.UserInteractionMode`
  * `UseSyncronizedForms` renamed to `ShowsUIOnMainThread`
* Samples have been updated and improved
  * Sample apps for [Avalonia](https://github.com/AvaloniaUI/Avalonia), WinForms, and WPF UIs
  * Sample app to demonstrate how to handle events yourself with your own, custom UI
* By default, the app cast signature file now has a `.signature` extension. The app cast downloader will look for a file with the old `.dsa` signature if data is not available or found in a `appcast.xml.signature` on your server. You can change the extension using the `signature-file-extension` option in the app cast generator and via the `XMLAppCast.SignatureFileExtension` property.
* `sparkle:dsaSignature` is now `sparkle:signature` instead. If no `sparkle:signature` is found, `sparkle:dsaSignature` will be used (if available). If `sparkle:dsaSignature` is not found, `sparkle:edSignature` will be used (if available). This is to give us as much compatibility with old versions of `NetSparkle` as well as the macOS `Sparkle` library.
* An entirely new app cast generator tool is now available for use.
* By default, the app cast generator tool now uses Ed25519 signatures. If you don't want to use files on disk to store your keys, set the `SPARKLE_PRIVATE_KEY` and `SPARKLE_PUBLIC_KEY` environment variables before running the app cast generator tool. You can also store these signatures in a custom location with the `--key-path` flag.
  * You can still use DSA signatures via the DSAHelper tool and the `DSAChecker` class. This is not recommended.
  * `Ed25519Checker` is the class responsible for handling Ed25519 signatures. `DSAChecker` sticks around for verifying DSA signatures if they're still used.
* Removed `AssemblyAccessor` class in lieu of `IAssemblyAccessor` implementors
* The server file name for each app cast download is now checked before doing any downloads or showing available updates to the client. To disable this behavior and use the name in the app cast, set `SparkleUpdater.CheckServerFileName` to `false`.
* `bool ignoreSkippedVersions = false` has been added to `CheckForUpdatesAtUserRequest`, `CheckForUpdatesQuietly`, and `GetUpdateStatus` to make ignoring skipped versions easier.
* The file name/path used by `RelaunchAfterUpdate` are controlled by `RestartExecutableName` and `RestartExecutablePath`, respectively. `SparkleUpdater` makes a best effort to figure these out for you; however, you can override them if you need to. NOTE: The way these parameters are fetched has CHANGED in recent previews (as of 2021-04-18) -- YOU HAVE BEEN WARNED!!
* **Breaking change**: `CheckForUpdatesQuietly` now shows no UI ever. It could show a UI before, which didn't make a lot of sense based on the function name. Make sure that if you use this function that you handle showing a UI yourself if necessary. (See the HandleEventsYourself sample if you want help.) You can always trigger the built-in `SparkleUpdater` by calling `_sparkle.ShowUpdateNeededUI(updateInfo.Updates)`. 
* **Breaking change**: DLL assembly names for .NET Framework WinForms UI dlls changed from `NetSparkle.UI.WinForms` to `NetSparkleUpdater.UI.WinForms`.
* **Breaking change**: DLL assembly names for Avalonia UI dlls changed from `NetSparkle.UI.Avalonia` to `NetSparkleUpdater.UI.Avalonia`.
* **We now rely on Portable.BouncyCastle** (BouncyCastle.Crypto.dll) for the ed25519 implementation. This means there is another DLL to reference when you use NetSparkle!
* **We now rely on System.Text.Json (netstandard2.0) OR Newtonsoft.Json (.NET Framework 4.5.2)** for the JSON items. This means there is another DLL to reference when you use NetSparkle, and it will change depending on if the `System.Text.Json` or `Newtonsoft.Json` item is used!
