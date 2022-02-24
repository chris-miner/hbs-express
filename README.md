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

To figure out how it worked and sort out configuration issues I also
consulted the [node-livereload](https://github.com/napcs/node-livereload#option-2-from-within-your-own-project) docs

The basic app was generated using express-generator:
```
npx express-generator --hbs tmp
```



