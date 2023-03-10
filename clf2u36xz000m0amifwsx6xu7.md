---
title: "Integrating MQTT into a Laravel project"
datePublished: Fri Mar 10 2023 17:50:50 GMT+0000 (Coordinated Universal Time)
cuid: clf2u36xz000m0amifwsx6xu7
slug: integrating-mqtt-into-a-laravel-project
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/sPLLVFJXlb8/upload/8e5969c898705d012ddc3430beb266c4.jpeg
tags: laravel, iot, mqtt, embedded-systems

---

Recently, I have been experimenting with [MQTT](https://www.hivemq.com/mqtt-essentials/), a lightweight messaging protocol that is designed to provide efficient, reliable, and real-time communication between devices and applications in machine-to-machine (M2M) and Internet of Things (IoT) contexts. It's a popular communication protocol used by IoT devices. For the project I am working on, I plan to send data from my device to an MQTT broker and have a web application subscribe to a topic to get data the device is publishing.

I installed [Mosquitto](https://mosquitto.org/), a popular MQTT broker and [MQTTX](https://mqttx.app/), an easy-to-use GUI MQTT client. I found a PHP library called [PHP-MQTT](https://github.com/php-mqtt/laravel-client) that makes integrating MQTT into your Laravel project easy. The code I used to get data from the MQTT broker is quite simple.

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use PhpMqtt\Client\Facades\MQTT;

class SubscribeToTopic extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'mqtt:subscribe';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Subscribe To MQTT topic';

    /**
     * Execute the console command.
     *
     * @return int
     */
    public function handle()
    {
        $mqtt = MQTT::connection();
        $mqtt->subscribe('devices/+/status', function(string $topic, string $message) {
            echo sprintf('Received message on topic [%s]: %s',$topic, $message);
        });

        $mqtt->loop(true);
        return Command::SUCCESS;
    }
}
```

![Publishing data to the MQTT Topic](https://cdn.hashnode.com/res/hashnode/image/upload/v1677678517367/b535f1c1-dbf0-4d37-95a8-c165e0e81b5b.png align="center")

![Data received by laravel application](https://cdn.hashnode.com/res/hashnode/image/upload/v1677678567033/871ed8ed-e8e8-4f0b-a594-72de466465cd.png align="center")

The code worked well locally, but I started thinking about how to run the command when deploying the project to production. My initial idea was to create a command and run it with Laravel's scheduler, but this approach did not work well.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677679090622/ec875518-b6e1-4952-b7ab-268d0e530942.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677679339694/957093ed-2735-4438-9808-366d6ded252a.png align="center")

The issue is that the command runs an infinite loop, causing the function to never return. Consequently, the scheduler creates multiple processes, all subscribed to the same topic. The code that runs when new data is received (e.g., storing the data in a database) will be executed multiple times, with the exact number depending on the number of processes created. To solve this problem, you can use a process monitor, such as (supervisor)\[[http://supervisord.org/](http://supervisord.org/)), to run the command in the background and restart it if it crashes. I copied the configuration example for running the queue worker command from the [Laravel documentation](https://laravel.com/docs/10.x/queues#configuring-supervisor) and edited it as follows:

```plaintext
[program:laravel-mqtt-subscriber]
process_name=%(program_name)s_%(process_num)02d
command=php /home/forge/app.com/artisan mqtt:subscribe
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=forge
numprocs=1
redirect_stderr=true
stdout_logfile=/home/forge/app.com/worker.log
stopwaitsecs=3600
```

The main point to note is that I changed the value of ***numprocs*** to 1 to prevent multiple processes from running the same command, which resolved the issue.