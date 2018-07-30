( This is a README document for main repository  [https://github.com/heroqu/react-i18n-site](https://github.com/heroqu/react-i18n-site). This document in Russian is [here](./README_RU.md) )

# React i18n Site - Documentation

## About the project

This project is what I've developed for my project-portfolio site.

In short it's a React site with:

- SPA
- i18n
- Docker microservices for backends
- separate subprojects for content management and translation
- and more...

Below is a detailed description of its architecture and building blocks.

# Table of Contents

<!-- toc -->

  * [Goals and priorities](#goals-and-priorities)
* [Subprojects](#subprojects)
* [Functional aspects](#functional-aspects)
  * [Internationalization and Content management](#internationalization-and-content-management)
  * [Routing inside application](#routing-inside-application)
    * [Single Page Application mode](#single-page-application-mode)
    * [Routing, redirecting and i18n](#routing-redirecting-and-i18n)
  * [Page naming convention](#page-naming-convention)
  * [Visual appearance](#visual-appearance)
    * [General page layout](#general-page-layout)
    * [Styling and theming](#styling-and-theming)
    * [Self-hosting the fonts](#self-hosting-the-fonts)
  * [Backend](#backend)
    * [http server for the site itself](#http-server-for-the-site-itself)
    * [Mailer](#mailer)
    * [Containerization and Microservices](#containerization-and-microservices)
* [1. Site, the React Application](#1-site-the-react-application)
  * [Component tree](#component-tree)
  * [App](#app)
  * [Redux](#redux)
  * [Router](#router)
    * [LocaleRouter](#localerouter)
    * [ScrollToTop](#scrolltotop)
  * [Intl](#intl)
  * [Theme](#theme)
  * [Layout](#layout)
    * [Header](#header)
    * [Footer](#footer)
    * [Content](#content)
  * [Miscellaneous components](#miscellaneous-components)
    * [Meta](#meta)
    * [NotFound](#notfound)
    * [LanguagePanel](#languagepanel)
    * [MenuTop](#menutop)
    * [MenuLeft](#menuleft)
    * [LastBuilt](#lastbuilt)
  * [React components embedded in content pages](#react-components-embedded-in-content-pages)
    * [The idea](#the-idea)
    * [List of currently used ones](#list-of-currently-used-ones)
    * [Data fetching and caching](#data-fetching-and-caching)
    * [Projects](#projects)
    * [ProjectFilter](#projectfilter)
    * [ProjectList](#projectlist)
    * [Accordion & AccordionSection](#accordion-accordionsection)
    * [ProjectTitle](#projecttitle)
    * [ProjectCard](#projectcard)
    * [MarkDown](#markdown)
    * [Gallery](#gallery)
    * [ResumeDownload](#resumedownload)
    * [AppLink](#applink)
    * [Link](#link)
    * [A](#a)
    * [ALink](#alink)
    * [MailForm](#mailform)
* [2. Authoring](#2-authoring)
* [3. Translating](#3-translating)
* [4. Mailer](#4-mailer)
* [5. Deploy](#5-deploy)
  * [Some specificities in Dockerfiles](#some-specificities-in-dockerfiles)
  * [docker-compose.yml](#docker-composeyml)

<!-- tocstop -->

## Goals and priorities

From the very beginning I've put before myself some goals to be reached and priorities to be followed. And here is what was really achieved:

- Restrain from using do-it-all solutions (ready-made CMS, static site generators etc.), keeping control over the codebase and avoiding vendor locking traps.
- Make it a Single Page Application (SPA).
- Try to stay with the standard build system setup of create-react-app.
- Make it fully internationalized (i18n), with languages addition as easy as editing the config file.
- Extract key phrase translation preparation into a separate subproject with all the routine tasks automated.
- Take advantage of component based development style and strive for separation of concern on all levels.
- Extract content management task into a separate subproject and free it from programming challenges as much as possible.
- Revert content authoring back to good old HTML (to make it great again). On top of that add even more power with the possibility of embedding arbitrary React components right inside the markup.
- Embedded components that require some data loading should fetch lazily, letting the main application start faster.
- If an embedded component needs some backend function, not directly related to the main backend, make it a separate server, eventually a microservice.
- Wrap all the backend servers into Docker containers.
- Arrange for running the whole site as a set of linked dockerized microservices with docker-compose.
- Find some docker aware hosting provider with deployment procedure simple and generic – ideally a mere docker-compose command – so that switching to another provider would be easy.

# Subprojects

By now the whole project had grown into 5 subprojects, with codebase in 5 sibling directories (each with its own git repo):

```
root
  ├─ site
  ├─ authoring
  ├─ translating
  ├─ mailer
  └─ deploy
```

Where:

1.  [Site](#1-site-the-react-application) – the main subproject. The React site app + static http server
2.  [Authoring](#2-authoring) – a content manager working place. Here one can author the the content pages for the site.
3.  [Translating](#3-translating) – a subproject to prepare key phrase translations for i18n system
4.  [Mailer](#4-mailer) – backend to serve the feedback form (the MailForm component placed at the site’s Contact page)
5.  [Deploy](#5-deploy) – docker-compose configuration to run the whole site as bundle of linked microservices and to deploy the site as a whole to a docker enabled hosting provider

Making subproject directories siblings is not required, but it helps by simplifying linking and interaction, like e.g.:

- key phrase extraction from __site__ into __translating__
- exporting the results form __authoring__ and __translating__ to __site__
- running microservices in __deploy__ with linking to the sources from __site__ and __mailer__

One can see the detailed description of each subproject in the dedicated sections. But firstly, let’s have a look at the project from a functional perspective:

# Functional aspects

## Internationalization and Content management

The site is fully internationalized (two languages are used currently, English and Russian, more can be added easily), and the management of content directly intersects with this i18n.

There are three main types of the sources the site's content can come from:

1.  __Common small text pieces__ inside menu items, button captions, footer etc.

All these texts are handled by [React-intl](https://www.npmjs.com/package/react-intl) package, and there is the [Translating](#3-translating) subproject dedicated to managing key-phrase translations for React-intl.

2.  __Texts of main content pages__ of the site (like _Intro_, _About_, _Contact_ etc.).

This is presumably the major content part of the site and there is also a dedicated [Authoring subproject](#2-authoring), where content manager can author those pages separately in each of the languages.

3.  __The content inside embedded components__ (more on embedded components in a [dedicated section](#react-components-embedded-in-content-pages) below).

The strategy to be applied for each particular component has to be chosen individually on a case-by-case basis.

Basically, if a component has some UI elements with texts (button captions, text labels etc.) those texts are good candidates for being handled by [React-intl](https://www.npmjs.com/package/react-intl).

Some of the embedded components may also load their data from external sources. In such cases one has to decide what is more appropriate: use [React-intl](https://www.npmjs.com/package/react-intl) to automatically translate, format and pluralize those texts, or get them loaded from data source already translated.

I do use two such data-intense components: [Projects](#projects) and [Gallery](#gallery). For both of them I've got all the texts translated in advance and stored in the data files – `projects.json` and `gallery.json` respectively.

There are some UI elements inside `Projects` component and those elements do utilize the [React-intl](https://www.npmjs.com/package/react-intl) translation functionality. But the main content texts – project titles and project descriptions – are all translated manually and stored in the source files ready-made.

## Routing inside application

### Single Page Application mode

Site operates in SPA mode, thanks to [React-router](https://www.npmjs.com/package/react-router) package, which takes control over browser's __history__ and __location__ objects. All intra-site hyperlinks are based on the `Link` component from that package, which allows for navigation without site reload by the browser.

### Routing, redirecting and i18n

There are two rules (or business requirements) that I had placed upon the application:

1.  __Root URL should redirect to the home page__

    Currently, I do the redirect `/ -> /intro`, which the Intro page

2.  __Pages in all languages but the default one should have locale prefix in URL__

    E.g. if our default language is English, and the two other languages are Russian and German, then addressing the _About_ page in those languages should be looking like this:

    ```
    English:    /about
    Russian:    /ru/about
    German:     /de/about
    ```

Both these rules are implemented inside the [Router](#router) component, which incapsulates all the routing logic inside application.

## Page naming convention

Pages, being created inside Authoring subproject, should obey the following naming style:

```
.../pages/en/you-dont-know.pug
```

then, after automatic conversion to React JSX it turns into:

```
.../pages/en/you-dont-know.js
```

The page components get their names (also automatically) similar to this one:

``` JavaScript
class Page__you_dont_know extends Component {...}
```

And the React-intl translation keys inside `messages.json` file should be:

``` JSON
{
  "en": {
    "nav.you-dont-know": "You don't know..."
  },
  "ru": {
    "nav.you-dont-know": "Вы не знаете..."
  },

}
```

This in turn leads to URLs looking like this:

```
http://some.domain/you-dont-know
```

So, in essence, we should adhere the naming style for PUG pages, and then make sure we have similar keys in `nav` groups for each language inside`messages.json` file.

## Visual appearance

### General page layout

The main layout of the page is served by a dedicated [Layout](#layout) component, which brings on screen three components [Header](#header), [Content](#content) and [Footer](#footer) and makes the Footer stick to the bottom of the page (by using some [CSS tricks](https://github.com/heroqu/react-i18n-site/blob/master/src/Layout/Layout.css)).

### Styling and theming

I use CSS Modules approach (supported by create-react-app build setup out of the box) in several key places of App hierarchy tree:

- [Theme](#theme)  (the module applying main css file and setting the theme for Material-UI)
- [Layout](#layout)
- [Content](#content) (renders main pages of the site)
- [Footer](#footer)
- [Projects](#projects) (a component embedded on some page)

Some elements (like [side menu](#menuleft), [top menu](#menutop), [Mail form](#mailform) on Contact page, [Language selector panel](#languagepanel)) use [Material-UI](https://www.npmjs.com/package/@material-ui/core) components, which in turn use CSS-in-JS and are styled partially in place and partially by the influence of [Theme](#theme) component (which is a Material-UI theme provider) located near the top of hierarchy.

### Self-hosting the fonts

I do self-host the fonts used throughout the site ([Arsenal](https://fonts.google.com/specimen/Arsenal) for texts and [Roboto](https://fonts.google.com/specimen/Roboto) for Material-UI elements), so that they would be included into application bundle by the webpack.

[Reportedly](https://www.bricolage.io/typefaces-easiest-way-to-self-host-fonts/), this can save from 300 to 1000 milliseconds during the application first start.

The `fonts` directory is looking like this:

```
fonts
├── index.js
├── arsenal-cyrillic
│   ├── index.js
│   └── 400
│       ├── arsenal-regular-cyrillic.woff
│       ├── arsenal-regular-cyrillic.woff2
│       ├── index.css
│       └── index.js
└── roboto-cyrillic
    ├── index.js
    ├── 300
    │   ├── index.css
    │   ├── index.js
    │   ├── roboto-light-cyrillic.woff
    │   └── roboto-light-cyrillic.woff2
    ├── 400
    │   ├── index.css
    │   ├── index.js
    │   ├── roboto-regular-cyrillic.woff
    │   └── roboto-regular-cyrillic.woff2
    └── 500
        ├── index.css
        ├── index.js
        ├── roboto-medium-cyrillic.woff
        └── roboto-medium-cyrillic.woff2
```

We see __index.js__ files at each level, and because of that we are able to import all the fonts with a single generic import statement:

```JavaScript
import './fonts'
```

Let's follow the flow of importing:

First it reaches the `fonts/index.js` file, which is:

```JavaScript
// fonts/index.js
import './roboto-cyrillic'
import './arsenal-cyrillic'
```

then go `fonts/roboto-cyrillic/index.js` and `fonts/arsenal-cyrillic/index.js` files, the first of which is here:

```JavaScript
// fonts/roboto-cyrillic/index.js
import './300'
import './400'
import './500'
```

then again, the first of the three looks like this:

```JavaScript
// fonts/roboto-cyrillic/300/index.js
import './index.css'
```

which finally points to a CSS file:

```css
/* fonts/roboto-cyrillic/300/index.css  */
@font-face {
  font-family: 'Roboto';
  font-style: normal;
  font-display: swap;
  font-weight: 300;
  src: local('Roboto Light '), local('Roboto-Light'),
    url('roboto-light-cyrillic.woff2') format('woff2'), /* Super Modern Browsers */
    url('roboto-light-cyrillic.woff') format('woff'); /* Modern Browsers */
}
```

In this way we load all the leaf font files, our payload.

__Why not just use the `typeface-<some font>` npm?__

This was the first option that I was thinking about. Yes, there are ready-made npm modules out there for each of the two fonts – [typeface-roboto](https://www.npmjs.com/package/typeface-roboto) and [typeface-arsenal](https://www.npmjs.com/package/typeface-arsenal) – that do almost exactly what is required. Unfortunately, there are two problems, preventing using them in our particular case:

1.  These npms load each font-face as one whole, including all the variants.

    In my application I do only use some of the variants – weight 300, 400 and 500 for Roboto, weight 400 for Arsenal – and I use no italics whatsoever. So, in my current solution I've cut each css file into smaller files – one per font variant, each placed in its own subdirectory – and selected to import only those variants that I really need. And this immediately yielded the reduction in bytes of about four times, and that's a lot.

2.  They include fonts with __latin__ subsets only, which would be fine for English-only website, but we try to go i18n here.

    So I had to get the __`.woff`__ and __`.woff2`__ files for each of the fonts that do include __latin + cyrillic__ subsets.

Still, the credits for original idea and implementation goes to npm's author Kyle Mathews (see [his original post](https://www.bricolage.io/typefaces-easiest-way-to-self-host-fonts/) for details), I've just added some little enhancement here.

## Backend

### http server for the site itself

The production bundle of the site app is just a few files (index.html, one JavaScript file, one CSS file and some other assets: images, svg files, icons and fonts), which means we can choose from a variety of http servers capable of serving static sites: Apache, Nginx, Express or even http.Server from Node core.

Currently I do use an http server from the [serve](https://www.npmjs.com/package/serve) package.

### Mailer

The [MailForm](#mailform) React component placed on the Contact page also needs some backend that would dispatch visitors' mail messages.

But really, the MailForm is not an intrinsic part of the site app as such, rather just an arbitrary React component that happens to be placed on one of the site’s pages by content manager, so I've made a separate server, the *Mailer*, to serve this mail dispatching. See the [Mailer subproject](#4-mailer) description for details.

### Containerization and Microservices

Currently the sites's backend consists of two independent servers:

- Site (node + [serve](https://www.npmjs.com/package/serve) static http server)
- Mailer (node + [Express](http://expressjs.com/) http server)

Both of them can be wrapped into Docker containers and can further be run together as two linked microservices (with the [Docker-compose](https://docs.docker.com/compose/) tool).

The details of how exactly this is all done see the description of [Deploy subproject](#5-deploy).

# 1. Site, the React Application

This is the main subproject, the very React application.

## Component tree

Here is the top part of React component hierarchy up to content pages:

```
App
└─ Redux                 (Redux provider initialized with Redux store)
   └─ Router                (all app routing logic assembled)
      │
   . . . . . . . . inside of Router: . . . . . . . . . . . . . . . . . . .
   .  │
   .  └─ BrowserRouter         (React-router provider, give SPA mode)
   .     └─ Switch
   .        ├─ Redirect          (Root -> Homepage)
   .        └─ Locale Router     (apply custom i18n URL structure)
   .           ├─ ScrollToTop      (reset position after page switching)
   .           │
   . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .
               │
               └─ Intl             (React-intl Provider, initialized with locale data)
                  └─ Theme           (Material-UI theme provider, override base theme)
                     └─ Layout             (make `Sticky Footer` layout)
                        ├─ Header
                        │  ├─ Menu Top           (wide screen)
                        │  ├─ Menu Left          (narrow screen)
                        │  └─ Language Switch    (any screen)
                        ├─ Content               (select which page to mount)
                        │  ├─ Meta               (update HTML document title)  
                        │  ├─ a page      <-- all content pages mount here
                        │  └─ ‘Not Found’ page
                        └─ Footer
                           ├─ ResumeDownload     (take care of what to download)  
                           └─ LastBuilt          (show last build timestamp)
```

Let’s take a look at key components one by one.

## App

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/App.js)

A stateless component that wires up several layers of context providers, the application router and finally mounts the [Layout](#layout).

``` JSX
const App = () => (
  <Redux>
    <Router>
      <Intl>
        <Theme>
          <Layout />
        </Theme>
      </Intl>
    </Router>
  </Redux>
)
```

One can see here that all the components go without explicit `props`, which shows a good level of decoupling: each component acts more or less like a black box, while the role of `<App>` is to place them in proper order.

## Redux

A component based on __Provider__ component from [React-redux](https://www.npmjs.com/package/react-redux) package, initialized with redux store, having the following structure of the state:

``` JavaScript
{
  locale,
  appUrl
}
```

## Router

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/Router/index.js)

A component that incapsulates all the routing logic of the application. It looks like this:

```JSX
const Router = ({ children }) => (
  <BrowserRouter>
    <Switch>
      <Redirect exact from="/" to={`/${HOME_PAGE}`} />
      <LocaleRouter>
        <ScrollToTop />
        {children}
      </LocaleRouter>
    </Switch>
  </BrowserRouter>
)
```

What we see here is the following:

- `<BrowserRouter>` – context provider from [React-router](https://www.npmjs.com/package/react-router) package

- `<Switch>` (from [React-router](https://www.npmjs.com/package/react-router)) – makes it sure that only the first matching route is rendered

- `<Redirect>` (again from [React-router](https://www.npmjs.com/package/react-router)) – implements our _first business rule_: __Root URL should redirect to the home page__.

- `<LocaleRouter>` – a local component that implements our _second business rule_, a special __Locale based URL structure__ that we are going to explain in details in the very next [LocaleRouter](#localerouter) section. Then inside go:

- `<ScrollToTop />` – a component that simply scrolls the page to the top each time the page location is changed (see the details in the [component's decription](#scrolltotop)).

- `{children}` – passes the baton down the app hierarchy.

### LocaleRouter

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/Router/LocaleRouter.js)

Takes care of the locale based URL structure rule, which is:

__“All languages, except for the default one, should go with URL prefix.”__

Basically, it calculates the correct URL based on current locale and current URL and then, in case correct URL differs from current URL, initiates redirection.

Here is the detailed step-by-step algorithm:

1.  Extract current locale value from cookie
    - with the help of [universal-cookie](https://www.npmjs.com/package/universal-cookie) package
    - If locale from cookie is empty or not supported – take the default locale (__en__ in our case)
2.  Take `location` object from props (it's there thanks to _withRouter_ function which “connects” us to [React-router](https://www.npmjs.com/package/react-router) context) and get current URL as `location.pathname`.
3.  Parse this URL: extract locale part and payload part (which I call `appUrl`).
    - E.g.: URL=`/ru/about`, locale part is `/ru`, appUrl is `/about`
4.  If __locale__ or __appUrl__ differ from what is stored in redux, then schedule the dispatch of redux action to update the state
    - I use delay-till-next-tick function here, so that all the state shaking would happen after the current render cycle.
5.  Now, knowing locale and appUrl, calculate the correct URL.
6.  Compare correct URL and current URL.
7.  If they do differ then fire the redirection with the aid of React-router’s `<Redirect>` component.
    - There are 2 reasons for such an URL mismatch that I can think of:
      - wrong locale prefix (`/en/about` while it should be just `/about`, `/ru/about` instead of `/de/about`, etc.)
      - wrong letter casing (`/About` or `/ABOUT` instead of `/about`)
8.  If no difference found then proceed to rendering children components.

__Note #1:__ We do not check the existence of the underlying page or resource at this stage, we only check the address format here: _is it looking like a bonafide address that does comply our URL rules or not._

__Note #2:__ Importantly, it is that delayed dispatch of `locale` and `appUrl` to the redux store (mentioned in point 4), that eventually leads to switching content pages on the screen. The switching happens inside the [Content component](#content) , which gets these `locale` and `appUrl` values from props and then chooses what page to render based on that. The redirection as such is not the cause of re-rendering and page switching, it only does update the `history` and the `location` objects of the browser, keeping them in sync with the application state (after all, this is what SPA is all about, switching cause and effect: instead of `URL -> state` we have `state -> URL`).

### ScrollToTop

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/Router/ScrollToTop.js)

Scrolls the page to the top each time the page location change is detected.

We have to take care of that, because in SPA mode the browser is effectively out of reloading business, and that's the price of usurping the power.

A verbatim copy of the one from React-router's documentation site.

## Intl

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/Intl/index.js)

A component that wraps an IntlProvider component from [React-intl](https://www.npmjs.com/package/react-intl) package, giving the contextual access to its i18n functions down the component hierarchy.

Wrapping includes some initialization steps, like loading the so called _locale data_ (formats and pluralization rules for particular locale), as well as injecting (through props) the current locale value and key phases translations.

Key phases translations preparation is being covered in details in [Translating subproject](#3-translating) section.

## Theme

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/Theme/index.js)

In this module we do two things:

- activate the main CSS file, which is:
    ``` JavaScript
    import './index.css'
    ```
- Setup the `<Theme>` component by wrapping the MuiThemeProvider – a theme provider from [Material-UI](https://www.npmjs.com/package/@material-ui/core) package. It gets initialized with our theme object, overriding some default CSS styles and media query threshold values.

The resulting Material-UI theme becomes accessible everywhere down the component hierarchy.

## Layout

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/Layout/index.js)

Makes the main layout and brings on screen three components:

- Header
- Content
- Footer

It also uses a CSS trick to stick the Footer to the bottom of the page in case the content height is insufficient.

I’ve made this a separate component with a single clear responsibility: the layout and nothing else.

### Header

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/components/Header/index.js)

A component that displays three child components, [MenuTop](#menutop), [MenuLeft](#menuleft) and [LanguagePanel](#languagepanel):

``` JSX
const Header = () => (
  <Fragment>
    <Hidden smDown>
      <MenuTop />
    </Hidden>
    <Hidden mdUp>
      <MenuLeft />
    </Hidden>
    <LanguagePanel />
  </Fragment>
)
```

Here:

- MenuTop – is displayed on screen with width >= 768px
- MenuLeft – is displayed on screens with width < 768px
- LanguagePanel is shown always.

The 768px value is one of the media query breakpoints that are set as part of our custom Material-UI theme inside [Theme](#theme) component.

### Footer

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/components/Footer/index.js)

- On the center: displays some text, translated by React-intl.

- On the left: shows [ResumeDownload component](#resumedownload).

- On the right: renders the [LastBuilt component](#lastbuilt) – but only when in debug mode.

Uses a horizontally seamless background picture – an indexed PNG with two colors, black and transparent, thus very light weight – only 2.6Kb at 815x600 px size.

### Content

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/components/Content.js)

Displays the main content of current page.

Here is the essential part of the source code:

``` JSX
import pages from './pages'
import NotFound from './NotFound'
import Meta from './Meta'

import { getI18nAttr } from '../lib/i18n'
import { slugToName } from '../lib/pageNaming'

// ...

const Content = props => {
  const { locale, appUrl } = props

  const pageKey = slugToName(appUrl)
  const Page = getI18nAttr(pages, locale, pageKey) || NotFound

  return (
    <Fragment>
      <Meta {...{ appUrl }} />
      <div className="Content">
        <Page {...props} />
      </div>
    </Fragment>
  )
}
```

There are two components that we render here:

- Meta
- Page

[Meta component](#meta) is responsible for setting HTML document metadata based on current page (currently the `title` attribute only).

The `Page` is the actual content, but we have to select the exact component that should go under that name. To do this we first load all the pages (each one is a full-fledged React component) with a single `import pages from './pages'` statement, which is possible thanks to all the index files at both levels of `src/components/pages` directory structure:

```
src
└─ components
   └─ pages
      ├─ index.js              <-- main index
      ├─ en
      │  ├─ index.js           <-- index inside locale
      │  │
      │  ├─ about.js
      │  ├─ contact.js
      │  ├─ experience.js
      │  ├─ open-source.js
      │  ├─ ...
      │  └─ skills.js
      └─ ru
         ├─ index.js          <-- index inside locale
         │
         ├─ about.js
         ├─ contact.js
         ├─ experience.js
         ├─ open-source.js
         ├─ ...
         └─ skills.js
```

All the leaf files are ready-made components prepared in advance (see [Authoring subproject](#2-authoring) for details on this).

Now we have the `pages` object, which contains all the individual page components and should look similar to this:

```JavaScript
const pages = {
  en: {
    intro,
    skills,
    experience,
    open_source,
    gallery,
    education,
    about,
    contact
  },
  ru: {
    intro,
    skills,
    experience,
    open_source,
    gallery,
    education,
    about,
    contact
  }
}
```

Now, to get the current page component we can write (roughly):

``` JavaScript
// in real code I use:
// const Page = getI18nAttr(pages, locale, appUrl) || NotFound

const Page =
    pages[locale][appUrl]
    || pages[defaultLocale][appUrl]
    || NotFound
```

Here we can see that if some page is not there in current language, we resort to the same page in default language, and if nothing is found, then fall back on `<NotFound>` special component, which is a our custom `404 page not found` error page.

The refined version of this complex assignment logic is incapsulated inside `getI18nAttr` utility function that is used in the real code.

Let's have a look inside the individual content component (that comes here ready-made from [Authoring](#2-authoring) subproject):

```JSX
import React, { Fragment } from 'react'

class About extends React.Component {
  render() {
    return (
      <Fragment>
        <h1 className="Title">
          About this site
        </h1>
        <div className="PageContent">
          <p>
            Lorem and Ipsum were here...
          </p>

         ...

        </div>
      </Fragment>
    )
  }
}

export default About
```

This is what will be used as the main content of current page.

## Miscellaneous components

### Meta

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/components/Meta.js)

Uses the `Helmet` component from [react-helmet](https://www.npmjs.com/package/react-helmet) package to set HTML document metadata based on current page (`locale`, `appUrl`). Currently I do only set the `title` of the page which is going to show up in the browser tab along with the favicon.

I use React-intl to display the text of the `title` in current language.

### NotFound

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/components/NotFound.js)

A component to display our custom `404 page not found` error page.

### LanguagePanel

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/components/Header/LanguagePanel.js)

Is styled with [Material-UI](https://www.npmjs.com/package/@material-ui/core) and displays buttons for each language (currently the two, English and Russian).

For a site with too many languages it's not going to work, sure, and we would have to use a drop-down list instead or something. But for a two or three languages the buttons are kind of straightforward, I think.

### MenuTop

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/components/Header/MenuTop.js)

Uses [Material-UI](https://www.npmjs.com/package/@material-ui/core). Active item is highlighted.

### MenuLeft

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/components/Header/MenuLeft.js)

Very similar to [MenuTop](#menutop).

### LastBuilt

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/components/LastBuilt.js)

A tiny component that is used inside [Footer](#footer) to display the date and time the application was built the last time.

This is basically of interest during the active stage of site development as it is sometimes useful to be sure that what you see on the screen is actually an updated version and not the one from cache. Therefore this component is only visible when the `REACT_APP_DEBUG_MODE` environment variable is set to 'true'.

Technically, to get that timestamp available at run time I wrote a [script](https://github.com/heroqu/react-i18n-site/blob/master/timestamp-the-build.js) that can be run before each build (see `scripts` inside [package.json](https://github.com/heroqu/react-i18n-site/blob/master/package.json)) to insert the `REACT_APP_BUILD_TIMESTAMP` var assigned with current timestamp value into `.env.local` file. This variable then becomes accessible at application runtime just like all the other variables from all the `.env` files – see [config.js](https://github.com/heroqu/react-i18n-site/blob/master/src/config.js) module which makes it happen.

## React components embedded in content pages

### The idea

Content manager, staying within [Authoring subproject](#2-authoring), can insert any desired React component into the content page, which is being edited in html (pug).

It sounds too good, but there is no catch, may be just a little trade off, which is that the situation implies some level of dialogue, or negotiation, between content manager and site programmer: content manager is in the role of a client who expresses a desire for some functionality and the programmer is the one who can provide that functionality in a form of some component.

The good thing is that when that negotiation is over and the list of what is available for inclusion is settled, content manager can use available components as he or she will.

### List of currently used ones

Here are the components that are currently being embedded in one or more of the content pages:

- [Projects](#projects)
- [Gallery](#gallery)
- [ResumeDownload](#resumedownload)
- [AppLink](#applink)
- [Link](#link)
- [A](#a)
- [MailForm](#mailform)

Again, from the point of content manager who inserts them inside [Authoring subproject](#2-authoring), all these components are more or less like black boxes, or, better put, like any other html tags:

```
<Projects />

<Gallery name='site_for_client_n' tag='web'>Site for Company N</Gallery>

<MailForm />

<AppLink to='/about'>About page</AppLink>

Download my pdf resume <ResumeDownload>here</ResumeDownload>
```

The little magic does happen while automatic conversion from pug to React: I use a special Gulp transformation step that looks for all the tags beginning with upper cased letters, and, considering them to be React components, adds appropriate import statements at the top of the final JSX file.

E.g. if we have `<Fragment>`, `<Link>` and `<Gallery>` tags somewhere on the page, then the following lines will be added at the top:

```JavaScript
import React, { Fragment } from 'react'
import Link from '../../Link'
import Gallery from '../../Gallery'
```

We can see here, that `Fragment` is treated differently, and purposely so. We do also assume that all the hand made components have to be rooted inside `../../` directory, which corresponds to `/site/src/components/`, so some discipline on the part of site programmer is expected.

### Data fetching and caching

Currently I have two embedded components on my site that fetch data from backend, [Gallery](#gallery) and [Projects](#projects). I considered two option of how to arrange fetching and storing the fetched data:

1. fetch in redux action and then keep the data in redux store
2. fetch inside the component instance
   1. and keep it in component state
   2. and keep it in cache

Initially I took the first option and used redux-thunk middleware to do async fetch and action dispatch. But then I've understood that although it works perfectly fine, it violates the separation of concerns principle that was introduced with embedded components. The idea is that main react application should be agnostic to what is happening with those components landed on content pages.

Having these thoughts in my mind, I finally switched to the second option: fetch all the data right from inside the component and don't pollute the code of the main app. The problem with this approach is that when we change the page and then come back again then the component on the page gets destroyed and then recreated from scratch again, which means a new cycle of fetching is initiated. This is partially alleviated by browser caching layer, so that instead of each time fetching all the data from backend a new, it only checks that the resource was not changed since last retrieval and gives the application the data from cache instead. But still there is a request to get this 304 response and it takes the time of a full round trip between client and the server.

To avoid this requesting business altogether, we can apply some caching at the application level. I made a very simple memory based cache that is being wrapped around the fetcher function for a given resource. Here is a little simplified version of how it looks:

``` JavaScript
function FetchWithCache(fecther, ttl) {
  let value
  let timestamp

  const isExpired = () =>
    // if never fetched
    typeof timestamp !== 'number' ||
    // or, if TTL is exceeded
    Date.now() > timestamp + ttl

  return async () => {
    let isNew = false
    if (isExpired()) {
      try {
        value = await fecther()
        timestamp = Date.now()
        isNew = true
      } catch (e) {
        console.error('Error fetching data', e)
        // and now just return older data
      }
    }

    return { value, timestamp, isNew }
  }
}
```

And then, inside the __Gallery__ component it goes:

``` JavaScript
import fetchJsonData from '../lib/fetchJsonData'
import FetchWithCache from '../../lib/FetchWithCache'

const fetcher = async () => fetchJsonData('/data/gallery.json')
const TTL = 3600000 // 1 hour Time to Live
const fetchWithCache = FetchWithCache(fetcher, TTL)

// ...

class Gallery extends Component {
  // ...

  // share images between all instances
  static images = []

  async loadData() {
    Gallery.images = (await fetchWithCache()).value

    // ...

    // go re-render
    this.setState( ... )
  }

  componentDidMount() {
    this.loadData().catch(console.error)
  }
  // ...
}
```

We see here, that data loading is initiated from `componentDidMount` as usual, but then the data is fetched through a fetcher wrapped with cache, so only the first time there is a real request to the backend (or, when the cache is expired). Actually, if there is more then one __Gallery__ component on the page (as it is with the Gallery page of my site), then each of the components makes a real request to the backend, as the cache is still empty at the check point, because fetching takes time and goes async.

Another nuance here is that we use __static images__ member instead of __this.state.images__ instance member to store the data, and this __Gallery.images__ is shared among all instances of Gallery. And as we are not using this.state, then we have to make sure to re-render – either with `this.setState( something )` or, if there is nothing to update inside `this.state` (as in case of `Projects` component), then at least call `this.forceUpdate()`.

See full source code for more details:

- [FetchWithCache](https://github.com/heroqu/react-i18n-site/blob/master/src/lib/FetchWithCache.js)
- [Gallery](https://github.com/heroqu/react-i18n-site/blob/master/src/components/Gallery.js)
- [Projects](https://github.com/heroqu/react-i18n-site/blob/master/src/components/Projects/index.js)

### Projects

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/components/Projects/index.js)

A rather complex embedded component consisting of a few building blocks:

```
Projects
├─ ProjectFilter
│  └─ Tag                    (many)
└─ ProjectList
   └─ Accordion
      └─ AccordionSection    (many)
         ├─ ProjectTitle
         │  └─ Tag           (many)
         └─ ProjectCard
            └─ MarkDown
```

Its purpose is to bring on screen the list of projects and the tag filter for that list.

Visually the page area of this component is divided into two panels, with `ProjectFiler` on the left and `ProjectList` on the right. On a narrow screen the second jumps under the first.

__Data loading__

Projects data has to be loaded from some external data source. Currently it is  `/public.data/projects.json` file (served as static asset by site's HTTP server), but it can just as easily be a collection from MongoDB or whatever (actually, I was using MongoDB for projects data until recently, when I switched to a json file for this site).

The loading starts from inside `ComponentDidMount`, the data then undergoes some processing for the purpose of renumbering, sorting, adding computed fileds and tags extraction, to finally end up in the component's state as two arrays `{ tags, projects }`, where `tags` is the list of all tags ever used inside projects, and the `projects` is the list of objects, each of which stands for individual project and looks similar to this:

```JavaScript
const individualProject = {
  id: 1,
  badge: 2,
  badgeMinor: 0,
  badgeFull: '2',
  startDate: { $date: '2015-03-01T00:00:00.000Z' },
  endDate: { $date: '2016-05-01T00:00:00.000Z' },
  monthSpan: '2015-03 – 2016-05',
  tags: [
    'Backend development',
    'Node.js',
    'JavaScript',
    'TypeScript',
    'Express',
    'Restify',
    'Mongo',
    'Microservices',
    'RESTfull API',
    'GitFlow'
  ],
  en: {
    name: 'Node.js backend development',
    employer: 'Company name',
    description:
      "In this role I was developing microservices for a Node.js backend. ..."
  },
  ru: {
    name: 'Бэкенд разработка на Node.js',
    description:
      'В этой компании я занимался разработкой микросервисов для Node.js бэкенда. ...'
  }
}
```

### ProjectFilter

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/components/Projects/ProjectFilter.js)

This component fills its space with a multiline list of all available tags that can be used to filter projects. User can select and deselect any tags with the mouse. All selected tags are then displayed in red color at the very top of the [Projects](#projects)' area. The `Reset` link can be used to deselect all the tags in one click, returning the filter to its initial state.

Filter logic is __OR__, which means, the more tags are selected, the more permissive it becomes, as if we would say in plain English: “Show me all the projects that have tag1, OR have tag2, OR have tag3, etc.,”. When no tags are selected, then no filtering is applied and all the projects are displayed.

### ProjectList

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/components/Projects/ProjectList.js)

This component show the list of the projects that match the current filter. Each projects shows up inside an expandable/collapsible element, an __AccordionSection__ component. All the expandable sections together are wrapped in a single controlling component called __Accordion__.

### Accordion & AccordionSection

__Accordion__ component (from [react-accordion-composable](https://www.npmjs.com/package/react-accordion-composable) npm package that authored by me) is used to introduce a collective behavior among otherwise independent expandable/collapsible sections. If the so called _Accordion mode_ is ON, then no more the one section can be expanded at a time. This is an _accordion effect_: you expand a section, the previously expanded section gets collapsed automatically. Accordion can also function in _Accordion mode_ OFF, which is equivalent to non-interference: each section can be expanded or collapsed independently.

An interesting feature of my accordion is that each section applies collapsibility to its children in a composable way. What I mean here is that one doesn’t have to use props to specify the title and body of the section, the section just takes the first child as the title part and all the rest as the body part. In this way it unobtrusively gives all the power to the children. It serves one and only one purpose, showing or hiding, while it is the children who have to decide what to show and what to hide. One can have a look at the [demo site](https://heroqu.github.io/react-accordion-composable-demo/) of [react-accordion-composable](https://www.npmjs.com/package/react-accordion-composable) that illustrates all these concepts.

In our case each section has two child components:

- __ProjectTitle__ is the title part and is always visible
- __ProjectCard__ is the body part and can be either hidden or visible

### ProjectTitle

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/components/Projects/ProjectTitle.js)

Depicts the title of a project, its number and the list of its tags on the second row.

### ProjectCard

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/components/Projects/ProjectCard.js)

Consists of upper band and main panel. The upper band displays the time span of the project and the employer's company name. The main panel holds the project's description, rendered from Markdown format to React JSX with the help of [MarkDown](#markdown) component (see below).

### MarkDown

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/components/Projects/MarkDown.js)

The goal of this component is to convert Markdown text to JSX on the fly.

Again, just like as with `pug` format of content pages that we use inside the [Authoring](#2-authoring) subproject, here we have the possibility to embed React components into the flow of markdown. I’ve used the [Gallery](#gallery) component in some of the project descriptions.

Technically, the component is a wrap around the same named (except for different letter casing of `D`) `Markdown` component from [markdown-to-jsx](https://www.npmjs.com/package/markdown-to-jsx) package, applying some configuration, namely, two tag replacement rules:

```
<Photo> -> <Gallery className='LinkInProjectCard'>
<a> -> <ALink className='LinkInProjectCard'>
```

While `Photo -> Gallery` is basically just a convenience rename, so that one could use more intuitive `<Photo src=...>` inside the content, the `<a> -> <ALink>` replacement is more important: the `<ALink>` component is an *intelligent selector*, which renders different components depending on the type of href address – external, intra-site or mailto (see details in the [ALink component description](#alink)).

### Gallery

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/components/Gallery.js)

This component utilizes the [react-image-lightbox](https://www.npmjs.com/package/react-image-lightbox) package, which allows a user to open a picture in a _lightbox mode_ (popup overlaying window, dimming out the current page).

The Gallery component loads its data only once after mounting, much the same way as the [Projects](#projects) component does (except here we don't need any post-processing).

After that moment the data is stored inside the state as `{ images }`, which holds an array of individual picture objects and looks like this:

```JavaScript
const galleryData = [
  //...,
  {
    order: 2010,
    name: 'vintergatan',
    src: '/data/gallery/sites/vintergatan.png',
    tags: 'web',
    en: {
      caption: 'Site for astrological association in Malmö, Sweden'
    },
    ru: {
      caption: 'Сайт для астрологического общества из Швеции'
    },
    year: 2002
  },
  //...
]
```

Here we can see, that each picture object has the `name` attribute that serves for simple addressing. We can therefore display this exact picture with the following piece of code:

```
    <Gallery name="vintergatan" />
```

Other mode to open the `Gallery`:

- to browse all images available:

```
    <Gallery />
```

- to browse a sub-collection – only pictures with certain tag (or tags):

```
    <Gallery tag="web">web</Gallery>
```

- to browse a sub-collection, starting from a specified image:

```
    <Gallery name="vintergatan" tag="web">web</Gallery>
```

And I think this is a pretty versatile way of showing pictures.

One last note about Gallery: it respects current locale to show image captions in current language.

See the [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/components/Gallery.js) for implementation details.

### ResumeDownload

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/components/ResumeDownload.js)

A little component that incapsulates what exactly should happen when visitor clicks on “Download my resume” link.

The idea is that content manager doesn’t have to know where exactly the downloadable file is on the server, or should it be a PDF built on the fly, should it be the same file for all languages or, on the contrary, many different files one per language, etc. All these decisions are delegated to site programmer and can be a subject to change.

In this way content manager can simply place `<ResumeDownload />` tag somewhere on some page and forget about it.

### AppLink

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/components/AppLink.js)

A component that is a wrap around the `Link` component from [React-router](https://www.npmjs.com/package/react-router) package. We know that the goal of the `Link` is to provide in-site navigation in a React-router way, i.e. without app reload by the browser. What the `AppLink` adds on top that is it takes care of URL locale prefixing, so that we could specify only the the logical part of an address that is void of language. E.g. we target the About page the same exact way in any language:

```JSX
<AppLink to='/about'>...</AppLink>
```

The `AppLink` will add the missing prefix to the `to` attribute at render time.

And here is the essential source code of this component:

```JSX
import { Link } from 'react-router-dom'

const AppLink = ({ locale, to, children, className }) => (
  <Link to={`${localeURLPrefix(locale)}${to}`} className={className}>
    {children}
  </Link>
)
```

where we calculate the prefix based on current locale.

This way we've considerably simplified inner referencing at authoring time.

Note, that although the [AppLink](#applink) component is not absolutely required in a strict sense (as [LocaleRouter](#localerouter) is going to fix the URL prefix ultimately anyway), but still it is useful, because it forms the correct URL _beforehand_ and thus saves us from unnecessary redirection cycle.

### Link

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/components/Link.js)

An alias for `<AppLink>`. Introduced to make writing content in the [Authoring](#2-authoring) little less verbose.

### A

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/components/A.js)

Is a wrap around the standard html `<a>` tag, pre-populated with

```
  target="_blank" rel="noopener noreferrer"
```

attributes to safely open an external link in a new window.

The purpose of these attributes is to fight the security vulnerability
related to opening link with `target="_blank"`. The point is that
without `rel="noopener noreferrer"` the newly opened page would
have access to the current page `window` object via `window.opener`,
and it can maliciously navigate current page to a different URL using
`window.opener.location = newURL`. See e.g. [here](https://developers.google.com/web/tools/lighthouse/audits/noopener)
for detailed explanation).

Now instead of

```html
<a href="https://github.com/heroqu/react-i18n-site"
   target="_blank"
   rel="noopener noreferrer"
>
Site repo
</a>
```

one can write a somewhat terser:

```html
<A href="https://github.com/heroqu/react-i18n-site">
Site repo
</A>
```

### ALink

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/components/ALink.js)

An *intelligent link*, a component that renders as one of the three different components based on the href address type:

- if href is `mailto`, or empty, then it renders a standard html `<a>` tag.
- if href is `http(s)`, then [A component](#A), which is `<a>` with `target="_blank" rel="noopener noreferrer"`.
- if href is internal, renders [Link component](#link), which is based on React-router's `Link`.

### MailForm

:link: [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/components/MailForm.js)

The goal of this component is to show on screen a mail feedback form. It uses several UI components from [Material-UI](https://www.npmjs.com/package/@material-ui/core) package, some of them directly (`Paper`), some through other components (`TextField` , `Button`). It does also utilize the validation capabilities provided by [react-material-ui-form-validator](https://www.npmjs.com/package/react-material-ui-form-validator) package, making sure each submitted field is not blank and that user’s email is valid.

When the user pushes the Send button, the component makes a POST request against the Mailer backend (see [Mailer subproject](#4-mailer)).

If the connection to the backend fails, or backend itself breaks or just unable to connect to the 3rd party SMTP, then red message is displayed to inform about server or connection problem. If mail sending succeeds, then white color message “Has been sent” informs about that too.

See the [source code](https://github.com/heroqu/react-i18n-site/blob/master/src/components/MailForm.js) for implementation details.

# 2. Authoring

:link: [repo](https://github.com/heroqu/rs-authoring)

A subproject that serves a working place for content manager.

Each page here is being prepared as a file in `pug` (`jade`) format, which is essentially an html.

Technically the authoring process consists of the following sequence of steps:

1.  Fill page files with content in `pug` format (the only manual step)

2.  __gulp pug2React__ – run the Gulp task to convert all the pug files to React JSX format

3.  __gulp indexBuildDir__ – insert `index.js` files into each locale sub-directory and into `build` root directory:

```
authoring
└── build
    ├── en
    │   ├── index.js      <-- this one
    │   ├── About.js
    │   ├── Contact.js
    │   ├── ...
    │   └── Skills.js
    ├── ru
    │   ├── index.js      <-- this one
    │   ├── About.js
    │   ├── Contact.js
    │   ├── ...
    │   └── Skills.js
    └── index.js          <-- and this one in the root
```

I've wrote a special [authoring/lib/indexBuildDir.js](https://github.com/heroqu/rs-authoring/blob/master/lib/indexBuildDir.js) script which does all the heavy lifting for this task.

The point of all this index files is to allow nested importing of all the components in one single step:

```JavaScript
import pages from 'path_to_build_dir'
```

This importing is going to happen inside the main React app, where all the content pages are to be consumed. Therefore generating all these index.js files here, in the Authoring, __increases the decoupling__ between the two subprojects.

And really, the React app (all this happen inside its [Content ](#content) component) imports all the pages at once without caring about particular names. And it is only at the very last moment, when the URL address requires it to mount a page with particular name, that the Content component does finally see, if the page with that name is available or not. And this is the run time. At compile time the React site stays essentially agnostic of what exactly is there on stock inside its `src/components/pages` directory.

4.  __gulp export__ – export (plain copy) all the resulting React JSX files to the directory of site subproject.

# 3. Translating

:link: [repo](https://github.com/heroqu/rs-translating)

This subproject is designed to facilitate the preparation of key phrase translation data for [React-intl](https://www.npmjs.com/package/react-intl) package. All routine operations can be executed as Gulp tasks.

Main files and directories to be dealt with here are looking like this:

```
translating
├── build
│   └── messages.json
├── edited
│   ├── en.json
│   ├── ru.json
│   └── ...
├── extracted
│   ├── Component1.json
│   ├── Component2.json
│   └── ...
├── sample
│   └── sample.json
│
├── gulpfile.js
└── package.json
```

Here are the steps one is supposed to do inside this subproject:

- `gulp extract`

  - Extracts key phrases from all `<FormattedMessage>` React components used inside the main site. The results are stored as json files inside `/extracted` directory.

- `gulp sample`

  - Collects the key phrases from all those files, dedupes them and write the result into the `sample.json` file inside `/sample` subdirectory. This file can be treated as a starting point for working with any particular language.

- Now we need some human to do the actual translation. One has to make a separate copy of`sample.json` for each target language, rename it accordingly (`en.json`, `ru.json` etc.), put it into `/edited` directory and finally edit: translate all the key phrases.

- `gulp build`

  - Merges all edited translation files into production-ready file `/build/messages.json`

- `gulp deploy`
  - Copies `/build/messages.json` file into a directory of the main React site.

# 4. Mailer

:link: [repo](https://github.com/heroqu/mailer)

Little server, exposing an HTTP API for sending email messages.

Can be used as a backend for feedback forms. It takes care of dispatching and effectively hides authentication credentials from the client (these credentials would be exposed to site visitors if we try to send directly from React app, which code is always visible).

It's based on Express and [Nodemailer](https://www.npmjs.com/package/nodemailer) and have a simple API: it listens for POST requests at `/send` route.

Two transport options are available:
- Gmail
  - sends SMTP requests to specified Gmail account.
- [Mailgun](https://www.mailgun.com/)
  - sends HTTP requests to the MAIL API of this provider.

To deliver a message, client has to make a POST request with `{ name, email, subject, message }` parameters in the body. When a new request arrives, server does the following:

- Extracts { name, email, subject, message } params from the request body
- Performs validations:
  - Validates that no parameter is blank
  - Validates that email parameter look like a valid email,
  - If any of these validations fail, sends an HTTP response with 400 status and appropriate message
- Sanitizes each parameter by removing possible html tags
- Creates an email message object
- Choose the transport (Mailgun or Gmail) and try to send the message through it with [Nodemailer](https://www.npmjs.com/package/nodemailer)
- If Nodemailer fails to dispatch the email, returns Nodemailer's error to the client.

Because this http server runs separately from main http server of the site, it listens at a different address, so the CORS policy is applied (with [cors](https://www.npmjs.com/package/cors) Express middleware) to avoid blocking cross origin requests by browser. CORS is configured with a whitelist of where the server can accept requests from.

All the configuration settings, including CORS whitelist, auth credentials, transport type and server port, are read from environment variables and an be set either directly or through one of .env files (which are then parsed and applied by [dotenv](https://www.npmjs.com/package/dotenv) utility).

In production this server can be started as Docker microservice (see [Deploy subproject description](#5-deploy) for details).

# 5. Deploy

:link: [repo](https://github.com/heroqu/rs-deploy)

The subproject holding [docker-compose](https://docs.docker.com/compose/) configuration to launch [Site](#1-site-the-react-application) and [Mailer](#mailer) servers as Docker microservices. Directories `/site` and `/mailer` are siblings to `/deploy`, and to simplify their referencing I’ve created two symlinks, so now it all looks like this:

```
deploy
├── docker-compose.yml
├── mailer -> ../mailer
└── site -> ../site
```

At the same time each of server directories has its own Dockerfile at the root, therefore effectively docker-compose has to deal with the following files:

```
deploy
├── docker-compose.yml
├── mailer -> ../mailer
│   ├── Dockerfile
│   └── ...
└── site -> ../site
    ├── Dockerfile
    └── ...
```

And we have everything in the scope of subproject root directory (unless symlinks are broken).

## Some specificities in Dockerfiles

Let's have a look at the Dockerfile from [Site](#1-site-the-react-application) subproject (the source code is [here](https://github.com/heroqu/react-i18n-site/blob/master/Dockerfile)):

```Dockerfile
# /site/Dockerfile

ARG DISTRO=node:10.6.0-alpine

ARG SERVE_VERSION=9.2.0

FROM $DISTRO as builder

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build

FROM $DISTRO as deploy

RUN npm i -g serve@$SERVE_VERSION

WORKDIR /usr/src/app

COPY --from=builder /usr/src/app/build/ .

ENV PORT=3010

EXPOSE $PORT

CMD ["serve"]
```

We can notice here that:

- We use `ARG` instructions to parametrize base linux image and version of [serve](https://www.npmjs.com/package/serve) package. Both are given default values that can be overridden inside `docker-compose.yml`.

- PORT environment variable is parametrized as well, and can be overridden in `docker-compose.yml` either. We use `ENV` instead of `ARG` though, as we need the port to also be available during the container's run time, when the [serve](https://www.npmjs.com/package/serve) server will be running.

- `EXPOSE $PORT` achieves double goal here:

  - exposes the specified port for the whole container
  - sets the working port for the [serve](https://www.npmjs.com/package/serve) server, which (starting from version 9.1) can read the port number from `PORT` environment variable
    - Lucky for us, as otherwise it would be not so easy to configure serve's port, because Docker's `CMD` does not support variable expansion in its exec form as of yet, so we couldn't just write `CMD ["serve", "-p", "$PORT"]` – that wouldn't work. And we wouldn't like to abandon the exec form of `CMD` either, so some workaround would be required.

- finally, we take advantage of [Docker multistage builds](https://docs.docker.com/develop/develop-images/multistage-build/) to make the final production image little lighter and cleaner
  - we first install the app and its dependencies, then build the production bundle (with `npm run build`) and stop there. At this point we start a from a new image (with the same exact node:alpine version) and copy only the build directory from previous image. We also do install [serve](https://www.npmjs.com/package/serve) npm package globally to be able to run static http server.

The Dockerfile for [Mailer](#4-mailer) is very similar (see the [source code](https://github.com/heroqu/mailer/blob/master/Dockerfile)), except it doesn't install `serve`, but it installs node-gyp, without which we would get a dependency install errors during npm install stage:

```Dockerfile
# for some npm packages to be able to build natively
# we need node-gyp
# see https://github.com/nodejs/docker-node/issues/282
RUN apk add --no-cache --virtual .gyp python make
```

## docker-compose.yml

Nothing exotic, here is the [source code](https://github.com/heroqu/rs-deploy/blob/master/docker-compose.yml).

