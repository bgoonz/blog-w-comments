# Tutorial: Make a blog with Next.js, React and Sanity

> Sometimes you just need a blog. So why not build it with something shiny like Sanity Headless CMS, React, and Next.js?

Sometimes you just need a blog. While there are loads of blogging platforms, there might be good reasons for having your blog content live along with your other content, be it documentation (as in our case), products, a portfolio or what have you. The content model, or the data schema, for a blog is also an easy place to get started with making something headless with Sanity and a detached frontend.

In this tutorial we'll make a blog with Sanity as the content backend and the React-based framework Next.js for rendering web pages.

If you don't feel like typing all the below, you can also:

[Check out the code on Github](https://github.com/sanity-io/sanity-blog-tutorial)

## [](#efb2b8be790b)0\. Create a project folder and a monorepo

In this project you will have two separate web apps:

1.  Sanity Studio – a React app that connects to the hosted API with all your blog content
2.  The blog frontend - a website built with Next.js

It can be useful to keep these codebases in the same folder and git repository (but you don't have to). Create a folder structure like this:

You can also place `.gitignore`, `.editorconfig`, or other config files in the root, as well as a `README.md`. If you want to track the project with git, run the command `git init` i the root folder in your terminal (or add the folder to your Git GUI tool of choice).

## [](#6122154b45e4)1\. Install Sanity and the preconfigured blog schemas

If you haven't already done so, install the Sanity Command Line (CLI) tooling with node package manager ([how to install npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)).

This allows you to run the `sanity init` command in your studio folder, and indeed, this is the next step. You'll be asked to create an account with your Google or Github login, or you can create a new account directly with Sanity.io. Afterwards you can create **a new project**, upon where you'll be asked to choose a project template. Select the **blog schema template**. First though, you'll need to give your project and dataset a name (you can add more datasets if you need one for testing) and choose the path to your studio folder.

When the installation is done, you can run `sanity start` inside the studio to launch the Studio where you can start editing your content. This content will be instantly synced to the cloud and available through the public APIs once you hit publish. By running `sanity deploy` you'll upload the studio and make it available on the web for those with access (you can add users by navigating to [manage.sanity.io](https://manage.sanity.io/)).

There's a lot you can do with the schemas now stored in your studio folder under `schemas/schema.js`, but that's for another tutorial. For now, we just want our blog up and running!

## [](#c5c3383956b1)2\. Install Next.js and get it running

[Next.js](https://github.com/zeit/next.js/) (by the industrious company [Zeit](https://zeit.co/)) comes with a neat setup for making React-based webpages that has server rendering on the first request, and lots of other useful features. If you are used to React or have tried out [create-react-app](https://github.com/facebook/create-react-app), it shouldn't be too hard to get started with. There is an excellent tutorial[](https://nextjs.org/learn/basics/getting-started/setup) that goes a bit deeper on [nextjs.org](https://nextjs.org/), but you should be able to tag along with this for now.

Run `npm init -y` in your web folder to create a package.json file for your project. After, install your Next.js dependencies with

and add the following to your package.json:

Now your package.json should look similar to this:

Next.js does routing out of the box based on where you plonk down files on your filesystem. So if you add a folder called `pages` and add to it `index.js` it will become the front page of your site. Likewise, if you add `about.js` in `/pages`, this will show up on [`localhost:3000/about`](http://localhost:3000/about) once you spin up the project. Just to make sure you have everything in place, try to add the following code to `pages/index.js`, and type `npm run dev` in your shell:

Now you should have a greeting to the world if you head to [`localhost:3000`](http://localhost:3000/) in your browser.

## [](#9e1fb1526b04)3\. Make a dynamic page template

So far so good, but now for the interesting part: Let’s fetch some content from Sanity and render it in React. Quit the next dev server with `ctrl + c`. Begin by installing the necessary dependencies for connecting to the Sanity API: `npm install @sanity/client`. Create a new file called `client.js` in the root frontend folder. Open the file and put in the following:

The `projectId` and `dataset`\-name should be the same as those you'll find in your `sanity.json` file in the studio folder. Now you can import this client when you need to fetch some content from your Sanity project.

Adding a new file for every new blog entry would be impractical. A hassle even. So let's make a page template that makes it possible for us to use the URL slugs from Sanity. Version 9 of Next.js introduced dynamic routing out for the box, where you had to use a custom server on earlier versions.

Add a `post.js` file to your pages folder as well, it should now look like this:

Open `post.js` and add the following code to it:

If you go to `localhost:3000/post?slug=whatever` you should now see “whatever” printed as an H1 on the page.

Wouldn't it be neat to have a bit nicer URLs? Next.js also lets you do clean URLs with dynamic routing. First, we have to add a folder called `post` inside of our `pages` folder, move your `post.js` file inside of the `post` folder and rename it to `[slug].js` – yes, _with_ the square brackets. Now you can go to `localhost:3000/post/whatever` and see the same result as before: "whatever" printer as an H1.

## [](#c69ec037032f)4\. Get some content from Sanity

If you haven't already, this is the point you should go back to your studio folder, make sure that sanity start is running and add a new post with some content in it. Create a post with the title "Hello world!" and hit **Generate** for the slug. Remember to hit **publish** (the blue button on the bottom where you edit your post) to make the content available in the public API.

If you greet the world, the world will eventually greet you back

Next.js comes with a special function called `getInitialProps` that is called and returns props to the react component before rendering the templates in `/pages`. This is a perfect place for fetching the data you want for a page.

We have now set up Next.js with a template for the front page (index.js), and a dynamic routing that makes it possible for the `[slug].js` template to take a slug under `/post/` as a query. Now the fun part begins, let's add some Sanity to the mix:

We're using async/await since we're doing an asynchronous API-call as it makes the code a bit easier to follow. We have also removed the router since the function for `getInitialProps` gets the same information in `context`. `client.fetch()` takes two arguments: [a query](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/docs/how-queries-work) and an object with parameters and values.

In order to allow the frontend server to actually get data from Sanity, we have to add its domain to [CORS-settings](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/docs/cors). In other words, we have to add `localhost:3000` (and eventually the domain you're hosting your blog on) to Sanity’s CORS origin settings. If you enter `sanity manage` into your shell you'll be taken to the project’s settings in your browser. Navigate to settings and add `http://localhost:3000` as a new origin.

Go to [http://localhost:3000/post/hello-world](http://localhost:3000/post/hello-world) and confirm that the H1 spells “Hello world!” (If it doesn't work, make sure that the slug field in the Studio says **hello-world**, and that the post is published).

You have now successfully connected your frontend with Sanity. 🎉

## [](#2dac7329b33e)5\. Add a byline with author and categories

In the Studio you'll discover, that you can add entries for authors and categories. Go and add at least one author with an image.

Yours truly

Go back to your blog post, and attach this author in the Author field, like this:

Publish the changes. What we have just done is to _reference_ an author from the blog post. References are a powerful part of Sanity and make it possible to connect and reuse content across types. If _inspect_ your block document (`ctrl+alt+i` on Windows, or `ctrl+opt+i` on macOS) in the Studio you'll see that the object looks something like this:

This is the content we would get if we now just pulled out the author-variable (`const { title, author } = await client.fetch('*[slug.current == $slug][0]',{ slug })`), which is not very useful to us in this case. This is where _projections_ in GROQ comes handy. [Projections is a powerful feature of GROQ](https://www.sanity.io/docs/data-store/query-cheat-sheet#object-projections) and allow us to specify the API-response to our needs. Head back to editor and add the projection `{title, "name": author->name}` right after the filter (`*[_type == "post" && slug.current == $slug][0]`):

Here I have added the projection `{title, "name": author->name}` to our query. Here we specify what in the document I want to be returned in the API call. We need to make a key for the author name, and follow the reference to the name-property on the author document with an arrow `->`. In other words, we ask Sanity to follow the id under `_ref`, and return just the value for the variable called `name` from that document.

Let's try to do the same with categories. First, create at least two categories in the Sanity Studio, remember to publish them as well. I added one for _Frontend_ and one for _Web_.

Add the categories to your blog post

Now we have an array of references to categories in our blog post. If you take a peek in the document inspector you'll see that these show up just as the author entry, objects with a `_ref`\-id. So we have to use projections to get those as well. Now the GROQ query has grown, so we'll change it to a variable, and add the npm package [groq](https://www.npmjs.com/package/groq) (`npm install groq`) to get support for [syntax highlighting](https://github.com/sanity-io/vscode-sanity) (in VS Code).

The projection for categories is done similarly to the author reference, the only difference is that we attach square brackets to the key `categories` because it is an array of references. So `categories[]->title` means "loop through all the entries in categories and returns the title from the referenced document".

But we want to add the photo of the author to the byline as well! Images and file assets in Sanity are also references, which means that if we are to get the author image we first have to follow the reference to the author document, and to the image asset. We could retrieve the imageUrl directly by accessing `"imageUrl": author->image.asset->url`, but this where it's easier to use the [image url-package](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/docs/presenting-images) we've made. Install the package in the frontend-project with `npm i @sanity/image-url`. It takes the image object and figures out where to get the image, it makes it easier to use the [hot spot-features](https://www.sanity.io/docs/front-ends/presenting-images#the-crop-and-hot-spot) and so on as well.

Having put in the code lines for the image URL-builder, we can send in the image-object from Sanity in the `urlFor()` function, and append the different methods (e.g. `.width(50)`) with the `.url()`\-method at the end.

## [](#277554f83305)6\. Add rich text content with Portable Text

A blog wouldn't be much without great support for text content. Rich text in Sanity is stored as [Portable Text](https://www.portabletext.org/) which lets us use it many different contexts: from HTML in the browser to speech fulfillments in [voice interfaces](https://www.smashingmagazine.com/2019/03/sanity-portabletext-speech-synthesis/). There's a lot to be said about Portable Text and the extensibility of it, but in this tutorial, we'll just use the out-of-the-box features that come with the package [block-content-to-react](https://github.com/sanity-io/block-content-to-react). Install it with `npm install @sanity/block-content-to-react`.

We import the react-component as `BlockContent`, and get the body from the post-document. We send in the body as `blocks`\-prop, and add `{...client.config()}` in order to let the `BlockContent` component know where (using the `projectId` and the `dataset` name) to get the images that might come in the rich text field.

We also added a prop called `imageOptions`, which controls the default output of images. And that's it! You can [customize the output of different elements, and even add your own custom block-types](https://github.com/sanity-io/block-content-to-react) by sending in what we have called “serializers” – those we'll cover in another blog post.

If everything went well, you should have a bare-bones blog template in place

## [](#1da9df69a1f7)7\. Add the blog posts to the frontpage

Now we want to have a simple list of your blog posts on the frontpage, that is, in `index.js`. We have already been through most of the requirements using the `client` to fetch data from Sanity, and `getInitialProps` to make the data available in the frontend template.

If you look closer at the GROQ query in `Index.getInitialProps` you'll notice that we look for documents that have `_type == "post"` and has a publish date that is lesser (i.e. before) whatever time it is at the moment. We also order the results after the publishing data, so that the newest posts end up first in the list.

In the template we loop (`map`) over the array of posts, and build the links using [Next.js Link component](https://nextjs.org/learn/basics/clean-urls-with-dynamic-routing/dynamic-routing). We point the `href` to the file path, and use `as` for the URL-slug.

## [](#3085b10bbadd)Deploy your blog on the web with `now`

To get your new blog on the web you can use Zeit’s now service. Install the CLI with `npm i -g now` and run the command `now login` to create an account. Once that's done, you can run `now` in the webfolder to deploy the blog. Remember to add your URL to the CORS origin settings. You can also connect a [custom domain to your deployment on `now`](https://zeit.co/docs/v2/custom-domains/) (remember CORS for that too).

And that's it for this tutorial! We have now covered a lot of ground when it comes to coding a frontend layer for a pretty common content setup, and yet just scraped the iceberg of features and nifty things we can do with the combination of Sanity and React. Of course, your work doesn't end here. Now you should make this blog your own with more HTML, CSS, and JavaScript.

You can download the [example project from GitHub](https://github.com/sanity-io/sanity-blog-tutorial), and feel free asking us questions on [](https://gitter.im/sanity-io/sanity)[Slack](https://slack.sanity.io/), or however else you might find us.

[Source](https://www.sanity.io/blog/build-your-own-blog-with-sanity-and-next-js?utm_source=readme)
