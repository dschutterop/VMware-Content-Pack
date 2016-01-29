The more recent versions of Graylog2 (0.9x and up) support the use of content packs. Not only is this an ideal way of sharing streams, dashboards and inputs with, say, the community, it also provides a nice backup for the times you wish you hadn't just deleted the input (or stream, dashboard) and want to re-create it.

You can add the content packs under System -> Content Packs in your Graylog2 web interface at any time.

This content pack provides you with:
- vmWare: vMotionBandwidth stream
- vmWare: Can't convert IP address of type 0 stream
- my humble vmWare dashboard
- A few vmWare extractors

Since vmWare doesn't output RFC compliant syslog, I use a logStash conversion instance on my Graylog2 servers.

My logstash config for vmWare syslog messages:

input {
    syslog { port => 1514 }
}

filter {
    dns {   reverse => [ "host" ]
            action  => [ "replace" ]
            add_tag => [ "dns" ] 
        }
}

output {
    gelf {  chunksize => 1420
            facility  => 'logstash-gelf'
            host      => '127.0.0.1'
            port      => 12204 
        }
}

As you can see, I catch the syslog info on port 1514 (a port which is easily configured as syslog port in ESX).Logstash then converts the (now valid) syslog message into the GELF format we know and love and sends it to port 12204 on the local host, where we have a (global) GELF UDP listener.

It's easy as Pi.
