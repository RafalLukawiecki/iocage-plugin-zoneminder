# iocage-plugin-zoneminder

Artifact repo for iocage zoneminder plugin.

## SSL/TLS Settings
This version of the Zoneminder plugin allows for customisation of SSL/TLS
options. It also automatically generates a self-signed SSL certificate and a
private key during its first installation so that the initial connection can be
secured. As this is a self-signed certificate it will cause _security warnings_ to
be thrown by the browser. Ideally, you would provide your own certificate and key
using the plugin settings to avoid the warnings, as soon as possible. The
self-signed certificate will expire after 366 days.

A possible scheme to do that efficiently on a host such as FreeNAS is to copy a
valid certificate and private key file from the host's
`/etc/certificates/mycert.pem` into the plugin at
`/mnt/YourDataset/iocage/jails/YourJail/root/usr/local/etc/ssl/mycert.pem`
directory and specify `/usr/local/etc/ssl/mycert.pem` for the plug in to use. You
need to do that for the private key, too. You can set-up a Tasks/Cron Job on the
FreeNAS host to copy them regularly, especially if using an automated certificate
renewal tool like ACME LetsEncrypt. This way there is no need for the plugin to
have to manage its certificate renewal and you will achieve a reasonable level of
network security.

### Settings
All the configurable settings are configured in settings.json. They are:
- `httpport`
- `httpsport`
- `sslcert`
- `sslkey`

Assuming your plugin jail is called "zoneminder" you can set them by calling
something along the lines of:

```
iocage set -P httpport=8349 zoneminder
iocage set -P sslcert=/usr/local/etc/ssl/mycert.pem zoneminder
```

Bear in mind if you are setting the value of sslcert and sslkey you _will_ get
an error once you have set the first parameter but before the second one. This
error, which you can ignore, simply states that the just-changed file does not
match the other one. Once the key and certificate match each other all is good.
A way to avoid that would be for iocage not to call the `servicerestart` command
when setting multiple values, but it does not do that at present. Another way would
be to only make one of the two settings require a restart, but that would be
error-prone in other way.

### Admin UI issue

Please note that the Manage admin UI defaults to HTTPS on the default port (443)
because of a bug in iocage (see https://github.com/iocage/iocage/issues/1163)
that prevents the adminportal URL from being
formed correctly using custom values. Once that bug has been fixed it will be
possible to replace the ui.json with the below one to make it work automatically:

```
{
    "adminportal": "%%S%%://%%IP%%%%P%%/zm",
    "adminportal_placeholders": {
	"%%S%%": "adminprotocol",
	"%%P%%": "adminport"
    },
    "docurl": "https://github.com/RafalLukawiecki/iocage-plugin-zoneminder"
}
```
