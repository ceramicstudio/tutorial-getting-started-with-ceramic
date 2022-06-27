![image](https://img.shields.io/badge/Publish%20Date-June%202022-blue?style=for-the-badge)
![image](https://img.shields.io/badge/Ceramic%20Version-V2.0.0-orange?style=for-the-badge)
# Getting Started With Ceramic

Welcome to Web 3!  

In this beginner friendly guide, I'll give you all the tools and knowledge needed to integrate the [Ceramic Network] into your Web 3 [dapps].  

The Ceramic Network is a decentralized data network that aims to bring composable data to Web 3 dapps.  There are many types of data that Ceramic can work with, but for this guide we can treat Ceramic like a decentralized NOSQL document database.

This guide is meant for you to follow along, so expect diagrams and code examples as you continue reading.



## Your Learning Experience

Along with this written guide, I have provided a [GitHub] repository containing all code I will reference.

If you prefer a video guide, rather than a written guide, you can watch a video walkthrough on the [Ceramic Youtube Channel].

Before you get started, it is implied that you have the general web development skills outlined below:


**Skills used in this guide**
- [Basic JavaScript]
- Understand fundamental differences between [client-side JS vs Server-side JS]
- JavaScript package management
- Basic understanding of [Webpack]

**Optional skills**
- [Git]
- Version control (i.e. GitHub, GitLab, BitBucket)


**Required tooling (needs to be installed before continuing)**
- Text Editor (i.e. VS Code, Sublime, vim)
- [NodeJS] v16 or higher
- [NPM] v8 or higher or [Yarn]



## Things We Need To Talk About

Before getting started, I will cover some key terms that will be used throughout this guide.


**[Decentralized Identifier]**

Often referred to as a [DID].

A [DID] is a unique identifier that contains metadata about you.  Things like your [public key], some verification information, what
service points you're allowed to access and a couple other things.

Simply put, [DID]s are *used* as Ceramic account identifiers.

>The dependency packages used for this are:
>- `dids`

**[DID Resolver]**

DID resolvers take a DID as input and return a [DID Document].

This resolution process turn a DID from something generic into a document that accurately describes an identity and what methods and capabilities that identity is allow to perform.

Simply put, a resovler hydrates a DID with *what* actions it is capable of performing.

>The dependency packages used for this are:
>- `key-did-resolver`
>- `@glazed/did-datastore`

**[Ethereum Providers]**

If you want your application to have access to a blockchain, you need to use a provider.

This guide connects to the Ethereum Blockchain, and therefore uses an Ethereum provider.

Providers are used in place of running blockchain nodes by yourself. Providers have two main tasks:

1. Tell your application what blockchain to connect to.
1. Once you are connected, run queries and send signed transactions that modify the blockchain's state.

Metamask is one of the most popular blockchain providers and it is the provider will use to connect our application to the Ethereum Blockchain.

Simply put, providers *authenticate* users to perform actions on the blockchain.

>The dependency packages used for this are:
>- `key-did-provider-ed25519`
>- `@glazed/did-session`
>- `@ceramicnetwork/blockchain-utils-linking`

**[Data StreamTypes]**

When I speak of data streams, I'm not talking about [streaming data] from a consumption point of view.  Streams are what Ceramic calls its data structures.  Feel free to read more about [streams].

A [StreamType] is just one of the possible data structures for a stream.  In this guide we will be working **indirectly** with the `TileDocument` [StreamType], which you can think of like a [JSON Object].  These StreamTypes are what handles everyting related to the data and they run on [Ceramic nodes].

Simply put, StreamTypes define the *data structure* and *how* that data's state is allowed to change.

>The dependency packages used for this are:
>- `@glazed/did-datastore`

**[Data models]**

Data models are typically used to represent an application feature.  Like Notes, user profiles, blog posts and even a social graph.

Data models are the heart of composable data.  It is common for a single application to use multiple data models, and it is common for a single data model to be used across multiple applications!

Composability done this way also makes the developer experience better. Building an application on Ceramic looks like browsing a marketplace of data models, plugging them into your app, and automatically gaining access to all data on the network that’s stored in these models.

Simply put, Data models are what enable *data composability* in an application.

## About the Application


You will be building a simple web application that performs a simple read and write operation to data on the Ceramic Network.  For this application to work properly it will need to complete the following steps in the order they are listed.

1. Use an Ethereum Provider to authenticate to the blockchain.
1. Once authenticated, resolve a DID to be used with Ceramic.
1. Use a Ceramic instance, with the supplied DID, to read and write to a `TileDocument` stream.

I mentioned a few of the dependencies above in the [Things We Need To Talk About] section, but there are a few other dependencies that are worth mentioning before you go any further.

**[Ceramic Client]**

This is the web client that allows your application to connect to [Ceramic nodes] that are a part of the network.

>The dependency packages used for this are:
>- `@ceramicnetwork/http-client`

**[Webpack]**

The JavaScript you will be writing uses Node packages, making it server-side code.  However, web browsers require client-side code.

Webpack is a nice module that will convert the server-side JavaScript that you will be writing into client-side JavaScript that your browser can understand.  

To accomplish this, we need a few dependencies.

>The dependency packages used for this are:
>- `webpack`
>- `webpack-cli`
>- `buffer`

## Building the Frontend

I will walk you through the main steps of building the frontend of this application using simple HTML and CSS

1. Let's start by creating a new directory for this project.  This process will vary based on your operating system, so choose the solution that fits your environment best.

    **Windows**

    ```
    md getting-started-with-ceramic
    ```

    **MacOS/Linux**

    ```
    mkdir getting-started-with-ceramic
    ```

1. Now, create a file named `index.html` in the root of that directory.  The `index.html` file should contain the following content:

    ```html
    <!DOCTYPE html>
    <html lang="en">

    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <link rel="stylesheet" href="style.css">
        <link rel="shortcut icon" href="/favicon.ico">

        <title>Getting Started</title>
    </head>

    <body>
        <!-- create header with connect button -->
        <header class="SiteHeader">
            <div class="HeaderContainer">
                <h1 id="pageTitle">Getting Start With Ceramic</h1>
            </div>
            <div class="HeaderContainer">
                <button id="walletBtn"></button>
            </div>

        </header>
        <div class="MainCont">
            <div class="DataBlocks">
                <div class="DataBlock">
                    <div id="basicProfile">
                        <div class="BodyContainer">
                            <h2>Basic Profile</h2>
                            <p>Read from Ceramic Datamodel</p>
                            <br>
                            <p class="ProfileData" id="profileName"></p>
                            <p class="ProfileData" id="profileGender"></p>
                            <p class="ProfileData" id="profileCountry"></p>
                        </div>
                    </div>
                </div>
            </div>
            <div class="ProfileForm">
                <div class="BodyContainer">
                    <h2>Update Basic Profile on Ceramic</h2>
                    <br>
                    <form id="profileForm">
                        <div class="formfield">
                            <label class="formLabel" for="name">Name:</label>
                            <input class="forminput" type="text" id="name" placeholder="John Doe">
                        </div>
                        <div class="formfield">
                            <label class="formLabel" for="country">Country:</label>
                            <input class="forminput" type="text" id="country" placeholder="USA">
                        </div>
                        <div class="formfield">
                            <label class="formLabel" for="gender">Gender:</label>
                            <select class="forminput" id="gender">
                                <option value="female">Female</option>
                                <option value="male">Male</option>
                                <option value="non-binary">Non-Binary</option>
                                <option value="other">Other</option>
                            </select>
                        </div>
                        <div class="formfield">
                            <input class="forminput" type="submit" id="submitBtn" value="Submit">
                        </div>
                    </form>
                </div>
            </div>
        </div>

        <!-- <button id="setBasicProf">Set Profile</button>
        <button id="getBasicProf">Get Profile</button> -->
        <script src="dist/bundle.js" type="module"></script>
    </body>

    </html>
    ```

1. Next, create a file named `style.css` in the root of the `getting-started-with-ceramic` directory.  This file should contain the following content:
    ```css
    * {
        margin: 0;
        padding: 0;
    }

    .SiteHeader {
        display: flex;
        justify-content: space-between;
        padding: 10px;
        background-color: orange;
    }

    .HeaderContainer {
        display: flex;
        align-items: center;
    }


    .MainCont {
        display: flex;
        justify-content: space-around;
        padding: 10px;
    }

    .DataBlock {

        margin-bottom: 10px;

    }

    .BodyContainer {
        background-color: lightsalmon;
        border: 1px solid black;
        border-radius: 30px;
        padding: 20px;
        min-width: 250px;
    }

    .ProfileForm {
        min-width: 400px;
    }



    .formfield {
        display: flex;
        justify-content: space-between;
        margin-bottom: 10px;
    }

    .forminput {
        min-width: 150px;

    }

    #submitBtn {
        display: block;
        margin: auto;
        width: auto;
    }

    .ProfileData {
        font-weight: bold;
    }
    ```

Great!  Now if you open up the `index.html` file in your browser, or using a utility like [LiveShare], you should see something like this:

![](https://i.imgur.com/ZH2egjl.png)



## Adding JavaScript and Ceramic

Right now your application is not capable of doing anything.  There is no logic built into it, it's just a static page with some content and some style.

In this step I will show you how to use providers, resolvers and Ceramic to transform this application from a static site to a web 3 dapp!

1. To start, initialize a new [NodeJS] project using either [NPM] or [Yarn]:

    **NPM**

    ```
    npm init -y
    ```

    **Yarn**

    ```
    yarn init -y
    ```
1. Next, install the above mentioned dependencies:
    
    **NPM**

    Dev dependencies
    
    ```
    npm install -D buffer dids key-did-provider-ed25519 key-did-resolver webpack webpack-cli
    ```

    Regular dependencies

    ```
    npm install @ceramicnetwork/blockchain-utils-linking @ceramicnetwork/http-client @glazed/did-datastore @glazed/did-session
    ```

    **Yarn**

    Dev dependencies 

    ```
    yarn add -D buffer dids key-did-provider-ed25519 key-did-resolver webpack webpack-cli
    ```

    Regular dependencies

    ```
    yarn add @ceramicnetwork/blockchain-utils-linking @ceramicnetwork/http-client @glazed/did-datastore @glazed/did-session
    ```

1. Now, create a file named `main.js` in the root of the `getting-started-with-ceramic` directory.

1. Start by importing the regular depndencies into this file:
    ```javascript
    //main.js 

    import { CeramicClient } from '@ceramicnetwork/http-client'
    import { EthereumAuthProvider } from '@ceramicnetwork/blockchain-utils-linking'
    import { DIDDataStore } from '@glazed/did-datastore'
    import { DIDSession } from '@glazed/did-session'
    ```
    >Did you notice that some packages come from `@ceramicnetwork` and others come from `@glazed`? 
    > 
    >Packages that come from @ceramicnetwork are part of the core Ceramic protocol.  They help connect applications to Ceramic nodes.
    >
    >Packges that come from @glazed are not part of the core Ceramic protocol, they are referred to as `middleware` and provide developers with some added functionality and convenience.
1. Following the dependency imports you should setup a series of DOM Element selectors.  This not only makes our code easier to read as it is written, but in larger applications this technique can add performance benefits.  Add the following to `main.js`
    ```javascript
    import { CeramicClient } from '@ceramicnetwork/http-client'
    import { EthereumAuthProvider } from '@ceramicnetwork/blockchain-utils-linking'
    import { DIDDataStore } from '@glazed/did-datastore'
    import { DIDSession } from '@glazed/did-session'

    const profileForm = document.getElementById('profileForm')
    const walletBtn = document.getElementById('walletBtn')
    const profileName = document.getElementById('profileName')
    const profileGender = document.getElementById('profileGender')
    const profileCountry = document.getElementById('profileCountry')
    const submitBtn = document.getElementById('submitBtn')
    ```
1. Using the `CeramiClient` that was just imported, create a new Ceramic Client instance by adding the following code to the `main.js` file:
    ```javascript
    //main.js

    import { CeramicClient } from '@ceramicnetwork/http-client'
    import { EthereumAuthProvider } from '@ceramicnetwork/blockchain-utils-linking'
    import { DIDDataStore } from '@glazed/did-datastore'
    import { DIDSession } from '@glazed/did-session'
    
    const profileForm = document.getElementById('profileForm')
    const walletBtn = document.getElementById('walletBtn')
    const profileName = document.getElementById('profileName')
    const profileGender = document.getElementById('profileGender')
    const profileCountry = document.getElementById('profileCountry')
    const submitBtn = document.getElementById('submitBtn')

    const ceramic = new CeramicClient("https://ceramic-clay.3boxlabs.com")
    ```
    >There are currently 4 possible networks to connect the Ceramic HTTP client to.  You can click each link to learn more about the networks.
    >- [Mainnet]
    >- [Clay Testnet] (recommended and currently being used by our application)
    >- [Dev Unstable]
    >- [Local]

1. Next, create a variable named `aliases` that will hold the reference information for the `BasicProfile` data model:
    ```javascript
    //main.js

    import { CeramicClient } from '@ceramicnetwork/http-client'
    import { EthereumAuthProvider } from '@ceramicnetwork/blockchain-utils-linking'
    import { DIDDataStore } from '@glazed/did-datastore'
    import { DIDSession } from '@glazed/did-session'
    
    const profileForm = document.getElementById('profileForm')
    const walletBtn = document.getElementById('walletBtn')
    const profileName = document.getElementById('profileName')
    const profileGender = document.getElementById('profileGender')
    const profileCountry = document.getElementById('profileCountry')
    const submitBtn = document.getElementById('submitBtn')

    const ceramic = new CeramicClient("https://ceramic-clay.3boxlabs.com")

    const aliases = {
        schemas: {
            basicProfile: 'ceramic://k3y52l7qbv1frxt706gqfzmq6cbqdkptzk8uudaryhlkf6ly9vx21hqu4r6k1jqio',

        },
        definitions: {
            BasicProfile: 'kjzl6cwe1jw145cjbeko9kil8g9bxszjhyde21ob8epxuxkaon1izyqsu8wgcic',
        },
        tiles: {},
    }
    ```
    >**Parts of a datamodel**
    >
    >`schemas`: Define the JSON schema for the data model.
    >
    >`definitions`: Link a user-friendly model name and description to a specific schema.
    >
    >`tiles`: Individual data records based on parameters set within the schema.
    >
    >![](https://i.imgur.com/WDKidX1.png)
1. The `DIDDataStore` allows the applicaiton to write and read data from Ceramic.  The `DIDDataStore` is based on Data models.  Add the following code to `main.js` to configure the `DIDDataStore` to use the `aliases` and `ceramic instance` defined above:
    ```javascript
    //main.js

    import { CeramicClient } from '@ceramicnetwork/http-client'
    import { EthereumAuthProvider } from '@ceramicnetwork/blockchain-utils-linking'
    import { DIDDataStore } from '@glazed/did-datastore'
    import { DIDSession } from '@glazed/did-session'
    
    const profileForm = document.getElementById('profileForm')
    const walletBtn = document.getElementById('walletBtn')
    const profileName = document.getElementById('profileName')
    const profileGender = document.getElementById('profileGender')
    const profileCountry = document.getElementById('profileCountry')
    const submitBtn = document.getElementById('submitBtn')

    const ceramic = new CeramicClient("https://ceramic-clay.3boxlabs.com")

    const aliases = {
        schemas: {
            basicProfile: 'ceramic://k3y52l7qbv1frxt706gqfzmq6cbqdkptzk8uudaryhlkf6ly9vx21hqu4r6k1jqio',

        },
        definitions: {
            BasicProfile: 'kjzl6cwe1jw145cjbeko9kil8g9bxszjhyde21ob8epxuxkaon1izyqsu8wgcic',
        },
        tiles: {},
    }

    const datastore = new DIDDataStore({ ceramic, model: aliases })
    ```
    >If your dapp required it, you could add more Data models to the `aliases` variable by adding the `schema`, `definition` and `tiles` necessary!

Great job!  You now have the basic foundation needed to get this application up and running.  All the configuration for the Ceramic Client and the Data models is complete.

## Authenticating With the Blockchain

This next section will guide you through using an Ethereum Provider, [Metamask], to authenticate a user with the Ethereum blockchain.

The authentication flow being used is called [Sign-In With Ethereum], but I will refer to it as SIWE from here on out.  

>Check out this awesome article to learn more: [Why Sign-In With Ethereum Is A Game Changer].

Let's add SIWE to this application!

1. This application needs an [async function], I will name it `authenticateWithEthereum`, that uses the Provider then uses the Resovler and finally assigns the DID to the Ceramic Client you created earlier.  Add this code to `main.js` to accomplish these tasks:

    ```javascript
    //main.js

    async function authenticateWithEthereum(ethereumProvider) {

        const accounts = await ethereumProvider.request({
        method: 'eth_requestAccounts',
        })

        const authProvider = new EthereumAuthProvider(ethereumProvider, accounts[0])

        const session = new DIDSession({ authProvider })

        const did = await session.authorize()

        ceramic.did = did
    }
    ```

    >The `DIDSession` is what it handling the SIWE authentication flow for you in this code snippet.

1. There are usually some logic checks that our application needs to do before the authentication flow can be started.  When developing dapps a common check is to make sure that a provider is available.  In this case, [Metamask] adds itself as the provider in our browsers `window` object.  It is referencable by `window.ethereum`.  If the end-user of the application has not installed [Metamask], or another provider, then our application will not be able to connect to the blockchain.  Let's take this knowledge and apply it to a new [async function] called `auth`.  Add the code below to `main.js`:
    ```javascript
    //main.js

    async function authenticateWithEthereum(ethereumProvider) {

        const accounts = await ethereumProvider.request({
        method: 'eth_requestAccounts',
        })

        const authProvider = new EthereumAuthProvider(ethereumProvider, accounts[0])

        const session = new DIDSession({ authProvider })

        const did = await session.authorize()

        ceramic.did = did
    }
    
    // newly added auth function here:
    async function auth() {
    if (window.ethereum == null) {
        throw new Error('No injected Ethereum provider found')
    }
    await authenticateWithEthereum(window.ethereum)
    }
    ```
    `auth()` first checks to see if `window.ethereum` exists before trying to call `authenticateWithEthereum()`.  This prevents the application from hanging if the user does not have an injected provider!

If you'd like to check your work, the complete `main.js` file should currently look like this:

```javascript
    //main.js

    // import all dependencies:
    import { CeramicClient } from '@ceramicnetwork/http-client'
    import { EthereumAuthProvider } from '@ceramicnetwork/blockchain-utils-linking'
    import { DIDDataStore } from '@glazed/did-datastore'
    import { DIDSession } from '@glazed/did-session'
    
    //cache a reference to the DOM Elements
    const profileForm = document.getElementById('profileForm')
    const walletBtn = document.getElementById('walletBtn')
    const profileName = document.getElementById('profileName')
    const profileGender = document.getElementById('profileGender')
    const profileCountry = document.getElementById('profileCountry')
    const submitBtn = document.getElementById('submitBtn')

    // create a new CeramicClient instance:
    const ceramic = new CeramicClient("https://ceramic-clay.3boxlabs.com")

    // reference the data models this application will use:
    const aliases = {
        schemas: {
            basicProfile: 'ceramic://k3y52l7qbv1frxt706gqfzmq6cbqdkptzk8uudaryhlkf6ly9vx21hqu4r6k1jqio',

        },
        definitions: {
            BasicProfile: 'kjzl6cwe1jw145cjbeko9kil8g9bxszjhyde21ob8epxuxkaon1izyqsu8wgcic',
        },
        tiles: {},
    }

    // configure the datastore to use the ceramic instance and data models referenced above:
    const datastore = new DIDDataStore({ ceramic, model: aliases })

    // this function authenticates the user using SIWE
    async function authenticateWithEthereum(ethereumProvider) {

        const accounts = await ethereumProvider.request({
        method: 'eth_requestAccounts',
        })

        const authProvider = new EthereumAuthProvider(ethereumProvider, accounts[0])

        const session = new DIDSession({ authProvider })

        const did = await session.authorize()

        ceramic.did = did
    }
    
    // check for a provider, then authenticate if the user has one injected:
    async function auth() {
        if (window.ethereum == null) {
        throw new Error('No injected Ethereum provider found')
        }
        await authenticateWithEthereum(window.ethereum)
    }    
```
----
## Reading Data Using Ceramic

The next function you write will use the `DIDDatastore` to retrieve data from the Ceramic network.  I will call it `getProfileFromCeramic` and it will be an [async function] like the ones before it.

This function will be declared in the `main.js` file.

1. Add the `getProfileFromCeramic` function to `main.js`:
    ```javascript
    //main.js

    async function getProfileFromCeramic() {
    try {
        
        //use the DIDDatastore to get profile data from Ceramic
        const profile = await datastore.get('BasicProfile')
        
        //render profile data to the DOM (not written yet)
        renderProfileData(profile)
    } catch (error) {
        console.error(error)
        }
    }
    ```
    As you can see, by calling `datastore.get()` method, you can simply reference the `definition` of the data model you wish to read data from.  

    The DIDDatastore uses the DID assinged to the Ceramic client to make this call.  It returns the profile object white get's stored in the `profile` variable.
1. You will need to create the `renderProfileData` function to extract this profile data and show it in the browser window.  Since this is NOT a guide on web development I will not go into detail about what this function is doing.  Add the following to your `main.js` file:
    ```javascript
    function renderProfileData(data) {
        if (!data) return
        data.name ? profileName.innerHTML = "Name:     " + data.name : profileName.innerHTML = "Name:     "
        data.gender ? profileGender.innerHTML = "Gender:     " + data.gender : profileGender.innerHTML = "Gender:     "
        data.country ? profileCountry.innerHTML = "Country:     " + data.country : profileCountry.innerHTML = "Country:     "
    }
    ```
    >I would like to point out that `data` is the `profile` object that is returned from the `datastore.get()` call.  The properties of data are defined in the `BasicProfile` data model.  Check out the data model in the [Ceramic datamodels repo] for a [full list of properties].

That's it!  That's all there is to reading data from the Ceramic Network using the `DIDDataStore`!

A complete version of what `main.js` should look like at this point can be found below:
```javascript
    //main.js

    // import all dependencies:
    import { CeramicClient } from '@ceramicnetwork/http-client'
    import { EthereumAuthProvider } from '@ceramicnetwork/blockchain-utils-linking'
    import { DIDDataStore } from '@glazed/did-datastore'
    import { DIDSession } from '@glazed/did-session'
    
    //cache a reference to the DOM Elements
    const profileForm = document.getElementById('profileForm')
    const walletBtn = document.getElementById('walletBtn')
    const profileName = document.getElementById('profileName')
    const profileGender = document.getElementById('profileGender')
    const profileCountry = document.getElementById('profileCountry')
    const submitBtn = document.getElementById('submitBtn')

    // create a new CeramicClient instance:
    const ceramic = new CeramicClient("https://ceramic-clay.3boxlabs.com")

    // reference the data models this application will use:
    const aliases = {
        schemas: {
            basicProfile: 'ceramic://k3y52l7qbv1frxt706gqfzmq6cbqdkptzk8uudaryhlkf6ly9vx21hqu4r6k1jqio',

        },
        definitions: {
            BasicProfile: 'kjzl6cwe1jw145cjbeko9kil8g9bxszjhyde21ob8epxuxkaon1izyqsu8wgcic',
        },
        tiles: {},
    }

    // configure the datastore to use the ceramic instance and data models referenced above:
    const datastore = new DIDDataStore({ ceramic, model: aliases })

    // this function authenticates the user using SIWE
    async function authenticateWithEthereum(ethereumProvider) {

        const accounts = await ethereumProvider.request({
        method: 'eth_requestAccounts',
        })

        const authProvider = new EthereumAuthProvider(ethereumProvider, accounts[0])

        const session = new DIDSession({ authProvider })

        const did = await session.authorize()

        ceramic.did = did
    }
    
    // check for a provider, then authenticate if the user has one injected:
    async function auth() {
        if (window.ethereum == null) {
        throw new Error('No injected Ethereum provider found')
        }
        await authenticateWithEthereum(window.ethereum)
    } 

    //retrieve BasicProfile data from ceramic using the DIDDatastore
    async function getProfileFromCeramic() {
        try {
        
        //use the DIDDatastore to get profile data from Ceramic
        const profile = await datastore.get('BasicProfile')
        
        //render profile data to the DOM (not written yet)
        renderProfileData(profile)
        } catch (error) {
        console.error(error)
        }
    }

    //Do some fun web dev stuff to present the BasicProfile in the DOM
    function renderProfileData(data) {
        if (!data) return
        data.name ? profileName.innerHTML = "Name:     " + data.name : profileName.innerHTML = "Name:     "
        data.gender ? profileGender.innerHTML = "Gender:     " + data.gender : profileGender.innerHTML = "Gender:     "
        data.country ? profileCountry.innerHTML = "Country:     " + data.country : profileCountry.innerHTML = "Country:     "
    }
```
## Writing Data Using Ceramic

The next piece to implement is writing data to the Ceramic Network using the `DIDDatastore`.

1. Like some of the other functions that have been written, the `updateProfileOnCeramic` function should by an [async function].  Add the following to `main.js`:
    ```javascript
    async function updateProfileOnCeramic() {
        try {
        const updatedProfile = getFormProfile()
        submitBtn.value = "Updating..."

        //use the DIDDatastore to merge profile data to Ceramic
        await datastore.merge('BasicProfile', updatedProfile)

        //use the DIDDatastore to get profile data from Ceramic
        const profile = await datastore.get('BasicProfile')

        renderProfileData(profile)

        submitBtn.value = "Submit"
        } catch (error) {
        console.error(error)
        }
    }
    ```
    >There are two important things to talk about before moving on.
    >
    >First: The `DIDDatastore` has two methods that allow writes to a data model:
    >- `merge()`, which only writes to fields that have changed
    >- `set()`, which overwrites ALL fields including those that have not changed.  This can lead to data be removed in an unwanted way.  It is recommended to use merge rather than set for that reason.
    >
    >Second: Reading data from the DIDDatastore simply to render it to the DOM using `renderProfileData()` in this scenario is sub-optimal.  There is no real need to read the data from Ceramic at this stage.  This was included to show you how simple reading and writing can be, as each take one line when using the DIDDatastore.

1. You probably noticed a call to `getFormProfile()` in the above code block.  That function does not currently exist.  Let's add it to `main.js` now.  Place the following code in `main.js`:
    ```javascript
    function getFormProfile() {

        const name = document.getElementById('name').value
        const country = document.getElementById('country').value
        const gender = document.getElementById('gender').value

        return {
            name,
            country,
            gender
        }
    }
    ```
    >If you are wondering how I came up with the object properties of `name`, `country` and `gender`, they are all found on the [BasicProfile] datamodel.  There are additional properties for a BasicProfile which are not referenced in this project.  You should explore the use of these properties in you own projects!

Wow!  You made it!  This is all you need to get started with Ceramic.  You now know enough to run out and create amazing dapps.

You're not quite done yet though.  There are some minor things that have to be built to get this application fully working.  


## Making The Buttons Work

This section as well as the next, [Configuring Webpack], are not necessarliy Ceramic related.  These sections cover some necessary taks that must take place to tie in the buttons for the applicaiton and to conver the server-side to into something the browser can understand.

**How buttons work**

The button elements for this application are going to use [Event Listeners] to execute functions when they are clicked.

All of the following code will need to be placed in `main.js`.

1. Let's start by creating a function that the event listener can call when a user clicks the "connect wallet" button.
    ```javascript
    async function connectWallet(authFunction, callback) {
        try {
        walletBtn.innerHTML = "Connecting..."
        await authFunction()
        await callback()
        walletBtn.innerHTML = "Wallet Connected"

        } catch (error) {
        console.error(error)
        }

    }
    ```
1. Currently the button element doesn't display any `innerHTML` so let's fix that before moving on.  Under the DOM caching that took place earlier in `main.js`, add the following line:
    ```javascript
    walletBtn.innerHTML = "Connect Wallet"
    ```
1. Another thing that is missing are the text placeholders where the profile data should be rendered.  You can set that placeholder text by adding this code under the `walletBtn.innerHTML` line.
    ```javascript
    walletBtn.innerHTML = "Connect Wallet"
    profileName.innerHTML = "Name: "
    profileGender.innerHTML = "Gender: "
    profileCountry.innerHTML = "Country: "
    ```
    
3. The last thing is adding two event listeners.  One will go on the "connect wallet" button and it will call the `connectWallet` function defined above.  The other will go on the button that is a part of the `profileForm` element.  Add these to lines to `main.js`:
    ```javascript
    walletBtn.addEventListener('click', async () => await connectWallet(auth, getProfileFromCeramic))

    profileForm.addEventListener('submit', async (e) => {
    e.preventDefault()
    await updateProfileOnCeramic()

    })
    ```

Alright!  That's all the JavaScript that the application needs!  Double check your work by referencing the full `main.js` file below:

```javascript
    //main.js

    // import all dependencies:
    import { CeramicClient } from '@ceramicnetwork/http-client'
    import { EthereumAuthProvider } from '@ceramicnetwork/blockchain-utils-linking'
    import { DIDDataStore } from '@glazed/did-datastore'
    import { DIDSession } from '@glazed/did-session'
    
    //cache a reference to the DOM Elements
    const profileForm = document.getElementById('profileForm')
    const walletBtn = document.getElementById('walletBtn')
    const profileName = document.getElementById('profileName')
    const profileGender = document.getElementById('profileGender')
    const profileCountry = document.getElementById('profileCountry')
    const submitBtn = document.getElementById('submitBtn')

    // give the wallet button an initial value to display
    walletBtn.innerHTML = "Connect Wallet"
    // setup placeholder text where profile should render
    walletBtn.innerHTML = "Connect Wallet"
    profileName.innerHTML = "Name: "
    profileGender.innerHTML = "Gender: "
    profileCountry.innerHTML = "Country: "

    // create a new CeramicClient instance:
    const ceramic = new CeramicClient("https://ceramic-clay.3boxlabs.com")

    // reference the data models this application will use:
    const aliases = {
        schemas: {
            basicProfile: 'ceramic://k3y52l7qbv1frxt706gqfzmq6cbqdkptzk8uudaryhlkf6ly9vx21hqu4r6k1jqio',

        },
        definitions: {
            BasicProfile: 'kjzl6cwe1jw145cjbeko9kil8g9bxszjhyde21ob8epxuxkaon1izyqsu8wgcic',
        },
        tiles: {},
    }

    // configure the datastore to use the ceramic instance and data models referenced above:
    const datastore = new DIDDataStore({ ceramic, model: aliases })

    // this function authenticates the user using SIWE
    async function authenticateWithEthereum(ethereumProvider) {

        const accounts = await ethereumProvider.request({
        method: 'eth_requestAccounts',
        })

        const authProvider = new EthereumAuthProvider(ethereumProvider, accounts[0])

        const session = new DIDSession({ authProvider })

        const did = await session.authorize()

        ceramic.did = did
    }
    
    // check for a provider, then authenticate if the user has one injected:
    async function auth() {
        if (window.ethereum == null) {
        throw new Error('No injected Ethereum provider found')
        }
        await authenticateWithEthereum(window.ethereum)
    } 

    //retrieve BasicProfile data from ceramic using the DIDDatastore
    async function getProfileFromCeramic() {
        try {
        
        //use the DIDDatastore to get profile data from Ceramic
        const profile = await datastore.get('BasicProfile')
        
        //render profile data to the DOM (not written yet)
        renderProfileData(profile)
        } catch (error) {
        console.error(error)
        }
    }

    //Do some fun web dev stuff to present the BasicProfile in the DOM
    function renderProfileData(data) {
        if (!data) return
        data.name ? profileName.innerHTML = "Name:     " + data.name : profileName.innerHTML = "Name:     "
        data.gender ? profileGender.innerHTML = "Gender:     " + data.gender : profileGender.innerHTML = "Gender:     "
        data.country ? profileCountry.innerHTML = "Country:     " + data.country : profileCountry.innerHTML = "Country:     "
    }

    //this function uses the datastore to write data to the Ceramic Network as well as read data back before populating the changes in the DOM
    async function updateProfileOnCeramic() {
        try {
        const updatedProfile = getFormProfile()
        submitBtn.value = "Updating..."

        //use the DIDDatastore to merge profile data to Ceramic
        await datastore.merge('BasicProfile', updatedProfile)

        //use the DIDDatastore to get profile data from Ceramic
        const profile = await datastore.get('BasicProfile')

        renderProfileData(profile)

        submitBtn.value = "Submit"
        } catch (error) {
        console.error(error)
        }
    }

    // Parse the form and return the values so the BasicProfile can be updated
    function getFormProfile() {

        const name = document.getElementById('name').value
        const country = document.getElementById('country').value
        const gender = document.getElementById('gender').value

        // object needs to conform to the datamodel
        // name -> exists
        // hair-color -> DOES NOT EXIST
        return {
            name,
            country,
            gender
        }
    }


    //a simple utility funciton that will get called from the event listener attached to the connect wallet button
    async function connectWallet(authFunction, callback) {
        try {
        walletBtn.innerHTML = "Connecting..."
        await authFunction()
        await callback()
        walletBtn.innerHTML = "Wallet Connected"

        } catch (error) {
        console.error(error)
        }

    }

    //add both event listeners to that the buttons work when they are clicked
    walletBtn.addEventListener('click', async () => await connectWallet(auth, getProfileFromCeramic))

    profileForm.addEventListener('submit', async (e) => {
    e.preventDefault()
    await updateProfileOnCeramic()

    })
```

## Configuring Webpack

The following section will configure [Webpack] for this application.

1.  In the root of the `getting-started-with-ceramic` directory create a new file called `webpack.config.js` and place the following contents inside of it:
    ```javascript
    const path = require('path');
    module.exports = {
        entry: './main.js',
        output: {
            path: path.resolve(__dirname, 'dist'),
            filename: 'bundle.js'
        },
        mode: 'development',
        resolve: {
            fallback: { buffer: require.resolve('buffer') }
        }
    }
    ```
    >Make sure to check out [Webpack] if you'd like to know what this code is doing.
1. Next, you'll need to edit the `package.json` file that **currently exists** in the root directory.  You will only modify the `scripts` block of this file.  Make the following change to `package.json`:
    ```json
    "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1",
        "build": "webpack"
    }
    ```
    >Just to be clear, the change being made here is the addition of a script, named build, that calls webpack.

    The complete `package.json` can be found below:
    ```json
    {
        "name": "getting-started-ceramic",
        "version": "1.0.0",
        "description": "",
        "main": "utils.js",
        "scripts": {
            "test": "echo \"Error: no test specified\" && exit 1",
            "build": "webpack"
        },
        "keywords": [],
        "author": "",
        "license": "ISC",
        "devDependencies": {
            "buffer": "^6.0.3",
            "dids": "^3.1.0",
            "key-did-provider-ed25519": "^2.0.0",
            "key-did-resolver": "^2.0.4",
            "webpack": "^5.72.1",
            "webpack-cli": "^4.9.2"
        },
        "dependencies": {
            "@ceramicnetwork/blockchain-utils-linking": "^2.0.4",
            "@ceramicnetwork/http-client": "^2.0.4",
            "@glazed/did-datastore": "^0.3.1",
            "@glazed/did-session": "^0.0.1"
        }
    }
    ```
    >There may be small version differences in this file depending on when you completed this guide.  This is normal and should not be anything to worry about.
1. The final step is to run this newly added script from the terminal or command line.  Running this script is what will package all the previous JavaScript into a version that your browser can interpret.  Regardless of operating system, the command will be the same:

    **NPM**
    ```
    npm run build
    ```

    **Yarn**
    ```
    yarn run build
    ```

## Congratulations!

Congrats!  You can now re-open the `index.html` file in your browser or by using [LiveShare].  

Using your [Metamask] wallet you will be able to [Sign-In With Ethereum], retrieve your `BasicProfile` from Ceramic and make changes to a limited set of properties for that profile!

>If you have never configured a `BasicProfile` on the [Ceramic Network] you will initially receive no data.  You'll need to create a profile using the wallet account of your choice using the [Self.id] app or by using the form included with this application!

Make sure to come join the [Ceramic Discord] for further help and to chat with the development team!

Good luck and happy building!




[Ceramic Network]: https://developers.ceramic.network
[dapps]: https://ethereum.org/en/dapps/
[Git]: https://git-scm.com/
[Basic JavaScript]: https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/JavaScript_basics
[Webpack]: https://webpack.js.org/
[client-side JS vs Server-side JS]: https://computersciencewiki.org/index.php/Client-side_scripting_and_server-side_scripting
[NodeJS]: https://www.nodejs.org
[NPM]: https://www.npmjs.com/
[Yarn]: https://yarnpkg.com/
[GitHub]: https://www.github.com/ceramicstudio/guide-getting-started-with-ceramic


[Ceramic Youtube Channel]: https://www.youtube.com/channel/UCgCLq5dx7sX-yUrrEbtYqVw

[Decentralized Identifier]: https://www.w3.org/TR/did-core/
[public key]: https://en.wikipedia.org/wiki/Public-key_cryptography
[DID]: https://www.w3.org/TR/did-core/#dfn-decentralized-identifiers
[DID Resolver]: https://www.w3.org/TR/did-core/#dfn-did-resolvers
[DID Document]: https://www.w3.org/TR/did-core/#dfn-did-documents
[Ethereum Providers]: https://docs.ethers.io/v4/api-providers.html#providers
[Metamask]: https://metamask.io/
[Data StreamTypes]: https://developers.ceramic.network/learn/glossary/#streamtypes
[streaming data]: https://en.wikipedia.org/wiki/Streaming_data
[streams]: https://developers.ceramic.network/learn/glossary/#streams
[StreamType]: https://developers.ceramic.network/learn/glossary/#streamtypes
[Data models]: https://developers.ceramic.network/docs/advanced/standards/data-models/#data-models
[JSON Object]: https://www.json.org/json-en.html
[Ceramic nodes]: https://developers.ceramic.network/learn/glossary/#nodes
[Things We Need To Talk About]: #things-we-need-to-talk-about
[Ceramic Client]: https://developers.ceramic.network/reference/core-clients/ceramic-http/
[LiveShare]: https://visualstudio.microsoft.com/services/live-share/
[Mainnet]: https://developers.ceramic.network/learn/networks/#mainnet
[Clay Testnet]: https://developers.ceramic.network/learn/networks/#clay-testnet
[Dev Unstable]: https://developers.ceramic.network/learn/networks/#dev-unstable
[Local]: https://developers.ceramic.network/learn/networks/#local
[async function]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function

[Sign-In With Ethereum]: https://login.xyz/
[Why Sign-In With Ethereum Is A Game Changer]: https://blog.spruceid.com/sign-in-with-ethereum-is-a-game-changer-part-1/
[Ceramic datamodels repo]: https://github.com/ceramicstudio/datamodels
[full list of properties]: https://github.com/ceramicstudio/datamodels/tree/main/models/identity-profile-basic
[BasicProfile]: https://github.com/ceramicstudio/datamodels/tree/main/models/identity-profile-basic
[Configuring Webpack]: #configuring-webpack
[Event Listeners]: https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener
[Self.id]: https://clay.self.id
[Ceramic Discord]: https://chat.ceramic.network
