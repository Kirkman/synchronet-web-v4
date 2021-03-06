# synchronet-web-v4
A web interface for Synchronet BBS

###Disclaimer

- Use this software at your own risk.  It's still being developed, and hasn't been thoroughly tested yet.

- This readme kind of sucks.  I'll put a better one on the Synchronet Wiki once I'm ready to bring this over to the Synchronet CVS repository.
	- Things may change quite a bit by the time I do bring this web interface over to the CVS, so try not to get too attached to any customizations that you make in the meantime.
		- However, if you're an early adopter, I would appreciate your feedback.

###Requirements

- This web interface has only been tested with Synchronet BBS 3.16c.  It will probably work with earlier and later versions.

- The *Files* page of this web interface relies on a script which was introduced *after* the release of Synchronet BBS 3.16c.  You can grab a copy of *filedir.js* [here](http://cvs.synchro.net/cgi-bin/viewcvs.cgi/*checkout*/exec/load/filedir.js?revision=1.2), and you should place it in your *exec/load/* directory.

###Quick start

I haven't actually tried these instructions on a clean Synchronet BBS installation, but they should work.

- Back up your Synchronet installation
- Shut down your BBS
- Clone [or download an archive of](https://github.com/echicken/synchronet-web-v4/archive/master.zip) this repository to a convenient location
	- Copy the contents of the downloaded *mods* directory into your local *mods* directory
	- Copy the contents of the downloaded *text* directory into your local *text* directory
	- Rename your current *web* directory to something like *web.old* and then copy the downloaded *web* directory in its place
- [Download fTelnet](https://github.com/rickparrish/fTelnet/archive/master.zip)
	- Extract the archive
	- Copy the *release* subdirectory into your new *web/root/* directory
	- Rename the copied *release* subdirectory to *ftelnet*
- Add the following section to your *ctrl/modopts.ini* file:
```ini
[web]
	; Unauthenticated visitors will be logged in as the user with this alias
	; (Only give this user privileges you want unknown web visitors to have)
	guest = Guest
	; Login sessions expire after this many seconds of inactivity
	timeout = 43200
	; Users disappear from the "Who's online" list after this many seconds
	inactivity = 900
	; Allow new users to register via the web interface
	user_registration = true
	; Enforce a minimum password length if user_registration is true
	minimum_password_length = 6
	; Limit the length of a telegram (in characters) that a web user can send
	maximum_telegram_length = 800
	; Which external program sections to list on the Games page (comma-separated)
	xtrn_sections = games,puzzle,rpg,erotic
	; Where (absolute or relative to 'exec') the 'lib' and 'root' directories live
	web_directory = ../web
	; Path to a .ans file to use as the ftelnet splash screen
	ftelnet_splash = ../text/synch.ans
	; Only load this many messages from each sub (default: 0 for all)
	; (If you get 'Out of memory' errors when viewing subs, tweak this setting)
	max_messages = 0
```
- Add the following section to your *ctrl/services.ini* file if it isn't there already:
```ini
[WebSocket]
Port=1123
Options=NO_HOST_LOOKUP
Command=websocketservice.js
```
- Add the following section to your *ctrl/services.ini* file:
```ini
[WebSocketRLogin]
Port=1513
Options=NO_HOST_LOOKUP
Command=websocket-rlogin-service.js
```
- Tell your router and firewall to open and forward ports *1123* and *1513* to your BBS
- If you were running ecWeb v3 and modified the *RootDirectory* value in the *[Web]* section of *ctrl/sbbs.ini* to point to *../web/root/ecwebv3*, change it back to *../web/root*.

- Edit the *[logon]* section of your *ctrl/modopts.ini* file, and ensure that it has an *rlogin_auto_xtrn* key with a value of *true*
	- NB: that's the *[logon]* section and not the *[login]* section

```ini
[logon]
rlogin_auto_xtrn = true
```

- Your *logon.js* file should have a block of code near the top that looks like this, but if it doesn't you should add it in:

```js
var options = load("modopts.js", "logon");

// Check if we're being asked to auto-run an external (web interface external programs section uses this)
if (options && (options.rlogin_auto_xtrn) && (bbs.sys_status & SS_RLOGIN) && (console.terminal.indexOf("xtrn=") === 0)) {
    var external_code = console.terminal.substring(5);
    if (!bbs.exec_xtrn(external_code)) {
        alert(log(LOG_ERR,"!ERROR Unable to launch external: '" + external_code + "'"));
    }
    bbs.hangup();
	exit();
}
```

- Start your BBS back up again

###Configuration

- Ensure that the *guest* user specified in the [web] section of *ctrl/modopts.ini* exists and has only the permissions that you want an unauthenticated visitor from the web to have.  This user probably shouldn't be able to post messages, and definitely shouldn't be able to post to networked message areas.
- Look at those *xtrn_sections* in the [web] section of *ctrl/modopts.ini*.  They should be the *internal codes* of all External Programs sections that you want to make available to authenticated users via the web.  (You probably don't have an *erotic* External Programs area, but if you do ... that's cool.)

###Customization

- This web interface uses [Bootstrap 3.3.5](http://getbootstrap.com/).  It should be possible to use any compatible stylesheet.
	- You can place your own CSS overrides in *web/root/css/style.css*
	- You can load another stylesheet of your own choosing in the &lt;head&gt; section of *web/root/index.xjs* (load it after the others)
- The sidebar module & page structure is *mostly* similar to the system used in ecWeb v3.  See [the old instructions](http://wiki.synchro.net/howto:ecweb#the_sidebar) for info on adding content.
	- You can force a link to a page to be placed in the *More* menu by using an underscore as a separator in its filename rather than a hyphen

###Uninstall

- To stop using this web interface, you can just revert to your previous *web* directory at any time.
- The [web] section added to *ctrl/modopts.ini* won't hurt anything if you leave it there, but you can delete it if you want
- Revert your *ctrl/services.ini* file to the backup you made prior to installing this web interface
- Undo any changes you made to your firewall & router during the *Quick Start*
