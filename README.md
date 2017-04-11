### To install on Ubuntu:

````
cd /usr/share/munin/plugins
git clone git://github.com/tjstein/php5-fpm-munin-plugins.git
chmod +x php5-fpm-munin-plugins/phpfpm_*
ln -s /usr/share/munin/plugins/php5-fpm-munin-plugins/phpfpm_average /etc/munin/plugins/phpfpm_average
ln -s /usr/share/munin/plugins/php5-fpm-munin-plugins/phpfpm_connections /etc/munin/plugins/phpfpm_connections
ln -s /usr/share/munin/plugins/php5-fpm-munin-plugins/phpfpm_memory /etc/munin/plugins/phpfpm_memory
ln -s /usr/share/munin/plugins/php5-fpm-munin-plugins/phpfpm_status /etc/munin/plugins/phpfpm_status
ln -s /usr/share/munin/plugins/php5-fpm-munin-plugins/phpfpm_processes /etc/munin/plugins/phpfpm_processes
service munin-node restart
```

For the phpfpm_status and phpfpm_connections plugins, you'll need to enable the _status_ feature included in versions 5.3.2+ of PHP-FPM. The directive can be found in the php5-fpm.conf file:
```
pm.status_path = /status
```

Jérôme Loyet from the Nginx forums provided some useful insight on how to get this working with Nginx. You'll essentially set up the status location directive like this:

```
location ~ ^/(status|ping)$ {
    include fastcgi_params;
    fastcgi_pass backend;
    fastcgi_param SCRIPT_FILENAME $fastcgi_script_name;
    allow 127.0.0.1:9000;
    allow stats_collector.localdomain;
    allow watchdog.localdomain;
    deny all;
}
```

You'll need to make sure that from within your box, you can curl /status with # curl http://localhost/status. You should get a response similar to this:

```
accepted conn:    40163
pool:             www
process manager:  dynamic
idle processes:   6
active processes: 0
total processes:  6
```

#### Note: 

The phpfpm_status plugin is particularly useful if you're using dynamic or on-demand process management. You can choose static, dynamic or on-demand in the php5-fpm.conf.

### Environment variables

* env.url: Set a custom url, defaults to _http://127.0.0.1/status_
* env.ports: Set a custom port, defaults to _80_
* env.phpbin: Set a custom php binary name, defaults to _php5-fpm_
* env.phppool: Set a custom php pool, defaults to www
* env.warning: Set the warning alarm, triggered when % of active/total connections higher than the defined value. Defaults to 0.6 (60%).
* env.critical: Set the critical alarm, triggered when % of active/total connections higher than the defined value. Defaults to 0.8 (80%).

#### Requirements:

libwww-perl