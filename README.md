<h2 align="center">Expo Custom Updater</h2>

## Intro

Hi! this is a small project done to help myself on new Expo projects to better handle OTA updates done via Expo Publish.  

Expo OTA updates is a great system that automatically checks for new version on application startup if you set updates.checkAutomatically in app.json (i.e. to 30000 for 30 seconds).
  
The problem is that the users will only see your code (and your fixes) on the NEXT application startup.  
This makes it hard ot predict when the new code will be actually available in the app, expecially with some users that never close their applications.  
  
Writing a manual update routine can be difficult as a bug in this process may cause your app to be stuck in an update cycle (speaking from experience 🤣) 

This library have two goals:
* Force an update on every app startup
* Force an update when the user go back to the application after some time

In this way your users will always run the up-to-date code published on Expo, either when opening the app for the fist time or when coming back to it!

## Install

* `npm install expo-custom-updater` or
* `yarn add expo-custom-updater`

## Just force an update on every app startup

```JavaScript
import ExpoCustomUpdater from 'expo-custom-updater'

const customUpdater = new ExpoCustomUpdater()

customUpdater.doUpdateIfAvailable()

```

doUpdateIfAvailable is an asyncronus Promise so you can await for it or better place it in the Expo loadResourcesAsync

```JavaScript
async function loadResourcesAsync() {
  await Promise.all([
    customUpdater.doUpdateIfAvailable(),
    Asset.loadAsync([
      require('....'),
      require('...'),
    ]),
    ....
```

## Full setup with App State Change listener 

This allows to check for updates when user returns into the app after some time.
You can set the following options in the class constructor:

* minRefreshSeconds Do not check for updates before minRefreshSeconds from the last check (default 300).  
A check for new version will not be done if user switch back to the app within 5 minutes  
* showDebugInConsole Show what the library is doing in the console (default false)
* beforeCheckCallback Callback function before the check, useful to show a loading / spinner screen
* beforeDownloadCallback Callback function before fetching a new update, useful to show a loading screen with a message about a new version
* afterCheckCallback Callback function after the check, useful to hide a loading screen if no updates are available (if an update is found the application is restarted).

You can then enable the listener for AppState changes that will trigger an update check when user comes back to the App by running
customUpdater.registerAppStateChangeListener()

Before and when the check is completed the two callbacks will be run, useful to show a loading screen 


```JavaScript
export default class App extends React.Component {
  constructor (props) {
    super(props)
    this.state = { showLoadingScreen: true }
    this.customUpdater = new ExpoCustomUpdater({
      minRefreshSeconds: 600,
      showDebugInConsole: true,
      beforeCheckCallback: () => this.setState({ showLoadingScreen: true }),
      beforeDownloadCallback: () => this.setState({updateMessage: 'A new version of the app is being downloaded'}),
      afterCheckCallback: () => this.setState({ showLoadingScreen: false })
    })
  }
  componentDidMount () {
    this.customUpdater.registerAppStateChangeListener()
  }

  componentWillUnmount () {
    this.customUpdater.removeAppStateChangeListener()
  }

```

 ## Notes:
* You can read activity logs from customUpdater.updateLog (array of strings), useful for debugging
* Expo does not support OTA updates from development or within the Expo App, so check for updates is skipped in __DEV__ mode.
* To test your application update method properly it is useful to compile an APK and install it to a connected device with "adb install xxx.apk", then you can play with expo publish to verify the setup 

Have fun!