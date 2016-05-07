# autossh

Persistent SSH tunnels for Node.js

### Install

Using npm

```
npm i -S autossh
```

### Usage

#### To Start

``` javascript
const autossh = require('autossh');

autossh({
  host: '111.22.333.444',
  username: 'root',
  localPort: 64444,
  remotePort: 5432
});
```

...is equivalent to...

``` bash
ssh -NL 64444:localhost:5432 -o "ExitOnForwardFailure yes" -o ServerAliveInterval=120 -o ServerAliveCountMax=1 root@111.22.333.444
```

#### Event Listeners

Autossh inherits from node.js's EventEmitter, and implements two events: `error`, `connect`

**error**

The `error` event will fire anytime there is an error throughout the life of the `autossh` process.

**connect**

The `connect` event will fire only once when the initial ssh connection is made

``` javascript
autossh({
  host: '111.22.333.444',
  username: 'root',
  localPort: 64444,
  remotePort: 5432
})
.on('error', err => {
  console.error('ERROR: ', err);
})
.on('connect', connection => {
  console.log('Tunnel established on port ' + connection.localPort);
  console.log('pid: ' + connection.pid);
});
```

#### Generate Dynamic Local Port

If you want to dynamically/randomly generate a port number, provide a string `auto` for the `localPort`.

The major benefit is that port conflicts will automatically be avoided--the generated port will not have been in use.

The generated `localPort` can be accessed from the connection object as `localPort`.

``` javascript
autossh({
  host: '111.22.333.444',
  username: 'root',
  localPort: 'auto',
  remotePort: 5432
})
.on('connect', connection => {
  console.log('connected: ', connection);
  console.log('localPort: ', connection.localPort);
});
```

#### Killing the Autossh Process

The autossh process will automatically die if the node process is closed, but you can manually kill the process using `kill`.

If you try to kill the ssh process from the command line while the node process is active, a new ssh tunnel will be established (which is the point of autossh). You will need to kill the node process first or call the `kill` method on the instance.

**Example 1**

``` javascript
const myAutossh = autossh({
  host: '111.22.333.444',
  username: 'root',
  localPort: 64444,
  remotePort: 5432
})
.on('connect', connection => {
  console.log('connected: ', connection);
});

myAutossh.kill();
```

**Example 2**

``` javascript
autossh({
  host: '111.22.333.444',
  username: 'root',
  localPort: 64444,
  remotePort: 5432
})
.on('connect', connection => {
  console.log('connected: ', connection);
  connection.kill();
});
```

#### Adjusting `serverAliveInterval` and `serverAliveCountMax`

These two options are the bread and butter butter as far as polling the ssh connection.

Basically, `serverAliveInterval` is an interval (in seconds) for how often we should ping the ssh connection and check if the connection is established. 

The `serverAliveCountMax` is a count for how many failed `serverAliveInterval` checks until we close the connection.

For example, if `serverAliveInterval=10` and `serverAliveCountMax=1` then the ssh connection would be checked every 10 seconds, and if there is 1 failure, then close (and, in the case of autossh, restart) the connection. If the connection never fails, then there will be no restart.

One more example, if `serverAliveInterval=5` and `serverAliveCountMax=0` then the ssh connection would be checked every 5 seconds, and if there are 0 failures, then close and restart the connection. The 0 means it doesn't care if there is a failure or not--close (and restart) every 5 seconds, regardless!

The default values are `serverAliveInterval=120` (120 seconds) and `serverAliveCountMax=1`.

You can set these options in the object you pass to `autossh`.

``` javascript
autossh({
  host: '111.22.333.444',
  username: 'root',
  localPort: 'auto',
  remotePort: 5432,
  serverAliveInterval: 30,
  serverAliveCountMax: 1
})
.on('connect', connection => {
  console.log('connected: ', connection);
  console.log('localPort: ', connection.localPort);
});
```

#### Specifying the Private Key File

Select a file from which the identity (private key) for public key authentication is read. The default is `~/.ssh/id_rsa`. 

You can set the private file path as `privateKey` in the object you pass to `autossh`.

```javascript
autossh({
  host: '111.22.333.444',
  username: 'root',
  localPort: 64444,
  remotePort: 5432,
  privateKey: '~/.ssh/github_rsa'
})
.on('error', err => {
  console.error('ERROR: ', err);
})
.on('connect', connection => {
  console.log('Tunnel established on port ' + connection.localPort);
  console.log('pid: ' + connection.pid);
});
```

#### Adjusting/Disabling Max Poll Count

When first trying to establish the ssh tunnel, `autoshh` will poll the local port until the connection has been established. The default max poll count is `30`.

**Adjusting the max poll count**

Set the `maxPollCount` property in the object passed to `autossh`:

```javascript
autossh({
  host: '111.22.333.444',
  username: 'root',
  localPort: 'auto',
  remotePort: 5432,
  maxPollCount: 50
})
.on('connect', connection => {
  console.log('connected: ', connection);
});
```

**Disabling the max poll count**

Set the `maxPollCount` property to `0` or `false` in the object passed to `autossh`:

```javascript
autossh({
  host: '111.22.333.444',
  username: 'root',
  localPort: 'auto',
  remotePort: 5432,
  maxPollCount: false
})
.on('connect', connection => {
  console.log('connected: ', connection);
});
```

**Warning:** The max poll count is there to prevent `autossh` from infinitely polling the local port. Rather than disabling it, it may be wise to set it to a high number (e.g. `500`).

#### Specifying a Different SSH Port

The designated port for SSH according to the Transmission Control Protocol (TCP) is port 22, but you can specify a different port if you are using a different port. Set the `sshPort` property in the object you pass to `autossh`.

```javascript
autossh({
  host: '111.22.333.444',
  username: 'root',
  localPort: 'auto',
  remotePort: 5432,
  sshPort: 9999
});
```