# Laravel Kafka
![docs/laravel-kafka.png](docs/laravel-kafka.png)

[![Continuous Integration](https://github.com/mateusjunges/laravel-kafka/actions/workflows/run-tests.yml/badge.svg)](https://github.com/mateusjunges/laravel-kafka/actions/workflows/run-tests.yml)
[![Check & fix styling](https://github.com/mateusjunges/laravel-kafka/actions/workflows/php-cs-fixer.yml/badge.svg)](https://github.com/mateusjunges/laravel-kafka/actions/workflows/php-cs-fixer.yml)

Do you use Kafka in your laravel packages? All packages I've seen until today, including some built by myself, does not provide a nice
syntax usage syntax or, if it does, the test process with these packages are very painful.

This package provides a nice way of producing and consuming kafka messages in your Laravel projects.

Follow these docs to install this package and start using kafka with ease.

# Installation
To install this package, you must have installed PHP RdKafka extension. You can follow the steps [here](https://github.com/edenhill/librdkafka#installation)
to install rdkafka in your system.

With RdKafka installed, require this package with composer:

```bash
composer require mateusjunges/laravel-kafka
```

# Usage
After installing the package, you can start producing and consuming Kafka messages.

# Producing Kafka Messages
To publish your messages to Kafka, you can use the `publishOn` method, of `Junges\Kafka\Facades\Kafka` class:

```php
use Junges\Kafka\Facades\Kafka;

Kafka::publishOn('broker', 'topic');
```

This method returns a `Junges\Kafka\Producers\ProducerBuilder::class` instance, and you can configure your message.

The `ProducerBuilder` class contains a few methods to configure your kafka producer. The following methods describes these methods.

## ProducerBuilder configuration methods
The `withConfigOption` method sets a `\RdKafka\Conf::class` option. You can check all available options [here][rdkafka_config].
This methods set one config per call, and you can use `withConfigOptions` passing an array of config name and config value 
as argument. Here's an example:

```php
use Junges\Kafka\Facades\Kafka;

Kafka::publishOn('broker', 'topic')
    ->withCOnfigOption('property-name', 'property-value')
    ->withConfigOptions([
        'property-name' => 'property-value'
    ]);
```

While you are developing your application, you can enable debug with the `withDebugEnabled` method.
To disable debug mode, you can use `->withDebugEnabled(false)`, or `withDebugDisabled` methods.

```php
use Junges\Kafka\Facades\Kafka;

Kafka::publishOn('broker', 'topic')
    ->withCOnfigOption('property-name', 'property-value')
    ->withConfigOptions([
        'property-name' => 'property-value'
    ])
    ->withDebugEnabled() // To enable debug mode
    ->withDebugDisabled() // To disable debug mode
    ->withDebugEnabled(false) // Also to disable debug mode
```
### Configuring the Kafka message payload
In kafka, you can configure your payload with a message, message headers and message key. All these configurations are available 
within `ProducerBuilder` class.

### Configuring message headers
To configure the message headers, use the `withHeaders` method:

```php
use Junges\Kafka\Facades\Kafka;

Kafka::publishOn('broker', 'topic')
    ->withHeaders([
        'header-key' => 'header-value'
    ])
```

### Configure the message payload
You can configure the message with the `withMessage` or `withMessageKey` methods. 

The `withMessage` sets the entire message, and it accepts a `Junges\Kafka\Message::class` instance as argument.

This is how you should use it:

```php
use Junges\Kafka\Facades\Kafka;
use Junges\Kafka\Message;

$message = new Message(
    headers: ['header-key' => 'header-value'],
    message: ['key' => 'value'],
    key: 'kafka key here'  
)

Kafka::publisOn('broker', 'topic')->withMessage($message);
```

The `withMessageKey` method sets only a key in your message.

```php
use Junges\Kafka\Facades\Kafka;

Kafka::publisOn('broker', 'topic')->withMessageKey('key', 'value');
```

### Using Kafka Keys
In Kafka, keys are used to determine the partition within a log to which a message get's appended to.
If you want to use a key in your message, you should use the `withKafkaKey` method:

```php
use Junges\Kafka\Facades\Kafka;

Kafka::publisOn('broker', 'topic')->withKafkaKey('your-kafka-key');
```

## Sending the message to Kafka
After configuring all your message options, you must use the `send` method, to send the message to kafka.

```php
use Junges\Kafka\Facades\Kafka;

/** @var \Junges\Kafka\Producers\ProducerBuilder $producer */
$producer = Kafka::publisOn('broker', 'topic')
    ->withConfigOptions(['key' => 'value'])
    ->withKafkaKey('your-kafka-key')
    ->withKafkaKey('kafka-key')
    ->withHeaders(['header-key' => 'header-value']);

$producer->send();
```
# Consuming Kafka Messages
If your application needs to read messages from a Kafka topic, you must create a consumer object, subscribe to the appropriate topic
and start receiving messages. 

To create a consumer using this package, you can use the `createConsumer` method, on Kafka facade:

```php
use Junges\Kafka\Facades\Kafka;

$consumer = Kafka::createConsumer('broker');
```

This method returns a `Junges\Kafka\Consumers\ConsumerBuilder::class` instance, and you can use it to configure your consumer.

## Subscribing to a topic
With a consumer created, you can subscribe to a kafka topic using the `subscribe` method:

```php
use Junges\Kafka\Facades\Kafka;

$consumer = Kafka::createConsumer('broker')->subscribe('topic');
```

Of course, you can subscribe to more than one topic at once, either using an array of topics or specifying one by one:

```php
use Junges\Kafka\Facades\Kafka;

$consumer = Kafka::createConsumer('broker')->subscribe('topic-1', 'topic-2', 'topic-n');

// Or, using array:
$consumer = Kafka::createConsumer('broker')->subscribe([
    'topic-1',
    'topic-2',
    'topic-n'
]);
```

## Configuring consumer groups
Kafka consumers belonging to the same consumer group share a group id. THe consumers in a group divides the topic partitions as fairly amongst themselves as possible by
establishing that each partition is only consumed by a single consumer from the group.

To attach your consumer to a consumer group, you can use the method `withConsumerGroupId` to specify the consumer group id:

```php
use Junges\Kafka\Facades\Kafka;

$consumer = Kafka::createConsumer('broker')->withConsumerGroupId('foo');
```

## 




[rdkafka_config]:https://github.com/edenhill/librdkafka/blob/master/CONFIGURATION.md

