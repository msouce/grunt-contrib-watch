# grunt-contrib-watch [![Build Status](https://travis-ci.org/gruntjs/grunt-contrib-watch.png?branch=master)](https://travis-ci.org/gruntjs/grunt-contrib-watch)

> Run predefined tasks whenever watched file patterns are added, changed or deleted.



## Getting Started
This plugin requires Grunt `~0.4.0`

If you haven't used [Grunt](http://gruntjs.com/) before, be sure to check out the [Getting Started](http://gruntjs.com/getting-started) guide, as it explains how to create a [Gruntfile](http://gruntjs.com/sample-gruntfile) as well as install and use Grunt plugins. Once you're familiar with that process, you may install this plugin with this command:

```shell
npm install grunt-contrib-watch --save-dev
```

Once the plugin has been installed, it may be enabled inside your Gruntfile with this line of JavaScript:

```js
grunt.loadNpmTasks('grunt-contrib-watch');
```




## Watch task
_Run this task with the `grunt watch` command._


### Settings

There are a number of options available. Please review the [minimatch options here](https://github.com/isaacs/minimatch#options). As well as some additional options as follows:

#### files
Type: `String|Array`

This defines what file patterns this task will watch. Can be a string or an array of files and/or minimatch patterns.

#### tasks
Type: `String|Array`

This defines which tasks to run when a watched file event occurs.

#### options.spawn
Type: `Boolean`
Default: true

Whether to spawn task runs in a child process. Setting this option to `false` speeds up the reaction time of the watch (usually 500ms faster for most) and allows subsequent task runs to share the same context. Not spawning task runs can make the watch more prone to failing so please use as needed.

Example:
```js
watch: {
  scripts: {
    files: ['**/*.js'],
    tasks: ['jshint'],
    options: {
      spawn: false,
    },
  },
},
```

*For backwards compatibility the option `nospawn` is still available and will do the opposite of `spawn`.*

#### options.interrupt
Type: `Boolean`
Default: false

As files are modified this watch task will spawn tasks in child processes. The default behavior will only spawn a new child process per target when the previous process has finished. Set the `interrupt` option to true to terminate the previous process and spawn a new one upon later changes.

Example:
```js
watch: {
  scripts: {
    files: '**/*.js',
    tasks: ['jshint'],
    options: {
      interrupt: true,
    },
  },
},
```

#### options.debounceDelay
Type: `Integer`
Default: 500

How long to wait before emitting events in succession for the same filepath and status. For example if your `Gruntfile.js` file was `changed`, a `changed` event will only fire again after the given milliseconds.

Example:
```js
watch: {
  scripts: {
    files: '**/*.js',
    tasks: ['jshint'],
    options: {
      debounceDelay: 250,
    },
  },
},
```

#### options.interval
Type: `Integer`
Default: 100

The `interval` is passed to `fs.watchFile`. Since `interval` is only used by `fs.watchFile` and this watcher also uses `fs.watch`; it is recommended to ignore this option. *Default is 100ms*.

#### options.event
Type: `String|Array`
Default: `'all'`

Specify the type watch event that trigger the specified task. This option can be one or many of: `'all'`, `'changed'`, `'added'` and `'deleted'`.

Example:
```js
watch: {
  scripts: {
    files: '**/*.js',
    tasks: ['generateFileManifest'],
    options: {
      event: ['added', 'deleted'],
    },
  },
},
```

#### options.forever
Type: `Boolean`
Default: true

This is *only a task level option* and cannot be configured per target. By default the watch task will duck punch `grunt.fatal` and `grunt.warn` to try and prevent them from exiting the watch process. If you don't want `grunt.fatal` and `grunt.warn` to be overridden set the `forever` option to `false`.

#### options.dateFormat
Type: `Function`

This is *only a task level option* and cannot be configured per target. By default when the watch has finished running tasks it will display the message `Completed in 1.301s at Thu Jul 18 2013 14:58:21 GMT-0700 (PDT) - Waiting...`. You can override this message by supplying your own function:

```js
watch: {
  options: {
    dateFormat: function(time) {
      grunt.log.writeln('The watch finished in ' + time + 'ms at' + (new Date()).toString());
      grunt.log.writeln('Waiting for more changes...');
    },
  },
  scripts: {
    files: '**/*.js',
    tasks: 'jshint',
  },
},
```

#### options.atBegin
Type: `Boolean`
Default: false

This option will trigger the run of each specified task at startup of the watcher.

#### options.livereload
Type: `Boolean|Number|Object`
Default: false

Set to `true` or set `livereload: 1337` to a port number to enable live reloading. Default and recommended port is `35729`.

If enabled a live reload server will be started with the watch task per target. Then after the indicated tasks have ran, the live reload server will be triggered with the modified files.

Example:
```js
watch: {
  css: {
    files: '**/*.sass',
    tasks: ['sass'],
    options: {
      livereload: true,
    },
  },
},
```

It's possible to get livereload working over https connections. To do this, pass an object to `livereload` with a `key` and `cert` paths specified.

Example:
```js
watch: {
  css: {
    files: '**/*.sass',
    tasks: ['sass'],
    options: {
      livereload: {
        port: 9000,
        key: 'path/to/ssl.key',
        cert: 'path/to/ssl.crt'
      }
    },
  },
},
```


#### options.cwd
Type: `String|Object`
Default: `process.cwd()`

Ability to set the current working directory. Defaults to `process.cwd()`. Can either be a string to set the cwd to match files and spawn tasks. Or an object to set each independently. Such as `options: { cwd: { files: 'match/files/from/here', spawn: 'but/spawn/files/from/here' } }`.

### Examples

```js
// Simple config to run jshint any time a file is added, changed or deleted
grunt.initConfig({
  watch: {
    files: ['**/*'],
    tasks: ['jshint'],
  },
});
```

```js
// Advanced config. Run specific tasks when specific files are added, changed or deleted.
grunt.initConfig({
  watch: {
    gruntfile: {
      files: 'Gruntfile.js',
      tasks: ['jshint:gruntfile'],
    },
    src: {
      files: ['lib/*.js', 'css/**/*.scss', '!lib/dontwatch.js'],
      tasks: ['default'],
    },
    test: {
      files: '<%= jshint.test.src %>',
      tasks: ['jshint:test', 'qunit'],
    },
  },
});
```

#### Using the `watch` event
This task will emit a `watch` event when watched files are modified. This is useful if you would like a simple notification when files are edited or if you're using this task in tandem with another task. Here is a simple example using the `watch` event:

```js
grunt.initConfig({
  watch: {
    scripts: {
      files: ['lib/*.js'],
    },
  },
});
grunt.event.on('watch', function(action, filepath, target) {
  grunt.log.writeln(target + ': ' + filepath + ' has ' + action);
});
```

**The `watch` event is not intended for replacing the standard Grunt API for configuring and running tasks. If you're trying to run tasks from within the `watch` event you're more than likely doing it wrong. Please read [configuring tasks](http://gruntjs.com/configuring-tasks).**

##### Compiling Files As Needed
A very common request is to only compile files as needed. Here is an example that will only lint changed files with the `jshint` task:

```js
grunt.initConfig({
  watch: {
    scripts: {
      files: ['lib/*.js'],
      tasks: ['jshint'],
      options: {
        spawn: false,
      },
    },
  },
  jshint: {
    all: {
      src: ['lib/*.js'],
    },
  },
});

// on watch events configure jshint:all to only run on changed file
grunt.event.on('watch', function(action, filepath) {
  grunt.config('jshint.all.src', filepath);
});
```

If you need to dynamically modify your config, the `spawn` option must be disabled to keep the watch running under the same context.

If you save multiple files simultaneously you may opt for a more robust method:

```js
var changedFiles = Object.create(null);
var onChange = grunt.util._.debounce(function() {
  grunt.config('jshint.all.src', Object.keys(changedFiles));
  changedFiles = Object.create(null);
}, 200);
grunt.event.on('watch', function(action, filepath) {
  changedFiles[filepath] = action;
  onChange();
});
```

#### Live Reloading
Live reloading is built into the watch task. Set the option `livereload` to `true` to enable on the default port `35729` or set to a custom port: `livereload: 1337`.

The simplest way to add live reloading to all your watch targets is by setting `livereload` to `true` at the task level. This will run a single live reload server and trigger the live reload for all your watch targets:

```js
grunt.initConfig({
  watch: {
    options: {
      livereload: true,
    },
    css: {
      files: ['public/scss/*.scss'],
      tasks: ['compass'],
    },
  },
});
```

You can also configure live reload for individual watch targets or run multiple live reload servers. Just be sure if you're starting multiple servers they operate on different ports:

```js
grunt.initConfig({
  watch: {
    css: {
      files: ['public/scss/*.scss'],
      tasks: ['compass'],
      options: {
        // Start a live reload server on the default port 35729
        livereload: true,
      },
    },
    another: {
      files: ['lib/*.js'],
      tasks: ['anothertask'],
      options: {
        // Start another live reload server on port 1337
        livereload: 1337,
      },
    },
    dont: {
      files: ['other/stuff/*'],
      tasks: ['dostuff'],
    },
  },
});
```

##### Enabling Live Reload in Your HTML
Once you've started a live reload server you'll be able to access the live reload script. To enable live reload on your page, add a script tag before your closing `</body>` tag pointing to the `livereload.js` script:

```html
<script src="//localhost:35729/livereload.js"></script>
```

Feel free to add this script to your template situation and toggle with some sort of `dev` flag.

##### Using Live Reload with the Browser Extension
Instead of adding a script tag to your page, you can live reload your page by installing a browser extension. Please visit [how do I install and use the browser extensions](http://feedback.livereload.com/knowledgebase/articles/86242-how-do-i-install-and-use-the-browser-extensions-) for help installing an extension for your browser.

Once installed please use the default live reload port `35729` and the browser extension will automatically reload your page without needing the `<script>` tag.

##### Using Connect Middleware
Since live reloading is used when developing, you may want to disable building for production (and are not using the browser extension). One method is to use Connect middleware to inject the script tag into your page. Try the [connect-livereload](https://github.com/intesso/connect-livereload) middleware for injecting the live reload script into your page.

##### Rolling Your Own Live Reload
Live reloading is made easy by the library [tiny-lr](https://github.com/mklabs/tiny-lr). It is encouraged to read the documentation for `tiny-lr`. If you would like to trigger the live reload server yourself, simply POST files to the URL: `http://localhost:35729/changed`. Or if you rather roll your own live reload implementation use the following example:

```js
// Create a live reload server instance
var lrserver = require('tiny-lr')();

// Listen on port 35729
lrserver.listen(35729, function(err) { console.log('LR Server Started'); });

// Then later trigger files or POST to localhost:35729/changed
lrserver.changed({body:{files:['public/css/changed.css']}});
```

##### Live Reload with Preprocessors
Any time a watched file is edited with the `livereload` option enabled, the file will be sent to the live reload server. Some edited files you may desire to have sent to the live reload server, such as when preprocessing (`sass`, `less`, `coffeescript`, etc). As any file not recognized will reload the entire page as opposed to just the `css` or `javascript`.

The solution is to point a `livereload` watch target to your destination files:

```js
grunt.initConfig({
  sass: {
    dev: {
      src: ['src/sass/*.sass'],
      dest: 'dest/css/index.css',
    },
  },
  watch: {
    sass: {
      // We watch and compile sass files as normal but don't live reload here
      files: ['src/sass/*.sass'],
      tasks: ['sass'],
    },
    livereload: {
      // Here we watch the files the sass task will compile to
      // These files are sent to the live reload server after sass compiles to them
      options: { livereload: true },
      files: ['dest/**/*'],
    },
  },
});
```

### FAQs

#### How do I fix the error `EMFILE: Too many opened files.`?
This is because of your system's max opened file limit. For OSX the default is very low (256). Temporarily increase your limit with `ulimit -n 10480`, the number being the new max limit.

In some versions of OSX the above solution doesn't work. In that case try `launchctl limit maxfiles 10480 10480 ` and restart your terminal. See [here](http://superuser.com/questions/261023/how-to-change-default-ulimit-values-in-mac-os-x-10-6).

#### Can I use this with Grunt v0.3?
`grunt-contrib-watch@0.1.x` is compatible with Grunt v0.3 but it is highly recommended to upgrade Grunt instead.

#### Why is the watch devouring all my memory/cpu?
Likely because of an enthusiastic pattern trying to watch thousands of files. Such as `'**/*.js'` but forgetting to exclude the `node_modules` folder with `'!**/node_modules/**'`. Try grouping your files within a subfolder or be more explicit with your file matching pattern.

Another reason if you're watching a large number of files could be the low default `interval`. Try increasing with `options: { interval: 5007 }`. Please see issues [#35](https://github.com/gruntjs/grunt-contrib-watch/issues/145) and [#145](https://github.com/gruntjs/grunt-contrib-watch/issues/145) for more information.

#### Why spawn as child processes as a default?
The goal of this watch task is as files are changed, run tasks as if they were triggered by the user themself. Each time a user runs `grunt` a process is spawned and tasks are ran in succession. In an effort to keep the experience consistent and continually produce expected results, this watch task spawns tasks as child processes by default.

Sandboxing task runs also allows this watch task to run more stable over long periods of time. As well as more efficiently with more complex tasks and file structures.

Spawning does cause a performance hit (usually 500ms for most environments). It also cripples tasks that rely on the watch task to share the context with each subsequent run (i.e., reload tasks). If you would like a faster watch task or need to share the context please set the `spawn` option to `false`. Just be aware that with this option enabled, the watch task is more prone to failure.


## Release History

 * 2013-08-25   v0.5.3   Fixed for live reload missing files.
 * 2013-08-16   v0.5.2   Fixed issue running tasks after gruntfile is reloaded. Ignores empty file paths.
 * 2013-07-20   v0.5.1   Fixed issue with options resetting.
 * 2013-07-18   v0.5.0   Added target name to watch event. Added atBegin option to run tasks when watcher starts. Changed nospawn option to spawn (nospawn still available for backwards compatibility). Moved libs/vars into top scope to prevent re-init. Bumped Gaze version to ~0.4. Re-grab task/target options upon each task run. Add dateFormat option to override the date/time output upon completion.
 * 2013-05-27   v0.4.4   Remove gracefully closing SIGINT. Not needed and causes problems for Windows. Ensure tasks are an array to not conflict with cliArgs.
 * 2013-05-11   v0.4.3   Only group changed files per target to send correct files to live reload.
 * 2013-05-09   v0.4.2   Fix for closing watchers.
 * 2013-05-09   v0.4.1   Removed "beep" notification. Tasks now optional with livereload option. Reverted "run again" with interrupt off to fix infinite recursion issue. Watchers now close more properly on task run.
 * 2013-05-03   v0.4.0   Option livereload to start live reload servers. Will reload a Gruntfile before running tasks if Gruntfile is modified. Option event to only trigger watch on certain events. Refactor watch task into separate task runs per target. Option forever to override grunt.fatal/warn to help keeping the watch alive with nospawn enabled. Emit a beep upon complete. Logs all watched files with verbose flag set. If interrupt is off, will run the tasks once more if watch triggered during a previous task run. tasks property is optional for use with watch event. Watchers properly closed when exiting.
 * 2013-02-28   v0.3.1   Fix for top level options.
 * 2013-02-27   v0.3.0   nospawn option added to run tasks without spawning as child processes. Watch emits 'watch' events upon files being triggered with grunt.event. Completion time in seconds and date/time shown after tasks ran. Negate file patterns fixed. Tasks debounced individually to handle simultaneous triggering for multiple targets. Errors handled better and viewable with --stack cli option. Code complexity reduced making the watch task code easier to read.
 * 2013-02-15   v0.2.0   First official release for Grunt 0.4.0.
 * 2013-01-18   v0.2.0rc7   Updating grunt/gruntplugin dependencies to rc6. Changing in-development grunt/gruntplugin dependency versions from tilde version ranges to specific versions.
 * 2013-01-09   v0.2.0rc5   Updating to work with grunt v0.4.0rc5.
 * 2012-12-15   v0.2.0a   Conversion to grunt v0.4 conventions. Remove node v0.6 and grunt v0.3 support. Allow watch task to be renamed. Use grunt.util.spawn "grunt" option. Updated to gaze@0.3.0, forceWatchMethod option removed.
 * 2012-11-01   v0.1.4   Prevent watch from spawning duplicate watch tasks
 * 2012-10-28   v0.1.3   Better method to spawn the grunt bin Bump gaze to v0.2.0. Better handles some events and new option forceWatchMethod Only support Node.js >= v0.8
 * 2012-10-17   v0.1.2   Only spawn a process per task one at a time Add interrupt option to cancel previous spawned process Grunt v0.3 compatibility changes
 * 2012-10-16   v0.1.1   Fallback to global grunt bin if local doesnt exist. Fatal if bin cannot be found Update to gaze 0.1.6
 * 2012-10-08   v0.1.0   Release watch task Remove spawn from helper Run on Grunt v0.4

---

Task submitted by [Kyle Robinson Young](http://dontkry.com)

*This file was generated on Tue Oct 15 2013 12:20:09.*
