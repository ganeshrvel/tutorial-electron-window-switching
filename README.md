# Prevent duplicate windows in an electron app invoked from renderer and main processes

- Author: [Ganesh Rathinavel](https://www.linkedin.com/in/ganeshrvel "Ganesh Rathinavel")
- License: [MIT](https://github.com/ganeshrvel/tutorial-electron-window-switching/blob/master/LICENSE "MIT")
- Website URL: [https://github.com/ganeshrvel/tutorial-electron-window-switching](https://github.com/ganeshrvel/tutorial-electron-window-switching/ "https://github.com/ganeshrvel/tutorial-electron-window-switching")
- Repo URL: [https://github.com/ganeshrvel/tutorial-electron-window-switching](https://github.com/ganeshrvel/tutorial-electron-window-switching/ "https://github.com/ganeshrvel/tutorial-electron-window-switching")
- Contacts: ganeshrvel@outlook.com

### The challenge
In an electron app, two or more operating system level processes run concurrently — the "main" and "renderer" processes.

Because these processes are isolated from each other, child window instance, has to be loaded separately for both - main and renderer processes. This phenomenon raises another issue— duplicate windows for the same URL origin

I have spent a significant amount of time researching on preventing the issue of duplication of windows for the same URL origin. This was originally implemented inside [OpenMTP - Advanced Android File Transfer Application for macOS](https://github.com/ganeshrvel/openmtp "OpenMTP - Advanced Android File Transfer Application for macOS").

###  Implementation:

Here, as an example, I will be creating a privacy policy window which can be invoked by both menu item and as well as an anchor tag.

- Install packages

```shell
$ npm install react-helmet

or

$ yarn add react-helmet
```

- Create a file *create-windows.js* and add the below code

```javascript
import { BrowserWindow, remote } from 'electron';
const PRIVACY_POLICY_PAGE_TITLE = `Privacy Policy`; // this will be used as an identifier to capture the same browser instance
let privacyPolicyWindow = null;

/**
 * Privacy Policy Window
 */

const undefinedOrNull = _var => {
  return typeof _var === "undefined" || _var === null;
};

const loadExistingWindow = (allWindows, title) => {
  if (!undefinedOrNull(allWindows)) {
    for (let i = 0; i < allWindows.length; i += 1) {
      const item = allWindows[i];
      if (item.getTitle().indexOf(title) !== -1) {
        item.focus();
        item.show();

        return item;
      }
    }
  }

  return null;
};

const createWindow = isRenderedPage => {
  const config = {
    width: 800,
    height: 600,
    minWidth: 600,
    minHeight: 400,
    show: false,
    resizable: true,
    title: `${PRIVACY_POLICY_PAGE_TITLE}`,
    minimizable: true,
    fullscreenable: true,
    webPreferences: {
      nodeIntegration: true
    }
  };

  // incoming call from a rendered page
  if (isRenderedPage) {
    const allWindows = remote.BrowserWindow.getAllWindows();

    return loadExistingWindow(allWindows, PRIVACY_POLICY_PAGE_TITLE)
      ? null
      : new remote.BrowserWindow(config);
  }

  // incoming call from the main process
  const allWindows = BrowserWindow.getAllWindows();

  return loadExistingWindow(allWindows, PRIVACY_POLICY_PAGE_TITLE)
    ? null
    : new BrowserWindow(config);
};

export const privacyPolicyCreateWindow = (isRenderedPage = false) => {
  try {
    if (privacyPolicyWindow) {
      privacyPolicyWindow.focus();
      privacyPolicyWindow.show();
      return privacyPolicyWindow;
    }

    // show the existing privacyPolicyWindow
    const _privacyPolicyWindowTemp = createWindow(isRenderedPage);
    if (!_privacyPolicyWindowTemp) {
      return privacyPolicyWindow;
    }

    privacyPolicyWindow = _privacyPolicyWindowTemp;
    privacyPolicyWindow.loadURL(
      `file://path/to/my-app/app/index.html#privacyPolicyPage`
    ); // @todo: change the path accordingly
	
    privacyPolicyWindow.webContents.on("did-finish-load", () => {
      privacyPolicyWindow.show();
      privacyPolicyWindow.focus();
    });

    privacyPolicyWindow.onerror = error => {
      console.error(error, `createWindows -> privacyPolicyWindow -> onerror`);
    };

    privacyPolicyWindow.on("closed", () => {
      privacyPolicyWindow = null;
    });

    return privacyPolicyWindow;
  } catch (e) {
    console.error(e, `createWindows -> privacyPolicyWindow`);
  }
};
```

- Edit your *menu.js* file

```javascript
import { privacyPolicyWindow } from './create-windows';

// Add to your menu
submenu: [
	{
		label: 'Privacy Policy from main process',
		click: () => {
			privacyPolicyWindow(false);
		}
	}
]
```

- The *PrivacyPolicyPage/index.jsx*

```javascript
import React, { Component } from 'react';
import { Helmet } from 'react-helmet';

const PRIVACY_POLICY_PAGE_TITLE = `Privacy Policy`;

class PrivacyPolicyPage extends Component {
  render() {
    return (
      <div>
        <Helmet titleTemplate={`%s`}>
          <title>{PRIVACY_POLICY_PAGE_TITLE}</title>
        </Helmet>
		<div>
			<p>Window body</p>
		</div>
      </div>
    );
  }
}

export default PrivacyPolicyPage;
```

- Edit your router file

```javascript
import PrivacyPolicyPage from './PrivacyPolicyPage';

//Add the route

<Switch>
	<Route
		key={PrivacyPolicyPage}
		path='/privacyPolicyPage'
		exact
		component={PrivacyPolicyPage}
		/>
</Switch>
```

- Add an anchor tag inside your home page

```javascript
import { privacyPolicyWindow } from './create-windows';

<a
	className={styles.a}
	onClick={() => {
	privacyPolicyWindow(true);
	}}
	>
	Privacy Policy from the renderer process
</a>
```

### Conclusion

- call **privacyPolicyCreateWindow(true)** from renderer process and call **privacyPolicyCreateWindow(false)** from the main process.
- *privacyPolicyCreateWindow* method will look up for the existing windows which match the title string and it returns the pre-existing window if it finds any.


### Clone
```shell
$ git clone --depth 1 --single-branch --branch master https://github.com/ganeshrvel/tutorial-electron-window-switching.git

$ cd tutorial-electron-window-switching
```

### Contribute
- Fork the repo and create your branch from master.
- Ensure that the changes pass linting.
- Update the documentation if needed.
- Make sure your code lints.
- Issue a pull request!

When you submit code changes, your submissions are understood to be under the same [MIT License](https://github.com/ganeshrvel/tutorial-electron-window-switching/blob/master/LICENSE "MIT License") that covers the project. Feel free to contact the maintainers if that's a concern.


### Buy me a coffee
Help me keep the app FREE and open for all.
Paypal me: [paypal.me/ganeshrvel](https://paypal.me/ganeshrvel "paypal.me/ganeshrvel")

### Contacts
Please feel free to contact me at ganeshrvel@outlook.com

### More repos
- [OpenMTP  - Advanced Android File Transfer Application for macOS](https://github.com/ganeshrvel/openmtp "OpenMTP  - Advanced Android File Transfer Application for macOS")
- [Tutorial Series by Ganesh Rathinavel](https://github.com/ganeshrvel/tutorial-series-ganesh-rathinavel "Tutorial Series by Ganesh Rathinavel")
- [npm: electron-root-path](https://github.com/ganeshrvel/npm-electron-root-path "Get the root path of an Electron Application")
- [Electron React Redux Advanced Boilerplate](https://github.com/ganeshrvel/electron-react-redux-advanced-boilerplate "Electron React Redux Advanced Boilerplate")

### License
tutorial-electron-window-switching is released under [MIT License](https://github.com/ganeshrvel/tutorial-electron-window-switching/blob/master/LICENSE "MIT License").

Copyright © 2018-Present Ganesh Rathinavel
