# cocotask
### Build task queue on either Rabbitmq, Kafka or Redis! Simple! Easy! and FAST!!!!

**Why creating this framework?**
* Most task queues using Rabbitmq/Kafka/Redis are doing the same thing, but there is no unified wrapper. It's a waste of time to write code for different MQs if all you need is a task queue to distribute jobs
* No need to worry about using Rabbitmq, Kafka or Redis. Just pick one and go! It's simply a few lines of config changes in future if you want to switch to a different underlying system
* Hide all details for connecting/subscribing/publishing/etc. You can just use the same API for any of Kafka/Rabbitmq/Redis!
* Minimize the efforts for team members to handcraft the code for exchange/queue handling
* Team members should focus on how to handle messages
* There lacks good mananger tool/lib to handle creating multiple consumers (although it's simple)
* Celery is the original idea, but it doesn't support Kafka and Windows platform. It's also tightly coupled with Python alone.
<hr>

## Test

1. install rabbitmq or kafka on local machine (either docker or pure rabbitmq). 
   - Rabbitmq: https://www.rabbitmq.com/
   - Kafka: https://kafka.apache.org/quickstart  (for kafka, you have to manually create a topic named `test_topic_1` in order to run the test. In order to try multiple consumers, you need to set partitions to 2 or above, not 1)
   - Redis: https://redis.io/download

2. make sure it's running Python 3.5 above

3. pip install pika kafka-python redis jsmin

4. pip install cocotask

5. now you have cocotask setup on your machine. To test, you can go to the ./test folder
   - Run: `python producer_test.py`   (this will post a string. Code is very simple)
   - In another window, under ./test, run:  **`cocotask ./config.json userworkers TestWorker 4`** 
     you'll see the worker starts and process 1 message we just posted.

6. So the key part is how we use cocotask command tool. the parameters are:
   - config_path: path to config file
   - module_name: module of your customized worker class
   - class_name: the class name of your own worker class (in the example above, it's userworkers.TestWorker, so the module name is userworkers, the class name is TestWorker)
   - number of workers: the total number of customer workers
   - logginglevel(optional): python logging level INFO/DEBUG/etc.
   - modulepath(optional): the relative path to find the module you defined. Default is '.'

7. Develop your own worker class and try

## Development

**Build your own worker**

```
from cocotask import CocoBaseWorker

class TestWorker(CocoBaseWorker):

    def process(self, body):
        print(body)
```
Check userworkers/test_worker.py for reference.

**Post a message**
```
from cocotask import CocoProducerManager as pm
import json

with open('config.json', 'r') as f:
    config = json.load(f)

producer = pm.create_instance(config)

producer.connect()
producer.send('12345678')
producer.close()
```
**Switch from RabbitMQ to Kafka or reverse**
In test/config.json
```
{
    "MQ_TYPE": "RMQ",  # change this to KAFKA if your underlying MQ is KAFKA

    "RMQ": {
        ...      
    },

    "KAFKA": {
        ...
    },

    "REDIS": {
        ...
    }
}

```
**We do support SASL_PLAINTEXT for kafka and simple auth in Redis as in the comments of the config file. Check their website to see how to setup the authentication**

You can build your own dictionary object as configuration for sure, as long as it contains the required fields.


That's it. Simple and straightforward.
