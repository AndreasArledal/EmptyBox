# EmptyBox
A complete isomorphic hackable blog service based on React JS

## Concept
Inspired by the Jekyll blog service. **EmptyBox** is a boilerplate ready for production. Instead of only producing a static page, you can run the service yourself. This puts **EmptyBox** in isomorphic mode. This will render the page on the server and deliver it quickly over the wire. On the client React JS takes over and adds your functionality. When the user continues to navigate your blog it will all happen in the client. This gives the best possible user experience.

## Features
- Isomorphic by default
- Write articles in Github Markdown
- Articles are automatically bundled with your application and lazy loaded
- When editing markdown files and saving the application LIVE updates the article
- Add whatever functionality you want. Simple APIs gives you access to the articles

## Getting started
1. Fork the repo and clone it to your computer
2. In project root `npm install` to install dependencies
3. `npm start` to start the service
4. Go to `localhost:3000`
5. Open the test article at `localhost:3000/articles/test.md`

## Deploying to Heroku
1. Create a Heroku account
2. Add Heroku git repo as upstream to your fork
3. When you want to push new version, push it to upstream

