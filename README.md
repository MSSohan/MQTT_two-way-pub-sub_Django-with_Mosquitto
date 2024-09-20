# MQTT-Client-Django-With-Core-Python-Two-Way-Communication-Using-Mosquitto-Broker

[MQTT](https://mqtt.org/) is a lightweight IoT messaging protocol based on publish/subscribe model, which can provide real-time reliable messaging services for connected devices with very little code and bandwidth. It is widely used in industries such as IoT, mobile Internet, smart hardware, [Internet of vehicles](https://www.emqx.com/en/blog/category/internet-of-vehicles), and power and energy.

[Django](https://www.djangoproject.com/) is an open-source Web framework and one of the more popular Python Web frameworks. This article mainly introduces how to connect, subscribe, unsubscribe, and send and receive messages between [MQTT client](https://www.emqx.com/en/blog/mqtt-client-tools) and MQTT Broker in the Django project.

We will write a simple MQTT client using [paho-mqtt](https://www.eclipse.org/paho/index.php?page=clients/python/index.php) client library. `paho-mqtt` is a widely used MQTT client library in Python that provides client support for MQTT 5.0, 3.1.1, and 3.1 on Python 2.7 and 3.x.


## Project Initialization

This project uses Python 3.8 for development testing, and the reader can confirm the version of Python with the following command.

```
$ python3 --version
Python 3.11.0
```
Create Virtual Environment.
```
python -m venv env
```
Active virtual environment and then,

Install Django and `paho-mqtt` using Pip.

```
pip install django
pip install paho-mqtt
```

Create a Django project.

```
django-admin startproject mqtt-test
```

The directory structure after creation is as follows.

```
├── manage.py
└── mqtt_test
  ├── __init__.py
  ├── asgi.py
  ├── settings.py
  ├── urls.py
  ├── views.py
  └── wsgi.py
```

## Using paho-mqtt

- Broker: `localhost`
- TCP Port: `1883`
- Websocket Port: `8083`

### Import paho-mqtt

```
import paho.mqtt.client as mqtt
```

### Writing connection callback

Successful or failed MQTT connections can be handled in this callback function, and this example will subscribe to the `uprint/kiosk` topic after a successful connection.

```
def on_connect(mqtt_client, userdata, flags, rc):
   if rc == 0:
       print('Connected successfully')
       mqtt_client.subscribe('uprint/kiosk')
   else:
       print('Bad connection. Code:', rc)
```

### Writing message callback

This function will print the messages received by the `uprint/kiosk` topic.

```
def on_message(mqtt_client, userdata, msg):
   print(f'Received message on topic: {msg.topic} with payload: {msg.payload}')
```

### Adding Django configuration items

Add configuration items for the MQTT broker in `settings.py`. Readers who have questions about the following configuration items and MQTT-related concepts mentioned in this article can check out the blog [The Easiest Guide to Getting Started with MQTT](https://www.emqx.com/en/blog/the-easiest-guide-to-getting-started-with-mqtt).

> This example uses anonymous authentication, so the username and password are set to empty.

```
MQTT_SERVER = 'localhost'
MQTT_PORT = 1883
MQTT_KEEPALIVE = 60
MQTT_USER = ''
MQTT_PASSWORD = ''
```

### Configuring the MQTT client

```
client = mqtt.Client()
client.on_connect = on_connect
client.on_message = on_message
client.username_pw_set(settings.MQTT_USER, settings.MQTT_PASSWORD)
client.connect(
    host=settings.MQTT_SERVER,
    port=settings.MQTT_PORT,
    keepalive=settings.MQTT_KEEPALIVE
)
```

### Creating a message publishing API

We create a simple POST API to implement MQTT message publishing.

> In actual applications, the API code may require more complex business logic processing.

Add the following code in `views.py`.

```
import json
from django.http import JsonResponse
from mqtt_test.mqtt import client as mqtt_client


def publish_message(request):
    request_data = json.loads(request.body)
    rc, mid = mqtt_client.publish(request_data['topic'], request_data['msg'])
    return JsonResponse({'code': rc})
```

Add the following code in `urls.py`.

```
from django.urls import path
from . import views

urlpatterns = [
    path('publish', views.publish_message, name='publish'),
]
```

### Start the MQTT client

Add the following code in `__init__.py`.

```
from . import mqtt
mqtt.client.loop_start()
```

At this point, we have finished writing all the code, and the full code can be found at [GitHub](https://github.com/MSSohan/MQTT-Client-Django).

Finally, execute the following command to run the Django project.

```
python manage.py runserver
```

When the Django application starts, the MQTT client will connect to the MQTT Broker and subscribe to the topic `uprint/kiosk`.
