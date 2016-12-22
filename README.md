# electron-distribution-guide

Guide to distribute Electron apps made simple

Moved to [`electron-osx-sign-guide`](https://github.com/sethlu/electron-osx-sign-guide)

## About

It is worth noting that most of the steps here taken have made unnecessary with packaging tools and signing tools. The documents here attached are targeted as specifications of stepwise procedures that eventually lead to distribution of products in a **conceptual** flow. Also, the way of categorizing target platforms here is slightly different from the current Electron documents; this is so that we may structure the steps slightly more logically. The packaging of Electron app is out of the scope of this document.

In the current phase of working/drafting, only Electron platforms `darwin` and `mas` are to be covered; however, your contributions are very welcome to our project. Additional thanks go to @jasonhinkle and @zcbenz for making a leap start. The guides here generally follows what described in the Electron documents.

## OS X

### Inside the Mac App Store

To distribute apps inside the Mac App Store, Electron platform `mas` must be used. Please avoid platform `darwin` here as protected calls in Electron forbid the app from being accepted.

The workflow here is branched into two parts:

- [Development-signed Application]
- [Mac App Store Deployment]

**Development-signed Application** is chosen when testing a product on registered devices; **Mac App Store Deployment**, when submitting a product for review by the iTunes Connect. Please navigate into each to know more about the manual distribution process.

> NB: There is a difference between the two and only one could be taken at a time. Though the actual app being the same, making for development and for distribution are different. It is more easily appreciated that during iOS app development only registered devices could run test apps installed; this mechanism now acts very much the same on the development for the Mac App Store.

#### Outdated support

*For Electron version < 1.1.1*

As solution made in [electron#5601](https://github.com/electron/electron/pull/5601) has not yet been announced in earlier versions, some of what mentioned above could be discarded. Please follow the guide below to distribute your app:

- [Mac App Store Deployment (outdated)]

### Outside the Mac App Store

To distribute apps outside the Mac App Store, Electron platform `darwin` should be used. However, platform `mas` should not emit warnings if used to be distributed outside the Mac App Store; only less feature than in the full-fledged platform `darwin` is available.

- [Developer ID-signed Application]

> NB: The changes introduced in Electron version 1.1.1 does not affect distribution outside the Mac App Store. Please feel free to follow the guide above.

[Mac General Prerequisites]: MacGeneralPrerequisites.md
[Development-signed Application]: DevelopmentSignedApplication.md
[Mac App Store Deployment]: MacAppStoreDeployment.md
[Mac App Store Deployment (outdated)]: MacAppStoreDeploymentOutdated.md
[Developer ID-signed Application]: DeveloperIDSignedApplication.md
