Blogging with Sanity and Next.js
Read the tutorial

Get started
# Install the Sanity command line interface
~/
> npm i -g @sanity/cli

# Initiate your own project in the studio folder
~/this-blog/studio
> sanity init

# Add a CORS-origin rule to allow the frontend to request data
~/this-blog/studio
> sanity cors add http://localhost:3000 --no-credentials

# Insert the projectId and dataset name from Sanity in client.js
~/this-blog/web
> nano client.js

# Install frontend dependencies
~/this-blog/web
> npm install

# Run Next.js in development mode
~/this-blog/web
> npm run dev
Deploy on now
~/this-blog/web
> npm i -g now
> now login
> now
Deploy as a static site on Netlify
Read the tutorial

~/this-blog/web
npm run export
# exports your site as static files in /out