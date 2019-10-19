# KSQL
confluent KSQL examples 

Step : Start Confluent Platform using the Confluent CLI.  
/home/nirav/nlangaliya/confluent-5.2.1/bin/confluent start


**Avro producer code**

```python
# -*- coding: utf-8 -*-
"""
Created on 10/1/19 12:00 PM

@author : Nirav Langaliya
"""

# File Name : AvroProducerKafkaFakeData.py

# Enter feature description here

# Enter steps here

from confluent_kafka import avro
from confluent_kafka.avro import AvroProducer
import json
from faker_schema.faker_schema import FakerSchema
import datetime,time



def delivery_report(err, msg):
    """ Called once for each message produced to indicate delivery result.
        Triggered by poll() or flush(). """
    if err is not None:
        print('Message delivery failed: {}'.format(err))
    else:
        print('Message delivered to {} [{}]'.format(msg.topic(), msg.partition()))


now = datetime.datetime.now()


schema_path = "//Users/nlangaliya/PycharmProjects/confluentkafkaexample/com_example_avro_testV2.avsc"

schema_read = avro.load(schema_path)

"""
avroProducer = AvroProducer(
        {
            'bootstrap.servers': 'localhost:9092',
            'schema.registry.url': 'http://localhost:8081'
        }
        , default_value_schema=schema_read)
"""
avroProducer = AvroProducer(
        {
            'bootstrap.servers': 'dgphxkfkap015.phx.gapinc.dev:9092,dgphxkfkap016.phx.gapinc.dev:9092,dgphxkfkap017.phx.gapinc.dev:9092',
            'schema.registry.url': 'http://dgphxkfkap015.phx.gapinc.dev:8081'
        }
        , default_value_schema=schema_read)
"""
schema = {'Name': 'name',
          'ID' : 'uuid4',
          'Contact': {'Email': 'email', 'PhoneNumber': 'phone_number'},
          'LocationDetails': {'CountryCode': 'country_code','City': 'city', 'Country': 'country', 'PostalCode': 'postalcode','Address': 'street_address'}
          }
"""
schema = {'Name': 'name',
          'ID' : 'uuid4',
          'Contact': {'Email': 'email', 'PhoneNumber': 'phone_number','WorkAddress':'address'},
          'LocationDetails': {'CountryCode': 'country_code','City': 'city', 'Country': 'country', 'PostalCode': 'postalcode','Address': 'street_address'}
          }

faker = FakerSchema()
RecCounter=0
for i in range(1,20000000000):
    try:
        data = faker.generate_fake(schema)
        line = str(data).replace("'",'"')
        try:
            line = json.loads(line.strip())
            line['DateTime'] = now.strftime("%Y-%m-%d %H:%M:%S")
            line['RecNumber'] = str(time.time()).replace('.','')
            RecCounter = RecCounter + 1
            line['RecCounter'] = str(RecCounter)
        except:
            print (data)
            print (line)
            print("Error while trying to convert data to json", data)
            continue
        #print(line)
        avroProducer.produce(topic='EDW_PART_DATA', value=line,callback=delivery_report)
        avroProducer.poll(1)
        #time.sleep(1)
    except:
        print("Error while trying to publish data",data)
        raise

avroProducer.flush()
```

