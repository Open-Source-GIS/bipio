bipio
=========

Welcome to the BipIO API Server. 

BipIO is Billion Instructions Per I/O - For People and Robots.  

[![NPM](https://nodei.co/npm/bipio.png?downloads=true)](https://nodei.co/npm/bipio/)

BipIO is a highly parallel nodejs based API integration framework (iPaas).  It uses [graph](http://en.wikipedia.org/wiki/Directed_graph) 
based <a href="http://en.wikipedia.org/wiki/Pipeline_(software)">pipelines</a> to create ephemeral endpoints, complex automated workflows and message distribution hubs with 3rd party API's and
[RPC's](http://en.wikipedia.org/wiki/Remote_procedure_call).  It's a RESTful JSON API that supports account level namespacing and multiple domains ([fqdn](http://en.wikipedia.org/wiki/Fully_qualified_domain_name)) per account.  Clients authenticate over HTTP Basic.

If you're familiar with Yahoo Pipes, IFTTT, Zapier, Mulesoft, Cloudwork or Temboo - the concept is a little similar. The server has a small footprint which lets you create and automate an internet of things that matter to you.   It can be installed alongside your existing open source app or prototype for out-of-band message transformation, feed aggregation, queuing, social network fanout or whatever you like, even on your Rasberry Pi.

The graph definitions, which are called [bips](https://bip.io/docs/resource/rest/bip), allow you to transform content 
between adjacent nodes and chain outputs to inputs indefinitely across disparate 'cloud' services.  You can put web-hooks, 
web-sockets, emails or event triggers infront of Bip graphs to perform useful work.

Some of their characteristics include :

 - dynamic or automatically derived naming
 - pausing or self-destructing after a certain time or impressions volume
 - binding to connecting clients with soft ACLs over the course of their 'life'
 - able to be reconfigured dynamically without changing a client implementation
 - infinitely extensible, from any channel to any other channel.
 - can serve (render) protected channel content while inheriting all of the above characteristics

Bipio is dynamic, flexible, fast, modular, opinionless and gplv3 open source.

![concept](https://bip.io/static/img/docs/bip_concept.png)

#### Bips & Channels

Bips and Channels are 1st class API resources.

Bips are configured by defining a graph ([hub](https://bip.io/docs/resource/rest/bip#resource_rest_bip_hubs)) 
across nodes ([channels](https://bip.io/docs/resource/rest/channel)) along with certain other metadata which defines the flavor,
lifespan and overall characteristics of the endpoint or trigger.  It's a fairly large topic, find out more in 
[the wiki](https://github.com/bipio-server/bipio/wiki/Bips).

Channels are reusable entities which perform a discrete unit of work and emit a predictable result.  The collection of channels
you create becomes something like a swatch from which you can orchestrate complex API workflows.  When dropped onto a bip's graph,
a channels export then becomes the next adjacent channels transformed import.
Parallel delivery is handled by an [AMQP](http://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol) transport to 
[RabbitMQ](http://www.rabbitmq.com/), and every edge of a bips graph can be independently processed by any subscribing 
node in the cluster.

Channels are largely decoupled from the graph resolution platform in self contained collections called Pods.  'Self Contained' 
meaning they are free from other system concerns and can operate independently.  For example you can pass authenticated API 
calls or 'render' channels without them having to be an active graph participant.  Channels can therefore store, track, serve or transform 
content and messages as part of a pipeline or in autonomous isolation.  For an authoritative list of officially
supported services, please see the bip-pod-* repos via [https://github.com/bipio-server](https://github.com/bipio-server) or
check the [website](https://bip.io/docs/pods) for quick at-a-glance channel definitions that you can use out of the box.  

Keep in mind that Pods and Channels are modular, so feel free to create your own.  As long as they honor the basic pod 
interface, your custom Pods should drop right in!  Take a look at the [Pod Boilerplate](https://github.com/bipio-server/bip-pod)
to get started.

##### Super Easy Integrations

Here's a quick example, lets say I have a private email address that I want to protect or obfuscate - I could use an SMTP bip to
create a temporary relay which will forward emails for 1 day only.  

**Here's how :**

Create an SMTP Forwarder Channel to email me with any messages it receives :
```
POST /rest/channel
{
 action : "email.smtp_forward",
 name : "Helo FuBa"
 config : {
   rcpt_to : "foo@bar.net"
 }
}

RESPONSE
{
 id : "206fe27f-5c98-11e3-8ad3-c860002bd1a4"
}
```

... And now I I can use that channel anywhere, its a permanent fixture in the system until explicitly deleted.  So now
to create the relay, I can build a simple SMTP bip with a single edge pointing to the 'Helo FuBa' channel :

```
POST /rest/bip
{
 type : "smtp",
 hub : {
   "source" : {
      edges : [ "206fe27f-5c98-11e3-8ad3-c860002bd1a4" ],
      transforms : {
        "206fe27f-5c98-11e3-8ad3-c860002bd1a4" : {
          "subject" : "[%source#subject%]",
          "body_html" : "[%source#body_html%]",
          "body_text" : "[%source#body_text%]",
          "reply_to" : "[%source#reply_to%]",
        }
      },
     _note : "^^ Transforms aren't mandatory, but here for illustration - you only need an edge"
   }
 },
 end_life : {
   imp : 0,
   time : '+1d'
 },
 note : "No name, no problem.  Let the system generate something short and unique"
}

RESPONSE
{
 name : "lcasKQosWire22"
 _repr : "lcasKQosWire22@yourdomain.net"
}
```

And thats it. We actually have a little [chrome extension](http://goo.gl/ZVIkfr) which does just this for web based email forms!
For extra credit, I could store attachments arriving on that email address straight to dropbox by just adding an edge - check out
how in the [cookbook](https://github.com/bipio-server/bipio/wiki/Email-Repeater,-Dropbox-Attachment-Save)

### Please Note

The BipIO server software is the basic framework for processing bips and their delivery graphs and is currently distributed headless.
For graphical representation of your bips, sign in to [bipio](https://bip.io) to mount your local install from your browser 
under My Account > Mounts > Create Mount.  

![Server Mount](https://bip.io/static/img/docs/server_mount.png)

The BipIO website is not a first class citizen or tightly coupled to one particular 
endpoint, so you can mount your local install(s) even if behind a firewall.  The dashboard will be migrated into express static
middleware and distributed with Bipio at a later date.

By itself, Bipio does not provide SSL termination or any load balancing beyond [node-cluster](http://nodejs.org/api/cluster.html).  If you need SSL termination this should be delegated to a forward proxy such as NginX, Apache, HAProxy etc.

Feel free to fork this repository and create pods as you need, and please help make 
[the community](https://groups.google.com/forum/#!forum/bipio-api) a better place. Pull Requests, issues, feature requests, 
integration ideas, general communication is always welcome.

Hosted/Commercial OEM solutions can be found at [https://bip.io](https://bip.io). Read the License section at the end of this 
readme for important info.

## Requirements

  - [Node.js >= 0.10.15](http://nodejs.org) **API and graph resolver**
  - [MongoDB Server](http://www.mongodb.org) **data store**
  - [RabbitMQ](http://www.rabbitmq.com) **message broker**

SMTP Bips are available out of the box with a Haraka plugin.  Configs under [bipio-contrib/haraka](https://github.com/bipio-server/bipio-contrib).

  - [Haraka](https://github.com/baudehlo/Haraka)

## Installation

    npm install bipio
    make install
    node ./src/server.js

Be sure to have a MongoDB server and Rabbit broker ready and available before install.  Otherwise, follow the prompts
during the `make install` script to get a basically sane server running that you can play with.

The server ships with several Pod dependencies which you can use right away - Email, Text/HTML Templating, Flow Control and Syndication.
Additional Pods can be found in the [GitHub Repository](https://github.com/bipio-server/bipio).  The [Pods](https://github.com/bipio-server/bipio/wiki/Pods)
section in the Wiki will guide any future installs.

For Ubuntu users, a sample upstart script is supplied in config/upstart_bip.conf which should be copied to 
/etc/init and reconfigured to suit your environment.  If you'd like it managed by Monit...

## Updating

Updating BipIO via `npm` will resolve any new dependencies for you, however if you're checking out from the repository 
directly with `git pull` you may need to manually run `npm install` to download any new dependencies (bundled pods, for example).

If you're going the `git pull` route and want to save this step, create a git 'post merge' hook by copying it from `./tools` like so :

    mkdir -p .git/hooks
    cp ./tools/post-merge .git/hooks
    chmod ug+x .git/hooks/post-merge

This will automatically install any missind dependencies every time you `git pull`

### Monit Config

/etc/monit/config.d/bipio.conf

    #!monit
    set logfile /var/log/monit.log

    check process node with pidfile "/var/run/bip.pid"
        start program = "/sbin/start bipio"
        stop program  = "/sbin/stop bipio"
        if failed port 5000 protocol HTTP
            request /
            with timeout 10 seconds
            then restart


### Crons

Periodic tasks will run from the server master instance automatically, you can find the config
in the `config/{environment}.json` file, keyed by 'cron'.  

* stats - network chord stats, every hour
* triggers - trigger channels, every 15 minutes
* expirer - bip expirer, every hour

To disable a cron, either remove it from config or set an empty string.

To have these crons handled by your system scheduler rather than the bipio server, disable the crons
in config as described.  Wrapper scripts can be found in ./tools for each of stats (`tools/generate-hub-stats.js`), 
triggers (`tools/bip-trigger.js`) and expirer (`tools/bip-expire.js`).

Here's some example wrappers.

#### Trigger Runner

Cron:
    */15 * * * * {username} /path/to/bipio/tools/trigger-runner.sh

trigger-runner.sh :

    #!/bin/bash
    # trigger-runner.sh
    export NODE_ENV=production
    export HOME="/path/to/bipio"
    cd $HOME (date && node ./tools/bip-trigger.js ) 2>&1 >> /path/to/bipio/logs/trigger.log

#### Expire Runner

Cron:
    0 * * * * {username} /path/to/bipio/tools/expire-runner.sh

expire-runner.sh :

    #!/bin/bash
    # expire-runner.sh
    export NODE_ENV=production
    export HOME="/path/to/bipio"
    cd $HOME (date && node ./tools/bip-expire.js ) 2>&1 >> /path/to/bipio/logs/cron_server.log

#### Stats Runner

Cron:
    */15 * * * * {username} /path/to/bipio/tools/stats-runner.sh

stats-runner.sh :

    #!/bin/bash
    # stats-runner.sh
    export NODE_ENV=production
    export HOME="/path/to/bipio"
    cd $HOME (date && node ./tools/generate-hub-stats.js ) 2>&1 >> /path/to/bipio/logs/stats.log

## Documentation

General API spec and tutorials can be found at https://bip.io.  For server setup and configuration guides,
keep an eye on the [Wiki](https://github.com/bipio-server/bipio/wiki), it will be continuously updated.

## License

[GPLv3](http://www.gnu.org/copyleft/gpl.html)

Our open source license is the appropriate option if you are creating an open source application under a license compatible with the GNU GPLv3. 

If you'd like to integrate BipIO with your proprietary system, GPLv3 is likely incompatible.  To secure a Commercial OEM License for Bipio,
please [reach us](mailto:enquiries@cloudspark.com.au)

![Cloud Spark](http://www.cloudspark.com.au/cdn/static/img/cs_logo.png "Cloud Spark - Rapid Web Stacks Built Beautifully")
Copyright (c) 2010-2014  [CloudSpark pty ltd](http://www.cloudspark.com.au)
