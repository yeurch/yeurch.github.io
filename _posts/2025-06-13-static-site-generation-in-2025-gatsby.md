---
layout: post
title:  "Static Site Generation in 2025 - Gatsby"
---
Following on from the previous post about static site generators, the next one I took for a spin was Gatsby.

Installing via `npm init gatsby` is a great experience. This will automatically download the `create-gatsby` package and executed it, meaning there's no need to install anything prior to development, other than node and npm.  This init script guides us through a terminal user interface (TUI) to select various options, such as project name, whether we want to use a CMS, and some optional plugins we might want to enable.  It finishes by suggesting we run `npm run develop` to launch our newly created site.

And that's where the wheels come off a little bit.  I'm running node version 22.11.0, and the first run gives an error:

![Gatsby error](/assets/images/20250613_gatsby/gatsby-error.png "Gatsby error")

Apparently, the solution is to downgrade node to version 20, but this workaround won't last forever, and the bug was [first reported](https://github.com/gatsbyjs/gatsby/issues/39019) a year ago. Node 22 is the current LTS release, so having to downgrade is disappointing.  I feel it's not a great first run experience, but it appears the "error" is benign (which begs the question, why not a warning?)  I also appreciate that this is a transitive dependency that Gatsby takes, so it might not be in their hands to fix ... I didn't have time to dig into whether Gatsby's direct dependencies have been patched to fix this.  But overall, I think that if a user is running the current LTS and encounters this, it should be at least called out in a fairly obvious way on the Gatsby quick start guide.

**UPDATE** - after writing the above, based on my use of Gatsby's [quick start](https://www.gatsbyjs.com/docs/quick-start/), I took a look at the full tutorial; [section 0](https://www.gatsbyjs.com/docs/tutorial/getting-started/part-0/) does say that the requirements are for Node.js (v18 or newer, but less than v21).  Still disappointing, but less so.  I feel this should be more prominently called out, including on the quick start guide. Further down the section 0 page, it says to install node with `brew install node` which would not meet the requirements ... at the time of writing, this will install Node v24.  It likely should say to use `brew install node@20`.

### Using React

The getting started guide then took me into writing `.js` pages to show pages.  This is a much steeper learning curve than just dropping Markdown files in a directory like with Hugo.  For example, to create an About page, the code below is suggested as a starting point and should be saved at `src/pages/about.js`:

```javascript
// Step 1: Import React
import * as React from 'react'

// Step 2: Define your component
const AboutPage = () => {
	return (
		<main>
			<h1>About me</h1>
			<p>Hi there! I'm Richard.</p>
		</main>
	)
}

export const Head = () => <title>About Me</title>

// Step 3: Export your component
export default AboutPage
```

The about page can then be found under `/about` in the generated site.

### What About Layouts?

Layouts can be implemented too, e.g.:

```javascript
import * as React from 'react'
import { Link } from 'gatsby'

const Layout = ({ pageTitle, children }) => {
	return (
		<div>
			<Link to="/">Home</Link>
			<hr/>
			<h1>{pageTitle}</h1>
			{children}
		</div>
	)
}

export default Layout
```

And then referencing it in the about page like this:

```javascript
import * as React from 'react'
import Layout from '../components/layout'

const AboutPage = () => {
	return (
		<Layout pageTitle="About Me">
			<p>Hi there! I'm Richard.</p>
		</Layout>
	)
}
```

### Deploying Using GitHub Actions

Just like with Hugo, there are pre-written GitHub Actions for deploying a Gatsby site to GitHub Pages, and it works great.

In short, you can use GitHub Actions to run `npm run build` which generates the static content for your site in the `./public` directory, and then deploy that to GitHub Pages using the `peaceiris/actions-gh-pages@v4` GitHub Action.

Something like this will do the trick, in `.github/workflows/gatsby.yml`:

```yaml
name: Deploy Gatsby site to GitHub Pages

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm install

      - name: Build Gatsby site
        run: npm run build

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public

```

Now, the Gatsby documentation doesn't seem to mention any hosting providers other than using Netlify, but as I say, you can host it anywhere you want.  Since Netlify's acquisition of Gatsby in 2023, they seem to be strongly pushing Netlify as _the place_ to host your Gatsby sites.

### A Note On Disk Space

Sometimes, the node ecosystem can lead to bloated disk space usage due to the number of packages downloaded to `node_modules`. Unfortunately, Gatsby is no exception to this, with my hello world example site weighing in at 629 MB, of which 567 MB is contained in the `node_modules` directory. Compare this with my hello world Hugo site from the previous post, which totals just 12 MB. For some with relatively modest storage capacity, this could be a real consideration when deciding which static site generator to use.

### Summary

Gatsby feels super powerful, but users without a decent grounding in React might find the learning curve to be pretty steep.  In terms of its use for hosting a simple blog, it's likely overkill, particularly for the kind of thing I'd want to do.
