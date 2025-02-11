# Configuration File

The configuration is an XML file.  There is no XSD for this currently.  The following specifies the supported tags.

## Root Node: `<server-config>`

Root tag for the config file.

Contains

- [`<server>`](#server)

Example.

    <server-config xmlns="http://couling.me/webdavd">
        <server><listen><port>80</port></listen></server>
    </server-config>



## `<server>`
webdavd will start one new instance for every `<server>` tag.  Typical configurations have just one of these per configuration file.

Contains

- [`<listen>`](#listen)
- [`<session-timeout>`](#session-timeout)
- [`<mime-file>`](#mime-file)
- [`<rap-binary>`](#rap-binary)
- [`<rap-timeout>`](#rap-timeout)
- [`<pam-service>`](#pam-service)
- [`<pgsql-host>`](#pgsql-host)
- [`<pgsql-port>`](#pgsql-port)
- [`<pgsql-database>`](#pgsql-database)
- [`<pgsql-user>`](#pgsql-user)
- [`<pgsql-password>`](#pgsql-password)
- [`<static-response-dir>`](#static-response-dir)
- [`<max-lock-time>`](#max-lock-time)
- [`<error-log>`](#error-log)
- [`<access-log>`](#access-log)
- [`<ssl-cert>`](#ssl-cert)

Example

    <server-config xmlns="http://couling.me/webdavd">
        <server>
            <listen>
		<port>80</port>
	    </listen>
        </server>
    </server-config>

## `<listen>`
Sets up a listening socket.  There must be at least one of these for the server to be active.  There are no default listening sockets.

Contains

- `<port>` - the port number to listen on.  Defaults to:
   - `80` if encryption is not enabled (http)
   - `443` if encryption is enabled (https)
 - `<host>` - the hostname or IP to bind the socket to.  This is NOT a virtual host.  It locks the socket down to a single IP.  eg: `<host>localhost</host>` forces this `listen` to only listen on `127.0.0.1`.  The default listens to any IP.
 - `<encryption>`  Enables or disables encryption.  Note that if any socket has ssl enabled then you MUST specify at least one certificate using [`<ssl-cert>`](#ssl-cert)
   - `none` - the port is not encrypted (https)
   - `ssl` - the port is encrypted (http)
 - [`<forward-to>`](#forward-to)

Example - A basic server might be configured as follows.  The server will listen both on 80 (http) and 443 (https).  But port 80 will simply forward clients to port 443.  This means that users always use https.  Users who accidentally type "http" will be automatically corrected.

    <server-config xmlns="http://couling.me/webdavd">
        <server>
            <listen>
                <port>80</port>
				<encryption>none</encryption>
				<forward-to><port>443</port></forward-to>
            </listen>
			<listen>
				<port>443</port>
				<encryption>ssl</encryption>
			</listen>
			<ssl-cert>
				<certificate>/etc/ssl/certs/local/server.crt</certificate>
				<key>/etc/ssl/private/server.key</key>
			</ssl-cert>
        </server>
    </server-config>

## `<forward-to>`
Sets a listening socket to be a http forwarding agent.  No content will be served from this port and no client authentication will be carried out.  All requests will be forwarded to a derivative of the specified forwarding address.

Contains

Note that tags contained within a `<forward-to>` are very similar to `<listen>` but are interpreted in a subtally different way.  Read carefully!

 - `<port>` - the port number to listen on.  Note that if forwarding to 80 or 443 with the encryption, the port number will not be sent to the client as the client should infer it for itself.
 - `<host>` - the hostname or IP to forward to.  By default the server picks whatever domain name the client specified in its request.  It's better to leave this default unless you specifically need to bounce to a different hostname.
 - `<encryption>` selects http or https
   - none specifies no encryption (http)
   - ssl specifies encryption (https)

Example - Set up the server to simply be a forwarding agent to `https://example.com:4430/`

    <server-config xmlns="http://couling.me/webdavd">
        <server>
			<listen>
				<port>80</port>
				<forward-to>
					<port>4430</port>
					<encryption>ssl</encryption>
					<host>example.com</host>
				</forward-to>
			</listen>
        </server>
    </server-config>

## `<session-timeout>`
Specifies the length of PAM session to be used by the server.  All sessions will be this long regardless of activity.  webdavd will continue to re-use PAM sessions for multiple requests across multiple clients as long as they use the same username and password.  This prevents rapid requests from hammering PAM.  If a user password changes while the session is open the user will be able to acccess webdavd with BOTH the new password and old password until the old session expires.  Default is `5:00` (5 minutes). See [Time Format](#Time Format)

Example - Keep PAM sessions open for an hour

    <server-config xmlns="http://couling.me/webdavd">
        <server>
		<listen><
			port>80</port>
		</listen>
        	<session-timeout>01:00:00</session-timeout>
	</server>
    </server-config>


## `<mime-file>`
To identify mime types from file extensions webdavd needs a `mime.types` file.  By default most systems have this stored in `/etc/mime.types`.  If you wish to use a customized file then specify the file location here.

Example - Uses an alternative mime file: `/usr/share/alternate/mime.types`

    <server-config xmlns="http://couling.me/webdavd">
        <server>
		<listen>
			<port>80</port>
		</listen>
        	<mime-file>/usr/share/alternate/mime.types</mime-file>
	</server>
    </server-config>

## `<rap-binary>`
webdavd is a binary in two parts.  The worker threads run a binary called the "rap".  This defaults to `/usr/lib/webdav/webdav-worker`.  If on your system the "rap" binary has a different file name then specify it here.

Example - Specifies the worker thread binary (rap) as being `/usr/lib/webdav/webdav-worker`

    <server-config xmlns="http://couling.me/webdavd">
        <server>
		<listen>
			<port>80</port>
		</listen>
        	<rap-binary>/usr/sbin/webdav-worker</rap-binary>
	</server>
    </server-config>

## `<rap-timeout>`
Communication with the worker threads should be rapid.  There are no long operations performed by the worker that should leave the master waiting a long time.  By default the operation will fail after 2 minutes and the worker will be killed.  See [time format](#Time Format)

Example

    <server-config xmlns="http://couling.me/webdavd">
        <server>
		<listen>
			<port>80</port>
		</listen>
        	<rap-timeout>/usr/sbin/webdav-worker</rap-timeout>
	</server>
    </server-config>

## `<pam-service>`
The service name used to configure PAM.  This is `webdavd` by default.  On many GNU / linux systems the service name specifies the file name in `/etc/pam.d/`  on other systems PAM services are configured in a single file.  Please consult the PAM documentation for your operating system for further details.

Example

    <server-config xmlns="http://couling.me/webdavd">
        <server>
		<listen>
			<port>80</port>
		</listen>
        	<pam-service>dav</pam-service>
	</server>
    </server-config>

## `<pgsql-host>`
The hostname or the IP address to connect the Postgresql database. Default to localhost.

Example

    <server-config xmlns="http://couling.me/webdavd">
        <server>
		<listen>
			<port>80</port>
		</listen>
        	<pgsql-host>myownserver.com</pgsql-host>
	</server>
    </server-config>

## `<pgsql-port>`
The port to connect to the Postgresql database. Default to 5432.

Example

    <server-config xmlns="http://couling.me/webdavd">
        <server>
		<listen>
			<port>80</port>
		</listen>
        	<pgsql-port>5433</pgsql-port>
	</server>
    </server-config>

## `<pgsql-database>`
The database name to connect to that contains users table to authenticate the web server. Default to postgres.

Example

    <server-config xmlns="http://couling.me/webdavd">
        <server>
		<listen>
			<port>80</port>
		</listen>
        	<pgsql-database>my_database</pgsql-database>
	</server>
    </server-config>

## `<pgsql-user>`
The user to connect to the Postgresql database. Default to postgres.

Example

    <server-config xmlns="http://couling.me/webdavd">
        <server>
		<listen>
			<port>80</port>
		</listen>
        	<pgsql-user>my_user</pgsql-user>
	</server>
    </server-config>

## `<pgsql-password>`
The password to connect to the Postgresql database. There is no default value. Note that the password must be in clear text.

Example

    <server-config xmlns="http://couling.me/webdavd">
        <server>
		<listen>
			<port>80</port>
		</listen>
        	<pgsql-password>my_password</pgsql-password>
	</server>
    </server-config>

## `<static-response-dir>`
Some error pages (eg 404) from the webdavd are static and specified separate files.  These are all stored in a single directory.  This tag specifies the location of the directory.  Default is: `/usr/share/webdav`

Example

    <server-config xmlns="http://couling.me/webdavd">
        <server>
		<listen>
			<port>80</port>
		</listen>
        	<static-response-dir>/usr/share/custom/webdav</static-response-dir>
	</server>
    </server-config>

## `<max-lock-time>`

Maximum time allowed for clients to lock a file. See [Time Format](#Time Format)

Example

    <server-config xmlns="http://couling.me/webdavd">
        <server>
		<listen>
			<port>80</port>
		</listen>
        	<max-lock-time>10:00</max-lock-time>
	</server>
    </server-config>

## `<error-log>`
The location to write the error log.  If unspecified the error log will be written to the stderr.

Example

    <server-config xmlns="http://couling.me/webdavd">
        <server>
		<listen>
			<port>80</port>
		</listen>
        	<access-log>/var/log/webdav-access.log</access-log>
        	<error-log>/var/log/webdav-error.log</error-log>
	</server>
    </server-config>

## `<access-log>`
The location to write the access log.  If unspecified the access log will be written to the stdout.

Example

    <server-config xmlns="http://couling.me/webdavd">
        <server>
		<listen>
			<port>80</port>
		</listen>
        	<access-log>/var/log/webdav-access.log</access-log>
        i	<error-log>/var/log/webdav-error.log</error-log>
	</server>
    </server-config>

## `<ssl-cert>`
Specifies a certificate to use for ssl server identification.  You can specify as many certificates as you need and the server will automatically pick the correct one for the domain name being requested.

Contains

 - `<certificate>` The certificate file identifying this server
 - `<chain>` If intermediary certificates then can be specified with this tag.  Note that if multiple `<chain>` tags are needed for a certificate then they must be specified in order.  Starting with the one closest to <`certificate>` and finishing with the one closest to the CA.
 - `<key>` The private key file for `<certificate>`

Example

    <server-config xmlns="http://couling.me/webdavd">
        <server>
			<listen>
				<port>443</port>
				<encryption>ssl</encryption>
			</listen>
			<ssl-cert>
				<certificate>/etc/ssl/certs/local/server.crt</certificate>
				<chain>/etc/ssl/certs/local/intermediate.crt</chain>
				<key>/etc/ssl/private/server.key</key>
			</ssl-cert>
        </server>
    </server-config>

## `<unprotect-options>`
Set `<unprotect-options>` to true if you would like to make OPTIONS requests available without previous authentication. This might be required for your CORS setup.

Note that this exposes the features of the server to everyone requesting them.

Example

    <server-config xmlns="http://couling.me/webdavd">
        <server>
		<listen>
			<port>80</port>
		</listen>
        	<unprotect-options>true</unprotect-options>
	</server>
    </server-config>

## Time Format
Times can be formatted as any of the following:

 - `ss` for example `15` is 15 seconds
 - `mm:ss` for example `23:01` is 23 minutes and 1 second
 - `hh:mm:ss` for example `03:20:00` is 3 hours 20 minutes and 0 seconds.

