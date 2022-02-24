# hbs-express
This project represents an attempt at creating an express app that
makes use of nodemon for server restarting, livereload for browser
refreshing, and Edge DevTools for CSS mirroring and client javascript
debugging in a VS Code environment.

The Microsoft Edge DevTools do the work of CSS mirrorming and client
javascript debugging, but there were configuration issues to figure
out when making this work in an express app.  The key detail being
that static resources are normally in a sub-directory in an express
app.  By default the tools would either not mirror the css changes
or not hit breakpoints.

I've used nodemon to restart my app on changes to server code and I
have used the VS Code Live Server extension to trigger browser refreshes,
but I hadn't used them together to reduce the number of manual refreshes
and restarts.  The problem being that Live Server is, well, a server,
and so is an express app.  So you can't use the two of them together
without a little effort.  I was inspired by an article written by 
[Panu Pitkamaki](https://bytearcher.com/contact/) wherein he explains
the steps needed to 
[_Refresh front and backend changes to browser with Express, LiveReload
and Nodemon_](https://bytearcher.com/articles/refresh-changes-browser-express-livereload-nodemon/).

To figure out how live reload worked and sort out configuration issues I also
consulted the
[node-livereload](https://github.com/napcs/node-livereload#option-2-from-within-your-own-project)
docs.  For working out nodemon it was helpful to look at the 
[nodemon](https://github.com/remy/nodemon#readme) docs.

The basic app was generated using express-generator:
```
npx express-generator --hbs tmp
```

I integrated the livereload functionality with this bit of code added to the `app.js` file:
```
/* for development purposes */
if (process.env.NODE_ENV === "development") {
  const livereload = require("livereload");
  const liveReloadServer = livereload.createServer({ debug: false, extraExts: ["hbs"] });
  liveReloadServer.watch(path.join(__dirname, 'public'));
  liveReloadServer.watch(path.join(__dirname, 'views'));

  liveReloadServer.server.once("connection", () => {
    setTimeout(() => {
      liveReloadServer.refresh("/");
    }, 100);
  });
}
```
This sets up the livereload server in-process and configures it to watch the public and views
subdirectories.  The `public` directory has static resources for the express app.
The `views` directory has the handlebars templating resources.  By adding `.hbs` as an
extra extension, livereload will do its thing when the template files change or when
any of the static resources are updated.

For any of that server code to work, the client needs its own bit of code.  To inject that
client side javascript code there needs to be a little middleware addition:
```
if (process.env.NODE_ENV === "development") {
  app.use(require('connect-livereload')());
}
```
I included this ahead of the static middleware and routing setup.  With these changes to the 
default handlebars express app, the browser will be refreshed if the static resources or 
templates change.  The rest of the work is configuring nodemon, creating some launch
configurations, and setting up paths so that Edge DevTools knows where to find javascript
and css.

For nodemon config I opted to include it in my package.json file.  The public and views
directories are ignored because changes there don't require a server restart.  As you may
recall, those directories are watched by the liveserver code.  Any changes simply trigger
a browser refresh.
```
"nodemonConfig": {
    "verbose": true,
    "ignore": [
      "public",
      "views"
    ]
}
```
  
On the launch configuration front there are two things to notice.  First, Edge is running
headless — not a requirement, but I thought it worked nicely.  The second thing is you
need to update Edge's `webRoot` setting.  I appended `/public` because that is where the
static resources are by default in a express-generator project.  Don't include that bit,
and you can't set/hit breakpoints.  Here's the config:
```
{
  "name": "Launch Edge",
  "request": "launch",
  "type": "pwa-msedge",
  "url": "http://localhost:3000",
  "webRoot": "${workspaceFolder}/public",
  "runtimeArgs": [
      "--headless"
  ],
}
```
To launch the sever side, you can run `npx nodemon ./bin/www` on the command line, 
or you can add it to the scripts section of your package.json file, which is what
I did:
```
"scripts": {
    "start": "node ./bin/www",
    "watch": "npx nodemon ./bin/www"
},
```
Adding it to the scripts section means you can run `npm run watch` on the command line.
Okay, not much of an improvement.  Head back over to launch.json and add another
configuration entry to make use of this script:
```
{
    "name": "Watch Express",
    "command": "npm run watch",
    "request": "launch",
    "type": "node-terminal",
    "env": {
        "NODE_ENV": "development"
    }
},
````
So by now, if everything worked as expected, you can launch your server under nodemon
and it will refresh the browser if you change a template or other static resource.
Furthermore, if you update any server code, nodemon will restart your app.  Debugging
client-side javascript should work, and the only thing that probably won't work is
CSS mirroring.  That's the Edge DevTools feature where you edit your CSS in the
DevTools inspector and it updates the css file in your project automatically.  But
in order for it to do that, the DevTools need to know that
`http://localhost:8080/stylesheets/style.css` is actually under `public/stylesheets/style.css`.
Otherwise you end up with an error like this one:
<img width="462" alt="155553542-f2c8b434-ee28-4481-b54b-e78252b3a7b5" src="https://user-images.githubusercontent.com/94078897/155611262-0773975d-b14a-48b4-8a9c-602a1f9b07e3.png">

The fix is to set up the Edge DevTools `pathMapping` for your workspace:
```
"vscode-edge-devtools.pathMapping": {
    "/": "${workspaceFolder}/public"
},
```

With all that in place, you can now set breakpoints, mirror css edits, reload your browser
automatically, and restart the server when code changes.  Hope you find this useful.  I also
hope you reach out to me with any corrections, advice, or questions.








